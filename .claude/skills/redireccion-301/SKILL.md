---
name: redireccion-301
description: >
  Usa esta skill siempre que el usuario necesite validar redirecciones 301/302 de URLs.
  Actívala ante solicitudes como: "verificar redirecciones", "validar URLs de redirección",
  "comprobar redirecciones 301", "verificar que una URL redirecciona a otra",
  "validar redirecciones desde Excel", "auditar redirecciones web".
compatibility:
  required_tools:
    - Bash (para ejecutar el script Python)
    - Write (para crear el script temporal)
  python_packages:
    - playwright (pip install playwright && python -m playwright install chromium)
    - openpyxl (pip install openpyxl)
    - requests (pip install requests)
---

# Skill: Redireccion 301 — Validador de Redirecciones de URLs

## Objetivo

Verificar que las URLs definidas en un archivo Excel redireccionen correctamente a sus URLs destino correspondientes. Lee la columna A (URLs origen) y columna B (URLs destino), verifica línea por línea que cada redirección funcione correctamente, y genera un informe con capturas de pantalla y resumen del comportamiento encontrado.

> **Verifica códigos de estado HTTP**: Detecta redirecciones 301, 302, 307, 308 y valida que la URL final sea la esperada.

---

## Paso 1 — Recopilar parámetros

**El script buscará automáticamente archivos Excel (.xlsx, .xls) en la carpeta raíz donde se ejecute.**

**Estructura requerida del archivo Excel:**
> - **Columna A**: URLs origen (que deben redireccionar)  
> - **Columna B**: URLs destino (hacia donde deben redireccionar)
> - **Primera fila**: Puede contener headers (se omite automáticamente)

**Ubicación del archivo:**
> Coloca tu archivo Excel en la carpeta: `C:\Users\jesus\Desktop\Ilumno\redireccion-301\`

Ejemplos de nombres válidos:
- `redirecciones.xlsx`
- `urls.xlsx` 
- `validar_redirecciones.xls`

**Ruta de salida automática:**
```
./output/                    (carpeta en la raíz de ejecución)
./output/screenshots/        (capturas de pantalla)
```

---

## Paso 2 — Verificar y crear/corregir script Python

**El script se crea automáticamente solo la primera vez o cuando hay errores.**

1. **Verificar si existe** `validar_redirecciones.py` en la carpeta
2. **Si no existe**: Crear el archivo completo
3. **Si existe**: Verificar que esté completo y funcional
4. **Si tiene errores**: Corregir automáticamente en tiempo real
5. **Si está correcto**: Proceder directamente a ejecutar

> **Optimización**: Esta lógica evita recrear el archivo innecesariamente, ahorrando tiempo y recursos.

**Verificaciones automáticas que realiza:**
- ✓ Archivo existe y es legible
- ✓ Contiene todas las funciones requeridas
- ✓ Imports correctos (playwright, openpyxl, requests)
- ✓ Rutas de salida configuradas correctamente
- ✓ Función de búsqueda automática de Excel

**Código de verificación automática:**

```python
# Este código de verificación se implementa antes de la ejecución principal
import os
from pathlib import Path

def verificar_y_preparar_script():
    """Verifica si el script existe y está completo, lo crea/corrige si es necesario"""
    script_path = Path('./validar_redirecciones.py')
    
    # Verificaciones requeridas
    funciones_requeridas = [
        'find_excel_file', 'load_excel_data', 'verificar_redireccion_browser',
        'verificar_redireccion_requests', 'verificar_redirecciones', 
        'generate_excel_report', 'generar_resumen_txt'
    ]
    
    imports_requeridos = [
        'from playwright.async_api import async_playwright',
        'import openpyxl', 'import requests', 'import asyncio'
    ]
    
    if script_path.exists():
        # Leer y verificar contenido existente
        with open(script_path, 'r', encoding='utf-8') as f:
            contenido = f.read()
        
        # Verificar que contiene todo lo necesario
        tiene_funciones = all(func in contenido for func in funciones_requeridas)
        tiene_imports = all(imp.split()[1] in contenido for imp in imports_requeridos)
        tiene_rutas_correctas = 'Path().cwd()' in contenido and 'OUTPUT_DIR' in contenido
        
        if tiene_funciones and tiene_imports and tiene_rutas_correctas:
            print("✓ Script validar_redirecciones.py está completo y correcto")
            return True
        else:
            print("⚠️ Script incompleto o con errores, corrigiendo...")
            return False
    else:
        print("❌ Script no existe, creando...")
        return False

# Solo crear/corregir si es necesario
if not verificar_y_preparar_script():
    # AQUI SE CREARIA EL SCRIPT COMPLETO SOLO SI ES NECESARIO
    crear_script_completo()
```

```python
import asyncio
import re
import sys
import os
import argparse
from datetime import datetime
from pathlib import Path
import urllib.parse

try:
    from playwright.async_api import async_playwright
except ImportError:
    print("ERROR: Playwright no instalado. Ejecuta: pip install playwright && python -m playwright install chromium")
    sys.exit(1)

try:
    import openpyxl
    from openpyxl.styles import PatternFill, Font, Alignment, Border, Side
    from openpyxl.utils import get_column_letter
except ImportError:
    print("ERROR: openpyxl no instalado. Ejecuta: pip install openpyxl")
    sys.exit(1)

try:
    import requests
except ImportError:
    print("ERROR: requests no instalado. Ejecuta: pip install requests")
    sys.exit(1)

OUTPUT_DIR = Path().cwd() / "output"
SCREENSHOTS_DIR = OUTPUT_DIR / "screenshots"
OUTPUT_DIR.mkdir(parents=True, exist_ok=True)
SCREENSHOTS_DIR.mkdir(parents=True, exist_ok=True)


def find_excel_file():
    """Busca automáticamente archivos Excel en la carpeta actual"""
    current_dir = Path().cwd()
    excel_files = list(current_dir.glob('*.xlsx')) + list(current_dir.glob('*.xls'))
    
    if not excel_files:
        print(f"ERROR: No se encontraron archivos Excel (.xlsx/.xls) en {current_dir}")
        print("Coloca tu archivo Excel en la carpeta de ejecución y vuelve a intentar.")
        return None
    elif len(excel_files) == 1:
        print(f"Archivo Excel encontrado: {excel_files[0].name}")
        return excel_files[0]
    else:
        print(f"Se encontraron {len(excel_files)} archivos Excel:")
        for i, file in enumerate(excel_files, 1):
            print(f"  {i}. {file.name}")
        print("Usando el primero por defecto...")
        return excel_files[0]


def load_excel_data(excel_file):
    """Carga las URLs desde el archivo Excel (columnas A y B)"""
    try:
        wb = openpyxl.load_workbook(excel_file)
        ws = wb.active
        
        redirections = []
        for row in range(2, ws.max_row + 1):  # Empieza desde fila 2 (asume headers en fila 1)
            url_origen = ws[f'A{row}'].value
            url_destino = ws[f'B{row}'].value
            
            if url_origen and url_destino:
                # Limpiar y normalizar URLs
                url_origen = str(url_origen).strip()
                url_destino = str(url_destino).strip()
                
                # Agregar protocolo si no existe
                if not url_origen.startswith(('http://', 'https://')):
                    url_origen = 'https://' + url_origen
                if not url_destino.startswith(('http://', 'https://')):
                    url_destino = 'https://' + url_destino
                
                redirections.append({
                    'fila': row,
                    'origen': url_origen,
                    'destino': url_destino
                })
        
        print(f"Cargadas {len(redirections)} redirecciones del archivo Excel")
        return redirections
        
    except Exception as e:
        print(f"ERROR leyendo archivo Excel: {e}")
        return []


async def verificar_redireccion_browser(page, url_origen, url_destino):
    """Verifica la redirección usando el navegador y toma screenshot"""
    try:
        print(f"    Verificando con navegador: {url_origen}")
        
        # Navegar a la URL origen
        response = await page.goto(url_origen, wait_until="networkidle", timeout=30000)
        
        # Esperar un poco para asegurar que las redirecciones se completen
        await page.wait_for_timeout(2000)
        
        # Obtener la URL final
        url_final = page.url
        
        # Normalizar URLs para comparación
        url_destino_normalizada = url_destino.rstrip('/')
        url_final_normalizada = url_final.rstrip('/')
        
        # Verificar si coincide
        redireccion_correcta = url_final_normalizada.lower() == url_destino_normalizada.lower()
        
        # Tomar screenshot
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S_%f")[:19]
        safe_origen = re.sub(r'[^a-zA-Z0-9]', '_', url_origen.replace('https://', '').replace('http://', ''))[:40]
        filename = f"{safe_origen}__{timestamp}.png"
        screenshot_path = str(SCREENSHOTS_DIR / filename)
        await page.screenshot(path=screenshot_path, full_page=True)
        
        return {
            'url_final': url_final,
            'redireccion_correcta': redireccion_correcta,
            'screenshot': screenshot_path,
            'status_code': response.status if response else None,
            'error': None
        }
        
    except Exception as e:
        error_msg = str(e)[:200]
        print(f"    ERROR en navegador: {error_msg}")
        return {
            'url_final': None,
            'redireccion_correcta': False,
            'screenshot': None,
            'status_code': None,
            'error': error_msg
        }


def verificar_redireccion_requests(url_origen, url_destino):
    """Verifica la redirección usando requests para obtener códigos de estado"""
    try:
        print(f"    Verificando códigos HTTP: {url_origen}")
        
        session = requests.Session()
        session.max_redirects = 10
        
        # Primera petición para ver la cadena de redirecciones
        response = session.get(url_origen, allow_redirects=False, timeout=30)
        
        redirections_chain = []
        current_url = url_origen
        
        # Seguir la cadena de redirecciones manualmente
        while response.status_code in [301, 302, 303, 307, 308]:
            redirections_chain.append({
                'url': current_url,
                'status_code': response.status_code,
                'location': response.headers.get('Location', '')
            })
            
            # Preparar siguiente URL
            location = response.headers.get('Location', '')
            if location.startswith('http'):
                current_url = location
            else:
                # URL relativa
                current_url = urllib.parse.urljoin(current_url, location)
            
            # Hacer siguiente petición
            response = session.get(current_url, allow_redirects=False, timeout=30)
            
            if len(redirections_chain) > 10:  # Evitar bucles infinitos
                break
        
        # URL final después de todas las redirecciones
        url_final = current_url
        
        # Verificar si la URL final coincide con el destino esperado
        url_destino_normalizada = url_destino.rstrip('/')
        url_final_normalizada = url_final.rstrip('/')
        redireccion_correcta = url_final_normalizada.lower() == url_destino_normalizada.lower()
        
        return {
            'redirections_chain': redirections_chain,
            'url_final': url_final,
            'final_status_code': response.status_code,
            'redireccion_correcta': redireccion_correcta,
            'error': None
        }
        
    except Exception as e:
        error_msg = str(e)[:200]
        print(f"    ERROR en requests: {error_msg}")
        return {
            'redirections_chain': [],
            'url_final': None,
            'final_status_code': None,
            'redireccion_correcta': False,
            'error': error_msg
        }


async def verificar_redirecciones(redirections):
    """Verifica todas las redirecciones"""
    results = []
    
    async with async_playwright() as p:
        browser = await p.chromium.launch(headless=True)
        context = await browser.new_context(
            viewport={'width': 1366, 'height': 768},
            user_agent='Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
        )
        page = await context.new_page()
        
        for i, redir in enumerate(redirections, 1):
            print(f"\n[{i}/{len(redirections)}] Verificando redirección:")
            print(f"  Origen : {redir['origen']}")
            print(f"  Destino: {redir['destino']}")
            
            # Verificar con requests (códigos HTTP)
            requests_result = verificar_redireccion_requests(redir['origen'], redir['destino'])
            
            # Verificar con navegador (captura visual)
            browser_result = await verificar_redireccion_browser(page, redir['origen'], redir['destino'])
            
            # Combinar resultados
            result = {
                'fila': redir['fila'],
                'url_origen': redir['origen'],
                'url_destino_esperada': redir['destino'],
                'url_final_obtenida': browser_result['url_final'] or requests_result['url_final'],
                'redireccion_correcta': browser_result['redireccion_correcta'] or requests_result['redireccion_correcta'],
                'redirections_chain': requests_result['redirections_chain'],
                'final_status_code': requests_result['final_status_code'],
                'screenshot': browser_result['screenshot'],
                'error_browser': browser_result['error'],
                'error_requests': requests_result['error']
            }
            
            results.append(result)
            
            # Log resultado
            if result['redireccion_correcta']:
                print(f"  ✓ REDIRECCIÓN CORRECTA")
            else:
                print(f"  ✗ REDIRECCIÓN INCORRECTA")
                print(f"    Esperada: {redir['destino']}")
                print(f"    Obtenida: {result['url_final_obtenida']}")
        
        await context.close()
        await browser.close()
    
    return results


def generate_excel_report(results, excel_file):
    """Genera el reporte Excel con los resultados"""
    wb = openpyxl.Workbook()
    
    # ── Estilos ──────────────────────────────────────────────────────────────
    header_fill = PatternFill("solid", fgColor="1F4E79")
    header_font = Font(color="FFFFFF", bold=True, size=11)
    correct_fill = PatternFill("solid", fgColor="C6EFCE")   # verde
    correct_font = Font(color="375623", bold=True)
    incorrect_fill = PatternFill("solid", fgColor="FFC7CE") # rojo 
    incorrect_font = Font(color="9C0006", bold=True)
    error_fill = PatternFill("solid", fgColor="FFEB9C")     # amarillo
    title_font = Font(bold=True, size=13, color="1F4E79")
    
    thin = Side(style='thin', color='BFBFBF')
    border = Border(left=thin, right=thin, top=thin, bottom=thin)
    
    center = Alignment(horizontal='center', vertical='center', wrap_text=True)
    left = Alignment(horizontal='left', vertical='center', wrap_text=True)
    
    # ════════════════════════════════════════════════════════════════════════
    # HOJA 1 — RESUMEN
    # ════════════════════════════════════════════════════════════════════════
    ws1 = wb.active
    ws1.title = "Resumen"
    
    # Título
    ws1.merge_cells("A1:G1")
    t = ws1['A1']
    t.value = f"REPORTE DE VERIFICACIÓN DE REDIRECCIONES  |  {datetime.now().strftime('%d/%m/%Y %H:%M:%S')}"
    t.font = title_font
    t.alignment = center
    ws1.row_dimensions[1].height = 30
    
    # Estadísticas generales
    total = len(results)
    correctas = sum(1 for r in results if r['redireccion_correcta'])
    incorrectas = total - correctas
    porcentaje_exito = round(correctas / total * 100) if total > 0 else 0
    
    stats_row = 3
    ws1[f'A{stats_row}'] = "ESTADÍSTICAS GENERALES:"
    ws1[f'A{stats_row}'].font = Font(bold=True, size=12)
    
    ws1[f'A{stats_row+1}'] = f"Total de redirecciones verificadas: {total}"
    ws1[f'A{stats_row+2}'] = f"Redirecciones correctas: {correctas}"
    ws1[f'A{stats_row+3}'] = f"Redirecciones incorrectas: {incorrectas}"
    ws1[f'A{stats_row+4}'] = f"Porcentaje de éxito: {porcentaje_exito}%"
    
    # Encabezados de la tabla
    headers = ["Fila", "URL Origen", "URL Destino Esperada", "URL Final Obtenida", "Estado", "Códigos HTTP", "Screenshot"]
    header_row = stats_row + 6
    
    for col, header in enumerate(headers, 1):
        cell = ws1.cell(row=header_row, column=col, value=header)
        cell.fill = header_fill
        cell.font = header_font
        cell.alignment = center
        cell.border = border
    
    # Datos
    for i, result in enumerate(results, header_row + 1):
        ws1.row_dimensions[i].height = 25
        
        # Fila
        ws1.cell(row=i, column=1, value=result['fila']).border = border
        ws1.cell(row=i, column=1).alignment = center
        
        # URL Origen
        cell = ws1.cell(row=i, column=2, value=result['url_origen'])
        cell.hyperlink = result['url_origen']
        cell.font = Font(color="0563C1", underline="single")
        cell.alignment = left
        cell.border = border
        
        # URL Destino Esperada
        cell = ws1.cell(row=i, column=3, value=result['url_destino_esperada'])
        cell.alignment = left
        cell.border = border
        
        # URL Final Obtenida
        cell = ws1.cell(row=i, column=4, value=result['url_final_obtenida'] or "ERROR")
        if result['url_final_obtenida']:
            cell.hyperlink = result['url_final_obtenida']
            cell.font = Font(color="0563C1", underline="single")
        cell.alignment = left
        cell.border = border
        
        # Estado
        if result['redireccion_correcta']:
            estado = "✓ CORRECTA"
            fill = correct_fill
            font = correct_font
        else:
            estado = "✗ INCORRECTA"
            fill = incorrect_fill
            font = incorrect_font
        
        cell = ws1.cell(row=i, column=5, value=estado)
        cell.fill = fill
        cell.font = font
        cell.alignment = center
        cell.border = border
        
        # Códigos HTTP
        if result['redirections_chain']:
            codes = " → ".join([f"{r['status_code']}" for r in result['redirections_chain']])
            if result['final_status_code']:
                codes += f" → {result['final_status_code']}"
        else:
            codes = str(result['final_status_code']) if result['final_status_code'] else "N/A"
        
        cell = ws1.cell(row=i, column=6, value=codes)
        cell.alignment = center
        cell.border = border
        
        # Screenshot
        cell = ws1.cell(row=i, column=7, value=result['screenshot'] or "N/A")
        cell.alignment = left
        cell.border = border
    
    # Ajustar anchos de columna
    column_widths = [8, 40, 40, 40, 15, 20, 50]
    for i, width in enumerate(column_widths, 1):
        ws1.column_dimensions[get_column_letter(i)].width = width
    
    # ════════════════════════════════════════════════════════════════════════
    # HOJA 2 — REDIRECCIONES INCORRECTAS
    # ════════════════════════════════════════════════════════════════════════
    ws2 = wb.create_sheet("Incorrectas")
    
    # Encabezados
    headers2 = ["Fila", "URL Origen", "URL Destino Esperada", "URL Final Obtenida", "Error Browser", "Error Requests"]
    for col, header in enumerate(headers2, 1):
        cell = ws2.cell(row=1, column=col, value=header)
        cell.fill = header_fill
        cell.font = header_font
        cell.alignment = center
        cell.border = border
    
    # Filtrar solo las incorrectas
    incorrectas_results = [r for r in results if not r['redireccion_correcta']]
    
    for i, result in enumerate(incorrectas_results, 2):
        ws2.row_dimensions[i].height = 20
        
        ws2.cell(row=i, column=1, value=result['fila']).border = border
        
        cell = ws2.cell(row=i, column=2, value=result['url_origen'])
        cell.hyperlink = result['url_origen']
        cell.font = Font(color="0563C1", underline="single")
        cell.border = border
        cell.fill = incorrect_fill
        
        ws2.cell(row=i, column=3, value=result['url_destino_esperada']).border = border
        ws2.cell(row=i, column=4, value=result['url_final_obtenida'] or "ERROR").border = border
        ws2.cell(row=i, column=5, value=result['error_browser'] or "").border = border
        ws2.cell(row=i, column=6, value=result['error_requests'] or "").border = border
    
    # Ajustar anchos
    for i, width in enumerate([8, 40, 40, 40, 30, 30], 1):
        ws2.column_dimensions[get_column_letter(i)].width = width
    
    # Guardar archivo
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    excel_filename = f"reporte_redirecciones_{timestamp}.xlsx"
    report_path = OUTPUT_DIR / excel_filename
    wb.save(str(report_path))
    
    return str(report_path)


def generar_resumen_txt(results, excel_file, report_path):
    """Genera un archivo de texto con resumen ejecutivo"""
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    summary_path = OUTPUT_DIR / f"resumen_redirecciones_{timestamp}.txt"
    
    total = len(results)
    correctas = sum(1 for r in results if r['redireccion_correcta'])
    incorrectas = total - correctas
    porcentaje_exito = round(correctas / total * 100) if total > 0 else 0
    
    with open(summary_path, 'w', encoding='utf-8') as f:
        f.write("=" * 80 + "\n")
        f.write("RESUMEN EJECUTIVO - VERIFICACIÓN DE REDIRECCIONES\n")
        f.write("=" * 80 + "\n\n")
        
        f.write(f"Fecha de análisis: {datetime.now().strftime('%d/%m/%Y %H:%M:%S')}\n")
        f.write(f"Archivo Excel analizado: {excel_file}\n")
        f.write(f"Reporte generado en: {report_path}\n\n")
        
        f.write("ESTADÍSTICAS GENERALES:\n")
        f.write("-" * 30 + "\n")
        f.write(f"Total de redirecciones verificadas: {total}\n")
        f.write(f"Redirecciones correctas: {correctas}\n")
        f.write(f"Redirecciones incorrectas: {incorrectas}\n")
        f.write(f"Porcentaje de éxito: {porcentaje_exito}%\n\n")
        
        if incorrectas > 0:
            f.write("REDIRECCIONES PROBLEMÁTICAS:\n")
            f.write("-" * 35 + "\n")
            for result in results:
                if not result['redireccion_correcta']:
                    f.write(f"Fila {result['fila']}:\n")
                    f.write(f"  Origen: {result['url_origen']}\n")
                    f.write(f"  Esperada: {result['url_destino_esperada']}\n")
                    f.write(f"  Obtenida: {result['url_final_obtenida'] or 'ERROR'}\n")
                    if result['error_browser'] or result['error_requests']:
                        f.write(f"  Errores: {result['error_browser'] or ''} {result['error_requests'] or ''}\n")
                    f.write("\n")
        else:
            f.write("¡EXCELENTE! Todas las redirecciones funcionan correctamente.\n")
        
        f.write("=" * 80 + "\n")
    
    return str(summary_path)


if __name__ == "__main__":
    print("=" * 80)
    print("  VERIFICADOR DE REDIRECCIONES 301/302")
    print("=" * 80)
    print(f"  Directorio de trabajo: {Path().cwd()}")
    print(f"  Output: {OUTPUT_DIR}")
    print("=" * 80)
    
    # Buscar archivo Excel automáticamente
    excel_file = find_excel_file()
    if not excel_file:
        sys.exit(1)
    
    print(f"  Procesando: {excel_file.name}")
    print("=" * 80)
    
    # Cargar datos del Excel
    redirections = load_excel_data(excel_file)
    if not redirections:
        print("ERROR: No se pudieron cargar datos del Excel")
        sys.exit(1)
    
    # Verificar redirecciones
    print(f"\nIniciando verificación de {len(redirections)} redirecciones...")
    results = asyncio.run(verificar_redirecciones(redirections))
    
    # Generar reportes
    print(f"\nGenerando reportes...")
    report_path = generate_excel_report(results, str(excel_file))
    summary_path = generar_resumen_txt(results, str(excel_file), report_path)
    
    # Resumen final
    total = len(results)
    correctas = sum(1 for r in results if r['redireccion_correcta'])
    incorrectas = total - correctas
    porcentaje = round(correctas / total * 100) if total > 0 else 0
    
    print("\n" + "=" * 80)
    print("VERIFICACIÓN COMPLETADA")
    print("=" * 80)
    print(f"Total verificadas: {total}")
    print(f"Correctas: {correctas}")
    print(f"Incorrectas: {incorrectas}")
    print(f"Éxito: {porcentaje}%")
    print(f"\nReportes generados:")
    print(f"  Excel: {report_path}")
    print(f"  Resumen: {summary_path}")
    print(f"  Screenshots: {SCREENSHOTS_DIR}")
    print("=" * 80)
```

---

## Paso 3 — Ejecución inteligente

**El skill ejecuta automáticamente el proceso optimizado:**

```bash
cd "C:\Users\jesus\Desktop\Ilumno\redireccion-301"
python validar_redirecciones.py
```

**Flujo de ejecución automática:**
1. 🔍 **Verificar archivo**: ¿Existe `validar_redirecciones.py`?
2. ⚙️ **Crear/corregir**: Solo si es necesario
3. 📁 **Buscar Excel**: Detectar archivos automáticamente 
4. ▶️ **Ejecutar verificación**: Procesar redirecciones
5. 📊 **Generar reportes**: Excel + TXT + Screenshots

> **EFICIENCIA**: El script se optimiza a sí mismo. Si ya está bien configurado, no se modifica.

> **CORRECCIÓN AUTOMÁTICA**: Si detecta errores en código o configuración, los corrige automáticamente antes de ejecutar.

---

## Paso 4 — Archivos generados

El script generará automáticamente:

1. **Reporte Excel** (`reporte_redirecciones_YYYYMMDD_HHMMSS.xlsx`):
   - **Hoja "Resumen"**: Estadísticas y tabla completa con todas las verificaciones
   - **Hoja "Incorrectas"**: Solo las redirecciones que fallaron

2. **Resumen ejecutivo** (`resumen_redirecciones_YYYYMMDD_HHMMSS.txt`):
   - Estadísticas generales
   - Lista de redirecciones problemáticas
   - Porcentaje de éxito

3. **Capturas de pantalla** (carpeta `output/screenshots/`):
   - Una imagen por cada URL origen verificada
   - Muestra cómo se ve la página final después de la redirección

---

## Funcionalidades principales

## Funcionalidades principales mantenidas

✅ **Verificación HTTP**: Detecta códigos 301, 302, 307, 308 y sigue cadenas de redirección
✅ **Verificación visual**: Navega con navegador real para confirmar comportamiento  
✅ **Capturas automáticas**: Screenshot de cada página para evidencia visual
✅ **Reporte completo**: Excel con estadísticas, detalles y análisis de errores
✅ **Manejo de errores**: Captura y reporta problemas de conectividad o URLs inválidas
✅ **Normalización**: Maneja URLs con/sin protocolo y diferencias en trailing slashes
✅ **Verificación y corrección automática**: Solo crea/modifica el script cuando es necesario
✅ **Optimización de recursos**: Evita recrear archivos innecesariamente
✅ **Autocorrección**: Detecta y repara errores en tiempo real
✅ **Ejecución inteligente**: Flujo adaptativo según estado del sistema

## Beneficios de la optimización

💰 **Ahorro de tokens**: No regenera código innecesariamente  
⚡ **Más rápido**: Ejecución directa si todo está correcto
🔧 **Autocorrectivo**: Detecta y repara problemas automáticamente
🎯 **Inteligente**: Se adapta al estado actual del sistema

> **Los reportes incluyen**: códigos de estado HTTP, cadena completa de redirecciones, URLs finales obtenidas vs esperadas, capturas de pantalla y análisis de errores.

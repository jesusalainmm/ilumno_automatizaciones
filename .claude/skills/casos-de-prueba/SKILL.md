---
name: casos-de-prueba
description: >
  Usa esta skill siempre que el usuario necesite generar casos de prueba QA a partir de una
  historia de usuario, ticket, o solicitud de desarrollo. El skill lee una plantilla Excel
  de referencia (Plantilla-casos-de-prueba.xlsx), un archivo TXT con reglas QA (reglas-qa.txt),
  visita los enlaces de la solicitud para obtener contexto, y produce un archivo Excel (.xlsx)
  NUEVO con casos de prueba detallados paso a paso. El usuario es un especialista QA que
  revisa la calidad después de que desarrollo entrega. Actívala ante solicitudes como
  "generar casos de prueba", "crear test cases", "diseñar casos de prueba QA",
  "caso de prueba para landing", "caso de prueba para formulario", "QA para esta tarea",
  "diseña los CP", o cualquier solicitud que incluya formato COMO/QUIERO/PARA, historias
  de usuario, criterios de aceptación, acceptance criteria, paso a paso QA, o plantilla
  de pruebas. También si adjunta un ticket con requerimientos funcionales.
compatibility:
  required_tools:
    - run_in_terminal (para ejecutar scripts Python)
    - create_file
    - read_file
    - replace_string_in_file
    - fetch_webpage (OBLIGATORIO para visitar enlaces de la solicitud)
    - semantic_search
  python_packages:
    - openpyxl (pip install openpyxl)
    - python-docx (pip install python-docx) — para leer archivos .docx de la carpeta anexos
    - PyPDF2 (pip install PyPDF2) — para leer archivos .pdf de la carpeta anexos
---

# Generador de Casos de Prueba QA

Skill para generar un archivo Excel (.xlsx) NUEVO con casos de prueba QA detallados, paso a paso,
a partir de tickets, historias de usuario o solicitudes de desarrollo.

## Contexto del usuario

El usuario es un **especialista QA** que recibe solicitudes de desarrollo (landings, páginas web,
formularios, módulos). El equipo de desarrollo construye primero, y luego el usuario debe
**revisar la calidad** de lo desarrollado. Este skill genera la plantilla Excel con todos los
casos de prueba que el QA debe ejecutar para validar esa entrega.

## Archivos de referencia

- **Plantilla Excel:** `casos-de-prueba/Plantilla-casos-de-prueba.xlsx` — estructura y formato a replicar
- **Reglas QA:** `casos-de-prueba/reglas-qa.txt` — reglas de negocio, convenciones y plantillas de pasos
- **Ejemplo de solicitud:** `casos-de-prueba/instrucciones-cp.txt` — ejemplo real de cómo llegan los tickets
- **Carpeta anexos:** `casos-de-prueba/anexos/` — archivos adicionales de contexto (imágenes, Word, Excel, PDF, etc.)

---

## Archivos de entrada

Antes de generar casos, SIEMPRE leer estos archivos:

### 1. Plantilla Excel de referencia
- **Ruta:** `casos-de-prueba/Plantilla-casos-de-prueba.xlsx` (relativa a la raíz del workspace)
- **Hoja "CP"** — Estructura de los casos de prueba con estas columnas:

| Columna | Campo | Descripción |
|---------|-------|-------------|
| A | Category | Categoría general del módulo (ej: HOMOLOGACIÓN, LANDING, CRM, etc.) |
| B | Subcategory | Subdivisión (ej: LANDING, BTL, ATL, FORMULARIO, DATA SEND, etc.) |
| C | CaseType | POSITIVO o NEGATIVO |
| D | Caso de Prueba | Título descriptivo del caso de prueba |
| E | Description | Explicación detallada de qué se valida y por qué |
| F | Steps | Pasos numerados para ejecutar la prueba |
| G | Result | Qué debe ocurrir si la funcionalidad es correcta |
| H | Estado de Caso Automatización | MANUAL, AUTOMATIZADO, EN ANALISIS, EN PROCESO, EN APROBACIÓN, CLIENTE INTERNO |
| I | Ruta Test Automatizado | Ruta del script si existe automatización |
| J | Nombre de Test | Nombre del test automatizado si aplica |

- **Hoja "PARAMETROS DE PRUEBAS"** — Valores válidos para dropdowns:
  - **Tipo de Módulo:** HUBSPOT, APP_BUILDER, DRUPAL, ILUMNO
  - **Severidad:** Crítica, Alta, Media, Baja
  - **Prioridad:** P1, P2, P3, P4
  - **Tipo de prueba:** Funcional, UI/UX, SEO, Performance, Seguridad, Accesibilidad, Analítica/GTM, Contenido
  - **Tipo Caso:** POSITIVO, NEGATIVO
  - **Etapa:** Smoke, Funcional, Performance, Backend, Frontend
  - **Subcategoria:** CALCULADORA, BTL, ATL, ESPECIALES, SENA, CUPONERA, PORTAL DEL COLABORADOR, BLOG, NEWS LETTER, LANDING, DATA SEND, UTMS, MÓDULOS, DATA LEAD, DATA POTENCIAL, MICROLANDING, PROGRAMA, BASIC PAGE, PÁGINA, HOME, MODAL, CONCEPTO, BLOGS, SIN CATEGORÍA
  - **Category:** CRM, EDICIÓN, FORMULARIO, OFERTA ACADEMICA, ENCOLAMIENTO, ASPIRANTE, ASESOR, FDI ORGANICO, FDI ASESOR, FDI ASPIRANTE, FDI BACK OFFICE, FDI-WHATSAPP, E-BOOKS COINS, MS-AUTH, E-BOOK, MICROLANDING, RECONSTRUCCIÓN, CONSTRUCCIÓN, REFACTORIZACIÓN, INTERNAS, COTIZADOR, PRUEBA, SCHEMA/DATOS ESTRUCTURADOS SEO, HOMOLOGACIÓN

### 2. Archivo TXT complementario (reglas-qa.txt)
- **Ruta:** `casos-de-prueba/reglas-qa.txt` (relativa a la raíz del workspace)
- Contiene instrucciones adicionales, convenciones de nomenclatura, criterios de calidad y reglas de negocio recurrentes que complementan la plantilla.
- **SIEMPRE leer este archivo** antes de generar los casos de prueba para incorporar las reglas más actualizadas.

### 3. Archivo TXT de ejemplo de solicitud (instrucciones-cp.txt)
- **Ruta:** `casos-de-prueba/instrucciones-cp.txt` (relativa a la raíz del workspace)
- Contiene un ejemplo real de solicitud con formato COMO/QUIERO/PARA, contexto, criterios de aceptación y enlaces de referencia.
- Usar como referencia de cómo interpretar solicitudes del usuario.

### 4. Carpeta de anexos (casos-de-prueba/anexos/)
- **Ruta:** `casos-de-prueba/anexos/` (relativa a la raíz del workspace)
- Puede contener archivos adicionales que el usuario coloca para dar más contexto sobre la tarea.
- **Tipos de archivo soportados y cómo leerlos:**

| Tipo | Extensiones | Cómo leer |
|------|------------|----------|
| Imágenes | .png, .jpg, .jpeg, .gif, .webp | Describir lo que se observa (mockups, capturas de diseño, referencias visuales). Usar para generar casos de validación visual |
| Excel | .xlsx, .xls | Leer con `openpyxl` vía script Python para extraer textos, datos de programas, tablas de contenido |
| Word | .docx | Leer con `python-docx` vía script Python para extraer textos, requisitos, especificaciones |
| PDF | .pdf | Leer con `PyPDF2` o `pdfplumber` vía script Python para extraer texto |
| TXT/CSV | .txt, .csv | Leer directamente con `read_file` |
| Fuentes | .ttf, .otf | Intentar leer como texto; si no es posible, anotar como referencia |
| Otros | cualquier otro | Intentar leer como texto; si no es posible, anotar como referencia |

- **SIEMPRE revisar esta carpeta** antes de generar casos de prueba.
- Si la carpeta está vacía, continuar sin anexos.
- Si tiene archivos, leerlos TODOS para extraer contexto adicional (textos, diseños, datos de programas, requisitos, etc.).

---

## Flujo de trabajo (SEGUIR EN ESTE ORDEN)

### Paso 1 — Leer archivos de referencia
- Leer `casos-de-prueba/Plantilla-casos-de-prueba.xlsx` para recordar estructura y columnas
- Leer `casos-de-prueba/reglas-qa.txt` para reglas de negocio y convenciones
- Listar `casos-de-prueba/anexos/` y leer TODOS los archivos que contenga (ver tabla de tipos en sección 4)
  - Para Excel/Word/PDF: generar un script Python pequeño que extraiga el texto y lo imprima
  - Para imágenes: analizar visualmente si es posible (mockups, capturas, diseños)
  - Para TXT/CSV: leer directamente con read_file
  - Usar la información extraída para enriquecer los casos de prueba con detalles específicos

### Paso 2 — Leer la solicitud completa del usuario
- El usuario pega un ticket, historia de usuario, o describe qué necesita
- Leer TODO el texto proporcionado: historia de usuario, contexto, criterios de aceptación, notas adicionales
- Identificar todos los enlaces mencionados (Figma, OneDrive, Google Docs, Monday.com, URLs de landing)

### Paso 3 — Visitar los enlaces para obtener contexto (OBLIGATORIO)
- Si la solicitud incluye URLs, usar `fetch_webpage` para consultarlas
- Esto permite entender QUÉ se está desarrollando, el diseño, los textos, las piezas gráficas
- **Si un enlace requiere autenticación** (login, error 403/401):
  1. Leer las credenciales desde `casos-de-prueba/credenciales.json` (contiene `username` y `password`)
  2. Usar Playwright para navegar a la URL, ingresar las credenciales y extraer el contenido
  3. Si `credenciales.json` no existe o está vacío, anotar "Consultar [tipo]: [URL] — requiere login, verificar manualmente" en los pasos del caso
- Los enlaces pueden ser de: Figma, OneDrive, Google Docs, Monday.com, la landing misma, HubSpot
- **IMPORTANTE:** No incluir las credenciales en el Excel generado ni en mensajes al usuario.

### Paso 4 — Hacer preguntas mínimas SOLO si falta info crítica
Preguntar ÚNICAMENTE lo que no se pueda inferir de la solicitud ni de los enlaces:

| Parámetro | Cuándo preguntar | Default si no responde |
|-----------|-----------------|------------------------|
| Tipo de landing | Si no queda claro si es HubSpot, Drupal, Ilumno u otra plataforma | Inferir del contexto |
| URLs de la página a probar | Si no se proporcionaron ni se pueden inferir | "PENDIENTE URL" |

| ¿Es landing nueva o actualización? | Si no queda claro | Inferir del contexto |

**NO preguntar** lo que ya esté en el ticket (programa, Figma, textos, criterios). Extraerlo directamente.

### Paso 5 — Generar el archivo Excel con casos de prueba
- Crear un script Python con openpyxl
- Generar el archivo .xlsx con la misma estructura de la plantilla
- Cada caso debe tener paso a paso MUY DETALLADO (que cualquier QA pueda ejecutarlo sin contexto previo)
- Guardar en `casos-de-prueba/generados/CP_{codigo}_{nombre}_{fecha}.xlsx`

### Paso 6 — Presentar resumen
- Informar al usuario: total de casos, positivos/negativos, cobertura por criterio de aceptación

---

## Cómo interpretar la solicitud del usuario

### Formato de historia de usuario (COMO / QUIERO / PARA)
El usuario puede proporcionar solicitudes en formato:
```
COMO: [rol/equipo]
QUIERO: [funcionalidad requerida]
PARA: [objetivo de negocio]
```

Extraer de aquí:
- **Rol solicitante** → ayuda a definir la Categoría
- **Funcionalidad** → define Subcategoría y tipo de casos
- **Objetivo** → complementa la Descripción

### Sección CONTEXTO
Describe el alcance técnico. Extraer:
- Plataforma (HubSpot, Drupal, etc.) → Tipo de Módulo
- Integraciones (CRM, App Builder, etc.) → Casos adicionales de integración
- Tipo de activo (landing, formulario, módulo, etc.) → Subcategoría

### Sección CRITERIOS DE ACEPTACIÓN
Cada criterio se convierte en **al menos un caso de prueba positivo** y cuando aplique un **caso negativo**. Ejemplo:

| Criterio | Caso Positivo | Caso Negativo |
|----------|--------------|---------------|
| "Que la landing tenga manejo de marca correcto" | Validar branding correcto | Validar que no haya elementos de marca incorrectos |
| "Que sea responsive con performance alta" | Verificar responsive en mobile/tablet | Verificar que no tenga errores en resoluciones atípicas |
| "Que el mapeo de campos sea correcto en HubSpot" | Enviar formulario y validar datos en HubSpot | Enviar datos erróneos y validar alertas |
| "Que el lead llegue a CRM" | Enviar lead y validar en CRM | Enviar formulario incompleto y validar que no cree registro |

### Sección NOTAS ADICIONALES
Agregar como observaciones en la Descripción de los casos afectados.

### URLs de referencia — SIEMPRE visitarlas
Cuando la solicitud incluya enlaces, el agente DEBE intentar acceder a cada uno:
- **Figma**: usar `fetch_webpage` → extraer secciones del diseño, estructura, colores, tipografía
- **OneDrive/Google Docs**: usar `fetch_webpage` → extraer textos, copy, datos del programa
- **Monday.com / Jira / Azure DevOps**: usar `fetch_webpage` → extraer detalles del ticket
- **URLs de landing/página**: usar `fetch_webpage` → ver si ya existe algo publicado, analizar formulario
- **Google Drive (Excel/Sheets)**: usar `fetch_webpage` → extraer datos de textos o contenido

Esto es CRÍTICO porque los enlaces contienen la información detallada de lo que desarrollo está
construyendo. Sin revisar los enlaces, los casos de prueba serán genéricos e incompletos.

### Acceso con credenciales a URLs protegidas
Cuando un enlace devuelve error 403/401 o muestra una página de login:
1. **Leer credenciales** desde `casos-de-prueba/credenciales.json`:
   ```python
   import json
   with open('casos-de-prueba/credenciales.json', 'r') as f:
       creds = json.load(f)
   usuario = creds['username']
   contrasena = creds['password']
   ```
2. **Acceder usando Playwright** con las credenciales obtenidas:
   ```python
   from playwright.sync_api import sync_playwright
   with sync_playwright() as p:
       browser = p.chromium.launch(headless=True)
       page = browser.new_page()
       page.goto(url)
       page.fill('input[type="email"], input[name="email"], #email', usuario)
       page.fill('input[type="password"], input[name="password"], #password', contrasena)
       page.click('button[type="submit"], input[type="submit"]')
       page.wait_for_load_state('networkidle')
       contenido = page.content()
       browser.close()
   ```
3. **Si `credenciales.json` no existe**, registrar en el paso a paso del caso:
   "Consultar [tipo de recurso]: [URL] — recurso protegido, verificar manualmente con acceso autorizado"
4. **SEGURIDAD:** NUNCA incluir las credenciales en el Excel generado, mensajes al usuario ni en memoria.

Si un enlace no es accesible por otra razón (URL rota, timeout), registrar en el paso a paso:
"Consultar [tipo de recurso]: [URL] — recurso no accesible, verificar manualmente"

---

## Catálogo de casos de prueba por tipo de solicitud

### Landing Pages (HubSpot/Drupal)
Generar estos casos automáticamente cuando la solicitud sea sobre landing pages:

| # | Tipo | Caso de Prueba | Descripción |
|---|------|---------------|-------------|
| 1 | POSITIVO | Validar creación landing {plataforma} {programa} | Se requiere crear landing para {programa}. Debe estar en {plataforma} |
| 2 | POSITIVO | Validar estructura/diseño vs referencia (Figma/mockup) | La landing debe coincidir con el diseño aprobado en Figma o el mockup de referencia |
| 3 | POSITIVO | Validar textos/copy correctos | Los textos deben coincidir con el documento de copy entregado |
| 4 | POSITIVO | Validar manejo de marca (logos, colores, tipografía) | La landing debe cumplir con los lineamientos de marca institucional |
| 5 | NEGATIVO | Revisar obligatoriedad y validación alertas campos del formulario | Validar alertas cuando no se diligencian campos obligatorios o se ingresan datos incorrectos |
| 6 | POSITIVO | Enviar formulario con datos correctos | Diligenciar todos los campos correctamente y enviar. Validar thank you page |
| 7 | POSITIVO | Verificar datos del lead en CRM/HubSpot | Tras envío del formulario, validar que el registro llegue correcto a CRM con negocio y detalle |
| 8 | POSITIVO | Verificar datos en App Builder (si aplica) | Validar que la data del formulario llegue correcta a App Builder |
| 9 | POSITIVO | Verificar evento DATASEND en dataLayer | Tras envío, verificar en consola el evento DATASEND con campos correctos (ciudad, email, programa, UTMs, etc.) |
| 10 | POSITIVO | Visualizar landing correctamente en Desktop | Verificar diseño y comportamiento en diferentes navegadores y resoluciones |
| 11 | POSITIVO | Visualizar landing correctamente en Mobile/Tablet (responsive) | Verificar responsive en Android/iOS con diferentes navegadores |
| 12 | POSITIVO | Validar performance (PageSpeed/Lighthouse) | La landing debe tener calificación alta en performance según Lighthouse |
| 14 | POSITIVO | Validar SEO básico (title, meta description, OG tags) | Verificar que tenga title, meta description y Open Graph tags correctos |
| 15 | POSITIVO | Validar UTMs en formulario | Verificar que las UTMs se capturen correctamente en el formulario y lleguen a CRM |

### Formularios (genérico)
Cuando la solicitud sea sobre formularios (sin landing completa):
- Casos 5-9 y 13 del catálogo de landings
- Agregar: validación de campos específicos del formulario según el contexto. Si aplican fuentes debe de agregar casos de validación de fuentes (que se apliquen correctamente, que no se mezclen, que sean legibles)

### Módulos/Componentes (Drupal)
Cuando la solicitud sea sobre módulos o componentes:
- Caso de creación/visualización del módulo
- Caso de edición de contenido
- Caso responsive
- Caso de integración con otros módulos si aplica

### Integraciones CRM
Cuando se mencione CRM, HubSpot, o App Builder:
- Validar mapeo de campos
- Validar creación de registro/negocio
- Validar reglas de atribución si se mencionan
- Validar deduplicación si aplica

---

## Generación del archivo Excel

### Script Python para generar el .xlsx

Usar `openpyxl` para crear el Excel con la misma estructura de la plantilla. El agente debe generar un script Python temporal y ejecutarlo.

**Estructura del script a generar:**

```python
import openpyxl
from openpyxl.styles import Font, PatternFill, Alignment, Border, Side
from openpyxl.utils import get_column_letter
from datetime import datetime

# --- DATOS DE CASOS DE PRUEBA (adaptar según solicitud) ---
casos = [
    # (Category, Subcategory, CaseType, CasoPrueba, Description, Steps, Result, EstadoAutomatizacion)
    ("CATEGORIA", "LANDING", "POSITIVO", "Título del caso", "Descripción...", "1. Paso 1\n2. Paso 2", "Resultado OK", "MANUAL"),
    # ... más casos
]

# --- CREAR WORKBOOK ---
wb = openpyxl.Workbook()
ws = wb.active
ws.title = "CP"

# Headers
headers = ["Category", "Subcategory", "CaseType", "Caso de Prueba",
           "Description", "Steps", "Result",
           "Estado de Caso Automatización", "Ruta Test Automatizado", "Nombre de Test"]

header_fill = PatternFill(start_color="4472C4", end_color="4472C4", fill_type="solid")
header_font = Font(bold=True, color="FFFFFF", size=11)
thin_border = Border(
    left=Side(style='thin'), right=Side(style='thin'),
    top=Side(style='thin'), bottom=Side(style='thin')
)

for col, header in enumerate(headers, 1):
    cell = ws.cell(row=1, column=col, value=header)
    cell.fill = header_fill
    cell.font = header_font
    cell.alignment = Alignment(horizontal='center', vertical='center', wrap_text=True)
    cell.border = thin_border

# Data rows
positivo_fill = PatternFill(start_color="E2EFDA", end_color="E2EFDA", fill_type="solid")
negativo_fill = PatternFill(start_color="FCE4EC", end_color="FCE4EC", fill_type="solid")

for row_idx, caso in enumerate(casos, 2):
    for col_idx, value in enumerate(caso, 1):
        cell = ws.cell(row=row_idx, column=col_idx, value=value)
        cell.alignment = Alignment(vertical='top', wrap_text=True)
        cell.border = thin_border
        if caso[2] == "NEGATIVO":
            cell.fill = negativo_fill
        else:
            cell.fill = positivo_fill

# Anchos de columna
col_widths = [18, 16, 14, 40, 50, 60, 35, 22, 25, 25]
for i, width in enumerate(col_widths, 1):
    ws.column_dimensions[get_column_letter(i)].width = width

# --- HOJA PARAMETROS ---
ws2 = wb.create_sheet("PARAMETROS DE PRUEBAS")
# (Copiar parámetros de la plantilla original)

# Guardar
output_path = r"casos-de-prueba/generados/CP_{nombre_archivo}.xlsx"
wb.save(output_path)
print(f"Archivo generado: {output_path}")
```

### Reglas para nombrar el archivo de salida
- Formato: `CP_{CODIGO}_{NombrePrograma}_{Fecha}.xlsx`
- Ejemplo: `CP_ANDI4229_Maestria_Gestion_Talento_Humano_2026-03-17.xlsx`
- Si no hay código, usar: `CP_{NombreProyecto}_{Fecha}.xlsx`

---

## Reglas de calidad para los casos generados

### Casos POSITIVOS (obligatorios)
- Cada criterio de aceptación debe tener al menos 1 caso positivo
- Los pasos deben ser **numerados, claros y reproducibles**
- Incluir URLs específicas en los pasos (o "PENDIENTE URL" si no se conoce)
- El resultado esperado debe ser **verificable y concreto**

### Casos NEGATIVOS (complementarios)
- Por cada caso positivo funcional, considerar un caso negativo
- Validar: campos vacíos, datos erróneos, formatos incorrectos, caracteres especiales
- No generar negativos para casos puramente visuales (diseño, responsive)

### Paso a Paso — Convenciones
- Empezar con el verbo: "Ingresar a...", "Validar que...", "Verificar que...", "Clic en...", "Diligenciar..."
- Incluir la URL o indicar "PENDIENTE URL" para URLs no entregadas
- Para validaciones CRM: incluir URL de HubSpot y qué campos verificar
- Para dataLayer: indicar qué abrir en consola y qué evento/campos esperar

### Cobertura mínima recomendada por tipo
| Tipo de solicitud | Mín. casos positivos | Mín. casos negativos |
|-------------------|---------------------|---------------------|
| Landing page | 8-12 | 2-3 |
| Formulario | 5-8 | 2-4 |
| Módulo/Componente | 4-6 | 1-2 |
| Integración CRM | 3-5 | 1-2 |

---

## Checklist de calidad antes de entregar

Antes de presentar el archivo al usuario, verificar:

- [ ] Todos los criterios de aceptación del ticket tienen al menos un caso de prueba
- [ ] Los pasos son claros, numerados y reproducibles por un QA sin contexto previo
- [ ] Las URLs de referencia están incluidas en los pasos (o marcadas como PENDIENTE)
- [ ] Hay casos positivos y negativos cuando aplica
- [ ] Los valores de Category, Subcategory y CaseType coinciden con los parámetros válidos de la plantilla
- [ ] El archivo Excel abre correctamente y tiene formato visual profesional
- [ ] Se incluye un resumen al usuario: total de casos, distribución positivos/negativos, cobertura por criterio

---

## Mensajes de error comunes y soluciones

| Error | Causa | Solución |
|-------|-------|----------|
| `ModuleNotFoundError: No module named 'openpyxl'` | openpyxl no instalado | `pip install openpyxl` |
| URL de Figma no accesible | Requiere autenticación | Solicitar al usuario capturas o descripción del diseño |
| URL de OneDrive no accesible | Requiere autenticación | Solicitar al usuario el contenido del documento |
| Categoría no reconocida | Tipo de solicitud nueva | Usar "SIN CATEGORÍA" y preguntar al usuario |

---

## Ejemplo completo de uso

### Input del usuario:
```
COMO: equipo de marketing
QUIERO: una landing donde esté la información de nuestro nuevo programa
PARA: usarla como activo digital en campañas paid y lograr conversión de clientes potenciales

CONTEXTO:
Se requiere landing para el nuevo programa, capturar nuevos leads.
Plantilla HubSpot. Textos: ANDI4229 Maestría en Gestión del Talento Humano.

CRITERIOS DE ACEPTACIÓN:
1. Manejo de marca correcto
2. Responsive con performance alta
3. Mapeo de campos correcto en HubSpot
4. Lead llegue a CRM correctamente
```

### Output esperado:
Excel con ~4-15 casos de prueba cubriendo:
- Creación y estructura de la landing (vs Figma/mockup)
- Textos y copy correctos (vs documento de textos)
- Branding/marca
- Responsive (desktop + mobile/tablet)
- Performance (Lighthouse)
- Formulario: campos obligatorios, validaciones, envío exitoso
- Integración HubSpot: mapeo de campos, registro de contacto
- Integración CRM: lead correcto con negocio/detalle
- DataSend: evento en dataLayer
- SEO básico y UTMs

---

## Output final

Al finalizar, el usuario debe recibir:

1. **`CP_{codigo}_{nombre}_{fecha}.xlsx`** — archivo Excel con casos de prueba completos y formateados
2. **Resumen en chat** — total de casos, distribución por tipo (positivo/negativo), cobertura por criterio de aceptación

El archivo se guarda en la carpeta `casos-de-prueba/generados/` del workspace.

## Entrevista inicial (hacer ANTES de empezar)

Si el usuario no especificó alguno de estos parámetros, pregúntale:

| Parámetro | Pregunta sugerida | Default si no responde |
|-----------|-------------------|------------------------|
| URL de la landing/página | "¿Cuál es la URL de la landing o página a probar?" | "PENDIENTE URL" |
| Tipo de módulo | "¿Qué plataforma se usa? (Hubspot, Drupal, App Builder)" | Inferir del contexto |
| Nombre del programa/producto | "¿Cuál es el nombre exacto del programa o producto?" | Extraer del ticket |
| Responsable QA | "¿Quién es el responsable QA?" | "Equipo QA" |
| ¿Hay Figma o diseño de referencia? | "¿Tienes enlace a Figma o mockups?" | Sin diseño |
| ¿Hay documento de textos? | "¿Tienes enlace al documento de textos/copy?" | Sin documento |
| ¿Se requiere integración CRM? | "¿El formulario debe enviar datos a CRM/HubSpot?" | Inferir del contexto |

---

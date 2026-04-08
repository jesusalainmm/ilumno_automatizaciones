---
name: search-content
description: >
  Usa esta skill siempre que el usuario necesite validar frases o partes de contenido.
  Actívala ante solicitudes como: "buscar frase en URL", "verificar si el texto está en la página",
  "validar contenido en landing", "buscar palabras en sitio web", "comprobar si el texto aparece",
  "evidenciar frase en página", "captura de donde está el texto", "buscar contenido en URLs".
compatibility:
  required_tools:
    - Bash (para ejecutar el script Python)
    - Write (para crear el script temporal)
  python_packages:
    - playwright (pip install playwright && python -m playwright install chromium)
    - openpyxl (pip install openpyxl)
---

# Skill: Search Content — Validador de Frases en URLs

## Objetivo

Buscar una o varias frases dentro de una o varias URLs, evidenciar visualmente dónde aparece cada frase mediante capturas de pantalla con resaltado, y generar un reporte Excel con los resultados discriminados por URL y frase.

> **La búsqueda es siempre case-insensitive**: no distingue entre mayúsculas y minúsculas. `"Contáctanos"`, `"contáctanos"` y `"CONTÁCTANOS"` se tratan como la misma frase.

---

## Paso 1 — Recopilar parámetros

Solicitar al usuario los dos parámetros requeridos si no los proporcionó en el mensaje:

**Parámetro 1 — URLs:**
> "¿Cuál(es) es(son) la(s) URL(s) donde quieres buscar el contenido? Si son varias, sepáralas por coma."

**Parámetro 2 — Frases:**
> "¿Qué frase o frases deseas buscar? Si son varias, sepáralas por coma."

Ejemplos válidos:
- `--urls "https://ejemplo.com, https://otro.com"`
- `--phrases "Contáctanos, Llámanos ahora, +57 300 000 0000"`

**Ruta de salida fija:**
```
C:\Users\jesus\Desktop\Ilumno\search-content\output\
C:\Users\jesus\Desktop\Ilumno\search-content\output\screenshots\
```

---

## Paso 2 — Crear script Python

Crear el archivo `C:\Users\jesus\Desktop\Ilumno\search-content\search_content.py` con el siguiente contenido:

```python
import asyncio
import re
import sys
import os
import argparse
from datetime import datetime
from pathlib import Path

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

OUTPUT_DIR = Path(r"C:\Users\jesus\Desktop\Ilumno\search-content\output")
SCREENSHOTS_DIR = OUTPUT_DIR / "screenshots"
OUTPUT_DIR.mkdir(parents=True, exist_ok=True)
SCREENSHOTS_DIR.mkdir(parents=True, exist_ok=True)

HIGHLIGHT_JS = """
(phrase) => {
    const phraseLC = phrase.toLowerCase();
    let found = false;
    let count = 0;
    let firstHighlight = null;

    // Inject highlight style
    if (!document.getElementById('__sc_style__')) {
        const style = document.createElement('style');
        style.id = '__sc_style__';
        style.textContent = `
            .__sc_highlight__ {
                background-color: #FFD700 !important;
                color: #000000 !important;
                border: 3px solid #FF0000 !important;
                border-radius: 3px !important;
                padding: 1px 3px !important;
                box-shadow: 0 0 12px rgba(255,0,0,0.9) !important;
                outline: 2px solid #FF4444 !important;
                position: relative !important;
                z-index: 9999 !important;
            }
        `;
        document.head.appendChild(style);
    }

    // Escape regex special chars
    const escaped = phrase.replace(/[.*+?^${}()|[\\]\\\\]/g, '\\\\$&');
    const regex = new RegExp(escaped, 'gi');

    // Walk all text nodes in body
    const walker = document.createTreeWalker(
        document.body,
        NodeFilter.SHOW_TEXT,
        {
            acceptNode: (node) => {
                const tag = node.parentElement && node.parentElement.tagName;
                if (['SCRIPT', 'STYLE', 'NOSCRIPT', 'TEXTAREA'].includes(tag)) {
                    return NodeFilter.FILTER_REJECT;
                }
                return NodeFilter.FILTER_ACCEPT;
            }
        },
        false
    );

    const nodesToProcess = [];
    let node;
    while (node = walker.nextNode()) {
        if (node.textContent.toLowerCase().includes(phraseLC)) {
            nodesToProcess.push(node);
        }
    }

    for (const textNode of nodesToProcess) {
        const text = textNode.textContent;
        const matches = text.match(regex);
        if (!matches) continue;

        const parts = text.split(regex);
        const fragment = document.createDocumentFragment();

        parts.forEach((part, i) => {
            if (part) fragment.appendChild(document.createTextNode(part));
            if (i < matches.length) {
                const span = document.createElement('span');
                span.className = '__sc_highlight__';
                span.textContent = matches[i];
                if (!firstHighlight) firstHighlight = span;
                fragment.appendChild(span);
                count++;
            }
        });

        textNode.parentNode.replaceChild(fragment, textNode);
        found = true;
    }

    if (firstHighlight) {
        firstHighlight.scrollIntoView({ behavior: 'instant', block: 'center' });
    }

    return { found, count };
}
"""


async def search_in_url(page, url, phrase):
    """Navega a la URL, busca la frase y toma screenshot si la encuentra."""
    try:
        print(f"    Navegando a: {url}")
        await page.goto(url, wait_until="networkidle", timeout=45000)
        # Extra wait for dynamic content
        await page.wait_for_timeout(2000)

        print(f"    Buscando frase: '{phrase}'")
        result = await page.evaluate(HIGHLIGHT_JS, phrase)

        found = result.get('found', False)
        count = result.get('count', 0)
        screenshot_path = None

        if found:
            timestamp = datetime.now().strftime("%Y%m%d_%H%M%S_%f")[:19]
            safe_url = re.sub(r'[^a-zA-Z0-9]', '_', url.replace('https://', '').replace('http://', ''))[:40]
            safe_phrase = re.sub(r'[^a-zA-Z0-9]', '_', phrase)[:30]
            filename = f"{safe_url}__{safe_phrase}__{timestamp}.png"
            screenshot_path = str(SCREENSHOTS_DIR / filename)
            await page.screenshot(path=screenshot_path, full_page=False)
            print(f"    ENCONTRADA — {count} ocurrencia(s) — Screenshot: {filename}")
        else:
            print(f"    NO ENCONTRADA")

        return {
            'url': url,
            'phrase': phrase,
            'found': found,
            'occurrences': count,
            'screenshot': screenshot_path,
            'error': None
        }

    except Exception as e:
        error_msg = str(e)[:200]
        print(f"    ERROR: {error_msg}")
        return {
            'url': url,
            'phrase': phrase,
            'found': False,
            'occurrences': 0,
            'screenshot': None,
            'error': error_msg
        }


async def run_search(urls, phrases):
    results = []

    async with async_playwright() as p:
        browser = await p.chromium.launch(headless=True)

        for url in urls:
            print(f"\n[URL] {url}")
            context = await browser.new_context(
                viewport={'width': 1366, 'height': 768},
                user_agent='Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36'
            )
            page = await context.new_page()

            for phrase in phrases:
                result = await search_in_url(page, url, phrase)
                results.append(result)

            await context.close()

        await browser.close()

    return results


def generate_excel_report(results, urls, phrases):
    wb = openpyxl.Workbook()

    # ── Estilos ──────────────────────────────────────────────────────────────
    header_fill   = PatternFill("solid", fgColor="1F4E79")
    header_font   = Font(color="FFFFFF", bold=True, size=11)
    found_fill    = PatternFill("solid", fgColor="C6EFCE")   # verde
    found_font    = Font(color="375623", bold=True)
    missing_fill  = PatternFill("solid", fgColor="FFC7CE")   # rojo claro
    missing_font  = Font(color="9C0006", bold=True)
    error_fill    = PatternFill("solid", fgColor="FFEB9C")   # amarillo
    partial_fill  = PatternFill("solid", fgColor="FCE4D6")   # naranja claro
    title_font    = Font(bold=True, size=13, color="1F4E79")

    thin = Side(style='thin', color='BFBFBF')
    border = Border(left=thin, right=thin, top=thin, bottom=thin)

    center = Alignment(horizontal='center', vertical='center', wrap_text=True)
    left   = Alignment(horizontal='left', vertical='center', wrap_text=True)

    # ════════════════════════════════════════════════════════════════════════
    # HOJA 1 — RESUMEN (MATRIZ URL x FRASE)
    # ════════════════════════════════════════════════════════════════════════
    ws1 = wb.active
    ws1.title = "Resumen"

    # Título
    ws1.merge_cells(f"A1:{get_column_letter(len(phrases) + 3)}1")
    t = ws1['A1']
    t.value = f"REPORTE DE BÚSQUEDA DE CONTENIDO  |  {datetime.now().strftime('%d/%m/%Y %H:%M:%S')}"
    t.font = title_font
    t.alignment = center
    ws1.row_dimensions[1].height = 30

    # Encabezados
    ws1.cell(row=3, column=1, value="URL").fill = header_fill
    ws1.cell(row=3, column=1).font = header_font
    ws1.cell(row=3, column=1).alignment = center
    ws1.cell(row=3, column=1).border = border

    for ci, phrase in enumerate(phrases, start=2):
        c = ws1.cell(row=3, column=ci, value=phrase)
        c.fill = header_fill
        c.font = header_font
        c.alignment = center
        c.border = border

    total_col = len(phrases) + 2
    pct_col   = len(phrases) + 3

    for col, lbl in [(total_col, "ENCONTRADAS"), (pct_col, "% ÉXITO")]:
        c = ws1.cell(row=3, column=col, value=lbl)
        c.fill = header_fill
        c.font = header_font
        c.alignment = center
        c.border = border

    # Filas por URL
    url_link_font = Font(color="0563C1", underline="single", size=10)
    for ri, url in enumerate(urls, start=4):
        ws1.row_dimensions[ri].height = 22
        c = ws1.cell(row=ri, column=1, value=url)
        c.hyperlink = url
        c.font = url_link_font
        c.alignment = left
        c.border = border

        n_found = 0
        for ci, phrase in enumerate(phrases, start=2):
            match = next((r for r in results if r['url'] == url and r['phrase'] == phrase), None)

            if match is None:
                val, fill, fnt = "N/A", error_fill, Font(color="7F6000")
            elif match['error']:
                val  = f"ERROR"
                fill = error_fill
                fnt  = Font(color="7F6000")
            elif match['found']:
                val  = f"ENCONTRADA ({match['occurrences']})"
                fill = found_fill
                fnt  = found_font
                n_found += 1
            else:
                val  = "NO ENCONTRADA"
                fill = missing_fill
                fnt  = missing_font

            cell = ws1.cell(row=ri, column=ci, value=val)
            cell.fill = fill
            cell.font = fnt
            cell.alignment = center
            cell.border = border

        pct = round(n_found / len(phrases) * 100) if phrases else 0
        total_cell = ws1.cell(row=ri, column=total_col, value=f"{n_found}/{len(phrases)}")
        total_cell.alignment = center
        total_cell.border = border
        total_cell.fill = found_fill if n_found == len(phrases) else (missing_fill if n_found == 0 else partial_fill)

        pct_cell = ws1.cell(row=ri, column=pct_col, value=f"{pct}%")
        pct_cell.alignment = center
        pct_cell.border = border
        pct_cell.fill = found_fill if pct == 100 else (missing_fill if pct == 0 else partial_fill)

    # Anchos
    ws1.column_dimensions['A'].width = 50
    for ci in range(2, len(phrases) + 4):
        ws1.column_dimensions[get_column_letter(ci)].width = max(20, len(phrases[ci - 2]) + 4) if ci - 2 < len(phrases) else 16

    # ════════════════════════════════════════════════════════════════════════
    # HOJA 2 — DETALLE  (MATRIZ: una fila por URL, columnas dedicadas por frase)
    # Estructura: URL | [Frase1] Estado | [Frase1] Ocurr. | [Frase1] Screenshot | [Frase2] ... |
    # ════════════════════════════════════════════════════════════════════════
    ws2 = wb.create_sheet("Detalle")

    phrase_col_start = 2          # columna B es la primera de frases
    cols_per_phrase  = 3          # Estado · Ocurrencias · Screenshot

    # ── Fila 1: encabezado fijo "URL" y nombre de cada frase (merged 3 cols) ──
    c = ws2.cell(row=1, column=1, value="URL")
    c.fill = header_fill; c.font = header_font
    c.alignment = center; c.border = border
    ws2.merge_cells(start_row=1, start_column=1, end_row=2, end_column=1)

    for pi, phrase in enumerate(phrases):
        col_start = phrase_col_start + pi * cols_per_phrase
        col_end   = col_start + cols_per_phrase - 1
        ws2.merge_cells(start_row=1, start_column=col_start, end_row=1, end_column=col_end)
        c = ws2.cell(row=1, column=col_start, value=phrase)
        c.fill  = PatternFill("solid", fgColor="2E75B6")   # azul medio por frase
        c.font  = Font(color="FFFFFF", bold=True, size=11)
        c.alignment = center
        c.border = border

    ws2.row_dimensions[1].height = 28

    # ── Fila 2: sub-encabezados Estado / Ocurrencias / Screenshot ──
    subheaders = ["Estado", "Ocurrencias", "Screenshot"]
    for pi in range(len(phrases)):
        for si, sh in enumerate(subheaders):
            col = phrase_col_start + pi * cols_per_phrase + si
            c = ws2.cell(row=2, column=col, value=sh)
            c.fill = header_fill; c.font = Font(color="FFFFFF", bold=True, size=10)
            c.alignment = center; c.border = border

    ws2.row_dimensions[2].height = 20
    ws2.freeze_panes = "B3"

    # ── Filas de datos: una por URL ──
    link_font = Font(color="0563C1", underline="single", size=10)
    for ri, url in enumerate(urls, start=3):
        ws2.row_dimensions[ri].height = 20
        c = ws2.cell(row=ri, column=1, value=url)
        c.hyperlink = url; c.font = link_font
        c.alignment = left; c.border = border

        for pi, phrase in enumerate(phrases):
            match = next((r for r in results if r['url'] == url and r['phrase'] == phrase), None)

            if match is None:
                estado, occ, ss_path = "N/A",           0, "—"
                fill, fnt = error_fill, Font(color="7F6000", bold=True)
            elif match['error']:
                estado, occ, ss_path = "ERROR",         0, match.get('error','')[:80]
                fill, fnt = error_fill, Font(color="7F6000", bold=True)
            elif match['found']:
                estado = "ENCONTRADA"
                occ    = match['occurrences']
                ss_path= match.get('screenshot') or "—"
                fill, fnt = found_fill, found_font
            else:
                estado, occ, ss_path = "NO ENCONTRADA", 0, "—"
                fill, fnt = missing_fill, missing_font

            base_col = phrase_col_start + pi * cols_per_phrase

            # Estado
            c = ws2.cell(row=ri, column=base_col, value=estado)
            c.fill = fill; c.font = fnt; c.alignment = center; c.border = border

            # Ocurrencias
            c = ws2.cell(row=ri, column=base_col + 1, value=occ)
            c.fill = fill; c.font = fnt; c.alignment = center; c.border = border

            # Screenshot
            c = ws2.cell(row=ri, column=base_col + 2, value=ss_path)
            c.alignment = left; c.border = border

    # ── Anchos de columna ──
    ws2.column_dimensions['A'].width = 50
    for pi, phrase in enumerate(phrases):
        base_col = phrase_col_start + pi * cols_per_phrase
        ws2.column_dimensions[get_column_letter(base_col)].width     = max(18, len(phrase) + 2)
        ws2.column_dimensions[get_column_letter(base_col + 1)].width = 13
        ws2.column_dimensions[get_column_letter(base_col + 2)].width = 55

    # ════════════════════════════════════════════════════════════════════════
    # HOJA 3 — SOLO FALLIDOS
    # ════════════════════════════════════════════════════════════════════════
    ws3 = wb.create_sheet("No Encontradas")
    for ci, h in enumerate(["#", "URL", "Frase No Encontrada", "Error"], start=1):
        c = ws3.cell(row=1, column=ci, value=h)
        c.fill = header_fill
        c.font = header_font
        c.alignment = center
        c.border = border

    fallidos = [r for r in results if not r['found']]
    for ri, res in enumerate(fallidos, start=2):
        ws3.row_dimensions[ri].height = 18
        ws3.cell(row=ri, column=1, value=ri - 1).border = border
        ws3.cell(row=ri, column=1).alignment = center

        c = ws3.cell(row=ri, column=2, value=res['url'])
        c.hyperlink = res['url']
        c.font = Font(color="0563C1", underline="single", size=10)
        c.alignment = left
        c.border = border
        c.fill = missing_fill

        c = ws3.cell(row=ri, column=3, value=res['phrase'])
        c.alignment = left
        c.border = border
        c.fill = missing_fill

        c = ws3.cell(row=ri, column=4, value=res.get('error') or "")
        c.alignment = left
        c.border = border

    ws3.column_dimensions['A'].width = 5
    ws3.column_dimensions['B'].width = 50
    ws3.column_dimensions['C'].width = 40
    ws3.column_dimensions['D'].width = 40

    # Guardar
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    out_path = OUTPUT_DIR / f"reporte_busqueda_{timestamp}.xlsx"
    wb.save(str(out_path))
    return str(out_path)


if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="Buscar frases en URLs y generar reporte")
    parser.add_argument('--urls',    required=True, help='URLs separadas por coma')
    parser.add_argument('--phrases', required=True, help='Frases a buscar separadas por coma')
    args = parser.parse_args()

    urls    = [u.strip() for u in args.urls.split(',')    if u.strip()]
    phrases = [p.strip() for p in args.phrases.split(',') if p.strip()]

    print("=" * 65)
    print("  SEARCH CONTENT — Validador de Frases en URLs")
    print("=" * 65)
    print(f"  URLs    : {len(urls)}")
    print(f"  Frases  : {len(phrases)}")
    print(f"  Búsquedas totales: {len(urls) * len(phrases)}")
    print(f"  Output  : {OUTPUT_DIR}")
    print("=" * 65)

    results = asyncio.run(run_search(urls, phrases))

    print("\nGenerando reporte Excel...")
    excel_path = generate_excel_report(results, urls, phrases)

    found_total = sum(1 for r in results if r['found'])
    total = len(results)

    print("\n" + "=" * 65)
    print(f"  RESULTADO: {found_total}/{total} frases encontradas")
    print(f"  Reporte : {excel_path}")
    print(f"  Capturas: {SCREENSHOTS_DIR}")
    print("=" * 65)
```

---

## Paso 3 — Verificar dependencias e instalar si faltan

```bash
pip install playwright openpyxl
python -m playwright install chromium
```

---

## Paso 4 — Ejecutar el script

Construir el comando con los parámetros recopilados del usuario:

```bash
python "C:\Users\jesus\Desktop\Ilumno\search-content\search_content.py" \
  --urls "URL1, URL2, URL3" \
  --phrases "frase uno, frase dos, frase tres"
```

Usar siempre el venv del proyecto:
```bash
source "c:/Users/jesus/Desktop/Ilumno/Ley no moslestar/.venv/Scripts/activate" && python "C:/Users/jesus/Desktop/Ilumno/search-content/search_content.py" --urls "URL1, URL2" --phrases "frase uno, frase dos"
```

---

## Paso 5 — Presentar resultados al usuario

Una vez ejecutado el script, reportar:

1. **Tabla Markdown resumen** con el estado de cada URL x Frase:
   - `ENCONTRADA (N)` — encontrada con N ocurrencias
   - `NO ENCONTRADA` — no aparece en la página
   - `ERROR` — falló la navegación

2. **Ruta del reporte Excel** generado en:
   `C:\Users\jesus\Desktop\Ilumno\search-content\output\reporte_busqueda_YYYYMMDD_HHMMSS.xlsx`

3. **Ruta de las capturas** de pantalla (solo para frases encontradas):
   `C:\Users\jesus\Desktop\Ilumno\search-content\output\screenshots\`

4. **Métricas finales**:
   - Total frases encontradas / total búsquedas
   - Porcentaje de éxito global

---

## Estructura del Reporte Excel

| Hoja | Contenido |
|------|-----------|
| **Resumen** | Matriz URL × Frase con estado por celda + % éxito por fila |
| **Detalle** | Tabla completa con ruta de screenshot por búsqueda |
| **No Encontradas** | Solo los casos fallidos para facilitar revisión |

### Código de colores
| Color | Significado |
|-------|-------------|
| Verde `#C6EFCE` | Frase ENCONTRADA |
| Rojo claro `#FFC7CE` | NO ENCONTRADA |
| Amarillo `#FFEB9C` | ERROR al navegar |
| Naranja claro `#FCE4D6` | Éxito parcial (algunas frases encontradas) |

---

## Reglas

1. Siempre usar **Playwright headless** (Chromium) — no `requests` ni BeautifulSoup sola.
2. Esperar `networkidle` + 2 segundos extra para contenido dinámico (lazy load, JS).
3. El resaltado inyecta CSS con borde rojo + fondo dorado + sombra para máxima visibilidad.
4. Las capturas muestran el **viewport centrado en la primera ocurrencia** de la frase.
5. Nunca interrumpir la búsqueda por error en una URL — continuar con las siguientes.
6. Si la URL no tiene protocolo, agregar `https://` automáticamente.
7. Output **siempre** en `C:\Users\jesus\Desktop\Ilumno\search-content\output\`.
8. Responder siempre en español.
9. Si Playwright o openpyxl no están instalados, instalarlos antes de ejecutar.
10. La búsqueda es **case-insensitive** (no distingue mayúsculas/minúsculas).
11. Ignorar texto dentro de etiquetas `<script>`, `<style>`, `<noscript>` y `<textarea>`.
12. El reporte Excel debe ser claro, ordenado y fácil de interpretar visualmente.
13. En caso de error, registrar el mensaje de error en el reporte y continuar.
14. Al finalizar, presentar un resumen claro al usuario con los resultados y rutas de los archivos generados.
15. Si en en listado de frases hay valores repetidos, tratarlos como una sola búsqueda (no repetir la búsqueda para la misma frase).

---

## Ejemplo de salida esperada en chat

```
## Resultados — Search Content

| URL | "Contáctanos" | "Llámanos ahora" | % Éxito |
|-----|--------------|-----------------|---------|
| https://ejemplo.com/landing | ENCONTRADA (3) | NO ENCONTRADA | 50% |
| https://otro.com/pagina    | ENCONTRADA (1) | ENCONTRADA (2) | 100% |

**Reporte Excel:** `C:\...\output\reporte_busqueda_20260326_142530.xlsx`
**Capturas:** `C:\...\output\screenshots\` (3 imágenes)

**Total:** 3/4 frases encontradas (75% éxito global)
```

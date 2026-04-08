---
name: fuentes-usadas
description: >
  Usa esta skill cuando el usuario quiera conocer, auditar o listar TODAS las fuentes
  tipográficas (fonts) utilizadas en una o varias URLs. Actívala ante solicitudes como
  "qué fuentes usa esta página", "listar tipografías", "fuentes usadas en esta URL",
  "auditar fonts", "qué Google Fonts carga", "fonts de esta landing", "tipografías web",
  "font audit", "listar fuentes CSS", o cualquier petición que implique descubrir las
  fuentes tipográficas de un sitio web.
compatibility:
  required_tools:
    - run_in_terminal (para ejecutar el script de extracción)
    - fetch_webpage (para obtener el HTML de las URLs)
    - create_file
    - read_file
  python_packages:
    - playwright (pip install playwright && python -m playwright install chromium)
    - beautifulsoup4 (pip install beautifulsoup4)
    - requests (pip install requests)
    - openpyxl (pip install openpyxl) — opcional, para reporte Excel
---

# Skill: Fuentes Usadas — Auditor de Tipografías Web

## Objetivo

Extraer y reportar **todas** las fuentes tipográficas que utiliza una o varias URLs,
incluyendo:

- Fuentes declaradas en CSS (`font-family`)
- Fuentes cargadas via `@font-face`
- Google Fonts (`fonts.googleapis.com`)
- Adobe Fonts / Typekit (`use.typekit.net`)
- Fuentes del sistema (fallbacks)
- Fuentes cargadas dinámicamente vía JavaScript
- Variables CSS (`--font-*`) que resuelvan a tipografías

## Flujo de Trabajo

### Paso 1 — Recopilar URLs

- El usuario proporciona una o más URLs.
- Si da un dominio sin protocolo, asumir `https://`.
- Si pide "todas las landings de X", preguntar cuáles exactamente o si existe un listado.

### Paso 2 — Generar y ejecutar script de extracción

Crear un script Python temporal en la carpeta del workspace que use **Playwright** para:

1. Navegar a cada URL con un navegador headless (Chromium).
2. Esperar a que la página cargue completamente (`networkidle`).
3. Extraer fuentes de MÚLTIPLES fuentes de datos:

```python
# === ESTRATEGIA DE EXTRACCIÓN (ejecutar en page.evaluate) ===

# A) document.fonts API — fuentes realmente renderizadas
fonts_rendered = page.evaluate("""() => {
    const fonts = new Set();
    for (const font of document.fonts) {
        fonts.add(font.family.replace(/['"]/g, ''));
    }
    return [...fonts];
}""")

# B) Computed styles — font-family de TODOS los elementos visibles
fonts_computed = page.evaluate("""() => {
    const fonts = new Set();
    document.querySelectorAll('*').forEach(el => {
        const style = getComputedStyle(el);
        const family = style.fontFamily;
        if (family) {
            family.split(',').forEach(f => {
                fonts.add(f.trim().replace(/['"]/g, ''));
            });
        }
    });
    return [...fonts];
}""")

# C) Hojas de estilo — @font-face y font-family en reglas CSS
fonts_stylesheets = page.evaluate("""() => {
    const fonts = new Set();
    const fontFaces = [];
    for (const sheet of document.styleSheets) {
        try {
            for (const rule of sheet.cssRules) {
                if (rule instanceof CSSFontFaceRule) {
                    const family = rule.style.getPropertyValue('font-family')
                        .replace(/['"]/g, '');
                    const src = rule.style.getPropertyValue('src');
                    fontFaces.push({ family, src });
                    fonts.add(family);
                }
                if (rule.style && rule.style.fontFamily) {
                    rule.style.fontFamily.split(',').forEach(f => {
                        fonts.add(f.trim().replace(/['"]/g, ''));
                    });
                }
            }
        } catch(e) { /* CORS: hoja externa */ }
    }
    return { fonts: [...fonts], fontFaces };
}""")

# D) Link tags — Google Fonts, Adobe Fonts, otras CDN
font_links = page.evaluate("""() => {
    const links = [];
    document.querySelectorAll('link[href*="fonts.googleapis.com"], link[href*="fonts.gstatic.com"], link[href*="use.typekit.net"], link[href*="fonts.adobe.com"]').forEach(link => {
        links.push(link.href);
    });
    return links;
}""")

# E) Variables CSS que contienen nombres de fuentes
css_vars = page.evaluate("""() => {
    const vars = {};
    const root = getComputedStyle(document.documentElement);
    const style = document.documentElement.style;
    // Buscar variables comunes de fuentes
    const allProps = [...document.styleSheets].flatMap(s => {
        try {
            return [...s.cssRules].flatMap(r => r.cssText.match(/--[\\w-]*font[\\w-]*/gi) || []);
        } catch(e) { return []; }
    });
    [...new Set(allProps)].forEach(v => {
        const val = root.getPropertyValue(v).trim();
        if (val) vars[v] = val;
    });
    return vars;
}""")

# F) Archivos de fuentes descargados (network requests)
# Capturar antes de navegar con page.on("response") filtrando .woff, .woff2, .ttf, .otf, .eot
```

4. Consolidar resultados eliminando duplicados y clasificando.

### Paso 3 — Clasificar las fuentes

Organizar las fuentes encontradas en categorías:

| Categoría | Ejemplo |
|---|---|
| **Google Fonts** | Montserrat, Open Sans, Roboto |
| **Adobe Fonts** | fuentes cargadas desde Typekit |
| **Custom (@font-face)** | fuentes propias del sitio |
| **Sistema / Fallback** | Arial, Helvetica, sans-serif, serif |
| **Desconocida** | fuentes que no encajan en las anteriores |

### Paso 4 — Generar reporte

Presentar el resultado en **formato Markdown** directamente en el chat con:

1. **Tabla resumen** por URL:
   - URL analizada
   - Total de fuentes únicas
   - Fuentes por categoría

2. **Detalle por fuente**:
   - Nombre de la fuente
   - Categoría (Google / Adobe / Custom / Sistema)
   - Formato del archivo (woff2, woff, ttf, etc.)
   - URL de carga (si aplica)
   - Elementos que la usan (top 5 selectores CSS)

3. **Hallazgos relevantes**:
   - Fuentes declaradas pero NO cargadas
   - Fuentes cargadas pero NO usadas en ningún elemento
   - Formatos desactualizados (eot, ttf sin woff2)
   - Exceso de variantes/pesos cargados

4. **Si el usuario lo pide**, generar un archivo Excel (.xlsx) o JSON con los datos completos.

## Reglas

1. **Siempre usar Playwright** (no solo requests/BeautifulSoup) porque muchas fuentes se cargan vía JS.
2. **Interceptar requests de red** para capturar archivos .woff/.woff2/.ttf/.otf/.eot descargados.
3. **No incluir fuentes genéricas CSS** como categorías propias (serif, sans-serif, monospace, cursive, fantasy) — marcarlas como "Genérica CSS".
4. **Deduplicar**: "Montserrat" y "'Montserrat'" son la misma fuente.
5. **Timeout**: Máximo 30 segundos por URL. Si falla, reportar el error y continuar con la siguiente.
6. Si Playwright no está instalado, instalarlo automáticamente antes de ejecutar.
7. Responder en español siempre.
8. Si el usuario pide comparar fuentes entre varias URLs, incluir una **tabla comparativa**.

## Ejemplo de Salida Esperada

```markdown
## 🔤 Fuentes Usadas — https://ejemplo.com

### Resumen
| Categoría       | Cantidad | Fuentes                          |
|-----------------|----------|----------------------------------|
| Google Fonts    | 2        | Montserrat, Open Sans            |
| Custom          | 1        | MiFuentePropia                   |
| Sistema         | 3        | Arial, Helvetica, sans-serif     |
| **Total único** | **6**    |                                  |

### Detalle
| Fuente        | Categoría    | Formato | Pesos cargados   | URL de carga                         |
|---------------|-------------|---------|-------------------|--------------------------------------|
| Montserrat    | Google Fonts | woff2   | 400, 600, 700     | fonts.gstatic.com/s/montserrat/...   |
| Open Sans     | Google Fonts | woff2   | 400, 400i, 700    | fonts.gstatic.com/s/opensans/...     |
| MiFuentePropia| Custom       | woff2   | 400               | ejemplo.com/fonts/mifuente.woff2     |

### Hallazgos
- ⚠️ Se cargan 3 pesos de Montserrat pero solo se usan 400 y 700 en el DOM.
- ✅ Todas las fuentes custom usan formato woff2 (óptimo).
```

## Instalación de Dependencias

Si las dependencias no están instaladas, ejecutar:

```bash
pip install playwright beautifulsoup4 requests openpyxl
python -m playwright install chromium
```

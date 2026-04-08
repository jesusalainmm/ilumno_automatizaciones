---
name: auditoria-seo
description: Cuando el usuario quiere auditar, revisar o diagnosticar problemas de SEO en su sitio. También usar cuando el usuario menciona "auditoría SEO", "SEO técnico", "por qué no estoy posicionando", "problemas de SEO", "SEO on-page", "revisión de meta etiquetas" o "chequeo de salud SEO". Para construir páginas a escala orientadas a keywords, ver programmatic-seo. Para añadir datos estructurados, ver schema-markup.
---

# Auditoría SEO — Skill Automatizado

Eres un experto en optimización para motores de búsqueda. Tu objetivo es auditar una URL proporcionada por el usuario, identificar problemas de SEO y generar un informe Excel con los resultados.

---

## PASO 0 — Solicitar la URL

**OBLIGATORIO: Antes de hacer cualquier cosa, pregunta al usuario:**

> ¿Cuál es la URL que deseas auditar?

Si el usuario ya proporcionó la URL en su mensaje, úsala directamente. Acepta una sola URL o una lista de URLs separadas por comas/saltos de línea.

---

## PASO 1 — Obtener el HTML de la página

Usa la herramienta `fetch_webpage` para descargar el contenido de la URL proporcionada. Si hay múltiples URLs, descarga cada una.

- Si `fetch_webpage` falla, informa al usuario y sugiere verificar la URL.
- Guarda mentalmente el HTML completo para analizar en los pasos siguientes.

---

## PASO 2 — Analizar el HTML (Auditoría automática)

Con el HTML obtenido, evalúa **todos** los puntos del checklist a continuación. Para cada punto asigna un veredicto:

| Veredicto | Significado |
|-----------|-------------|
| ✅ OK | Cumple correctamente |
| ⚠️ ADVERTENCIA | Cumple parcialmente o podría mejorar |
| ❌ ERROR | No cumple, problema detectado |
| ℹ️ N/A | No aplica o no se puede verificar desde el HTML |

### 2.1 — Meta Tags y Head

| # | Punto de verificación | Cómo verificar |
|---|----------------------|----------------|
| 1 | Title tag presente y único | Buscar `<title>` — debe existir exactamente uno |
| 2 | Title entre 50-60 caracteres | Contar caracteres del contenido de `<title>` |
| 3 | Meta description presente | Buscar `<meta name="description">` |
| 4 | Meta description entre 150-160 chars | Contar caracteres del `content` |
| 5 | Canonical tag presente | Buscar `<link rel="canonical">` |
| 6 | Canonical apunta a URL correcta | Comparar href con la URL auditada |
| 7 | Viewport meta configurado | Buscar `<meta name="viewport">` |
| 8 | Charset declarado | Buscar `<meta charset>` o `Content-Type` |
| 9 | Idioma declarado | Buscar `lang` en `<html>` |
| 10 | Favicon presente | Buscar `<link rel="icon">` o `<link rel="shortcut icon">` |
| 11 | Open Graph tags (og:title, og:description, og:image) | Buscar `<meta property="og:...">` |
| 12 | Twitter Card tags | Buscar `<meta name="twitter:...">` |
| 13 | Robots meta (noindex/nofollow) | Buscar `<meta name="robots">` — alertar si tiene noindex |

### 2.2 — Estructura de Encabezados

| # | Punto de verificación | Cómo verificar |
|---|----------------------|----------------|
| 14 | Exactamente un H1 | Contar `<h1>` en la página |
| 15 | H1 contiene texto relevante | Verificar contenido del H1 |
| 16 | Jerarquía de headings correcta | No debe haber saltos (H1→H3 sin H2) |
| 17 | Encabezados no vacíos | Ningún `<h1>`..`<h6>` vacío |

### 2.3 — Imágenes

| # | Punto de verificación | Cómo verificar |
|---|----------------------|----------------|
| 18 | Todas las imágenes tienen alt | Buscar `<img>` sin atributo `alt` o con alt vacío |
| 19 | Imágenes usan lazy loading | Buscar `loading="lazy"` en `<img>` |
| 20 | Formatos modernos (WebP/AVIF) | Revisar extensiones en `src` |

### 2.4 — Enlaces

| # | Punto de verificación | Cómo verificar |
|---|----------------------|----------------|
| 21 | Enlaces internos presentes | Contar `<a href>` al mismo dominio |
| 22 | Enlaces externos con rel apropiado | Verificar `rel="noopener"` en enlaces externos target="_blank" |
| 23 | Sin enlaces rotos (href vacíos o #) | Buscar `<a href="">` o `<a href="#">` |

### 2.5 — Rendimiento y Recursos

| # | Punto de verificación | Cómo verificar |
|---|----------------------|----------------|
| 24 | CSS crítico o inline | Buscar `<style>` en head o atributos inline |
| 25 | JavaScript defer/async | Buscar `<script>` sin defer/async en head |
| 26 | Sin CSS/JS bloqueante en head | Scripts en head sin defer/async |

### 2.6 — Datos Estructurados

| # | Punto de verificación | Cómo verificar |
|---|----------------------|----------------|
| 27 | Schema.org / JSON-LD presente | Buscar `<script type="application/ld+json">` |
| 28 | Schema válido (estructura correcta) | Parsear el JSON-LD y verificar @type |

### 2.7 — Seguridad y Protocolo

| # | Punto de verificación | Cómo verificar |
|---|----------------------|----------------|
| 29 | URL usa HTTPS | Verificar protocolo de la URL |
| 30 | Sin contenido mixto | No hay recursos http:// dentro de una página https:// |

### 2.8 — Contenido

| # | Punto de verificación | Cómo verificar |
|---|----------------------|----------------|
| 31 | Contenido visible suficiente (>300 palabras) | Contar texto visible (sin tags) |
| 32 | Sin contenido oculto sospechoso | Buscar `display:none` en grandes bloques de texto |
| 33 | Ratio texto/HTML razonable (>10%) | Calcular proporción texto/HTML total |

### 2.9 — Revisión Ortográfica

| # | Punto de verificación | Cómo verificar |
|---|----------------------|----------------|
| 34 | Detectar idioma de la página | Leer el atributo `lang` de `<html>`. Si no existe, inferir del contenido visible. Idiomas soportados: es, en, pt, fr, de, it, etc. |
| 35 | Ortografía del Title | Revisar el texto del `<title>` buscando errores ortográficos en el idioma detectado |
| 36 | Ortografía de la Meta Description | Revisar el texto de `<meta name="description">` buscando errores ortográficos |
| 37 | Ortografía de los Encabezados (H1-H6) | Revisar el texto de cada encabezado buscando errores ortográficos |
| 38 | Ortografía del contenido visible | Extraer el texto visible de la página (sin tags HTML) y buscar errores ortográficos. Ignorar: nombres propios, marcas, URLs, emails, código, palabras técnicas y anglicismos comunes del sector |

**Criterios para la revisión ortográfica:**

- **Idioma**: Usar el idioma detectado en `lang` del `<html>`. Si es `es` o `es-*`, revisar en español. Si es `en` o `en-*`, en inglés. Etc.
- **Qué reportar**: Palabras mal escritas, tildes faltantes (en español), errores de concordancia evidentes, mayúsculas incorrectas en medio de frase.
- **Qué ignorar**: Nombres propios, marcas registradas, siglas, URLs, emails, hashtags, @ mentions, palabras en otro idioma entrecomilladas, términos técnicos del sector (SEO, CTA, landing, etc.), código.
- **Veredicto**:
  - ✅ OK → 0 errores ortográficos encontrados
  - ⚠️ ADVERTENCIA → 1-5 errores ortográficos encontrados
  - ❌ ERROR → 6+ errores ortográficos encontrados
- **Detalle**: Listar cada error encontrado con formato: `"[palabra incorrecta]" → sugerencia correcta (ubicación: title/h1/h2/párrafo)`

### 2.10 — Responsividad y Compatibilidad Móvil

Esta sección revisa si la página tiene una arquitectura responsive adecuada para dispositivos móviles analizando el HTML y CSS.

| # | Punto de verificación | Cómo verificar |
|---|----------------------|----------------|
| 39 | Meta viewport presente y correcto | Buscar `<meta name="viewport">`. Debe tener `width=device-width` y `initial-scale=1`. Alertar si tiene `maximum-scale=1` o `user-scalable=no` (impide zoom, problema de accesibilidad) |
| 40 | Sin ancho fijo en body/containers | Buscar en `<style>` y atributos `style` inline: `width` en px en body, main, o contenedores principales (>992px fijo es problemático). `max-width` es aceptable |
| 41 | Uso de media queries | Buscar `@media` en `<style>` o en los `<link>` con atributo `media`. Debe haber al menos breakpoints para móvil (≤768px) y tablet (≤1024px) |
| 42 | Imágenes responsivas | Buscar `<img>` con ancho fijo en px en atributo `width` sin `max-width:100%`. Buscar `<picture>` con `<source>` para diferentes tamaños. Verificar si usan `srcset` |
| 43 | Tablas responsivas | Buscar `<table>`. Si existen, verificar que tengan `overflow-x:auto` en un contenedor o alguna estrategia responsive (no deben causar scroll horizontal) |
| 44 | Tamaño de fuentes legible en móvil | Buscar `font-size` menores a 12px en el CSS. Verificar que no usen unidades absolutas (pt, cm) sino relativas (rem, em, %, vw) |
| 45 | Touch targets adecuados (botones/enlaces) | Buscar `<button>`, `<a>`, `<input>` con `height`/`width`/`padding` muy pequeños (<44px). Google recomienda mínimo 48x48px para tap targets |
| 46 | Sin contenido que desborda horizontalmente | Buscar elementos con `width` > `100vw`, `position:absolute` con `left`/`right` grandes, o `overflow:hidden` en body (indicador de problema enmascarado) |
| 47 | Flexbox o Grid para layout | Buscar `display:flex` o `display:grid` en el CSS. Si el layout usa solo `float` o `position:absolute` para columnas, es un diseño legacy no responsive |
| 48 | No usa sitio móvil separado (m.) | Verificar que no exista `<link rel="alternate" media="only screen and (max-width:...)"` ni redirección a m.dominio.com. Google prefiere responsive sobre sitio separado |

**Criterios de veredicto para responsividad:**

- ✅ **OK**: El check se cumple correctamente
- ⚠️ **ADVERTENCIA**: Cumple parcialmente (ej: tiene viewport pero con restricciones de zoom; tiene media queries pero no cubre móvil)
- ❌ **ERROR**: No cumple (ej: sin viewport, anchos fijos en px, sin media queries, layout con floats)
- ℹ️ **N/A**: No se puede verificar desde el HTML estático (ej: CSS externo no accesible)

**Contexto importante para el análisis:**
- Si el CSS está en archivos externos (no inline), indicar como N/A con nota: "CSS externo no analizable desde HTML estático" y sugerir inspección manual
- Si la página usa frameworks CSS (Bootstrap, Tailwind, etc.), detectar las clases características y reportar como OK si están bien aplicadas
- Detectar frameworks por clases: `col-`, `container`, `row` (Bootstrap); `flex`, `grid`, `sm:`, `md:`, `lg:` (Tailwind); `v-container`, `v-row` (Vuetify)

---

## PASO 3 — Ejecutar Lighthouse

Tras el análisis HTML, ejecuta **Google Lighthouse** contra la URL para obtener métricas reales de rendimiento, accesibilidad, buenas prácticas y SEO.

### Cómo ejecutar Lighthouse

1. Verificar que `lighthouse` esté disponible (viene con Node.js):
   ```
   npx lighthouse --version
   ```
   Si no está instalado:
   ```
   npm install -g lighthouse
   ```

2. Ejecutar Lighthouse en modo headless con salida JSON:
   ```
   npx lighthouse "{{URL}}" --output=json --output-path="C:\Users\jesus\Desktop\Ilumno\auditoria-seo\output\{{DOMINIO_CARPETA}}_{{YYYYMMDD}}\lighthouse_{{DOMINIO}}.json" --chrome-flags="--headless --no-sandbox" --only-categories=performance,accessibility,best-practices,seo
   ```

3. Si Lighthouse falla (Chrome no disponible, timeout, etc.), **no bloquear la auditoría**. Registrar en el informe:
   - Veredicto: ℹ️ N/A
   - Detalle: "Lighthouse no pudo ejecutarse: [motivo del error]"
   - Recomendación: "Ejecutar manualmente en Chrome DevTools > Lighthouse"

### Datos a extraer del JSON de Lighthouse

| Dato | Ruta en JSON | Descripción |
|------|-------------|-------------|
| **Performance Score** | `categories.performance.score` | Puntuación 0-100 rendimiento |
| **Accessibility Score** | `categories.accessibility.score` | Puntuación 0-100 accesibilidad |
| **Best Practices Score** | `categories.best-practices.score` | Puntuación 0-100 buenas prácticas |
| **SEO Score** | `categories.seo.score` | Puntuación 0-100 SEO según Lighthouse |
| **FCP** | `audits.first-contentful-paint` | First Contentful Paint (ms) |
| **LCP** | `audits.largest-contentful-paint` | Largest Contentful Paint (ms) |
| **TBT** | `audits.total-blocking-time` | Total Blocking Time (ms) |
| **CLS** | `audits.cumulative-layout-shift` | Cumulative Layout Shift |
| **SI** | `audits.speed-index` | Speed Index (ms) |
| **TTI** | `audits.interactive` | Time to Interactive (ms) |

### Audits fallidos a reportar

Del JSON, extraer todos los audits donde `score < 1` (fallidos o parciales):

```
audits[key].score < 1 AND audits[key].score !== null
```

Para cada audit fallido extraer:
- `title` — Nombre del audit
- `description` — Descripción del problema
- `score` — Puntuación (0 a 0.99)
- `displayValue` — Valor mostrado (ej: "3.2 s", "15 elements")
- Categoría a la que pertenece (Performance/Accessibility/Best Practices/SEO)

### Umbrales de puntuación Lighthouse

| Rango | Color | Veredicto |
|-------|-------|----------|
| 90-100 | Verde | ✅ Excelente |
| 50-89 | Naranja | ⚠️ Necesita mejoras |
| 0-49 | Rojo | ❌ Pobre |

### Métricas Core Web Vitals — Umbrales

| Métrica | Bueno | Necesita mejora | Pobre |
|---------|-------|-----------------|-------|
| FCP | ≤ 1.8s | ≤ 3.0s | > 3.0s |
| LCP | ≤ 2.5s | ≤ 4.0s | > 4.0s |
| TBT | ≤ 200ms | ≤ 600ms | > 600ms |
| CLS | ≤ 0.1 | ≤ 0.25 | > 0.25 |
| SI | ≤ 3.4s | ≤ 5.8s | > 5.8s |
| TTI | ≤ 3.8s | ≤ 7.3s | > 7.3s |

---

## PASO 4 — Capturar Screenshots de la página auditada

Tras el análisis HTML y Lighthouse, genera screenshots de la URL en **todos los dispositivos** usando Playwright. Esto permite documentar visualmente los hallazgos encontrados.

### Definición de la carpeta de salida

Antes de cualquier otro paso, construye la ruta base de salida:

```
DOMINIO_CARPETA = dominio extraído de la URL (sin www, sin protocolo, puntos → guion bajo)
CARPETA_BASE    = C:\Users\jesus\Desktop\Ilumno\auditoria-seo\output\{DOMINIO_CARPETA}_{YYYYMMDD}\
CARPETA_SCREENSHOTS = {CARPETA_BASE}screenshots\
```

**Ejemplo:** Para `https://www.ejemplo.com/producto` → carpeta `ejemplo_com_20260325`

### Dispositivos a capturar (OBLIGATORIO: todos)

| Dispositivo | Nombre archivo | Viewport | User-Agent |
|-------------|---------------|----------|-----------|
| Desktop | `desktop_1920x1080.png` | 1920×1080 | Chrome desktop |
| Mobile Android (Pixel 7) | `mobile_android_pixel7_412x915.png` | 412×915 | Chrome Android |
| Mobile iOS (iPhone 14) | `mobile_ios_iphone14_390x844.png` | 390×844 | Safari iOS |
| Tablet Android (Galaxy Tab S8) | `tablet_android_galaxy_tab_800x1280.png` | 800×1280 | Chrome Android tablet |
| Tablet iOS (iPad Air) | `tablet_ios_ipad_820x1180.png` | 820×1180 | Safari iPad |

### Script Python para screenshots con Playwright

Genera y ejecuta este script (`screenshots_seo.py`) en la carpeta del proyecto:

```python
import asyncio
from playwright.async_api import async_playwright
import os

URL = "{{URL}}"
CARPETA_SCREENSHOTS = r"C:\Users\jesus\Desktop\Ilumno\auditoria-seo\output\{{DOMINIO_CARPETA}}_{{YYYYMMDD}}\screenshots"
os.makedirs(CARPETA_SCREENSHOTS, exist_ok=True)

DISPOSITIVOS = [
    {
        "nombre": "desktop_1920x1080",
        "viewport": {"width": 1920, "height": 1080},
        "user_agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/121.0.0.0 Safari/537.36",
        "is_mobile": False,
        "device_scale_factor": 1,
    },
    {
        "nombre": "mobile_android_pixel7_412x915",
        "viewport": {"width": 412, "height": 915},
        "user_agent": "Mozilla/5.0 (Linux; Android 13; Pixel 7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/121.0.6167.101 Mobile Safari/537.36",
        "is_mobile": True,
        "device_scale_factor": 2.625,
    },
    {
        "nombre": "mobile_ios_iphone14_390x844",
        "viewport": {"width": 390, "height": 844},
        "user_agent": "Mozilla/5.0 (iPhone; CPU iPhone OS 17_2 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.2 Mobile/15E148 Safari/604.1",
        "is_mobile": True,
        "device_scale_factor": 3,
    },
    {
        "nombre": "tablet_android_galaxy_tab_800x1280",
        "viewport": {"width": 800, "height": 1280},
        "user_agent": "Mozilla/5.0 (Linux; Android 13; SM-X706B) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/121.0.6167.101 Safari/537.36",
        "is_mobile": False,
        "device_scale_factor": 2,
    },
    {
        "nombre": "tablet_ios_ipad_820x1180",
        "viewport": {"width": 820, "height": 1180},
        "user_agent": "Mozilla/5.0 (iPad; CPU OS 17_2 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.2 Mobile/15E148 Safari/604.1",
        "is_mobile": True,
        "device_scale_factor": 2,
    },
]

async def capturar():
    async with async_playwright() as p:
        for dispositivo in DISPOSITIVOS:
            browser = await p.chromium.launch(headless=True)
            context = await browser.new_context(
                viewport=dispositivo["viewport"],
                user_agent=dispositivo["user_agent"],
                is_mobile=dispositivo["is_mobile"],
                device_scale_factor=dispositivo["device_scale_factor"],
            )
            page = await context.new_page()
            try:
                await page.goto(URL, wait_until="networkidle", timeout=30000)
                await page.wait_for_timeout(2000)
                ruta = os.path.join(CARPETA_SCREENSHOTS, f"{dispositivo['nombre']}.png")
                await page.screenshot(path=ruta, full_page=True)
                print(f"✅ Screenshot guardado: {ruta}")
            except Exception as e:
                print(f"❌ Error en {dispositivo['nombre']}: {e}")
            finally:
                await browser.close()

asyncio.run(capturar())
```

### Instalación de Playwright si no está disponible

```bash
pip install playwright
python -m playwright install chromium
```

### Si Playwright falla

Si Playwright no puede ejecutarse (sin Chrome, sin red, timeout):
- **No bloquear la auditoría**. Continuar con el Excel.
- Registrar en el Excel (hoja Resumen): "Screenshots no disponibles — ejecutar `screenshots_seo.py` manualmente"
- Indicar al usuario los pasos para ejecutarlo manualmente.

---

## PASO 5 — Generar el informe Excel

Tras completar el análisis, **genera un script Python** que cree un archivo Excel (.xlsx) con los resultados. El script debe usar `openpyxl`.

### Estructura del Excel

El archivo Excel debe tener las siguientes hojas:

#### Hoja 1: "Resumen"
| Columna | Contenido |
|---------|-----------|
| A | URL auditada |
| B | Fecha de auditoría |
| C | Total checks realizados |
| D | Total ✅ OK |
| E | Total ⚠️ Advertencias |
| F | Total ❌ Errores |
| G | Puntuación SEO (% de OK sobre total aplicable) |

#### Hoja 2: "Detalle Auditoría"
| Columna | Contenido |
|---------|-----------|
| A | # (número del check) |
| B | Categoría (Meta Tags, Encabezados, Imágenes, etc.) |
| C | Punto de verificación |
| D | Veredicto (✅/⚠️/❌/ℹ️) |
| E | Detalle encontrado (valor actual del title, largo, etc.) |
| F | Recomendación (qué hacer para corregir si aplica) |

#### Hoja 3: "Revisión Ortográfica"
| Columna | Contenido |
|---------|----------|
| A | # (número secuencial) |
| B | Idioma detectado |
| C | Ubicación (Title / Meta Description / H1 / H2 / Párrafo / etc.) |
| D | Texto original con el error |
| E | Palabra incorrecta |
| F | Sugerencia de corrección |
| G | Severidad (Error grave / Error menor / Advertencia) |

**Formato de la hoja Revisión Ortográfica:**
- **Severidad "Error grave"** (tildes que cambian significado, palabras inexistentes): Fondo rojo claro (#FFC7CE)
- **Severidad "Error menor"** (tildes opcionales, mayúsculas): Fondo amarillo claro (#FFEB9C)
- **Severidad "Advertencia"** (posible error pero podría ser intencional): Fondo gris claro (#D9D9D9)
- Si no hay errores ortográficos, incluir una fila con mensaje: "No se encontraron errores ortográficos. ✅"

#### Hoja 4: "Revisión Responsive"
| Columna | Contenido |
|---------|----------|
| A | # (número del check) |
| B | Punto de verificación |
| C | Veredicto (✅/⚠️/❌/ℹ️) |
| D | Detalle encontrado (código o evidencia del HTML) |
| E | Recomendación |
| F | Framework detectado (si aplica: Bootstrap, Tailwind, etc.) |

**Formato de la hoja Revisión Responsive:**
- Mismos colores de veredicto que la hoja Detalle Auditoría (✅ verde, ⚠️ amarillo, ❌ rojo, ℹ️ gris)
- Si se detecta un framework CSS, incluir una fila de resumen al inicio: "Framework detectado: [nombre] — Las clases responsivas se verifican según su sistema"
- Resumen al final con puntuación responsive: % de checks OK sobre el total aplicable

#### Hoja 5: "Lighthouse"
| Columna | Contenido |
|---------|----------|
| A-D (fila 1-2) | **Puntajes generales**: Performance, Accessibility, Best Practices, SEO (0-100) con colores según umbral |
| A-D (fila 4+) | **Core Web Vitals**: FCP, LCP, TBT, CLS, SI, TTI con valores, umbrales y veredicto |

**Tabla de audits fallidos (desde fila 10+):**
| Columna | Contenido |
|---------|----------|
| A | Categoría (Performance / Accessibility / Best Practices / SEO) |
| B | Nombre del audit |
| C | Puntuación (0-1) |
| D | Valor mostrado (ej: "3.2 s") |
| E | Descripción / Recomendación |

**Formato de la hoja Lighthouse:**
- **Puntajes 90-100**: Fondo verde (#C6EFCE), fuente negrita grande
- **Puntajes 50-89**: Fondo naranja (#FFEB9C), fuente negrita grande
- **Puntajes 0-49**: Fondo rojo (#FFC7CE), fuente negrita grande
- Core Web Vitals coloreados según umbrales (verde/naranja/rojo)
- Audits agrupados por categoría con separador visual
- Si Lighthouse no pudo ejecutarse, mostrar mensaje: "Lighthouse no disponible — ejecutar manualmente en Chrome DevTools"

#### Hoja 6: "Plan de Acción"
| Columna | Contenido |
|---------|-----------|
| A | Prioridad (Crítica / Alta / Media / Baja) |
| B | Problema |
| C | Acción recomendada |
| D | Impacto esperado |

### Formato del Excel

El script debe aplicar estos estilos:
- **Encabezados**: Fondo azul oscuro (#1F4E79), texto blanco, negrita
- **Filas alternas**: Color de fondo alterno para legibilidad
- **Columna de veredicto**: Color de fondo según resultado:
  - ✅ OK → Verde claro (#C6EFCE)
  - ⚠️ ADVERTENCIA → Amarillo claro (#FFEB9C)
  - ❌ ERROR → Rojo claro (#FFC7CE)
  - ℹ️ N/A → Gris claro (#D9D9D9)
- **Ancho de columnas**: Autoajustado al contenido
- **Bordes**: Bordes finos en todas las celdas
- **Hoja Resumen**: Incluir un mini dashboard con celdas grandes y la puntuación SEO destacada

### Ubicación del archivo generado

- **Carpeta base por URL:** `C:\Users\jesus\Desktop\Ilumno\auditoria-seo\output\{DOMINIO_CARPETA}_{YYYYMMDD}\`
- **Excel:** `{CARPETA_BASE}auditoria_seo_{DOMINIO}_{YYYYMMDD}.xlsx`
- **JSON Lighthouse:** `{CARPETA_BASE}lighthouse_{DOMINIO}.json`
- **Screenshots:** `{CARPETA_BASE}screenshots\`
  - `desktop_1920x1080.png`
  - `mobile_android_pixel7_412x915.png`
  - `mobile_ios_iphone14_390x844.png`
  - `tablet_android_galaxy_tab_800x1280.png`
  - `tablet_ios_ipad_820x1180.png`

Donde:
- `DOMINIO_CARPETA` = dominio extraído de la URL (sin www, sin protocolo, puntos → guion bajo)
- `YYYYMMDD` = fecha del día de ejecución

**Ejemplo de estructura para `https://ejemplo.com`:**
```
C:\Users\jesus\Desktop\Ilumno\auditoria-seo\output\
└── ejemplo_com_20260325\
    ├── auditoria_seo_ejemplo_com_20260325.xlsx
    ├── lighthouse_ejemplo_com.json
    └── screenshots\
        ├── desktop_1920x1080.png
        ├── mobile_android_pixel7_412x915.png
        ├── mobile_ios_iphone14_390x844.png
        ├── tablet_android_galaxy_tab_800x1280.png
        └── tablet_ios_ipad_820x1180.png
```

---

## PASO 6 — Ejecutar el script y entregar

1. Crea la estructura de carpetas:
   - `C:\Users\jesus\Desktop\Ilumno\auditoria-seo\output\{DOMINIO_CARPETA}_{YYYYMMDD}\`
   - `C:\Users\jesus\Desktop\Ilumno\auditoria-seo\output\{DOMINIO_CARPETA}_{YYYYMMDD}\screenshots\`
2. Ejecuta Lighthouse contra la URL (PASO 3) y guarda el JSON en la carpeta del dominio.
3. Ejecuta el script de screenshots (PASO 4). Instala Playwright si es necesario.
4. Genera el script Python en `C:\Users\jesus\Desktop\Ilumno\auditoria-seo\auditoria_seo.py` incorporando datos de Lighthouse.
5. Instala `openpyxl` si no está disponible (`pip install openpyxl`).
6. Ejecuta el script Python del Excel.
7. Confirma al usuario la ubicación de **todos** los archivos generados (Excel + screenshots).
8. Muestra un resumen en texto con:
   - Puntuación SEO general (análisis HTML)
   - Puntuación Responsive
   - Puntajes Lighthouse (Performance, Accessibility, Best Practices, SEO)
   - Core Web Vitals (FCP, LCP, TBT, CLS)
   - Top 5 problemas encontrados (errores)
   - Top 3 quick wins (advertencias fáciles de resolver)
   - Lista de screenshots generados con su dispositivo

---

## Marco de Conocimiento SEO (Referencia para el análisis)

### Orden de Prioridad de Hallazgos
1. **Rastreo e indexación** — ¿Google puede encontrar e indexar la página?
2. **Fundamentos técnicos** — ¿La página carga bien?
3. **Rendimiento (Lighthouse)** — ¿Cumple Core Web Vitals?
4. **Optimización on-page** — ¿El contenido está optimizado?
5. **Accesibilidad (Lighthouse)** — ¿Es accesible para todos los usuarios?
6. **Calidad del contenido** — ¿Merece posicionar?
7. **Responsividad móvil** — ¿La página funciona bien en móvil?
8. **Datos estructurados** — ¿Google entiende el contenido?

### Criterios de Prioridad para Plan de Acción

| Prioridad | Criterio |
|-----------|----------|
| **Crítica** | Impide indexación o rastreo (noindex, canonical rota, sin title). Lighthouse Performance < 50 |
| **Alta** | Afecta posicionamiento directo (H1 ausente, meta description faltante, contenido pobre, errores ortográficos graves). Lighthouse Accessibility < 70. Core Web Vitals en rojo |
| **Media** | Mejora experiencia y CTR (Open Graph, favicon, Twitter Cards, problemas responsive parciales) |
| **Baja** | Optimizaciones menores (lazy loading, formatos de imagen, orden de scripts) |

### Señales E-E-A-T a considerar
- **Experiencia**: Contenido con experiencia real demostrada
- **Expertise**: Información precisa y autor visible
- **Autoridad**: Reconocimiento externo, citaciones
- **Trust**: HTTPS, datos de contacto, políticas legales

---

## Problemas comunes por tipo de sitio (contexto adicional)

| Tipo | Problemas frecuentes |
|------|---------------------|
| **Landing page** | Title genérico, sin H1 claro, sin schema, meta description ausente |
| **E-commerce** | Contenido duplicado, filtros mal gestionados, imágenes sin alt |
| **Blog** | Canibalización, contenido desactualizado, mal enlazado interno |
| **SaaS** | Páginas de producto débiles, blog desconectado |
| **Local** | NAP inconsistente, sin schema local |

---

## Habilidades relacionadas

- programmatic-seo
- schema-markup
- page-cro
- analytics-tracking
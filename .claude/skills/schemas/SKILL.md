---
name: schemas
description: >
  Usa esta skill siempre que el usuario necesite validar un agente automatizador especializado en validación de datos estructurados (JSON-LD / Schema) en páginas web.
---

# Agente de Validación de Schemas

Tu tarea es ejecutar una validación completa de datos estructurados leyendo un archivo Excel, extrayendo JSONs desde Google Drive y páginas web, comparándolos, escribiendo los resultados en el **mismo archivo Excel de origen** fila por fila, y generando un informe Excel adicional al finalizar.

---

## MODO DE EJECUCIÓN — Elegir antes de comenzar

**Antes de hacer cualquier otra cosa**, detecta si ya existe el archivo `schemas_automation.py` en el directorio de trabajo del usuario.

### MODO A — Script ya existe (ejecución sin IA)

Si `schemas_automation.py` **existe**:

1. Ejecutar directamente:
   ```bash
   python schemas_automation.py
   ```
2. Mostrar al usuario el progreso y resultado final.
3. Si el script falla con un error, reportar el error y preguntar si la IA debe corregirlo.
4. **No consumir tokens adicionales** — toda la lógica la ejecuta el script Python.

---

### MODO B — Primera ejecución (la IA ejecuta las tareas y genera el script)

Si `schemas_automation.py` **NO existe**:

1. Ejecutar todas las tareas descritas en los PASOS 1 al 10.
2. Al finalizar, **generar automáticamente `schemas_automation.py`** con toda la lógica implementada.
3. Avisar al usuario:
   > "Se ha creado `schemas_automation.py`. La próxima vez el proceso correrá automáticamente sin consumir tokens de IA."

---

### Especificaciones del script `schemas_automation.py` a generar

El script debe ser autónomo. Librerías requeridas:

```
openpyxl       → leer y escribir el Excel de origen y el informe Excel
requests       → obtener HTML de páginas web y documentos Drive
beautifulsoup4 → parsear HTML y buscar data-schema-type
selenium       → abrir Chrome, tomar capturas de pantalla
json           → parsear y formatear JSON
Pillow         → manipular imágenes de capturas
re             → expresiones regulares para nombre de archivos
```

El script debe:
- Recibir la ruta del Excel como argumento: `python schemas_automation.py ruta/archivo.xlsx`
- Si no se pasa argumento, buscar automáticamente el primer `.xlsx` en el directorio actual.
- Procesar fila por fila desde la fila 2 hasta la última con datos.
- **Sobrescribir el mismo Excel de origen** escribiendo resultados en columnas C, D, E y F.
- Crear las carpetas junto al archivo Excel si no existen, y guardar ahí todo lo generado:
  `output/`, `output/drive/`, `output/schemas/`, `output/capturas/`.
- Guardar archivos `.json` del Drive formateados en `output/drive/` (uno por fila).
- Guardar archivos `.json` del schema de la página web formateados en `output/schemas/` (uno por fila).
- Tomar capturas de pantalla usando Selenium con Chrome headless y guardarlas en `output/capturas/`.
- Generar el informe `output/reporte_validacion_schema.xlsx` con 4 hojas al finalizar.
- Registrar errores en `output/error_log.txt` sin detener el proceso.
- Imprimir en consola el progreso de cada fila procesada.

---

## Herramientas disponibles

| Herramienta         | Capacidades                                                               |
|---------------------|---------------------------------------------------------------------------|
| Excel Reader/Writer | Leer columnas A y B, escribir resultados en columnas C, D, E y F          |
| Navegador Chrome    | Abrir URLs, obtener HTML fuente completo, tomar capturas                  |
| File System         | Crear archivos `.json`, guardar JSON formateado, guardar capturas         |
| JSON Parser         | Validar, formatear y comparar estructuras JSON                            |
| WebFetch            | Obtener contenido de páginas web y documentos Google Drive                |

---

## Estructura del archivo Excel de origen

Las columnas de **lectura** y **escritura** son sobre el **mismo archivo Excel** que el usuario proporciona:

| Columna | Cabecera en el Excel              | Operación                               |
|---------|-----------------------------------|-----------------------------------------|
| A       | Enlace programa                   | Solo lectura (URL de la página web)     |
| B       | Enlace documento                  | Solo lectura (URL Google Drive con JSON)|
| C       | data-schema-type                  | **Escribir** `SI` / `NO`               |
| D       | Schema en el head                 | **Escribir** `SI` / `NO`               |
| E       | Comparacion de Schema             | **Escribir** `Aprobado` / `No Aprobado`|
| F       | Prueba de texto enriquecido       | **Escribir** `Aprobado` / `No Aprobado`|

> El archivo Excel de origen es **sobrescrito** directamente. Los resultados se escriben en sus columnas correspondientes fila a fila.

---

## Flujo de ejecución — por cada fila

Procesar **fila por fila** desde la fila 2 hasta la última fila con datos.

---

### PASO 1 — Leer el archivo Excel y preparar carpeta output

- Abrir el archivo Excel indicado por el usuario.
- Determinar la **ruta raíz** del archivo Excel (la carpeta donde está ubicado).
- Crear las siguientes carpetas dentro de esa misma raíz si no existen:
  ```
  <raíz del Excel>/output/
  <raíz del Excel>/output/drive/
  <raíz del Excel>/output/schemas/
  <raíz del Excel>/output/capturas/
  ```
- **Todo lo que se genere** (`.json`, capturas, informe, log de errores) se guarda dentro de `output/`.
- Determinar el número total de filas con datos.
- Columnas de lectura: **A** (Enlace programa = URL web) y **B** (Enlace documento = URL Drive).
- Columnas de escritura: **C** (data-schema-type), **D** (Schema en el head), **E** (Comparacion de Schema), **F** (Prueba de texto enriquecido).
- Comenzar el ciclo desde la fila 2.

---

### PASO 2 — Obtener JSON desde Google Drive (columna B)

**Entrada:** URL del documento Google Drive en la columna B de la fila actual.

**Descripción:** La columna B contiene el enlace a un **documento de Google Drive** (Google Doc u otro formato de texto). Al abrir ese enlace se muestra un documento con contenido de texto. Dentro de ese documento hay un JSON escrito como texto. El objetivo es leer todo el contenido del documento, localizar el JSON dentro del texto y extraerlo.

**Proceso:**
1. Abrir el enlace de la columna B en el navegador.
2. Esperar a que el documento cargue completamente.
3. **Leer todo el contenido de texto del documento** (el documento puede ser un Google Doc, un archivo de texto plano o similar).
4. Dentro del texto del documento, **buscar y extraer el JSON**:
   - Buscar la primera aparición de `{` y la última de `}` que formen un bloque JSON válido.
   - Extraer ese bloque completo como texto.
5. Parsear el texto extraído como JSON para validar que es bien formado.
6. Formatear con indentación de 2 espacios (pretty print JSON).
7. Guardar internamente como `JSON_DRIVE`.
8. Guardar el JSON en un archivo `.json` dentro de `output/drive/`:
   - **Nombre del archivo:** tomar la URL de la columna A (la página web, no la URL del Drive) y reemplazar `:` `/` `\` `.` por guion bajo `_`, con extensión `.json`.
   - Ejemplo: `https://example.com/pagina` → `output/drive/https___example_com_pagina.json`
   - El mismo nombre que el archivo web permite emparejar ambos archivos por fila fácilmente.

**Estrategias para leer el contenido del documento Drive:**
- Si la URL es de Google Docs: convertir a URL de exportación de texto plano:
  `https://docs.google.com/document/d/{ID}/export?format=txt`
- Si la URL es de visualización directa (`/view` o `/edit`): obtener el texto renderizado con Selenium.
- En ambos casos, leer **todo** el contenido antes de buscar el JSON.

**Si el JSON no se encuentra, es inválido o el enlace no carga:**
- Registrar error en `output/error_log.txt` con: fila, URL Drive, descripción del fallo.
- Asignar `JSON_DRIVE = null`.
- No crear archivo en `output/drive/`.
- Continuar con PASO 3.

---

### PASO 3 — Analizar la página web y extraer `data-schema-type` (columna A)

**Entrada:** URL de la página web en la columna A de la fila actual.

**Proceso:**
1. Abrir la URL con Chrome.
2. Obtener el **código fuente HTML completo** de la página.
3. Buscar el atributo `data-schema-type` en el HTML. Puede aparecer de **dos formas distintas**:

---

**FORMA 1 — El JSON está en el contenido del `<script>`:**
```html
<script type="application/ld+json" data-schema-type="program">
  { "@context": "https://schema.org", ... }
</script>
```
- El atributo `data-schema-type` identifica el elemento pero el **JSON está dentro del tag `<script>`** (como innerHTML).
- Extraer el **contenido interno del `<script>`** que tenga el atributo `data-schema-type`.

**FORMA 2 — El JSON está directamente en el valor del atributo:**
```html
<div data-schema-type='{"@context":"https://schema.org",...}'></div>
```
- El JSON es el **valor del atributo** `data-schema-type` directamente.
- Extraer el valor del atributo.

---

4. Aplicar la lógica de extracción según la forma detectada:

| Condición                                                                 | Resultado                                           |
|---------------------------------------------------------------------------|-----------------------------------------------------|
| `data-schema-type` no existe en ningún elemento del HTML                  | `JSON_WEB = null`                                   |
| Existe en `<script>` pero el contenido interno está vacío                 | `JSON_WEB = ""`                                     |
| Existe en `<script>` y tiene JSON en su contenido interno                 | Extraer innerHTML del `<script>` → `JSON_WEB`       |
| Existe como atributo en otro elemento y su valor tiene contenido          | Extraer valor del atributo → `JSON_WEB`             |
| Existe pero está vacío en ambas formas                                    | `JSON_WEB = ""`                                     |

5. Parsear `JSON_WEB` como JSON y formatear con indentación de 2 espacios (pretty print).
   Registrar internamente el tipo de ubicación detectado como `SCHEMA_UBICACION`:
   - `"<script>"` si el JSON vino del contenido interno de un tag `<script data-schema-type>`.
   - `"Atributo"` si el JSON vino del valor directo del atributo `data-schema-type`.
   - `"No encontrado"` si el atributo no existe o está vacío.
   Este valor se usará en el informe para identificar la forma en que está implementado el schema.
6. Guardar el JSON en un archivo `.json` dentro de `output/schemas/`:
   - **Nombre del archivo:** tomar la URL de la columna A y reemplazar `:` `/` `\` `.` por guion bajo `_`, con extensión `.json`.
   - Ejemplo: `https://example.com/pagina` → `output/schemas/https___example_com_pagina.json`
   - El mismo nombre que el archivo drive permite emparejar ambos archivos por fila fácilmente.

---

### PASO 4 — Verificar existencia y contenido de `data-schema-type` → escribir en columna C

**Condición:** ¿El atributo `data-schema-type` existe en el HTML Y contiene un valor no vacío?

| Resultado                   | Valor a escribir en **columna C** "data-schema-type" del Excel |
|-----------------------------|----------------------------------------------------------------|
| Sí existe y tiene contenido | `SI`                                                           |
| No existe o está vacío      | `NO`                                                           |

→ Escribir en **columna C** de la fila actual del Excel de origen.

---

### PASO 5 — Verificar que `data-schema-type` esté dentro del `<head>` → escribir en columna D

**Proceso:**
1. Usar el HTML completo del PASO 3.
2. Localizar el bloque `<head> ... </head>`.
3. Buscar dentro de ese bloque el atributo `data-schema-type` en **cualquiera de sus dos formas**:
   - **Forma 1:** `<script ... data-schema-type="...">` dentro del `<head>`.
   - **Forma 2:** cualquier elemento con `data-schema-type="..."` dentro del `<head>`.
4. Si el elemento con `data-schema-type` se encuentra **dentro del `<head>`** → `SI`.
5. Si está fuera del `<head>` (por ejemplo en el `<body>`) o no existe → `NO`.

| Condición                                                                 | Valor en **columna D** "Schema en el head" del Excel |
|---------------------------------------------------------------------------|------------------------------------------------------|
| El elemento con `data-schema-type` está dentro de `<head>`                | `SI`                                                 |
| El elemento con `data-schema-type` está fuera de `<head>` o no existe     | `NO`                                                 |

→ Escribir en **columna D** de la fila actual del Excel de origen.

---

### PASO 6 — Comparar JSON_DRIVE con JSON_WEB → escribir en columna E

**Regla principal:**
> `JSON_WEB` (de la página) debe **contener** o ser **igual** a `JSON_DRIVE` (del Drive).
> Esto significa: cada clave y valor de `JSON_DRIVE` debe existir en `JSON_WEB` con el mismo valor exacto.
> `JSON_WEB` puede tener campos adicionales — eso es válido.
> Lo que NO es válido: que falte algún campo de `JSON_DRIVE` en `JSON_WEB`, o que el valor difiera.

**Proceso:**
1. Tomar `JSON_DRIVE` (PASO 2) y `JSON_WEB` (PASO 3).
2. Recorrer **cada clave y valor** de `JSON_DRIVE` (incluyendo objetos anidados).
3. Para cada campo de `JSON_DRIVE`, buscar la misma clave en `JSON_WEB` y comparar el valor.
4. Si todos los campos de `JSON_DRIVE` están en `JSON_WEB` con los mismos valores → `Aprobado`.
5. Si falta algún campo o el valor es diferente → `No Aprobado`.

| Condición                                                          | Valor en **columna E** "Comparacion de Schema" del Excel |
|--------------------------------------------------------------------|----------------------------------------------------------|
| `JSON_WEB` contiene todos los campos/valores de `JSON_DRIVE`       | `Aprobado`                                               |
| `JSON_WEB` es exactamente igual a `JSON_DRIVE`                     | `Aprobado`                                               |
| Algún campo de `JSON_DRIVE` falta o difiere en `JSON_WEB`          | `No Aprobado`                                            |
| `JSON_WEB` es null o vacío                                         | `No Aprobado`                                            |
| `JSON_DRIVE` es null o vacío                                       | `No Aprobado`                                            |

→ Escribir en **columna E** de la fila actual del Excel de origen.

---

### PASO 7 — Prueba Rich Results de Google → escribir en columna F

**Proceso:**
1. Tomar la URL de la **columna A** de la fila actual.
2. Construir el enlace de prueba:
   ```
   https://search.google.com/test/rich-results/result?id=T-gaHqOpjtwl-glv3aocFQ&url={URL_COLUMNA_A}
   ```
3. Abrir ese enlace en el navegador.
4. Buscar si aparece **"Elementos válidos detectados"** u otro indicador de aprobación.

| Condición                        | Valor en **columna F** "Prueba de texto enriquecido" del Excel |
|----------------------------------|----------------------------------------------------------------|
| Se detectan elementos válidos    | `Aprobado`                                                     |
| No se detectan elementos válidos | `No Aprobado`                                                  |

→ Escribir en **columna F** de la fila actual del Excel de origen.

---

### PASO 8 — Captura de evidencia visual

Para cada fila, tomar capturas de pantalla de:

1. La página web analizada (columna A).
2. La sección del código HTML donde aparece `data-schema-type`.
3. El resultado del test Rich Results de Google.

**Nombre de cada captura:** usar el mismo formato que los archivos JSON (URL de columna A con `:` `/` `\` `.` reemplazados por `_`), añadiendo un sufijo que identifica qué captura es, con extensión `.png`:

| Captura                         | Nombre del archivo                                        |
|---------------------------------|-----------------------------------------------------------|
| Página web                      | `https___example_com_pagina_web.png`                      |
| Sección HTML con data-schema-type | `https___example_com_pagina_schema.png`                 |
| Resultado Rich Results          | `https___example_com_pagina_richresults.png`              |

Guardar las tres capturas en `output/capturas/`.

---

### PASO 9 — Manejo de errores

Si ocurre alguno de estos eventos:
- URL inaccesible o no carga
- JSON inválido o malformado
- Timeout o página sin respuesta
- Atributo `data-schema-type` no encontrado
- Documento Google Drive inaccesible

→ Registrar en `output/error_log.txt`: número de fila, columna afectada, URL, paso fallido, tipo de error y descripción.
→ **Continuar con la siguiente fila** sin detener el proceso.

---

### PASO 10 — Guardar Excel de origen y avanzar a la siguiente fila

Al finalizar cada fila:
1. Confirmar que se escribieron valores en columnas C, D, E y F.
2. Guardar el archivo Excel de origen con los cambios (sobrescribir).
3. Pasar a la siguiente fila.

---

## Generar informe final en Excel

Al terminar todas las filas, crear el archivo: **`output/reporte_validacion_schema.xlsx`**

Este archivo es **adicional y separado** del Excel de origen, guardado dentro de `output/`. Tiene 4 hojas:

---

### Hoja 1 — "Resumen"

| Campo                        | Valor            |
|------------------------------|------------------|
| Fecha y hora de ejecución           | DD/MM/YYYY HH:MM |
| Total filas procesadas              | N                |
| Total Aprobados (col E)             | N                |
| Total No Aprobados (col E)          | N                |
| Total con schema en HEAD (col D)    | N                |
| Total Rich Results Aprobados        | N                |
| Total schema en `<script>`          | N                |
| Total schema en atributo            | N                |
| Total sin schema                    | N                |
| Total errores registrados           | N                |

Formato: encabezados negrita, fondo `#1F3864`, texto blanco, bordes en todas las celdas.

---

### Hoja 2 — "Resultados"

Una fila por URL procesada:

| # Fila | URL Página (A) | URL Drive (B) | Schema existe (C) | Ubicación schema | En HEAD (D) | JSON igual (E) | Rich Results (F) | Observaciones |
|--------|----------------|---------------|-------------------|------------------|-------------|----------------|------------------|---------------|

**Columna "Ubicación schema":** indica dónde está el contenido del `data-schema-type`:
- `<script>` → el JSON está dentro del tag `<script type="application/ld+json" data-schema-type="...">`.
- `Atributo` → el JSON está directamente en el valor del atributo `data-schema-type="..."`.
- `No encontrado` → el atributo `data-schema-type` no existe en la página.

**Formato condicional:**
- `Aprobado` / `SI` → fondo verde `#C6EFCE`, texto `#276221`.
- `No Aprobado` / `NO` → fondo rojo `#FFC7CE`, texto `#9C0006`.
- `<script>` → fondo azul claro `#DDEBF7`, texto azul oscuro `#1F4E79`.
- `Atributo` → fondo morado claro `#E2CFED`, texto morado oscuro `#4B0082`.
- `No encontrado` → fondo gris `#F2F2F2`, texto gris oscuro `#595959`.
- Columnas de URL → hipervínculo clickeable.
- Ajustar ancho de columnas automáticamente.

---

### Hoja 3 — "Errores"

| # Fila | URL | Paso fallido | Tipo de error | Descripción |
|--------|-----|--------------|---------------|-------------|

Encabezados con fondo naranja `#F4B942`. Si no hay errores, escribir "Sin errores encontrados" en A2.

---

### Hoja 4 — "Evidencias"

Una sección por fila procesada:
- Título: número de fila + URL.
- **Indicador de ubicación del schema:** etiquetar claramente si el `data-schema-type` fue encontrado en `<script>` o en atributo directo, o si no se encontró.
- Capturas incrustadas: página web, sección HTML con `data-schema-type`, resultado Rich Results.
- Fragmento del HTML donde aparece `data-schema-type` mostrando el tag completo (con su forma: `<script>` o atributo).
- Fragmentos JSON de Drive (`output/drive/`) y de la web (`output/schemas/`) en fuente `Courier New`.
- Imágenes redimensionadas a máximo 600px de ancho sin distorsión.

---

## Reglas de ejecución

- Procesar **todas** las filas sin excepción.
- **No detener** el proceso ante fallos — registrar en `output/error_log.txt` y continuar.
- **Sobrescribir el Excel de origen** tras procesar cada fila (C, D, E, F).
- Guardar **todos los archivos generados** (`.json`, capturas, informe, log) dentro de `output/`.
- El informe Excel debe tener formato visual con colores, bordes y columnas ajustadas.

---

## Resultado final esperado

Al terminar deben existir:

1. **Excel de origen sobrescrito** (`Schemas.xlsx`) con columnas C, D, E y F completadas por fila.
2. **`output/`** — carpeta creada junto al Excel con todo lo generado:
   - `output/schemas/*.json` — un archivo `.json` por fila con el JSON del `data-schema-type` extraído de la página web.
   - `output/drive/*.json` — un archivo `.json` por fila con el JSON extraído y formateado del documento Drive.
   - `output/capturas/https___example_com_pagina_web.png`, `https___example_com_pagina_schema.png`, `https___example_com_pagina_richresults.png` — mismo formato de nombre que los JSON, con sufijo por tipo de captura.
   - `output/reporte_validacion_schema.xlsx` — informe con 4 hojas: Resumen, Resultados, Errores y Evidencias.
   - `output/error_log.txt` — registro de todos los errores encontrados.
3. **`schemas_automation.py`** — script Python en la raíz del Excel para ejecuciones futuras sin IA.

---

## Mantenimiento del script de automatización

Si el usuario indica que algo cambió (nueva columna, lógica o formato) y el script ya existe:
1. Leer `schemas_automation.py`.
2. Aplicar solo los cambios necesarios.
3. Guardar y avisar al usuario.

Si el usuario dice **"regenerar script"** o **"recrear automatización"**:
- Borrar `schemas_automation.py` existente.
- Ejecutar en MODO B completo (IA ejecuta y regenera el script).

Si el usuario dice **"forzar IA"** o **"ejecutar con IA"**:
- Ignorar el script aunque exista.
- Ejecutar en MODO B sin sobreescribir el script existente a menos que el usuario lo confirme.

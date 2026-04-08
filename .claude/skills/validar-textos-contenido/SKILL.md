---
name: validar-textos-contenido
description: >
  Usa esta skill siempre que el usuario necesite validar textos a partir de un archivo (Excel, Word, PDF, Imágenes, Etc.) de referencia
  y contenido según diseño Figma e imágenes.
  Actívala ante solicitudes como: "validar textos", "revisar contenido", "validar landing",
  "comparar textos con diseño", "verificar copy", "QA de textos", "validar contenido vs anexos".
---

# Skill: Validar Textos y Contenido Web

## Paso 1 — Solicitar información al usuario

Antes de iniciar, confirmar con el usuario:

1. **URL a validar** — Si no la proporcionó, solicitarla:
   > "Para iniciar la validación necesito la URL de la página web o landing a revisar. Por favor compártela."

2. **Archivo de referencia** — Verificar si existe en `C:\Users\jesus\Desktop\Ilumno\validar-textos-contenido\anexos\`.
   Si no existe, pedirlo al usuario.

3. **Anexos adicionales** — Confirmar si hay imágenes, PDFs, Word u otros archivos en `anexos\`.

---

## Paso 2 — Leer archivos de referencia

- Leer el archivo de referencia en `C:\Users\jesus\Desktop\Ilumno\validar-textos-contenido\anexos\`
- Leer todos los demás archivos en `anexos\`: PDFs, imágenes Figma, Word, capturas, etc.
- Organizar los textos de referencia por sección o componente cuando sea posible.

---

## Paso 3 — Ejecutar el script Python (multi-dispositivo)

Ejecutar el script ubicado en:
`C:\Users\jesus\Desktop\Ilumno\validar-textos-contenido\validar_textos.py`

```bash
# Activar venv
"C:\Users\jesus\Desktop\Ilumno\Ley no moslestar\.venv\Scripts\activate"

# Ejecutar validación
python "C:\Users\jesus\Desktop\Ilumno\validar-textos-contenido\validar_textos.py" --url "URL_A_VALIDAR"
```

El script realiza automáticamente la validación en **3 dispositivos**:

| Dispositivo | Resolución | User Agent |
|-------------|-----------|------------|
| Desktop | 1280 x 900 | Chrome Windows |
| Android (Pixel 5) | 393 x 851 | Chrome Mobile Android 13 |
| iOS (iPhone 14) | 390 x 844 | Safari iOS 17 |

Para cada dispositivo:
- Navega la URL completa (con scroll para cargar lazy content)
- Extrae todos los textos visibles del DOM
- Toma captura de pantalla full page
- Compara textos contra el archivo de referencia/anexos

Outputs generados en `generados/`:
- `reporte_validacion_YYYYMMDD_HHMMSS.xlsx` — Reporte Excel con 5 hojas
- `screenshots/screenshot_*_Desktop_*.png`
- `screenshots/screenshot_*_Android_*.png`
- `screenshots/screenshot_*_iOS_*.png`
- `log_validacion_*.txt`

---

## Paso 4 — Validación visual complementaria

Revisar manualmente las capturas generadas comparando con el diseño Figma/imágenes adjuntas:

- Jerarquía visual y layout por dispositivo
- Tipografías, colores y espaciados
- Textos en estados especiales: hover, error, vacío
- Comportamiento responsive entre vistas

---

## Paso 5 — Revisar y presentar el reporte

El reporte Excel contiene 5 hojas:

| Hoja | Contenido |
|------|-----------|
| Resumen General | Totales APROBADO/FALLIDO/ADVERTENCIA y % aprobación |
| Hallazgos | Detalle de cada texto por dispositivo con evidencia |
| Por Dispositivo | Estadísticas comparativas Desktop vs Android vs iOS |
| Fallidos | Solo los defectos para reportar al equipo de desarrollo |
| Capturas | Lista y rutas de todos los screenshots generados |

---

## Estados de hallazgos

| Estado | Significado |
|--------|-------------|
| APROBADO | Texto exactamente igual al de referencia |
| ADVERTENCIA | Similitud parcial o texto extra no esperado |
| FALLIDO | Texto de referencia no encontrado en la página |

---

## Instalación de dependencias (primera vez)

```bash
pip install playwright openpyxl pandas Pillow python-docx PyMuPDF
playwright install chromium
```

Ver manual completo en:
`C:\Users\jesus\Desktop\Ilumno\validar-textos-contenido\MANUAL_USUARIO.md`

---
name: ejecutar-casos-de-prueba
description: >
  Usa esta skill siempre que el usuario necesite ejecutar casos de prueba QA a partir de un
  archivo Excel de referencia, un conjunto de reglas de negocio y una solicitud de desarrollo. El
  agente generará un nuevo archivo Excel con casos de prueba detallados, paso a paso, para validar la calidad de una entrega (landing, formulario, módulo, integración CRM).
  Este skill es ideal para especialistas QA que necesitan una guía estructurada y completa para revisar la calidad de lo desarrollado por el equipo técnico, asegurando que se cumplan los criterios de aceptación y las reglas de negocio establecidas.
  Debe de generar un excel con el resumen de los casos de prueba, con pasos detallados, URLs específicas, y un formato visual profesional que facilite su ejecución por parte del equipo QA.
---
# Skill: ejecutar-casos-de-prueba

## Objetivo

Ejecutar casos de prueba QA de forma estructurada a partir de un archivo Excel, apoyándose en información adicional de la carpeta `anexos`, y guardar todos los artefactos generados en la carpeta `generados`.

---

## Paso 1 — Leer el archivo Excel de casos de prueba

1. Busca en el directorio de trabajo el archivo Excel con los casos de prueba (extensión `.xlsx` o `.xls`).
2. Lee todas las hojas del archivo e identifica:
   - ID del caso de prueba
   - Descripción / título
   - Precondiciones
   - Pasos de ejecución
   - Resultado esperado
   - Estado inicial (si existe)

---

## Paso 2 — Leer la carpeta `anexos`

1. Lista todos los archivos dentro de la carpeta `anexos/` del directorio de trabajo.
2. Lee y procesa cada archivo encontrado:
   - Documentos de texto (`.txt`, `.md`): léelos directamente para extraer reglas de negocio, criterios de aceptación, contexto adicional, URLs de ambientes, credenciales de prueba, etc.
   - Imágenes (`.png`, `.jpg`, `.gif`): obsérvalas para entender flujos visuales, wireframes o evidencias de referencia.
   - Archivos Excel (`.xlsx`, `.xls`): léelos para extraer datos de prueba, tablas de configuración, etc.
   - PDFs: léelos para extraer especificaciones o documentación.
3. Consolida toda la información de `anexos/` como contexto de ejecución antes de iniciar las pruebas.

> **Nota:** Si la carpeta `anexos/` no existe o está vacía, continúa con la información disponible en el Excel y en el mensaje del usuario.

---

## Paso 3 — Ejecutar cada caso de prueba

Para cada caso de prueba en el Excel:

1. Lee el ID, descripción, precondiciones y pasos.
2. Usa el contexto de `anexos/` para completar datos faltantes (URLs, credenciales, reglas especiales).
3. Ejecuta los pasos uno a uno de forma lógica y documenta:
   - **Resultado obtenido:** lo que realmente sucede al seguir el paso.
   - **Estado:** `APROBADO` si el resultado obtenido coincide con el esperado, `FALLIDO` si hay discrepancia, `BLOQUEADO` si no se puede ejecutar.
   - **Observaciones:** detalles relevantes, desviaciones, comportamientos inesperados.
   - **Evidencias:** URLs visitadas, capturas de pantalla (si aplica), fragmentos de respuesta.

---

## Paso 4 — Guardar artefactos en la carpeta `generados`

**Todos los archivos producidos durante la ejecución deben guardarse en la carpeta `generados/`.**

1. Si la carpeta `generados/` no existe, créala.
2. Guarda allí:
   - El informe de ejecución final (Excel `.xlsx` o Markdown `.md`).
   - Cualquier archivo adicional generado (logs, capturas exportadas, reportes intermedios).
3. Nombra los archivos con el patrón: `ejecucion_<nombre-del-caso-o-lote>_<YYYY-MM-DD>.xlsx` (o la extensión correspondiente).

---

## Paso 5 — Generar el informe de ejecución

Crea un archivo Excel de resumen en `generados/` con las siguientes columnas:

| ID | Título | Precondiciones | Pasos ejecutados | Resultado esperado | Resultado obtenido | Estado | Evidencias | Observaciones |
|----|--------|---------------|-----------------|-------------------|-------------------|--------|------------|---------------|

Adicionalmente incluye una hoja o sección de resumen ejecutivo con:
- Total de casos ejecutados
- Total APROBADOS / FALLIDOS / BLOQUEADOS
- Porcentaje de cobertura aprobada
- Fecha de ejecución
- Observaciones generales

---

## Reglas generales

- Nunca omitas un caso de prueba del Excel sin ejecutarlo o justificarlo.
- Si un paso es ambiguo, usa el contexto de `anexos/` para inferir el comportamiento correcto.
- Sé preciso en el estado: no marques como APROBADO si hay dudas; usa BLOQUEADO y documenta el motivo.
- Todos los archivos de salida van **siempre** en `generados/`, nunca en la raíz ni en `anexos/`.

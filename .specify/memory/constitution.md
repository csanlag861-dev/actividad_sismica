<!--
Sync Impact Report
Version change: no existía → 1.0.0 (constitución inicial)
Modified principles: ninguno; se crea la primera versión.
Added principles:
  - I. Requisitos del enunciado y rúbrica
  - II. Trazabilidad y tratamiento fiel de los datos
  - III. Ingesta persistente e idempotente
  - IV. Análisis comparable y evidencia verificable
  - V. Decisiones explícitas y alcance proporcionado
Added sections:
  - Clasificación del alcance
  - Fuentes y límites de aplicación
Removed sections: ninguna.
Follow-up TODO: confirmar la fecha original de ratificación.
-->

# Constitución del Observatorio de actividad sísmica

## Principios fundamentales

### I. Requisitos del enunciado y la rúbrica
El proyecto DEBE cumplir los requisitos identificados como obligatorios en el enunciado y su
rúbrica. En particular, el backend requerido —productor, Kafka, consumidor, PostgreSQL y API
FastAPI— DEBE ejecutarse como cinco servicios de Docker Compose; el panel DEBE usar Angular y
Leaflet, consultar la API y presentar el análisis exigido. La guía y las FAQ aclaran la entrega,
pero declaran que no añaden tareas ni modifican la rúbrica. Así se evalúa el trabajo pedido, no una
interpretación ampliada de ejemplos o sugerencias.

### II. Trazabilidad y tratamiento fiel de los datos
El sistema y el análisis DEBEN identificar las fuentes, consultas, periodos, filtros, fechas de
extracción, unidades y transformaciones relevantes, y conservar los originales conforme al alcance
documentado. Los datos ausentes DEBEN seguir distinguiéndose de cero; los registros rechazados
DEBEN conservarse con su motivo y referencia al original. La asignación geográfica DEBE basarse en
el catálogo y sus límites conocidos: los eventos sin país no se atribuyen al país más cercano.
Estas reglas hacen que los resultados puedan revisarse sin ocultar incertidumbre ni alterar el
significado de la fuente.

### III. Ingesta persistente e idempotente
La identidad del evento y su versión DEBEN distinguirse: una repetición de la misma versión no
puede incrementar el recuento y una versión antigua no puede reemplazar una más reciente. El estado
necesario para reanudar el procesamiento DEBE persistir; los cambios en PostgreSQL DEBEN guardarse
antes de confirmar el offset de Kafka. La reproducción histórica DEBE quedar aislada de los datos
actuales o usar un mecanismo equivalente y DEBE identificarse como reproducción. Esto evita
duplicados, pérdida silenciosa de datos y confusión entre datos históricos y actuales.

### IV. Análisis comparable y evidencia verificable
Los filtros aplicados al mapa, la tabla y las estadísticas DEBEN ser coherentes. Las comparaciones
históricas DEBEN usar periodos y filtros equivalentes, indicar denominadores y limitaciones, y no
presentarse como predicciones sísmicas. La entrega DEBE incluir las conclusiones, pruebas,
mediciones y análisis que exige el enunciado; una prueba se considera evidencia solo después de
ejecutarse y registrar su resultado observado. Esto permite distinguir resultados medidos de
suposiciones y conclusiones de afirmaciones no respaldadas.

### V. Decisiones explícitas y alcance proporcionado
Las decisiones propias —por ejemplo, el alcance geográfico o el modelo de datos— DEBEN documentarse
y justificarse dentro de las alternativas permitidas por el enunciado. Las propuestas de mejora y
ampliaciones opcionales DEBEN identificarse como tales, nunca como requisitos del profesor. No se
exige añadir herramientas o infraestructura que las fuentes declaran innecesarias; la complejidad
añadida DEBE corresponder a una necesidad explicada. Así se conserva la autoría de las decisiones
del estudiante sin convertir ejemplos u opciones en obligaciones.

## Clasificación del alcance

- **Requisitos del profesor:** son los puntos que el enunciado y la rúbrica presentan como
  obligatorios. El contrato de API define la respuesta mínima que debe implementar el backend.
- **Decisiones del estudiante:** son elecciones de implementación o de alcance que las fuentes
  dejan abiertas. Deben respetar los requisitos, quedar explicadas y, cuando afecten a resultados,
  mantenerse coherentes entre extracción, API, panel y análisis.
- **Propuestas:** son mejoras o alternativas expresamente opcionales. No se contabilizan como
  requisitos ni sustituyen las evidencias obligatorias.

En particular, la generación automática de informes de calidad y un panel de calidad son
opcionales; el análisis documentado en una memoria o notebook y la conservación de rechazos con su
motivo sí forman parte del trabajo descrito en las fuentes.

## Fuentes y límites de aplicación

Esta constitución se basa en los documentos de [docs/source-of-truth](../../docs/source-of-truth):
[ENUNCIADO.md](../../docs/source-of-truth/ENUNCIADO.md) y su rúbrica establecen las tareas y
criterios; [CONTRATO-API.md](../../docs/source-of-truth/CONTRATO-API.md) define el contrato mínimo;
[Guia-entregables-observatorio-sismico.md](../../docs/source-of-truth/Guia-entregables-observatorio-sismico.md)
y [FAQ-ALUMNADO.md](../../docs/source-of-truth/FAQ-ALUMNADO.md) explican cómo presentar y entender
el trabajo sin añadir requisitos; [FUENTES-Y-LICENCIAS.md](../../docs/source-of-truth/FUENTES-Y-LICENCIAS.md)
documenta procedencia y licencias; [README.md](../../docs/source-of-truth/README.md) y
[VALIDACION.md](../../docs/source-of-truth/VALIDACION.md) describen el material de apoyo y el
alcance de sus comprobaciones.

El mapa y los eventos ficticios entregados sirven para probar la demostración, no para sustituir
el análisis con datos reales. Las fuentes tampoco exigen disponibilidad permanente, automatización
de informes ni servicios adicionales como Spark, Flink o Kubernetes. Las cifras ilustrativas de
las guías no son resultados del proyecto.

## Gobernanza

Para modificar esta constitución, se DEBE indicar la razón del cambio y si procede de un cambio
explícito en las fuentes del profesor, de una decisión del estudiante o de una propuesta. La
modificación DEBE conservar esa distinción y no convertir una propuesta en requisito. Las fuentes
no establecen un proceso de aprobación adicional; esta constitución no lo añade.

La versión sigue semver: MAJOR para retirar o redefinir de forma incompatible un principio, MINOR
para añadir principios o ampliar materialmente su alcance y PATCH para aclaraciones editoriales
sin cambio de significado.

Al revisar el cumplimiento, se contrastará cada requisito obligatorio con el enunciado, el
contrato aplicable y evidencia observada. Las decisiones se revisarán frente a su justificación;
las propuestas opcionales no serán condiciones de cumplimiento. No se impone una herramienta de
pruebas específica.

**Versión**: 1.0.0 | **Ratificada**: TODO(RATIFICATION_DATE): confirmar la fecha original de adopción | **Última enmienda**: 2026-11-03

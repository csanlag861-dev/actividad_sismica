# Requisitos y aceptación

**Estado global:** borrador inicial para revisión del estudiante. Los criterios descritos son expectativas verificables, no evidencia de implementación ni de pruebas superadas. Los identificadores son provisionales y estables en estos documentos.

**Convenciones:** «Obligatorio» se usa cuando el enunciado o el contrato lo exigen; «Opcional/recomendado» cuando las fuentes lo marcan como alternativa o ampliación; «No especificado» cuando no hay una decisión o nivel de obligatoriedad inequívoco. Las aclaraciones del estudiante se identifican y no se atribuyen al profesor.

## Funcionales

| ID | Descripción verificable | Fuente y sección | Carácter | Dependencias | Aceptación / método | Estado |
|---|---|---|---|---|---|---|
| REQ-F-001 | El productor consulta por HTTP GET el feed USGS `all_day.geojson` aproximadamente cada 60 s; trata cada respuesta como ventana móvil de últimas 24 h, no como push ni día natural. | [ENUNCIADO](../source-of-truth/ENUNCIADO.md), §4.1; §5 | Obligatorio | REQ-TEC-001 | Inspeccionar configuración/logs y comparar consultas separadas; registrar frecuencia y ventana asumida. | Listo para planificar |
| REQ-F-002 | El análisis live acepta solo `properties.type == "earthquake"` y registra el motivo de exclusión de otros tipos. | ENUNCIADO, §4.1; §6 | Obligatorio | REQ-F-001, REQ-QUAL-001 | Inyectar un tipo distinto y verificar motivo conservado y ausencia en el análisis sísmico. | Listo para planificar |
| REQ-F-003 | El productor detecta eventos nuevos y revisiones posteriores mediante `id` y `updated`, evita republicar versiones conocidas y espera confirmación Kafka antes de marcar una versión como publicada. | ENUNCIADO, §5 | Obligatorio | REQ-ARCH-001 | Probar evento nuevo, duplicado, revisión y error entre publicación/registro; revisar estado persistido. | Listo para planificar |
| REQ-F-004 | El histórico se descarga del catálogo USGS como año completo; se registran consultas, fecha y número de registros, se subdividen consultas que excedan 20.000 y se deduplican límites de intervalos. | ENUNCIADO, §4.2 | Obligatorio (año completo); ver filtros de trabajo en REQ-F-005 | REQ-TEC-001 | Reproducir el registro de extracción y comprobar cobertura temporal, consultas, conteos y deduplicación. | Listo para planificar |
| REQ-F-005 | El subconjunto histórico de trabajo y comparación utiliza 2025, ámbito mundial, magnitud ≥4 y tipo `earthquake`, según la propuesta del profesor confirmada por el estudiante. La comparación con el periodo reciente conserva filtros equivalentes; la media diaria histórica divide por todos los días de 2025, incluidos los días con cero eventos. Estos filtros no se aplican a la ingesta live. | ENUNCIADO, §§4.2 y 8; aclaración confirmada del estudiante en [00](00-principios-y-alcance.md#supuestos-documentales-y-preguntas-abiertas), DEC-P-05 | Aclaración confirmada del estudiante sobre propuesta del profesor; reglas de comparación también enunciado | REQ-F-004 | Inspeccionar parámetros/registro histórico; verificar que live no hereda `min_mag=4` ni límite geográfico histórico; comparar periodos con filtros equivalentes y comprobar que el denominador de la media diaria incluye todos los días históricos, incluso los de cero eventos. | Alcance aclarado; listo para planificar |
| REQ-F-006 | El consumidor valida, transforma e integra los eventos y conserva en cuarentena los registros no tratables con original y motivo duradero. | ENUNCIADO, §6; §7 | Obligatorio | REQ-ARCH-001 | Inyectar un caso inválido controlado; comprobar motivo/original en almacenamiento y que no se convierta en punto inventado. | Listo para planificar |
| REQ-F-007 | Las revisiones actualizan el estado actual solo si son más recientes; los eventos fuera de la ventana live no se borran por esa sola ausencia. | ENUNCIADO, §§5 y 7; FAQ, preguntas 12 y 15 | Obligatorio | REQ-F-003, REQ-DATA-002 | Ejecutar secuencia de versión nueva, repetida, revisión posterior y versión antigua; confirmar estado final e historial. | Listo para planificar |
| REQ-F-008 | El panel incluye mapa interactivo, fronteras, círculos, ficha, tamaño por magnitud, color por profundidad, leyenda y tratamiento visible de nulos. | ENUNCIADO, §8 | Obligatorio | REQ-API-001, REQ-DATA-003 | Prueba manual con datos reales y campos nulos; comprobar leyenda, ficha y correspondencia de variables visuales. | Listo para planificar |
| REQ-F-009 | El panel presenta recuentos por país o subregión y ventanas fijas de una hora UTC basadas en tiempo de evento, distribuciones de magnitud/profundidad con nulos contabilizados y tabla consultable. | ENUNCIADO, §8 | Obligatorio | REQ-API-002, REQ-DATA-002, REQ-DATA-003 | Comparar salidas con consultas independientes sobre registros controlados; comprobar categoría sin país y nulos. | Listo para planificar |
| REQ-F-010 | Los filtros de fecha, magnitud y país/región se aplican coherentemente al mapa y análisis; la comparación usa filtros equivalentes entre periodo reciente e histórico, informa los denominadores y considera días históricos sin eventos. | ENUNCIADO, §8; CONTRATO-API, «GET /api/eventos»; aclaración confirmada en DEC-P-05 | Obligatorio | REQ-F-005, REQ-API-001, REQ-API-002, REQ-DATA-004 | Consultar los mismos filtros por mapa/estadísticas; contrastar conteos y denominador histórico, incluidos días con cero, y revisar que el periodo reciente aplica los mismos filtros pertinentes. | Pendiente de especificación de estadísticas |
| REQ-F-011 | El panel redacta al menos tres conclusiones fundamentadas, con cifras, filtros y periodo; no las presenta como predicción o evaluación de riesgo. | ENUNCIADO, §8; §12 | Obligatorio | REQ-F-009, REQ-F-010, REQ-DOC-003 | Revisar memoria/notebook y panel: cada conclusión identifica datos, periodo, filtros y denominador. | Pendiente de resultados reales |
| REQ-F-012 | El panel muestra último acceso correcto, última ingesta USGS, periodo consultado y aviso de datos desactualizados o backend no disponible; conserva/identifica última lectura ante fallo y recupera consultas al volver el túnel. | ENUNCIADO, §8; CONTRATO-API, «GET /api/salud» y «Acceso desde Netlify» | Obligatorio; esquema de salud pendiente | REQ-API-003, REQ-DEP-003 | Simular cierre y reapertura de API/túnel; comprobar aviso, distinción de tiempos y recuperación automática. | Pendiente de especificación de `/api/salud` |
| REQ-F-013 | Se realiza replay acelerado de selección histórica por Kafka, conservando tiempo de evento y añadiendo tiempo de reproducción; queda separado de live, identificado «REPRODUCCIÓN» y contempla duplicado/revisión. | ENUNCIADO, §11 | Obligatorio | REQ-ARCH-001, REQ-DATA-005 | Ejecutar replay controlado; verificar aislamiento, etiqueta, fechas originales y casos de revisión/duplicado. | Listo para planificar |

## No funcionales

| ID | Descripción verificable | Fuente y sección | Carácter | Dependencias | Aceptación / método | Estado |
|---|---|---|---|---|---|---|
| REQ-NF-001 | Los errores HTTP 429/5xx, timeout y JSON inválido se registran y manejan con reintentos de espera progresiva; un fallo no borra el estado válido anterior. | ENUNCIADO, §5 | Obligatorio | REQ-F-001, REQ-QUAL-001 | Simular errores controlados y revisar reintentos, logs y conservación del último estado válido. | Listo para planificar |
| REQ-NF-002 | El procesamiento es al menos una vez e idempotente; transacción de escritura precede a confirmación del offset. No se exige transacción distribuida. | ENUNCIADO, §7; FAQ, pregunta 13 | Obligatorio | REQ-F-006, REQ-F-007 | Interrumpir/reprocesar antes y después de persistencia; comprobar que no hay duplicación y offsets reanudan. | Listo para planificar |
| REQ-NF-003 | Si falla la base, `/api/estadisticas` devuelve un error explícito y no presenta cero eventos como resultado válido. | CONTRATO-API, «Otros endpoints a implementar» | Obligatorio | REQ-API-002 | Desconectar DB y comprobar respuesta de error explícita, distinta de un resultado estadístico cero. | Pendiente de especificación parcial |
| REQ-NF-004 | No se incluyen credenciales en `config.json`, frontend o Git; el token de ngrok queda fuera del repositorio. | CONTRATO-API, «Acceso desde Netlify»; ENUNCIADO, §9 | Obligatorio | REQ-DEP-002 | Inspección de archivos/configuración entregados y búsqueda de secretos antes de publicar. | Listo para planificar |

## Arquitectónicos

| ID | Descripción verificable | Fuente y sección | Carácter | Dependencias | Aceptación / método | Estado |
|---|---|---|---|---|---|---|
| REQ-ARCH-001 | Ejecutar productor Python, Kafka, consumidor Python, PostgreSQL y API FastAPI como cinco servicios separados de Docker Compose en el equipo del alumno; un comando `docker compose up --build -d` los levanta conjuntamente. | ENUNCIADO, §2; §12 | Obligatorio | REQ-TEC-001 | Levantar desde configuración reproducible y comprobar cinco servicios separados y conectividad. | Listo para planificar |
| REQ-ARCH-002 | El productor y consumidor continúan su trabajo sin que el panel esté abierto; Kafka y PostgreSQL no se exponen por ngrok. | ENUNCIADO, §§2 y 9 | Obligatorio | REQ-ARCH-001, REQ-DEP-003 | Mantener cerrado el panel y comprobar ingesta; inspeccionar exposición del túnel. | Listo para planificar |
| REQ-ARCH-003 | Se configuran comprobaciones de salud y reintentos para dependencias, sin asumir que el orden de arranque implica disponibilidad. | ENUNCIADO, §2; README del material base, §3 | Obligatorio | REQ-ARCH-001 | Arrancar servicios con dependencias retrasadas/no disponibles y comprobar espera/reintento y recuperación. | Listo para planificar |

## Tecnológicos

| ID | Descripción verificable | Fuente y sección | Carácter | Dependencias | Aceptación / método | Estado |
|---|---|---|---|---|---|---|
| REQ-TEC-001 | Usar Python + aiokafka para productor/consumidor, Apache Kafka, PostgreSQL, Python + FastAPI, Angular + Leaflet, ngrok y Netlify conforme a responsabilidades del enunciado. | ENUNCIADO, §2 | Obligatorio | — | Inspeccionar dependencias, servicios y flujo funcional. | Listo para planificar |
| REQ-TEC-002 | Respetar Angular 20.3, ngx-leaflet 20.0 y Leaflet 1.9.4 del frontend de apoyo. Para otras tecnologías sin versión fijada, LTS es preferencia del estudiante, no exigencia docente. | README del material base, §1; aclaración del estudiante | Versiones del apoyo fijadas; resto, preferencia no obligatoria | REQ-TEC-001 | Revisar manifests/lockfile y que no se sustituyan las versiones del apoyo sin revisión. | Listo para planificar |

## Datos

| ID | Descripción verificable | Fuente y sección | Carácter | Dependencias | Aceptación / método | Estado |
|---|---|---|---|---|---|---|
| REQ-DATA-001 | Conservar GeoJSON original y metadatos de extracción antes de transformar; registrar procedencia, consultas, fecha y conteos; evitar copias idénticas indefinidas mediante retención definida. | ENUNCIADO, §§4.2 y 5 | Obligatorio | REQ-F-001, REQ-F-004 | Revisar muestras originales, metadatos y registro reproducible de consultas/retención. | Listo para planificar |
| REQ-DATA-002 | Preservar identidad, actualización y versiones; timestamps UTC milisegundos convertidos/documentados; coordenadas GeoJSON como longitud, latitud, profundidad km; desconocidos numéricos/país como nulos, nunca NaN/Infinity. | ENUNCIADO, §§4.1, 6 y 7; CONTRATO-API, `GET /api/eventos` | Obligatorio | REQ-F-003 | Validar fixtures de conversión y JSON; revisar una revisión y valores desconocidos/null. | Listo para planificar |
| REQ-DATA-003 | Integrar cada evento con copia Natural Earth mediante punto dentro de polígono, en WGS84/EPSG:4326; usar `ADM0_A3` tras verificar unicidad; conservar «Sin país asignado» y documentar fronteras/múltiples coincidencias. | ENUNCIADO, §4.3; FUENTES-Y-LICENCIAS, «Catálogo incluido» | Obligatorio | REQ-DATA-002 | Comprobar unicidad del identificador, casos dentro/fuera de polígono y regla documentada para bordes/colisiones. | Pendiente de definir regla de frontera |
| REQ-DATA-004 | La comparación histórica explicita filtros, periodo, unidad y registros sin valor; los filtros estadísticos equivalen a los del mapa. | CONTRATO-API, «Otros endpoints a implementar» | Obligatorio; estructura de respuesta pendiente | REQ-DATA-002, REQ-F-005 | Aceptación contractual tras revisión; comprobar igualdad de filtros y metadatos para casos de nulos. | Pendiente de especificación |
| REQ-DATA-005 | Identificar mensajes con `source`, `ingested_at`, `mode` (`live`/`replay`) y evento original; Kafka usa el ID de evento como clave. | ENUNCIADO, §5 | Obligatorio | REQ-ARCH-001, REQ-F-003 | Inspeccionar mensajes live y replay y comprobar clave/campos requeridos. | Listo para planificar |

## Calidad

| ID | Descripción verificable | Fuente y sección | Carácter | Dependencias | Aceptación / método | Estado |
|---|---|---|---|---|---|---|
| REQ-QUAL-001 | Aplicar validación y transformaciones documentadas; convertir fechas a UTC; separar ocurrencia/actualización/ingesta; conservar rechazos en cuarentena con motivo y referencia al original; no rellenar nulos indiscriminadamente. | ENUNCIADO, §6; FAQ, preguntas 16–17 | Obligatorio | REQ-F-006, REQ-DATA-002 | Comprobar reglas con casos controlados y presentar resultados antes/después, incluyendo cuarentena. | Listo para planificar |
| REQ-QUAL-002 | Analizar completitud, unicidad, validez, consistencia, actualidad y exactitud con límites declarados; nivel de rúbrica depende del número/profundidad de dimensiones y acciones. | ENUNCIADO, §§6 y 13 | Obligatorio analizar; umbrales de niveles según rúbrica | REQ-DATA-001, REQ-QUAL-001 | Memoria/notebook con consulta/código, denominador, resultado, decisión y limitación por dimensión analizada. | Pendiente de datos reales |

## Pruebas

| ID | Descripción verificable | Fuente y sección | Carácter | Dependencias | Aceptación / método | Estado |
|---|---|---|---|---|---|---|
| REQ-TEST-001 | Evento nuevo produce una fila actual visible por API, mapa y análisis. | ENUNCIADO, §11 | Obligatorio | REQ-F-003, REQ-API-001 | Ejecutar caso nuevo extremo a extremo y conservar evidencia. | Pendiente de implementación |
| REQ-TEST-002 | Repetir versión no aumenta eventos; revisión posterior cambia estado/agregaciones; versión antigua no revierte la nueva. | ENUNCIADO, §11; FAQ, pregunta 12 | Obligatorio | REQ-F-007, REQ-NF-002 | Reproducir secuencia controlada y comparar filas actuales, versiones y agregados. | Pendiente de implementación |
| REQ-TEST-003 | Magnitud nula se conserva y visualiza ND; coordenada inválida se pone en cuarentena con motivo; evento marítimo queda sin país y en análisis. | ENUNCIADO, §§4.3, 6 y 11 | Obligatorio | REQ-QUAL-001, REQ-DATA-003 | Ejecutar tres fixtures y revisar base, cuarentena y panel. | Pendiente de implementación |
| REQ-TEST-004 | Reinicio del consumidor reanuda sin duplicar efectos persistidos. | ENUNCIADO, §11 | Obligatorio | REQ-NF-002, REQ-ARCH-003 | Detener/reiniciar en punto controlado y comparar registros/offsets. | Pendiente de implementación |
| REQ-TEST-005 | Al cerrar túnel el panel muestra fallo y última lectura; al reabrirlo vuelve a consultar automáticamente. | ENUNCIADO, §11 | Obligatorio | REQ-F-012, REQ-DEP-003 | Prueba manual de desconexión y recuperación, con evidencia de estados. | Pendiente de contrato de salud |
| REQ-TEST-006 | Replay circula por Kafka sin mezclar live y replay. | ENUNCIADO, §11 | Obligatorio | REQ-F-013, REQ-DATA-005 | Ejecutar replay identificable y comprobar que la consulta/live no mezcla los datos según el mecanismo de aislamiento elegido. | Pendiente de definir aislamiento |
| REQ-TEST-007 | La entrega documenta mediciones de tiempo y memoria con al menos tres tamaños de entrada; datos sintéticos, si existen, quedan identificados y fuera del análisis real. | ENUNCIADO, §10 | Obligatorio | — | Revisar procedimiento, equipo, tamaños, medidas, unidades y separación de datos sintéticos. | Pendiente de mediciones |

## Despliegue

| ID | Descripción verificable | Fuente y sección | Carácter | Dependencias | Aceptación / método | Estado |
|---|---|---|---|---|---|---|
| REQ-DEP-001 | Entregar `compose.yaml`, Dockerfile(s), dependencias fijadas, inicialización de base, `.env.example` sin credenciales, volúmenes persistentes y retención limitada de Kafka. | ENUNCIADO, §9; §12 | Obligatorio | REQ-ARCH-001 | Arranque desde instrucciones en entorno limpio; reinicio conserva datos; inspeccionar secretos/retención. | Listo para planificar |
| REQ-DEP-002 | Uvicorn escucha en `0.0.0.0:8000`; CORS permite origen exacto Netlify y `http://localhost:4200`, GET/OPTIONS y `ngrok-skip-browser-warning`; ngrok termina HTTPS y el frontend utiliza URL HTTPS. | CONTRATO-API, «Acceso desde Netlify» y «Transporte y contenedores»; ENUNCIADO, §9 | Obligatorio | REQ-ARCH-001, REQ-API-001 | Verificar configuración, preflight OPTIONS, GET desde origen autorizado y acceso con HTTPS del túnel. | Listo para planificar |
| REQ-DEP-003 | Exponer solo la API de consulta por túnel; no publicar Kafka, PostgreSQL, administración o secretos; documentar procedimiento de activación de equipo/túnel para defensa. | ENUNCIADO, §9 y §12; CONTRATO-API, «Acceso desde Netlify» | Obligatorio | REQ-DEP-001, REQ-DEP-002 | Inspeccionar puertos/rutas expuestos y seguir procedimiento desde otro dispositivo. | Listo para planificar |
| REQ-DEP-004 | Publicar Angular como SPA en Netlify; documentar URL y configuración requerida de API. | ENUNCIADO, §§2, 9 y 12; README del material base, §4 | Obligatorio | REQ-DEP-002, REQ-API-001 | Compilar/publicar y comprobar carga desde otro dispositivo; no se exige SSR. | Listo para planificar |

## Documentación y entrega

| ID | Descripción verificable | Fuente y sección | Carácter | Dependencias | Aceptación / método | Estado |
|---|---|---|---|---|---|---|
| REQ-DOC-001 | Entregar repositorio ejecutable, URL del panel y procedimiento de activación para defensa; no incluir `node_modules`, credenciales, volúmenes binarios o grandes descargas; documentar regeneración de originales. | ENUNCIADO, §12 | Obligatorio | REQ-DEP-001, REQ-DEP-004 | Revisión de repositorio y ejecución guiada desde instrucciones. | Listo para planificar |
| REQ-DOC-002 | Documentar diccionario de datos, procedencia/consultas, modelo conceptual-lógico-físico, SQL/índices, reglas y limitaciones. | ENUNCIADO, §§5–7 y 12 | Obligatorio | REQ-DATA-001, REQ-DATA-002, REQ-DATA-003 | Revisar que campos, grano, claves, relaciones, unidades, fuentes y decisiones sean rastreables. | Listo para planificar |
| REQ-DOC-003 | Documentar análisis de calidad en memoria o notebook con código/consultas, resultados antes/después, decisiones, ejemplos y limitaciones; no requiere generador ni documento separado. | ENUNCIADO, §6; FAQ, preguntas 17 y 34 | Obligatorio; soporte automático opcional | REQ-QUAL-001, REQ-QUAL-002 | Revisar evidencias reproducibles y su denominador; no aceptar afirmaciones sin resultados. | Pendiente de resultados |
| REQ-DOC-004 | Documentar complejidad de al menos tres operaciones, mediciones de tiempo/memoria en tres tamaños, costes (hardware, electricidad, almacenamiento, transferencia, mantenimiento, desarrollo), supuestos y propuestas de escalado. | ENUNCIADO, §10 | Obligatorio | REQ-TEST-007 | Revisar operaciones, método, equipo, medidas, supuestos y estimación razonada para 100 millones. | Pendiente de mediciones |
| REQ-DOC-005 | Entregar evidencias de pruebas de aceptación y reproducción y poder explicar individualmente el flujo completo y una decisión técnica propia. | ENUNCIADO, §§11–12 | Obligatorio | REQ-TEST-001–006, REQ-F-013 | Repetir demo y revisar evidencias; defensa oral individual. | Pendiente de implementación |

## Requisitos mínimos del contrato API

### `GET /api/eventos`

Contrato obligatorio existente en [CONTRATO-API.md](../source-of-truth/CONTRATO-API.md), «GET /api/eventos»:

- Responde HTTP 200 con `Content-Type: application/json` y GeoJSON `FeatureCollection`; sin eventos, `features: []`, no error.
- Cada evento aparece una sola vez en su versión actual. Por defecto: terremotos de las últimas 24 horas.
- Metadata: `generated`, `last_ingestion`, `mode`, `count`, `truncated`. Los tiempos son milisegundos Unix UTC; `generated` no prueba que la ingesta esté activa.
- Feature: Point válido obligatorio; coordenadas [longitud, latitud, profundidad km]. Propiedades mínimas mostradas: `mag`, `magType`, `place`, `time`, `updated`, `country_id`, `country_name`.
- Valores desconocidos de magnitud, profundidad o país se representan como `null`, no NaN/Infinity.
- Filtros: `start` ISO8601 inclusivo, `end` ISO8601 exclusivo, `min_mag` y `country_id`. Magnitud nula no cumple `min_mag` y se cuenta separadamente.
- Si se limita tamaño, se documenta el máximo y, con truncado, `truncated: true`, total completo y mecanismo de paginación. El panel lo muestra y no calcula estadísticas globales sobre muestra truncada. El componente inicial espera colección pequeña y completa.

**Aceptación:** pruebas de contrato con respuesta vacía, respuesta con evento conocido, filtros extremos inclusivo/exclusivo, nulos, valores JSON válidos y conjunto truncado. Los casos de truncado/paginación deben cumplir el contrato si se adopta límite. La elección de límite no está fijada.

### `GET /api/estadisticas` y `GET /api/salud`

El contrato solo indica para estadísticas conteos por hora UTC/país, distribuciones y comparación histórica; filtros equivalentes a mapa; cada resultado identifica periodo, filtros, unidad y registros sin valor. Si falla la base, debe responder error explícito y no simular cero eventos.

Para salud solo declara estado de API y fecha de última ingesta, sin credenciales.

**Aclaración confirmada por el estudiante:** los contratos detallados de ambas rutas están pendientes y se propondrán posteriormente para revisión. No se definen aquí payloads, códigos concretos de error, nombres de campos adicionales ni semántica no establecida en las fuentes.

| ID | Descripción verificable | Categoría | Fuente y sección | Carácter | Dependencias | Aceptación / método | Estado |
|---|---|---|---|---|---|---|---|
| REQ-API-001 | `GET /api/eventos` cumple la forma GeoJSON, HTTP y metadata, unicidad/versionado, valores desconocidos, filtros y reglas de truncado descritas arriba. | Funcional / API | CONTRATO-API, `GET /api/eventos` | Obligatorio | REQ-DATA-002, REQ-F-007 | Probar respuesta vacía, feature válido, filtros, nulos, fechas inclusivas/exclusivas y truncado/paginación si se limita tamaño. | Listo para planificar |
| REQ-API-002 | `GET /api/estadisticas` ofrece conteos por hora UTC/país, distribuciones y comparación histórica; aplica filtros equivalentes al mapa y cada resultado identifica periodo, filtros, unidad y registros sin valor. | Funcional / API | CONTRATO-API, «Otros endpoints a implementar» | Obligatorio en capacidades; detalle de contrato no especificado | REQ-DATA-002, REQ-F-005 | Proponer contrato y aprobarlo antes de implementarlo; luego comparar cada agregado/filtro contra consulta de referencia. | Pendiente de especificación y revisión |
| REQ-API-003 | `GET /api/salud` informa estado de API y fecha de última ingesta, sin credenciales; los contratos detallados aún requieren propuesta y revisión. | Funcional / API | CONTRATO-API, «Otros endpoints a implementar» | Obligatorio en propósito; detalle de contrato no especificado | REQ-ARCH-001, REQ-F-003 | Proponer contrato y aprobarlo; probar servicio disponible, ingesta antigua y fallo sin exponer secretos. | Pendiente de especificación y revisión |

## Dependencias importantes

| Requisito dependiente | Depende de | Motivo |
|---|---|---|
| REQ-F-003 / REQ-F-007 / REQ-NF-002 | REQ-ARCH-001 y REQ-DATA-002 | La detección de versiones, persistencia e idempotencia requiere servicios y una identidad/versionado consistente. |
| REQ-F-008 / REQ-F-009 / REQ-F-010 | REQ-API-001/002 y REQ-DATA-003/004 | Panel, clasificación y análisis requieren API y datos enriquecidos con filtros comparables. |
| REQ-F-012 / REQ-TEST-005 | REQ-API-003 y REQ-DEP-002 | Estado visible y recuperación requieren definición de salud y acceso del frontend. |
| REQ-F-013 / REQ-TEST-006 | REQ-DATA-005 y decisión de aislamiento | Replay debe distinguir modo, conservar tiempos y no mezclar live. |
| REQ-DOC-003 / REQ-DOC-004 / REQ-DOC-005 | Implementación y evidencia ejecutada | Documentar resultados requiere mediciones/pruebas reales, no solo el comportamiento esperado. |

## Incompletos o pendientes de definición

En la columna «Impacto», **no bloquea planificación** permite avanzar el trabajo general con los límites conocidos; **bloquea implementación** impide implementar/configurar el aspecto concreto hasta cerrar y revisar la definición. Son bloqueos de alcance, no estados de avance.

| Tema/ID | Estado y distinción de autoridad | Spec donde resolver | Impacto |
|---|---|---|---|
| Estadísticas — DEC-P-01 / REQ-API-002 | El contrato docente solo fija capacidades mínimas; el estudiante confirmó que el equipo preparará una propuesta. Payload y semántica detallada pendientes, no aprobados. | Futura `docs/spec/04-contratos-api.md` (no creado); reflejar aprobación en esta spec 01 | No bloquea planificación global; bloquea especificar/implementar `/api/estadisticas` |
| Salud — DEC-P-02 / REQ-API-003 | El contrato docente fija solo estado de API y última ingesta, sin credenciales; el estudiante confirmó que el equipo propondrá detalles. Payload pendiente, no aprobado. | Futura `docs/spec/04-contratos-api.md` (no creado); reflejar aprobación en esta spec 01 | No bloquea planificación global; bloquea especificar/implementar `/api/salud` |
| Modelo físico — DEC-P-03a | Requisito docente: explicar grano, claves, relaciones e índices; no se prescribe un modelo físico concreto. Ejemplos de FAQ/guía no son obligación. | Futura `docs/spec/05-modelo-logico-fisico.md` (no creado), sobre conceptos en [03](03-modelo-datos-calidad.md) | No bloquea planificación conceptual; bloquea implementación de esquema/persistencia |
| Índices — DEC-P-03b | Índices concretos no especificados; deben justificarse según consultas y mediciones. | Futura `docs/spec/05-modelo-logico-fisico.md` (no creado) | No bloquea planificación general; bloquea definir/implementar índices físicos, no el diseño de consultas |
| Tópicos — DEC-P-03c | Las fuentes exigen Kafka y configurar topic; no fijan nombres ni organización live/replay. | Revisión futura de [02-arquitectura-flujos.md](02-arquitectura-flujos.md) | No bloquea planificación; bloquea configuración e integración de productor/consumidor |
| Aislamiento replay — DEC-P-03d | El enunciado exige no mezclar replay con live; tópico/esquema/tablas separados o mecanismo equivalente son alternativas, no decisión aprobada. | Revisión futura de [02-arquitectura-flujos.md](02-arquitectura-flujos.md) y, si afecta almacenamiento, futura `docs/spec/05-modelo-logico-fisico.md` | No bloquea planificación del flujo; bloquea integración/implementación de replay |
| Puntos de frontera — DEC-P-04 / REQ-DATA-003 | El enunciado exige documentar una regla, pero no prescribe cuál; no hay propuesta aprobada. | Revisión futura de [03-modelo-datos-calidad.md](03-modelo-datos-calidad.md) | No bloquea planificación; bloquea cerrar y probar asignación espacial en casos limítrofes |
| Retención — DEC-P-RET | El enunciado exige conservar originales y definir retención, y Kafka con retención limitada; duraciones y política concreta no especificadas. | Revisión futura de [03-modelo-datos-calidad.md](03-modelo-datos-calidad.md), coordinada con [02](02-arquitectura-flujos.md) para Kafka | No bloquea planificación conceptual; bloquea fijar políticas de retención/purga y su configuración |
| Comparación histórica — DEC-P-05 / REQ-F-005 | **Aclaración confirmada:** 2025, global, magnitud ≥4, `earthquake`; para trabajo/filtros/comparación, no filtro live. La regla de filtros equivalentes y denominador de días sin eventos procede del enunciado. | Aclaración ya aplicada en esta spec y [03](03-modelo-datos-calidad.md); no queda decisión abierta | No bloquea planificación; la aceptación de resultados requiere datos/pruebas reales |
| Fecha de entrega — DEC-P-06 | **Aclaración confirmada:** proyecto individual, plazo comunicado dentro de 20 días. Fecha calendario exacta no confirmada y no se infiere. | Especificación futura de calendario/entrega solo si se requiere; no hay documento creado | No bloquea planificación técnica ni implementación; bloquea fijar calendario exacto de entrega/defensa |
| Carga inicial histórica — REQ-F-004 | El enunciado fija extracción y registro; no prescribe un proceso de carga único. FAQ ofrece alternativas orientativas. | Revisión futura de [02-arquitectura-flujos.md](02-arquitectura-flujos.md) | No bloquea planificación; bloquea elegir/implementar el mecanismo concreto de carga |
| Listeners Kafka — REQ-DEP-001 / REQ-ARCH-001 | Deben usarse nombres de servicio dentro de Compose y documentar listeners si se accede desde host; valores/topología concretos no fijados. | Revisión futura de [02-arquitectura-flujos.md](02-arquitectura-flujos.md) | No bloquea diseño funcional; bloquea configuración de red y acceso host |
| Ejecución de ngrok — REQ-DEP-002/003 | El profesor admite ejecución en host o contenedor; no se ha elegido alternativa. | Revisión futura de [02-arquitectura-flujos.md](02-arquitectura-flujos.md) o instrucciones de despliegue posteriores | No bloquea planificación; bloquea configuración concreta del túnel/despliegue |
| Umbral de datos desactualizados — REQ-F-012 | El enunciado exige avisar, pero no define umbral numérico. No se propone valor. | Propuesta futura en la revisión de [02-arquitectura-flujos.md](02-arquitectura-flujos.md), coordinada con 04 si afecta a salud API | No bloquea planificación; bloquea decidir el comportamiento concreto de aviso |
| Resultado del material base | [VALIDACION.md](../source-of-truth/VALIDACION.md) informa compilación/frontend y catálogo, no backend/API/pipeline implementados o probados. | No es decisión de diseño; actualizar evidencia al realizar pruebas futuras, conforme a REQ-TEST | No bloquea planificación; no se puede declarar aceptación del backend antes de pruebas reales |

Las aclaraciones del estudiante —modalidad, plazo comunicado, preferencia LTS solo donde no hay versión fijada, flujo de herramientas, permisos de agentes, tutoría y no introducir hooks al inicio— están registradas en [00-principios-y-alcance.md](00-principios-y-alcance.md). Son preferencias/procedimientos del estudiante, no requisitos docentes ni bloqueos de implementación del producto.

## Trazabilidad requisito → fuente → evaluación

| Requisitos | Fuente principal | Criterio de evaluación o prueba |
|---|---|---|
| REQ-F-001–005, REQ-DATA-001 | ENUNCIADO §§4–5 | Consultas registradas, tipos/unidades, deduplicación, frecuencia y tratamiento de live/histórico. |
| REQ-F-006–007, REQ-NF-002, REQ-DATA-002, REQ-QUAL-001 | ENUNCIADO §§5–7, §11 | Cuarentena, idempotencia, secuencia de versiones, persistencia transaccional y nulos. |
| REQ-DATA-003, REQ-TEST-003 | ENUNCIADO §§4.3, 6, 11 | Join espacial, evento marítimo, reglas de frontera y categoría sin país. |
| REQ-F-008–012, REQ-API-001–003 | ENUNCIADO §8; CONTRATO-API | Contratos de API, filtros coherentes, mapa/tabla/gráficos, estadísticas y estado/error. |
| REQ-F-013, REQ-TEST-006, REQ-DATA-005 | ENUNCIADO §11 | Replay Kafka aislado, etiquetado, fechas originales y escenarios de duplicado/revisión. |
| REQ-ARCH-001–003, REQ-TEC-001–002, REQ-DEP-001–004 | ENUNCIADO §§2, 9, 12; README del material base §§1, 3–4 | Cinco servicios Compose, tecnologías, versiones frontend, arranque, CORS, túnel y publicación. |
| REQ-QUAL-002, REQ-DOC-002–004, REQ-TEST-007 | ENUNCIADO §§6, 10, 12–13 | Evidencias de calidad, modelo, análisis, tres conclusiones, tres tamaños medidos, costes y reflexión conforme rúbrica. |
| REQ-DOC-001, REQ-DOC-005 | ENUNCIADO §§11–13; guía de entregables | Repositorio reproducible, URL/procedimiento, evidencias y explicación individual. |

La tabla de rúbrica del profesor está en [ENUNCIADO.md, §13](../source-of-truth/ENUNCIADO.md#13-evaluación). Los niveles se aplican allí; este documento no inventa ponderaciones ni resultados.
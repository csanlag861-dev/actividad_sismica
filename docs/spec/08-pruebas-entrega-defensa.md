# Pruebas, entrega y defensa

**Estado:** borrador pendiente de revisión del estudiante. Organiza los criterios de aceptación, las evidencias, los entregables y la preparación de la defensa; no implementa ni ejecuta pruebas. Todo escenario de aceptación empieza en estado **Pendiente**.

## 1. Propósito y límites

Esta especificación reúne cómo verificar el observatorio y qué evidencias y entregables deberán prepararse. No declara que el proyecto esté implementado, desplegado o que haya superado pruebas. Las pruebas deben ejecutarse y registrarse antes de cambiar su estado a «Ejecutada» o declarar un resultado.

Los requisitos y sus identificadores canónicos permanecen en [01-requisitos-aceptacion.md](01-requisitos-aceptacion.md). Los flujos de ingesta se detallan en [04-ingesta-procesamiento.md](04-ingesta-procesamiento.md), el contrato API en [05-api-contratos.md](05-api-contratos.md), el comportamiento del panel en [06-panel-analisis.md](06-panel-analisis.md) y el despliegue/operación en [07-despliegue-operacion.md](07-despliegue-operacion.md). Esta spec los enlaza sin duplicarlos ni cambiar sus criterios.

## 2. Estrategia de verificación

Para cada requisito aplicable se mantiene la cadena:

**requisito y fuente → criterio de aceptación → prueba/caso → evidencia observable → resultado registrado.**

Como **organización propuesta**, las verificaciones pueden agruparse en:

- **Unitarias:** reglas aisladas, como normalización UTC, filtros y tratamiento de nulos.
- **Integración:** interacción de productor, Kafka, consumidor y PostgreSQL; idempotencia, offsets y recuperación.
- **Contrato:** forma y semántica de API conforme a [05-api-contratos.md](05-api-contratos.md).
- **Aceptación/extremo a extremo:** recorrido desde evento procesado hasta API/panel y operación de túnel.
- **Revisión documental:** modelo, procedencia, calidad, mediciones, entregables e instrucciones.

Estas categorías son una propuesta de organización, no herramientas ni un framework exigido por el profesor. Las fuentes permiten evidencias mediante consultas SQL, logs seleccionados, capturas y demostraciones reproducibles; no prescriben un framework de pruebas. (ENUNCIADO, §11; Guía de entregables, §§2.8–2.9.)

Un criterio descrito o un test escrito no equivale a una prueba ejecutada. Para cada ejecución futura se registrarán, como mínimo, la versión del código/configuración probada, las precondiciones/datos usados, el resultado observado y la evidencia que permita repetir o revisar el caso. La forma exacta de organizar los ficheros de evidencia no está prescrita.

## 3. Matriz de pruebas de aceptación

Las precondiciones y pasos se describen a alto nivel. No se fija cómo inyectar eventos ni cómo aislar el replay: esas decisiones deben seguir las specs de ingesta, datos y arquitectura.

| Escenario / requisitos relacionados | Precondiciones y datos/caso de prueba | Pasos a alto nivel | Resultado esperado según fuentes | Evidencia que se debe guardar | Estado |
|---|---|---|---|---|---|
| Evento nuevo: `REQ-TEST-001`, `REQ-F-003`, `REQ-API-001` | Pipeline disponible; caso de evento nuevo identificable. | Enviar/seleccionar el evento, esperar persistencia y consultar API/panel. | Una fila/estado actual por evento, visible en API y panel/análisis. (ENUNCIADO, §11.) | Mensaje/procedencia, consulta de almacenamiento, respuesta de API y captura o registro del panel, con el mismo `id`. | Pendiente |
| Reenvío de la misma versión: `REQ-TEST-002`, `REQ-F-007`, `REQ-NF-002` | Evento previamente persistido con su `id` y `updated`. | Reenviar la misma versión y comparar el estado antes/después. | No aumenta el número de eventos ni duplica los efectos persistidos. (ENUNCIADO, §§7 y 11.) | Conteos/consulta antes y después; evidencia del mensaje repetido y estado resultante. | Pendiente |
| Revisión posterior: `REQ-TEST-002`, `REQ-F-007` | Evento y versión previa conocidos; revisión del mismo `id` con `updated` posterior. | Procesar la revisión y comparar estado actual, historial y agregaciones afectadas. | Se actualiza el estado actual y sus agregaciones sin contar un evento adicional. (ENUNCIADO, §§7–8 y 11.) | Valores antes/después, versiones observadas y agregados afectados. | Pendiente |
| Versión antigua posterior a la nueva: `REQ-TEST-002`, `REQ-F-007` | Hay una versión nueva persistida y una versión anterior disponible para prueba. | Procesar la secuencia nueva→antigua. | La versión antigua no sustituye a la más reciente. (ENUNCIADO, §11.) | Comparación de `updated`/campos del estado actual antes y después; historial observado. | Pendiente |
| Magnitud nula: `REQ-TEST-003`, `REQ-DATA-002` | Caso válido con `mag=null`. | Persistirlo, consultarlo por API y observar representación/filtros/análisis. | Se conserva como desconocida, se muestra como ND y no se reemplaza por cero; no satisface `min_mag` y se contabiliza aparte donde corresponda. (ENUNCIADO, §§6 y 11; CONTRATO-API, «GET /api/eventos».) | Registro/consulta, respuesta JSON y captura de la representación o conteo de nulos. | Pendiente |
| Coordenadas inválidas: `REQ-TEST-003`, `REQ-F-006`, `REQ-QUAL-001` | Caso sintético inválido, identificado como dato de prueba y separado de los datos reales. | Enviar el caso por la validación del consumidor y consultar cuarentena/API. | Se conserva el rechazo con motivo; no aparece un punto inventado en la consulta. (ENUNCIADO, §§6 y 11.) | Fixture sintético etiquetado, registro de cuarentena y verificación de ausencia en eventos válidos. | Pendiente |
| Evento marítimo/sin país: `REQ-TEST-003`, `REQ-DATA-003` | Evento válido cuyas coordenadas no caen en un polígono asignable. | Procesarlo y revisar país, mapa/tabla y análisis. | Se conserva con «Sin país asignado» y permanece en el análisis; no se imputa el país más cercano. (ENUNCIADO, §§4.3, 8 y 11.) | Evento y resultado geográfico en almacenamiento/API; vista de categoría sin país. | Pendiente |
| Reinicio del consumidor: `REQ-TEST-004`, `REQ-NF-002`, `REQ-ARCH-003` | Mensajes disponibles; elegir un punto controlado de interrupción sin presuponer mecanismo físico no aprobado. | Interrumpir/reiniciar el consumidor y permitir reentrega si corresponde. | Reanuda el consumo sin duplicar efectos ya persistidos; escritura precede a confirmación del offset. (ENUNCIADO, §§7 y 11.) | Logs/offsets disponibles y comparación de registros/efectos antes y después. | Pendiente |
| Cierre del túnel: `REQ-TEST-005`, `REQ-F-012`, `REQ-DEP-003` | Panel con una lectura API satisfactoria previa; túnel funcionando al inicio. | Cerrar el acceso por túnel y observar el panel. | El panel informa que no puede actualizar y conserva/identifica la lectura previa sin presentarla como nueva. (ENUNCIADO, §§8 y 11; README del material base, §3.) | Captura del error con marca temporal de última lectura; registro de solicitud fallida sin secretos. | Pendiente |
| Reapertura del túnel: `REQ-TEST-005`, `REQ-F-012`, `REQ-DEP-003` | Prueba anterior con fallo visible; túnel/API restaurados. | Reabrir el túnel y esperar la siguiente consulta prevista. | Las consultas se recuperan automáticamente. (ENUNCIADO, §11.) | Captura/log de consulta satisfactoria posterior y cambio de estado del panel. | Pendiente |
| Replay aislado: `REQ-TEST-006`, `REQ-F-013`, `REQ-DATA-005` | Selección histórica ordenada; mecanismo de aislamiento aprobado y entorno de prueba diferenciado. | Reproducir de forma acelerada por Kafka manteniendo tiempo original y registrando tiempo de reproducción; consultar modo replay. | Replay no se mezcla con live, se identifica «REPRODUCCIÓN» y permite los casos controlados de duplicado/revisión. (ENUNCIADO, §11; FAQ, pregunta 32.) | Selección/procedencia, logs/mensajes, verificación de aislamiento y captura de la etiqueta. | Pendiente; bloqueada hasta resolver aislamiento |

## 4. Pruebas de API, filtros y panel

La estructura de `GET /api/eventos`, los requisitos mínimos de estadísticas/salud y los detalles aprobados están en [05-api-contratos.md](05-api-contratos.md); las pruebas no deben crear contratos alternativos.

| Caso de verificación | Criterio y trazabilidad | Dependencias/estado |
|---|---|---|
| `GET /api/eventos`: respuesta válida vacía y FeatureCollection | Respuesta vacía válida, no error; GeoJSON y propiedades/metadata definidos en contrato. `REQ-API-001`. Fuente: CONTRATO-API, «GET /api/eventos». | Pendiente de implementación y evidencia. |
| Filtros y timestamps de eventos | `start` inclusivo, `end` exclusivo, `min_mag`, `country_id`; timestamps en milisegundos Unix UTC; coordenadas y unidades conforme al contrato. `REQ-API-001`, `REQ-DATA-002`. | Pendiente de prueba contractual. |
| Estadísticas: periodo, filtros, unidades y faltantes | Verificar capacidades obligatorias y la forma aprobada en 05; contrastar contra cálculo independiente con el mismo conjunto/filtros. `REQ-API-002`, `REQ-DATA-004`, `REQ-F-009`, `REQ-F-010`. Fuente: CONTRATO-API, «Otros endpoints a implementar»; ENUNCIADO, §8. | Pendiente. Las variantes que 05 §9 mantenga expresamente como decisión pendiente no se convierten en criterio hasta resolverse. |
| Salud: última ingesta y no exposición de credenciales | Verificar estado mínimo, última ingesta y ausencia de credenciales. `REQ-API-003`, `REQ-NF-004`, `REQ-F-012`. Fuente: CONTRATO-API, «Otros endpoints a implementar». | Pendiente. Umbral numérico de obsolescencia permanece por decidir en 05 §5. |
| Fallo de PostgreSQL para estadísticas | Debe haber error explícito; no respuesta exitosa con cero eventos. `REQ-NF-003`, `REQ-API-002`. Fuente: CONTRATO-API, «Otros endpoints a implementar». | Pendiente de prueba con fallo controlado. Código HTTP/formato exacto solo según decisiones aprobadas en 05. |
| Coherencia mapa/tabla/estadísticas | Los filtros de fecha, magnitud y país/región producen conjuntos/denominadores coherentes; comparación muestra periodo y filtros efectivos. `REQ-F-010`, `REQ-API-001`, `REQ-API-002`, `REQ-DATA-004`. Fuente: ENUNCIADO, §8; 05 §§3–4. | Pendiente de integración y contraste independiente. |
| Respuesta parcial/paginada | Si la respuesta de eventos indica truncamiento, el panel lo comunica y continúa según contrato; no deriva estadísticas globales de una página. `REQ-API-001`, `REQ-F-008`. Fuente: CONTRATO-API, «GET /api/eventos»; 05 §3. | Pendiente. La lógica general de truncamiento es exigida; límite numérico y elementos de navegación siguen pendientes en 05 §3.2. |
| Estados del panel | Verificar carga, vacío, fallo, última lectura conservada/antigua, replay y recuperación conforme a 06 y a ENUNCIADO §§8, 11. `REQ-F-008`–`REQ-F-013`, `REQ-TEST-005`–`REQ-TEST-006`. | Pendiente; textos y representación visual siguen siendo decisiones de interfaz. |

El estudiante aprobó el fichero 05; ese hecho no completa automáticamente detalles que el propio documento dejó sin valor decidido, como el umbral de obsolescencia o el límite numérico de página. Para esos casos, la prueba del comportamiento general puede diseñarse, pero la verificación de un valor concreto depende de que se cierre primero en 05.

## 5. Verificación de calidad de datos

**Requisito del profesor:** documentar el análisis de calidad en un apartado de la memoria o notebook con código/consultas, resultados antes/después y explicación de decisiones; no se exige generar el informe automáticamente ni crear un documento o panel de calidad separado. La cuarentena duradera sí debe conservar originales y motivos. (ENUNCIADO, §6; FAQ, preguntas 16–17; Guía de entregables, §2.4.)

El apartado de calidad deberá dejar evidencia de:

- **Conjunto, periodo y denominador:** especificar si se cuentan mensajes, versiones observadas o eventos actuales; indicar fuente, periodo y filtros.
- **Código o consultas:** incluirlos o enlazarlos de forma que otra persona pueda revisar cómo se obtuvieron los resultados.
- **Dimensiones:** analizar completitud, unicidad, validez, consistencia, actualidad y exactitud dentro de lo que se pueda verificar. Si una dimensión no puede comprobarse, describir la limitación. (ENUNCIADO, §6.)
- **Antes/después:** cuando se aplique transformación, mostrar efecto y número de registros afectados; conservar unidades/UTC y no rellenar automáticamente desconocidos con cero. (ENUNCIADO, §6.)
- **Decisiones y limitaciones:** justificar tratamiento de ausencias, anomalías y transformaciones; explicar por qué validación de formato no prueba exactitud física.
- **Cuarentena:** incluir ejemplos verificables de originales/referencias y motivos duraderos, sin datos reales innecesarios en la documentación.
- **Datos sintéticos:** marcar cualquier caso artificialmente dañado y mantenerlo separado de los datos reales. No inventar problemas en el conjunto real. (ENUNCIADO, §6; FAQ, preguntas 16–17.)

Los niveles de la rúbrica consideran número/profundidad de dimensiones y acciones justificadas; no se inventan umbrales de aprobación adicionales. (ENUNCIADO, §13.)

## 6. Mediciones, complejidad y costes

**Requisitos del profesor:** justificar al menos tres estructuras u operaciones; medir tiempo y memoria con al menos tres tamaños de entrada; documentar equipo, procedimiento y resultados; razonar qué cambiaría con 100 millones de registros; explicar costes y supuestos; identificar datos sintéticos y separarlos del análisis sísmico real. (ENUNCIADO, §10; FAQ, pregunta 33; Guía de entregables, §2.7.)

La evidencia de esta sección debe incluir:

1. Al menos tres operaciones de la implementación, por ejemplo búsqueda de versiones por identificador, integración espacial y agregación por ventanas; para cada una, explicar estructura/coste medio y peor caso cuando aplique.
2. Mediciones de tiempo y memoria en tres o más tamaños, anotando equipo, procedimiento, unidad y resultado observado. No se rellenan cifras estimadas como si fueran medidas.
3. Un análisis razonado de escalado a 100 millones, sustentado por operaciones/mediciones; no hace falta desplegar o generar realmente ese volumen.
4. Costes de hardware/recursos, electricidad, almacenamiento, transferencia, mantenimiento y tiempo de desarrollo; distinguir mediciones de supuestos e indicar cómo se obtuvieron. No inventar precios ni considerar automáticamente una cuota gratuita como coste total cero.
5. Si se generan datos sintéticos para probar escala, identificarlos y excluirlos del análisis de sismicidad real.

Todo resultado permanece pendiente hasta que exista evidencia. `REQ-TEST-007` y `REQ-DOC-004` son los identificadores existentes para estas obligaciones.

## 7. Entregables

La lista agrupa, sin añadir ni eliminar, los entregables del profesor. Fuente principal: ENUNCIADO, §12; detalle orientativo: Guía de entregables, §§1–3.

| Entregable | Contenido que debe poder verificarse | Requisitos existentes |
|---|---|---|
| Repositorio ejecutable | Código de productor, consumidor, API y frontend; Dockerfile(s), `compose.yaml` con cinco servicios, dependencias fijadas, inicialización de base y `.env.example` sin credenciales. | REQ-ARCH-001, REQ-DEP-001, REQ-TEC-001, REQ-DOC-001 |
| Instrucciones reproducibles | Requisitos/versiones usadas, configuración local, arranque, inicialización/carga histórica, estado/logs, ejecución y publicación del frontend, activación de ngrok, pruebas/replay, parada y datos conservados. | REQ-DEP-001–004, REQ-DOC-001 |
| Panel y activación para defensa | URL de Netlify y procedimiento para activar backend/túnel durante la defensa. | REQ-DEP-003, REQ-DEP-004, REQ-DOC-001 |
| Diccionario, procedencia y consultas | Campos y unidades, fuentes, consultas históricas, fechas, conteos y reproducibilidad de datos originales. | REQ-DATA-001–002, REQ-DOC-002 |
| Calidad en memoria o notebook | Código/consultas, resultados, denominadores, antes/después, decisiones, cuarentena y límites; sin obligación de generador ni archivo separado. | REQ-QUAL-001–002, REQ-DOC-003 |
| Modelo y SQL | Modelo conceptual/lógico/físico, granos, claves/relaciones, SQL e índices justificados. | REQ-DOC-002 |
| Análisis | Al menos tres conclusiones fundamentadas, con cifras reales, periodos/filtros identificables y capturas que muestren los filtros; comparación equivalente. | REQ-F-005, REQ-F-009–011, REQ-DATA-004 |
| Complejidad, mediciones, costes y mejoras | Tres operaciones justificadas, tres tamaños medidos en tiempo/memoria, equipo/procedimiento, análisis de escala, costes y propuestas de mejora sustentadas. | REQ-TEST-007, REQ-DOC-004 |
| Pruebas y replay | Evidencias de aceptación y reproducción histórica aislada e identificada. | REQ-TEST-001–006, REQ-F-013, REQ-DOC-005 |
| Explicación individual | Explicar el flujo completo y una decisión técnica propia. | REQ-DOC-005 |

**Exclusiones expresas del repositorio:** no subir credenciales, `node_modules`, volúmenes binarios ni grandes conjuntos descargables; documentar ubicación/procedimiento para regenerar los datos originales. (ENUNCIADO, §12; README del material base, §4; Guía de entregables, §§1.3–1.4.)

No se exige un documento separado de calidad, un generador de informe, un panel de calidad ni un archivo de memoria con un nombre específico. La organización documental de la guía es orientativa. (ENUNCIADO, §6; FAQ, preguntas 17 y 34; Guía de entregables, §§2 y «Organización sugerida de la documentación».)

## 8. Preparación de la defensa

### Lista de comprobación basada en el profesor

Antes de la defensa, la verificación guiada debería cubrir:

- [ ] Arrancar los cinco servicios con Compose y comprobar su estado.
- [ ] Comprobar que la base contiene los datos necesarios y que la ingesta funciona.
- [ ] Activar ngrok y comprobar la URL HTTPS de la API configurada en el frontend y CORS.
- [ ] Abrir el panel desde otro dispositivo y comprobar datos, filtros y estado de actualización.
- [ ] Preparar una selección de replay por Kafka, con fechas originales, aislamiento e identificación «REPRODUCCIÓN».
- [ ] Poder explicar el trayecto de un evento desde USGS hasta el panel, la responsabilidad de cada componente y una decisión técnica propia.

Fuente: Guía de entregables, §§3.1–3.3; ENUNCIADO, §§11–12. La prueba del material base indica que no se ha desplegado una cuenta real de Netlify ni abierto el túnel del estudiante; por tanto, todos estos puntos son preparatorios, no confirmaciones de éxito. (VALIDACION.md, «Límites de esta comprobación».)

### Preferencias confirmadas del estudiante

- El proyecto se realizará individualmente; la explicación del flujo y de una decisión técnica será individual, en consonancia con ENUNCIADO, §12.
- La entrega se comunicó dentro de 20 días, pero no se ha confirmado una fecha calendario exacta; no se infiere ni se incorpora al calendario de defensa.
- El tutor formulará preguntas al cierre de tareas y durante la revisión de documentación; al final se realizará un simulacro de defensa. Es una preferencia del estudiante, no un requisito docente.
- El tutor debe esperar respuestas, ayudar a corregir conceptos sin inventar explicaciones y limitar aclaraciones didácticas a `docs/learning/` cuando estén verificadas y cuenten con aprobación del estudiante.
- Copilot en VS Code coordina; Codex y Antigravity reciben tareas acotadas con artefactos canónicos. Agentes de análisis/revisión son read-only por defecto; implementar requiere tarea aprobada y despliegues/acciones destructivas requieren confirmación. Son reglas operativas confirmadas por el estudiante, no criterios de aceptación del producto.

## 9. Trazabilidad y estado de evidencias

Las pruebas listadas anteriormente y las matrices relacionadas son **Pendientes** hasta aportar evidencia observable. No se marca ninguna como «Ejecutada» en esta spec. `VALIDACION.md` registra comprobaciones del frontend de apoyo y del catálogo, pero no valida el backend/pipeline ni equivale a ejecutar las pruebas de aceptación del proyecto.

| Requisito | Fuente principal | Prueba/evidencia | Documento o entregable de registro | Estado |
|---|---|---|---|---|
| `REQ-TEST-001`–`REQ-TEST-004` | ENUNCIADO, §11; §§7–8 para revisiones | Evento nuevo, duplicado, revisión, versión antigua, nulos, coordenadas inválidas, evento marítimo y reinicio; consultar matriz §3. | Evidencias de prueba y logs/consultas; resumen en memoria o notebook. | Pendiente |
| `REQ-TEST-005` | ENUNCIADO, §11; §§8–9 | Cierre y reapertura de túnel, fallo visible y recuperación. | Evidencia del panel, logs/registro de solicitudes y README de ejecución. | Pendiente |
| `REQ-TEST-006`, `REQ-F-013` | ENUNCIADO, §11; FAQ, pregunta 32 | Replay por Kafka, aislado, ordenado y etiquetado, con duplicado/revisión controlados. | Procedimiento y evidencias de reproducción. | Pendiente |
| `REQ-API-001`–`REQ-API-003`, `REQ-NF-003` | CONTRATO-API; [05-api-contratos.md](05-api-contratos.md) | Respuestas de contrato, filtros, errores y estados de salud conforme a las partes aprobadas. | Suite/evidencias de contrato y registro de decisiones pendientes en 05. | Pendiente |
| `REQ-F-008`–`REQ-F-012` | ENUNCIADO, §8; `CONTRATO-API.md` | Panel, filtros coherentes, estados de error/antigüedad, truncamiento y tabla. | Capturas con filtros visibles, comprobaciones y evidencia de API. | Pendiente |
| `REQ-QUAL-001`–`REQ-QUAL-002`, `REQ-DOC-003` | ENUNCIADO, §6; FAQ, preguntas 16–17 | Consultas/código, denominador, resultados, transformaciones, cuarentena y límites. | Apartado de memoria o notebook. | Pendiente |
| `REQ-TEST-007`, `REQ-DOC-004` | ENUNCIADO, §10; FAQ, pregunta 33 | Mediciones de tiempo/memoria en tres tamaños, equipo, procedimiento, complejidad, costes y escala. | Memoria o notebook con resultados observados y supuestos separados. | Pendiente |
| `REQ-DEP-001`–`REQ-DEP-004`, `REQ-DOC-001` | ENUNCIADO, §§2, 9 y 12; Guía de entregables, §§1 y 3 | Arranque, persistencia, CORS/ngrok, exposición, Netlify y acceso desde otro dispositivo. | Repositorio y README reproducible, más evidencias de despliegue. | Pendiente |
| `REQ-DOC-002`, `REQ-DOC-005` | ENUNCIADO, §§5–7 y 12 | Diccionario/modelo/procedencia; explicación individual del flujo y decisión. | Memoria/notebook, SQL y defensa individual. | Pendiente |

El valor «Pendiente» describe el estado inicial del plan de verificación, no el resultado de una ejecución. Una vez realizada una prueba, registrar resultado observado (aprobada/fallida), fecha de ejecución según el registro real, evidencia y defectos asociados. No inventar fechas, capturas ni resultados.

## 10. Decisiones y dependencias pendientes

| Dependencia pendiente | Qué bloquea | Dónde resolver |
|---|---|---|
| Aprobación del contrato API | No bloquea pruebas de los requisitos ya definidos; bloquea la aceptación de detalles que la spec API no haya cerrado. El archivo 05 figura aprobado por el estudiante, pero su propio texto conserva algunas subdecisiones como pendientes. | Revisar en [05-api-contratos.md](05-api-contratos.md) §9 antes de probar valores concretos; preservar el contrato aprobado y no reabrir lo que ya se aprobó. |
| Aislamiento concreto de replay | Bloquea la ejecución verificable de `REQ-TEST-006` y la demostración sin mezcla live/replay; no bloquea diseñar el caso de prueba. | Resolver en [02-arquitectura-flujos.md](02-arquitectura-flujos.md) y coordinar con [03-modelo-datos-calidad.md](03-modelo-datos-calidad.md) / [04-ingesta-procesamiento.md](04-ingesta-procesamiento.md). |
| Regla de puntos en fronteras/múltiples coincidencias | Bloquea pruebas concluyentes de asignación de los casos limítrofes, no las demás pruebas de calidad o geografía. | Resolver en 03 antes de validar esa clase de fixtures. |
| Umbral de obsolescencia de datos | Bloquea probar el valor/categoría stale/fresh concreta; no bloquea comprobar que el panel distingue última consulta e ingesta. | Acordar en [05-api-contratos.md](05-api-contratos.md) §5 antes de la prueba de aceptación correspondiente. |
| Paginación y límite numérico | Bloquea probar la navegación y el límite específicos; no bloquea verificar el requisito general de `truncated`, total y que el panel no derive estadísticas de una muestra. | Completar en 05 §3.2 y después validar controles de interfaz según 06. |
| Formato de capturas/evidencias y ubicación exacta | No bloquea pruebas ni entregables; afecta a su organización y facilidad de revisión. | Elegir al preparar la memoria/README, sin crear un entregable separado no exigido. |
| Fecha calendario exacta de entrega | Bloquea fijar una fecha exacta de calendario, no la preparación técnica ni las pruebas. | Esperar confirmación del estudiante/docente; no deducirla del plazo comunicado de 20 días. |
| Resultado real de build/despliegue del proyecto | Bloquea afirmar que el proyecto está desplegado/probado; no bloquea documentar criterios. | Ejecutar en implementación futura y registrar evidencia. `VALIDACION.md` solo cubre comprobaciones declaradas del material base. |


# Ingesta y procesamiento

**Estado:** borrador pendiente de revisión del estudiante. Esta especificación organiza los requisitos de extracción, transformación y procesamiento citando las fuentes; no acredita implementación ni aprueba decisiones técnicas pendientes.

## 1. Propósito y límites

Esta spec concreta el flujo desde las fuentes de datos hasta el procesamiento y la persistencia, distinguiendo extracción live, carga histórica y replay demostrativo.

Se limita a los comportamientos y datos requeridos por los documentos del profesor y a las aclaraciones confirmadas del estudiante. No implementa el pipeline ni fija:

- el contrato detallado de `/api/estadisticas` o `/api/salud`;
- el esquema físico final, tablas, SQL o índices;
- nombres de topics, particiones, formato de serialización o configuración exacta de Kafka;
- duración y mecanismo concreto de retención;
- estrategia técnica única de carga inicial o recuperación;
- regla de desempate para puntos en fronteras o coincidencias espaciales múltiples.

Fuentes principales: [ENUNCIADO](../source-of-truth/ENUNCIADO.md), §§4–7 y 11; [CONTRATO-API](../source-of-truth/CONTRATO-API.md), `GET /api/eventos` cuando afecta a nulos y geometría. La jerarquía documental se mantiene según [README de specs](README.md).

## 2. Fuentes y alcance

### 2.1 Feed live de USGS

**Requisito del profesor:** consultar mediante HTTP GET el feed `all_day.geojson` aproximadamente cada 60 segundos. Cada respuesta contiene una ventana móvil de las últimas 24 horas; no es el día natural anterior ni una conexión push. Los eventos pueden publicarse con retraso y revisarse. (ENUNCIADO, §4.1.)

Los campos de origen que se deben reconocer/documentar son `id`, `properties.time`, `properties.updated`, `properties.mag`, `properties.magType`, `properties.type`, `properties.place` y `geometry.coordinates`. Los tiempos de USGS son milisegundos desde Unix epoch; las coordenadas GeoJSON son longitud, latitud y profundidad en km. (ENUNCIADO, §4.1.)

El análisis sísmico live selecciona `properties.type == "earthquake"` y conserva el motivo de exclusión de otros tipos. (ENUNCIADO, §4.1.) **No se aplican al feed live los filtros históricos `min_mag >= 4` ni el ámbito mundial como limitación adicional.** Esto último mantiene la aclaración del estudiante y la propuesta histórica diferenciadas: ver [00-principios-y-alcance.md](00-principios-y-alcance.md#supuestos-documentales-y-preguntas-abiertas), DEC-P-05.

### 2.2 Catálogo histórico de USGS

**Aclaración confirmada por el estudiante sobre la propuesta del profesor:** el conjunto histórico de trabajo es año 2025, ámbito mundial, magnitud igual o superior a 4 y tipo `earthquake`. Se usa para trabajo, filtros, pruebas y comparación histórica, no para restringir la ingesta live. (Propuesta del profesor: ENUNCIADO, §4.2; comparación: §8; aclaración: DEC-P-05 en [00](00-principios-y-alcance.md#supuestos-documentales-y-preguntas-abiertas).)

**Requisitos del profesor para extracción histórica:** descargar un año completo; dividir consultas por meses y subdividir intervalos si una consulta supera el límite documentado de 20.000 resultados; no truncar silenciosamente; deduplicar eventos repetidos en límites temporales; registrar consultas, fecha de extracción y número de registros. Descargar una vez y reutilizar el conjunto. (ENUNCIADO, §4.2.)

El catálogo es una instantánea en la fecha de extracción, no un registro completo de todas las revisiones pasadas. La comparación debe usar filtros equivalentes para periodos reciente e histórico; la media diaria histórica incluye los días con cero eventos. (ENUNCIADO, §§4.2 y 8; FAQ, pregunta 23 como aclaración orientativa.)

### 2.3 Catálogo Natural Earth

Se utiliza la copia entregada para enriquecer eventos mediante integración espacial, además de su uso cartográfico en el frontend. Contiene polígonos y atributos como `ADM0_A3`, `NAME_ES`, `ADMIN`, `CONTINENT` y `SUBREGION`. (ENUNCIADO, §4.3; [FUENTES-Y-LICENCIAS](../source-of-truth/FUENTES-Y-LICENCIAS.md), «Catálogo incluido».)

El proveedor describe `ADM0_A3` como identificador de entidad Natural Earth, no como código ISO oficial universal. Las fronteras son simplificadas; no deben usarse como jurisdicción legal ni para inferir con precisión eventos costeros. (ENUNCIADO, §4.3; FUENTES-Y-LICENCIAS, «Catálogo incluido».)

## 3. Flujo live

### 3.1 Extracción, versiones y publicación

1. **Consultar y registrar.** El productor consulta el feed aproximadamente cada 60 segundos y registra la extracción y errores relevantes. La ventana consultada es móvil de 24 horas. (Requisito del profesor: ENUNCIADO, §§4.1 y 5.)
2. **Filtrar tipo.** El productor/flujo live incluye los terremotos (`type == "earthquake"`) y registra el motivo de exclusión para otros tipos. No se inventan filtros adicionales de magnitud o área para live. (ENUNCIADO, §4.1; aclaración DEC-P-05 en [00](00-principios-y-alcance.md#supuestos-documentales-y-preguntas-abiertas).)
3. **Comparar estado.** Se contrasta `id` y `updated` con el estado persistido de versiones ya publicadas. Un evento nuevo y una revisión posterior se publican; una versión ya publicada se omite. (ENUNCIADO, §5.)
4. **Versión antigua.** No debe sustituir ni re-publicarse como novedad si existe una versión posterior. La FAQ explica expresamente que una versión recibida con `updated` anterior a la conocida no se publica como novedad; esta explicación aclara el control de revisiones del enunciado, sin fijar una estructura física. (ENUNCIADO, §§5 y 7; FAQ, pregunta 10.)
5. **Construir el mensaje.** La clave Kafka es el identificador de evento. El mensaje incluye `source`, `ingested_at`, `mode` (`live` o `replay`) y el evento original. (Requisito del profesor: ENUNCIADO, §5.)
6. **Esperar ACK.** El productor espera la confirmación de Kafka antes de marcar esa versión como publicada. Si ocurre una caída entre publicación y actualización de estado, puede producirse reenvío; el consumidor debe tolerarlo. (ENUNCIADO, §5.)

Los documentos no fijan los valores posibles de `source`, el nombre del topic, el formato de `ingested_at`, la serialización del mensaje, número de particiones ni semántica adicional del ACK. Esas decisiones no se completan aquí.

### 3.2 Consumo, persistencia y confirmación de offset

1. El consumidor lee el evento, valida y transforma explícitamente, enriquece geográficamente y persiste el resultado y las versiones necesarias. Los rechazos se conservan en cuarentena con motivo y referencia al original. (ENUNCIADO, §§4.3, 6 y 7.)
2. La escritura a PostgreSQL es transaccional; el offset de Kafka se confirma después de la escritura persistida. No se exige una transacción distribuida Kafka–PostgreSQL: el requisito es procesamiento al menos una vez con escritura idempotente. (ENUNCIADO, §7.)
3. El estado actual solo cambia cuando la versión recibida es más reciente; las versiones recibidas no deben reemplazar una versión posterior. El contrato de `GET /api/eventos` requiere que cada evento se presente una sola vez en su versión actual. (ENUNCIADO, §7; CONTRATO-API, `GET /api/eventos`.)
4. Reprocesar el mismo mensaje puede ocurrir tras una interrupción; no debe aumentar el número de eventos ni duplicar los efectos persistidos. Los mensajes, las versiones distintas observadas y los eventos actuales son granos distintos. (ENUNCIADO, §§5, 7 y 11; FAQ, preguntas 12–14.)
5. Un fallo de transformación no debe crear coordenadas inventadas: el dato no tratable se conserva en cuarentena con el motivo. El detalle de retries/reanudación, excepto el orden de commit indicado, queda pendiente de diseño. (ENUNCIADO, §§6–7; FAQ, pregunta 13.)

## 4. Carga histórica

### 4.1 Extracción y preparación

**Aclaración del estudiante:** usar el subconjunto histórico 2025/global/magnitud ≥4/tipo `earthquake` confirmado en §2.2; mantenerlo separado conceptualmente de la ingesta live.

**Requisitos del profesor:** ejecutar las consultas por meses; subdividir si se exceden 20.000 registros; deduplicar eventos compartidos entre límites de consulta; guardar consultas, fecha y conteos; reutilizar el conjunto descargado. (ENUNCIADO, §4.2.)

La extracción y preparación deben ser reproducibles y coherentes con las reglas de validación, versionado y enriquecimiento indicadas en este documento. El histórico no constituye un registro exhaustivo de revisiones retrospectivas. Si el productor live se detiene más de 24 horas, el enunciado indica explicar cómo recuperar el hueco con el catálogo histórico. (ENUNCIADO, §§4.2 y 5.)

### 4.2 Carga inicial frente a replay demostrativo

- **Carga inicial histórica:** ingesta única para disponer de datos del conjunto histórico y reutilizarlos. El profesor exige el conjunto y la procedencia; no prescribe un único mecanismo para cargarlo en almacenamiento. La FAQ permite batch o reutilizar lógica común, y advierte que las pruebas del replay sí deben pasar por Kafka. La FAQ es orientativa, no amplía los requisitos. (ENUNCIADO, §§4.2, 5 y 12; FAQ, pregunta 21.)
- **Replay demostrativo:** flujo acelerado para mostrar actividad durante la defensa. Sí debe procesarse por Kafka, con orden temporal, etiqueta de reproducción, tiempos de evento y reproducción y aislamiento respecto a live. (Requisito del profesor: ENUNCIADO, §11.)

El mecanismo de carga histórica inicial sigue siendo una decisión de diseño; no se debe inferir que carga inicial y replay son el mismo flujo o tienen las mismas condiciones de aislamiento.

## 5. Validación, transformación y cuarentena

El consumidor aplica reglas explícitas de validación y transformación. El análisis de calidad se documenta en memoria o notebook con código/consultas, resultados antes/después, decisiones y limitaciones; no se exige generador automático ni panel de calidad. (ENUNCIADO, §6; FAQ, pregunta 17.)

| Aspecto | Regla trazada a las fuentes |
|---|---|
| Tipo de evento | Para live, seleccionar `earthquake`; registrar motivo de exclusión de otros tipos. (ENUNCIADO, §4.1.) |
| Geometría y coordenadas | El endpoint de eventos requiere geometría Point válida; coordenadas GeoJSON en orden longitud, latitud, profundidad (km). Coordenadas inválidas no se convierten en puntos inventados; registrar rechazo y motivo. (ENUNCIADO, §§4.1 y 6; CONTRATO-API, `GET /api/eventos`; prueba, ENUNCIADO, §11.) |
| Tiempo | `time` y `updated` llegan en milisegundos Unix UTC. Convertir/documentar UTC sin confundir ocurrencia, actualización, extracción e ingesta. (ENUNCIADO, §§4.1 y 6.) |
| Magnitud y profundidad nulas | Conservar desconocidos como nulos; no rellenar con cero. La magnitud nula no cumple `min_mag` y se contabiliza aparte. (ENUNCIADO, §6; CONTRATO-API, `GET /api/eventos`.) |
| Valores negativos o inusuales | Magnitud negativa no es automáticamente error; profundidad negativa o valor inusual no se rechaza sin estudiar y justificar su significado. No se establecen aquí nuevos límites de validez. (ENUNCIADO, §6; FAQ, pregunta 16.) |
| Rechazos | Conservar duraderamente el original o referencia inequívoca y el motivo de rechazo. No perder registros rechazados silenciosamente. (ENUNCIADO, §6; FAQ, pregunta 13.) |
| Datos sintéticos de prueba | Mantener casos artificialmente dañados separados e identificados; no presentarlos como hallazgos de datos reales. (ENUNCIADO, §6.) |
| Cambios de transformación | Registrar qué transformaciones cambian datos y cuántos registros afectan. (ENUNCIADO, §6.) |

Las fuentes indican ejemplos de validez como tipos, fechas parseables, coordenadas finitas/en rango y geometría Point (ENUNCIADO, §6). No se fijan aquí rangos numéricos adicionales para magnitud/profundidad ni tolerancias nuevas.

## 6. Enriquecimiento geográfico

El consumidor integra cada evento con Natural Earth mediante operación espacial punto-en-polígono, usando un sistema compatible WGS84/EPSG:4326. Se verifica unicidad de `ADM0_A3` antes de usarlo como identificador y se conservan atributos disponibles del catálogo. (Requisito del profesor: ENUNCIADO, §4.3.)

No se infiere país a partir del texto de `place`; tampoco se asigna automáticamente el país más cercano. Un evento marítimo o sin polígono coincidente se conserva como «Sin país asignado», porque las costas son simplificadas y la cobertura geográfica puede ser incompleta; no es automáticamente un rechazo. (ENUNCIADO, §4.3.)

**Decisión pendiente DEC-P-04:** definir y documentar el tratamiento de puntos sobre fronteras o con varias coincidencias. El enunciado exige que exista una regla, pero no la prescribe; no consta una propuesta aprobada en las specs existentes. Hasta resolverla, no se presenta ninguna opción como requisito ni como decisión.

Fuente adicional sobre límites: FUENTES-Y-LICENCIAS, «Catálogo incluido», que advierte de fronteras simplificadas, limitaciones costeras y ausencia de algunos territorios pequeños.

## 7. Persistencia, versiones y recuperación

- **Originales y procedencia:** conservar el GeoJSON original y metadatos de extracción antes de transformar; registrar consultas, fecha y número de registros del histórico. Evitar copias idénticas indefinidas mediante una retención que aún debe definirse. (ENUNCIADO, §§4.2 y 5.)
- **Versiones y estado actual:** conservar información suficiente para distinguir las versiones observadas y servir una sola versión actual por evento. Solo una revisión posterior puede sustituir el estado actual. (ENUNCIADO, §7; CONTRATO-API, `GET /api/eventos`.)
- **Idempotencia:** procesar al menos una vez y hacer persistencia idempotente; completar la escritura transaccional antes del commit de offset. (ENUNCIADO, §7.)
- **Salida del feed:** no borrar el evento cuando desaparece de la ventana móvil de 24 horas; eso no demuestra que USGS lo haya eliminado. La reconciliación exhaustiva de retirados/fusionados fuera de ventana es una ampliación, y debe documentarse como límite. (ENUNCIADO, §5.)
- **Hueco de ingesta:** tras una parada superior a 24 horas, explicar el procedimiento de recuperación mediante el catálogo histórico. El procedimiento concreto, intervalos y controles de duplicados quedan por diseñar. (ENUNCIADO, §5.)
- **Retención:** no se fija duración para originales, histórico, versiones o cuarentena. Kafka debe tener retención limitada, pero la duración no está indicada. (ENUNCIADO, §§5, 6 y 9; DEC-P-RET en [03-modelo-datos-calidad.md](03-modelo-datos-calidad.md#12-decisiones-abiertas).)

Esta spec no define SQL, tablas ni políticas concretas de purga.

## 8. Replay

**Requisito del profesor:** para asegurar una demostración con actividad, reproducir de forma acelerada una selección histórica ordenada por fecha; conservar el tiempo original del evento y añadir el tiempo de reproducción; utilizar Kafka; separar replay de live mediante topics y tablas/esquema separados o mecanismo equivalente; etiquetar claramente «REPRODUCCIÓN»; reutilizar para el productor de replay el procesamiento del consumidor; incluir casos controlados de duplicado y revisión. (ENUNCIADO, §11.)

Si la selección es del año 2025, la consulta/vista de replay debe usar el intervalo histórico correspondiente; el filtro por defecto de últimas 24 horas no mostrará esos eventos en la fecha de demostración. Mantener el tiempo original, no modificarlo para forzar su aparición. (FAQ, pregunta 32, como orientación del profesor.)

**Decisión pendiente DEC-P-03d:** topic separado, tablas/esquema separados u opción equivalente son alternativas mencionadas por el enunciado, no decisión aprobada. También siguen pendientes los nombres de topics, la partición de datos y la consulta de replay en el panel. Esta spec no elige mecanismo.

## 9. Errores y recuperación

| Situación | Comportamiento exigido por las fuentes | Diseño operativo que sigue pendiente |
|---|---|---|
| HTTP timeout, 429/5xx o error JSON de USGS | Aplicar reintentos con espera progresiva, registrar error y no borrar el estado válido anterior. (ENUNCIADO, §5.) | Duración de timeout, número de reintentos, límites/backoff y estado final tras agotarlos no están fijados. |
| Kafka no disponible para productor | No marcar versión como publicada sin confirmación Kafka. (ENUNCIADO, §5.) | Política de retry, espera, pausa y alerta del productor no especificada. |
| PostgreSQL no disponible para consumidor | No confirmar offset hasta persistir; conservar posibilidad de reentrega y efecto idempotente. (ENUNCIADO, §7.) | Retry/backoff, pausa del consumo o supervisión no especificados. No se prescribe DLQ. |
| Reenvío tras ACK Kafka/caída de productor | Puede ocurrir; consumidor tolera reenvío sin duplicar efectos. (ENUNCIADO, §5.) | El estado de publicación y su actualización concreta no se diseñan aquí. |
| Reenvío tras caída del consumidor | Commit de PostgreSQL antes de confirmar offset; reinicio reanuda sin duplicar efectos. (ENUNCIADO, §§7 y 11; FAQ, pregunta 13.) | Configuración de commits automáticos/manuales y punto exacto de interrupción de pruebas pendientes. |
| Evento inválido | Registrar cuarentena duradera con motivo/referencia al original antes de avanzar de forma que el rechazo no se pierda. (ENUNCIADO, §6; FAQ, pregunta 13.) | Esquema de cuarentena y estrategia ante fallo al escribir rechazo pendientes. |
| Evento fuera de ventana live | Mantener historial; no interpretarlo como borrado por USGS. (ENUNCIADO, §5.) | Reconciliación exhaustiva fuera del alcance mínimo. |
| Reinicio tras más de 24 horas | Explicar recuperación del hueco mediante catálogo histórico. (ENUNCIADO, §5.) | Lógica de reconciliación, partición temporal y aislamiento de importación pendientes. |

La disponibilidad de Kafka/PostgreSQL debe cubrirse además con health checks y reintentos de dependencias en Compose; el orden de arranque no implica que estén preparados. (ENUNCIADO, §2.) No se fijan umbrales, tiempos ni herramientas.

## 10. Criterios de aceptación y trazabilidad

Los resultados de esta tabla son criterios esperados, no evidencia de pruebas ya ejecutadas. Los identificadores referencian [01-requisitos-aceptacion.md](01-requisitos-aceptacion.md) y no se renombran.

| Escenario | Resultado esperado según fuentes | Requisitos existentes | Evidencia/prueba verificable |
|---|---|---|---|
| Evento nuevo live | Una fila/evento actual llega por Kafka, se persiste y queda visible en API/análisis; se conserva `id` y procedencia. (ENUNCIADO, §§5, 7 y 11.) | REQ-F-001, REQ-F-003, REQ-TEST-001 | Inyectar/seleccionar un ID nuevo; contrastar mensaje, estado actual, respuesta API y conteos. |
| Misma versión duplicada | No se publica como nueva si ya es conocida; si se reentrega al consumidor, no aumenta eventos ni efectos. (ENUNCIADO, §§5, 7 y 11; FAQ, preguntas 10 y 12.) | REQ-F-003, REQ-F-007, REQ-NF-002, REQ-TEST-002 | Reenviar el mismo `id` y `updated`; comparar conteo actual y versiones antes/después. |
| Revisión posterior | Se registra la versión observada más reciente, actualiza el evento actual y refleja cambios en agregaciones afectadas sin sumar un evento adicional. (ENUNCIADO, §§5, 7–8 y 11.) | REQ-F-003, REQ-F-007, REQ-TEST-002 | Enviar misma identidad con `updated` posterior y comprobar estado/ventanas. |
| Versión antigua después de una nueva | No reemplaza la versión actual más reciente ni se publica como novedad atrasada. (ENUNCIADO, §7; FAQ, pregunta 10.) | REQ-F-007, REQ-TEST-002 | Enviar secuencia nueva→antigua; comparar `updated`/campos del estado actual e historial. |
| Magnitud nula | Se conserva como nula, no como cero; no cumple filtro `min_mag` y se contabiliza aparte. (CONTRATO-API, `GET /api/eventos`; ENUNCIADO, §6.) | REQ-DATA-002, REQ-TEST-003 | Fixture con `mag=null`; comprobar persistencia, filtro y representación `null`/ND según superficie. |
| Coordenadas inválidas | Registro rechazado en cuarentena con motivo; no aparece un punto inventado. (ENUNCIADO, §§6 y 11.) | REQ-F-006, REQ-QUAL-001, REQ-TEST-003 | Fixture sintético fuera de validez documentada; inspeccionar original/motivo y ausencia en geometrías válidas. |
| Evento marítimo | Permanece en los datos/análisis con «Sin país asignado»; no se imputa el país más cercano. (ENUNCIADO, §§4.3, 8 y 11.) | REQ-DATA-003, REQ-TEST-003 | Evento válido sin polígono contenedor; revisar país y conteos por categoría. |
| Reinicio del consumidor | Reanuda el consumo y no duplica efectos que ya fueron persistidos. (ENUNCIADO, §11.) | REQ-NF-002, REQ-ARCH-003, REQ-TEST-004 | Detenerlo en una prueba controlada, reiniciar y comparar filas/offsets/contadores. |
| Error del feed | Timeout/429/5xx/JSON inválido se registra y reintenta con espera progresiva; estado válido previo se conserva. (ENUNCIADO, §5.) | REQ-NF-001 | Simular cada clase de respuesta y revisar logs, intentos y estado previo. |
| Replay separado | Replay acelerado pasa por Kafka, conserva tiempo de evento, añade tiempo de reproducción, se rotula «REPRODUCCIÓN» y no se mezcla con live. (ENUNCIADO, §11.) | REQ-F-013, REQ-DATA-005, REQ-TEST-006 | Ejecutar dataset marcado replay; comprobar orden, ambos tiempos, etiqueta y aislamiento con mecanismo aún por elegir. |
| Recuperar hueco >24 h | Procedimiento explica uso del catálogo histórico sin truncado silencioso ni duplicación de eventos. (ENUNCIADO, §§4.2 y 5.) | REQ-F-004, REQ-F-007, REQ-DATA-001 | Simular pausa documentada y contrastar intervalo recuperado, consultas registradas y unicidad final. |

## 11. Decisiones pendientes

Las decisiones de esta tabla no bloquean la planificación general salvo que se indique lo contrario; bloquean únicamente la implementación o configuración que dependa de ellas. Ninguna alternativa se considera aprobada.

| Decisión | Estado y requisito afectado | Resolver en | Efecto |
|---|---|---|---|
| Regla para puntos en fronteras/múltiples coincidencias (DEC-P-04) | No decidida. El profesor exige documentar una regla espacial, no cuál. REQ-DATA-003, REQ-TEST-003. | Revisar en `03-modelo-datos-calidad.md` antes de cerrar geocodificación. | No bloquea planificar; bloquea cerrar y probar el comportamiento definitivo de casos limítrofes. |
| Duración/estrategia de retención de originales, histórico, versiones, cuarentena y Kafka (DEC-P-RET) | Duraciones no especificadas. El profesor exige preservar originales, definir retención y limitar Kafka. REQ-DATA-001, REQ-DEP-001. | `03-modelo-datos-calidad.md` para datos; `02-arquitectura-flujos.md` para Kafka; decidir/configurar al preparar persistencia. | No bloquea planificar; bloquea políticas de purga y configuración de retención. |
| Nombre(s), particiones y configuración de topics Kafka (DEC-P-03c) | No especificados. Productor usa topic por configuración; no se asigna nombre aquí. REQ-F-003, REQ-DATA-005, REQ-ARCH-001. | Revisar `02-arquitectura-flujos.md` y resolver en tarea de integración/configuración Kafka. | No bloquea planificar; bloquea integración/configuración ejecutable. |
| Serialización y formato/unidad de `ingested_at` y metadatos | No fijados; semántica del campo sí es requerida. REQ-DATA-002, REQ-DATA-005. | `03-modelo-datos-calidad.md`, coordinado con contrato/mensaje cuando se especifique. | No bloquea planificar; bloquea fijar esquema del mensaje e interoperabilidad productor-consumidor. |
| Mecanismo operativo de carga inicial histórica | El profesor fija periodo, consultas y procedencia, no un mecanismo único. REQ-F-004, REQ-DATA-001. | Revisar `02-arquitectura-flujos.md`; resolver antes de implementar el cargador. | No bloquea planificar; bloquea escoger e implementar ruta de carga. |
| Recuperación de huecos después de parada >24 h | La necesidad de explicar recuperación con catálogo histórico es docente; estrategia de deduplicación/reconciliación concreta no fijada. REQ-F-004, REQ-F-007. | Revisar `02-arquitectura-flujos.md` y `03-modelo-datos-calidad.md` durante diseño de carga/persistencia. | No bloquea planificar; bloquea implementar recuperación reproducible. |
| Aislamiento concreto de replay (DEC-P-03d) | Topic/schema/tablas separados o mecanismo equivalente son opciones docentes, no decisión aprobada. REQ-F-013, REQ-DATA-005, REQ-TEST-006. | `02-arquitectura-flujos.md` y coordinación con `03-modelo-datos-calidad.md`. | No bloquea planificar; bloquea integración y prueba de replay sin mezcla. |
| Reglas de retry/backoff y recuperación frente a Kafka/PostgreSQL no disponibles | Comportamientos de ACK, commit e idempotencia sí son obligatorios; tiempos/número de reintentos y política de pausa no especificados. REQ-NF-001, REQ-NF-002, REQ-ARCH-003. | Resolver como detalle de implementación/configuración al implementar productores y consumidor, preservando requisitos. | No bloquea planificar; bloquea cerrar configuración resiliente, no el diseño funcional. |

El diseño de contrato detallado de estadísticas/salud queda fuera de esta spec y corresponde a una propuesta posterior, según aclaración del estudiante (DEC-P-01/02). El esquema físico y SQL también quedan fuera y siguen pendientes según DEC-P-03a/03b.

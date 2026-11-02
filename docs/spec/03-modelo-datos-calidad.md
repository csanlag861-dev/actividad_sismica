# Modelo de datos y calidad

**Estado:** borrador de especificación pendiente de revisión del estudiante. Define conceptos, granos, campos y reglas requeridas; no decide el esquema físico, el SQL, los índices ni una política cuantitativa de retención.

## 1. Objetivo y alcance

Establecer qué información debe conservarse, cómo se distingue cada unidad de datos y qué controles de calidad son necesarios para que el flujo sea trazable, reejecutable e idempotente. Se relaciona con la extracción del productor, el procesamiento del consumidor y las consultas de la API descritas en [02-arquitectura-flujos.md](02-arquitectura-flujos.md).

El enunciado requiere conservar GeoJSON original y metadatos de extracción antes de transformar, validar/enriquecer en el consumidor, almacenar estado persistente, presentar un modelo conceptual/lógico/físico y registrar cuarentena. (Requisito docente: [ENUNCIADO](../source-of-truth/ENUNCIADO.md), §§5–7 y 12.) No prescribe un diseño físico de tablas. La propuesta de conceptos de esta spec no es un esquema aprobado.

## 2. Entidades y granos

| Concepto | Grano que debe poder distinguirse | Identificadores/datos relevantes | Estado |
|---|---|---|---|
| **Evento sísmico** | Una identidad de evento USGS, con independencia del número de mensajes o versiones. | `id` de USGS. | Concepto requerido para evitar contar mensajes como eventos. (ENUNCIADO, §§5, 7 y 11.) |
| **Versión recibida** | Una observación/versionado de un mismo evento fuente. La identidad de negocio se basa en `id` y `updated`; debe poder reconocerse una repetición exacta. | `id`, `updated`, evento original, `source`, `ingested_at`, `mode`. | Requerido por el control de revisiones; clave física exacta se deja pendiente. (ENUNCIADO, §§5 y 7.) |
| **Respuesta/registro original de extracción** | Una respuesta o conjunto obtenido en una consulta a una fuente, antes de transformar; debe asociarse a metadatos de extracción. | Fuente/URL o parámetros consultados, fecha de extracción, conteo, contenido original. | Requerido; formato de archivo/almacenamiento no fijado. (ENUNCIADO, §§4.2 y 5.) |
| **Metadatos de extracción** | Una ejecución/consulta de extracción, distinguible de los registros devueltos. | Fuente, consulta/parámetros, fecha, número de registros y resultado/error relevante. | Requerido documentar consultas, fecha y conteos para histórico; detalle de esquema pendiente. (ENUNCIADO, §4.2.) |
| **Evento actual consultable** | Como máximo una versión actual por identidad de evento en el ámbito de consulta correspondiente. | `id`, campos normalizados, última `updated`, país asignado si procede, modo/ámbito. | Requisito del contrato de eventos y del tratamiento de revisiones. (CONTRATO-API, `GET /api/eventos`; ENUNCIADO, §7.) |
| **Entidad geográfica** | Una entidad de la copia Natural Earth. | `ADM0_A3`, `NAME_ES`, `ADMIN`, `CONTINENT`, `SUBREGION` cuando estén presentes. | Fuente geográfica entregada; integridad/unicidad del identificador se comprueba, no se presupone. (ENUNCIADO, §4.3; FUENTES-Y-LICENCIAS, «Catálogo incluido».) |
| **Rechazo / cuarentena** | Un registro o mensaje que no pudo tratarse conforme a reglas y su causa concreta. | Referencia al original/mensaje, motivo, contexto de procesamiento y procedencia disponible. | Debe conservarse de forma duradera con motivo; representación y grano físico pendientes. (ENUNCIADO, §6; FAQ, pregunta 13.) |
| **Dato de replay** | Una versión/evento procesado en modo reproducción, distinguible de live aunque conserve el tiempo original del evento. | `mode=replay`, tiempo de evento, tiempo de reproducción, `id` y original. | Separación y rotulación requeridas; mecanismo y campo exacto pendientes. (ENUNCIADO, §§5 y 11.) |

Las entidades descritas no obligan a tener una tabla por concepto. Las fuentes permiten modelos equivalentes si se documentan grano, claves, relaciones e índices (ENUNCIADO, §7; guía de entregables, §2.5).

## 3. Campos de origen, unidades y tiempos

### 3.1 Campos del evento USGS

| Campo | Significado y tratamiento requerido | Fuente |
|---|---|---|
| `id` | Identificador del evento; se conserva a través de revisiones y sirve para no confundir mensajes con eventos. | ENUNCIADO, §§4.1 y 5. |
| `properties.updated` / `updated` | Momento de actualización de esta versión, distinto de ocurrencia e ingesta. Se usa en la comparación de versiones. | ENUNCIADO, §§4.1 y 5. |
| `properties.time` / `time` | Tiempo de ocurrencia del evento; milisegundos desde Unix epoch y UTC. Se usa para intervalos y ventanas analíticas. | ENUNCIADO, §§4.1 y 8. |
| `properties.mag` / `mag` | Magnitud, puede faltar/ser nula. No convertir desconocido en cero. | ENUNCIADO, §6; CONTRATO-API, `GET /api/eventos`. |
| `properties.magType` / `magType` | Tipo de magnitud; conservar su significado y no asumir que escalas distintas sean intercambiables sin justificación. | ENUNCIADO, §4.1; FAQ, pregunta 9. |
| `properties.type` / `type` | Tipo de evento. La ingesta live del análisis sísmico selecciona `earthquake` y registra el motivo de exclusión de otros tipos. | ENUNCIADO, §4.1. |
| `properties.place` / `place` | Texto descriptivo, no fuente para calcular país. | ENUNCIADO, §4.3. |
| `geometry.coordinates` | GeoJSON [longitud, latitud, profundidad en km]. Validar finitud/rangos de coordenadas y geometría Point; la profundidad desconocida puede ser nula. | ENUNCIADO, §§4.1 y 6; CONTRATO-API, `GET /api/eventos`. |
| `source` | Origen de la publicación/mensaje, según el envelope exigido. Valor/conjunto de valores no se fija en estas fuentes. | ENUNCIADO, §5. |
| `ingested_at` | Tiempo de recepción/ingesta, distinto del tiempo de evento y `updated`. Unidad/serialización exacta requiere convención posterior; no se confunde con timestamps fuente. | ENUNCIADO, §5; §6. |
| `mode` | Identifica `live` o `replay`. El histórico de trabajo y replay deben mantenerse distinguibles del feed live. | ENUNCIADO, §5 y §11. |
| Tiempo de reproducción | Instante adicional de reproducción; se conserva junto con el tiempo original. El nombre/unidad exactos del campo no están especificados. | ENUNCIADO, §11. |

Los tiempos fuente son milisegundos Unix UTC. En la API de eventos se devuelven tiempos en milisegundos Unix UTC. Si se normalizan internamente a otro tipo temporal, la conversión y unidad deben documentarse y ser verificables. (ENUNCIADO, §4.1; CONTRATO-API, `GET /api/eventos`.)

## 4. Nulabilidad y valores desconocidos

- `mag`, profundidad y país pueden ser desconocidos y se conservan como `null`; nunca se serializan NaN/Infinity en el endpoint de eventos. (CONTRATO-API, `GET /api/eventos`.)
- La magnitud nula no satisface `min_mag` y se contabiliza por separado. (CONTRATO-API, `GET /api/eventos`.)
- Las distribuciones deben indicar registros sin valor; los cálculos que requieren magnitud/profundidad explicitan cuántas observaciones excluyen. (ENUNCIADO, §8; CONTRATO-API, «Otros endpoints a implementar».)
- No se rellena indiscriminadamente con cero. Una magnitud negativa no es automáticamente inválida; profundidad negativa no debe rechazarse sin investigar su significado. (ENUNCIADO, §6.)
- `country_id`/país puede no estar asignado. «Sin país asignado» se mantiene como categoría analítica válida, no como país imputado ni rechazo automático. (ENUNCIADO, §4.3 y §8.)
- Geometría válida Point es obligatoria para `GET /api/eventos`; un registro con coordenadas inválidas se pone en cuarentena con motivo y no se transforma en un punto inventado. (CONTRATO-API, `GET /api/eventos`; ENUNCIADO, §11.)

## 5. Reglas de calidad y transformación

El consumidor aplica reglas explícitas; el informe de calidad puede estar en memoria o notebook, con consultas/código, resultados antes/después, decisiones y limitaciones. No se exige generador automático ni panel de calidad. La cuarentena duradera sí se conserva. (Requisito docente: ENUNCIADO, §6; FAQ, pregunta 17 como aclaración.)

| Dimensión | Control/evidencia | Tratamiento y límite |
|---|---|---|
| Completitud | Porcentaje disponible por campo; identificar ausencias de magnitud y descripción/localización. | Mantener nulos cuando sean desconocidos; informar denominador. (ENUNCIADO, §6.) |
| Unicidad | Duplicados de versión y unicidad de eventos del estado actual. | Distinguir mensajes repetidos, versiones distintas y eventos. No sumar cada mensaje como terremoto. (ENUNCIADO, §§6–7.) |
| Validez | Tipos, fechas parseables, coordenadas finitas y dentro de rango, geometría Point. | Transformar fechas a UTC; rechazar datos que no puedan tratarse y conservarlos con motivo. No inventar localización. (ENUNCIADO, §6; CONTRATO-API, `GET /api/eventos`.) |
| Consistencia | Coherencia de unidades, claves geográficas, relación entre `id`/`updated` y orden de revisiones. | Conservar longitud/latitud/profundidad en orden/unidades documentados; actualizar estado actual solo si revisión es más reciente. (ENUNCIADO, §§4.1, 4.3 y 7.) |
| Actualidad | Registrar y comparar tiempo de ocurrencia, `updated`, ingesta, extracción/consulta y generación de respuesta según aplique. | Diferenciar datos antiguos de respuesta recién generada; no confundir `generated` con ingesta activa. (ENUNCIADO, §8; CONTRATO-API, `GET /api/eventos`.) |
| Exactitud | Explicar qué puede contrastarse y qué no, incluidas limitaciones de USGS y del catálogo espacial simplificado. | Validar formato/rangos no prueba exactitud física; declarar límites en el análisis. (ENUNCIADO, §6; FUENTES-Y-LICENCIAS, «Catálogo incluido».) |

Reglas adicionales de transformación:

- Guardar original antes de transformar y registrar qué transformación cambia datos y cuántos registros afecta. (ENUNCIADO, §§5–6.)
- Separar tiempo de evento de tiempo de proceso; mantener UTC y unidades declaradas. (ENUNCIADO, §6.)
- Los eventos `type != earthquake` excluidos del análisis sísmico se registran con motivo; los casos artificialmente dañados de prueba se distinguen de los datos reales. (ENUNCIADO, §§4.1 y 6.)
- No confundir desaparición del feed al salir de sus últimas 24 horas con eliminación por USGS. Mantener el histórico guardado y documentar el límite de reconciliación fuera de ventana. (ENUNCIADO, §5.)

## 6. Versionado e idempotencia

1. El productor compara `id` y `updated` con estado persistente: publica nuevos y revisiones posteriores; omite una versión ya publicada. La clave del mensaje Kafka es el ID del evento. (ENUNCIADO, §5.)
2. El consumidor conserva las versiones observadas sin sobrescribir una versión posterior con otra antigua. El estado actual consultable representa una sola versión actual por evento. (ENUNCIADO, §7; CONTRATO-API, `GET /api/eventos`.)
3. Reprocesar el mismo mensaje puede ocurrir; la escritura debe ser idempotente, preservando un solo efecto actual. Se guarda transaccionalmente y se confirma offset después de la escritura. No se exige transacción distribuida Kafka–PostgreSQL. (ENUNCIADO, §7.)
4. Una revisión puede cambiar magnitud, ubicación o tiempo de ocurrencia. Las agregaciones afectadas se recalculan o se derivan de estado actual; no se incrementan ciegamente por mensaje. (ENUNCIADO, §8.)
5. El número de mensajes recibidos, versiones distintas observadas y eventos actuales son granos distintos. La prueba controlada de duplicado/revisión debe demostrar esta diferencia. (ENUNCIADO, §11.)

La restricción física para identificar una versión (por ejemplo, qué columnas formarían una clave única) no está prescrita por las fuentes y queda pendiente del modelo lógico/físico. Los ejemplos de FAQ son orientación, no obligación. (ENUNCIADO, §7; FAQ, preguntas 12–13 y 20.)

## 7. Integración espacial

- Usar la copia Natural Earth entregada, con geometrías `Polygon`/`MultiPolygon`; mantener puntos y polígonos en WGS84/EPSG:4326. (ENUNCIADO, §4.3.)
- Integrar con operación espacial de punto dentro de polígono; no deducir país desde `place`. Se permite elegir librería compatible, pero no se prescribe una concreta. (ENUNCIADO, §4.3.)
- Comprobar unicidad de `ADM0_A3` antes de usarlo como identificador de la entidad. No etiquetarlo universalmente como ISO oficial. (ENUNCIADO, §4.3; FUENTES-Y-LICENCIAS, «Catálogo incluido».)
- Conservar país nulo/«Sin país asignado» para eventos que no coincidan; no usar país más cercano ni tratar automáticamente el caso como error. Causas posibles: mar, costas simplificadas o cobertura incompleta. (ENUNCIADO, §4.3.)
- El catálogo es cartográficamente simplificado, no delimita jurisdicción legal ni asigna con precisión eventos costeros y puede omitir territorios pequeños. (FUENTES-Y-LICENCIAS, «Catálogo incluido».)
- Puntos exactamente en frontera y múltiples coincidencias requieren una regla documentada. La regla concreta no está decidida; ver DEC-P-04. (ENUNCIADO, §4.3.)

## 8. Procedencia y retención

| Elemento | Necesidad documental | Decisión aún abierta |
|---|---|---|
| Feed live | Conservar originales/metadatos antes de transformar; registrar consultas/errores; evitar copias idénticas indefinidas y documentar retención. | Duración, medio de almacenamiento y política de deduplicación. (ENUNCIADO, §5.) |
| Histórico | Registrar URL/parámetros, fecha de extracción, número de registros y límites/subconsultas; descargar una vez y reutilizar. | Formato/ubicación exactos y procedimiento de regeneración concreto. El histórico es instantánea del catálogo. (ENUNCIADO, §4.2; §12.) |
| Versiones | Preservar versiones observadas suficientes para probar revisión/idempotencia y reconocer el estado actual más reciente. | Horizonte de retención de versiones no está fijado. (ENUNCIADO, §7.) |
| Cuarentena | Conservar duraderamente el registro rechazado y el motivo, con referencia al original; no perderlo silenciosamente. | Duración, formato, campos operativos y eventual purga no están fijados. (ENUNCIADO, §6; FAQ, pregunta 13.) |
| Replay | Mantener separación suficiente de live y conservar tiempo original y de reproducción. | Aislamiento y retención exactos pendientes. (ENUNCIADO, §11.) |

**DEC-P-RET — política concreta de retención:** pendiente de decisión. No se infiere duración para originales, histórico, versiones, cuarentena ni mensajes Kafka; el enunciado exige almacenamiento persistente, evitar copias idénticas indefinidas y retención limitada de Kafka, pero no proporciona duraciones. (ENUNCIADO, §§5 y 9.)

## 9. Candidatos conceptuales/lógicos (propuestas no aprobadas)

Las siguientes agrupaciones son una **propuesta técnica para ayudar a planificar**, basada en los granos exigidos y en orientaciones de ENUNCIADO §7. No obligan a crear tablas concretas, no son una decisión del profesor y requieren revisión del estudiante:

1. **Registro de extracción/original**: una entidad para consulta/respuesta bruta con metadatos y referencia al GeoJSON.
2. **Versión recibida**: una entidad por versión observada, asociada al evento y a su extracción/mensaje.
3. **Evento actual**: proyección consultable de la versión más reciente conocida, una por evento en el ámbito correspondiente.
4. **Catálogo geográfico**: entidades Natural Earth, identificadas por `ADM0_A3` tras comprobar unicidad.
5. **Cuarentena**: registro rechazado, referencia al original y motivo.
6. **Replay**: dimensión/ámbito de ejecución que mantiene replay distinguible de live; separación física o lógica por decidir.

El enunciado ofrece como orientación hecho de evento actual, dimensiones, historial de versiones y estado de ingesta/calidad, pero permite otro modelo si se explican grano, claves, relaciones e índices. (ENUNCIADO, §7.) No se escribe SQL ni se fijan claves físicas aquí.

## 10. Criterios de aceptación verificables

| ID existente | Criterio de aceptación para los datos |
|---|---|
| REQ-F-003 / REQ-F-007 | En secuencia de evento nuevo, duplicado, revisión posterior y versión antigua, hay un evento actual, la revisión más reciente prevalece y las versiones distintas quedan distinguibles. |
| REQ-F-006 / REQ-QUAL-001 | Una entrada inválida de prueba queda en cuarentena con original/referencia y motivo; no aparece como geometría inventada. |
| REQ-DATA-001 | Para extracción elegida se pueden reconstruir fuente, consulta/parámetros, fecha y conteo; el original se conserva antes de transformación. |
| REQ-DATA-002 | Conversión de timestamps mantiene UTC/unidad declarada; coordenadas respetan orden y profundidad km; desconocidos se conservan como `null`, no NaN/Infinity en API. |
| REQ-DATA-003 | Se verifica unicidad `ADM0_A3`, asignación espacial con CRS compatible y existencia de caso sin país; está escrita la regla de frontera antes de usarla como decisión. |
| REQ-DATA-004 / REQ-F-010 | Al comparar, filtros entre periodo reciente e histórico son equivalentes; se muestran periodo, unidad, nulos/valores sin dato y denominador histórico, incluidos días con cero. |
| REQ-QUAL-002 / REQ-DOC-003 | Evidencia de calidad identifica conjunto, periodo y denominador y presenta consulta/código, resultado, decisión y límite para las dimensiones analizadas. No se afirma que pruebas sintéticas sean datos reales. |
| REQ-F-013 / REQ-TEST-006 | Los registros replay se identifican, conservan tiempos de evento/reproducción y quedan aislados de live mediante el mecanismo que se acuerde. |

Estos son métodos previstos; no hay resultados de implementación o ejecución declarados en esta spec.

## 11. Trazabilidad

| Tema de esta spec | Requisitos existentes | Fuente docente principal |
|---|---|---|
| Evento/versiones/original y procedencia | REQ-F-003, REQ-F-004, REQ-DATA-001, REQ-DATA-002 | ENUNCIADO, §§4.1–4.2 y 5. |
| Actualización, duplicados e idempotencia | REQ-F-006, REQ-F-007, REQ-NF-002, REQ-TEST-001–004 | ENUNCIADO, §§5, 7 y 11. |
| Cuarentena y calidad | REQ-QUAL-001, REQ-QUAL-002, REQ-DOC-003, REQ-TEST-003 | ENUNCIADO, §6; rúbrica, §13. |
| Geografía y nulos | REQ-DATA-003, REQ-TEST-003, REQ-F-009 | ENUNCIADO, §§4.3, 6 y 8; FUENTES-Y-LICENCIAS, «Catálogo incluido». |
| Comparación histórica | REQ-F-005, REQ-F-010, REQ-DATA-004 | ENUNCIADO, §§4.2 y 8; aclaración DEC-P-05 en [00](00-principios-y-alcance.md#supuestos-documentales-y-preguntas-abiertas). |
| Replay | REQ-F-013, REQ-DATA-005, REQ-TEST-006 | ENUNCIADO, §§5 y 11. |

Los identificadores referencian [01-requisitos-aceptacion.md](01-requisitos-aceptacion.md) y permanecen sin cambios. No se añaden requisitos normativos nuevos en esta spec.

## 12. Decisiones abiertas

«No bloquea planificación» significa que se puede seguir definiendo conceptos y pruebas; «bloquea implementación» limita el aspecto concreto hasta cerrar y revisar la definición. Las rutas marcadas como futuras son destinos propuestos para resolver la pregunta, no archivos ya existentes ni autorización para crearlos en esta tarea.

| Decisión | Estado y autoridad | Spec donde resolver | Impacto |
|---|---|---|---|
| DEC-P-04 — puntos en frontera o con varias coincidencias | El profesor exige documentar una regla espacial, pero no prescribe cuál; no hay propuesta aprobada. (ENUNCIADO, §4.3.) | Revisión futura de esta spec `03-modelo-datos-calidad.md` | No bloquea planificación del modelo; bloquea cerrar y probar la asignación espacial para esos casos |
| DEC-P-RET — retención de originales, metadatos, versiones y cuarentena | El profesor exige conservar originales y definir retención; duración/política concreta no especificadas. Para Kafka se requiere retención limitada, sin duración fijada. (ENUNCIADO, §§5 y 9.) | Revisión futura de esta spec; coordinar retención del broker en `02-arquitectura-flujos.md` | No bloquea planificación conceptual; bloquea políticas de purga/retención y su configuración |
| DEC-P-03a — esquema físico y estructura final | Requisito docente: documentar grano, claves, relaciones e índices; no se prescribe diseño físico único. | Futura `docs/spec/05-modelo-logico-fisico.md` (no creada), partiendo de esta spec | No bloquea planificación conceptual; bloquea implementación de esquema/persistencia |
| DEC-P-03b — índices | Índices físicos no especificados; deben justificarse según consultas y mediciones. (ENUNCIADO, §§7 y 10.) | Futura `docs/spec/05-modelo-logico-fisico.md` (no creada) | No bloquea planificación general; bloquea selección/implementación de índices |
| DEC-P-03d — aislamiento de replay | Requisito docente de separar replay/live; topic, schema, tablas o mecanismo equivalente son alternativas no aprobadas. (ENUNCIADO, §11.) | `02-arquitectura-flujos.md` para estrategia; futura `docs/spec/05-modelo-logico-fisico.md` (no creada) si implica separación física | No bloquea planificación; bloquea integración e implementación del replay |
| Agregaciones materializadas | No se ha decidido si harán falta. El profesor permite recalcular ventanas afectadas o consultar el estado actual. No crear agregaciones físicas por suposición. (ENUNCIADO, §8.) | Futura `docs/spec/04-contratos-api.md` (no creada) para necesidades del endpoint; futura `docs/spec/05-modelo-logico-fisico.md` (no creada) solo si se elige materialización | No bloquea planificación analítica; bloquea diseñar/implementar tablas materializadas, si se adoptan |
| Representación/unidad interna de `ingested_at` y tiempo de replay | La semántica se requiere, pero formato/nombre/unidad interna concreta no están fijados. (ENUNCIADO, §§5 y 11.) | Futura `docs/spec/05-modelo-logico-fisico.md` (no creada), coordinada con `04-contratos-api.md` si se expone por API | No bloquea planificación del flujo; bloquea fijar esquema y serialización final |
| Datos geográficos de trabajo | Aclaración del estudiante: subconjunto 2025 mundial, magnitud ≥4, `earthquake`; no filtra live. Los filtros equivalentes y denominadores de comparación vienen del enunciado. (ENUNCIADO, §§4.2 y 8.) | Ya registrado en [00](00-principios-y-alcance.md#supuestos-documentales-y-preguntas-abiertas) y en esta spec, §11 | No es pendiente ni bloquea planificación; resultados concretos requieren extracción y pruebas |

Ninguna de estas decisiones se aprueba por incluirla en una propuesta conceptual. Los documentos futuros citados no se crean aquí; cualquier decisión requerirá revisión del estudiante antes de considerarse aprobada o implementarse.

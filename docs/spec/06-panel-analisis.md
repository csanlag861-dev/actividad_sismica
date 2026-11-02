# Panel y análisis

**Estado:** borrador pendiente de revisión del estudiante. Describe el comportamiento esperado del panel Angular/Leaflet a partir del enunciado y del contrato de [05-api-contratos.md](05-api-contratos.md), aprobado por el estudiante. No implementa el frontend ni declara ejecutadas pruebas.

## 1. Propósito y límites

Esta especificación define qué información y estados debe poder presentar el panel: exploración de eventos procesados, filtros, análisis temporal/geográfico, comparación histórica y estado/actualidad de los datos. El panel consulta la API, no USGS directamente. (Requisito del profesor: [ENUNCIADO](../source-of-truth/ENUNCIADO.md), §§2 y 8; [README del material base](../source-of-truth/README.md), §§3 y 5.)

El panel no implementa el pipeline ni redefine endpoints, parámetros o payloads. Para respuestas y semánticas HTTP, se remite a [05-api-contratos.md](05-api-contratos.md). Tampoco prescribe el diseño visual detallado, una biblioteca adicional de gráficas ni la configuración de despliegue. La arquitectura de servicios y la publicación SPA se describen en [02-arquitectura-flujos.md](02-arquitectura-flujos.md).

## 2. Usuarios y objetivos de uso

**Usuario contemplado por las fuentes:** quien consulta y presenta el observatorio durante el trabajo y la demostración del proyecto; no se definen perfiles adicionales.

El panel debe permitir:

- explorar eventos ya procesados y entender sus atributos;
- comparar periodos con filtros y denominadores visibles;
- reconocer el periodo y filtros de los datos mostrados;
- identificar si la API responde, cuándo se consultó correctamente y cuándo ocurrió la última ingesta correcta;
- advertir de datos desactualizados, de un backend inaccesible o de una respuesta parcial.

Estas capacidades proceden de ENUNCIADO, §8, y de la prueba de cierre/reapertura del túnel en §11. No se afirma que el estado visual sustituya las evidencias del backend.

## 3. Vistas y capacidades requeridas

Las capacidades siguientes son **requisitos del profesor**, salvo que se indique lo contrario (ENUNCIADO, §8):

1. **Mapa interactivo:** fondo cartográfico, fronteras, círculos y ficha consultable del terremoto.
2. **Codificación visual:** tamaño del símbolo por magnitud y color por profundidad; leyenda y tratamiento visible de valores nulos. El símbolo expresa una escala visual de magnitud, no energía ni área afectada (FAQ, pregunta 27).
3. **Recuentos:** por país o subregión, y por ventanas fijas de una hora UTC basadas en el tiempo de ocurrencia del evento. Incluir «Sin país asignado».
4. **Distribuciones:** magnitud y profundidad; indicar registros sin valor.
5. **Comparación histórica:** comparar periodos equivalentes bajo filtros equivalentes; incluir días con cero eventos en el denominador de la media diaria histórica y avisar que el periodo reciente puede estar incompleto por retrasos.
6. **Filtros:** fecha, magnitud y país/región, aplicados de forma coherente al mapa y a los gráficos/estadísticas.
7. **Estado del sistema:** último acceso correcto a la API, última ingesta correcta de USGS, periodo consultado y aviso cuando los datos estén desactualizados o el backend no responda.
8. **Tabla consultable:** alternativa al mapa y al color para explorar eventos.

Las conclusiones del proyecto deben estar respaldadas por cifras, periodo y filtros, comparar muestras equivalentes y no presentarse como predicción de terremotos ni evaluación de riesgo. El enunciado exige al menos tres conclusiones fundamentadas. (ENUNCIADO, §§8 y 12.)

## 4. Filtros y modos de consulta

La interacción sigue el contrato de [05-api-contratos.md](05-api-contratos.md); esta spec no añade nombres de parámetros ni reglas de servidor.

- El usuario puede establecer filtros de fecha, magnitud y país/región en las vistas correspondientes. Los filtros efectivos y el periodo se identifican junto a los resultados.
- Para eventos, se conserva la semántica aprobada en 05 §3: intervalo con inicio inclusivo y final exclusivo, magnitud nula que no satisface el filtro de magnitud y filtros de país conforme al contrato.
- Las estadísticas deben recibir filtros equivalentes a los aplicados al mapa/tabla; no se calcularán sobre la página visible del mapa ni sobre una respuesta parcial. (ENUNCIADO, §8; 05 §§3–4.)
- El periodo predeterminado de eventos es el definido por el contrato: las últimas 24 horas. Las consultas históricas y los periodos efectivos de estadísticas/comparación se muestran según 05; el panel no deriva silenciosamente otro periodo.
- **Aclaración del estudiante:** el subconjunto histórico de trabajo es 2025, ámbito mundial, magnitud ≥4 y tipo `earthquake`, para filtros, pruebas y comparación histórica. No restringe la ingesta live. Para comparar, se muestran los filtros efectivos equivalentes y la base histórica disponible, conforme a 05 §4.3 y a ENUNCIADO, §§4.2 y 8.
- Si una selección de mapa es más amplia que la cobertura del histórico, se presenta la comparación según la decisión aprobada en 05 §4.3; no se oculta que los filtros efectivos difieren ni se presenta una comparación no equivalente como si lo fuera.

Las formas de consultas y campos concretos se rigen por 05; si una capacidad no está cubierta por ese contrato, se registra como dependencia en §8 y no se inventa una respuesta.

## 5. Estados de interfaz

El frontend debe poder comunicar los estados observables de la tabla. Los nombres, textos exactos, iconos, colores y disposición son decisiones de diseño salvo que la fuente indique explícitamente el comportamiento.

| Estado | Comportamiento esperado y fuente |
|---|---|
| Carga inicial | Indicar que la consulta está en curso y distinguirla de una respuesta válida sin eventos. La presentación concreta es decisión de interfaz. |
| Datos disponibles | Mostrar resultados junto a su periodo y filtros efectivos; identificar la hora de lectura/generación y la última ingesta cuando estén disponibles. (ENUNCIADO, §8; CONTRATO-API, «GET /api/eventos» y 05 §§3, 5.) |
| Sin eventos | Mostrar el resultado vacío como consulta válida, no como fallo. En eventos se representa conforme al GeoJSON vacío de 05 §3. (CONTRATO-API, «GET /api/eventos».) |
| Error de API o túnel | Informar que no se pudo consultar; no presentar el fallo como cero eventos ni como éxito reciente. (ENUNCIADO, §§8 y 11; CONTRATO-API, «Otros endpoints a implementar».) |
| Última lectura conservada | Si se conserva la visualización previa tras un fallo, identificarla claramente como anterior y no actualizar su marca de tiempo como si fuera nueva. (ENUNCIADO, §§8 y 11; README del material base, §3.) |
| Datos desactualizados | Mostrar el estado cuando se conozca; el umbral numérico no se inventa. Su definición y la señal disponible dependen de la decisión pendiente de salud en 05 §5. |
| Respuesta parcial/truncada | Si el contrato de eventos devuelve `truncated`, informar que la colección mostrada es parcial y ofrecer continuación conforme a 05 §3. No calcular estadísticas globales con esa página. El tamaño de página y sus controles siguen condicionados por las decisiones pendientes allí. |
| Replay | Identificar visiblemente los datos de reproducción como **«REPRODUCCIÓN»**, conservando su distinción respecto al modo live. (ENUNCIADO, §11.) |

El frontend de apoyo programa la siguiente consulta después de finalizar la anterior, usa timeout y conserva la capa previa con un aviso si falla (README del material base, §3). El requisito de actualización periódica del mapa es aproximadamente cada 60 segundos (ENUNCIADO, §9); cambios a ese comportamiento necesitan revisión, no se prescriben aquí.

## 6. Mapa, tabla y accesibilidad de la información

### Información de evento

La ficha y la tabla deben permitir consultar los atributos que el contrato de eventos establece: `id`; magnitud (`mag`) y tipo (`magType`); lugar (`place`); tiempo de ocurrencia (`time`) y actualización (`updated`); país (`country_id`, `country_name`); y profundidad, procedente de la tercera coordenada GeoJSON. Las coordenadas se interpretan como longitud, latitud y profundidad en km; los tiempos se presentan conforme a UTC y las unidades del contrato. (CONTRATO-API, «GET /api/eventos» y [05](05-api-contratos.md) §3.)

Magnitud, profundidad y país desconocidos deben permanecer explícitamente desconocidos, no convertirse en cero ni en texto que sugiera un valor medido. El enunciado exige tratamiento visible de nulos; para magnitud nula, la prueba de aceptación prescribe su visualización como ND. País no asignado se puede presentar como «Sin país asignado», sin imputar el más cercano. (CONTRATO-API, «GET /api/eventos»; ENUNCIADO, §§4.3, 8 y 11.)

### Alternativa tabular, símbolos y mapa base

- La tabla consultable es requisito y alternativa al mapa y al color, no una ampliación opcional. (ENUNCIADO, §8.)
- El tamaño del círculo representa solo una escala visual de magnitud, no energía ni área afectada; se conserva una leyenda que permita interpretar las variables. (FAQ, pregunta 27; ENUNCIADO, §8.)
- **Si se usan teselas estándar de OpenStreetMap**, mostrar la atribución visible obligatoria y respetar la política del proveedor; el mapa base es externo y depende de Internet. (FUENTES-Y-LICENCIAS, «Visualización»; README del material base, §1.)
- **Recomendación de diseño, no requisito docente identificado:** revisar contraste, legibilidad, navegación por teclado y que la información no dependa únicamente del color o de la forma. La selección de una técnica o biblioteca concreta queda abierta.

No se prescribe paleta, disposición, tipo de gráfica ni biblioteca de visualización.

## 7. Comparación histórica y conclusiones

La comparación debe ser interpretable y trazable:

- Identificar periodo, filtros efectivos, unidad y denominador de cada resultado. (CONTRATO-API, «Otros endpoints a implementar» y 05 §4.)
- Aplicar filtros equivalentes a periodo reciente e histórico; el histórico de trabajo acordado es 2025, mundial, magnitud ≥4, `earthquake`, y no modifica los filtros de ingesta live. (Aclaración del estudiante; ENUNCIADO, §§4.2 y 8.)
- Incluir los días de cero eventos en la media diaria histórica. Para el año 2025 completo, se divide entre los 365 días del periodo, según la aclaración orientativa de FAQ, pregunta 23.
- Comunicar que el periodo reciente puede estar incompleto por retrasos. (ENUNCIADO, §8.)
- Si la media histórica es cero, la variación porcentual no está definida: mostrar ND y explicarlo, según FAQ, pregunta 23. La forma exacta de visualización no está prescrita.
- No inferir predicción ni riesgo a partir de recuentos observados. (ENUNCIADO, §8.)

El proyecto debe entregar al menos tres conclusiones fundamentadas con cifras, filtros y periodo. Los resultados y cifras concretos solo se incorporarán después de analizar datos reales y verificar sus denominadores; esta spec no inventa resultados. (ENUNCIADO, §§8 y 12.)

## 8. Dependencias del panel

La columna de campos se limita a las superficies descritas en 05, cuyo fichero fue aprobado por el estudiante. No se proponen rutas, parámetros o payloads alternativos.

| Capacidad del panel | Endpoint/campos de [05-api-contratos.md](05-api-contratos.md) | Requisitos existentes | Verificación |
|---|---|---|---|
| Mapa y tabla de eventos | `GET /api/eventos`: GeoJSON, `features`, metadata, `id`, `geometry`, propiedades de evento, país, tiempos y campos nulos; filtros y estado `truncated` según 05 §3. | REQ-F-008, REQ-F-009, REQ-API-001, REQ-DATA-002 | Contrastar puntos/fichas/filas con la respuesta API; verificar únicos actuales, coordenadas/unidades, nulos y truncamiento. |
| Filtros del mapa/tabla | `GET /api/eventos` y semántica de filtros de 05 §3. | REQ-F-010, REQ-API-001 | Ejecutar iguales filtros en vistas tabular y cartográfica; comprobar pertenencia de resultados y extremos de fecha. |
| Recuentos y distribuciones | `GET /api/estadisticas`: periodo, filtros, unidades, recuentos horarios/país, distribuciones y faltantes según 05 §4. | REQ-F-009, REQ-API-002, REQ-DATA-004 | Comparar agregados con una consulta independiente sobre el mismo conjunto actual y filtros. |
| Comparación histórica | `GET /api/estadisticas`, bloque de comparación y filtros efectivos según 05 §4.3. | REQ-F-005, REQ-F-010, REQ-F-011, REQ-API-002, REQ-DATA-004 | Verificar periodos, filtros equivalentes y denominador de todos los días; usar evidencia real antes de redactar conclusiones. |
| Salud/actualidad | `GET /api/salud` según 05 §5; `last_ingestion` en metadata de eventos según 05 §3; hora de último acceso mantenida por el cliente. | REQ-F-012, REQ-API-003, REQ-TEST-005 | Comprobar distinción de tiempos, fallo y recuperación; aplicar solo el umbral que se acuerde en 05. |
| Errores de consulta | Esquema y estados de error de 05 §6; la API estadística no debe presentar cero como éxito si falla la base. | REQ-NF-003, REQ-API-002, REQ-API-003 | Simular error y comprobar mensaje de fallo, conservación/identificación de datos previos y recuperación posterior. |
| Respuesta parcial | `truncated`, total y continuación según 05 §3.2 y decisiones de paginación allí registradas. | REQ-API-001, REQ-F-008 | Revisar que el panel comunica parcialidad, navega según el contrato y no obtiene estadísticas globales de una página. |
| Modo replay | `mode`/etiqueta de reproducción conforme a 05 §3 y al flujo descrito en 04 §8. | REQ-F-013, REQ-DATA-005, REQ-TEST-006 | Verificar que la vista identifica «REPRODUCCIÓN» y no confunde el conjunto con live. |
| Atribución del mapa base | No procede de un endpoint; depende de si se usan teselas estándar OpenStreetMap. | REQ-F-008; ENUNCIADO, §8; FUENTES-Y-LICENCIAS, «Visualización» | Inspección visual de atribución cuando se utilicen esas teselas. |

La página de eventos y la respuesta estadística son superficies independientes: aunque el mapa esté paginado, las estadísticas deben representar el conjunto correspondiente a sus filtros y periodo, no la suma de lo que casualmente se cargó en la interfaz. Si una capacidad no está cubierta o los datos necesarios no están disponibles en los contratos aprobados, se registra como dependencia a revisar; no se crea un contrato nuevo desde el panel.

## 9. Criterios de aceptación

Los siguientes escenarios son verificaciones futuras; **no declaran pruebas ejecutadas**. Los identificadores son los existentes en [01-requisitos-aceptacion.md](01-requisitos-aceptacion.md).

| Escenario | Resultado esperado | Requisitos existentes | Fuente | Evidencia verificable |
|---|---|---|---|---|
| Datos provienen de la API | El panel muestra datos procesados desde la API y no consulta USGS directamente. | REQ-F-008, REQ-API-001 | ENUNCIADO, §§2 y 8; README del material base, §§3 y 5 | Inspeccionar solicitudes de red y comprobar que el panel usa el endpoint contratado. |
| Filtros coherentes | Mismos filtros efectivos se reflejan en mapa, tabla y gráficos; comparación declara los filtros que utiliza. | REQ-F-010, REQ-API-001, REQ-API-002, REQ-DATA-004 | ENUNCIADO, §8; CONTRATO-API, «GET /api/eventos» y «Otros endpoints a implementar» | Aplicar un conjunto de filtros y comparar IDs/conteos con respuestas y consulta de referencia. |
| Nulos y país no asignado | Campos desconocidos se distinguen de cero; magnitud nula se visualiza como ND; «Sin país asignado» se mantiene en análisis. | REQ-F-008, REQ-F-009, REQ-DATA-002, REQ-DATA-003, REQ-TEST-003 | ENUNCIADO, §§4.3, 8 y 11; CONTRATO-API, «GET /api/eventos» | Fixture con nulos y evento sin polígono; revisar mapa, ficha, tabla y agregados. |
| Comparación equivalente | Periodos y filtros son visibles/equivalentes; denominador histórico incluye días sin eventos; retraso del periodo reciente se comunica. | REQ-F-005, REQ-F-010, REQ-F-011, REQ-API-002, REQ-DATA-004 | ENUNCIADO, §§4.2 y 8; FAQ, pregunta 23 | Verificar filtros efectivos, periodo y denominador contra cálculo independiente; no usar cifras ficticias como resultados. |
| Lectura antigua tras fallo | Un error no hace pasar una lectura anterior por una nueva; se identifica el error y la marca temporal previa. | REQ-F-012, REQ-TEST-005 | ENUNCIADO, §§8 y 11; README del material base, §3; FAQ, pregunta 31 | Interrumpir API/túnel tras una respuesta válida y comprobar estado/tiempo presentado. |
| Recuperación de API/túnel | El panel vuelve a consultar y recupera datos al restaurarse el acceso. | REQ-F-012, REQ-TEST-005 | ENUNCIADO, §11; README del material base, §3 | Cerrar y reabrir el túnel y comprobar una consulta posterior satisfactoria. |
| Resultado truncado | Se muestra parcialidad y navegación según 05; ninguna estadística global se deriva solo de la página de eventos. | REQ-F-008, REQ-F-010, REQ-API-001 | CONTRATO-API, «GET /api/eventos»; 05 §3.2 | Probar respuesta que indique truncamiento y verificar etiqueta, continuación y fuente separada de estadísticas. |
| Replay | La vista etiqueta los datos como «REPRODUCCIÓN» y mantiene su distinción de live. | REQ-F-013, REQ-DATA-005, REQ-TEST-006 | ENUNCIADO, §11; 04 §8 | Consultar replay controlado y verificar etiqueta/modo y conjunto seleccionado. |
| Alternativa tabular | Se pueden consultar los eventos en tabla sin depender del mapa ni de su codificación por color. | REQ-F-009 | ENUNCIADO, §8 | Revisar que filas y campos relevantes se consultan y que la tabla permite identificar nulos. |
| Atribución OpenStreetMap | Si se usan teselas estándar de OpenStreetMap, su atribución es visible. | REQ-F-008 | FUENTES-Y-LICENCIAS, «Visualización»; README del material base, §1 | Inspección visual en la vista con dicho mapa base. |
| Aviso de datos desactualizados | El panel comunica desactualización usando el estado/umbral acordado, sin inventar un umbral local contradictorio. | REQ-F-012, REQ-API-003 | ENUNCIADO, §8; CONTRATO-API, «Otros endpoints a implementar»; 05 §5 | Probar estados frescos/antiguos solo después de disponer de umbral acordado; verificar que no hay un valor codificado distinto. |

## 10. Decisiones pendientes

Las decisiones de esta tabla son de interfaz o dependencias concretas; no alteran los requisitos del profesor ni reabren payloads aprobados. Se puede planificar el trabajo general con ellas abiertas, resolviéndolas antes de la parte de interfaz afectada.

| Decisión pendiente | Impacto | Cuándo resolver |
|---|---|---|
| Biblioteca concreta para gráficas | No bloquea planificar ni implementar el consumo de API; afecta la realización de distribuciones y series. | Al planificar la vista de análisis, respetando Angular existente y sin añadir dependencias antes de decidirlas. |
| Disposición visual, navegación y relación entre mapa, tabla y gráficos | No bloquea requisitos funcionales ni la integración API; sí la implementación visual y su validación de usabilidad. | Durante el diseño de interfaz antes de cerrar componentes/pantallas. |
| Lenguaje visual para carga, vacío, error, lectura anterior y recuperación; textos, iconos y avisos | El comportamiento de informar/conservar está requerido; los detalles visuales no están prescritos. | Al diseñar estados y antes de pruebas manuales de aceptación. |
| Umbral numérico/representación de obsolescencia | El aviso de datos desactualizados sí se exige; no hay umbral docente. Depende de la decisión de salud de 05 §5, no debe sustituirse con una regla independiente del cliente. | Acordar primero el contrato/umbral de salud y después concretar su representación; no bloquea la planificación del panel. |
| Tamaño numérico de página y controles visuales de continuación | La comunicación de truncamiento y la navegación siguen 05 §3.2. El límite/paginación exactos no se fijan aquí. | Resolver el límite en el contrato/API si aún no quedó acordado y definir después los controles de UI; bloquea la navegación paginada, no las vistas completas. |
| Presentación cuando los filtros generales son más amplios que la cobertura histórica | Puede afectar a la comparabilidad y a las conclusiones; la interfaz debe seguir la decisión de 05 §4.3 y no ocultar filtros distintos. | Antes de implementar la comparación histórica; verificar que el usuario ve los filtros efectivos. |
| Accesibilidad concreta y pruebas de usabilidad | Recomendación de diseño; las fuentes no establecen un estándar, nivel de conformidad ni técnica concreta. | Al diseñar y revisar el panel; no introducir una biblioteca/estándar como requisito sin aprobación. |
| Uso de teselas OpenStreetMap | Si se eligen, la atribución visible y la política del proveedor aplican; las teselas requieren Internet. No se exige usar ese proveedor como única opción. | Al cerrar el fondo cartográfico y verificar atribución y dependencia de red. |


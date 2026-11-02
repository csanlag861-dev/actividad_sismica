# Principios y alcance

## Objetivo del proyecto

**Requisito del profesor.** Construir un observatorio que recoja datos sísmicos, los procese y permita explorar dónde se registra actividad y cómo se compara con un periodo histórico equivalente. Debe evidenciar extracción automática, calidad, integración, modelado, análisis y reflexión sobre costes y complejidad. El mapa por sí solo no satisface el proyecto. Fuente: [ENUNCIADO.md](../source-of-truth/ENUNCIADO.md), §§1, 3 y 8.

El proyecto no es un sistema de predicción de terremotos ni una evaluación de riesgo: las conclusiones deben referirse a recuentos observados y sus filtros, periodo y cobertura. Fuente: ENUNCIADO, §8; FAQ, preguntas 1 y 24.

## Alcance funcional de alto nivel

El trabajo consiste en:

1. Consultar periódicamente el feed live de USGS y cargar una vez un conjunto histórico desde el catálogo.
2. Publicar eventos y revisiones en Kafka; validar, transformar, enriquecer y persistir resultados en PostgreSQL, preservando originales, versiones y rechazos.
3. Enriquecer eventos mediante asignación espacial al catálogo Natural Earth, sin inferir país a partir del texto de `place`.
4. Exponer por FastAPI datos procesados para el panel Angular/Leaflet; ofrecer mapa, filtros, tabla, gráficos, comparación histórica y estado del sistema.
5. Publicar el panel como SPA en Netlify y habilitar acceso a la API mediante ngrok durante la demostración.
6. Documentar la calidad, el modelo, las pruebas, el análisis, las mediciones, costes, límites y procedimiento de ejecución/defensa.

Fuente: ENUNCIADO, §§2–12; [CONTRATO-API.md](../source-of-truth/CONTRATO-API.md).

### Exclusiones explícitas

- No se exige Spark, Flink, Kubernetes, aprendizaje automático ni un servidor cloud permanente; las ventanas pueden calcularse con Python y SQL. (ENUNCIADO, §2.)
- No se requiere disponibilidad 24/7: el equipo, contenedores y túnel pueden apagarse fuera de la demostración. (ENUNCIADO, §9.)
- No se requiere una transacción distribuida Kafka–PostgreSQL: el enunciado fija procesamiento al menos una vez con escritura idempotente. (ENUNCIADO, §7.)
- No es obligatorio generar automáticamente informes de calidad ni añadir un panel de calidad. La cuarentena duradera de rechazos sí forma parte del alcance. (ENUNCIADO, §6; FAQ, pregunta 17.)
- No se exige resolver exhaustivamente eliminaciones, fusiones o revisiones de eventos que ya quedaron fuera del feed. Debe documentarse la limitación. (ENUNCIADO, §5.)
- No se exige desarrollar desde cero un motor cartográfico ni sustituir el mapa de apoyo. (ENUNCIADO, §§3, 13.)

## Actores y componentes principales

| Actor o componente | Responsabilidad según el material |
|---|---|
| Estudiante | Desarrollar y explicar individualmente el flujo completo y una decisión técnica propia. La modalidad individual es además una aclaración confirmada por el estudiante. |
| USGS | Proporcionar el feed live y el catálogo histórico. |
| Productor Python | Consultar el feed, detectar versiones y publicar novedades en Kafka. |
| Kafka | Transportar y retener mensajes. |
| Consumidor Python | Validar, limpiar, enriquecer, registrar rechazos y escribir de forma idempotente. |
| PostgreSQL | Persistir datos preparados, originales, versiones y estado necesario. |
| API FastAPI | Consultar los datos procesados y servirlos al frontend. |
| ngrok | Proporcionar acceso HTTPS a la API local; no autentica al usuario. |
| Angular + Leaflet | Mostrar mapa, filtros, tabla, gráficos y estado. |
| Netlify | Servir el frontend SPA. |
| Natural Earth | Aportar la copia geográfica incluida para fronteras y asignación espacial, con sus límites cartográficos. |

Fuente: ENUNCIADO, §§2–4 y 9; FAQ, preguntas 2 y 3; FUENTES-Y-LICENCIAS, «Catálogo incluido».

## Arquitectura prescrita, sin diseño adicional

**Requisito del profesor.** Productor Python, Kafka, consumidor Python, PostgreSQL y API FastAPI se ejecutan como cinco servicios separados de Docker Compose en el equipo del alumno. El frontend Angular es SPA publicada en Netlify. ngrok puede ejecutarse en el equipo o como agente/contenedor. Kafka y PostgreSQL no se exponen por el túnel. La entrega debe permitir arrancar los cinco servicios conjuntamente con `docker compose up --build -d`. Fuente: ENUNCIADO, §2.

No se prescribe aquí el modelo físico, el esquema de tablas, la distribución de tópicos, el número de imágenes Python ni la solución exacta para cargas iniciales o aislamiento de replay: se implementarán según las alternativas admitidas por el enunciado y se documentarán. El detalle mínimo de transporte, CORS y endpoints indicado por el contrato se recoge en [01-requisitos-aceptacion.md](01-requisitos-aceptacion.md).

## Fuentes de datos

### Live

USGS `all_day.geojson`, consultado por HTTP GET aproximadamente cada 60 segundos. Es una ventana móvil de las últimas 24 horas, no el día natural anterior ni una conexión push. Se consideran campos de identificación, tiempo, actualización, magnitud, tipo de magnitud, tipo de evento, lugar y coordenadas; el tipo live se filtra a `earthquake` y se conserva el motivo de exclusión. Fuente: ENUNCIADO, §4.1.

**Aclaración confirmada por el estudiante:** el filtro histórico propuesto de 2025, global, magnitud ≥4 y tipo `earthquake` se usará como subconjunto de trabajo/pruebas para filtros y comparación histórica. No se aplicará automáticamente a la ingesta live ni cambiará el alcance geográfico del feed live. La regla del feed live sobre el tipo `earthquake` sí procede del enunciado.

### Histórico

El enunciado propone un año completo con alcance manejable y ofrece como propuesta de partida 2025, mundial, magnitud ≥4 y tipo `earthquake`; permite acotar una zona si los filtros se documentan y se mantienen iguales en la comparación. Fuente: ENUNCIADO, §4.2.

**Aclaración confirmada por el estudiante:** se utilizará exactamente el subconjunto propuesto (2025, mundial, magnitud ≥4, `earthquake`) para trabajo, pruebas de filtros y comparación histórica. No es filtro de la ingesta live. El histórico es una instantánea del catálogo en la fecha de extracción, no un registro completo de revisiones pasadas.

### Geográfica

Se usará la copia entregada de Natural Earth, con entidades `Polygon`/`MultiPolygon` y atributos como `ADM0_A3`, `NAME_ES`, `ADMIN`, `CONTINENT` y `SUBREGION`. La asignación es espacial con coordenadas compatibles WGS84/EPSG:4326; los eventos sin polígono coincidente permanecen como «Sin país asignado». `ADM0_A3` no debe presentarse como código ISO oficial universal. Fuente: ENUNCIADO, §4.3; FUENTES-Y-LICENCIAS, «Catálogo incluido».

## Aclaraciones confirmadas por el estudiante

Estas aclaraciones no son requisitos del profesor:

- **Modalidad y plazo:** el proyecto es individual y se comunicó una entrega dentro de 20 días. No se ha confirmado aquí una fecha calendario exacta; queda pendiente confirmar esa fecha, sin inferirla. Fuente de estas aclaraciones: confirmación del estudiante; el enunciado original indica que modalidad y fecha se comunicarían en clase (ENUNCIADO, §1).
- **Versiones:** donde los materiales no indiquen versión, se prefieren versiones LTS. Es una preferencia del estudiante, no un requisito docente. No desplaza las versiones fijadas para el frontend de apoyo: Angular 20.3, ngx-leaflet 20.0 y Leaflet 1.9.4 (README del material base, §1).
- **Histórico:** se confirma el subconjunto de 2025 mundial, magnitud ≥4, tipo `earthquake`, con el propósito y límite descritos arriba.
- **API:** los contratos detallados de `/api/estadisticas` y `/api/salud` no están definidos en los materiales revisados; el equipo los propondrá posteriormente para revisión.
- **Herramientas de trabajo:** Copilot en VS Code coordina; Codex y Antigravity reciben tareas acotadas que usan los artefactos canónicos del proyecto. Se prefiere una fuente portable compartida de personalizaciones cuando sea posible; no se presupone que las tres herramientas interpreten igual agentes, skills o hooks.
- **Permisos y seguridad operativa:** agentes de análisis/revisión read-only por defecto. Implementar requiere tarea aprobada; despliegues y acciones destructivas requieren confirmación.
- **Tutoría:** el tutor tipo profesor acompaña el trabajo, formula preguntas al cerrar tareas y al revisar documentación; al final se realizará un simulacro de defensa. Sus aclaraciones solo se incorporarán a `docs/learning/` con respuestas verificadas y aprobación del estudiante.
- **Documentación didáctica:** la documentación futura irá en `docs/learning/`, organizada por componente y concepto y enlazada al código; se profundizará especialmente en archivos o funciones difíciles.
- **Hooks:** no se añadirán al principio; se revisarán más adelante, incrementalmente y de forma específica por herramienta.

Estas preferencias de coordinación y enseñanza no amplían los requisitos funcionales del observatorio ni crean entregables para esta especificación.

## Conceptos y términos consistentes

- **Evento**: identidad sísmica `id` de USGS; no equivale a cada mensaje recibido.
- **Versión**: estado observado de un evento, identificado por `id` y `updated`; una versión posterior puede sustituir el estado actual, no borrar el historial recibido.
- **Live**: ingesta del feed de ventana móvil; conserva su alcance geográfico del proveedor.
- **Histórico**: conjunto descargado una vez desde el catálogo, con filtros registrados.
- **Replay / reproducción**: tránsito acelerado de una selección histórica por Kafka, aislado de live y etiquetado «REPRODUCCIÓN».
- **Tiempo de evento**, **actualización USGS**, **ingesta** y **generación de respuesta** son tiempos distintos; los timestamps de USGS se expresan en milisegundos Unix UTC.
- **Sin país asignado**: resultado válido de falta de coincidencia espacial; no es automáticamente un rechazo.
- **Cuarentena**: almacenamiento duradero del original rechazado y su motivo.
- **Actual**: versión más reciente conocida que se utiliza para consulta y agregación.

Fuentes: ENUNCIADO, §§4–8 y 11; CONTRATO-API, `GET /api/eventos`.

## Límites del proyecto y del material base

- El paquete aporta un frontend/cartografía de apoyo y datos de demostración ficticios, no backend ni pipeline implementados. (ENUNCIADO, §3; README del material base, §§1–5.)
- `VALIDACION.md` declara compilación de frontend y comprobaciones del catálogo y los recursos incluidos. También declara que no hubo prueba visual completa, despliegue real en Netlify ni túnel del estudiante. No es evidencia de implementación o validación del backend.
- Natural Earth tiene fronteras simplificadas, no aptas para jurisdicción legal ni asignación precisa en costas; algunos territorios pequeños no están incluidos. El fondo de teselas OpenStreetMap depende de Internet, su política y disponibilidad. (FUENTES-Y-LICENCIAS, «Catálogo incluido» y «Visualización».)
- El frontend publicado puede seguir cargando cuando el equipo local o túnel están apagados, pero no habrá nuevos datos del backend. La disponibilidad continua no forma parte del alcance mínimo. (ENUNCIADO, §9; CONTRATO-API, «Acceso desde Netlify».)
- La carga histórica representa una instantánea; el feed puede retrasarse y revisar eventos. El alcance mínimo no reconcilia exhaustivamente eliminaciones/fusiones fuera de la ventana.

## Supuestos documentales y preguntas abiertas

En la columna «Impacto», **no bloquea planificación** significa que el trabajo de alto nivel puede continuar con los límites ya conocidos; **bloquea implementación** significa que no se debe codificar/configurar el aspecto concreto hasta cerrar y revisar la decisión. Estos estados no convierten una propuesta en decisión aprobada.

| ID | Naturaleza | Enunciado | Estado actual | Spec donde resolver | Impacto |
|---|---|---|---|---|---|
| DOC-P-01 | Supuesto documental | Los documentos listados en `docs/source-of-truth/` son el material base disponible en este workspace; no se infiere que estén implementadas las funcionalidades descritas. | Aplicado al redactar | No aplica | No bloquea planificación ni implementación |
| DEC-P-01 | Requisito docente + aclaración del estudiante | Las fuentes no contienen el contrato detallado de `/api/estadisticas`. El equipo elaborará una propuesta para revisión. | Aclaración confirmada; propuesta y payload pendientes de revisión/aprobación | Futura `docs/spec/04-contratos-api.md` (archivo aún no creado); después, actualizar trazabilidad en `01-requisitos-aceptacion.md` | No bloquea la planificación global; bloquea la implementación de este endpoint hasta revisar el contrato |
| DEC-P-02 | Requisito docente + aclaración del estudiante | Las fuentes solo indican el propósito mínimo de `/api/salud`. El equipo propondrá el contrato detallado para revisión. | Aclaración confirmada; propuesta y payload pendientes de revisión/aprobación | Futura `docs/spec/04-contratos-api.md` (archivo aún no creado); después, actualizar trazabilidad en `01-requisitos-aceptacion.md` | No bloquea la planificación global; bloquea la implementación de este endpoint hasta revisar el contrato |
| DEC-P-03a | Decisión de diseño pendiente | Modelo físico y estructura de tablas. Las fuentes orientan sobre grano, claves y relaciones, pero no prescriben un diseño físico único. | Pendiente; no hay propuesta aprobada | Futura `docs/spec/05-modelo-logico-fisico.md` (archivo aún no creado), partiendo de conceptos de `03-modelo-datos-calidad.md` | No bloquea la planificación conceptual; bloquea implementación de persistencia/esquema |
| DEC-P-03b | Decisión de diseño pendiente | Índices y justificación según consultas. No hay un conjunto de índices impuesto por las fuentes. | Pendiente; requiere consultas previstas y justificación/medición | Futura `docs/spec/05-modelo-logico-fisico.md` (archivo aún no creado) | No bloquea planificación general; bloquea decidir/implementar índices físicos |
| DEC-P-03c | Decisión de diseño pendiente | Nombres y organización de tópicos para live/replay. | Pendiente; no hay propuesta aprobada | Revisión futura de [02-arquitectura-flujos.md](02-arquitectura-flujos.md) | No bloquea planificación de componentes; bloquea configuración final e integración de productores/consumidores |
| DEC-P-03d | Decisión de diseño pendiente | Mecanismo de aislamiento de replay (tópicos/tablas/esquema u opción equivalente). El enunciado exige separación, no una técnica concreta. | Pendiente; alternativas no aprobadas | Revisión futura de [02-arquitectura-flujos.md](02-arquitectura-flujos.md), coordinada con la futura `05-modelo-logico-fisico.md` | No bloquea planificación del flujo; bloquea implementación integrada del replay |
| DEC-P-04 | Decisión de diseño pendiente | Regla para puntos en frontera o con varias coincidencias espaciales. El enunciado exige documentar una regla, pero no prescribe cuál; no se ha aprobado una propuesta. | Pregunta abierta; no es un requisito de solución concreta | Revisión futura de [03-modelo-datos-calidad.md](03-modelo-datos-calidad.md) | No bloquea planificación del modelo; bloquea implementación definitiva de asignación espacial para esos casos |
| DEC-P-05 | Propuesta del profesor + aclaración confirmada del estudiante | Subconjunto histórico de trabajo y comparación: año 2025, ámbito mundial, magnitud ≥4, tipo `earthquake`. Se usa para trabajo, filtros y comparación; no se aplica como filtro a la ingesta live. Para comparación reciente se conservan filtros equivalentes y denominadores de las fuentes, incluida la media diaria sobre todos los días de 2025. | Aclaración confirmada; alcance establecido, no pendiente | Registrado en [01-requisitos-aceptacion.md](01-requisitos-aceptacion.md) y [03-modelo-datos-calidad.md](03-modelo-datos-calidad.md) | No bloquea planificación; los resultados concretos requieren datos y pruebas |
| DEC-P-06 | Aclaración del estudiante + dato pendiente | El proyecto es individual y se comunicó entrega dentro de 20 días; no se confirmó aquí fecha calendario exacta. | Modalidad/plazo comunicado aclarados; fecha exacta pendiente, no inferida | Especificación futura de calendario/entrega, si llega a ser necesaria; actualmente no hay una spec creada para ello | No bloquea planificación técnica ni implementación; bloquea fijar calendario de entrega/defensa |
| DEC-P-RET | Decisión de diseño pendiente | Duración y estrategia concreta de retención de originales, histórico, versiones, cuarentena y mensajes Kafka. | Pendiente; no se han propuesto duraciones | Revisión futura de [03-modelo-datos-calidad.md](03-modelo-datos-calidad.md); la retención del broker se coordina con [02-arquitectura-flujos.md](02-arquitectura-flujos.md) | No bloquea planificación conceptual; bloquea configuración de retención y políticas de purga |

Las decisiones de coordinación y didáctica indicadas en «Aclaraciones confirmadas por el estudiante» son preferencias/procedimientos del estudiante, no requisitos del profesor. En particular, Copilot en VS Code coordina y las otras herramientas reciben tareas acotadas apoyadas en artefactos canónicos; los permisos de agentes, tutoría y revisión futura de hooks se mantienen como aclaraciones, no como requisitos de producto.

## Decisiones que todavía no deben tomarse

Hasta revisar las fuentes y las decisiones pendientes, no se debe:

- tratar como aprobados los contratos detallados de estadísticas o salud: el equipo preparará propuestas y el payload seguirá pendiente hasta su revisión;
- interpretar el subconjunto histórico 2025 como filtro live o como mandato de limitar la cobertura geográfica live;
- fijar versiones LTS donde las fuentes no las fijan, ni sustituir las versiones concretas del frontend de apoyo;
- elegir un esquema físico, particionado, librería geoespacial, estrategia concreta de tópicos o patrón de replay como si lo exigiera el profesor;
- tratar como requisito las recomendaciones del FAQ/guía, los ejemplos ilustrativos o mejoras explícitamente opcionales;
- convertir validaciones declaradas del frontend en evidencia de backend implementado;
- fijar fecha calendario de entrega a partir de la sola indicación «dentro de 20 días»;
- convertir una regla geoespacial propuesta para fronteras o coincidencias en decisión aprobada antes de revisarla.

La siguiente revisión corresponde al estudiante. No se amplía esta especificación ni se implementan componentes hasta recibirla.
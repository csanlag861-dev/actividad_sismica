# Plan técnico: Modelo de persistencia sísmica

**Rama/feature**: `002-modelo-persistencia`  
**Estado**: Propuesta para revisión y aprobación; no describe una implementación realizada.  
**Especificación**: [spec.md](./spec.md)

## Objetivo y fuentes

Definir una persistencia PostgreSQL versionada que conserve el estado actual de cada evento, sus versiones y originales, los rechazos con su motivo y las respuestas GeoJSON completas durante 90 días. El esquema debe poder prepararse desde una base vacía antes de que FastAPI atienda, sin `create_all`, servicios Compose adicionales ni cambios en rutas o respuestas de la API. Live y replay compartirán tablas, pero su modo participará en la identidad y podrá filtrarse en las consultas internas.

Fuentes consultadas:

- [Especificación y criterios aprobados para aclaración](./spec.md), incluidos FR-001–FR-008 y SC-001–SC-009.
- [Constitución](../../.specify/memory/constitution.md), principios I–V.
- [Enunciado](../../docs/source-of-truth/ENUNCIADO.md), §§ 2, 5–7, 9, 11–13.
- [Contrato mínimo de la API](../../docs/source-of-truth/CONTRATO-API.md): referencia de compatibilidad; este plan no propone cambiar su interfaz.
- [Guía de entregables](../../docs/source-of-truth/Guia-entregables-observatorio-sismico.md), §§ 1.2–1.4 y 2.2–2.5.
- Skills e instrucciones aplicables: `sdd-observatorio`, `consumidor-calidad-postgresql`, `api-fastapi-observatorio`, `docker-compose-demostracion`, `ingesta-usgs-aiokafka`, `pruebas-aceptacion-replay` e instrucciones de backend, PostgreSQL, Kafka y Docker.

## Alcance

### Incluido

- Diseño y persistencia de evento actual, versión recibida, procedencia/respuesta de fuente y rechazo.
- Identidad e idempotencia por modo, evento y versión; preservación de nulos, unidades, timestamps y originales.
- Retención de las respuestas completas por 90 días, independiente de la conservación sin caducidad automática de los originales de versiones y rechazos.
- SQLAlchemy para el mapeo y Alembic para migraciones versionadas. Una migración inicial se revisa y aplica desde Alembic; no se crea el esquema con `Base.metadata.create_all()`.
- Arranque de la API tras PostgreSQL healthy y tras completar correctamente las migraciones, dentro del servicio Compose `api` ya existente.
- Exportación del SQL derivado de las migraciones y enlace desde la memoria o notebook del proyecto.
- Verificaciones de base vacía, migraciones pendientes, fallo de inicio y persistencia tras reinicio.

### Fuera de alcance

- Crear ahora cualquier archivo salvo este `plan.md`; en particular, no se generan `tasks.md`, `research.md`, `data-model.md`, `quickstart.md`, contratos, código ni configuración.
- Implementar rutas, parámetros o cambios de respuestas de API; el contrato existente permanece intacto.
- Añadir servicios Compose, automatización de informes de calidad, herramientas de infraestructura o almacenamiento externo.
- Cambiar reglas de ingesta o el contrato de mensajes Kafka que no sean necesarias para persistir la procedencia acordada.
- Definir como hechos resultados de pruebas todavía no ejecutadas.

## Arquitectura y estado actual pertinente

**Hechos observados**:

- `compose.yaml` declara los cinco servicios previstos. PostgreSQL usa volumen `postgres_data` y healthcheck con `pg_isready`; el servicio `api` ya depende de que PostgreSQL esté healthy.
- Productor, consumidor y API aparecen como placeholders Python; no se observa aún código de aplicación, modelos SQLAlchemy, migraciones Alembic ni pruebas de backend en el repositorio.
- El consumidor está configurado para conectarse a PostgreSQL. El productor solo declara actualmente su dependencia de Kafka, por lo que guardar la respuesta completa desde el productor requeriría añadirle configuración/dependencia de PostgreSQL, sin añadir un servicio.
- El contrato de consulta ya define la interfaz mínima. No se modificarán rutas, filtros ni la forma de respuesta para atender este cambio.
- La especificación ya resolvió el plazo de 90 días, la conservación duradera de originales y el uso de tablas compartidas con `mode` en la identidad.

**Diseño propuesto para aprobación**:

1. Mantener el servicio `api` existente como responsable de aplicar `alembic upgrade head` durante el arranque, y arrancar Uvicorn solo si la migración termina correctamente.
2. Guardar la respuesta completa no transformada y sus metadatos de extracción en PostgreSQL desde el productor, que es quien recibe el cuerpo íntegro de USGS. El consumidor mantiene la responsabilidad de guardar eventos, versiones y cuarentena. Esta separación evita simular que los mensajes individuales del feed equivalen a la respuesta completa.
3. Usar entidades persistentes con estos granos:
   - **Respuesta de fuente**: una extracción GeoJSON completa, con origen, fecha de consulta, modo y vencimiento.
   - **Evento actual**: una fila por `(mode, event_id)`.
   - **Versión recibida**: una fila por `(mode, event_id, updated)`, con su original conservado sin la caducidad de la respuesta completa.
   - **Rechazo**: un registro por mensaje rechazado, con motivo, modo, fecha y original localizable y duradero.
   - **Versión de esquema**: gestionada por la tabla de control de Alembic, no por una tabla de dominio adicional.
4. Mantener en las versiones y los rechazos una copia del original necesaria para su conservación autónoma. La referencia a una respuesta de fuente facilita trazabilidad, pero la caducidad de esta no debe borrar el original asociado a una versión o rechazo.
5. Incluir `mode` en las claves únicas y en las operaciones de consulta del repositorio. La capa que atiende la API seguirá invocando las consultas con el modo que corresponda a su comportamiento actual, sin exponer un parámetro o ruta nueva.

La estructura física concreta, tipos, restricciones, índices y ubicación exacta de módulos se detallarán al descomponer el trabajo, tras inspeccionar el backend cuando exista. No se presupone que los nombres de entidad anteriores sean nombres definitivos de tablas.

## Componentes y archivos previsiblemente afectados

Los siguientes son destinos previsibles de una implementación posterior, no archivos creados o modificados en esta fase:

- `compose.yaml`: conservar los cinco servicios, dependencia de salud de PostgreSQL y volumen; adaptar configuración/comando de `api` para migrar antes de Uvicorn. Si se adopta el diseño de captura en productor, añadirle credenciales/configuración de DB y dependencia `postgres: service_healthy`.
- Backend Python por crear (ubicación final por confirmar): dependencias fijadas de SQLAlchemy/Alembic; configuración de conexión; modelos/repositorios compartidos por productor, consumidor y API.
- Directorio de migraciones Alembic por crear: configuración, entorno de migraciones y revisión inicial legible/revisable.
- Inicio del servicio API por crear: ejecución secuencial de migración y `exec` de Uvicorn; error de migración debe terminar el proceso antes de abrir el puerto.
- Pruebas backend por crear o integrar: modelo/restricciones, retención, secuencia de versiones, arranque y recuperación.
- Artefacto SQL generado desde la cadena de migraciones y memoria/notebook con enlace. La ubicación del documento de memoria/notebook se confirmará al descomponer el trabajo.

No se prevén cambios en `frontend/`, en rutas de API, en servicios de Compose distintos a los ajustes de conexión/dependencias indicados ni en la topología de cinco servicios.

## Dependencias y restricciones

- PostgreSQL 16.6 ya está declarado en Compose con volumen persistente y healthcheck.
- El arranque fiable requiere tanto la condición Compose `service_healthy` como fallo explícito ante errores de conexión o migración. Uvicorn no debe iniciarse en paralelo con la migración.
- SQLAlchemy y Alembic son decisiones expresas de la solicitud; no se usará `create_all` como mecanismo de inicialización.
- La ejecución debe respetar `PGHOST=postgres` en la red Compose y no exponer credenciales en Git ni en recursos públicos.
- Los cambios de datos del consumidor deben confirmarse en PostgreSQL antes de confirmar offsets Kafka. No se requiere transacción distribuida.
- Las restricciones únicas/upserts deben hacer idempotentes los reenvíos; una versión antigua no puede reemplazar el evento actual más reciente.
- Los valores desconocidos permanecen nulos, sin conversión a cero; se conservan UTC, timestamps Unix de origen y coordenadas/unidades documentadas.
- `mode` (`live`/`replay`) debe ser parte de la identidad en las tablas compartidas y estar presente en las consultas internas para impedir mezclas accidentales.
- La retención de respuestas completas no debe eliminar originales de versiones/rechazos. Los originales duraderos no tendrán TTL automático; su persistencia ante borrado manual de volúmenes o pérdida física no se afirma ni se amplía aquí a una política de backup.
- No se exige framework de pruebas nuevo ni servicio de migraciones separado. La estrategia debe encajar en las herramientas que se establezcan para el backend.

## Estrategia de implementación propuesta

1. **Acordar el diseño de almacenamiento**: documentar el grano, las claves naturales/únicas, relaciones, nulabilidad, unidades, procedencia, retenciones e índices imprescindibles. Mantener las decisiones físicas mínimas para cumplir los criterios.
2. **Establecer persistencia común**: incorporar configuración de conexión y modelos SQLAlchemy que puedan usar los componentes que escriben/consultan PostgreSQL; definir el mecanismo de guardar respuesta íntegra desde el productor y registros normalizados/cuarentena desde el consumidor.
3. **Crear la migración inicial**: generar la revisión Alembic desde el modelo, revisar manualmente el SQL y constraints contra el diseño y probarla sobre PostgreSQL vacío. La migración es la fuente de verdad del esquema.
4. **Aplicar migraciones al arranque**: ajustar el comando de inicio del `api` para ejecutar `alembic upgrade head` y, solo tras éxito, Uvicorn. Conservar `depends_on: condition: service_healthy`; hacer visible y no reintentar indefinidamente un error de migración. No crear un contenedor/servicio extra.
5. **Integrar escrituras idempotentes**: persistir versiones/originales y rechazo/motivo en transacciones; actualizar el estado actual solamente si la versión recibida es más reciente; confirmar offset después del commit. Verificar que `mode` se propaga por toda clave y consulta interna.
6. **Aplicar retención de respuestas completas**: asignar vencimiento desde la fecha de extracción y purgar únicamente respuestas cuya edad alcance 90 días. Ejecutar la limpieza en un componente existente —propuesta: ciclo de ingesta del productor— sin eliminar las copias originales de versiones/rechazos. Medir el volumen observado antes de añadir optimizaciones de almacenamiento.
7. **Verificar operación y documentación**: exportar el SQL desde las migraciones, enlazarlo desde la memoria/notebook correspondiente, probar una base vacía y un reinicio con datos, y comprobar que el contrato de API no cambió.

## Decisiones justificadas y clasificación

| Clasificación | Decisión | Justificación |
|---|---|---|
| Requisito obligatorio | Modelo documentado conceptual, lógico y físico, diccionario, procedencia, versiones y rechazos | FR-001–FR-003; Enunciado §§ 5–7 y Guía § 2.5. |
| Requisito obligatorio | Nulos fieles, versión repetida idempotente, versión antigua no reemplaza a la reciente | Constitución II–III; FR-003; Enunciado §§ 6–7. |
| Decisión acordada por el estudiante | Retención de respuesta completa 90 días y originales de versiones/rechazos duraderos | FR-004; aclaración de la especificación. |
| Decisión acordada por el estudiante | SQLAlchemy/Alembic, migración inicial revisada y aplicada antes de Uvicorn; sin `create_all` | FR-005 y solicitud delimitada. |
| Decisión acordada por el estudiante | Tablas compartidas con modo en identidad y consultas capaces de separar `live`/`replay` | FR-008; aclaración de la especificación. |
| Decisión acordada por el estudiante | Sin servicios nuevos, rutas nuevas ni cambios al contrato/respuesta de API | FR-006 y SC-008. |
| Decisión acordada por el estudiante | SQL de esquema generado desde migraciones y enlazado desde memoria/notebook | FR-007; solicitud y Guía § 2.5. |
| Propuesta técnica para aprobación | El productor guarda la respuesta completa en PostgreSQL; el consumidor persiste las versiones normalizadas y rechazos | El productor es el punto que recibe el cuerpo de respuesta íntegro; evita enviar una copia completa por cada evento o inventar un servicio. Añade dependencia del productor a PostgreSQL, que debe validarse frente al código cuando exista. |
| Propuesta técnica para aprobación | Aplicar limpieza de respuestas vencidas desde el ciclo de ingesta ya existente | Cumple retención sin añadir scheduler/servicio; la limpieza nunca debe recorrer ni borrar originales duraderos. |
| Propuesta técnica para aprobación | Claves con modo: `(mode,event_id)` y `(mode,event_id,updated)` | Hace explícita la identidad requerida en las tablas compartidas y permite aislamiento por consultas. |
| Propuesta opcional excluida | Informes automáticos, panel de calidad, particionado o almacenamiento externo | No exigidos y aumentan dependencias/operación sin criterio aprobado que lo requiera. |

## Riesgos y mitigaciones

| Riesgo | Mitigación propuesta |
|---|---|
| El volumen de respuestas completas retenidas 90 días puede crecer rápidamente. | Medir tamaño y frecuencia reales; comprobar espacio del volumen y documentar el resultado. No reducir la retención acordada. Evitar deduplicación, compresión o servicios externos salvo evidencia y nueva aprobación. |
| Guardar en PostgreSQL desde el productor introduce una dependencia adicional y escrituras separadas entre DB y Kafka. | Guardar el original al recibirlo y mantener reintentos/idempotencia; no marcar publicación como completada antes de la confirmación Kafka. Probar caídas entre persistencia y publicación y documentar duplicados tolerados. |
| Una migración fallida o incompatible puede impedir que la API arranque. | Migración inicial revisada, prueba en base vacía y datos existentes de fixture; proceso de inicio secuencial, logs claros y comprobación de código de salida. No continuar a Uvicorn. |
| El uso de `mode` solo en algunas claves/joins podría contaminar consultas live con replay. | Restricciones únicas y filtros explícitos por modo en los repositorios; fixtures con evento/versión idénticos en ambos modos y consultas separadas para cada uno. |
| Purgar una respuesta fuente podría borrar originales requeridos a largo plazo. | Separar retención de la respuesta completa y retención de originales por versión/rechazo; no utilizar borrado en cascada desde la respuesta hacia esos registros. |
| Un cambio incidental de modelo podría alterar el contrato de la API o presentar nulos como cero. | Pruebas de no regresión del contrato actual y casos de persistencia/lectura de nulos, unidades y timestamps; limitar cambios al acceso a datos. |
| El volumen Compose preserva datos ante reinicio normal, no ante su eliminación manual. | Probar reinicio sin `down -v`; comunicar el límite de recuperación y no afirmar backup/archivo externo, que no está en el alcance aprobado. |

## Criterios de aceptación y verificaciones observables

Las referencias identifican requisitos de `spec.md`; las verificaciones son trabajo futuro y no resultados observados.

| Criterio | Relación | Verificación propuesta |
|---|---|---|
| Modelo y diccionario cubren entidades, granos, relaciones, claves, restricciones y todos los campos persistidos del alcance. | FR-001, SC-001 | Revisar modelo conceptual/lógico/físico y diccionario contra los modelos y migraciones; cada campo persistido debe tener significado, origen, unidad/formato y tratamiento de ausencia. |
| Evento actual, versiones, respuesta fuente y rechazo pueden distinguirse; rechazo conserva motivo y original localizable. | FR-002, SC-003 | Pruebas de persistencia con evento válido y rechazado; inspeccionar filas y originales vinculados después de procesarlos. |
| Duplicados y revisiones preservan el estado correcto y los nulos no se convierten en cero. | FR-003, SC-002, SC-003 | Reproducir la secuencia de aceptación de la spec (2 eventos actuales y 3 versiones distintas); reenvío no aumenta recuentos, versión antigua no revierte la reciente; comparar campos nulos/unidades antes y después. |
| Respuesta completa vence al cumplir 90 días; originales de cada versión y rechazo continúan disponibles. | FR-004, SC-004 | Probar limpieza con marcas de tiempo controladas a ambos lados del umbral; verificar la eliminación/expiración de respuestas y la permanencia independiente de originales. |
| PostgreSQL healthy y migración preceden a Uvicorn; un fallo de migración impide aceptar conexiones/API y queda registrado. | FR-005, SC-005 | Arranque Compose aislado con volumen nuevo; revisar orden/logs y sondear disponibilidad solo después de Uvicorn. Probar fallo controlado de conexión/migración y verificar salida no cero y ausencia de API disponible. |
| El esquema queda en la última revisión y sin cambios de modelo pendientes después del arranque. | FR-005, SC-005 | Comparar `alembic current` con `alembic heads` y ejecutar `alembic check` en la base iniciada; esperar revisión vigente y cero operaciones pendientes. |
| Los datos y estado se mantienen tras reinicio normal. | SC-006 | Guardar fixtures, recrear/reiniciar servicios sin borrar el volumen, y comparar registros/recuentos previos y posteriores. |
| SQL se deriva de las migraciones y la memoria/notebook contiene un enlace accesible. | FR-007, SC-007 | Regenerar el archivo SQL desde Alembic, contrastar revisión con el esquema aplicado y comprobar que el enlace del documento abre el artefacto. |
| Se conserva el contrato actual; no se agregan rutas ni servicios Compose. | FR-006, SC-008 | Como la API actual es un placeholder, contrastar los cambios con `CONTRATO-API.md` y verificar que esta implementación de persistencia no define rutas ni modifica respuestas; comparar servicios Compose y confirmar que siguen siendo los cinco obligatorios. |
| Live y replay comparten tablas pero mantienen identidad y consultas separadas. | FR-008, SC-009 | Persistir fixtures con mismos `event_id`/`updated` en ambos modos; confirmar coexistencia y resultados SQL/repositorio filtrados sin contaminación cruzada. |

## Verificaciones técnicas transversales

- Validación de configuración Compose y dependencias de salud; arranque aislado de PostgreSQL/API sin borrar volúmenes existentes.
- Aplicación de todas las migraciones desde una base vacía; comprobación de `current` frente a `heads`, `alembic check` y ausencia de uso de `create_all`.
- Pruebas de constraints e idempotencia para misma versión, revisión posterior, versión antigua, rechazo, nulo y colisión de evento/versión entre modos.
- Prueba de error de migración: log de causa, proceso API detenido y puerto/rutas no disponibles.
- Prueba de persistencia después de reinicio con volumen preservado y de separación de retenciones.
- Exportación reproducible del SQL desde migraciones y prueba del enlace en memoria/notebook.
- Revisión de no regresión del contrato y de que el cambio no añade rutas, servicios, secretos ni dependencias opcionales.
- Registrar comandos, entorno, resultados observados y pruebas no ejecutadas como tales. Esta propuesta no declara pruebas ejecutadas.

## Constitución y alcance proporcionado

| Principio | Evaluación |
|---|---|
| I. Requisitos del enunciado y rúbrica | PASS: se cubren modelado, persistencia, SQL y pruebas de aceptación delimitadas; el contrato de API se conserva. |
| II. Trazabilidad y tratamiento fiel de los datos | PASS: original, fuente, unidades, nulos y motivo/referencia de rechazo forman parte del diseño. |
| III. Ingesta persistente e idempotente | PASS: claves por evento/versión/modo, estado persistido y orden PostgreSQL antes del offset. |
| IV. Análisis comparable y evidencia verificable | PASS: se exigen comprobaciones observables y no se presentan resultados no ejecutados como evidencia. |
| V. Decisiones explícitas y alcance proporcionado | PASS: las elecciones técnicas propuestas se distinguen de obligaciones; no se añaden servicios ni mejoras opcionales. |

## Bloqueos, decisiones y supuestos explícitos

- No se identifica una decisión pendiente sobre comportamiento, retención, identidad live/replay, API o criterio de aceptación: la especificación registra esas aclaraciones.
- Las decisiones técnicas sobre quién persiste la respuesta completa y dónde se ejecuta la limpieza se presentan como propuestas para aprobación, no como requisitos de las fuentes.
- La ubicación exacta del backend Python y de la memoria/notebook que enlazará el SQL no está establecida en el estado observado; debe confirmarse al descomponer los archivos, sin cambiar el alcance.
- No se afirma que exista ya una API implementada ni que las rutas hayan sido probadas: los servicios de backend en `compose.yaml` son placeholders.
- La aprobación de este plan debe preceder a la descomposición. Este documento no genera tareas ni implementa código.

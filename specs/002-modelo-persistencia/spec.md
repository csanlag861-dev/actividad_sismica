# Especificación: Modelo de persistencia sísmica

**Feature Branch**: `specs/002-modelo-persistencia`  
**Creada**: 2026-11-05  
**Estado**: Borrador actualizado tras aclaraciones  
**Entrada**: Definir la persistencia de eventos sísmicos y su documentación, sin alterar la interfaz actual de la API.

## Clarifications

### Session 2026-11-05

- **Pregunta sobre retención**: ¿Qué plazo de conservación aplicar a las respuestas GeoJSON completas de la fuente? **Respuesta**: 90 días; los originales de versiones y rechazos se conservan de forma duradera.
- **Pregunta sobre live/replay**: ¿Se debe separar el almacenamiento de replay y live o compartirlo con una identificación explícita? **Respuesta**: almacenamiento y tablas compartidos; el modo forma parte de la identidad y las consultas deben permitir distinguirlo para evitar mezclar replay con live.

## Escenarios de usuario y pruebas

### Historia 1 — Conservar un registro sísmico fiel y revisable (Prioridad: P1)

Como responsable de analizar la actividad sísmica, necesito distinguir cada terremoto de las versiones recibidas y poder revisar los datos de origen, para que las revisiones, los valores ausentes y los registros rechazados no se conviertan en información engañosa.

**Por qué esta prioridad**: La integridad de los datos determina que el estado consultable, las estadísticas y el análisis sean fiables.

**Prueba independiente**: Procesar una secuencia controlada de mensajes y comparar eventos actuales, versiones conservadas, rechazos y originales con los resultados esperados.

**Escenarios de aceptación**:

1. **Dado** un evento recibido por primera vez, **cuando** se registra, **entonces** queda un estado actual y queda disponible su original.
2. **Dado** el mismo evento y la misma versión recibidos de nuevo, **cuando** se procesa la repetición, **entonces** no aumenta el número de eventos actuales ni de versiones distintas.
3. **Dado** un evento con una versión posterior y después una versión antigua, **cuando** ambas se procesan, **entonces** se conservan las versiones recibidas y el estado actual sigue reflejando la más reciente.
4. **Dado** un campo ausente en el origen, **cuando** se guarda y consulta, **entonces** permanece ausente y no se convierte en cero; las fechas y medidas mantienen sus unidades y significado.
5. **Dado** un registro rechazado, **cuando** se consulta su historial de calidad, **entonces** se conserva el motivo del rechazo y una referencia que permite localizar su original.

### Historia 2 — Poder iniciar y reiniciar el sistema sin perder el estado (Prioridad: P1)

Como persona que ejecuta la demostración, necesito que el almacenamiento esté listo antes de que la API atienda consultas y que sus datos sobrevivan a un reinicio, para no depender de inicializaciones manuales ni perder el historial persistido.

**Por qué esta prioridad**: La demostración y la reanudación del procesamiento dependen de que el esquema y los datos estén disponibles de forma reproducible.

**Prueba independiente**: Iniciar con una base vacía, comprobar que la API solo queda disponible tras preparar el esquema, reiniciar los servicios y comparar los datos antes y después.

**Escenarios de aceptación**:

1. **Dada** una base vacía y PostgreSQL aún no preparado, **cuando** se inicia la API, **entonces** espera a que el almacenamiento esté saludable y aplica los cambios de esquema antes de aceptar solicitudes.
2. **Dado** que no se puede preparar el esquema, **cuando** falla el inicio, **entonces** la API no comienza a atender solicitudes y el error queda visible en el registro de inicio.
3. **Dado** un esquema actualizado, **cuando** se comprueba si quedan cambios de esquema pendientes, **entonces** no queda ninguno sin aplicar.
4. **Dado** que ya hay datos guardados, **cuando** se reinician los servicios, **entonces** los datos y el estado siguen disponibles.

### Historia 3 — Explicar el modelo y la procedencia de los datos (Prioridad: P2)

Como alumno que debe justificar su modelo, necesito una descripción comprensible de qué representa cada dato, de dónde procede y cómo se relaciona con los demás, para poder defender las decisiones de persistencia y reproducir la creación del esquema.

**Por qué esta prioridad**: El modelo y el diccionario forman parte de los entregables evaluados y permiten revisar el significado de los datos sin depender de conocer la implementación.

**Prueba independiente**: Revisar la documentación del modelo, el diccionario y la referencia al SQL de creación junto con sus fuentes y evidencias.

**Escenarios de aceptación**:

1. **Dado** el modelo documentado, **cuando** se revisan sus niveles conceptual, lógico y físico, **entonces** se pueden identificar entidades, relaciones, campos, claves, restricciones y el significado de cada fila.
2. **Dado** cualquier campo documentado, **cuando** se consulta el diccionario, **entonces** se encuentran su significado, procedencia, unidad o formato aplicable, tratamiento de ausencia y restricciones pertinentes.
3. **Dado** el SQL de creación del esquema, **cuando** se revisa su procedencia y la memoria o notebook, **entonces** se identifica que fue generado desde las migraciones y se encuentra el enlace solicitado.

## Casos límite

- Base de datos vacía, almacenamiento que aún no está saludable y fallo al preparar el esquema.
- Repetición exacta de una versión, recepción tardía de una versión antigua y llegada de una revisión posterior.
- Magnitud, profundidad, país u otro dato de origen ausente; no se debe confundir con cero.
- Rechazo por dato no tratable, cuyo motivo y original deben seguir siendo localizables.
- Vencimiento del plazo de conservación de una respuesta completa: solo afecta a esa respuesta, no a los originales duraderos de versiones y rechazos.
- Datos de reproducción histórica frente a datos live en almacenamiento y tablas compartidos, con el modo incluido en la identidad y disponible para distinguirlos en las consultas.

## Requisitos

Las etiquetas distinguen requisitos obligatorios derivados de las fuentes y decisiones del estudiante acordadas en esta solicitud. Las referencias a la solicitud remiten a la petición que originó esta especificación.

### Requisitos funcionales

- **FR-001 — [Requisito obligatorio]**: La documentación debe presentar un único modelo coherente en sus niveles conceptual, lógico y físico, y un diccionario que permita comprender los campos, las claves, las relaciones y las restricciones. **Fuente**: [Enunciado, §7](../../docs/source-of-truth/ENUNCIADO.md); [Guía, §2.5](../../docs/source-of-truth/Guia-entregables-observatorio-sismico.md).
- **FR-002 — [Requisito obligatorio]**: Deben poder distinguirse la identidad de un evento, su estado actual, cada versión recibida, la procedencia de los datos y los registros rechazados. Cada rechazo debe conservar su motivo y una referencia al original. **Fuente**: [Constitución, principios II y III](../../.specify/memory/constitution.md); [Enunciado, §§5–7 y 11](../../docs/source-of-truth/ENUNCIADO.md); [FAQ, §§12–13 y 16](../../docs/source-of-truth/FAQ-ALUMNADO.md).
- **FR-003 — [Requisito obligatorio]**: Los datos ausentes deben seguir siendo ausentes; las fechas, unidades, claves y restricciones deben conservar el significado definido por sus fuentes. La repetición de una versión no debe duplicar el resultado, y una versión antigua no debe sustituir a una más reciente. **Fuente**: [Constitución, principios II y III](../../.specify/memory/constitution.md); [Contrato de API](../../docs/source-of-truth/CONTRATO-API.md); [Enunciado, §§6–7 y 11](../../docs/source-of-truth/ENUNCIADO.md).
- **FR-004 — [Decisión del estudiante, acordada en la solicitud]**: Las respuestas GeoJSON completas de la fuente se conservan durante 90 días. Los originales correspondientes a cada versión de evento y a cada rechazo se conservan de forma duradera, incluso después de vencer aquel plazo.
- **FR-005 — [Decisión del estudiante, acordada en la solicitud]**: Los cambios del esquema se mantienen de forma versionada. Antes de aceptar consultas, la API espera a que el almacenamiento esté saludable y actualiza el esquema hasta su versión vigente. Si el almacenamiento o la actualización del esquema impiden el inicio, la API no debe quedar disponible y el error debe resultar visible. La solicitud fija SQLAlchemy y Alembic para la implementación posterior.
- **FR-006 — [Decisión del estudiante, acordada en la solicitud]**: El alcance no añade un servicio desplegable, no incorpora rutas de consulta y no cambia las respuestas existentes de la API. **Fuente de límite**: petición del usuario; [Constitución, principio V](../../.specify/memory/constitution.md); [Contrato de API](../../docs/source-of-truth/CONTRATO-API.md).
- **FR-007 — [Requisito obligatorio y forma de entrega acordada]**: El SQL de creación debe proceder de las migraciones y debe estar enlazado desde la memoria o notebook. **Fuente**: [Guía, §2.5](../../docs/source-of-truth/Guia-entregables-observatorio-sismico.md); petición del usuario.
- **FR-008 — [Decisión del estudiante, acordada en la aclaración]**: Los datos de reproducción histórica y live comparten almacenamiento y tablas. El modo (live o replay) forma parte de la identidad de los registros y las consultas deben permitir distinguirlo para evitar mezclar los datos de replay con los live. **Fuente**: [Constitución, principio III](../../.specify/memory/constitution.md); [Enunciado, §§5 y 11](../../docs/source-of-truth/ENUNCIADO.md); aclaración del usuario, 2026-11-05.

### Entidades conceptuales

- **Evento**: identidad estable del terremoto y estado actual consultable; no equivale a un mensaje recibido.
- **Versión recibida**: observación de una versión de un evento, con su fecha de actualización y su original.
- **Procedencia o respuesta de fuente**: respuesta GeoJSON y metadatos necesarios para explicar su origen y periodo de conservación.
- **Rechazo**: registro no aceptado como evento válido, ligado al original y acompañado de un motivo.
- **Estado de migración**: información que permite saber qué cambios del esquema están aplicados.
- **Modo de procesamiento**: live o replay; se almacena en las tablas compartidas y forma parte de la identidad de los registros para que las consultas puedan distinguir los modos.

Estas entidades expresan necesidades de información, no un diseño definitivo de tablas o columnas.

## Criterios de éxito medibles

- **SC-001**: La documentación permite identificar los tres niveles del modelo, sus entidades y relaciones, y explica el 100 % de los campos persistidos incluidos en el alcance mediante el diccionario.
- **SC-002**: En la secuencia de aceptación de la FAQ (evento E1, repetición, revisión posterior, repetición antigua y evento E2), el resultado es de 2 eventos actuales y 3 versiones distintas; E1 conserva como estado actual la revisión más reciente.
- **SC-003**: En los casos de aceptación con valores ausentes, rechazos y unidades conocidas, ningún nulo se convierte en cero, y el 100 % de rechazos conserva motivo y referencia al original.
- **SC-004**: Una respuesta GeoJSON completa deja de ser requerida tras 90 días; los originales de versiones y rechazos siguen conservados después de ese plazo.
- **SC-005**: Con una base vacía, la API no acepta solicitudes antes de que PostgreSQL esté saludable y las migraciones terminen; después del inicio no quedan migraciones pendientes.
- **SC-006**: Tras reiniciar los servicios, siguen presentes todos los registros y estados guardados antes del reinicio.
- **SC-007**: La entrega contiene el SQL de creación derivado de las migraciones y la memoria o notebook contiene un enlace accesible a ese SQL.
- **SC-008**: La compatibilidad se mantiene: no se añade un servicio Compose, no se incorporan rutas y no cambia la respuesta definida por el contrato actual.
- **SC-009**: Al guardar datos live y replay en el almacenamiento y las tablas compartidos, el modo forma parte de la identidad de cada registro; las consultas pueden filtrar o separar ambos modos, de modo que una consulta live no incluya registros replay y viceversa.

## Dependencias y supuestos

- PostgreSQL y el servicio API ya forman parte de la arquitectura de cinco servicios obligatorios; el cambio no añade otro servicio. **Fuente**: [Enunciado, §§2 y 9](../../docs/source-of-truth/ENUNCIADO.md).
- El contrato de API existente continúa siendo la referencia para su interfaz; este alcance no altera sus rutas ni respuestas. **Fuente**: [Contrato de API](../../docs/source-of-truth/CONTRATO-API.md).
- Se da por acordado el plazo de 90 días para respuestas GeoJSON completas y la conservación duradera de los originales de versiones y rechazos, por decisión expresa del usuario.
- Live y replay comparten almacenamiento y tablas; el modo integra la identidad de los registros y debe poder distinguirse en las consultas, según la decisión expresa del usuario.
- No se especifica aquí una estructura física detallada de tablas y columnas; esta especificación define qué debe permitir y documentar el modelo.

## Fuera de alcance y propuestas opcionales

- Cambios del contrato de API, nuevas rutas o servicios de Compose.
- Reconciliación exhaustiva de eventos eliminados o fusionados fuera de la ventana del feed; el enunciado la considera ampliación opcional. **Fuente**: [Enunciado, §5](../../docs/source-of-truth/ENUNCIADO.md).
- Informes automáticos o un panel de calidad: no se solicitan para esta petición y las fuentes los señalan como opcionales. **Fuente**: [Enunciado, §6](../../docs/source-of-truth/ENUNCIADO.md).
- Plan de implementación, tareas o código.

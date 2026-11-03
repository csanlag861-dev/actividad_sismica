# Feature Specification: Subagentes especializados

**Feature Branch**: `sin especificar`

**Created**: 2026-11-03

**Status**: Draft

**Input**: User description: "Quiero que me crees los siguientes subagentes: Backend, Frontend, Analista de Requisitos (Product Manager), QA Tester, Deploy. Consulta los documentos relevantes de `docs/source-of-truth/` y `constitution.md`. Añade el subagente Verificador para contrastar la tarea solicitada, el plan y lo realizado, y comprobar que se ha creado lo requerido por las fuentes de verdad."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Alinear el trabajo con los requisitos (Priority: P1)

Como responsable del proyecto, quiero que el Analista de Requisitos traduzca las fuentes oficiales del observatorio en necesidades priorizadas y comprobables, para que el trabajo de los demás agentes cumpla el enunciado sin ampliar accidentalmente el alcance.

**Why this priority**: Los demás agentes necesitan una interpretación común y trazable de los requisitos antes de proponer o realizar cambios.

**Independent Test**: Presentar una tarea de ejemplo al agente y comprobar que identifica los requisitos aplicables, su prioridad y sus criterios de aceptación, y distingue las obligaciones de las decisiones del estudiante y las propuestas opcionales.

**Acceptance Scenarios**:

1. **Given** que se le asigna una tarea del observatorio, **When** el agente analiza su alcance, **Then** identifica el enunciado y el contrato aplicables y señala la evidencia o criterio que permite verificar cada requisito.
2. **Given** que una guía ofrece una opción o ejemplo, **When** esa opción no es obligatoria en las fuentes principales, **Then** el agente no la presenta como requisito del proyecto.

---

### User Story 2 - Implementar por especialidad (Priority: P1)

Como responsable del proyecto, quiero asignar el trabajo de servicios o de la interfaz a agentes especializados, para que cada uno actúe dentro de su área y mantenga coherencia con el contrato y los requisitos compartidos.

**Why this priority**: Backend y Frontend son áreas centrales del sistema y deben coordinarse sin romper las responsabilidades ni el contrato entre ellas.

**Independent Test**: Asignar una tarea representativa a cada agente y comprobar que identifica los requisitos relevantes de su área, los límites con otras áreas y el resultado que debe entregar.

**Acceptance Scenarios**:

1. **Given** una tarea de datos o servicios, **When** se asigna al agente Backend, **Then** considera el flujo de ingesta, persistencia, calidad, revisiones y consulta de datos según los requisitos aplicables.
2. **Given** una tarea de visualización, **When** se asigna al agente Frontend, **Then** considera el uso de los datos procesados por el sistema y la coherencia de mapa, tabla, filtros, análisis y estados visibles.
3. **Given** que una tarea cruza límites entre áreas, **When** el agente detecta una dependencia o incompatibilidad con otra área, **Then** la hace explícita en lugar de asumir un contrato nuevo.

---

### User Story 3 - Verificar resultados con evidencia (Priority: P2)

Como responsable del proyecto, quiero que el QA Tester compruebe los criterios de aceptación y registre resultados observados, para distinguir el comportamiento probado de lo que solo está implementado.

**Why this priority**: La evaluación exige pruebas ejecutadas y evidencia, no afirmaciones basadas únicamente en el código.

**Independent Test**: Entregar al agente una prueba de aceptación y comprobar que informa del resultado esperado, el resultado observado y la evidencia; si no se ejecutó, no la declara superada.

**Acceptance Scenarios**:

1. **Given** una prueba del enunciado, **When** el agente la ejecuta o verifica con evidencia disponible, **Then** documenta por separado el resultado esperado y el observado.
2. **Given** que una prueba no puede ejecutarse o falla, **When** el agente informa del estado, **Then** identifica la limitación o el fallo sin declarar éxito.

---

### User Story 4 - Preparar un despliegue reproducible (Priority: P2)

Como responsable del proyecto, quiero que el agente Deploy prepare y compruebe el procedimiento de puesta en marcha y publicación descrito por el proyecto, para que la demostración sea reproducible y no exponga credenciales.

**Why this priority**: La entrega requiere ejecutar conjuntamente los servicios previstos, publicar el panel y explicar cómo activar el backend y el acceso durante la defensa.

**Independent Test**: Pedir al agente que revise un procedimiento de despliegue y comprobar que cubre los servicios obligatorios, persistencia, preparación, publicación y protección de secretos, sin exigir infraestructura adicional.

**Acceptance Scenarios**:

1. **Given** una propuesta de puesta en marcha, **When** el agente la contrasta con las fuentes, **Then** contempla los cinco servicios obligatorios del backend y el procedimiento de publicación y acceso del panel.
2. **Given** una configuración que contiene o podría contener credenciales, **When** el agente revisa su uso o documentación, **Then** exige mantenerlas fuera de archivos públicos y del control de versiones.
3. **Given** una mejora opcional de infraestructura, **When** no es necesaria para cumplir el enunciado, **Then** el agente la identifica como opcional y no como condición de entrega.

---

### User Story 5 - Verificar la correspondencia del trabajo (Priority: P1)

Como responsable del proyecto, quiero que el Verificador contraste la tarea solicitada, el plan propuesto y el trabajo realizado con los requisitos aplicables de las fuentes de verdad, para detectar omisiones o desviaciones antes de dar la tarea por terminada.

**Why this priority**: Permite comprobar que el resultado responde a lo pedido y que no se han omitido entregables obligatorios definidos en las fuentes, o que se han creado features de más.

**Independent Test**: Dar al agente una solicitud, un plan, un resultado realizado y los documentos fuente aplicables; comprobar que identifica coincidencias, desviaciones y requisitos o entregables no cubiertos, con referencias a su origen.

**Acceptance Scenarios**:

1. **Given** una tarea solicitada, un plan y el trabajo realizado, **When** el Verificador los contrasta, **Then** informa de si el resultado cumple la solicitud y el plan, y señala cada discrepancia concreta.
2. **Given** requisitos aplicables en `docs/source-of-truth/`, **When** el Verificador revisa el resultado, **Then** comprueba si se creó lo solicitado por esas fuentes y enumera los elementos obligatorios ausentes o incompletos con referencia a la fuente.
3. **Given** que el plan contradice la solicitud o un requisito obligatorio de las fuentes, **When** el Verificador detecta el conflicto, **Then** lo informa explícitamente y no redefine el alcance ni declara la tarea conforme por asumir que el plan prevalece.

### Edge Cases

- Si dos documentos parecen discrepar, los agentes distinguen los requisitos del enunciado y el contrato de las aclaraciones de las guías, y señalan la discrepancia en vez de inventar una regla.
- Si una solicitud no especifica un resultado verificable, el Analista de Requisitos explicita la ambigüedad y propone criterios o pide la decisión que falte antes de tratarla como requisito.
- Si una prueba, publicación o servicio externo no está disponible, QA Tester y Deploy informan de lo que no pudo verificarse y no presentan un resultado supuesto como observado.
- Si no se proporciona un plan, un resultado realizado o una fuente aplicable, el Verificador identifica qué comparación no puede completar e informa de esa limitación sin inferir que hubo conformidad.
- Si el plan y el resultado coinciden entre sí pero omiten un requisito obligatorio de las fuentes, el Verificador marca la omisión como incumplimiento aunque la ejecución siga el plan.
- Si una petición propone una ampliación, complejidad o dato que las fuentes no requieren, el agente responsable la marca como decisión o propuesta y explica su impacto en el alcance.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: La solución MUST proporcionar exactamente seis perfiles identificables: Backend, Frontend, Analista de Requisitos (Product Manager), QA Tester, Deploy y Verificador.
- **FR-002**: Cada perfil MUST declarar su propósito, responsabilidades, resultados esperados y límites respecto a los otros perfiles.
- **FR-003**: Los seis perfiles MUST consultar y respetar la constitución del proyecto y los documentos aplicables de `docs/source-of-truth/` antes de recomendar o realizar trabajo del observatorio.
- **FR-004**: Los perfiles MUST distinguir entre requisitos obligatorios, decisiones del estudiante y propuestas opcionales; las guías y FAQ MUST aclarar, pero no añadir obligaciones o modificar la rúbrica.
- **FR-005**: El perfil Analista de Requisitos MUST convertir necesidades en requisitos priorizados, criterios de aceptación comprobables y dependencias explícitas, conservando la trazabilidad con las fuentes.
- **FR-006**: El perfil Backend MUST orientar su trabajo a los requisitos de ingesta, tratamiento fiel y trazable de datos, idempotencia y revisiones, persistencia, cuarentena y contrato de consulta aplicables al backend.
- **FR-007**: El perfil Frontend MUST orientar su trabajo a presentar los datos procesados por el sistema y mantener coherentes el mapa, la tabla, los filtros, el análisis y los estados de actualización o error requeridos, conforme a la interfaz de consulta acordada.
- **FR-008**: El perfil QA Tester MUST comprobar criterios de aceptación, informar resultados esperados y observados, aportar evidencia disponible y señalar pruebas no ejecutadas o fallidas sin marcarlas como superadas.
- **FR-009**: El perfil Deploy MUST orientar la puesta en marcha y publicación a los servicios y procedimientos exigidos por las fuentes, favorecer una ejecución reproducible y evitar la exposición de credenciales.
- **FR-010**: Cada perfil MUST señalar dependencias, incompatibilidades y decisiones que afecten a otro perfil o al contrato común antes de asumir un comportamiento no especificado.
- **FR-011**: Ningún perfil MUST convertir una mejora opcional en condición de cumplimiento ni exigir herramientas, servicios o infraestructura que las fuentes no requieren.
- **FR-012**: Los perfiles MUST preservar las limitaciones y el significado de los datos: no tratar ausencias como cero, no presentar datos ficticios como análisis real y no presentar el análisis sísmico como predicción.
- **FR-013**: El perfil Verificador MUST contrastar la tarea solicitada, el plan propuesto y el trabajo realizado, e informar de coincidencias, desviaciones y aspectos no verificables.
- **FR-014**: El perfil Verificador MUST comprobar el trabajo realizado frente a los requisitos y entregables obligatorios aplicables de `docs/source-of-truth/`, citar su origen e identificar elementos ausentes o incompletos.
- **FR-015**: El perfil Verificador MUST señalar los conflictos entre solicitud, plan y fuentes, sin considerar que la coincidencia entre plan y resultado basta para demostrar cumplimiento.
- **FR-016**: El perfil Verificador MUST distinguir la revisión de correspondencia y completitud de la ejecución de pruebas, que corresponde al perfil QA Tester.

### Key Entities *(include if feature involves data)*

- **Perfil de agente**: Especialidad con propósito, responsabilidades, resultados esperados, límites y criterios de actuación.
- **Requisito del proyecto**: Necesidad atribuida a una fuente, clasificada como obligatoria, decisión del estudiante o propuesta, y vinculada a un criterio comprobable.
- **Evidencia de verificación**: Resultado observado de una comprobación, separado de su resultado esperado y del estado de ejecución.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Los seis perfiles solicitados están disponibles y se pueden distinguir por nombre y especialidad.
- **SC-002**: En una revisión de aceptación, el 100 % de los perfiles identifica las fuentes del proyecto y diferencia requisitos obligatorios de decisiones y propuestas opcionales.
- **SC-003**: Una tarea de ejemplo asignada a cada perfil produce responsabilidades y límites coherentes con su especialidad, sin duplicar la autoridad de otro perfil.
- **SC-004**: En todas las pruebas de aceptación revisadas, QA Tester separa el resultado observado del esperado y no declara superada ninguna prueba no ejecutada.
- **SC-005**: Una revisión del procedimiento de despliegue confirma los servicios obligatorios, el carácter reproducible de la puesta en marcha y la ausencia de credenciales expuestas, sin exigir infraestructura opcional.
- **SC-006**: En cada revisión de aceptación del Verificador, el resultado incluye la cobertura de la solicitud, del plan y de los requisitos fuente aplicables, y registra toda discrepancia o elemento obligatorio ausente que se haya detectado.

## Assumptions

- Los agentes son asistentes de trabajo para este repositorio y deben adaptar sus acciones al alcance y estado real del proyecto.
- El enunciado y su rúbrica determinan las tareas obligatorias; el contrato define las obligaciones de consulta e integración; la constitución fija principios; la guía y las FAQ explican sin ampliar los requisitos.
- Los perfiles pueden coordinarse cuando una tarea cruza especialidades, pero deben identificar explícitamente sus límites y dependencias.
- La solicitud establece seis roles y no solicita agregar otros perfiles ni obligar a adoptar herramientas o infraestructura opcionales.
- El Verificador revisa la correspondencia y completitud del trabajo, mientras que QA Tester se centra en ejecutar o comprobar pruebas y registrar evidencia observada.

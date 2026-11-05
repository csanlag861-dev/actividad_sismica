# Research: Subagentes especializados

## Decisión 1: Ubicación y descubrimiento de los perfiles

- **Decision**: Crear nueve perfiles como archivos Markdown `*.agent.md` en `.github/agents/`.
- **Rationale**: Es la ubicación de ámbito workspace documentada para agentes personalizados de VS Code y permite compartir las definiciones en el repositorio.
- **Alternatives considered**: Perfiles de usuario fuera del repositorio; instrucciones globales; skills. Se descartan porque la petición define subagentes específicos de este proyecto, no preferencias personales ni reglas siempre activas.

## Decisión 2: Forma de cada perfil

- **Decision**: Cada archivo incluirá frontmatter YAML con `description` específica y un cuerpo que delimite objetivo, responsabilidades, límites, forma de trabajo y resultados.
- **Rationale**: La descripción permite que el agente aparezca y sea descubrible para la delegación; el cuerpo concreta su especialidad sin mezclarla con los otros perfiles.
- **Alternatives considered**: Un solo perfil generalista o una configuración compartida sin definiciones individuales. Se descartan porque dificultan distinguir responsabilidades y permisos.

## Decisión 3: Permisos de herramientas

- **Decision**: Asignar solo los alias necesarios para el rol y mantener el Verificador en modo de solo lectura y búsqueda.
- **Rationale**: La referencia de personalización recomienda conjuntos mínimos y admite los alias `read`, `search`, `edit` y `execute`. El Verificador queda limitado a lectura y búsqueda para que no cambie lo que revisa. Backend y Frontend pueden requerir edición y ejecución; Analista de Requisitos, lectura, búsqueda y edición; QA, lectura, búsqueda y ejecución; Deploy, lectura, búsqueda y ejecución, con edición solo si la tarea asignada exige modificar archivos de despliegue. Los alias disponibles pueden depender del harness y de la sesión activa.
- **Alternatives considered**: Omitir permisos o conceder todas las herramientas a cada perfil. Se descartan porque no expresan límites y aumentan el acceso innecesario.

## Decisión 4: Jerarquía y evidencia del proyecto

- **Decision**: Usar la constitución como principios transversales; el enunciado y su rúbrica como fuente de obligaciones; el contrato como especificación de la API; las guías y FAQ como aclaraciones, no como requisitos nuevos. Exigir a QA que distinga resultado esperado, observado y evidencia; el Verificador registra cobertura, discrepancias y omisiones frente a las fuentes.
- **Rationale**: Esta jerarquía está expresada en la constitución y las fuentes, y evita que los agentes conviertan opciones en obligaciones o que declaren resultados no comprobados.
- **Alternatives considered**: Tratar todas las guías como requisitos equivalentes o combinar QA y Verificador en un solo rol. Se descartan porque contradirían la gobernanza documentada y la distinción de responsabilidades de la especificación.

## Decisión 5: Coordinación, documentación y mentoría

- **Decision**: Mantener Coordinador / Orchestrator Agent, Documentador y Profesor / Mentor Agent como perfiles diferenciados. El Coordinador puede delegar y revisar, pero solicita aprobación ante cambios materiales; el Documentador edita documentos pero no modifica código; el Mentor hace coaching interactivo y limita el seguimiento al contexto accesible salvo registro aprobado por el alumno.
- **Rationale**: Estas fronteras evitan que la orquestación sustituya la especialidad, que la documentación invente resultados o cambie requisitos y que el seguimiento académico prometa memoria persistente no disponible.
- **Alternatives considered**: Un solo perfil generalista que coordine, implemente, documente y enseñe; o almacenar automáticamente un historial del alumno. Se descartan por mezcla de responsabilidades, riesgo de cambios no autorizados y persistencia de datos personales de aprendizaje no solicitada.

## Decisión 6: Validación

- **Decision**: Comprobar la forma y el contenido de cada perfil, revisar permisos y responsabilidades contra el contrato de perfiles, y confirmar el descubrimiento y uso en el selector de agentes de VS Code.
- **Rationale**: La validación cubre tanto la estructura de los archivos como el propósito funcional de separación de roles. La documentación oficial confirma `.github/agents/` como ubicación workspace, archivos `.agent.md`, y que `description` es necesaria en el esquema de Copilot. La disponibilidad de alias y el comportamiento pueden variar por harness; hay que verificar los agentes en el harness previsto, no inferir disponibilidad solo del archivo.
- **Alternatives considered**: Confiar solo en una comprobación textual o declarar funcionalidad sin abrir los perfiles. Se descartan porque no verificarían el descubrimiento real ni las responsabilidades.

**Referencia consultada**: Documentación oficial de VS Code sobre [custom agent file locations](https://code.visualstudio.com/docs/agent-customization/custom-agents#_custom-agent-file-locations), [file structure and fields](https://code.visualstudio.com/docs/agents/reference/custom-agent-file-reference#_file-structure), [tools](https://code.visualstudio.com/docs/agents/reference/custom-agent-file-reference#_tools), y [harness compatibility](https://code.visualstudio.com/docs/agents/reference/custom-agent-file-reference#_harness-compatibility).

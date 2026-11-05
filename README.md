# Observatorio de actividad sísmica

## Preparar GitHub Spec Kit en otro equipo (Windows)

Este repositorio ya incluye la configuración de GitHub Spec Kit para GitHub Copilot (`.specify/` y `.github/skills/`). Al clonar el repositorio no hay que volver a inicializar Spec Kit; solo instalar su CLI y abrir el proyecto con Copilot.

### Requisitos

- Windows 10/11 y PowerShell.
- Git.
- Python 3.11 o posterior.
- Visual Studio Code, la extensión GitHub Copilot y una sesión iniciada.
- `uv`, recomendado para instalar el CLI de Spec Kit.

Si faltan Git o Python, se pueden instalar con WinGet:

```powershell
winget install --id Git.Git -e
winget install --id Python.Python.3.12 -e
```

Instala `uv` y el CLI oficial de Spec Kit:

```powershell
winget install --id astral-sh.uv -e
```

Cierra y vuelve a abrir PowerShell para actualizar el `PATH`, y ejecuta:

```powershell
uv --version
uv tool install specify-cli
specify version
specify check
```

`specify check` comprueba las herramientas del entorno. GitHub Copilot de VS Code no necesita instalar un CLI adicional.

### Clonar y abrir este proyecto

Primero sube los cambios que quieras usar en casa a GitHub. En el otro equipo, abre PowerShell y ejecuta:

```powershell
git clone https://github.com/csanlag861-dev/actividad_sismica.git
Set-Location .\actividad_sismica
code .
```

Abre Copilot Chat en VS Code y selecciona el agente que quieras. Los perfiles del proyecto están en `.github/agents/`:

- **Coordinador / Orchestrator Agent**: interpreta peticiones, delega trabajo y supervisa dependencias, calidad y decisiones de alcance.
- **Analista de Requisitos (Product Manager)**: convierte necesidades y fuentes del proyecto en requisitos priorizados y criterios de aceptación.
- **Backend** y **Frontend**: implementan los servicios de datos y el panel.
- **QA Tester**: ejecuta pruebas y registra resultados observados y evidencia.
- **Verificador**: compara la solicitud, el plan, los cambios y las fuentes; solo lectura.
- **Deploy**: prepara y valida la ejecución y publicación reproducibles.
- **Documentador**: mantiene documentación técnica y didáctica, evidencia y material para la defensa.
- **Profesor / Mentor Agent**: hace preguntas, evalúa comprensión y refuerza conceptos para ayudar al alumno a defender y modificar el proyecto con autonomía.

> **Importante:** no ejecutes `specify init` dentro de este repositorio clonado. Ya tiene Spec Kit inicializado y sus archivos se descargan con Git. `specify init` es para crear un proyecto nuevo o integrar Spec Kit en otro repositorio; puede sobrescribir archivos gestionados que tengan el mismo nombre.

### Flujo de Spec Kit para una nueva funcionalidad

Los siguientes son comandos para escribir en el **chat del agente** de VS Code, no en PowerShell. Ejecuta cada paso por separado y revisa el resultado antes de continuar:

```text
/speckit-specify Describe aquí la funcionalidad que quieres crear.
/speckit-clarify
/speckit-plan
/speckit-tasks
/speckit-analyze
/speckit-implement
/speckit-converge
```

Si la especificación no necesita aclaraciones, puedes omitir `/speckit-clarify`. Si `/speckit-analyze` o `/speckit-converge` encuentra problemas o tareas pendientes, corrige lo indicado y repite la implementación/verificación hasta que el resultado esté completo.

La constitución del proyecto ya está en `.specify/memory/constitution.md`; no hace falta volver a crearla. Para comenzar un proyecto completamente nuevo, instala el CLI y ejecuta en PowerShell:

```powershell
specify init mi-proyecto --integration copilot
Set-Location .\mi-proyecto
code .
```

Consulta la [guía oficial de instalación de Spec Kit](https://github.github.io/spec-kit/installation.html) y la [guía para proyectos existentes](https://github.github.io/spec-kit/guides/existing-projects.html).

## Ejecutar el mapa de demostración

El frontend requiere Node.js 22.12 o posterior dentro de la rama 22, o Node.js 24. Desde PowerShell:

```powershell
Set-Location .\frontend
npm ci
npm start
```

Abre <http://localhost:4200>. Para requisitos del proyecto, fuentes, despliegue y validación del observatorio, consulta [`docs/source-of-truth/README.md`](docs/source-of-truth/README.md).
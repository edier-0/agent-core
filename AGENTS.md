# AGENTS.md — agent-core

Software Factory Portátil y Arnés de Ingeniería de Software para Agentes de IA.

## Stack y Estructura
- **Tecnologías**: Arnés agnóstico basado en Markdown, YAML, JSON, Bash/PowerShell y Model Context Protocol (MCP).
- **Arquitectura y Modelo Fábrica-Instalador**:
  - En este repositorio fuente (`agent-core`), los componentes maestros residen unificados dentro de `.agents/` (`.agents/agents/`, `.agents/workflows/`, `.agents/skills/`, `.agents/templates/`, `.agents/mcp/`) bajo el principio de "Cero Basura en la Raíz".
  - En proyectos cliente destino (`/init` o `/adopt`), el arnés se empaqueta e instala estrictamente dentro de `.agent/` ("Cero Basura en la Raíz").
- **Rutas Clave**: Código y arnés unificados en `.agents/` (`.agents/agents/`, `.agents/skills/`, `.agents/workflows/`, `.agents/mcp/`), especificaciones en `specs/`, adaptadores y plantillas en `.agents/templates/`.

## Configuración de Git y Flujo de Trabajo
- `GIT_WORKFLOW_MODE`: "branch_pr"
- `PR_TARGET_BRANCH`: "main"
- `AGENT_CAN_MERGE`: false

## Convenciones y Reglas de Dominio
- **Idioma**: Toda la documentación, prompts, workflows, especificaciones y mensajes operan exclusivamente en **español**.
- **Rutas**: Utilizar siempre barras diagonales directas (`/`) en toda referencia a archivos.
- **Estándar de Encuestas al Usuario**: Todas las preguntas de clarificación, entrevistas o decisiones dirigidas al usuario deben formularse obligatoriamente como tarjetas de selección (herramienta interactiva nativa `ask_question` o tarjetas visuales con opción recomendada de primero y opción abierta para escribir otra respuesta libre).

## Forma de Trabajar (SDD + TDD Innegociable)
- **Spec-Driven Development**: Ningún cambio sin spec aprobada en `specs/NNN-nombre/` (`spec.md` en EARS, `plan.md` y `tasks.md`).
- **Test-Driven Development**: Rojo (test primero que falle) -> Verde (código mínimo) -> Refactor.
- **Commits atómicos**: `<tipo>(<NNN>): <Tn> - <descripción en español>` con staging quirúrgico (`git add <rutas>`).
- **Política de Git Push Único y Controlado**: Queda estrictamente prohibido realizar `git push` intermedio durante las tareas individuales TDD (`T1` a `Tn`). Se autoriza exactamente un único `git push -u origin <rama>` en la Fase 3 de `/review-and-pr` tras la validación integral y aprobación de `@reviewer` en `/sdd-validate`, como paso indispensable para abrir el Pull Request y activar las compuertas de CI en GitHub.

## Orquestación Multiagente y Visibilidad en UI
- **Veto de Auto-Edición y Consola para el Coordinador**:
  - En plataformas multiagente (Antigravity, OpenCode), el hilo principal opera estrictamente como `@coordinator`.
  - Queda **terminantemente prohibido que el `@coordinator` ejecute herramientas de edición (`edit_file`, `write_file`) o consola de desarrollo (`run_command`)** directamente en el hilo principal.
  - Toda creación o edición de código, pruebas o arnés debe delegarse sin excepción a `@implementer` en su subagente (`invoke_subagent` con `Workspace: "inherit"`).
  - Toda redacción de especificaciones, planes y tareas se delega a `@planner`.
  - Toda auditoría de código, tests o diff se delega a `@reviewer`.
- **Delegación Exclusiva de Consola Git a `@implementer`**:
  - Todo comando de consola de Git (inspección de estado, ramas, checkout, rebase, staging quirúrgico, commit, push, limpieza) es ejecutado **estricta y exclusivamente por `@implementer`**.
  - `@coordinator` y `@planner` tienen prohibido ejecutar comandos de consola Git directamente.
- **En Google Antigravity**:
  - **Granularidad de `update_steps` (Límite de 12 Pasos)**: Es obligatorio invocar `update_steps` para pintar y actualizar en vivo el *Toggle Overview Panel*. `update_steps` mapea **exclusivamente las 7-8 Fases Canónicas del Flujo SDD** (evitando desbordar el límite de la API):
    1. `Entrevista de Requerimientos (/sdd-spec)`
    2. `Clarificación QA (/sdd-clarify)`
    3. `Plan Técnico (/sdd-plan)`
    4. `Desglose de Tareas (/sdd-tasks)`
    5. `Implementación TDD (/sdd-implement)`
    6. `Validación Integral (/sdd-validate)`
    7. `Revisión y Pull Request (/review-and-pr)`
  - **Progreso Fino de Tareas**: El avance de tareas atómicas (`T1`..`Tn`) **no se agrega a `update_steps`**; se comunica en tiempo real mediante `update_focus` (ej. `"Ejecutando T2: Servicio de autenticación con TDD"`) y marcando casillas `- [x]` en `specs/NNN-nombre/tasks.md`.
  - Es obligatorio despachar subagentes (`@planner`, `@implementer`, `@reviewer`) mediante `invoke_subagent` fijando `Workspace: "inherit"`. Queda estrictamente prohibido simular los roles en texto si se dispone de estas herramientas.
  - Formular decisiones y compuertas de fase mediante tarjetas interactivas `ask_question`.
- **Estandarización de Prompt para Subagentes (`TypeName: "self"`)**:
  - Para neutralizar la herencia del veto de auto-edición del coordinador en subsesiones, todo despacho vía `invoke_subagent` debe incluir obligatoriamente la directiva:
    ```
    Actúas en el rol de @<rol> [o @implementer / @planner / @reviewer].
    DIRECTIVA DE AUTORIDAD Y PERMISOS:
    Dentro del alcance de tu rol, tus capacidades de creación/edición de archivos y ejecución de comandos de consola están plenamente activas y autorizadas.
    El veto de auto-edición y consola aplica estricta y exclusivamente al hilo principal del @coordinator.
    OBJETIVO: <descripción atómica de la tarea>.
    ```
- **Protocolo de Serialización Git y Prevención de `.git/index.lock` (Windows)**:
  - Al operar con `Workspace: "inherit"`, la ejecución concurrente de comandos Git genera excepciones `EBUSY / index.lock` en el sistema de archivos de Windows.
  - Se establece una regla de **exclusión mutua**: el `@coordinator` no invoca ningún comando de Git mientras un subagente `@implementer` esté activo.
  - Todo el ciclo Git (status, add, commit, push) se ejecuta de forma sincrónica y exclusiva dentro del turno de `@implementer`. Si se detecta un `.git/index.lock` huérfano por interrupción abrupta, se verifica su obsolescencia y se remueve antes de operar.
- **En OpenCode**:
  - Es obligatorio despachar a los especialistas configurados en `.opencode/agents/` (`@planner`, `@implementer`, `@reviewer`) para que la TUI materialice el árbol jerárquico de sub-sesiones y muestre en vivo el streaming de comandos y pruebas.
  - Los comandos slash operan nativamente desde `.opencode/commands/`.
- **En Entornos Monohilo (Claude Code, Cursor, Codex, Windsurf, Aider)**:
  - Es obligatorio anunciar de forma visible en el chat cada transición de fase mediante: `[CAMBIO DE ROL: @agente]`.
- **Compuertas de Pausa y Confirmación Humana**:
  - **Pausas Innegociables**: Al finalizar la spec (`/sdd-clarify`), al finalizar el plan técnico y tareas (`/sdd-plan`, `/sdd-tasks`), al concluir la validación integral (`/sdd-validate`) y antes de abrir o fusionar un Pull Request (`/review-and-pr`).
  - **Ejecución TDD**: En la fase de implementación (`/sdd-implement`), el avance de tareas es secuencial y reporta su estado al checklist. El usuario puede pausar tras cada tarea atómica o autorizar autonomía continua para completar el lote de tareas de la spec.

## Límites de Autoridad
- **Siempre**: Seguir SDD y TDD; ejecutar compuertas de calidad pre-commit; auditar ausencia de credenciales.
- **Pregunta antes**: Modificar arquitectura; abrir PR; fusionar PR; borrar ramas locales o remotas.
- **Nunca**: Modificar código sin test previo en rojo; comitear secretos o tokens hardcodeados; usar `git add .` indiscriminado; pushear código intermedio, roto o a medio implementar a ramas compartidas (único push remoto autorizado tras validación integral en `/review-and-pr`).

---
description: Auto-adopta agent-core reorganizando quirúrgicamente los archivos expuestos a .agent/ y configurando los conectores nativos por agente (Claude Code, OpenCode, Codex, Antigravity, Cursor, etc.).
---

# Workflow: Auto-Adopción y Reorganización del Entorno (`/adopt`)

Este workflow detecta el entorno de ejecución activo (o la preferencia indicada por el usuario), reorganiza quirúrgicamente cualquier archivo del arnés expuesto en la raíz hacia `.agent/` y genera los conectores nativos necesarios manteniendo intacta la fuente canónica de la verdad en `.agent/`.

---

## Fase 1: Evaluación y Normalización del Repositorio (Cero Basura en Raíz)

1. **Detección y Reorganización de Árbol Expuesto**:
   - Comprueba si existen carpetas del arnés expuestas en la raíz (`agents/`, `workflows/`, `skills/`, `templates/`, `mcp/`).
   - Si existen en la raíz en lugar de `.agent/`:
     - Crea el directorio canónico `.agent/` si no existe.
     - Mueve de inmediato:
       - `agents/` $\rightarrow$ `.agent/agents/`
       - `workflows/` $\rightarrow$ `.agent/workflows/`
       - `skills/` $\rightarrow$ `.agent/skills/`
       - `templates/` $\rightarrow$ `.agent/templates/`
       - `mcp/` $\rightarrow$ `.agent/mcp/`
   - Si existe `README.md` en la raíz y contiene la firma determinista `<!-- AGENT-CORE-ROOT-README -->` o la frase `Fábrica de Software Agéntica Portátil (*Portable Agentic Software Factory*)`:
     - Se identifica inequívocamente como el manual de `agent-core`: muévelo a `.agent/README.md` (preservando el manual de consulta offline del arnés).
     - Inicializa un `README.md` básico para la aplicación del usuario.
     - *(Si no contiene dicha firma determinista, se preserva intacto como el README legítimo del proyecto del usuario)*.
   - Si existe `plan.md` en la raíz (plan interno de construcción de `agent-core`):
     - Muévelo a `.agent/docs/history/agent-core-plan.md` para no contaminar la raíz.
2. **Comprobación de Archivos Fundacionales**:
   - Comprueba si existen:
     - `AGENTS.md` (raíz).
     - `MEMORY.md` (raíz).
     - `docs/constitution.md`.
   - Si faltan estos archivos:
     - Invoca automáticamente el workflow `/init` para completar el proceso de bootstrap y entrevista técnica.

---

## Fase 2: Identificación del Agente Activo

El agente detecta su propio entorno de ejecución o consulta al usuario:
- **OpenCode**: Entorno de desarrollo multiagente nativo en terminal/IDE.
- **Claude Code**: Terminal CLI de Anthropic (monohilo con soporte de comandos `.claude/commands/`).
- **Codex**: Agente / CLI de OpenAI.
- **Antigravity**: Entorno Google Antigravity con soporte de subagentes y skills nativas.
- **Cursor / Windsurf / Aider**: IDEs y asistentes basados en reglas Markdown/MDC.

---

## Fase 3: Generación de Conectores Nativos por Plataforma

### 1. Si el entorno es OpenCode:
- **Configuración de agentes y comandos nativos**:
  - Crea los directorios `.opencode/agents/` y `.opencode/commands/`.
  - Copia físicamente los subagentes desde `.agent/agents/` hacia `.opencode/agents/` (`coordinator.md`, `planner.md`, `implementer.md`, `reviewer.md`), preservando permisos y directivas.
  - Copia físicamente los workflows desde `.agent/workflows/` hacia `.opencode/commands/` (mapeando los comandos slash `/sdd-*` ejecutables en la TUI).
  - *Regla técnica innegociable*: Emplea exclusivamente **copia física de archivos**, prohibiendo el uso de enlaces simbólicos (*symlinks*) que fallan en Windows sin privilegios elevados.
- **Configuración de manifiesto `opencode.json`**:
  - Genera `opencode.json` en la raíz a partir de `.agent/templates/adapters/opencode.json.template`.
  - Declara `"agents": { "directory": ".opencode/agents" }` y `"commands": { "directory": ".opencode/commands" }` para que el motor de OpenCode arme el árbol jerárquico de control y permita visualizar las sub-sesiones en tiempo real.
  - Configura los servidores MCP respetando la regla DevSecOps de variables de entorno (`{env:...}`).

### 2. Si el entorno es Claude Code:
- **Archivo de instrucciones principal (`CLAUDE.md`)**:
  - Genera `CLAUDE.md` en la raíz a partir de `.agent/templates/adapters/CLAUDE.md.template`.
  - Configura el protocolo de cambio de rol explícito (`[CAMBIO DE ROL: @planner]`, `[CAMBIO DE ROL: @implementer]`, `[CAMBIO DE ROL: @reviewer]`).
- **Comandos Slash Nativos (`.claude/commands/`)**:
  - Crea el directorio `.claude/commands/`.
  - Genera los comandos personalizados (`sdd-spec.md`, `sdd-clarify.md`, `sdd-plan.md`, `sdd-tasks.md`, `sdd-implement.md`, `sdd-validate.md`, `sdd-change.md`, `sdd-status.md`, `review-and-pr.md`) enlazando directamente al contenido y directivas de `.agent/workflows/`. Esto habilita autocompletado nativo en el CLI de Claude Code.

### 3. Si el entorno es Codex:
- **Archivo de directivas (`CODEX.md`)**:
  - Genera `CODEX.md` en la raíz a partir de `.agent/templates/adapters/CODEX.md.template`.
  - Vincula el flujo de comandos a `AGENTS.md` y `.agent/workflows/`.

### 4. Si el entorno es Cursor, Windsurf o Aider:
- **Regla nativa de agente**:
  - Si es Cursor: genera `.cursor/rules/agent-core.mdc` a partir de `.agent/templates/adapters/generic-agent.md.template`.
  - Si es Windsurf: genera `.windsurfrules` a partir de dicha plantilla.
  - Si es Aider o genérico: genera `.aider.conf.yml` o referencia a `AGENTS.md`.

### 5. Si el entorno es Antigravity:
- **Registro de Skills para Comandos Slash**:
  - Organiza y verifica que las skills residan en `.agent/skills/<nombre>/SKILL.md` (o `.agents/skills/<nombre>/SKILL.md`) con encabezados frontmatter válidos (`name: <nombre>`, `description: ...`) para que el chat de Antigravity descubra y autocomplete los comandos slash (`/sdd`, `/sdd-plan`, etc.).
- **Gobernanza de Interfaz y Subagentes Nativos**:
  - Asegura que `AGENTS.md` en la raíz contenga las directivas de visibilidad obligatorias:
    - Invocación de `update_steps` para pintar y actualizar en vivo el *Toggle Overview Panel*.
    - Invocación de `invoke_subagent` fijando obligatoriamente `Workspace: "inherit"` para materializar los subagentes en el panel lateral (prohibiendo simulación en texto).
    - Uso de `update_focus` para el foco superior y `ask_question` para las tarjetas interactivas de decisión y compuertas de fase.

---

## Fase 4: Determinación del Modo de Ejecución

El workflow notifica y configura el modo de operación:

1. **Plataformas Multiagente Nativas**:
   - **Google Antigravity**: El `@coordinator` opera mediante herramientas nativas (`invoke_subagent` con `Workspace: "inherit"`, `update_steps` para el checklist visual, `update_focus` y `ask_question`). Cada subagente se materializa con su propia tarjeta de ejecución en el panel lateral y devuelve un reporte sintético.
   - **OpenCode**: El `@coordinator` despacha subagentes declarados en `.opencode/agents/` (`@planner`, `@implementer`, `@reviewer`). OpenCode genera automáticamente un árbol de sub-sesiones en su TUI/GUI, mostrando en streaming el progreso y las llamadas a herramientas.
2. **Plataformas Monohilo (Claude Code, Cursor, Codex, Windsurf, Aider)**:
   - El agente único opera en su hilo continuo asumiendo internamente el rol de Coordinador.
   - Anuncia cada transición de fase de forma obligatoria y explícita en el chat:
     - `[CAMBIO DE ROL: @planner]`
     - `[CAMBIO DE ROL: @implementer]`
     - `[CAMBIO DE ROL: @reviewer]`
   - El estado reside en disco (`specs/NNN/`), permitiendo compactar contexto (`/compact`) en cualquier momento sin perder trazabilidad.

---

## Fase 5: Confirmación Final

Emite una respuesta de confirmación en una sola línea indicando:
> *"✅ Entorno configurado con éxito para [Nombre del Agente] en modo [Multiagente Nativo / Monohilo Coordinado]. Todo el arnés reside en `.agent/` y la raíz está limpia. Puedes comenzar ejecutando `/sdd-spec` o consultar el estado con `/sdd-status`."*

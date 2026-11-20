# agent-core

> **Software Factory Portátil y Arnés de Ingeniería para Agentes de IA**  
> *Desarrollo Guiado por Especificaciones (SDD) · Desarrollo Guiado por Pruebas (TDD) · Flujo de Ramas y PRs Protegidos · DevSecOps Estricto · Subagentes Especializados*

`agent-core` es un arnés de ingeniería de software agnóstico al stack tecnológico y portable entre plataformas de IA (OpenCode, Google Antigravity, Claude Code, OpenAI Codex, Cursor, Windsurf, Aider y más). Transforma cualquier repositorio en una fábrica de software disciplinada y reproducible donde la especificación manda, los tests prueban y la integración ocurre exclusivamente mediante compuertas de calidad automatizadas.

---

## 1. Arquitectura y Principios de Diseño

```text
agent-core/
├── agents/                   # 4 Subagentes especializados (formato híbrido OpenCode + Markdown universal)
│   ├── coordinator.md        # Orquestador del ciclo SDD (shell: deny, edit: deny)
│   ├── planner.md            # Planificador y arquitecto (specs/** allow, MEMORY.md allow, shell: deny)
│   ├── implementer.md        # Implementador TDD con acceso a shell y guardarraíles de gobernanza
│   └── reviewer.md           # Revisor de QA en doble modo (clarificación y validación, edit: deny)
│
├── mcp/                      # Catálogo y configuración de MCPs recomendados
│   └── recommended-mcps.md   # GitHub, Context7, Chrome DevTools y degradación en 3 niveles
│
├── templates/                # Plantillas universales y adaptadores por plataforma
│   ├── AGENTS.md.template    # Resumen conciso de stack y reglas (~30-40 líneas)
│   ├── AGENTS-monorepo-root.template
│   ├── AGENTS-monorepo-module.template
│   ├── MEMORY.md.template    # Memoria viva de decisiones y aprendizajes (~50 líneas)
│   ├── MEMORY-module.template
│   ├── constitution.md.template # Principios supremos innegociables (máx 6 principios)
│   ├── adapters/             # Adaptadores nativos por agente (CLAUDE.md, CODEX.md, opencode.json, etc.)
│   └── specs/                # Plantillas para spec.md (EARS), plan.md y tasks.md (checkpoints)
│
├── workflows/                # 12 Workflows ejecutables / Comandos slash (/sdd-*, /init, /adopt)
│   ├── init.md               # Bootstrap del repositorio (Día 0 o actualización no destructiva)
│   ├── adopt.md              # Auto-adopción y generación de conectores nativos por agente
│   ├── sdd-constitution.md   # Creación y auditoría de docs/constitution.md
│   ├── sdd-spec.md           # Captura de requerimientos, árbol limpio y rama temática
│   ├── sdd-clarify.md        # QA funcional con @reviewer [MODO: CLARIFICACION_SPEC]
│   ├── sdd-plan.md           # Arquitectura técnica, contratos y límite de 10 tareas
│   ├── sdd-tasks.md          # Desglose en tareas atómicas (~20-30 min) y checkpoints
│   ├── sdd-implement.md      # Implementación atómica bajo ciclo TDD Red-Green-Refactor
│   ├── sdd-validate.md       # Validación integral con @reviewer [MODO: VALIDACION_IMPLEMENTACION]
│   ├── sdd-change.md         # Modificación de alcance en vuelo aditiva y controlada
│   ├── sdd-status.md         # Consulta de máquina de estados y archivado de canceladas
│   └── review-and-pr.md      # Sincronización, rebase, PR en 3 niveles y eliminación autorizada
│
└── skills/                   # Batería de 27 skills agrupadas en 6 categorías
    ├── router/               # using-agent-skills (Enrutador maestro)
    ├── methodology/          # sdd, test-driven-development, source-driven-development, idea-refine,
    │                         # interview-me, planning-and-task-breakdown, constraint-driven-development,
    │                         # doubt-driven-development
    ├── architecture/         # api-and-interface-design, incremental-implementation, context-engineering,
    │                         # documentation-and-adrs, observability-and-instrumentation, performance-optimization
    ├── quality/              # code-review-and-quality, code-simplification, debugging-and-error-recovery,
    │                         # deprecation-and-migration
    ├── delivery/             # git-pr-workflow, devsecops, docker-hardening, ci-cd-and-automation,
    │                         # shipping-and-launch
    └── frontend/             # frontend-ui-engineering, browser-testing-with-devtools, ui-ux-pro-max
```

---

## 2. Instalación y Adopción Local-First (Offline)

`agent-core` opera bajo la política **Local-First (Offline)**: prioriza el trabajo sin conexión utilizando una copia local del núcleo antes de requerir acceso a internet.

### Opción A: Inicialización desde Copia Local (Recomendada)
Si tienes el repositorio clonado localmente (ej. en `~/.agent-core` o `C:/Users/.../agent-core`):

```bash
# Dentro de tu repositorio destino:
npx skills add <ruta-local-a-agent-core>/skills/router/using-agent-skills
# O invocar el workflow de inicialización apuntando a la fuente local:
/init --source C:/Users/edier/Documents/EDIER/agent-core
```

### Opción B: Inicialización desde GitHub (Nuevo Equipo)
```bash
git clone https://github.com/<tu-usuario>/agent-core.git ~/.agent-core
```

---

## 3. El Prompt Universal de Auto-Adopción (Día 0 vs Día N)

Copia y pega el siguiente bloque en el chat de **cualquier agente de IA** (Claude Code, OpenCode, Codex, Cursor, Windsurf o Antigravity) para activar el sistema de inmediato:

```markdown
Eres el AGENTE PRINCIPAL operando en este repositorio.
Tu objetivo es activar el flujo de ingeniería de software SDD + TDD de agent-core.

Sigue ESTRICTAMENTE este protocolo en orden:

1. EVALÚA EL ESTADO DEL REPOSITORIO (Día 0 vs Día N):
   - Revisa si existen en la raíz o en docs/ los siguientes archivos: `AGENTS.md`, `MEMORY.md`, `docs/constitution.md`.
   - CASO A: REPOSITORIO NUEVO (DÍA 0 - No existen `AGENTS.md` ni `constitution.md`):
     - Tu primera tarea es ejecutar el proceso de inicialización (Bootstrap).
     - Consulta la plantilla y guía en `.agent/workflows/init.md` (o `workflows/init.md` si estás dentro de agent-core).
     - Realiza la entrevista técnica al usuario (lenguaje, framework, base de datos, comandos de test y dev).
     - Genera los archivos fundacionales del repositorio:
       * `AGENTS.md` (resumen conciso de stack, comandos y límites innegociables de ~30-40 líneas).
       * `docs/constitution.md` (máximo 6 principios rectores de arquitectura en ~10-15 líneas).
       * `MEMORY.md` (estado inicial del proyecto en ~25-50 líneas).
       * `.env.example` (plantilla con variables requeridas documentadas sin credenciales reales).
     - Una vez creados y validados con el usuario, procede al Paso 2.
   - CASO B: REPOSITORIO EXISTENTE COMPLETO (DÍA N - Ya existen `AGENTS.md`, `MEMORY.md` y `docs/constitution.md`):
     - Lee `AGENTS.md` para conocer el stack, los comandos exactos y los límites innegociables.
     - Lee `docs/constitution.md` para asimilar los principios innegociables de arquitectura.
     - Lee `MEMORY.md` para conocer el estado actual y decisiones tomadas.
     - NO modifiques ni muevas la estructura de código preexistente del proyecto.
     - Procede al Paso 2.
   - CASO C: REPOSITORIO EN ESTADO PARCIAL (Existe algún archivo base pero faltan otros):
     - Identifica cuáles archivos faltan (`AGENTS.md`, `docs/constitution.md` o `MEMORY.md`).
     - Genera exclusivamente los archivos faltantes a partir de las plantillas en `.agent/templates/` respetando los archivos ya existentes sin sobrescribirlos.
     - Procede al Paso 2.

2. GENERA TU ADAPTADOR NATIVO (Sin mover la fuente canónica de la verdad en .agent/):
   - Identifica qué tipo de agente eres (Claude Code, Cursor, OpenCode, Codex, Antigravity, etc.).
   - Si no tienes configurado tu archivo nativo de instrucciones, genéralo a partir de las plantillas en `.agent/templates/adapters/` (o `.agent/workflows/adopt.md`):
     * Si eres Claude Code: crea/actualiza `CLAUDE.md` y genera los comandos en `.claude/commands/`.
     * Si eres Codex: crea/actualiza `CODEX.md`.
     * Si eres OpenCode: copia físicamente agentes a `.opencode/agents/` y comandos a `.opencode/commands/`, y configura `opencode.json`.
     * Si eres Cursor / otro: crea la regla nativa correspondiente (ej. `.cursor/rules/agent-core.mdc`).
   - Asegura que tu adaptador apunte a las skills en `.agent/skills/` y comandos en `.agent/workflows/`.

3. MODO DE ORQUESTACIÓN Y EJECUCIÓN:
   - Revisa tus capacidades de ejecución:
     * SI TU ENTORNO ADMITE SUBAGENTES EN HILOS/SESIONES AISLADAS (ej. OpenCode, Antigravity):
       Opera como Coordinador despachando tareas a los subagentes especializados (`@planner`, `@implementer`, `@reviewer`) ubicados en `.agent/agents/`.
     * SI TU ENTORNO ES MONOHILO / SIN SOPORTE DE HILOS SEPARADOS (ej. Claude Code, Cursor, Codex, Windsurf):
       Asume tú mismo el rol de Coordinador en este hilo de conversación y ejecuta el flujo secuencialmente adoptando cada rol según la fase activa, anunciándolo explícitamente en el chat:
       - `[CAMBIO DE ROL: @planner]` para redactar y refinar specs, planes y tareas.
       - `[CAMBIO DE ROL: @implementer]` para escribir tests en rojo, código y ponerlos en verde.
       - `[CAMBIO DE ROL: @reviewer]` para QA funcional de specs y validación RF por RF con tests.

4. CONFIRMACIÓN FINAL:
   - Confirma al usuario en una sola línea que el entorno está listo, indicando el agente detectado, el modo de ejecución (multiagente nativo o monohilo coordinado) y ofreciendo iniciar con `/sdd-spec` o el siguiente requerimiento.
```

---

## 4. Mapeo de Adaptadores para los 4 Agentes Principales

| Agente / Plataforma | Archivo Generado | Conexión con `agent-core` | Modo de Despacho |
| :--- | :--- | :--- | :--- |
| **OpenCode** | `.opencode/agents/`<br>`.opencode/commands/`<br>`opencode.json` | Copia física de archivos (sin symlinks) desde `.agent/agents/` y `.agent/workflows/`. Esquema MCP nativo. | Multiagente nativo en sesiones aisladas (`@planner`, `@implementer`, `@reviewer`). |
| **Claude Code** | `CLAUDE.md`<br>`.claude/commands/sdd-*.md` | Protocolo de cambio de rol explícito en `CLAUDE.md` y comandos slash nativos autocompletables en `.claude/commands/`. | Monohilo coordinado con anuncios `[CAMBIO DE ROL: @agente]`. |
| **OpenAI Codex** | `CODEX.md` | Directivas de límites, comandos exactos de verificación y referencia canónica a `.agent/`. | Monohilo coordinado. |
| **Google Antigravity** | `AGENTS.md`<br>`.agent/skills/` | Configuración nativa de subagentes en `.agent/agents/` y registro de skills en `.agent/skills/`. | Multiagente nativo mediante `invoke_subagent`. |
| **Cursor / Windsurf / Aider** | `.cursor/rules/agent-core.mdc`<br>`.windsurfrules` | Reglas globales de contexto que vinculan el ciclo SDD + TDD y convenciones del proyecto. | Monohilo coordinado. |

---

## 5. El Ciclo de Desarrollo Spec-Driven (SDD + TDD)

```text
/sdd-spec ──► /sdd-clarify ──► /sdd-plan ──► /sdd-tasks ──► /sdd-implement ──► /sdd-validate ──► /review-and-pr
 (Borrador)    (QA Funcional)  (Arquitectura) (10 Tareas)     (TDD Rojo-Verde)  (QA Integral)     (Sync & PR)
```

1. **`/sdd-spec [nombre]`**: Comprueba árbol limpio de Git, crea rama temática `<tipo>/<NNN-slug>` en modo `branch_pr`, captura requerimientos en entrevista y genera `spec.md` en sintaxis formal EARS en español.
2. **`/sdd-clarify`**: Convoca a `@reviewer` en `[MODO: CLARIFICACION_SPEC]` para auditar ambigüedades, contradicciones y casos límite (máx 2 rondas).
3. **`/sdd-plan`**: `@planner` diseña la arquitectura técnica, contratos de interfaz, persistencia y estrategia de pruebas en `plan.md` (límite estricto de máx 10 tareas).
4. **`/sdd-tasks`**: `@planner` desglosa las tareas atómicas secuenciales (~20-30 min) en `tasks.md` con sintaxis rígida de checkpoints indentados y condiciones "Hecho cuando:".
5. **`/sdd-implement [Tn]`**: `@implementer` infiere o toma la tarea, verifica la línea base (`TEST_COMMAND`), ejecuta el ciclo TDD (Rojo $\rightarrow$ Verde $\rightarrow$ Refactor), aplica staging quirúrgico y commit atómico. Cero push intermedio.
6. **`/sdd-validate`**: `@reviewer` en `[MODO: VALIDACION_IMPLEMENTACION]` audita la feature completa RF por RF contra tests globales, `git diff`, DevTools (con chequeo pasivo de servidor) y compuertas de seguridad.
7. **`/sdd-change`**: Protocolo aditivo para modificar requerimientos en progreso sin desmarcar tareas previas completadas.
8. **`/sdd-status`**: Consulta la máquina de estados SDD, sugiere el siguiente comando y gestiona el archivo seguro de specs canceladas.
9. **`/review-and-pr`**: Sincroniza con rebase la rama base, resuelve colisiones numéricas correlativas con renombrado de rama (`git branch -m`), abre el PR en 3 niveles de degradación (GitHub MCP $\rightarrow$ `gh` CLI $\rightarrow$ Web URL), sincroniza release a `main` protegida y limpia ramas tras autorización explícita.

---

## 6. Reglas Innegociables de Gobernanza y Seguridad

- **La Spec Manda:** Prohibido escribir código sin spec, plan y tareas aprobadas por el usuario.
- **TDD Innegociable:** Escribe siempre el test que falle en rojo antes de implementar el código de producción.
- **Cero Credenciales en Código (DevSecOps):** Queda terminantemente prohibido quemar contraseñas, tokens o cadenas de conexión como fallback (`|| 'postgresql://...'`). Todo script exige variables de entorno desde `.env` y falla de inmediato si no están presentes.
- **Aislamiento de Producción:** Queda terminantemente prohibido ejecutar pruebas o semilleros alteradores contra bases de datos remotas de producción.
- **Gobernanza Git:** La rama base nunca recibe merges directos locales; toda integración se realiza mediante Pull Request validado por CI.
- **Salvaguarda de Ramas:** Se debe pedir confirmación explícita al usuario **antes de abrir PR, antes de fusionar PR y antes de eliminar cualquier rama local o remota**.
- **Normalización Universal de Rutas:** Toda referencia a archivos en documentación, scripts y llamadas a herramientas utiliza exclusivamente **barras diagonales directas (`/`)**.
- **Convención Lingüística:** Toda la documentación, prompts, workflows y mensajes de error operan exclusivamente en **español**.

---

## Licencia

MIT © 2026 agent-core. Diseñado para potenciar el desarrollo de software confiable asistido por Inteligencia Artificial.

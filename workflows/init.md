---
description: Inicializa agent-core en un repositorio (Día 0 o existente), configurando AGENTS.md, docs/constitution.md, MEMORY.md y .env.example mediante entrevista o auditoría local-first.
---

# Workflow: Inicialización del Repositorio (Bootstrap)

Este workflow configura los cimientos de gobernanza y arnés técnico de `agent-core` en el repositorio destino. Es ejecutado directamente por el **agente principal / root** (no por subagentes aislados).

Opera bajo la política **Local-First (Offline)**: prioriza fuentes locales antes de acceder a la red.

---

## Modos de Ejecución

1. **Inicialización Estándar (`/init` o `/init --source <ruta>`)**:
   - Para proyectos nuevos (Greenfield) o proyectos existentes sin arnés de agentes.
2. **Actualización No Destructiva (`/init --update` o `/init --update --source <ruta>`)**:
   - Para repositorios que ya tienen `.agent/` y desean sincronizar skills, subagentes o workflows actualizados desde `agent-core` sin perder configuraciones locales.

---

## PARTE 1: Inicialización Estándar

### Fase 1: Comprobación de Control de Versiones (Git) y Fuente Local

1. **Auditoría de Git**:
   - Comprueba si el directorio actual cuenta con control de versiones (`.git`).
   - Si `.git` **NO existe**:
     - Solicita confirmación explícita al usuario para inicializar Git:
       ```bash
       git init && git checkout -b main
       ```
     - Realiza un commit inicial vacío o con los archivos base para habilitar compuertas de Git y baseline de TDD.
2. **Detección de Fuente Local-First (`--source`)**:
   - Si se proporcionó `--source <ruta>`, verifica la existencia de esa ruta en disco.
   - Si no se proporcionó argumento, comprueba si existe una copia local en `~/.agent-core` o consulta al usuario:
     > *"¿Tienes el repositorio `agent-core` clonado en una ruta local de este equipo (ej. `C:/Users/.../agent-core` o `~/agent-core`)? Si es así, indícame la ruta para trabajar 100% offline. Si prefieres clonarlo de GitHub o usar plantillas locales, confírmalo."*
   - Si se define una ruta local válida, copia plantillas, agentes y skills directamente desde esa ruta local.
   - Si se requiere conexión remota en un equipo nuevo, clona el núcleo con:
     ```bash
     git clone <repo-url> ~/.agent-core
     ```

---

### Fase 2: Identificación del Contexto del Proyecto

1. Comprueba si el directorio actual contiene código fuente, dependencias (`package.json`, `requirements.txt`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `pom.xml`, etc.) o commits previos.
2. Determina el escenario:
   - **Escenario A: Proyecto Existente (Día N)**: Auditar el código actual, linters, runners de pruebas y convenciones.
   - **Escenario B: Proyecto Desde Cero (Día 0 / Greenfield)**: Entrevistar al usuario para definir stack, arquitectura y comandos.

---

### Fase 3: Auditoría o Entrevista Técnica

#### Si es Proyecto Existente:
1. Inspecciona los archivos clave del repositorio (árbol de directorios, manifiestos de dependencias, scripts de test, docker-compose, linters).
2. Formula un máximo de 3 a 5 preguntas concretas sobre lo que **no se puede deducir del código**:
   - ¿Cuáles son las reglas de dominio o trampas críticas que un agente suele romper aquí?
   - ¿Qué acciones o módulos están estrictamente prohibidos tocar o cambiar (guardrails)?
   - ¿Qué comando exacto de test consideras indispensable y cuál es tu servidor de desarrollo local (`DEV_SERVER_URL`)?
   - ¿Qué flujo de Git utilizas (`branch_pr` con `develop`/`main`, o `trunk` directo en `main`)?

#### Si es Proyecto Desde Cero (Greenfield):
1. Entrevista al usuario brevemente:
   - Objetivo del proyecto y arquitectura inicial (monolito modular, API REST, CLI, frontend SPA).
   - Stack tecnológico (lenguaje, framework, base de datos).
   - Runner de pruebas a configurar (ej. Vitest, Jest, Pytest, Cargo test).
   - Modo de Git preferido (`branch_pr` por defecto, o `trunk`).

---

### Fase 4: Auditoría Obligatoria de `TEST_COMMAND`

El comando de pruebas registrado en `AGENTS.md` debe ser **estrictamente de ejecución única, no interactivo y con terminación determinista**.

1. Revisa que el comando sugerido no active modo interactivo ni observador continuo (*watch mode*).
2. Ajusta automáticamente según el gestor:
   - Jest: `npm test -- --watchAll=false`
   - Vitest: `npx vitest run`
   - Pytest: `pytest`
   - Cargo: `cargo test`
   - Go: `go test ./...`
3. Prohibido registrar comandos como `npm test` si por defecto inician Vitest o Jest en modo interactivo/watch.

---

### Fase 5: Propuesta y Validación de Archivos Fundacionales

Antes de escribir en disco, presenta al usuario el borrador completo de los siguientes archivos para su aprobación:

#### 1. `AGENTS.md` (Raíz del proyecto, 30-40 líneas)
Utiliza como base `templates/AGENTS.md.template`. Debe contener:
- **Stack y estructura**: componentes principales y directorios canónicos.
- **Comandos**: comandos exactos de dev, build, lint, typecheck y `TEST_COMMAND` de terminación única.
- **Variables de arnés**:
  - `DEV_SERVER_URL`: URL del servidor local de desarrollo (o vacío si es backend/CLI).
  - `GIT_WORKFLOW_MODE`: `branch_pr` o `trunk`.
  - `PR_TARGET_BRANCH`: `develop` o `main`.
  - `AGENT_CAN_MERGE`: `false` (por defecto) o `true`.
- **Convenciones**: idioma, estilos y reglas de importación.
- **Reglas de dominio / trampas conocidas**: advertencias críticas de negocio.
- **Forma de trabajar**: referencia a SDD + TDD y skills en `.agent/`.
- **Límites**: ✅ Siempre / ⚠️ Pregunta antes / 🚫 Nunca.
- **Verificación**: suite y linters de cierre.

#### 2. `docs/constitution.md` (Máximo 6 principios innegociables, ~10-15 líneas)
Utiliza como base `templates/constitution.md.template`. Define los invariantes arquitecturales del proyecto (ej. cero credenciales en código, cobertura de pruebas previa, contratos estrictos, inmutabilidad de esquemas sin migración).

#### 3. `MEMORY.md` (Raíz del proyecto, ~25-50 líneas)
Utiliza como base `templates/MEMORY.md.template`. Inicializa el estado actual, decisiones arquitectónicas iniciales y próximos pasos.

#### 4. `.env.example` (Plantilla documentada sin secretos)
Si el proyecto utiliza variables de entorno, genera `.env.example` documentando cada clave requerida con valores dummy/ejemplo.
- **Regla estricta DevSecOps**: Cero contraseñas reales, tokens o claves de API.
- Asegura que `.env` esté listado en `.gitignore`.

---

### Fase 6: Generación y Despliegue de Componentes

Tras la confirmación explícita del usuario:
1. Crea los archivos fundacionales validados (`AGENTS.md`, `docs/constitution.md`, `MEMORY.md`, `.env.example`).
2. Despliega la estructura canónica en `.agent/`:
   - Copia los 4 subagentes en `.agent/agents/` (`coordinator.md`, `planner.md`, `implementer.md`, `reviewer.md`).
   - Copia los workflows en `.agent/workflows/`.
   - Copia las skills en `.agent/skills/`.
   - Copia el catálogo MCP en `.agent/mcp/recommended-mcps.md`.
   - Crea el directorio transitorio `.agent/scratch/` y añade la regla correspondiente a `.gitignore`.
3. Normaliza todas las rutas a barras directas (`/`).
4. Invoca automáticamente el workflow `/adopt` para configurar los conectores del agente activo en el entorno.

---

## PARTE 2: Actualización No Destructiva (`/init --update`)

Si se ejecuta con el modificador `--update`:

1. **Inspección de Diferencias**:
   - Compara los subagentes en `.agent/agents/`, workflows en `.agent/workflows/` y skills en `.agent/skills/` contra las plantillas fuente en `agent-core`.
2. **Detección de Personalizaciones y Respaldo Preventivo**:
   - Si un archivo en `.agent/` tiene modificaciones locales respecto a la versión upstream:
     - Crea un respaldo preventivo en `.agent/backup-<timestamp>/<archivo>`.
     - Notifica al usuario mostrando el diff y solicita confirmación explícita antes de sobrescribirlo.
3. **Blindaje Total de Archivos de Gobernanza**:
   - **PROHIBIDO SOBRESCRIBIR**: `AGENTS.md`, `MEMORY.md`, `docs/constitution.md` y archivos de entorno (`.env*`). Estos archivos son sagrados y pertenecen al repositorio destino.
4. **Sincronización de Adaptadores OpenCode (`.opencode/`)**:
   - Si existe el directorio `.opencode/`, sincroniza las actualizaciones desde `.agent/agents/` y `.agent/workflows/` hacia `.opencode/agents/` y `.opencode/commands/` mediante copia física de archivos, aplicando el mismo protocolo de respaldo preventivo.
5. Emite un informe final con los archivos actualizados y los respaldos generados.

---
description: Inicializa agent-core en un repositorio (Día 0 o existente), desplegando o reorganizando quirúrgicamente los archivos en .agent/ y configurando AGENTS.md, docs/constitution.md, MEMORY.md y .env.example.
---

# Workflow: Inicialización del Repositorio (Bootstrap y Reorganización Quirúrgica)

Este workflow configura los cimientos de gobernanza y arnés técnico de `agent-core` en el repositorio destino. Es ejecutado directamente por el **agente principal / root** (no por subagentes aislados).

Opera bajo la política **Local-First (Offline)** y el principio de **Cero Basura en la Raíz**: todo el arnés de agentes reside estrictamente dentro de `.agent/`, dejando la raíz del proyecto 100% limpia para el código de la aplicación del usuario.

---

## Modos de Ejecución

1. **Inicialización Estándar (`/init` o `/init --source <ruta>`)**:
   - Para proyectos nuevos (Greenfield) o proyectos existentes sin arnés de agentes.
   - Detecta si `agent-core` fue clonado directamente en la raíz y reorganiza los archivos quirúrgicamente.
2. **Actualización No Destructiva (`/init --update` o `/init --update --source <ruta>`)**:
   - Para repositorios que ya tienen `.agent/` y desean sincronizar skills, subagentes o workflows actualizados desde `agent-core` sin perder configuraciones locales.

---

## PARTE 1: Inicialización y Reorganización Quirúrgica

### Fase 1: Detección de Despliegue y Reorganización Quirúrgica

Antes de cualquier entrevista o generación de archivos, el agente analiza la estructura del directorio actual:

#### Escenario 1: Clonado Directo de `agent-core` (Árbol Expuesto en la Raíz)
Si el repositorio actual contiene carpetas como `agents/`, `workflows/`, `skills/`, `templates/` o `mcp/` expuestas directamente en la raíz:
1. **Reorganización Automática hacia `.agent/`**:
   - Crea el contenedor canónico `.agent/`.
   - Mueve `agents/` $\rightarrow$ `.agent/agents/`.
   - Mueve `workflows/` $\rightarrow$ `.agent/workflows/`.
   - Mueve `skills/` $\rightarrow$ `.agent/skills/`.
   - Mueve `mcp/` $\rightarrow$ `.agent/mcp/`.
   - Mueve `templates/` $\rightarrow$ `.agent/templates/`.
2. **Tratamiento de `README.md` de `agent-core`**:
   - Si el `README.md` en la raíz corresponde a la documentación de `agent-core`, muévelo a `.agent/README.md`.
   - Esto preserva el manual del arnés completo dentro de `.agent/` para consulta offline y libera la raíz para el `README.md` propio de la aplicación del usuario.
3. **Tratamiento de `plan.md` de `agent-core`**:
   - Si existe `plan.md` en la raíz (plan interno de construcción de `agent-core`), muévelo a `.agent/docs/history/agent-core-plan.md`.
   - La raíz no debe contener un `plan.md` huérfano, ya que las características del proyecto se gestionarán exclusivamente dentro de `specs/NNN-nombre/plan.md`.
4. **Resultado**: La raíz queda 100% libre de residuos del núcleo.

#### Escenario 2: Instalación desde Fuente Externa (`/init --source <ruta>` o `~/.agent-core`)
Si se ejecuta en un proyecto destino independiente:
1. Copia directamente desde la fuente hacia `.agent/`:
   - `.agent/agents/`
   - `.agent/workflows/`
   - `.agent/skills/`
   - `.agent/mcp/`
   - `.agent/templates/`
   - `.agent/README.md` (como documentación del arnés)
2. **PROHIBICIÓN ESTRICTA**: Prohibido copiar `plan.md` o el `README.md` de `agent-core` a la raíz del proyecto destino.

---

### Fase 2: Comprobación de Git

1. **Auditoría de Git**:
   - Comprueba si el directorio actual cuenta con control de versiones (`.git`).
   - Si `.git` **NO existe**:
     - Solicita confirmación explícita al usuario para inicializar Git:
       ```bash
       git init && git checkout -b main
       ```
     - Realiza un commit inicial vacío o con los archivos base para habilitar compuertas de Git y baseline de TDD.

---

### Fase 3: Identificación del Contexto del Proyecto

1. Comprueba si el directorio actual contiene código fuente, dependencias (`package.json`, `requirements.txt`, `pyproject.toml`, `Cargo.toml`, `go.mod`, etc.) o commits previos.
2. Determina el escenario:
   - **Escenario A: Proyecto Existente (Día N)**: Auditar el código actual, linters, runners de pruebas y convenciones.
   - **Escenario B: Proyecto Desde Cero (Día 0 / Greenfield)**: Entrevistar al usuario para definir stack, arquitectura y comandos.

---

### Fase 4: Auditoría o Entrevista Técnica (Tarjetas de Selección con Opción Libre)

**Estándar Obligatorio de Formulación**: Todas las preguntas de esta fase deben presentarse obligatoriamente bajo el formato de **tarjetas de selección con opción libre**:
- Si la plataforma dispone de herramienta interactiva nativa (ej. `ask_question`), invocarla enviando las opciones sin numerar con `(Recommended)` en la sugerencia preferente según el contexto analizado (la UI despliega tarjetas de clic y la opción automática para escribir otra respuesta).
- Si opera en texto plano / Markdown, estructurar cada pregunta como tarjetas visuales de selección con opciones `[A] (Recomendada)`, `[B]` y la opción explícita `[O] Escribir otra respuesta`.
- Queda prohibido formular preguntas abiertas en texto plano sin opciones preconfiguradas.

#### Si es Proyecto Existente:
1. Inspecciona los archivos clave del repositorio (árbol de directorios, manifiestos de dependencias, scripts de test, docker-compose, linters).
2. Formula un máximo de 3 a 5 preguntas concretas sobre lo que **no se puede deducir del código** estructuradas en tarjetas:
   - ¿Qué flujo de Git prefieres utilizar? (`(Recommended) branch_pr con ramas dedicadas y PRs`, `trunk con desarrollo directo en main`, u otra respuesta).
   - ¿Cuáles son las reglas de dominio o trampas críticas que ningún agente debe romper?
   - ¿Qué acciones o módulos están estrictamente prohibidos tocar o cambiar (guardrails)?
   - ¿Qué comando exacto de test consideras indispensable y cuál es tu servidor de desarrollo local (`DEV_SERVER_URL`)?

#### Si es Proyecto Desde Cero (Greenfield):
1. Entrevista al usuario mediante tarjetas de selección:
   - Tipo de arquitectura inicial (`(Recommended) Monolito modular desacoplado`, `API REST backend pura`, `Aplicación Frontend SPA`, `CLI / Herramienta de terminal`).
   - Stack tecnológico principal (ofreciendo las opciones más robustas del ecosistema con recomendación fundamentada).
   - Runner de pruebas determinista a configurar (ej. Vitest, Pytest, Jest, Cargo test).
   - Modo de Git preferido (`(Recommended) branch_pr con PR`, `trunk directo`).

---

### Fase 5: Auditoría Obligatoria de `TEST_COMMAND`

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

### Fase 6: Propuesta y Validación de Archivos Fundacionales

Antes de escribir en disco, presenta al usuario el borrador completo de los siguientes archivos para su aprobación:

#### 1. `AGENTS.md` (Raíz del proyecto, 30-40 líneas)
Utiliza como base `.agent/templates/AGENTS.md.template`. Debe contener:
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
Utiliza como base `.agent/templates/constitution.md.template`. Define los invariantes arquitecturales del proyecto (ej. cero credenciales en código, cobertura de pruebas previa, contratos estrictos, inmutabilidad de esquemas sin migración).

#### 3. `MEMORY.md` (Raíz del proyecto, ~25-50 líneas)
Utiliza como base `.agent/templates/MEMORY.md.template`. Inicializa el estado actual, decisiones arquitectónicas iniciales y próximos pasos.

#### 4. `.env.example` (Plantilla documentada sin secretos)
Si el proyecto utiliza variables de entorno, genera `.env.example` documentando cada clave requerida con valores dummy/ejemplo.
- **Regla estricta DevSecOps**: Cero contraseñas reales, tokens o claves de API.
- Asegura que `.env` esté listado en `.gitignore`.

#### 5. `README.md` del Proyecto
Si el proyecto es nuevo o no tenía `README.md`, genera un archivo limpio para la aplicación del usuario:
```markdown
# [Nombre del Proyecto]

> [Descripción breve de la aplicación]

## Requisitos y Comandos
Consulta `AGENTS.md` para conocer los comandos de desarrollo, pruebas y ejecución.
```

---

### Fase 7: Confirmación y Activación del Adaptador

Tras la confirmación explícita del usuario:
1. Escribe los archivos fundacionales validados (`AGENTS.md`, `docs/constitution.md`, `MEMORY.md`, `.env.example`, `README.md`).
2. Crea el directorio transitorio `.agent/scratch/` y añade la regla correspondiente a `.gitignore`.
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
   - **PROHIBIDO SOBRESCRIBIR**: `AGENTS.md`, `MEMORY.md`, `docs/constitution.md`, `README.md` (de la raíz) y archivos de entorno (`.env*`). Estos archivos son sagrados y pertenecen al repositorio destino.
4. **Sincronización de Adaptadores OpenCode (`.opencode/`)**:
   - Si existe el directorio `.opencode/`, sincroniza las actualizaciones desde `.agent/agents/` y `.agent/workflows/` hacia `.opencode/agents/` y `.opencode/commands/` mediante copia física de archivos, aplicando el mismo protocolo de respaldo preventivo.
5. Emite un informe final con los archivos actualizados y los respaldos generados.

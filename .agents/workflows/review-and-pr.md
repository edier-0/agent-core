---
description: Sincroniza la rama temática, resuelve colisiones numéricas, abre el Pull Request en 3 niveles, gestiona release a main y limpia ramas tras autorización.
---

# Workflow: Revisión Final, Pull Request y Sincronización (`/review-and-pr`)

Este workflow gestiona la fase final de integración en Git: sincronización con la rama base, resolución asistida de conflictos y colisiones de numeración, publicación de la rama, apertura del Pull Request en 3 niveles de degradación elegante, sincronización a producción y limpieza post-merge con confirmación explícita.

**Gobernanza de Consola y Roles**:
Toda invocación de comandos de consola de Git (`git fetch`, `git rebase`, `git branch`, `git push`, `git checkout`, `git merge`, `git pull`, etc.) y CLI (`gh`) se delega y ejecuta **exclusivamente a través de `@implementer`** (`invoke_subagent` con `Workspace: "inherit"`). El `@coordinator` gestiona las confirmaciones con el usuario y la apertura de PR vía MCP GitHub (si está disponible), sin ejecutar consola en el hilo principal. `@planner` se limita a actualizar especificaciones y metadatos.

---

## Pre-Condiciones

1. La spec debe estar en `Estado: validada` tras la aprobación de `@reviewer` en `/sdd-validate`.
2. Todos los tests de la suite (`TEST_COMMAND`) deben estar en verde.
3. El árbol de trabajo debe estar limpio sin cambios residuales.

---

## Fase 1: Sincronización Pre-PR y Resolución de Conflictos

1. **Sincronización con la Rama Base**:
   - Lee `PR_TARGET_BRANCH` de `AGENTS.md` (por defecto `develop` o `main`).
   - El `@coordinator` despacha a `@implementer` para ejecutar la sincronización y el rebase en local:
     - Si existe repositorio remoto configurado (`origin`):
       ```bash
       git fetch origin
       git rebase origin/<base_branch>
       ```
     - Si opera en modo local-first / offline (sin remoto):
       ```bash
       git rebase <base_branch>
       ```
2. **Tratamiento de Conflictos Git**:
   - **Conflictos Mecánicos Triviales (Resolución Autónoma por `@implementer`)**:
     Se consideran triviales **únicamente** aquellos conflictos consistentes en:
     1. Ordenamiento alfabético de imports o dependencias sin conflicto de versiones.
     2. Saltos de línea o formateo de espacios en blanco / indentación cosmética (whitespace/Prettier).
     3. Entradas disjuntas e independientes en `.gitignore`.
     En estos casos exclusivos, `@implementer` resuelve el conflicto, continúa el rebase (`git rebase --continue`) y ejecuta `TEST_COMMAND` para asegurar que todo permanezca en verde.
   - **Conflictos de Lógica, Negocio o Arquitectura**:
     Cualquier conflicto que altere firmas de funciones, contratos de tipos, lógica de control, indentación sintáctica (Python/YAML) o tests **NO es trivial**. El subagente **SE DETIENE DE INMEDIATO**, reporta las líneas en conflicto al `@coordinator` y este solicita al usuario resolver el conflicto manualmente.

---

## Fase 2: Detección y Resolución de Colisiones Numéricas

Si durante la sincronización se detecta que otro PR concurrente integró una carpeta con el mismo identificador numérico en la rama base (ej. ya existe `specs/003-notificaciones/` y la rama actual también es `003-perfil`):
1. `@planner` calcula el siguiente correlativo libre disponible `max(NNN) + 1` (ej. `004`).
2. `@planner` renombra la carpeta local: `specs/003-perfil/` -> `specs/004-perfil/` y actualiza el identificador en `spec.md`, `plan.md` y `tasks.md`.
3. En modo `branch_pr`, `@coordinator` despacha a `@implementer` para renombrar la rama local de Git y generar el commit de ajuste:
   ```bash
   git branch -m feature/003-perfil feature/004-perfil
   git commit -am "chore(004): reasignar identificador correlativo por colisión de numeración"
   ```

---

## Fase 3: Publicación (Push Único y Controlado) de la Rama Temática

**Política de Push Único y Controlado**:
De acuerdo con las reglas innegociables de gobernanza de `agent-core`, queda terminantemente prohibido realizar `git push` intermedio durante las tareas individuales TDD (`T1` a `Tn`). El push remoto se autoriza **única y exclusivamente en esta Fase 3**, una vez completada la validación integral y obtenida la aprobación formal de `@reviewer` en `/sdd-validate`. Este paso es indispensable para publicar la rama en el repositorio remoto, permitir la apertura del Pull Request y disparar las compuertas automatizadas de CI en GitHub.

Una vez rebasada y sincronizada limpiamente en local:
1. El `@coordinator` despacha a `@implementer` para publicar la rama temática al repositorio remoto:
   ```bash
   git push -u origin <tipo>/<NNN-slug>
   ```
   *(Si se aplicó rebase y la rama ya existía en remoto: `git push --force-with-lease origin <tipo>/<NNN-slug>`)*.

---

## Fase 4: Apertura del Pull Request (Estrategia en 3 Niveles con Comprobación Pasiva)

El sistema intenta abrir el Pull Request utilizando la mejor herramienta disponible bajo un protocolo estricto de comprobación pasiva previa que previene excepciones y bloqueos interactivos:

1. **Nivel 1: Servidor GitHub MCP (Nativo del `@coordinator`)**:
   - **Comprobación Pasiva Previa**: El `@coordinator` verifica de forma pasiva si el servidor MCP `github` está disponible y con token activo configurado. Si el token no está configurado o el servidor MCP no responde, no lanza excepciones ni interrumpe el flujo; degrada de inmediato al Nivel 2.
   - Si está disponible y autenticado, invoca la herramienta `create_pull_request`:
     - Base: `<base_branch>` (ej. `develop` o `main`).
     - Head: `<tipo>/<NNN-slug>`.
     - Título: `<tipo>(<NNN>): <nombre descriptivo de la feature>`.
     - Cuerpo: Resumen de requerimientos funcionales, criterios de aceptación, comandos de verificación y referencia a `specs/NNN-slug/spec.md`.
2. **Nivel 2: GitHub CLI (`gh`) (Delegado a `@implementer`)**:
   - **Comprobación Pasiva Previa**: El `@coordinator` despacha a `@implementer` para ejecutar una comprobación pasiva de autenticación con `gh auth status`.
   - Si `gh` no está instalado en el sistema o `gh auth status` falla (no autenticado / token expirado), **NO** se lanzan excepciones ni se abren prompts interactivos bloqueantes (prohibido lanzar comandos interactivos como `gh auth login`). El arnés degrada automática e inmediatamente al Nivel 3.
   - Si `gh auth status` es exitoso, el coordinador despacha a `@implementer` para abrir el PR:
     ```bash
     gh pr create --base <base_branch> --head <tipo>/<NNN-slug> --title "<tipo>(<NNN>): <nombre>" --body "<resumen>"
     ```
3. **Nivel 3: Enlace Web Directo (Degradación Asistida No Bloqueante)**:
   - Si no hay MCP autenticado ni sesión activa en CLI, el coordinador genera e imprime la URL de apertura directa en el navegador:
     ```
     https://github.com/<owner>/<repo>/compare/<base_branch>...<tipo>/<NNN-slug>?expand=1
     ```

---

## Fase 5: Fusión y Sincronización a Producción (`main`)

1. **Autorización de Merge**:
   - Revisa `AGENT_CAN_MERGE` en `AGENTS.md`.
   - Si es `true`: consulta confirmación al usuario antes de proceder a la fusión.
   - Si es `false`: notifica que el PR queda abierto a la espera de revisión y aprobación humana en GitHub.
2. **Sincronización Asistida a Producción (`main`) mediante Release PR**:
   - Si la rama base del PR fue `develop`, tras fusionarse en `develop`, consulta al usuario:
     > *"¿Deseas sincronizar estos cambios a producción (`main`) mediante un Pull Request de Release ahora?"*
   - Si el usuario autoriza, de acuerdo con la Regla 5 de Gobernanza (prohibición estricta de merges directos locales a ramas protegidas o producción), la sincronización a `main` se realiza **exclusivamente mediante la apertura de un Pull Request de Release (`compare: develop -> base: main`)** sujeto a compuertas de CI en verde y aprobación humana:
     - **Nivel 1 (GitHub MCP)**: Si está disponible, `@coordinator` crea el PR: Base `main`, Head `develop`, Título `release: sincronizar develop a main`.
     - **Nivel 2 (GitHub CLI `gh`)**: Si no hay MCP, `@coordinator` despacha a `@implementer`:
       ```bash
       gh pr create --base main --head develop --title "release: sincronizar develop a main" --body "Release PR de sincronización de develop hacia producción."
       ```
     - **Nivel 3 (Enlace Web Asistido)**: Si no hay MCP ni CLI, el coordinador genera el enlace para apertura asistida en el navegador:
       ```
       https://github.com/<owner>/<repo>/compare/main...develop?expand=1
       ```
   - El Pull Request de Release queda abierto a la espera de que los workflows de CI pasen en verde y el equipo autorice el merge en GitHub. Queda estrictamente prohibido realizar merge local o push directo a `main`.

---

## Fase 6: Protocolo Estricto de Limpieza Post-Merge

Tras confirmarse la integración del PR en la rama base:
1. **REGLA MANDATORIA DE AUTORIZACIÓN**:
   - El agente **DEBE PREGUNTAR SIEMPRE AL USUARIO** antes de eliminar cualquier rama local o remota.
   - Queda terminantemente prohibido ejecutar `git branch -d` o `git push origin --delete` de forma automática sin permiso explícito.
2. Tras la confirmación del usuario, el `@coordinator` despacha a `@implementer` para ejecutar la limpieza de ramas:
   ```bash
   git checkout <base_branch>
   git pull origin <base_branch>
   git branch -d <tipo>/<NNN-slug>
   git push origin --delete <tipo>/<NNN-slug>
   ```

---

## Fase 7: Cierre de la Especificación y Memoria

1. `@planner` actualiza el encabezado de `specs/NNN-nombre/spec.md`:
   ```markdown
   - **Estado**: completada
   - **Fecha de Cierre**: AAAA-MM-DD
   ```
2. Registra el resumen en `MEMORY.md`:
   - Feature implementada y cerrada.
   - Decisiones de arquitectura tomadas y su justificación.
   - Poda activa para mantener `MEMORY.md` por debajo de ~50 líneas.
3. **Recuperación Asistida de `git stash` (Si Aplica)**:
   - Si al inicializar la spec en `/sdd-spec` se resguardaron cambios en stash:
     - El `@coordinator` consulta al usuario:
       > *"Se detectaron cambios resguardados en stash previos a esta spec. ¿Deseas recuperarlos ahora (`git stash pop`)?"*
     - Tras la autorización del usuario, el `@coordinator` despacha a `@implementer` para ejecutar `git stash pop` y verificar el estado del árbol de trabajo.
4. Emite un mensaje final de felicitación y cierre de ciclo:
   > *"🎉 Feature [NNN] integrada y cerrada con éxito. El repositorio está limpio, sincronizado y listo para la próxima especificación."*

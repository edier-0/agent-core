---
name: git-pr-workflow
description: Flujo estricto de Git, ramas protegidas, compuertas de calidad, apertura de PR en 3 niveles, sincronización a main y eliminación autorizada de ramas.
category: delivery
---

# Git & PR Workflow — Guía Canónica de Control de Versiones

Protocolo canónico de control de versiones, compuertas de calidad y ciclo de vida de Pull Requests para proyectos asistidos por agentes de IA. Garantiza aislamiento total de desarrollo, trazabilidad de cambios, cero regresiones y despliegues limpios hacia producción.

---

## 1. Principios Innegociables de Rama y Aislamiento

1. **Modos de Flujo de Trabajo (`GIT_WORKFLOW_MODE` en `AGENTS.md`)**:
   - `branch_pr` (por defecto): Todo trabajo se desarrolla en su rama temática `<tipo>/<NNN-nombre>` y se integra exclusivamente mediante Pull Request mediador.
   - `trunk`: Para prototipos o proyectos directos, las tareas se implementan en `main` bajo ciclo TDD con commits atómicos locales y cero push intermedio hasta que la spec completa esté validada.
2. **Ramas Base y Destino (`PR_TARGET_BRANCH` en `AGENTS.md`)**:
   - En flujos GitFlow estándar: `develop` como rama de integración y `main` como rama de producción.
   - En flujos GitHub Flow simplificados: `main` como rama base directa.
3. **Aislamiento Total (Feature Branches Dedicadas en `branch_pr`)**:
   - **PROHIBIDO** commitear directamente sobre la rama base (`develop` o `main`) en modo `branch_pr`.
   - Todo trabajo se ejecuta en una rama temática aislada partiendo siempre del estado más reciente de la rama base remota.
   - Prefijos semánticos obligatorios:
     - `feature/<NNN-nombre>`: Nuevas funcionalidades de negocio.
     - `fix/<NNN-nombre>`: Correcciones de bugs o hotfixes.
     - `chore/<NNN-nombre>`: Mantenimiento, dependencias o tooling.
     - `refactor/<NNN-nombre>`: Reestructuración de código sin alterar comportamiento.
     - `docs/<NNN-nombre>`: Documentación, especificaciones SDD o constitución.
   - **Numeración Correlativa Global Única (`NNN = max(NNN) + 1`)**:
     El cálculo del identificador correlativo `NNN` es unívoco y global para todo el repositorio en `specs/`, compartido transversalmente sin importar el prefijo semántico (`feature/`, `fix/`, `chore/`, `refactor/`, etc.). Cada nueva spec toma `max(NNN) + 1` del conjunto completo de carpetas existentes en `specs/`, eliminando colisiones en el árbol de trabajo.
4. **Prohibición de Merges Locales a Ramas Base**:
   - La rama base local nunca recibe merges manuales directos de la rama temática de trabajo. El Pull Request en remoto es el único mecanismo autorizado de integración.
5. **Protocolo Estricto de Puntos de Control Humanos**:
   - **Checkpoint 1:** Confirmación del usuario **ANTES** de abrir el Pull Request en GitHub (salvo orden expresa de ejecución desatendida).
   - **Checkpoint 2:** Confirmación del usuario **ANTES** de fusionar (merge) el Pull Request (salvo orden expresa de ejecución desatendida).
   - **Checkpoint 3 (Salvaguarda Inviolable):** Autorización explícita **ANTES** de eliminar cualquier rama, ya sea local (`git branch -d`) o remota (`git push origin --delete`). **Nunca** borrar ramas de forma automática.

---

## 2. Secuencia Operativa Paso a Paso

```text
[base] ───────────────────────────────────────────────────┬─► (PR remoto fusionado) ─► [producción]
  │                                                       ▲
  └──► [tipo/NNN-nombre] ──► Commits TDD ──► Pre-Push ────┘
                             Atómicos        Gates
```

### Paso 1: Inicialización Limpia de Rama
Partir siempre del estado idéntico al servidor remoto (`origin/<base_branch>`):
```bash
git checkout <base_branch>
git pull origin <base_branch>
git checkout -b <tipo>/<NNN-slug>
```

### Paso 2: Desarrollo y Commits Atómicos Locales
1. Aplicar la metodología **SDD** y ciclo **Red-Green-Refactor** de TDD:
   - Tests primero en rojo.
   - Implementación mínima necesaria en verde.
   - Refactorización y linters limpios.
2. Realizar commits granulares y autocontenidos bajo **Conventional Commits**:
   - `<tipo>(<NNN>): <Tn> - <descripción clara en español>`
3. **Staging Quirúrgico:** `git add <archivo1> <archivo2>` exclusivamente. Prohibido el uso indiscriminado de `git add .` o `git add -A`.
4. **Prohibición Total de Push Intermedio:** Los commits se acumulan localmente durante las tareas y no se suben al remoto hasta completar la spec.

### Paso 3: Puertas de Calidad Pre-Integración (Quality Gates)
Antes de sincronizar o publicar la rama, ejecutar y aprobar todas las verificaciones locales:
1. **Chequeo de Tipos y Linters:** `TYPECHECK_COMMAND` y `LINT_COMMAND`.
2. **Suite de Pruebas Automatizadas:** `TEST_COMMAND`.
3. **Compilación / Build:** `BUILD_COMMAND` (si aplica).
4. **Auditoría DevSecOps:** Inspeccionar el diff staged asegurando cero secretos o credenciales expuestas.

### Paso 4: Sincronización Pre-PR y Rebase
Para evitar conflictos en remoto y mantener un historial lineal limpio:
```bash
git fetch origin
git rebase origin/<base_branch>
```
*(Si opera en local-only / offline: `git rebase <base_branch>`).*

- **Resolución Asistida:** Resolver conflictos mecánicos triviales, ejecutar tests para verificar verde y continuar (`git rebase --continue`).
- **Parada Obligatoria:** Si el conflicto involucra lógica de negocio o decisiones de diseño, detenerse y solicitar resolución manual al usuario.

### Paso 5: Publicación (Push) al Remoto y Espera de CI
1. Publicar la rama temática al repositorio remoto:
   ```bash
   git push -u origin <tipo>/<NNN-slug>
   # O con lease si se rebasó:
   git push --force-with-lease origin <tipo>/<NNN-slug>
   ```
2. **Espera Obligatoria de CI en GitHub Actions:** Monitorear hasta que el 100% de los checks de CI concluyan exitosamente en verde. Si algún check falla, notificar, corregir localmente con TDD, re-publicar y volver a verificar.

### Paso 6: Apertura del Pull Request (Estrategia en 3 Niveles)
Tras la autorización del usuario y con CI en verde:
1. **Nivel 1 (GitHub MCP):** Invocar la herramienta nativa `create_pull_request`.
2. **Nivel 2 (GitHub CLI):** Si está instalado `gh`:
   ```bash
   gh pr create --base <base_branch> --head <tipo>/<NNN-slug> --title "<tipo>(<NNN>): <titulo>" --body "<cuerpo>"
   ```
3. **Nivel 3 (Enlace Web Directo):** Generar e imprimir la URL de compare en GitHub para apertura manual asistida.

### Paso 7: Fusión y Sincronización a Producción
1. Solicitar confirmación al usuario antes de fusionar el PR.
2. Ejecutar la fusión (Squash and Merge).
3. **Sincronización a Producción (`main`)**:
   - Si la rama base fue `develop`, sincronizar localmente `develop` y fusionar en `main`.
   - **Manejo de Ramas Protegidas:** Si GitHub rechaza el push directo a `main` por reglas de protección, generar automáticamente un Pull Request de Release (`develop -> main`).

### Paso 8: Salvaguarda Obligatoria para Eliminación de Ramas
Una vez integrado y validado el cambio:
1. **REGLA ESTRICTA:** Preguntar explícitamente al usuario:
   > *"Los cambios han sido integrados con éxito en la rama base. ¿Autorizas la eliminación de la rama temática `<tipo>/<NNN-slug>` en local y remoto?"*
2. **Únicamente tras confirmación afirmativa del usuario:**
   ```bash
   git checkout <base_branch>
   git pull origin <base_branch>
   git branch -d <tipo>/<NNN-slug>
   git push origin --delete <tipo>/<NNN-slug>
   ```

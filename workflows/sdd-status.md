---
description: Consulta el estado de la spec activa en la máquina de estados SDD, recomienda el siguiente comando y gestiona el archivado seguro de specs descartadas.
---

# Workflow: Estado del Ciclo SDD y Gestión de Specs (`/sdd-status`)

Este workflow inspecciona la especificación activa en `specs/`, evalúa su estado actual dentro de la máquina de estados formal de SDD, informa del progreso de tareas y recomienda con exactitud el siguiente comando a ejecutar. También gestiona la cancelación y archivado seguro de specs descartadas.

**Gobernanza de Consola y Roles**:
Toda consulta o mutación del repositorio mediante comandos de consola Git (`git branch`, `git status`, `git add`, `git commit`, `git checkout`, `git reset`) se delega y ejecuta **exclusivamente a través de `@implementer`**. El `@coordinator` y `@planner` tienen prohibido ejecutar comandos de consola directamente.

---

## Modos de Invocación

1. **Consulta de Estado (`/sdd-status`)**: Inspecciona la spec activa y sugiere el siguiente paso.
2. **Descarte / Cancelación (`/sdd-status --cancel`)**: Cancela de forma limpia y segura la spec activa.

---

## PARTE 1: Inspección de Estado

### Fase 1: Detección Determinista de Spec Activa (Árbol de Decisión de 3 Niveles)

El `@coordinator` determina la especificación activa siguiendo un árbol de decisión estricto de 3 niveles:

1. **Nivel 1 (Rama Git Activa)**:
   - Si la rama actual coincide con el patrón `feature/NNN-slug` (o `fix/NNN-slug`), la spec activa es inequívocamente `specs/NNN-slug/`.
2. **Nivel 2 (Estado en Metadatos de Specs)**:
   - Si se opera en `main`, `develop` o una rama genérica, escanea todas las carpetas en `specs/` y revisa el encabezado de cada `spec.md`.
   - Si existe **exactamente una spec** con `- **Estado**: en_progreso` (o `planificada`), se selecciona automáticamente dicha spec.
3. **Nivel 3 (Ambigüedad o Ausencia)**:
   - Si se detectan múltiples specs en progreso o ninguna, no asumir ninguna por defecto.
   - Presenta al usuario una tarjeta interactiva (`ask_question` o menú estructurado) listando las specs encontradas con su estado actual para que el usuario elija explícitamente sobre cuál desea operar.

*(Para determinar la rama Git actual en plataformas multiagente, el `@coordinator` despacha una consulta de lectura no destructiva a `@implementer` si no cuenta con dicha información en su contexto)*.

---

### Fase 2: Evaluación en la Máquina de Estados Canónica

| Estado | Condición Detectada | Siguiente Comando Recomendado |
| :--- | :--- | :--- |
| **`borrador`** | `spec.md` creada por `/sdd-spec`, pendiente de clarificación de QA | `/sdd-clarify` |
| **`clarificada`** | `spec.md` aprobada; `plan.md` aún no generado | `/sdd-plan` |
| **`clarificada`** | `plan.md` aprobado; `tasks.md` aún no generado | `/sdd-tasks` |
| **`planificada`** | `plan.md` y `tasks.md` listos; tareas pendientes de iniciar | `/sdd-implement` (o `/sdd-implement T1`) |
| **`en_progreso`** | Tareas en desarrollo con tests bajo ciclo TDD | `/sdd-implement` (siguiente tarea pendiente) |
| **`en_progreso`** | 100% de tareas completadas (`- [x]`) pero sin QA integral | `/sdd-validate` |
| **`validada`** | `@reviewer` emitió `VEREDICTO: APROBADO` RF por RF | `/review-and-pr` |
| **`completada`** | PR integrado en rama base y registrado en `MEMORY.md` | *(Ninguno — spec finalizada)* |
| **`descartada`** | Spec cancelada explícitamente y archivada | *(Ninguno — spec archivada)* |

---

### Fase 3: Emisión del Informe de Estado

Presenta al usuario un reporte estructurado y conciso:
```markdown
### Estado de la Especificación Activa
- **Spec**: specs/NNN-slug/
- **Rama Actual**: feature/NNN-slug
- **Estado SDD**: [borrador | clarificada | planificada | en_progreso | validada | completada]
- **Avance de Tareas**: X / Y tareas completadas ([X/Y]%)
- **Próximo Comando Recomendado**: `/<comando-recomendado>`
```

---

## PARTE 2: Protocolo de Cancelación y Descarte (`--cancel`)

Si el usuario decide cancelar o descartar un requerimiento:

1. **Actualización de Metadatos**:
   - `@planner` actualiza el encabezado de `specs/NNN-nombre/spec.md`:
     ```markdown
     - **Estado**: descartada
     - **Fecha de Cancelación**: AAAA-MM-DD
     - **Motivo**: [descripción o justificación del descarte]
     ```
2. **Tratamiento según el Modo de Git (Delegado a `@implementer`)**:

   #### En modo `branch_pr`:
   - El `@coordinator` despacha a `@implementer` para realizar el commit local de archivado en la rama temática:
     ```bash
     git add specs/NNN-nombre/spec.md
     git commit -m "docs(NNN): archivar spec descartada"
     ```
   - `@implementer` devuelve el árbol de trabajo a la rama base:
     ```bash
     git checkout <base_branch>
     ```
   - El `@coordinator` solicita confirmación explícita al usuario mediante tarjeta de selección interactiva:
     - `[A] (Recomendada) Renombrar rama a abandoned/NNN-slug para preservar trazabilidad histórica (git branch -m feature/NNN-slug abandoned/NNN-slug)`.
     - `[B] Eliminar rama local feature/NNN-slug (git branch -D) descartando el código parcial`.
     - `[C] Conservar rama intacta para reanudar en el futuro`.
   - Tras la respuesta del usuario, `@coordinator` despacha a `@implementer` para ejecutar la opción elegida.

   #### En modo `trunk` (Limpieza Estricta de `main`):
   - Dado que los commits intermedios de tareas inconclusas residen localmente en `main`, `@coordinator` despacha a `@implementer` para ejecutar un reset al estado base sincronizado previo a la spec:
     ```bash
     git reset --hard origin/main
     ```
   - Inmediatamente después, `@implementer` genera un único commit limpio de archivado en `main`:
     ```bash
     git add specs/NNN-nombre/spec.md
     git commit -m "docs(NNN): archivar spec descartada"
     ```
   - Esto deja `main` 100% libre de código parcial o a medio implementar, preservando la justificación histórica de la cancelación en `specs/NNN-nombre/spec.md`.

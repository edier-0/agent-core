---
description: Valida integralmente la spec implementada mediante QA exhaustivo con @reviewer en [MODO: VALIDACION_IMPLEMENTACION] (tests globales, diff, DevTools y reversión limpia).
---

# Workflow: Validación Integral de la Especificación (`/sdd-validate`)

Este workflow ejecuta la validación técnica y funcional final de la feature implementada. Es orquestado por el Coordinador, quien despacha a `@implementer` para correr la suite global de pruebas y convoca a `@reviewer` en modo de auditoría de código para verificar que cada requerimiento funcional (RF) de la spec se cumpla plenamente antes de autorizar la apertura del Pull Request.

**Gobernanza de Consola y Roles**:
Toda ejecución de comandos de consola (`TEST_COMMAND`, `git status`, `git checkout`, `git clean`, `git reset`, `PKG_RESTORE_COMMAND`) se delega y ejecuta **exclusivamente a través de `@implementer`** (`invoke_subagent` con `Workspace: "inherit"`). El `@coordinator` despacha a `@implementer` para correr la suite de pruebas completa y entrega el resultado y logs a `@reviewer` para su auditoría técnica e imparcial de requerimientos. `@reviewer` audita el diff, código y pruebas sin ejecutar consola directamente en el hilo principal ni modificar código.

---

## Pre-Condiciones

1. Todas las tareas de `specs/NNN-nombre/tasks.md` deben estar marcadas como completadas (`- [x]`).
2. Todos los commits locales de las tareas individuales deben haber sido realizados limpiamente.

---

## Fase 1: Invocación de `@reviewer` en Modo Validación

1. **Ejecución Previa de Suite de Pruebas Global (`TEST_COMMAND`)**:
   - El `@coordinator` despacha a `@implementer` para ejecutar la suite de integración completa (`TEST_COMMAND`, o en monorrepositorios las pruebas de integración raíz).
   - `@implementer` ejecuta el comando y devuelve el reporte detallado de ejecución al `@coordinator`.
   - Si los tests fallan en esta fase, el coordinador no convoca a auditoría a `@reviewer`; solicita a `@implementer` corregir bajo TDD o detiene el flujo.
   - Si el 100% de los tests terminan en verde, el `@coordinator` inyecta al subagente revisor los resultados y el contexto:

> `[MODO: VALIDACION_IMPLEMENTACION]`  
> Audita la implementación completa de la spec en `specs/NNN-nombre/spec.md`.  
> Revisa la suite de pruebas global (`TEST_COMMAND`) ejecutada exitosamente por `@implementer`, el diff completo contra la rama base y las interfaces visuales si aplica.

### Dimensiones de Evaluación de QA:
1. **Auditoría de Resultados de la Suite Global (`TEST_COMMAND`)**:
   - `@reviewer` audita el log de ejecución provisto por `@implementer`, verificando que la cobertura de pruebas cubra todos los RFs y que ningún test haya sido omitido (`skip`) sin justificación.
2. **Inspección de `git diff` contra la Rama Base**:
   - Comprueba que el código implementado se adhiera estrictamente a `plan.md` y `docs/constitution.md`.
   - Verifica la ausencia de archivos temporales, logs, comentarios `TODO` huérfanos o código comentado.
3. **Auditoría DevSecOps**:
   - Inspecciona el diff completo asegurando que no existan credenciales, tokens ni fallbacks inseguros.
4. **Validación Visual con Chrome DevTools MCP (Si Aplica)**:
   - Solo se ejecuta si la spec modifica componentes de UI Y la variable `DEV_SERVER_URL` está configurada en `AGENTS.md`.
   - **Comprobación Pasiva de Servidor**: Comprueba si `DEV_SERVER_URL` responde. Si no está activo, `@reviewer` se detiene y solicita al usuario iniciar el servidor (`START_DEV_COMMAND`) en su consola. Prohibido lanzar procesos demonio en segundo plano dentro del subagente.
   - Si el proyecto es puramente backend, CLI o carece de interfaz visual, se omite esta comprobación.

---

## Fase 2: Evaluación de Veredicto

### Caso A: `VEREDICTO: CAMBIOS NECESARIOS`

`@reviewer` emite un informe estructurado:
```markdown
VEREDICTO: CAMBIOS NECESARIOS

Discrepancias detectadas:
1. [archivo:línea] [RF incumplido o test fallido] -> Esperado: [comportamiento correcto]
2. [archivo:línea] [problema de seguridad o convención] -> Esperado: [corrección]
```

#### Protocolo de Corrección (Máximo 2 Vueltas):
1. **Vuelta 1**: El Coordinador asigna las correcciones a `@implementer`, quien ajusta el código bajo la disciplina TDD y realiza un commit correctivo (`fix(NNN): corregir observaciones de QA`). `@reviewer` re-valida.
2. **Vuelta 2**: Si persisten fallos, se repite el ciclo por una segunda y última vez.
3. **Parada ante Fallo Persistente tras la Vuelta 2**:
   - El Coordinador **NO** borra el código automáticamente; preserva el árbol de trabajo con el diff visible.
   - Detiene el flujo y consulta al usuario:
     > *"La validación técnica no superó la segunda vuelta de QA. ¿Deseas (1) intentar un enfoque alternativo, (2) revertir la feature completa al estado inicial limpio de la rama, o (3) ajustar manualmente los archivos?"*

#### Protocolo de Reversión Limpia (Si el Usuario Elige Revertir):
1. Limpieza de archivos modificados y reset al estado base (ejecutado por `@implementer` tras despacho del `@coordinator`):
   ```bash
   git checkout .
   git clean -fd
   git submodule update --init --recursive 2>/dev/null || true
   # En modo branch_pr con remoto origin:
   git reset --hard origin/<base_branch>
   # En modo branch_pr local-only:
   # git reset --hard <base_branch>
   # En modo trunk:
   # git reset --hard origin/main
   ```
2. **Restauración de Dependencias**: Si se modificaron dependencias, `@implementer` corre `PKG_RESTORE_COMMAND` (ej. `npm install` o `pip install -r`).
3. **Alerta de Base de Datos**: Si hubo migraciones, alerta al usuario para que revierta su base de datos local según su flujo habitual (sin ejecutar comandos destructivos automatizados).

---

### Caso B: `VEREDICTO: APROBADO`

Cuando `@reviewer` confirma que la suite pasa, el diff es limpio y los RFs están cubiertos:
```markdown
VEREDICTO: APROBADO
La implementación cumple íntegramente los requerimientos funcionales, contratos técnicos y compuertas de calidad.
```

---

## Fase 3: Aprobación Humana y Cierre Técnico

1. El Coordinador presenta el balance final al usuario (resumen de cambios, tests superados y diff de archivos).
2. **Protocolo ante Respuesta Humana**:
   - **Si el usuario aprueba**:
     - `@planner` actualiza el encabezado de `specs/NNN-nombre/spec.md`:
       ```markdown
       - **Estado**: validada
       ```
     - Sugiere proceder con la apertura del Pull Request:
       > *"✅ Feature validada y lista para integración. Para sincronizar la rama base, abrir el PR y gestionar el merge, ejecuta `/review-and-pr`."*
   - **Si el usuario rechaza por desajuste menor**: El Coordinador reasigna a `@implementer` con el feedback puntual bajo TDD.
   - **Si el usuario rechaza por nuevos requerimientos**: Se detiene la validación y se invoca `/sdd-change` para reformular el alcance.

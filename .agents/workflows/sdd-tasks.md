---
description: Desglosa el plan técnico en una lista secuencial de máximo 10 tareas atómicas (~20-30 min) en specs/NNN-nombre/tasks.md con sintaxis estricta de checkpoints y TDD.
---

# Workflow: Desglose de Tareas Atómicas (`/sdd-tasks`)

Este workflow toma el plan técnico aprobado y genera la lista secuencial de tareas de implementación en `specs/NNN-nombre/tasks.md`. Es ejecutado por `@planner` bajo la coordinación del agente principal.

---

## Pre-Condiciones

1. `specs/NNN-nombre/spec.md` debe estar en `Estado: clarificada`.
2. `specs/NNN-nombre/plan.md` debe estar formalmente aprobado por el usuario.

---

## Reglas Innegociables de Desglose

1. **Límite Estricto de 10 Tareas y Dimensionamiento Operativo (~20-30 min)**:
   - Una spec no puede contener más de 10 tareas (`T1` a `T10`).
   - Cada tarea debe tener un alcance atómico dimensionado en aproximadamente **20 a 30 minutos de ejecución agéntica**, definido operativamente como:
     1. **Alcance Cognitivo**: 1 único caso de prueba unitario (fase roja) y el código de producción mínimo necesario para superarlo (fase verde).
     2. **Volumen de Código**: Modificación de máximo 1 a 3 archivos y un diff no superior a ~100-150 líneas netas.
     3. **Autonomía de Ejecución**: Un ciclo de trabajo resoluble por el subagente en 2 a 4 turnos de herramientas sin requerir replanificación arquitectónica.
2. **Orden Secuencial y Dependencias**:
   - Las tareas se organizan en estricto orden de menor a mayor dependencia:
     1. Tareas fundacionales / contratos de interfaz / modelos / esquemas.
     2. Lógica de negocio y servicios con tests unitarios.
     3. Controladores / Endpoints / Vistas UI e integración.
     4. Ajustes finales, documentación y linter.
3. **Disciplina TDD en Cada Tarea**:
   - Cada tarea ejecutable debe incluir la redacción previa de pruebas automatizadas que fallen en rojo antes de escribir el código de producción.
4. **Sintaxis Rígida de Checkpoints**:
   - Para evitar romper el reconocimiento automatizado de casillas booleanas (`- [ ]`), los checkpoints se escriben **estrictamente como sub-viñetas indentadas** debajo de la tarea:
     ```markdown
     - [ ] T2: Implementar servicio de autenticación y generación de JWT
       - *Checkpoint*: Tests unitarios redactados en rojo en `tests/auth.service.test.ts`. Código de servicio pendiente.
     ```
   - La casilla principal permanece intacta como `- [ ]` hasta que la tarea completa esté en verde y verificada, momento en el cual se marca `- [x]` y se remueve o consolida la sub-viñeta.

---

## Fase 1: Redacción de `specs/NNN-nombre/tasks.md`

Utilizando la plantilla `.agents/templates/specs/tasks.md.template` (o `.agent/templates/specs/tasks.md.template`), redacta el archivo con la siguiente estructura:

```markdown
# Tareas de Implementación NNN: [Nombre de la Feature]

- **Spec**: specs/NNN-slug/spec.md
- **Plan**: specs/NNN-slug/plan.md
- **Total de Tareas**: X (máximo 10)

---

## Lista de Tareas Secuenciales

- [ ] T1: Configurar contratos y modelos de datos
  - *Hecho cuando*: Esquemas de validación redactados y tests de tipado en verde.
  - *Archivos*: `src/contracts/`, `tests/contracts.test.ts`.

- [ ] T2: Implementar lógica de negocio del servicio principal
  - *Hecho cuando*: Pruebas unitarias en verde (`vitest run tests/feature.test.ts`).
  - *Archivos*: `src/services/feature.service.ts`, `tests/feature.test.ts`.

- [ ] T3: Exponer endpoint API con middleware de validación
  - *Hecho cuando*: Supertest devuelve 200 OK con payload válido y 400 ante payload inválido.
  - *Archivos*: `src/controllers/`, `src/routes/`.
```

---

## Fase 2: Validación Humana y Transición de Estado

1. El Coordinador presenta la lista de tareas al usuario para su aprobación.
2. Tras la confirmación y aprobación de las tareas por parte del usuario:
   - `@planner` actualiza el encabezado de `specs/NNN-nombre/spec.md`:
     ```markdown
     - **Estado**: en_progreso
     ```
     *(Formalizar `Estado: en_progreso` al concluir la planificación de tareas garantiza que la spec quede lista para el desarrollo sin requerir que `@implementer` intente editar `spec.md`)*.
3. Indica el siguiente comando a ejecutar:
   > *"✅ Tareas desglosadas y aprobadas. La especificación pasa a estado `en_progreso`. Para iniciar la implementación de la primera tarea bajo el ciclo TDD, ejecuta `/sdd-implement` (o `/sdd-implement T1`)."*

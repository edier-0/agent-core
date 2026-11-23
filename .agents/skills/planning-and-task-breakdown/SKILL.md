---
name: planning-and-task-breakdown
description: Descompone especificaciones aprobadas en planes de arquitectura ejecutables (plan.md) y tareas atómicas secuenciales (tasks.md) de 20-30 min con checkpoints y TDD.
category: methodology
---

# Planning & Task Breakdown — Guía de Planificación Técnica

Descompone especificaciones funcionales aprobadas en planes de arquitectura ejecutables (`plan.md`) y tareas granulares y ordenadas (`tasks.md`). Una planificación meticulosa marca la diferencia entre un desarrollo predecible sin regresiones y una implementación caótica.

---

## 1. Cuándo Aplicar Esta Skill

- Al completar y clarificar una especificación funcional en **`sdd`** (`specs/NNN-nombre/spec.md`).
- Cuando un requerimiento se sienta demasiado complejo o amplio para empezar a codificar directamente.
- Para estimar el alcance técnico y definir la secuencia rigurosa de dependencias entre base de datos, backend y frontend.
- En los workflows ejecutables `/sdd-plan` y `/sdd-tasks`.

> **Regla de Oro:** **Cero Código durante la Planificación.** En esta fase opera en modo lectura y diseño. La salida son exclusivamente los documentos `plan.md` y `tasks.md`, nunca código de producción.

---

## 2. Elaboración del Plan Técnico (`plan.md`)

El plan técnico define el **CÓMO** de la solución, mapeando responsabilidades y mitigando riesgos:

1. **Arquitectura y Responsabilidades:**
   - Listado exacto de componentes, módulos y capas a crear o modificar.
2. **Grafo Canónico de Dependencias:**
   ```text
   Modelo de Persistencia / Esquema de Base de Datos
        │
        ▼
   Contratos de Datos, Esquemas de Validación y Tipos
        │
        ▼
   Servicio de Lógica de Negocio (Transacciones e Invariantes)
        │
        ▼
   Controladores, Endpoints de API o Interfaces de Comunicación
        │
        ▼
   Pruebas de Integración Automatizadas (TEST_COMMAND)
        │
        ▼
   Vistas, Componentes de UI y Experiencia de Usuario
   ```
3. **Decisiones Técnicas Justificadas:**
   - Registrar la alternativa elegida y por qué se descartaron las demás (ej. cálculo en base de datos vs. en memoria).
4. **Estrategia de Testing y Verificación:**
   - Casos límite y matriz de pruebas a cubrir con `TEST_COMMAND`.

---

## 3. Descomposición de Tareas (`tasks.md`)

Las tareas deben cumplir los siguientes estándares innegociables:

1. **Tamaño Atómico:** Cada tarea debe poder implementarse, probarse y verificarse en **20 a 30 minutos**.
2. **Límite Estricto de 10 Tareas:** Si un requerimiento exige más de 10 tareas atómicas, debe dividirse obligatoriamente en specs secuenciales por fases (ej. backend en Fase 1 y frontend en Fase 2).
3. **Criterio de Aceptación Verificable:** Cada tarea debe incluir una condición exacta de finalización (*Hecho cuando...*).
4. **Sintaxis Rígida de Checkpoints:**
   - Los checkpoints de progreso se escriben estrictamente como **sub-viñetas indentadas** debajo de la tarea:
     ```markdown
     - [ ] T2: Implementar servicio de autenticación
       - *Checkpoint*: Tests unitarios redactados en rojo en `tests/auth.test.ts`. Servicio pendiente.
     ```
   - La casilla principal permanece intacta como `- [ ]` hasta que la tarea completa esté en verde y verificada, momento en el cual se marca `- [x]` y se consolida la sub-viñeta.

---

## 4. Plantilla de Tareas Secuenciales

```markdown
# Tareas de Implementación NNN: [Nombre de la Feature]

- **Spec**: specs/NNN-slug/spec.md
- **Plan**: specs/NNN-slug/plan.md
- **Total de Tareas**: X (máximo 10)

---

## Lista de Tareas Secuenciales

- [ ] T1: Diseñar esquemas de validación y tipos de datos
  - *Hecho cuando*: Pruebas unitarias de esquema pasen en verde validando datos correctos y rechazando inválidos.
  - *Archivos*: `src/contracts/`, `tests/contracts.test.ts`.

- [ ] T2: Implementar lógica de negocio y servicio de dominio
  - *Hecho cuando*: Las pruebas unitarias verifiquen la lógica y transacciones atómicas en verde.
  - *Archivos*: `src/services/`, `tests/service.test.ts`.

- [ ] T3: Exponer endpoints de API con validación estricta
  - *Hecho cuando*: Pruebas de integración confirmen HTTP 200/201 en payload válido y HTTP 400 ante datos erróneos.
  - *Archivos*: `src/controllers/`, `src/routes/`.

- [ ] T4: Construir componente de interfaz de usuario y accesibilidad
  - *Hecho cuando*: Componente renderizado conforme a diseño y compilación limpia (`npm run build`).
  - *Archivos*: `src/components/`.
```

---

## 5. Transición a la Implementación
Una vez que el usuario aprueba formalmente `plan.md` y `tasks.md`, se inicia la codificación secuencial tarea por tarea ejecutando el workflow `/sdd-implement` bajo la disciplina de **`test-driven-development`** e **`incremental-implementation`**.

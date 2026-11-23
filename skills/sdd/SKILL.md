---
name: sdd
description: Guía canónica de Spec-Driven Development (SDD) para agent-core. Usar al redactar, clarificar, planificar, implementar o validar especificaciones en specs/NNN-nombre/.
category: methodology
---

# Spec-Driven Development (SDD)

El ciclo de desarrollo guiado por especificaciones rige todo el desarrollo de software en el proyecto. Separa rigurosamente la definición del problema (QUÉ) del diseño técnico (CÓMO) y de la ejecución (CÓDIGO).

---

## 1. El Flujo Canónico de Fases

```text
Constitución → Spec → Clarificación → Plan → Tareas → Implementación (TDD) → Validación (QA) → Cambio
```

### Reglas Innegociables:
1. **Aprobación Explícita**: Nunca pases a la siguiente fase sin la aprobación explícita del usuario.
2. **La Spec Manda**: Si un comportamiento o requerimiento no está en la spec, no se implementa. Si falta una decisión de negocio, detente y pregunta.
3. **Orden de Modificación**: Cualquier cambio de requisitos se realiza primero en la spec (`spec.md`), luego en el plan (`plan.md`), después en las tareas (`tasks.md`) y por último en el código.
4. **Centralización en `specs/`**: Cada spec reside exclusivamente en su propia carpeta en la raíz: `specs/NNN-nombre/` conteniendo `spec.md`, `plan.md` y `tasks.md`.
5. **Máquina de Estados**: `borrador` → `clarificada` → `planificada` → `en_progreso` → `validada` → `completada` (o `descartada`).
6. **Memoria Viva**: Al concluir cada fase o tarea, actualiza `MEMORY.md`.
7. **Exhaustividad Inicial en Proyectos Greenfield (Día 0)**: Para crear la primera versión de los requerimientos de un proyecto nuevo, el agente debe recorrer de forma minuciosa el Framework de los 8 Ejes Técnicos (Dominio, Stack, Arquitectura, Base de Datos, Seguridad/DevSecOps, Testing/TDD, Infraestructura/Docker y Gobernanza Git) sin dejar cabos sueltos ni ambigüedades arquitectónicas antes de escribir código.

---

## 2. Requisitos en Sintaxis Formal EARS (Español)

Todo requerimiento funcional en `spec.md` debe formularse en una de las 5 variantes de sintaxis EARS:
- **Ubicuo**: `RF-x: EL SISTEMA [hará X]`.
- **Disparado por evento**: `RF-x: CUANDO [evento], EL SISTEMA [hará X]`.
- **Disparado por estado**: `RF-x: MIENTRAS [estado], EL SISTEMA [hará X]`.
- **Condicional / Excepcional**: `RF-x: SI [condición], ENTONCES EL SISTEMA [hará X]`.
- **Complejo (Evento + Estado)**: `RF-x: MIENTRAS [estado], CUANDO [evento], EL SISTEMA [hará X]`.

*Cada RF debe ser medible y verificable: prohibido usar adjetivos vagos como "rápido", "intuitivo" o "seguro" sin métricas concretas.*

---

## 3. Estructura de Artefactos por Spec

- **`spec.md` (Qué y Por Qué)**: Describe el problema, actores, historias de usuario, requerimientos EARS, casos límite y criterios de aceptación. Cero detalles de código, rutas de archivos ni frameworks.
- **`plan.md` (Cómo Técnico)**: Arquitectura, contratos de interfaz, esquemas de validación, persistencia y estrategia de pruebas con `TEST_COMMAND`. Límite estricto de máximo 10 tareas atómicas.
- **`tasks.md` (Ejecución)**: Tareas secuenciales atómicas (~20-30 min) con sintaxis estricta de checkpoints como sub-viñetas y condición "Hecho cuando:".

---

## 4. Implementación y Validación

- **Una Sola Tarea a la Vez**: Ciclo TDD estricto (tests en rojo primero, código mínimo en verde, refactorización con linters).
- **Verificación**: `TEST_COMMAND` en verde y commits convencionales por tarea (`<tipo>(<NNN>): <Tn> - <descripción>`).
- **Validación Final**: `@reviewer` audita la feature completa RF por RF antes de autorizar la apertura del Pull Request vía `/review-and-pr`.

---

## 5. Estándar de Encuestas y Preguntas al Usuario (Tarjetas con Opción Libre)

En cualquier fase donde el agente necesite consultar decisiones de diseño, resolver vacíos de la spec o clarificar requerimientos con el usuario:
- **Herramienta Nativa (`ask_question`)**: Si el entorno la admite, usar obligatoriamente la herramienta interactiva de preguntas. Proporcionar las opciones sin numerar, la recomendada de primero con `(Recommended)`. La interfaz muestra tarjetas interactivas y la opción automática para escribir otra respuesta (*"Other"*).
- **Modo Chat / Markdown Plano**: Formatear como **Tarjetas de Opciones**:
  ```markdown
  > **[Pregunta concreta sobre la decisión o caso límite]**
  >
  > - **[A] (Recomendada)**: [Opción sugerida con justificación arquitectural]
  > - **[B]**: [Opción alternativa]
  > - **[O] Escribir otra respuesta**: (Escribe tu alternativa personalizada si aplica).
  ```
- **Regla Estricta**: Queda prohibido formular preguntas abiertas difusas o listados densos sin opciones predefinidas. Siempre se ofrece una vía rápida de selección y una salida abierta para texto libre.

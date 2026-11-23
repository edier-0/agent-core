---
name: context-engineering
description: Optimiza la carga y gestión de contexto para agentes de IA. Usar al iniciar sesiones, cuando la calidad de respuesta decaiga o al alternar entre módulos.
category: architecture
---

# Context Engineering — Guía de Gestión de Contexto

Alimenta a los agentes con la información precisa en el momento oportuno. El contexto es la palanca más determinante en la calidad y exactitud del código generado: muy poco contexto produce alucinaciones; demasiado contexto satura la ventana de atención y genera pérdida de foco o desobediencia a las reglas.

---

## 1. La Jerarquía de Contexto en el Repositorio

Estructura el contexto desde lo más persistente hasta lo más transitorio:

```text
┌────────────────────────────────────────────────────────┐
│ 1. REGLAS PERMANENTES: AGENTS.md y docs/constitution.md│ ← Siempre activas en el repositorio
├────────────────────────────────────────────────────────┤
│ 2. MEMORIA DE SESIONES: MEMORY.md                      │ ← Estado actual, decisiones y errores a evitar (~50 líneas)
├────────────────────────────────────────────────────────┤
│ 3. ESPECIFICACIÓN ACTIVA: specs/NNN-nombre/            │ ← spec.md, plan.md y tasks.md de la tarea
├────────────────────────────────────────────────────────┤
│ 4. CÓDIGO DEL DOMINIO / MÓDULO                         │ ← Solo los archivos relevantes de la tarea actual
├────────────────────────────────────────────────────────┤
│ 5. EVIDENCIA DE EJECUCIÓN: Errores y Tests             │ ← Logs quirúrgicos de fallos (TEST_COMMAND / Linters)
└────────────────────────────────────────────────────────┘
```

---

## 2. Buenas Prácticas de Ingeniería de Contexto

1. **Leer `MEMORY.md` al Iniciar:** Antes de cualquier cambio, consulta el estado actual y las decisiones previas para no repetir errores pasados.
2. **Inspección Quirúrgica por Rangos y Búsqueda Selectiva:**
   - Prohibido el volcado completo de archivos masivos a la ventana de contexto.
   - Emplear herramientas de búsqueda selectiva (`grep`, offsets de línea, búsqueda de símbolos) para leer exclusivamente las funciones o bloques afectados.
3. **Carga Focalizada de Archivos:**
   - Carga exclusivamente los archivos afectados por la tarea activa y su suite de pruebas correspondiente.
   - No leas archivos masivos no relacionados (archivos generados, bundles de build, esquemas de migración pasados) que solo degradan la atención del modelo.
4. **Manejo de Saturación de Contexto:**
   - Si una sesión de conversación acumula demasiadas interacciones y notas que el agente empieza a ignorar convenciones o a responder con menor precisión, compacta el contexto (`/compact`) o reanuda en una nueva sesión apuntando directamente a `tasks.md`.
5. **Poda Activa de `MEMORY.md`:**
   - Mantén `MEMORY.md` conciso (máximo ~50 líneas). Al finalizar una tarea o spec, resume lo implementado y elimina lo que ya no aporte valor futuro. Si una decisión se vuelve regla permanente, muévela a `AGENTS.md` o a la constitución.

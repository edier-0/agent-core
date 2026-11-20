---
name: constraint-driven-development
description: Establece y defiende las restricciones de calidad e invariantes técnicos del proyecto. Impide la degradación silenciosa del estándar (supresiones @ts-ignore, tests saltados o rebajas de aserciones).
---

# Constraint-Driven Development — Guía de Barras de Calidad

Establece la barra de calidad del proyecto como un contrato explícito e impide que los agentes o desarrolladores la degraden silenciosamente para dar por terminada una tarea. En proyectos de desarrollo asistido por IA, los agentes generan código a gran velocidad; esta skill garantiza que no se silencien errores con atajos artificiales (`@ts-ignore`, `eslint-disable`, `test.skip`, `as any` o eliminación de aserciones).

---

## 1. La Barra de Calidad Innegociable

En nuestro repositorio, las restricciones fundamentales emanan de la **Constitución del Software** (`docs/constitution.md`) y la sección de límites de `AGENTS.md`:

```text
┌────────────────────────────────────────────────────────┐
│            RESTRICCIONES TÉCNICAS SUPREMAS             │
├────────────────────────────────────────────────────────┤
│ 1. Cero Supresiones de Tipos (@ts-ignore / as any)     │
│ 2. Cero Tests Saltados o Comentados (it.skip / # skip) │
│ 3. Cero Fallbacks de Credenciales (|| 'secret_token') │
│ 4. Respeto Estricto de Invariantes de Dominio y Datos  │
│ 5. Control de Cambios Vía PR (Sin merges a rama base)  │
└────────────────────────────────────────────────────────┘
```

---

## 2. Detección de Degradación de la Barra de Calidad (*Anti-Cheating Guard*)

Al revisar cualquier diferencial de código (`git diff`), se verifica que **no** se haya introducido ninguna de las siguientes trampas:

| Patrón Sospechoso | Intento Tramposo Típico | Veredicto |
| :--- | :--- | :--- |
| `// @ts-ignore` o `// @ts-nocheck` | Ocultar un error de tipos en vez de tipar el modelo o la función. | 🔴 Prohibido. Tipar correctamente con esquemas o interfaces. |
| `it.skip(...)` o `@pytest.mark.skip` | Apagar un test que falló para que la suite termine en verde. | 🔴 Prohibido. Corregir la regresión de raíz. |
| `as any` o casts inseguros | Saltear restricciones de validación o tipos de base de datos. | 🔴 Prohibido. Usar tipos seguros generados por el ORM/framework. |
| Reducción de aserciones en tests | Eliminar `expect(total).toBe(...)` dejando solo `expect(res).toBeDefined()`. | 🔴 Prohibido. Toda prueba debe evaluar valores específicos y exactos. |
| Fallbacks permisivos silenciosos | `try { ... } catch (e) { return null; }` | 🔴 Prohibido. Capturar y canalizar excepciones adecuadamente. |

---

## 3. Protocolo de Defensa de Restricciones

1. **Si un test falla:** Aplica la skill `debugging-and-error-recovery` y soluciona la causa raíz; **nunca** deshabilites la prueba ni bajes el umbral de aserción.
2. **Si el compilador de tipos arroja error:** Ajusta la firma de la función, modelo o esquema; **nunca** uses `@ts-ignore` o `any`.
3. **Si una regla de negocio se complica:** Detente y consulta al usuario; **nunca** asumas atajos que violen los invariantes de `docs/constitution.md`.

---
name: code-simplification
description: Simplifica el código reduciendo la complejidad cognitiva innecesaria mientras preserva exactamente el comportamiento. Usar tras pasar tests en TDD o al refactorizar código denso.
---

# Code Simplification — Guía de Reducción de Complejidad

Simplifica el código reduciendo la complejidad cognitiva mientras preservas exactamente el comportamiento funcional. El objetivo no es simplemente tener menos líneas de código; el objetivo es tener código que sea más fácil de leer, comprender, depurar y mantener por cualquier otro desarrollador o agente de IA.

> **La Pregunta de Oro de la Simplificación:**  
> *"¿Un nuevo ingeniero de software entendería esta implementación más rápido y con menos esfuerzo mental que la versión original?"*

---

## 1. Cuándo Usar la Simplificación

- Tras completar la fase verde de TDD (`test-driven-development`), cuando la funcionalidad ya opera pero la implementación se siente pesada o sobrecargada.
- Durante la revisión de código (`code-review-and-quality`) si se detectan anidamientos excesivos o nombres confusos.
- Al encontrarse con funciones demasiado largas (>40 líneas) o con múltiples responsabilidades cruzadas.
- Al consolidar lógica duplicada o dispersa en varios archivos.

> **Cuándo NO usar:**  
> - Si el código ya es claro y legible: no simplifiques por mero afán estético o estilo personal.  
> - Si aún no comprendes al 100% lo que hace el código: primero comprende y cubre con tests, luego simplifica.  
> - Si estás a punto de rediseñar o reemplazar el módulo por completo.

---

## 2. Los Cinco Principios Fundamentales

```text
┌────────────────────────────────────────────────────────┐
│             PRINCIPIOS DE SIMPLIFICACIÓN               │
├────────────────────────────────────────────────────────┤
│ 1. Preservar Comportamiento Exacto (Cero Regresiones) │
│ 2. Alinear con las Convenciones del Repositorio        │
│ 3. Reducir la Carga Cognitiva (Early Returns)          │
│ 4. Eliminar Indirecciones Prematuras e Innecesarias    │
│ 5. Purgar Código Muerto y Comentarios Obsoletos       │
└────────────────────────────────────────────────────────┘
```

### Principio 1: Preservar el Comportamiento Exacto
La simplificación cambia el *cómo*, nunca el *qué*. Todas las entradas, salidas, efectos secundarios, transacciones y excepciones deben permanecer idénticas.
- **Lista de Verificación:**
  - ¿Produce la misma respuesta ante entradas válidas e inválidas?
  - ¿Mantiene los mismos códigos de error o excepciones de negocio?
  - ¿Siguen pasando todos los tests existentes sin modificar una sola línea de aserción?

### Principio 2: Alinear con las Convenciones del Repositorio
- Respetar los estándares definidos en `AGENTS.md` (idioma, estilos de importación, separación de capas).
- No alterar la arquitectura establecida (Controlador $\rightarrow$ Servicio $\rightarrow$ Persistencia).

### Principio 3: Reducir Carga Cognitiva mediante Cláusulas de Guarda
Reemplaza escaleras profundas de `if / else` por salidas tempranas (*early returns*):

```typescript
// ❌ ANTES: Anidamiento profundo y difícil de seguir mentalmente
async function processOrderFulfillment(orderId: string, quantity: number) {
  const order = await database.order.findUnique({ where: { id: orderId } });
  if (order) {
    if (order.status === 'APPROVED') {
      if (order.availableQuantity >= quantity) {
        return await executeFulfillment(order, quantity);
      } else {
        throw new BusinessError('Cantidad disponible insuficiente', 400);
      }
    } else {
      throw new BusinessError('La orden no está aprobada para despacho', 400);
    }
  } else {
    throw new BusinessError('Orden no encontrada', 404);
  }
}

// ✅ DESPUÉS: Cláusulas de guarda lineales, planas y claras
async function processOrderFulfillment(orderId: string, quantity: number) {
  const order = await database.order.findUnique({ where: { id: orderId } });
  if (!order) {
    throw new BusinessError('Orden no encontrada', 404);
  }

  if (order.status !== 'APPROVED') {
    throw new BusinessError('La orden no está aprobada para despacho', 400);
  }

  if (order.availableQuantity < quantity) {
    throw new BusinessError('Cantidad disponible insuficiente', 400);
  }

  return executeFulfillment(order, quantity);
}
```

### Principio 4: Eliminar Indirecciones Prematuras
- No crees funciones auxiliares de 1 sola línea utilizadas en un único lugar si solo fragmentan la lectura.
- No construyas interfaces genéricas o factorías dinámicas abstractas si solo existe una sola implementación real en el sistema.

### Principio 5: Purgar Código Muerto y Huérfano
- Eliminar variables no utilizadas (`const _temp`).
- Eliminar bloques de código comentado (`// const oldLogic = ...`).
- Eliminar logs de depuración temporal (`console.log`, `print`).

---

## 3. Protocolo de Verificación Obligatorio

Cada sesión de simplificación debe seguir esta secuencia estricta:

```bash
# 1. Comprobar que el estado previo esté en verde
TEST_COMMAND
TYPECHECK_COMMAND

# 2. Aplicar la simplificación de forma quirúrgica en el archivo

# 3. Comprobar que el estado posterior siga en verde exacto
TEST_COMMAND
TYPECHECK_COMMAND
```

Si cualquier prueba falla o el tipado arroja error, **la simplificación alteró el comportamiento**. Reviértela de inmediato (`git checkout <archivo>`) o corrige la discrepancia antes de continuar.

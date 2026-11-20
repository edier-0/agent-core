---
name: performance-optimization
description: Optimiza el rendimiento de la aplicación en backend, consultas de base de datos y frontend. Usar ante problemas de lentitud, consultas N+1 o sobrecarga de memoria.
---

# Performance Optimization — Guía Sistemática de Rendimiento

Guía de optimización de rendimiento y buenas prácticas de ingeniería para bases de datos, backend y frontend. Establece el principio fundamental: **Mide primero, optimiza solo lo comprobado**. La optimización prematura sin métricas introduce complejidad innecesaria sin mejorar la experiencia real de los usuarios.

---

## 1. El Flujo Sistemático de Optimización

```text
1. MEDIR      ──► Establecer línea base con datos reales (tiempos de respuesta, consultas)
2. IDENTIFICAR──► Localizar el cuello de botella exacto (ej. consulta N+1 o loop en cliente)
3. RESOLVER   ──► Aplicar la corrección focalizada en la capa correcta
4. VERIFICAR  ──► Medir nuevamente; conservar el cambio solo si la mejora es medible
5. BLINDAR    ──► Proteger contra regresiones con índices o tests de rendimiento
```

---

## 2. Optimización de Base de Datos y Persistencia (Backend)

La gran mayoría de problemas de rendimiento en aplicaciones transaccionales provienen de patrones deficientes de acceso a datos.

### A. Erradicación del Problema N+1
Nunca ejecutes consultas dentro de un bucle iterativo:

```typescript
// ❌ INEFICIENTE (N+1): Ejecuta 1 consulta inicial + N consultas adicionales en bucle
const orders = await db.order.findMany();
for (const order of orders) {
  order.items = await db.orderItem.findMany({ where: { orderId: order.id } });
}

// ✅ ÓPTIMO: Una sola consulta relacional eficiente mediante JOIN o agregación
const orders = await db.order.findMany({
  include: {
    items: {
      include: {
        product: true
      }
    }
  }
});
```

### B. Agregaciones en Base de Datos vs. Memoria de la Aplicación
Deja que el motor de base de datos calcule los totales matemáticos en lugar de descargar miles de filas a la memoria RAM de la aplicación:

```typescript
// ❌ INEFICIENTE: Descarga 10.000 registros para sumarlos en memoria
const records = await db.payment.findMany({ where: { accountId } });
const total = records.reduce((acc, p) => acc + p.amount, 0);

// ✅ ÓPTIMO: Agregación nativa instantánea en la base de datos
const aggregate = await db.payment.aggregate({
  where: { accountId },
  _sum: { amount: true }
});
const total = aggregate._sum.amount ?? 0;
```

### C. Higiene en Transacciones Interactivas
Las transacciones de base de datos interactúan bloqueando conexiones y filas. **Prohibido realizar llamadas de red externas, envíos de correo o notificaciones adentro de una transacción**:

```typescript
// ❌ PELIGROSO: La llamada externa de red mantiene abierta la transacción de base de datos
await db.$transaction(async (tx) => {
  const order = await tx.order.create({ ... });
  await sendExternalNotification(order); // Si la red tarda 5s, bloquea la BD
});

// ✅ CORRECTO: Persistir primero la transacción atómica; emitir eventos externos después
const order = await db.$transaction(async (tx) => {
  return await tx.order.create({ ... });
});
// Notificación asíncrona desacoplada
sendExternalNotification(order).catch(console.error);
```

### D. Índices Estratégicos
Asegurar índices en campos frecuentemente filtrados, unidos en JOINs o utilizados para ordenamiento (`status`, `createdAt`, claves foráneas).

---

## 3. Optimización de Frontend y Carga de Red

### A. Carga Diferida por Rutas (*Route-level Lazy Loading*)
Cada vista principal debe cargarse bajo demanda mediante importación dinámica para mantener el bundle inicial liviano:

```typescript
const routes = [
  {
    path: '/orders',
    name: 'Orders',
    component: () => import('@/views/OrdersView.vue') // Chunk separado
  },
  {
    path: '/dashboard',
    name: 'Dashboard',
    component: () => import('@/views/DashboardView.vue')
  }
];
```

### B. Paginación y Poda de Listas
En vistas que gestionan cientos o miles de registros:
- Aplicar paginación o límites en servidor (`limit / offset` o paginación por cursor).
- Prohibido renderizar miles de nodos DOM directos sin paginación o virtual scrolling.

---

## 4. Checklist de Aprobación de Rendimiento
- [ ] No existen bucles de consultas asíncronas dentro de iteradores (cero N+1).
- [ ] Las consultas pesadas aplican proyección específica (`select`) para no transferir columnas innecesarias.
- [ ] Cálculos acumulativos o matemáticos se delegan al motor de base de datos.
- [ ] Cero operaciones I/O lentas de red dentro de bloques transaccionales.
- [ ] Empaquetado de frontend verificado sin dependencias pesadas innecesarias (`BUILD_COMMAND`).

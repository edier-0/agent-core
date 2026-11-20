---
name: observability-and-instrumentation
description: Instrumenta el código con registros estructurados (JSON), correlación y métricas de diagnóstico. Usar al crear endpoints, integraciones asíncronas o auditar flujos críticos.
---

# Observability & Instrumentation — Guía de Observabilidad y Diagnóstico

El código que no puedes observar es código que no puedes operar. La observabilidad es la capacidad de responder *"¿qué está haciendo el sistema y por qué?"* desde el exterior, mediante los registros estructurados, eventos y métricas que emite la aplicación. La instrumentación no es un accesorio posterior al lanzamiento; se escribe a la par que la funcionalidad.

---

## 1. Cuándo Aplicar Esta Skill

- Al construir nuevos endpoints o flujos de negocio que correrán en producción.
- Al implementar integraciones externas asíncronas (webhooks, colas de mensajería, conexiones WebSocket o APIs de terceros).
- Al procesar operaciones financieras, transaccionales o de inventario sensibles.
- Cuando un incidente en producción haya tomado demasiado tiempo en diagnosticarse por falta de información en los logs.

---

## 2. Los Tres Pilares de la Observabilidad

### A. Registros Estructurados (*Structured Logs*)
Los logs no son simples cadenas de texto crudas como `console.log('pasó por aquí')`. Deben emitirse como objetos JSON estructurados con contexto accionable:

```typescript
// ✅ CORRECTO: Registro estructurado con contexto accionable
logger.info({
  level: 'info',
  event: 'ORDER_STATUS_CHANGED',
  orderId: order.id,
  previousStatus: 'PENDING',
  newStatus: 'APPROVED',
  userId: req.user?.id,
  timestamp: new Date().toISOString()
});
```

### B. Trazabilidad de Auditoría de Negocio
En sistemas transaccionales, la auditoría más crítica no vive solo en stdout, sino de forma inmutable en la base de datos:
- Cada alteración de inventario o balance debe emitir un registro de auditoría (`deltaQuantity`, `previousValue`, `newValue`, `reason`, `userId`).
- Las transacciones financieras deben registrar explícitamente el canal de pago o pasarela y el identificador de referencia externo.

### C. Supervisión de Ciclo de Vida de Conexiones Asíncronas
En conexiones externas de larga duración (WebSockets, workers, sockets de mensajería):
- Distinguir con precisión entre errores transitorios de red (que ameritan reconexión exponencial automática) y errores permanentes de autenticación (que exigen intervención del operador).
- Prohibido borrar sesiones o tokens locales ante caídas transitorias de conexión.

---

## 3. Manejo y Registro Centralizado de Errores

Toda excepción no controlada debe canalizarse por un middleware centralizado de errores:
1. **En el Servidor:** Registra el error completo de forma estructurada con su stack trace, contexto del request y payload recibido (saneando contraseñas, tokens y datos de tarjetas).
2. **Hacia el Cliente:** Responde de forma saneada con un mensaje comprensible y código HTTP semántico, **sin exponer jamás contraseñas, tokens de API ni trazas internas de la base de datos**.

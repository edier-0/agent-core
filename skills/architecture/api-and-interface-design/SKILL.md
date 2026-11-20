---
name: api-and-interface-design
description: Guía el diseño de APIs REST y contratos de interfaz estables. Usar al diseñar endpoints, esquemas de validación, contratos entre módulos o límites entre backend y frontend.
---

# API & Interface Design — Guía Canónica de Diseño de APIs

Guía canónica para el diseño de APIs REST, contratos de datos y límites modulares. Un buen diseño de interfaz hace que la acción correcta sea intuitiva y la acción errónea sea imposible. Esta skill establece los estándares para controladores, servicios, esquemas de validación y comunicación cliente-servidor.

---

## 1. Cuándo Aplicar Esta Skill

- Al crear nuevos endpoints REST o actualizar rutas y contratos existentes.
- Al definir esquemas de validación de datos (Zod, Pydantic, Joi, Marshmallow).
- Al diseñar contratos entre la API y el cliente HTTP frontend.
- Al modelar DTOs (Data Transfer Objects) o tipos compartidos entre capas.
- Al modificar firmas públicas de servicios de dominio o librerías internas.

---

## 2. Principios Fundamentales de Diseño

### A. Ley de Hyrum
> *"Con un número suficiente de usuarios de una API, todos los comportamientos observables de tu sistema terminarán siendo dependencias para alguien, sin importar lo que prometa el contrato escrito."*

- **Exposición Intencional:** Expón únicamente los campos que el consumidor realmente necesita. No filtres hashes de contraseñas, tokens internos ni metadatos de infraestructura.
- **Estabilidad de Errores:** Emplea códigos HTTP semánticos y mensajes de error estandarizados y comprensibles.

### B. Contrato Primero (*Contract-First*)
Diseña el contrato antes de escribir la lógica del servicio:
1. Define el esquema de validación de entrada (`body`, `query`, `params`).
2. Define la interfaz del resultado tipado (DTO de salida).
3. Implementa el controlador y el servicio para satisfacer ese contrato.

---

## 3. Estructura Modular Canónica

Se recomienda organizar las APIs por módulos de dominio con responsabilidad única por archivo:

```text
src/modules/orders/
  ├── orders.routes.ts      # Enrutamiento, middlewares de auth y validación de esquema
  ├── orders.controller.ts  # Controlador delgado: extrae inputs, llama servicio, next(error)
  ├── orders.service.ts     # Lógica de dominio, transacciones y reglas de negocio
  └── orders.schema.ts      # Esquemas de validación de datos y tipos inferidos
```

### A. Esquemas y Validación de Entradas (`orders.schema.ts`)
```typescript
import { z } from 'zod';

export const createOrderSchema = z.object({
  customerId: z.string().uuid('ID de cliente inválido'),
  deliveryDate: z.string().datetime().optional(),
  items: z.array(
    z.object({
      productId: z.string().uuid('ID de producto inválido'),
      quantity: z.number().int().positive('La cantidad debe ser mayor a 0'),
      unitPrice: z.number().positive('El precio debe ser positivo')
    })
  ).min(1, 'La orden debe incluir al menos un ítem')
});

export type CreateOrderInput = z.infer<typeof createOrderSchema>;
```

### B. Controlador Delgado (*Thin Controller*)
El controlador no realiza cálculos matemáticos ni consultas directas a la base de datos; solo orquesta el transporte HTTP:
```typescript
export const createOrder = async (req: Request, res: Response, next: NextFunction) => {
  try {
    const order = await orderService.createOrder(req.body, req.user?.id);
    return res.status(201).json({
      success: true,
      data: order
    });
  } catch (error) {
    next(error);
  }
};
```

---

## 4. Estándar de Respuestas y Manejo de Errores

### A. Respuesta Exitosa Estandarizada
```json
{
  "success": true,
  "data": {
    "id": "e2b1-4f9a-8c7d",
    "orderNumber": "ORD-1042",
    "total": 450.00,
    "status": "APPROVED"
  }
}
```

### B. Respuesta de Error Controlada
```json
{
  "success": false,
  "error": "Stock insuficiente para satisfacer el ítem solicitado",
  "statusCode": 400
}
```

### C. Códigos de Estado HTTP Canónicos
- `200 OK`: Consulta o actualización exitosa.
- `201 Created`: Recurso creado satisfactoriamente.
- `400 Bad Request`: Error de validación de esquema o regla de negocio violada.
- `401 Unauthorized`: Falta de token de autenticación o sesión expirada.
- `403 Forbidden`: Permisos o roles insuficientes (RBAC).
- `404 Not Found`: Recurso no encontrado.
- `409 Conflict`: Conflicto de unicidad, versión o concurrencia.
- `500 Internal Server Error`: Error interno no controlado (capturado por el middleware global).

---

## 5. Reglas Innegociables de API

1. **Cero Cuerpos Crudos:** Prohibido pasar `req.body` directo a la capa de persistencia sin parsear previamente por un esquema de validación estricto.
2. **Consumo Centralizado en Frontend:** El cliente frontend consume la API exclusivamente a través de un cliente HTTP centralizado (ej. `src/api/client.ts`). Prohibido instanciar llamadas `fetch()` o clientes ad-hoc dispersos en componentes.
3. **Marcas de Tiempo Explícitas:** Fechas y timestamps se transmiten en formato ISO 8601 estándar (`YYYY-MM-DDTHH:mm:ss.sssZ`).

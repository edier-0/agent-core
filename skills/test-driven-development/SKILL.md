---
name: test-driven-development
description: Guía el desarrollo guiado por pruebas mediante el ciclo Red-Green-Refactor. Usar al implementar nueva lógica, corregir errores o modificar cualquier comportamiento existente.
category: methodology
---

# Test-Driven Development (TDD) — Guía Canónica

Escribe una prueba que falle antes de escribir el código que la haga pasar. Para la corrección de errores, reproduce primero el fallo con un test antes de intentar solucionarlo (Patrón *Prove-It*). Las pruebas son la evidencia irrefutable de funcionamiento; "parece que funciona" no es un criterio de entrega aceptable.

---

## 1. Cuándo Usar TDD

- **Siempre que implementes nueva lógica o comportamiento de negocio** (cálculos, validaciones, flujos de estado, procesamiento de datos).
- **Al corregir cualquier bug** (escribe primero el test que demuestre y reproduzca el fallo reportado).
- **Al modificar funcionalidad existente o refactorizar código.**
- **Al incorporar casos límite** (saldos negativos, valores nulos, divisiones por cero, estados de error).
- **Cualquier cambio con riesgo de regresión.**

> **Cuándo NO usar:** Modificaciones exclusivas de documentación en Markdown, assets estáticos de imagen o estilos visuales puros sin lógica.

---

## 2. Herramental de Pruebas y Terminación Determinista

Antes de ejecutar pruebas, consulta `AGENTS.md` para conocer los comandos exactos del repositorio:

- **Comando Estándar (`TEST_COMMAND`)**: Debe ser de ejecución única, no interactivo y con terminación determinista (ej. `npx vitest run`, `pytest`, `cargo test`, `npm test -- --watchAll=false`). Queda estrictamente prohibido el modo *watch* interactivo que congele la consola.
- **Prueba Enfocada / Unitaria**:
  ```bash
  # Ejemplo según runner del proyecto:
  npx vitest run tests/orders.test.ts
  pytest tests/test_orders.py -k "test_calculate_total"
  ```
- **Línea Base (*Baseline Check*)**: Antes de escribir código para una tarea, ejecuta la suite. Si la suite ya falla antes de comenzar (*failing baseline*), **DETENTE**. No construyas código sobre una base rota.

---

## 3. El Ciclo Red-Green-Refactor

```text
       ROJO (RED)                     VERDE (GREEN)                  REFACTORIZAR
   Escribe un test que           Escribe el código mínimo         Limpia y optimiza la
   falle por la razón     ──►    necesario para que       ──►     implementación sin   ──► (repetir)
   correcta                      la prueba pase                   romper las pruebas
        │                              │                                  │
        ▼                              ▼                                  ▼
   El test FALLA                  El test PASA                     Los tests siguen
   (Error esperado)               (Verificación)                   PASANDO (Verde)
```

### Paso 1: ROJO (RED) — Escribir un Test que Falle
1. Ubica o crea la prueba en la carpeta de tests del proyecto (ej. `tests/`).
2. Modela el comportamiento esperado desde la perspectiva del consumidor:
   - ¿Qué entradas recibe la función, endpoint o servicio?
   - ¿Qué invariantes de persistencia o estado deben modificarse?
   - ¿Qué código de respuesta o error debe emitirse ante entradas inválidas?
3. Ejecuta la prueba enfocada.
4. **Verificación Crítica:** La prueba DEBE fallar, y debe fallar por la razón esperada (ejemplo: `Method not implemented` o `expected 400 received 404`), no por errores de sintaxis o problemas de configuración del entorno.

### Paso 2: VERDE (GREEN) — Implementación Mínima
1. Escribe la menor cantidad de código posible en el servicio o módulo para satisfacer la prueba.
2. Evita diseñar abstracciones prematuras en este paso; el objetivo primordial es poner la barra en verde.
3. Vuelve a ejecutar la prueba y confirma que pasa al 100%.

### Paso 3: REFACTORIZAR — Limpieza y Estándares
1. Con la red de seguridad en verde, optimiza la legibilidad y estructura del código:
   - Extrae funciones puras o utilitarios reutilizables.
   - Aplica esquemas de validación rigurosos.
   - Elimina código duplicado.
   - Aplica la skill `code-simplification` si el flujo acumula complejidad innecesaria.
2. Comprueba que todos los tests continúen pasando sin modificaciones.
3. Ejecuta `LINT_COMMAND` o `TYPECHECK_COMMAND` si están definidos en `AGENTS.md`.

---

## 4. Patrones Clave de Pruebas

### A. Patrón Prove-It (Reproducción de Bugs)
Jamás intentes corregir un bug sin reproducirlo primero con una prueba:
```typescript
it('debe excluir órdenes canceladas del cálculo de balance pendiente', async () => {
  // 1. Arrange: Crear cliente y órdenes (una completada y una cancelada)
  const customer = await createTestCustomer();
  await createTestOrder({ customerId: customer.id, total: 500, status: 'DELIVERED' });
  await createTestOrder({ customerId: customer.id, total: 300, status: 'CANCELLED' });

  // 2. Act: Consultar balance pendiente
  const balance = await customerService.calculateBalance(customer.id);

  // 3. Assert: El balance debe ser exactamente 500, ignorando los 300 cancelados
  expect(balance).toBe(500);
});
```

### B. Pruebas de Atomicidad e Integridad Transaccional
Cuando una operación altere múltiples entidades (ej. stock, órdenes y pagos), valida que ante un fallo no queden estados corruptos:
```typescript
it('debe revertir inventario si la creación del pago falla', async () => {
  const initialStock = await getInventoryStock(itemId);

  await expect(
    orderService.processOrderWithPayment({ ...invalidPaymentPayload })
  ).rejects.toThrow();

  const finalStock = await getInventoryStock(itemId);
  expect(finalStock).toBe(initialStock); // Cero impacto colateral
});
```

---

## 5. Antipatrones a Evitar (Trampas de TDD)

| Antipatrón | Por Qué es Peligroso | Corrección Obligatoria |
| :--- | :--- | :--- |
| **Escribir código antes del test** | Sesga la prueba para que coincida con lo implementado, no con lo especificado. | Borra o comenta el código nuevo; escribe el test primero y míralo fallar en rojo. |
| **Asertos vacíos o tautológicos** | `expect(res.body).toBeDefined()` pasa incluso si la lógica retorna un objeto erróneo. | Evalúa valores específicos exactos: `expect(res.body.currentStock).toBe(12)`. |
| **Ignorar fallas colaterales** | "Mi test pasa, pero fallaron 2 en otro archivo". | Detén la línea. Una regresión en otra suite indica acoplamiento o violación de invariantes. |
| **Mocks excesivos que falsean la realidad** | Mockear todo oculta errores de tipos, sintaxis SQL y restricciones relacionales. | Usa bases de datos o mocks de integración fieles para validar contratos reales. |

---

## 6. Verificación de Cierre de Tarea
Antes de dar la tarea por finalizada y realizar el commit atómico quirúrgico:
- [ ] La nueva prueba falló en rojo antes de la implementación.
- [ ] La nueva prueba pasa en verde de forma determinista y reproducible.
- [ ] La suite de pruebas pasa completa sin regresiones (`TEST_COMMAND`).
- [ ] La verificación de tipos y linter aprueba sin nuevos errores (`TYPECHECK_COMMAND`).

---
name: incremental-implementation
description: Entrega cambios de forma incremental en rebanadas verticales delgadas y verificables. Usar al implementar features multifile, refactorizaciones o tareas de una spec.
category: architecture
---

# Incremental Implementation — Guía de Implementación Incremental

Construye mediante rebanadas verticales delgadas (*thin vertical slices*): implementa una pieza mínima, pruébala, verifícala y solo entonces continúa con la siguiente. Queda terminantemente prohibido implementar una característica completa de extremo a extremo en una sola pasada masiva. Cada incremento debe dejar el sistema en un estado funcional, compilable y testeable.

---

## 1. Cuándo Usar la Implementación Incremental

- Al abordar cualquier tarea (`Tn`) resultante de una especificación SDD (`specs/NNN-nombre/tasks.md`).
- Al realizar cambios que afecten a más de un archivo o capa simultáneamente.
- En refactorizaciones de módulos o servicios existentes.
- Cada vez que sientas la tentación de escribir más de 80–100 líneas de código antes de ejecutar una prueba o compilar.

> **Cuándo NO usar:** Cambios atómicos de una sola función en un único archivo donde el alcance ya es microscópico y autocontenido.

---

## 2. El Ciclo del Incremento

```text
┌────────────────────────────────────────────────────────┐
│                                                        │
│   Implementar ──► Probar (TEST_CMD) ──► Verificar (Lint/Type)
│        ▲                                       │       │
│        │                                       ▼       │
│        └────── Siguiente Rebanada ◄─────── Commit      │
│                (Llevar adelante)          Atómico      │
│                                                        │
└────────────────────────────────────────────────────────┘
```

Para cada rebanada delgada:
1. **Implementar:** Escribe el cambio más pequeño posible que agregue valor comprobable.
2. **Probar:** Ejecuta las pruebas del componente o módulo (`TEST_COMMAND`).
3. **Verificar:** Confirma que el tipado y linter se mantengan limpios (`TYPECHECK_COMMAND`).
4. **Commit Atómico:** Guarda el avance con un commit semántico en español siguiendo la convención (`<tipo>(<NNN>): <Tn> - <descripción>`).
5. **Avanzar:** Pasa a la siguiente rebanada sin reiniciar el contexto ni dejar cabos sueltos.

---

## 3. Estrategia de Rebanadas Verticales

La estrategia más robusta para evitar ramas rotas es construir un camino vertical completo a través de las capas del sistema:

```text
Paso 1: Esquema de Validación y Contrato de Entrada (schemas / DTOs)
   │
   ▼
Paso 2: Lógica de Servicio de Dominio y Persistencia (services / models)
   │
   ▼
Paso 3: Controlador Delgado y Rutas de Transporte (controllers / routes)
   │
   ▼
Paso 4: Pruebas de Integración Automatizadas (tests)
   │
   ▼
Paso 5: Vista de Usuario o Componente de Interfaz (components / views)
```

### Ejemplo Práctico: Implementar un Ajuste de Inventario

- **Rebanada 1 (Contrato de Validación):**
  Definir el esquema de validación para la entrada del ajuste (identificador de ítem, cantidad, motivo).
  *Verificación:* Pruebas unitarias del esquema validando casos correctos e incorrectos.

- **Rebanada 2 (Servicio y Transacción de Base de Datos):**
  Implementar el servicio que ejecuta la transacción atómica: actualiza el stock y crea el registro de auditoría.
  *Verificación:* Prueba de integración en la base de datos confirmando atomicidad.

- **Rebanada 3 (Controlador y Ruta HTTP):**
  Exponer el endpoint `POST /api/inventory/adjustments` validando el body con el middleware correspondiente.
  *Verificación:* Prueba HTTP confirmando respuesta HTTP 201 y código 400 ante payloads inválidos.

- **Rebanada 4 (Integración en Interfaz de Usuario):**
  Conectar el modal o formulario en el frontend consumiendo la API mediante el cliente centralizado.
  *Verificación:* Compilación y empaquetado limpio (`BUILD_COMMAND`).

---

## 4. Reglas Innegociables de Alcance y Disciplina

1. **Cirugía de Precisión:** Modifica exclusivamente los archivos necesarios para la tarea actual. Prohibido "aprovechar" para refactorizar módulos ajenos o borrar comentarios que no comprendas.
2. **Cero Código Huérfano:** No introduzcas banderas booleanas, interfaces o endpoints "por si acaso se necesitan mañana". Si no está en la spec actual, no se implementa.
3. **Preservación del Build Verde:** Si un incremento rompe una suite existente, no continúes hacia la siguiente rebanada. Detén el trabajo y aplica `debugging-and-error-recovery`.
4. **Sincronización de Memoria:** Al finalizar cada rebanada o tarea clave, actualiza el progreso en `tasks.md` y `MEMORY.md`.

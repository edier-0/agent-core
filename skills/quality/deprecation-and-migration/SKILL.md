---
name: deprecation-and-migration
description: Gestiona la deprecación, retiro seguro de código legacy y migraciones de esquema de base de datos sin tiempo de inactividad (Zero-Downtime y Expand & Contract).
---

# Deprecation & Migration — Guía de Migración Segura

El código es un pasivo de mantenimiento, no un activo. Cada línea de código acarrea costos continuos de soporte, seguridad y carga cognitiva. Esta skill guía la deprecación controlada y la migración segura de datos, APIs y componentes sin provocar caídas de servicio (*zero-downtime*) ni romper a los clientes que consumen el sistema.

---

## 1. El Patrón Expandir y Contraer (*Expand & Contract*) en Base de Datos

Al modificar esquemas relacionales o colecciones con datos existentes, **queda terminantemente prohibido renombrar o eliminar columnas/campos activos en un solo paso destructivo**. Aplica el ciclo en 3 fases:

```text
FASE 1: EXPANDIR          FASE 2: TRANSICIÓN           FASE 3: CONTRAER
┌─────────────────┐       ┌─────────────────┐          ┌─────────────────┐
│ Columna Antigua │       │ Columna Antigua ──(Migrar) │                 │
│        +        │  ──►  │        +         Datos     │  ──►            │
│ Columna Nueva   │       │ Columna Nueva   ◄───────── │ Columna Nueva   │
│   (Opcional)    │       │ (Doble Escritura)          │  (Definitiva)   │
└─────────────────┘       └─────────────────┘          └─────────────────┘
```

1. **Fase 1 (Expandir):** Agrega el nuevo campo como opcional (o con valor por defecto seguro) en el esquema de base de datos. Aplica la migración sin tocar ni bloquear el campo antiguo.
2. **Fase 2 (Transición):** Actualiza la capa de servicio para escribir en ambos campos (*doble escritura*) y leer preferentemente del nuevo. Ejecuta un script seguro de backfill para migrar los registros históricos de forma gradual.
3. **Fase 3 (Contraer):** Una vez verificado que el 100% del tráfico y consultas utilizan el nuevo campo, retira el campo antiguo del esquema y ejecuta la poda en la base de datos sin riesgo.

---

## 2. Gestión de APIs y Rutas Legacy

Cuando conviven versiones anteriores de una API o restos de arquitecturas previas con la capa modular canónica:

### Reglas Innegociables:
- **Prohibido agregar nuevas features en rutas o carpetas legacy:** Todo nuevo código pertenece exclusivamente a la arquitectura modular canónica.
- **Preservación de Alias durante Transición:** Conservar compatibilidad con rutas legadas que consumidores externos o frontends antiguos aún utilicen.
- **Migración Progresiva:** Para migrar un endpoint legacy:
  1. Implementa y prueba la lógica en la capa canónica.
  2. Haz que la ruta legacy delegue como alias transparente al nuevo controlador o servicio.
  3. Actualiza los clientes internos para apuntar al nuevo endpoint.
  4. Retira la ruta legacy solo tras verificar la ausencia total de tráfico en registros de acceso.

---

## 3. Checklist de Deprecación Segura
- [ ] La alternativa de reemplazo está 100% implementada y probada antes de marcar algo como deprecado.
- [ ] No existen dependencias huérfanas en el frontend o servicios externos llamando al recurso a retirar.
- [ ] Las migraciones de base de datos se aplicaron sin bloqueos de tabla destructivos.
- [ ] Se documentó la decisión y cronograma de retiro en `MEMORY.md` o en un ADR.

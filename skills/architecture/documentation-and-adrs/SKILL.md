---
name: documentation-and-adrs
description: Registra decisiones de arquitectura (ADRs) y documentación técnica de soporte. Usar al tomar decisiones estructurales, elegir librerías o cambiar contratos de API.
---

# Documentation & ADRs — Guía de Registro de Decisiones de Arquitectura

Documenta las decisiones y sus motivos, no solo el código resultante. La documentación más valiosa explica el **por qué** (*why*) —el contexto de negocio, las restricciones operativas y las alternativas descartadas—. El código muestra qué se construyó; la documentación explica por qué se construyó de esa forma y previene que futuros desarrolladores o agentes de IA reviertan decisiones intencionales por desconocimiento.

---

## 1. Dónde Residen las Decisiones en el Repositorio

En nuestro proyecto, las decisiones se documentan según su nivel de permanencia e impacto:

| Tipo de Decisión | Destino Canónico | Propósito |
| :--- | :--- | :--- |
| **Principios e Invariantes Supremos** | `docs/constitution.md` | Invariantes innegociables de gobernanza, seguridad y arquitectura (máx 6 principios). |
| **Decisiones Operativas entre Sesiones** | `MEMORY.md` | Estado activo, decisiones tomadas, aprendizajes y errores a evitar (máx ~50 líneas). |
| **Decisiones Técnicas de una Feature** | `specs/NNN-nombre/plan.md` | Justificación técnica de componentes, contratos de datos y alternativas descartadas. |
| **Decisiones de Arquitectura Mayores (ADR)** | `docs/adr/NNN-titulo.md` | Cambios estructurales mayores, adopción de frameworks o migraciones de plataforma. |

---

## 2. Estructura Canónica de un ADR (`docs/adr/NNN-titulo.md`)

```markdown
# ADR-NNN: [Título Corto y Descriptivo]

- **Estado:** Propuesto | Aceptado | Deprecado | Reemplazado por ADR-XXX
- **Fecha:** AAAA-MM-DD
- **Autores:** [Nombres o Roles]

## Contexto y Planteamiento del Problema
¿Qué situación operativa, de escala o técnica motivó esta decisión? ¿Cuáles eran las restricciones del entorno o del negocio?

## Alternativas Consideradas
1. **Alternativa A (Descartada):** Descripción y por qué se consideró inviable o costosa.
2. **Alternativa B (Elegida):** Descripción del enfoque seleccionado.

## Decisión Adoptada
Explicación concisa del cambio y de la solución técnica acordada.

## Consecuencias y Trade-offs
- **Positivas:** Qué beneficios inmediatos y a largo plazo se obtienen.
- **Negativas / Compromisos:** Qué complejidad adicional se asume y cómo se mitigará.
```

---

## 3. Reglas de Documentación

1. **No documentes lo evidente:** Prohibido agregar comentarios que solo repitan la sintaxis del código (`// incrementa el contador`).
2. **Documenta el Por Qué:** Todo comentario de código debe justificar la razón detrás de una decisión no obvia o caso límite.
3. **Poda Activa de Memoria:** Al actualizar `MEMORY.md`, resume o elimina lo que ya no aporte valor operativo inmediato; si algo se convierte en regla permanente, propón moverlo a `AGENTS.md` o a la constitución.

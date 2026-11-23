---
name: idea-refine
description: Refina ideas preliminares en conceptos claros y accionables mediante pensamiento divergente y convergente. Usar cuando una idea sea imprecisa o se quieran explorar alternativas antes de redactar una spec SDD.
category: methodology
---

# Idea Refine — Guía de Refinamiento de Conceptos

Transforma ideas incipientes en conceptos técnicos y operativos sólidos mediante pensamiento divergente (expandir opciones) y convergente (evaluar trade-offs, descartar y focalizar). Esta skill asegura que antes de redactar especificaciones formales en `sdd`, se haya elegido la solución más simple, elegante y de mayor impacto para el proyecto.

---

## 1. Cuándo Aplicar Esta Skill

- Cuando se tiene una idea general de negocio pero sin arquitectura definida.
- Cuando existen múltiples formas de resolver un problema y se requiere evaluar los trade-offs antes de comprometerse con un plan.
- Al evaluar si una funcionalidad debe construirse internamente o resolverse mediante procesos operativos existentes o herramientas de terceros.
- Frases de activación del usuario: *"ayúdame a pensar esta idea"*, *"exploremos opciones para..."*, *"evalúa alternativas para..."*, *"stress-test my plan"*.

---

## 2. El Proceso de Refinamiento en 3 Fases

```text
1. EXPANDIR (Divergente)  ──► Explorar 3 variaciones o enfoques radicalmente distintos
2. EVALUAR (Convergente)  ──► Contrastar contra docs/constitution.md y relación costo/beneficio
3. FOCALIZAR (Sharpen)    ──► Producir un documento de 1 página con el alcance exacto
```

### Fase 1: Expandir (Pensamiento Divergente)
Presenta al menos 3 enfoques posibles:
1. **La Solución Mínima (MVP Pragmático):** ¿Cómo resolver el 80% del valor con el 20% del esfuerzo sin introducir esquemas complejos?
2. **La Solución Canónica y Escalable:** Integración completa en la arquitectura modular del backend y persistencia relacional/documental.
3. **La Solución Operativa o Basada en Herramientas Existentes:** ¿Se puede resolver reutilizando servicios ya disponibles sin agregar deuda técnica nueva?

### Fase 2: Evaluar y Estresar Suposiciones
Filtra las opciones aplicando los principios innegociables del proyecto:
- ¿Viola algún principio de `docs/constitution.md` (seguridad, integridad de datos, acoplamiento)?
- ¿Añade complejidad accidental innecesaria para el equipo de desarrollo?
- ¿Resuelve el problema real del usuario final en producción o es solo un adorno estético?

### Fase 3: Documento de Síntesis (One-Pager)
El resultado del refinamiento es una propuesta ejecutiva que resume:
- **Definición del Problema:** Qué dolor o necesidad real resuelve.
- **Dirección Recomendada:** Enfoque seleccionado y por qué se descartaron las otras alternativas.
- **Alcance MVP (Qué entra):** Requisitos nucleares indispensables.
- **Fuera de Alcance (Qué NO entra explícitamente):** Blindaje contra inflación de alcance (*scope creep*).

---

## 3. Transición hacia SDD
Una vez que el usuario aprueba la dirección refinada, la salida de esta skill alimenta directamente el workflow `/sdd-spec` para formalizar los requerimientos en `specs/NNN-nombre/spec.md`.

---
description: Inicializa o actualiza el archivo AGENTS.md mediante auditoría (proyecto existente) o entrevista guiada (proyecto nuevo), validando un borrador previo en español.
---

# Workflow: Inicialización de AGENTS.md

Sigue estrictamente las siguientes fases sin saltarte la validación del usuario.

---

### Fase 1: Identificación del contexto
1. Comprueba si el directorio actual contiene código fuente, ficheros de dependencias (`package.json`, `composer.json`, `Dockerfile`, `.sln`, etc.) o commits previos.
2. Pregunta o confirma con el usuario:
   - **Opción A:** Proyecto existente (auditar código actual).
   - **Opción B:** Proyecto desde cero (definir stack y arquitectura desde cero).

---

### Fase 2: Recolección y Análisis

#### Si es Proyecto Existente:
- Inspecciona los archivos clave del repositorio (árbol de directorios, manifiestos de dependencias, linters, scripts de test, docker-compose).
- Formula un máximo de 3 a 5 preguntas concretas sobre lo que **no se puede deducir del código**:
  - ¿Cuáles son las reglas de dominio o trampas críticas que un agente suele romper aquí?
  - ¿Qué acciones están estrictamente prohibidas tocar o cambiar (guardrails)?
  - ¿Qué comandos exactos de verificación consideras indispensables?

#### Si es Proyecto Desde Cero:
- Entrevista al usuario brevemente sobre:
  - Objetivo principal del proyecto y audiencia.
  - Stack técnico definido (lenguajes, frameworks, bases de datos).
  - Arquitectura elegida (modular monolith, microservicios, capas, etc.).
  - Restricciones clave (librerías prohibidas, estilos obligatorios, reglas de testing).

---

### Fase 3: Borrador de Validación (Obligatorio)
**NO crees ni modifiques `AGENTS.md` todavía.**

Presenta en el chat una propuesta completa con la siguiente estructura, redactada en español:

1. **Resumen ejecutivo:** Explicación clara de qué ha detectado o interpretado el agente sobre el proyecto.
2. **Propuesta de `AGENTS.md` (formato final):**
   - Debe cumplir la regla de síntesis (~35 a 45 líneas).
   - Estructura:
     - `# AGENTS.md — [Nombre del Proyecto]`
     - `## Stack y estructura`
     - `## Comandos` (ejecutar, test, lint, build)
     - `## Convenciones` (estilo, idioma, enlace a `.agents/rules/` si aplica)
     - `## Reglas de dominio / trampas conocidas`
     - `## Forma de trabajar` (planificación previa, tamaño de PR/commits)
     - `## Límites` (✅ Siempre / ⚠️ Preguntar / 🚫 Nunca)
     - `## Verificación`
     - `## Referencias` (enlace previsto a `MEMORY.md` y carpeta de specs/SDD)
3. **Pregunta de confirmación:** Solicita aprobación explícita o ajustes antes de persistir los cambios en disco.

---

### Fase 4: Generación
Una vez que el usuario apruebe explícitamente el borrador:
1. Escribe el archivo `AGENTS.md` en la raíz del repositorio.
2. Si el proyecto requirió desglosar convenciones extensas, crea `.agents/rules/conventions.md` y enlázalo desde `AGENTS.md`.
3. Confirma la creación y resume los próximos pasos para continuar con `MEMORY.md`.
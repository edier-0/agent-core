---
name: using-agent-skills
description: Enrutador y meta-skill maestro de agent-core. Gobierna cómo los agentes descubren, secuencian y aplican las 27 skills del proyecto organizadas en 6 categorías según la fase del ciclo de vida de desarrollo.
category: router
---

# Using Agent Skills — Enrutador Maestro del Ecosistema

Colección orquestada de flujos de trabajo de ingeniería organizados por fases de desarrollo. Cada skill codifica un proceso riguroso que siguen los ingenieros de software senior y DevOps. Esta meta-skill guía a los agentes de IA para descubrir, secuenciar y aplicar la habilidad adecuada para cada tarea, evitando la improvisación y garantizando el cumplimiento de `docs/constitution.md` y `AGENTS.md`.

---

## 1. Árbol de Decisión y Descubrimiento de Skills por Categoría

Cuando llegue una solicitud o tarea, identifica la fase de desarrollo y activa la skill correspondiente dentro de su categoría canónica:

```text
Llega una Tarea o Requerimiento
    │
    ├── 1. METHODOLOGY (Definición, Requerimientos y Gobernanza)
    │   ├── ¿La petición es ambigua o falta contexto de negocio? ──► interview-me
    │   ├── ¿Idea incipiente y necesitas explorar alternativas? ────► idea-refine
    │   ├── ¿Nueva funcionalidad, cambio o refactor mayor? ─────────► sdd (Spec-Driven Development)
    │   ├── ¿Falta fijar la barra de calidad o reglas técnicas? ────► constraint-driven-development
    │   ├── ¿Decisiones de alto riesgo, financieras o críticas? ────► doubt-driven-development
    │   ├── ¿Tienes la spec aprobada y requieres tareas? ───────────► planning-and-task-breakdown
    │   ├── ¿Nueva lógica, cambio de regla o bugfix? ───────────────► test-driven-development
    │   └── ¿Validar sintaxis contra versión en dependencias? ──────► source-driven-development
    │
    ├── 2. ARCHITECTURE (Diseño Estructural y Contratos)
    │   ├── ¿Endpoints REST, esquemas de validación o contratos? ──► api-and-interface-design
    │   ├── ¿Estrategia de entrega en rebanadas delgadas? ──────────► incremental-implementation
    │   ├── ¿Cargar contexto y afinar archivos relevantes? ─────────► context-engineering
    │   ├── ¿Registrar el porqué de una decisión técnica? ──────────► documentation-and-adrs
    │   ├── ¿Instrumentar logs estructurados o auditoría? ──────────► observability-and-instrumentation
    │   └── ¿Problemas de lentitud, queries N+1 o bundle? ──────────► performance-optimization
    │
    ├── 3. FRONTEND (Interfaz de Usuario y Experiencia)
    │   ├── ¿Diseño de UI, vistas, modales y accesibilidad? ───────► frontend-ui-engineering
    │   ├── ¿Catálogo de estilos, paletas o gráficos UI/UX? ───────► ui-ux-pro-max
    │   └── ¿Inspección del DOM, consola o viewport móvil? ────────► browser-testing-with-devtools
    │
    ├── 4. QUALITY (Revisión, Calidad y Depuración)
    │   ├── ¿Auditoría multiaxial antes de integrar? ──────────────► code-review-and-quality
    │   ├── ¿Código denso o con anidamientos excesivos? ───────────► code-simplification
    │   ├── ¿Falló un test, build roto o bug en runtime? ───────────► debugging-and-error-recovery
    │   └── ¿Retiro de código legacy o migración de esquema? ───────► deprecation-and-migration
    │
    └── 5. DELIVERY (Entrega Continua, Seguridad y Git)
        ├── ¿Ramas dedicadas, commits y ciclo de PR remoto? ───────► git-pr-workflow
        ├── ¿Secretos, cero fallbacks y auditoría de paquetes? ─────► devsecops
        ├── ¿Contenedores multi-stage, BuildKit y hardening? ───────► docker-hardening
        ├── ¿Compuertas de CI/CD en pipelines automatizados? ───────► ci-cd-and-automation
        └── ¿Checklist pre-lanzamiento, rollback y despliegue? ─────► shipping-and-launch
```

---

## 2. Comportamientos Operativos No Negociables

Estos principios aplican en todo momento para cualquier agente en el proyecto:

### 1. Hacer Explícitas las Suposiciones (*Surface Assumptions*)
Antes de implementar cualquier tarea no trivial, explicita tus asunciones:
```text
SUPOSICIONES QUE ESTOY ASUMIENDO:
1. [Suposición sobre reglas de negocio]
2. [Suposición sobre arquitectura o base de datos]
3. [Suposición sobre alcance]
→ Si alguna es incorrecta, corrígeme antes de proceder.
```
Rellenar silenciosamente requisitos ambiguos es la causa principal de retrabajo.

### 2. Gestionar la Confusión Activamente
Si encuentras contradicciones entre la spec, la Constitución o el código existente:
1. **DETENTE.** No avances conjeturando.
2. Identifica con exactitud la discrepancia.
3. Plantea la pregunta de clarificación al usuario y espera su respuesta.

### 3. Cuestionar cuando Sea Necesario (*Push Back*)
No seas una máquina complaciente que dice *"¡Por supuesto!"* a todo. Si un enfoque solicitado introduce vulnerabilidades de seguridad, rompe contratos críticos o viola invariantes constitucionales:
- Señala el riesgo técnico de forma directa.
- Cuantifica el impacto negativo.
- Propón una alternativa sólida alineada con `docs/constitution.md`.

### 4. Imponer Simplicidad (*Enforce Simplicity*)
La tendencia natural es sobrecomplicar. Resístela activamente:
- ¿Se puede resolver en menos líneas de código?
- ¿Estas abstracciones justifican su costo de mantenimiento?
- Prefiere la solución obvia, robusta y testeable.

### 5. Disciplina Quirúrgica de Alcance
Modifica exclusivamente los archivos requeridos para la tarea en curso. **Prohibido:**
- Borrar comentarios que no entiendas.
- "Limpiar" o refactorizar archivos no relacionados con la tarea activa.
- Agregar funcionalidades no solicitadas en la spec porque "parecen útiles".

### 6. Verificar con Pruebas, Jamás Asumir
Una tarea no está terminada hasta que la verificación automatizada aprueba. "Parece que funciona" no es evidencia; se requieren pruebas pasando en verde (`TEST_COMMAND`), compilación limpia (`TYPECHECK_COMMAND`) y empaquetado exitoso.

### 7. Estándar de Consultas y Encuestas: Tarjetas de Selección con Opción Libre
Toda encuesta, entrevista de requerimientos, clarificación de spec o pregunta técnica dirigida al usuario debe estructurarse obligatoriamente bajo el **formato de tarjetas de selección**:
- **En entornos con herramienta interactiva nativa (ej. `ask_question`)**: Usar obligatoriamente la herramienta interactiva de preguntas. Proporcionar las opciones sin numerar, la recomendada al inicio prefijada con `(Recommended)`, permitiendo que la interfaz renderice tarjetas interactivas y la opción automática para escribir otra respuesta libre (*"Other"*).
- **En entornos de chat / Markdown plano**: Formatear como **Tarjetas de Opciones Visuales**:
  > **[Pregunta concreta en una sola línea]**
  >
  > - **[A] (Recomendada)**: [Opción preferente según arquitectura y mejores prácticas]
  > - **[B]**: [Opción alternativa 1]
  > - **[C]**: [Opción alternativa 2 (si aplica)]
  > - **[O] Escribir otra respuesta**: [Permite al usuario redactar una alternativa libre personalizada]
- **Prohibición**: Queda terminantemente prohibido formular preguntas abiertas difusas, párrafos densos o listados sin opciones predefinidas que fuercen al usuario a escribir respuestas largas desde cero.

---

## 3. Secuencia Típica del Ciclo de Vida de una Característica

Para el desarrollo completo de una nueva funcionalidad en el proyecto, el orden canónico de invocación es:

```text
1.  interview-me                → Clarificar la necesidad real de negocio
2.  idea-refine                 → Evaluar alternativas y definir el alcance inicial
3.  constraint-driven-development → Establecer la barra de calidad e invariantes
4.  sdd                         → Redactar la especificación en EARS (specs/NNN/spec.md)
5.  planning-and-task-breakdown → Diseñar plan.md y descomponer en tareas tasks.md
6.  context-engineering         → Cargar el contexto quirúrgico del módulo
7.  source-driven-development   → Validar contratos contra las dependencias oficiales
8.  incremental-implementation  → Construir rebanada vertical por rebanada
9.  ui-ux-pro-max               → Aplicar paletas y componentes visuales consistentes
10. frontend-ui-engineering     → Construir vistas, componentes y accesibilidad
11. api-and-interface-design    → Diseñar endpoints y esquemas de validación
12. observability-and-instrumentation → Instrumentar logs estructurados y trazabilidad
13. doubt-driven-development    → Someter decisiones críticas a examen adversarial
14. test-driven-development     → Ciclo Red-Green-Refactor con TEST_COMMAND
15. browser-testing-with-devtools → Validar el renderizado en navegador y viewport móvil
16. code-review-and-quality     → Auditoría multiaxial antes de integrar
17. code-simplification         → Purgar complejidad manteniendo comportamiento exacto
18. devsecops                   → Escaneo de secretos, cero fallbacks y auditoría de paquetes
19. git-pr-workflow             → Rama temática, Quality Gates y ciclo de PR remoto
20. docker-hardening            → Validar build de contenedor multi-stage
21. documentation-and-adrs      → Registrar el porqué de la solución en MEMORY.md o ADR
22. deprecation-and-migration   → Retirar endpoints o esquemas legacy de forma segura
23. shipping-and-launch         → Checklist final pre-lanzamiento y sincronización a main
```

---

## 4. Catálogo de Referencia de las 27 Skills por Categoría

Las 27 skills residen en el primer nivel de `skills/<nombre-skill>/SKILL.md` para compatibilidad nativa con Antigravity, OpenCode y Claude Code, y preservan su taxonomía semántica mediante el campo `category: <categoria>` en su frontmatter YAML:

### Categoría 1: Enrutamiento (`category: router`)
| Skill | Resumen Operativo de una Línea |
| :--- | :--- |
| `using-agent-skills` | Meta-skill y enrutador maestro para descubrir y secuenciar las habilidades del sistema. |

### Categoría 2: Metodología y Requerimientos (`category: methodology`)
| Skill | Resumen Operativo de una Línea |
| :--- | :--- |
| `interview-me` | Entrevista socrática de 1 pregunta a la vez hasta alcanzar ~95% de certeza en los requerimientos. |
| `idea-refine` | Expande y converge ideas incipientes en propuestas concretas y accionables. |
| `constraint-driven-development` | Establece y defiende invariantes técnicos y restricciones de calidad innegociables. |
| `sdd` | Spec-Driven Development: ciclo formal con sintaxis EARS, specs, planes y tareas atómicas. |
| `planning-and-task-breakdown` | Descompone specs aprobadas en `plan.md` y tareas atómicas de 20–30 min. |
| `test-driven-development` | TDD estricto: escribe el test que falla en rojo antes del código (Patrón Prove-It). |
| `source-driven-development` | Fundamenta decisiones en documentación oficial y versiones reales de librerías. |
| `doubt-driven-development` | Examen adversarial de decisiones de alto riesgo, persistencia o lógica crítica. |

### Categoría 3: Arquitectura y Diseño (`category: architecture`)
| Skill | Resumen Operativo de una Línea |
| :--- | :--- |
| `api-and-interface-design` | Endpoints REST, controladores delgados y esquemas de validación de datos. |
| `incremental-implementation` | Construye en rebanadas verticales delgadas, independientes y verificables. |
| `context-engineering` | Carga el contexto preciso en el momento exacto, evitando saturación de tokens. |
| `documentation-and-adrs` | Registra el *porqué* de las decisiones de diseño en ADRs y `MEMORY.md`. |
| `observability-and-instrumentation` | Instrumenta logs estructurados JSON, correlación y métricas de diagnóstico. |
| `performance-optimization` | Erradicación de consultas N+1, optimización de índices, bundle y consultas pesadas. |

### Categoría 4: Calidad y Mantenibilidad (`category: quality`)
| Skill | Resumen Operativo de una Línea |
| :--- | :--- |
| `code-review-and-quality` | Auditoría multiaxial antes de integrar (corrección, arquitectura, seguridad, rendimiento). |
| `code-simplification` | Reduce carga cognitiva y complejidad innecesaria preservando comportamiento exacto. |
| `debugging-and-error-recovery` | Triaje sistemático de causa raíz en 5 pasos y protocolo de Parar la Línea. |
| `deprecation-and-migration` | Retiro seguro de código legacy y migraciones de base de datos sin downtime (Expand/Contract). |

### Categoría 5: Entrega, DevOps y Seguridad (`category: delivery`)
| Skill | Resumen Operativo de una Línea |
| :--- | :--- |
| `git-pr-workflow` | Ramas temáticas, compuertas de calidad, Pull Requests remotos y eliminación autorizada. |
| `devsecops` | Prohibición total de fallbacks de credenciales, escaneo de secretos y auditoría de paquetes. |
| `docker-hardening` | Contenedores seguros multi-stage, optimización de caché, usuarios no-root y Alpine. |
| `ci-cd-and-automation` | Pipelines de integración continua automatizados y compuertas de calidad pre-merge. |
| `shipping-and-launch` | Listas de verificación pre-lanzamiento, monitoreo post-despliegue y planes de rollback. |

### Categoría 6: Frontend y Experiencia de Usuario (`category: frontend`)
| Skill | Resumen Operativo de una Línea |
| :--- | :--- |
| `frontend-ui-engineering` | Interfaces profesionales, accesibilidad WCAG AA, responsive design y gestión de estado. |
| `ui-ux-pro-max` | Catálogo de diseño con estilos, paletas, tipografías, gráficos y directrices UX. |
| `browser-testing-with-devtools` | Verificación en vivo de UI, DOM, consola y viewport con Chrome DevTools MCP. |

---

## 5. Descubrimiento de Capacidades Externas Bajo Demanda (`find-skills`)

Para evitar saturar el repositorio instalando paquetes prescindibles, cualquier requerimiento especializado futuro se consulta al vuelo mediante el CLI de `skills.sh`:

1. **Búsqueda interactiva:** `npx skills find <término>` (ej. `npx skills find redis`, `npx skills find playwright`).
2. **Evaluación de madurez:** Solo considerar skills con **más de 1.000 instalaciones** y de fuentes oficiales/reconocidas.
3. **Uso efímero:** `npx skills use <paquete>@<skill>` para ejecuciones puntuales sin alterar la estructura local de archivos.

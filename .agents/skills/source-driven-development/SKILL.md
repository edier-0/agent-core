---
name: source-driven-development
description: Fundamenta las decisiones de implementación en la documentación oficial y versiones exactas de las librerías instaladas. Usar al diseñar APIs, configurar herramientas o adoptar librerías.
category: methodology
---

# Source-Driven Development — Guía Canónica

Fundamenta cada decisión de diseño y código en la documentación oficial y en las versiones exactas instaladas en el repositorio. Los datos de entrenamiento de los modelos de IA se desfasan, las APIs cambian y las mejores prácticas evolucionan. Esta skill garantiza que el código sea fidedigno, moderno y libre de patrones obsoletos o deprecados.

---

## 1. Cuándo Aplicar Esta Skill

- Al utilizar librerías centrales del stack donde la versión específica altera el comportamiento (ej. versiones mayores de ORMs, frameworks web, tipados estrictos o gestores de estado).
- Al configurar o modificar herramientas de build, empaquetado y runners (Vite, Webpack, Docker, Vitest, Pytest, Cargo).
- Al implementar patrones arquitectónicos o plantillas que se replicarán en otros módulos del sistema.
- Cuando una API arroja errores de tipado o firmas inesperadas.

> **Cuándo NO usar:** Lógica pura de programación o algoritmos estándar que operan idéntico en cualquier versión (bucles, estructuras de datos nativas, comparaciones matemáticas).

---

## 2. El Proceso en 4 Pasos

```text
1. DETECTAR   ──► Identificar versiones exactas en el manifiesto de dependencias
2. CONSULTAR  ──► Verificar sintaxis y contratos en la documentación oficial o tipos (.d.ts)
3. IMPLEMENTAR──► Aplicar el patrón recomendado para esa versión exacta
4. CITAR      ──► Justificar la decisión técnica en el código, plan o spec
```

### Paso 1: Detección de Versiones Reales
Antes de escribir código conjetural, inspecciona los manifiestos de dependencias reales del proyecto:
- Node.js / TypeScript: `package.json` (resolución de módulos, ESM vs CJS, versiones de dependencias).
- Python: `pyproject.toml`, `requirements.txt` o `Pipfile`.
- Rust / Go / Java: `Cargo.toml`, `go.mod`, `pom.xml` o `build.gradle`.

### Paso 2: Validación contra Documentación Oficial
- **Prohibido adivinar APIs:** Si una función o método de una librería de terceros genera dudas, consulta la documentación oficial o los archivos de definición de tipos (`.d.ts`, stubs o firmas de función).
- **Convenciones de Resolución de Módulos:** Respeta estrictamente la resolución configurada (ej. `NodeNext` en TypeScript exige sufijos `.js` en imports relativos ESM).

### Paso 3: Aplicación de Patrones Canónicos
- Prefiere siempre los patrones idiomáticos recomendados por la versión instalada (ej. Composition API moderna en Vue 3, Hooks funcionales en React, modelos Pydantic v2 en FastAPI, etc.).
- Prohibido reintroducir patrones legados o deprecados procedentes de ejemplos antiguos de internet.

---

## 3. Checklist de Verificación
- [ ] La sintaxis empleada corresponde fielmente a la versión instalada en el manifiesto de dependencias.
- [ ] No se utilizaron APIs marcadas como deprecadas en la versión activa.
- [ ] Se evitaron parches provisionales que contradigan los manuales oficiales de las librerías.
- [ ] Si se adoptó una API con comportamiento sensible, se citó la referencia en el código o en `MEMORY.md`.

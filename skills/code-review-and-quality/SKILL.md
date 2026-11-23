---
name: code-review-and-quality
description: Realiza revisión multidimensional de código y compuertas de calidad antes de cualquier merge o PR. Evalúa corrección, legibilidad, arquitectura, seguridad y rendimiento.
category: quality
---

# Code Review & Quality Gates — Guía Multidimensional de Revisión

Revisión exhaustiva multidimensional de código con compuertas de calidad obligatorias. Todo cambio debe ser auditado antes de solicitar la apertura de un Pull Request o autorizar su integración hacia la rama base.

> **El Estándar de Aprobación Senior:** Aprueba un cambio cuando definitivamente mejore la salud general, robustez y claridad del código, incluso si no es 100% idéntico a cómo lo habrías escrito tú. No bloquees por preferencias estéticas menores si el código cumple las convenciones del proyecto y no introduce deuda técnica.

---

## 1. Cuándo Aplicar Esta Skill

- Antes de abrir cualquier Pull Request hacia la rama base (como parte de los Quality Gates de `git-pr-workflow`).
- Al finalizar la implementación de una característica o tarea de una spec.
- Al evaluar código generado por subagentes o colaboradores externos.
- Al concluir una corrección de bugs (verificando tanto el fix como el test de regresión).
- En el workflow ejecutable `/sdd-validate`.

---

## 2. Los Cinco Ejes de la Revisión de Código

```text
                      REVISIÓN MULTIAXIAL
                               │
       ┌───────────────┬───────┴───────┬───────────────┐
       ▼               ▼               ▼               ▼
1. Corrección   2. Legibilidad   3. Arquitectura   4. Seguridad   5. Rendimiento
   y Dominio       y Estilo        y Capas           DevSecOps       y Base Datos
```

### Eje 1: Corrección Funcional e Invariantes de Dominio
- ¿El código cumple exactamente con lo especificado en la spec (`spec.md`)?
- ¿Se respetan los invariantes innegociables de `docs/constitution.md`?
- ¿Se manejan todos los casos límite (valores en 0, nulos, cancelaciones, estados de error)?
- ¿Todos los tests nuevos y existentes pasan en verde (`TEST_COMMAND`)?

### Eje 2: Legibilidad, Estilo y Convenciones del Repositorio
- **Idioma del Proyecto:** ¿Los comentarios, documentación y mensajes de interfaz y error respetan la convención del proyecto (español por defecto)?
- **Claridad de Nombres:** Nombres de variables y funciones autodescriptivos; prohibido el uso de nombres genéricos crudos (`data`, `temp`, `res2`).
- **Control de Flujo:** ¿Se evitan anidamientos profundos mediante cláusulas de guarda (*early returns*)?
- **Cero Código Muerto:** ¿Se eliminaron logs de depuración (`console.log`, `print`), código comentado o variables sin uso?

### Eje 3: Arquitectura y Desacoplamiento de Capas
- **Controladores / Endpoints Delgados:** La capa de transporte solo valida entradas y delega la ejecución al servicio de negocio.
- **Servicios con Reglas de Negocio:** Toda la lógica de cálculo, transformaciones y persistencia reside en la capa de servicio de dominio.
- **Manejo Centralizado de Errores:** Se utilizan excepciones tipadas y códigos de respuesta estandarizados.
- **Componentes de UI:** Si se modificó la interfaz, se respetan los contratos de accesibilidad, separación de estado y responsive design.

### Eje 4: Seguridad y DevSecOps
- **Prohibición Total de Fallbacks de Credenciales:** Confirmar que no existan cadenas de conexión ni contraseñas como fallback (`|| 'secret_key'`) en ningún archivo tocado.
- **Validación Estricta de Entradas:** Toda entrada externa pasa por esquemas de validación rigurosos.
- **Autorización y Autenticación:** Los endpoints protegidos aplican los middlewares o guardias de autenticación y roles adecuados.
- **Higiene de Secretos:** Confirmar que `.env` y credenciales privadas no hayan sido añadidos al commit (`git status`).

### Eje 5: Rendimiento y Base de Datos
- **Prevención de Consultas N+1:** ¿Se realizan consultas en lote o asociaciones adecuadas en lugar de consultas individuales en bucles?
- **Transacciones Atómicas:** ¿Las operaciones que tocan múltiples entidades críticas están envueltas en transacciones?
- **Índices y Filtrado:** ¿Las consultas a tablas con alto volumen aprovechan índices apropiados y límites de paginación?

---

## 3. Matriz de Decisión y Severidad de Hallazgos

| Tipo de Hallazgo | Nivel de Severidad | Acción Requerida |
| :--- | :--- | :--- |
| **Vulnerabilidad de Seguridad o Fuga de Credenciales** | 🔴 Bloqueante | Detener de inmediato. Corregir antes de cualquier commit. |
| **Violación de Invariante Constitucional o Fallo de Test** | 🔴 Bloqueante | No se aprueba el cambio. Debe resolverse en la rama temática. |
| **Regresión de Rendimiento Crítica (ej. N+1 masivo)** | 🟠 Importante | Corregir antes de autorizar el PR. |
| **Oportunidad de Simplificación o Nombres Poco Claros** | 🟡 Sugerencia | Aplicar `code-simplification` si no retrasa el objetivo. |
| **Preferencia Personal Estética sin Impacto Técnico** | 🟢 Omitir | No bloquear la entrega. |

---

## 4. Checklist de Aprobación de Quality Gate
Antes de declarar el código listo para la apertura de Pull Request:
- [ ] La compilación y chequeo de tipos aprueba sin errores (`TYPECHECK_COMMAND`).
- [ ] La compilación de frontend / distribución aprueba limpiamente (`BUILD_COMMAND` si aplica).
- [ ] Pruebas unitarias y de integración pasando al 100% (`TEST_COMMAND`).
- [ ] Cero secretos o credenciales expuestas en el diff (`git diff --staged`).
- [ ] Convenciones de arquitectura, estilo y código limpio plenamente respetadas.

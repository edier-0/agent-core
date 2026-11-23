---
name: doubt-driven-development
description: Somete decisiones críticas a una revisión adversarial para identificar fallos ocultos. Usar en transacciones complejas, integridad de datos, migraciones o lógica de alto riesgo.
category: methodology
---

# Doubt-Driven Development — Guía de Examen Adversarial

Una respuesta expresada con seguridad no garantiza que sea correcta. En sesiones largas de trabajo o diseños complejos, las suposiciones tienden a transformarse silenciosamente en "hechos" sin que nadie las cuestione. El desarrollo guiado por la duda (*Doubt-Driven Development*) es la postura disciplinada de someter decisiones críticas a un examen adversarial —orientado a **refutar**, no a justificar complacientemente— mientras corregir el rumbo aún es económico.

---

## 1. Cuándo Aplicar Esta Skill

Una decisión es **crítica o de alto riesgo** en el proyecto cuando:
- Afecta la **integridad financiera, contable o transaccional** (cálculo de balances, facturación, pasarelas de pago o conciliaciones).
- Cruza fronteras transaccionales complejas que involucran simultáneamente múltiples tablas, colecciones o servicios externos.
- Introduce una migración de esquema de base de datos que altere tablas con datos en producción.
- Modifica el flujo de autenticación, autorización, tokens de sesión o políticas de roles (RBAC).
- El agente siente la tentación de afirmar *"esto es seguro"* o *"esto no romperá nada"* sin evidencia técnica irrefutable.

> **Cuándo NO usar:** Operaciones mecánicas triviales (formateo, renombrado menor de variables locales) o cuando la corrección sea evidente y verificable al 100% por el compilador o linter.

---

## 2. El Proceso de Examen Adversarial en 4 Pasos

```text
1. FORMULAR   ──► Explicitar la decisión técnica y la afirmación de seguridad
2. ATACAR     ──► "¿Cómo puede fallar esto en producción?" (Rol de abogado del diablo)
3. CONTRASTAR ──► Verificar contra docs/constitution.md y casos límite
4. CONCLUIR   ──► Mantener, corregir o blindar con una prueba automatizada
```

### Paso 1: Formular la Afirmación
Escribe con claridad qué asumes como seguro:
> *"Afirmo que actualizar el inventario dentro de esta transacción previene condiciones de carrera si dos usuarios compran la última unidad disponible simultáneamente."*

### Paso 2: Atacar la Afirmación (Pensamiento Adversarial)
Hazte las preguntas incómodas y extremas:
- ¿Qué sucede si la conexión con la base de datos se interrumpe a mitad de la transacción?
- ¿Qué pasa si la solicitud se cancela o aborta por timeout 5 segundos después? ¿El estado se revierte limpiamente o queda inconsistente?
- ¿Qué ocurre si el cliente y el servidor manejan zonas horarias distintas al almacenar marcas de tiempo (`timestamps`)?
- ¿Qué sucede ante reintentos automáticos de la red (*idempotencia*)? ¿Se cobrará o registrará dos veces?

### Paso 3: Contrastar contra Invariantes Constitucionales
- ¿Esta solución persiste datos derivados que deberían calcularse dinámicamente?
- ¿Acopla componentes privados de otro módulo violando los límites de capas?
- ¿Introduce dependencias circulares o esquemas destructivos?

### Paso 4: Veredicto y Blindaje
- Si encuentras un modo de fallo: **corrígelo antes de commitear**.
- Si la solución resiste el ataque adversarial: **agrega una prueba automatizada en `TEST_COMMAND` que documente y blinde el caso límite superado**.

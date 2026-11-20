---
name: interview-me
description: Extrae lo que el usuario realmente necesita mediante una entrevista socrática de una sola pregunta a la vez hasta alcanzar ~95% de certidumbre. Usar cuando el requerimiento sea ambiguo antes de redactar una spec SDD.
---

# Interview Me — Guía de Entrevista Socrática de Requerimientos

Extrae lo que el usuario realmente necesita en lugar de lo que cree que debería pedir. Logra esto mediante una entrevista socrática de **una sola pregunta a la vez**, acompañada de tu mejor hipótesis, hasta alcanzar un ~95% de confianza sobre el objetivo de negocio subyacente.

> **Momento de Mínimo Coste:** El instante más económico para descubrir una discrepancia de requerimientos es **antes** de redactar cualquier especificación SDD, plan arquitectónico o línea de código. Una vez que comienzas a construir, cambiar de rumbo cuesta tiempo, frustración y retrabajo.

---

## 1. Cuándo Aplicar Esta Skill

- La petición carece de al menos uno de estos elementos: **para quién** es (rol de usuario), **por qué** se necesita ahora, qué define el **éxito** o cuál es la **restricción crítica**.
- La solicitud es genérica o convencional (ej. *"haz un dashboard"*, *"hazlo más rápido"*, *"organiza los pedidos"*).
- Estás tentado a rellenar silenciosamente requisitos ambiguos con suposiciones personales.
- Dos valores razonables entran en tensión y el usuario no ha elegido cuál priorizar (ej. simplicidad vs. flexibilidad; control estricto vs. rapidez operativa).
- El usuario lo solicita explícitamente: *"entrevístame"*, *"hazme preguntas"*, *"cuestióname"* o `/grill-me`.

> **Cuándo NO usar:**  
> - Peticiones directas, inequívocas y autocontenidas (ej. *"corrige esta errata en la línea 4"*, *"agrega este índice a la columna X"*).  
> - Consultas de información técnica pura (*"¿dónde se ubican los contratos de API?"*).  
> - Ya cuentas con $\ge 95\%$ de certeza demostrable.

---

## 2. El Proceso en 4 Pasos

### Paso 1: Formular Hipótesis con Nivel de Confianza
Antes de preguntar, escribe en una sola frase tu interpretación actual de lo que el usuario busca, junto a un porcentaje honesto de certidumbre (0–100%):

```text
HIPÓTESIS: Buscas alertar al usuario cuando una orden califique para un beneficio promocional para incentivar el cierre de la transacción.
CONFIANZA: ~40% — falta confirmar: si el canje se aplica automáticamente en la orden o si se redime como un cupón manual.
```

### Paso 2: Hacer UNA Sola Pregunta en Formato de Tarjetas de Selección
No abrumes al usuario con preguntas abiertas densas ni listas infinitas. Realiza una sola pregunta concreta a la vez estructurada como **tarjetas de selección**:
- **En plataformas con herramienta nativa (ej. `ask_question`)**: Invoca la herramienta configurando las opciones sin numerar, con la opción recomendada al inicio con `(Recommended)`. La interfaz provee tarjetas interactivas y la opción automática para escribir otra respuesta (*"Other"*).
- **En interfaces de chat / Markdown**: Formula la pregunta como tarjetas visuales con opciones etiquetadas y la opción explícita de respuesta libre:
  ```markdown
  > **¿El operador debe poder procesar pagos combinados (ej. efectivo y transferencia) en una sola orden?**
  >
  > - **[A] (Recomendada)**: Sí, permitir pagos divididos y registrar montos por cada método.
  > - **[B]**: No, cada orden admite un único método de pago cerrado.
  > - **[O] Escribir otra respuesta**: (Escribe tu alternativa personalizada si aplica).
  ```

### Paso 3: Actualizar la Hipótesis
Con la respuesta del usuario (ya sea seleccionando una opción o escribiendo su propio texto libre):
- Ajusta el porcentaje de certidumbre.
- Si la confianza sigue por debajo del 95%, formula la siguiente pregunta enfocada en el mismo formato de tarjetas.

### Paso 4: Condición de Parada ($\ge 95\%$ de Confianza)
Sabes que has terminado cuando puedes predecir con exactitud cómo respondería el usuario a los casos límite y decisiones de diseño. En ese momento:
1. Presenta un resumen ejecutivo de entendimiento.
2. Solicita confirmación para transicionar formalmente al workflow `/sdd-spec` e iniciar la redacción de `specs/NNN-nombre/spec.md`.

---

## 3. Estándar Innegociable de Encuestas y Preguntas
- **Cero Preguntas Abiertas Difusas**: Nunca lances preguntas al aire sin opciones base concretas.
- **Opción Recomendada Siempre Primero**: Fundamenta técnicamente la primera opción basada en la arquitectura y buenas prácticas.
- **Salida Libre Obligatoria**: Ofrece siempre la posibilidad de escribir otra respuesta (*"Other"* / `[O]`) para que el usuario nunca quede atrapado si ninguna alternativa cubre su necesidad de negocio.

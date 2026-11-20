---
name: browser-testing-with-devtools
description: Pruebas y verificación interactiva en el navegador mediante Chrome DevTools MCP. Usar para inspeccionar el DOM, capturar errores de consola y validar la UI en tiempo real.
---

# Browser Testing with DevTools — Guía de Verificación Interactiva

Guía para la verificación y pruebas interactivas de la interfaz de usuario en tiempo de ejecución utilizando el servidor MCP de Chrome DevTools (`chrome-devtools`). Proporciona al agente ojos directos en el navegador para auditar el DOM real, registrar errores de consola de JavaScript/framework, inspeccionar peticiones de red hacia la API y verificar visualmente la responsividad móvil.

---

## 1. Cuándo Aplicar Esta Skill

- Al desarrollar o modificar componentes de interfaz y vistas de usuario.
- Al depurar errores visuales en modales y formularios (comprobar la regla de las 3 capas fijas).
- Al auditar llamadas de red fallidas de clientes HTTP hacia la API.
- Al comprobar la experiencia móvil en viewport estrecho (375 px) vs escritorio (1280 px).
- Al verificar que los cambios no generen advertencias de reactividad o excepciones en la consola del navegador.
- En la fase de validación de specs (`/sdd-validate`).

> **Cuándo NO usar:** En proyectos puramente backend, CLI, librerías o entornos headless sin interfaz gráfica, o cuando `DEV_SERVER_URL` no esté configurado en `AGENTS.md`.

---

## 2. Herramientas MCP Disponibles en el Entorno

El servidor `chrome-devtools` expone herramientas de inspección:

| Herramienta MCP | Acción | Uso Típico |
| :--- | :--- | :--- |
| `puppeteer_navigate` | Abre una URL en el navegador | Navegar a la URL local de desarrollo (`DEV_SERVER_URL`). |
| `puppeteer_screenshot` | Captura imagen de pantalla | Validar visualmente el diseño, alineación y ausencia de desbordamientos. |
| `puppeteer_click` | Simula click en un elemento | Abrir modales, alternar pestañas o accionar botones de confirmación. |
| `puppeteer_fill` | Rellena campos de texto/inputs | Probar flujos de formularios, inputs numéricos o autenticación. |
| `puppeteer_evaluate` | Ejecuta JavaScript en la página | Consultar el estado del almacén, variables del DOM o posiciones de scroll. |

---

## 3. Protocolo de Pruebas de Interfaz

1. **Comprobación Pasiva del Servidor de Desarrollo:**
   - Lee `DEV_SERVER_URL` desde `AGENTS.md` (ej. `http://localhost:5173` o `http://localhost:3000`).
   - Comprueba si la URL está respondiendo.
   - **Regla Estricta (Cero Procesos Demonio):** Si el servidor no está activo, el subagente se detiene y solicita al usuario iniciarlo en su consola (`START_DEV_COMMAND`). Prohibido iniciar procesos demonio en segundo plano dentro del subagente.
2. **Navegación a la Vista:**
   - Usar `puppeteer_navigate` a la ruta bajo prueba.
3. **Auditoría de Consola y Red:**
   - Verificar que no aparezcan excepciones rojas (`Uncaught TypeError`, `401 Unauthorized` o `500 Server Error`).
4. **Verificación de Flujo Interactivo:**
   - Simular el flujo completo del usuario (abrir modal $\rightarrow$ ingresar datos $\rightarrow$ confirmar).
5. **Captura de Evidencia:**
   - Tomar captura de pantalla con `puppeteer_screenshot` para validar el resultado visual en viewport móvil y escritorio.

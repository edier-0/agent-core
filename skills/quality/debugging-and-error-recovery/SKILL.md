---
name: debugging-and-error-recovery
description: Guía la depuración sistemática de causa raíz y recuperación ante fallos. Usar cuando fallen tests, se rompa el build, surjan bugs en runtime o el comportamiento no coincida con lo esperado.
---

# Debugging & Error Recovery — Guía Sistemática de Depuración

Guía de depuración sistemática con triaje estructurado para la resolución de errores y fallos en el software. Cuando algo se rompa, detén la creación de nuevas características, preserva la evidencia y sigue un método riguroso para identificar y solucionar la causa raíz. Adivinar o cambiar código al azar consume tiempo y oculta los problemas reales.

---

## 1. La Regla de "Parar la Línea" (Stop-the-Line)

Ante cualquier comportamiento inesperado, regresión o fallo:

```text
1. PARAR      ──► Detener de inmediato la adición de nuevas características
2. PRESERVAR  ──► Guardar los logs de error, trazas y pasos de reproducción
3. DIAGNOSTICAR► Aplicar el triaje sistemático (de la causa al síntoma)
4. CORREGIR   ──► Implementar la solución mínima que resuelva la causa raíz
5. BLINDAR    ──► Crear una prueba de regresión automatizada permanente
6. REANUDAR   ──► Continuar solo tras verificar el sistema en verde
```

> **Advertencia Senior:** Jamás avances hacia la siguiente tarea de una spec si la prueba actual está fallando. Los errores se acumulan exponencialmente: una inconsistencia no resuelta en una capa inferior provocará fallos masivos en capas superiores más adelante.

---

## 2. El Triaje Sistemático en 5 Pasos

### Paso 1: Reproducir de Forma Determinista
Haz que el fallo ocurra a voluntad. Si no puedes reproducir el error consistentemente, no podrás confirmar que tu solución realmente lo resolvió:
- **En Backend / Lógica:** Escribe un test enfocado en la suite de pruebas que simule la petición o llamada fallida (Patrón *Prove-It*).
- **En Compilación / Tipado:** Ejecuta el comando exacto que falló (`TYPECHECK_COMMAND` o `BUILD_COMMAND`).
- **En Contenedores / Servidores:** Inspecciona los logs de salida y los códigos de error del proceso.

### Paso 2: Aislar y Localizar
Reduce la superficie de búsqueda para encontrar el origen exacto:
- **Inspección de Pila (*Stack Trace*):** Identifica el primer archivo de nuestro código fuente en la traza de error, descartando wrappers internos del runtime o dependencias de terceros.
- **Diferencial de Git Reciente:** ¿Qué cambió entre el último estado funcional y el actual?
  ```bash
  git diff HEAD~1
  # o revisar cambios locales no commiteados:
  git diff
  ```
- **Aislamiento de Componentes:** Si falla una transacción de base de datos, aísla si el error proviene de una restricción relacional, de un valor nulo no admitido o de la lógica de negocio.

### Paso 3: Identificar la Causa Raíz (Los 5 Por Qués)
No te conformes con mitigar el síntoma superficial:
- *Síntoma:* La API devuelve HTTP 500 al procesar una orden.
- *¿Por qué 1?* Porque la función arrojó `TypeError: Cannot read properties of undefined (reading 'price')`.
- *¿Por qué 2?* Porque el ítem enviado no tenía precio asignado en la base de datos.
- *¿Por qué 3?* Porque el esquema de validación permitió enviar un `itemId` inexistente sin comprobar su existencia previa.
- *Causa Raíz:* Falta de validación de existencia en la capa de servicio antes de operar y ausencia de una excepción de negocio controlada (HTTP 404).

### Paso 4: Corrección Quirúrgica
Aplica la solución más simple y directa que elimine la causa raíz identificada:
- No realices refactorizaciones no solicitadas en el mismo commit del bugfix.
- Asegúrate de canalizar el error con las excepciones estándar del proyecto.
- Prohibido silenciar el error mediante bloques `try / catch` vacíos o fallbacks silenciosos.

### Paso 5: Blindar contra Recurrencia (Prueba de Regresión)
Cada bug solucionado debe dejar como legado una prueba automatizada en `TEST_COMMAND` que garantice que el error nunca volverá a ocurrir:
```typescript
it('REGRESIÓN: debe arrojar error 404 si se intenta crear una orden con ítem inexistente', async () => {
  await expect(
    orderService.createOrder({ items: [{ itemId: 'id-inexistente', quantity: 1 }] })
  ).rejects.toThrow();
});
```

---

## 3. Matriz de Diagnóstico Rápido

| Error Frecuente | Causa Típica | Acción Correctiva |
| :--- | :--- | :--- |
| `Cannot find module ... or type declarations` | Problema en resolución de módulos o extensión faltante en import relativo. | Revisar la resolución del compilador y la extensión en el import. |
| `Foreign key constraint failed` / Error de Integridad | Se intenta insertar un registro con un `id` padre que no existe en BD. | Validar existencia de la entidad previa a la inserción y emitir error 404. |
| `TypeError: Cannot read properties of undefined` | Falta validación de entrada o acceso desprotegido a propiedades opcionales. | Incorporar esquema de validación estricto y verificar propiedades con encadenamiento opcional `?.`. |
| Timeouts en descarga de paquetes / build | Congestión de red o falta de caché en el gestor de dependencias. | Usar montajes de caché en el build y banderas no interactivas. |
| Discrepancias de fecha / saltos de día | Uso de fechas UTC en lugar de la zona horaria de la aplicación. | Utilizar los utilitarios de fecha y zona horaria oficiales del proyecto. |

---

## 4. Checklist de Cierre y Verificación
- [ ] El error se reprodujo inicialmente de manera confiable con un test en rojo.
- [ ] La corrección quirúrgica resolvió la causa raíz sin introducir efectos colaterales.
- [ ] El test de regresión pasa en verde de forma estable.
- [ ] La suite de pruebas completa pasa sin fallos (`TEST_COMMAND`).
- [ ] La compilación y chequeo de tipos no arroja advertencias (`TYPECHECK_COMMAND`).

---
name: shipping-and-launch
description: Prepara y valida despliegues a producción con listas de verificación, monitoreo y planes de reversión (rollback). Usar al liberar versiones o sincronizar producción.
category: delivery
---

# Shipping & Production Launch — Guía de Despliegue a Producción

Guía para desplegar a producción con absoluta certidumbre. El objetivo de ingeniería no es simplemente "hacer push", sino desplegar de forma segura, con compuertas de calidad automatizadas, supervisión activa de estado y un plan de contingencia (*rollback*) probado en caso de anomalías.

---

## 1. La Lista de Verificación Pre-Lanzamiento (Pre-Launch Checklist)

Antes de integrar un cambio en la rama de producción (`main`) y desplegarlo, deben verificarse todos los elementos:

### A. Calidad de Código y Pruebas
- [ ] Chequeo de tipos y linters aprobado sin errores (`TYPECHECK_COMMAND` y `LINT_COMMAND`).
- [ ] Empaquetado y build de frontend/distribución completado limpiamente (`BUILD_COMMAND`).
- [ ] Suite completa de pruebas unitarias e integración en verde (`TEST_COMMAND`).
- [ ] Cero logs de depuración (`console.log`, `print`) o código comentado en los archivos modificados.

### B. Seguridad y DevSecOps
- [ ] Escaneo de secretos limpio: ninguna contraseña, token JWT o clave API expuesta en el diff de Git.
- [ ] Política de cero fallbacks cumplida: todos los scripts y módulos exigen explícitamente variables desde `.env`.
- [ ] Auditoría de dependencias de producción limpia (cero vulnerabilidades altas o críticas).
- [ ] Permisos de autenticación y autorización (RBAC) validados en los recursos nuevos o modificados.

### C. Base de Datos y Persistencia
- [ ] Esquema de base de datos sincronizado en el entorno destino mediante migraciones no destructivas.
- [ ] Backup de seguridad de la base de datos generado antes de cualquier modificación estructural de datos.
- [ ] Cero consultas N+1 en las rutas y servicios afectados.

### D. Contenedores y Runtime
- [ ] Imagen de Docker construida con arquitectura multi-stage (`docker-hardening`).
- [ ] Proceso ejecutándose bajo usuario sin privilegios root.
- [ ] Endpoint de salud respondiendo correctamente (código HTTP 200 en healthcheck).

---

## 2. Secuencia de Sincronización hacia Producción (`main`)

Siguiendo el protocolo de `git-pr-workflow`:

```bash
# 1. Asegurar que la rama base de integración esté actualizada con el PR aprobado
git checkout develop
git pull origin develop

# 2. Sincronizar a producción (main)
git checkout main
git merge develop
git push origin main

# 3. Retornar inmediatamente a la rama base de desarrollo
git checkout develop
```
*Nota: Si GitHub rechaza el push directo a `main` por reglas de protección de rama, crea un Pull Request de Release (`develop -> main`) para aprobación en la plataforma.*

---

## 3. Plan de Contingencia y Reversión (*Rollback Plan*)

Si se detecta una anomalía o regresión crítica en producción tras el despliegue:

1. **Reversión Inmediata de Git:**
   ```bash
   git checkout main
   git revert -m 1 HEAD # Si fue un merge commit
   git push origin main
   ```
2. **Reversión de Contenedor / Despliegue:**
   Redirigir el tráfico a la imagen previa o versión anterior del artefacto en el orquestador.
3. **Reversión de Base de Datos:**
   Restaurar el snapshot o backup preventivo si la migración de datos corrompió registros, o aplicar la migración de compensación (patrón Expand & Contract).

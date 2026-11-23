---
name: docker-hardening
description: Estándares de contenedores Docker seguros, builds multi-stage híbridos, optimización de caché, usuarios no-root y hardening en producción.
category: delivery
---

# Docker Hardening & Multi-Stage Optimization — Guía de Contenedores

Guía canónica y estándares de ingeniería senior para la construcción, empaquetado, hardening y ejecución de contenedores Docker seguros y eficientes. Diseñada para garantizar builds rápidos, optimización de caché, portabilidad en entornos de CI/CD y despliegues seguros en producción.

---

## 1. Principios Fundamentales del Contenedor Seguro

1. **Arquitectura Multi-Stage Estricta:**
   - **Stage 1 (Builder):** Entorno completo con herramientas de compilación, transpiladores, dependencias de desarrollo y empaquetadores. Podado posterior de dependencias antes del traspaso.
   - **Stage 2 (Runner):** Imagen de producción mínima (ej. Alpine Linux o Distroless), sin compiladores, sin herramientas de desarrollo y sin código fuente no transpilado.
2. **Optimización de Caché y Montajes:**
   - Aprovechar montajes de caché de BuildKit (`--mount=type=cache,target=/root/.cache` o `/root/.npm`) para evitar descargas redundantes de paquetes.
   - Ordenar las instrucciones de `Dockerfile` de menor a mayor frecuencia de cambio (manifiestos de dependencias primero, código fuente después).
3. **Hardening de Seguridad en Producción:**
   - **Gestión PID 1:** Uso mandatario de un gestor de init (`dumb-init` o `tini`) como entrypoint del contenedor para recibir y propagar señales UNIX (`SIGTERM`, `SIGINT`) correctamente y evitar procesos zombies.
   - **Usuario No Privilegiado:** El proceso debe correr bajo un usuario no-root (`USER appuser` o `USER node`). Jamás ejecutar la aplicación en runtime como `root`.
   - **Superficie de Ataque Mínima:** Copiar únicamente los artefactos de compilación transpilados, dependencias de producción y activos públicos indispensables.
   - **Healthcheck Integrado:** Definir un chequeo de salud nativo (`HEALTHCHECK`) para que los orquestadores (Docker Compose, Kubernetes) detecten fallos sin intervención manual.

---

## 2. Plantilla Canónica de Dockerfile Multi-Stage

Patrón de referencia para aplicaciones web y APIs:

```dockerfile
# ==============================================================================
# STAGE 1: Builder (Compilación y generación de artefactos)
# ==============================================================================
FROM node:20-alpine AS builder

# Instalar librerías nativas necesarias para herramientas de compilación
RUN apk add --no-cache openssl libc6-compat

WORKDIR /app

# Copiar manifiestos de dependencias y configuración
COPY package*.json ./
COPY tsconfig*.json ./

# Instalar dependencias con caché de BuildKit
RUN --mount=type=cache,target=/root/.npm \
    npm ci --ignore-scripts

# Copiar código fuente
COPY src ./src/

# Transpilar y compilar a artefactos de producción (dist/)
RUN npm run build

# Podar dependencias dejando solo las necesarias para producción
RUN npm prune --omit=dev --ignore-scripts

# ==============================================================================
# STAGE 2: Runner de Producción (Hardened, Mínimo y Seguro)
# ==============================================================================
FROM node:20-alpine AS runner

# Instalar dumb-init para gestión adecuada de señales UNIX PID 1
RUN apk add --no-cache dumb-init

WORKDIR /app

ENV NODE_ENV=production
ENV PORT=3000

# Copiar manifiesto
COPY package*.json ./

# Copiar dependencias de producción podadas desde el builder
COPY --from=builder /app/node_modules ./node_modules

# Copiar artefactos transpilados finales
COPY --from=builder /app/dist ./dist

# Crear usuario y grupo no privilegiado si no existe
RUN chown -R node:node /app

# Ejecutar como usuario sin privilegios root
USER node

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=5s --start-period=15s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/api/health || exit 1

ENTRYPOINT ["/usr/bin/dumb-init", "--"]
CMD ["node", "dist/app.js"]
```

---

## 3. Higiene de `.dockerignore`

El contexto de build debe mantenerse lo más liviano posible para acelerar la transferencia al demonio de Docker. `.dockerignore` debe excluir rigurosamente:

```dockerignore
.git
.github
.agent
specs
docs
scratch
tests
coverage
*.md
.env*
!.env.example
docker-compose*.yml
node_modules
dist
build
.cache
```

---

## 4. Entrypoint y Ciclo de Vida en Runtime

1. **Gestión de Migraciones:**
   - Las migraciones de base de datos se ejecutan preferentemente en un paso previo de CI/CD o contenedor de migración efímero (`init container`), no en cada réplica del servidor web.
   - Si se ejecutan en un script de entrypoint, condicionarlas a una variable explícita (`RUN_MIGRATIONS=true`).
2. **Pase de Mando a PID 1:**
   - La última línea de cualquier script de entrypoint personalizado debe usar `exec "$@"` para transferir el control y PID 1 al proceso principal.

---

## 5. Checklist de Verificación Senior

- [ ] **Build multi-stage limpio:** La imagen final no contiene compiladores ni dependencias de desarrollo.
- [ ] **Usuario no-root:** El proceso corre bajo usuario sin privilegios de administración.
- [ ] **Manejo de señales:** El contenedor se apaga de inmediato ante `SIGTERM` sin esperar el timeout forzado de 10 segundos.
- [ ] **Healthcheck activo:** El contenedor reporta estado `"healthy"` tras iniciar.
- [ ] **Contexto liviano:** `.dockerignore` excluye carpetas de git, tests, documentación y secretos.

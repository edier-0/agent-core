---
name: ci-cd-and-automation
description: Automatiza y gestiona pipelines de CI/CD y compuertas de calidad automatizadas (GitHub Actions, GitLab CI). Usar al configurar o auditar flujos de integración continua.
category: delivery
---

# CI/CD & Pipeline Automation — Guía Canónica de Integración Continua

Guía canónica para la automatización de compuertas de calidad e integración continua. El pipeline de CI/CD es el mecanismo de aplicación estricta de las reglas del proyecto: garantiza que ningún cambio llegue a las ramas base (`develop` o `main`) sin haber superado de forma automatizada las pruebas, tipos, auditoría de seguridad y empaquetado de contenedores.

---

## 1. Principios del Pipeline de Integración Continua

1. **Desplazar a la Izquierda (*Shift Left*):** Detectar problemas lo más temprano posible. Un fallo de tipos o linter tarda segundos en detectarse en CI; el mismo fallo descubierto en producción cuesta horas.
2. **Fallar Rápido (*Fail-Fast*):** Cualquier paso que falle (tipos, tests, auditoría o build de Docker) detiene inmediatamente el pipeline y marca el Pull Request en rojo.
3. **Cancelación en Progreso (*Cancel-in-Progress*):** Si se envían nuevos commits a una rama abierta con PR, la ejecución previa se cancela automáticamente para ahorrar minutos de cómputo.
4. **Cero Secretos en Manifiestos:** Ningún secreto, token o contraseña se escribe en los archivos de workflow; todo valor sensible se inyecta mediante GitHub Secrets o variables de entorno protegidas.

---

## 2. Anatomía Canónica del Pipeline

El pipeline opera en trabajos (*jobs*) desacoplados y secuenciales:

```text
Pull Request hacia rama base
   │
   ▼
┌────────────────────────────────────────────────────────┐
│ JOB 1: Type Check, Security Audit & Tests (test)       │
│   ├── 📦 Instalación limpia de dependencias (ci)       │
│   ├── 🛡️ Auditoría de dependencias (seguridad)        │
│   ├── 🔍 Chequeo estricto de tipos / Linters           │
│   ├── 🎨 Compilación de frontend / assets (si aplica)  │
│   ├── 🗄️ Base de datos efímera de pruebas              │
│   └── 🧪 Ejecución de suite automatizada (TEST_COMMAND) │
└────────────────────────────────────────────────────────┘
   │
   ▼ (Si Job 1 pasa en verde)
┌────────────────────────────────────────────────────────┐
│ JOB 2: Container Image Build Test (docker-build)       │
│   ├── 🐳 Setup Docker Buildx                           │
│   └── 🔨 Build Multi-Stage autónomo (sin host deps)   │
└────────────────────────────────────────────────────────┘
   │
   ▼
✅ PR listo para revisión humana y fusión (Squash & Merge)
```

---

## 3. Compuertas de Calidad Específicas

### Compuerta 1: Auditoría de Dependencias (DevSecOps)
Verifica que las dependencias de producción no introduzcan vulnerabilidades de severidad alta o crítica:
```bash
npm audit --omit=dev --audit-level=high
# O equivalente en Python/Rust: pip-audit / cargo audit
```

### Compuerta 2: Tipado Estricto y Linters
Verifica que los archivos respeten los contratos de tipos sin errores de compilación ni fallos sintácticos:
```bash
TYPECHECK_COMMAND
LINT_COMMAND
```

### Compuerta 3: Pruebas de Integración y Base de Datos
Ejecuta la suite automatizada completa contra bases de datos de prueba locales o servicios en contenedores efímeros:
```bash
TEST_COMMAND
```

### Compuerta 4: Construcción de Contenedor Multi-Stage
Comprueba que el `Dockerfile` sea 100% autónomo y capaz de compilar la aplicación sin depender de archivos de la máquina física local:
```bash
docker build --no-cache -t app:ci .
```

---

## 4. Resolución de Fallos en CI

Si el pipeline de CI/CD se marca en rojo en un Pull Request:

1. **Consultar los Logs del Workflow:** Identificar el paso exacto que falló (ej. `Type Check`, `Audit` o `Test`).
2. **Reproducir en Local:** Ejecutar el comando exacto en tu terminal local para reproducir el fallo de forma determinista.
3. **Corregir en la Rama Temática:** No intentes solucionar el problema creando commits directos en la rama base. Corrige en tu rama temática (`feature/*`, `fix/*`).
4. **Publicar y Revalidar:**
   ```bash
   git push origin <tipo>/<NNN-slug>
   ```
   El pipeline de CI se reanudará automáticamente sobre el nuevo commit.

---

## 5. Checklist de Verificación
- [ ] Todo nuevo endpoint o módulo cuenta con pruebas automatizadas en `TEST_COMMAND`.
- [ ] Ningún secreto ni variable privada se pasa en texto plano en los archivos de workflow.
- [ ] La compilación y empaquetado de Docker funciona en limpio sin depender de dependencias del host.
- [ ] El pipeline bloquea el merge si existe cualquier check fallido.

---
name: devsecops
description: Estándares de seguridad DevSecOps, política estricta de cero fallbacks de credenciales, escaneo de secretos, auditoría de dependencias y blindaje de producción.
---

# DevSecOps & Security Hardening — Guía de Seguridad e Invariantes

Guía canónica y directivas de seguridad innegociables para el desarrollo, auditoría y despliegue del sistema. Esta skill establece los controles que previenen fugas de credenciales en repositorios, vulnerabilidades en dependencias, ejecución indebida de pruebas en bases de datos de producción y fallos de autorización.

---

## 1. La Regla de Oro: Prohibición Total de Fallbacks de Credenciales

Queda **terminantemente prohibido** quemar o hardcodear cadenas de conexión, contraseñas, tokens JWT o claves de API como valor por defecto o fallback (`|| 'postgresql://...'` o `|| 'secret_token'`) en scripts de utilidad, archivos de prueba, migraciones o código de aplicación.

### Patrón Prohibido (Vulnerabilidad Crítica):
```javascript
// ❌ INACEPTABLE: Si falta la variable, expone credenciales reales en el repositorio
const dbUrl = process.env.DATABASE_URL || 'postgresql://app_user:SecretPass123@prod-db.example.com/production';
```

### Patrón Obligatorio (Fail-Fast Seguro):
```javascript
// ✅ CORRECTO: Exigir explícitamente la variable de entorno y fallar de inmediato
const dbUrl = process.env.DATABASE_URL;
if (!dbUrl) {
  console.error('\n❌ ERROR DE SEGURIDAD: La variable de entorno DATABASE_URL no está definida en .env');
  console.error('👉 Por directiva constitucional de DevSecOps, está prohibido hardcodear credenciales por defecto.');
  process.exit(1);
}
```

---

## 2. Matriz de Detección de Secretos Pre-Commit

Antes de ejecutar cualquier commit o abrir un Pull Request, el agente o desarrollador debe inspeccionar el diff staged (`git diff --staged`) para verificar que **ninguno** de los siguientes patrones sea introducido:

| Tipo de Secreto | Patrón de Riesgo | Acción Correctiva Inmediata |
| :--- | :--- | :--- |
| **Cadenas de Conexión de BD** | `postgresql://`, `mongodb://`, `mysql://`, `redis://` | Mover a `.env` y usar variables de entorno. Verificar `.gitignore`. |
| **Tokens JWT o Claves de Sesión** | `eyJhbGciOi...`, `JWT_SECRET=...` | Usar únicamente variables de entorno o valores dummy en pruebas. |
| **Claves Privadas y Certificados** | `-----BEGIN PRIVATE KEY-----`, `.pem`, `.key` | Asegurar que rutas de certificados estén en `.gitignore`. |
| **Tokens de Proveedores Cloud / APIs** | `ghp_...`, `AWS_SECRET_ACCESS_KEY`, `sk_live_...` | Reemplazar inmediatamente por Secrets del CI y variables locales. |

---

## 3. Plantilla Segura de `.env.example`

El repositorio solo debe albergar `.env.example` con valores de plantilla ficticios y explicativos:

```env
# ==============================================================================
# PLANTILLA DE CONFIGURACIÓN DE ENTORNO
# ADVERTENCIA: NUNCA colocar credenciales de producción o tokens reales aquí.
# ==============================================================================

NODE_ENV=development
PORT=3000

# Base de datos local de desarrollo y pruebas
DATABASE_URL="postgresql://postgres:postgrespassword@localhost:5432/app_dev?schema=public"
TEST_DATABASE_URL="postgresql://postgres:postgrespassword@localhost:5432/app_test?schema=public"

# Base de datos remota / producción (Solo en .env privado, no llenar aquí)
PROD_DATABASE_URL="postgresql://<usuario>:<password>@<host-cloud>/<dbname>?sslmode=require"

# Seguridad y Autenticación
JWT_SECRET="reemplazar-con-un-secreto-aleatorio-seguro-minimo-32-caracteres"
JWT_EXPIRES_IN="1d"
```

---

## 4. Auditoría de Dependencias y Seguridad en CI

El pipeline de integración continua debe ejecutar de forma obligatoria compuertas de seguridad:

1. **Auditoría de Vulnerabilidades en Dependencias:**
   - En Node.js: `npm audit --omit=dev --audit-level=high`
   - En Python: `pip-audit`
   - En Rust: `cargo audit`
   *Criterio de Aceptación:* Ninguna vulnerabilidad de nivel `high` o `critical` es tolerada en dependencias de producción.
2. **Superficie Reducida en Contenedores:**
   - La imagen final de Docker debe excluir todas las dependencias de desarrollo (`devDependencies`).
   - Ningún contenedor de producción debe correr con privilegios `root` (usar usuarios no privilegiados dedicados).

---

## 5. Validación de Entradas y Protección contra Inyección

1. **Esquemas de Validación en Toda Entrada Externa:**
   - Todo endpoint debe validar obligatoriamente los datos recibidos mediante esquemas estrictos de validación (Zod, Pydantic, etc.).
   - Prohibido pasar payloads crudos sin tipar a consultas de base de datos o comandos de sistema.
2. **Encabezados de Seguridad HTTP:**
   - Configurar encabezados de seguridad estándar para protección contra XSS, Clickjacking y MIME sniffing.
   - Restringir CORS estrictamente a los orígenes autorizados en producción.

---

## 6. Procedimiento de Respuesta ante Fugas Accidentales

Si por error se commitea una credencial sensible antes de hacer push:
1. **NO HACER PUSH.**
2. Deshacer el commit preservando los archivos modificados:
   ```bash
   git reset HEAD~1
   ```
3. Reemplazar la credencial por la variable de entorno correspondiente y asegurar `.gitignore`.
4. Si la credencial llegó a ser publicada en el remoto:
   - Rotar y revocar inmediatamente la clave en el proveedor cloud o servicio afectado.
   - Purgar el commit del historial de Git mediante herramientas como `git-filter-repo` o rebase interactivo.

---

## 7. Aislamiento Estricto de Entornos: Prohibición de Pruebas en Producción

Queda **terminantemente prohibido** ejecutar la suite de pruebas (`vitest`, tests unitarios o suites de integración) apuntando a la base de datos de producción o cualquier host cloud remoto.

### Mecanismo de Blindaje Obligatorio:
1. **Runner de Pruebas:** Evalúa al inicio las variables de conexión de base de datos. Si detecta dominios remotos o coincidencia con variables de producción, aborta inmediatamente con error fatal antes de cargar las pruebas.
2. **Setup Global de Pruebas:** Valida que `TEST_DATABASE_URL` apunte a `localhost` o contenedores de prueba locales. Bloquea la suite si detecta hosts remotos.
3. **Cliente de Base de Datos:** Guardián de defensa en profundidad: si el entorno es de pruebas (`NODE_ENV === 'test'`) e intenta instanciar una conexión remota de producción, lanza una excepción inmediata.

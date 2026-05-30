# Instalación inicial fullstack (MONO)

Guía operativa para **proyectos monoempresa** al crear o completar el scaffold de `backend/` + `frontend/`. Complementa:

- [`docs/_base/00-inicio-arquitectura.md`](../../_base/00-inicio-arquitectura.md) (visión y checklist §7)
- [`docs/_base/symlinks_paqsuite_ia.md`](../../_base/symlinks_paqsuite_ia.md) (herencia IA, §4.0)
- Prompt: `prompts/scaffold-fullstack-inicio-proyecto.md` (PaqSuite-IA-BASE)

---

## 1) Prerrequisitos

| Herramienta | Versión mínima |
|-------------|----------------|
| PHP | 8.1+ (extensiones: mbstring, openssl, pdo, tokenizer, xml, ctype, json, bcmath) |
| Composer | 2.x |
| Node.js | 18+ (recomendado 20 LTS) |
| npm | 9+ |

**Modo:** declarar **MONO** antes de instalar (sin `X-Company-Id`, tenancy vía `X-Paq-Cliente`).

---

## 2) Orden de instalación

```text
1. Symlinks herencia (docs/_base, docs/_mono, docs/00-contexto/_mono, .cursor/rules)
2. Backend: Laravel 10 + envelope ApiResponse + health /api/v1
3. Frontend: React 18 + Vite 5 + TypeScript + dependencias transversales
4. .env (raíz y/o backend/.env + frontend/.env)
5. Verificación: php artisan test + npm run test:all
```

---

## 3) Backend — Laravel 10 + envelope

### 3.1 Instalación (repositorio con carpeta `backend/` parcial o vacía)

Desde la **raíz del producto**:

```powershell
# Si backend/ solo tenía scaffold suelto, respaldar y reemplazar:
composer create-project laravel/laravel:^10.0 backend-laravel-tmp --prefer-dist --no-interaction
# Fusionar: mover contenido de backend-laravel-tmp a backend/ (o reemplazar carpeta)
# Copiar integraciones PaqSuite (ver §3.2)
cd backend
copy .env.example .env
php artisan key:generate
composer install
php artisan test
php artisan serve --port=8000
```

En **PedidosWeb** el backend ya incluye Laravel completo; para nuevos repos repetir el mismo patrón.

### 3.2 Integraciones obligatorias PaqSuite (post-Laravel)

| Pieza | Ubicación | Propósito |
|-------|-----------|-----------|
| **Envelope** | `app/Http/Responses/ApiResponse.php` | Respuestas `error` / `respuesta` / `resultado` |
| **Contrato** | [`00-arquitectura-api/envelope-respuestas.md`](./00-arquitectura-api/envelope-respuestas.md) | Spec humana + rangos `error` |
| **Rutas API** | `routes/api.php` con prefijo `v1` → URL `/api/v1/*` | Alineado a `RouteServiceProvider` (`prefix api`) |
| **Health** | `GET /api/v1/health` | Smoke backend + envelope |
| **OpenAPI raíz** | `OpenApi.php` (raíz backend) + **L5-Swagger** (`/api/documentation`) |
| **L5-Swagger** | `darkaonline/l5-swagger` ^8.6; config `config/l5-swagger.php`; ver [`docs/_base/00-openapi-l5-swagger-scaffold.md`](../../_base/00-openapi-l5-swagger-scaffold.md) |
| **Sanctum** | Incluido en Laravel 10 skeleton | Auth Bearer (login TR) |
| **Tests** | `tests/Feature/HealthCheckTest.php`, `tests/Unit/ApiResponseTest.php` | Regresión envelope |

**Uso del envelope en controllers:**

```php
use App\Http\Responses\ApiResponse;

return ApiResponse::success(['status' => 'up']);
return ApiResponse::error(1001, 'tenant.invalid', 400);
```

`resultado` vacío se serializa como **`{}`**, nunca `null`.

### 3.2.1 OpenAPI / L5-Swagger (scaffold obligatorio)

Guía detallada: [`docs/_base/00-openapi-l5-swagger-scaffold.md`](../../_base/00-openapi-l5-swagger-scaffold.md).

```powershell
cd backend
composer require darkaonline/l5-swagger:^8.6
php artisan vendor:publish --provider="L5Swagger\L5SwaggerServiceProvider"
composer dump-autoload
composer openapi
php artisan serve --port=8000
# UI: http://localhost:8000/api/documentation
```

| Pieza | Ubicación |
|-------|-----------|
| Raíz spec | `OpenApi.php` + `classmap` en `composer.json` |
| Config | `config/l5-swagger.php` (annotations: `app/` + `OpenApi.php`) |
| Spec generado | `storage/api-docs/api-docs.json` |
| Alias Composer | `"openapi": "@php artisan l5-swagger:generate"` |

### 3.3 Variables de entorno backend (`backend/.env.example`)

| Variable | Ejemplo | Notas |
|----------|---------|--------|
| `APP_URL` | `http://localhost:8000` | |
| `TENANT_HEADER_NAME` | `X-Paq-Cliente` | MONO |
| `TENANT_ALLOWED_CLIENTS` | `desarrollo,demo` | Stub hasta TR tenancy |
| `TENANT_DEFAULT_CLIENT` | `desarrollo` | Desarrollo local |
| `FRONTEND_URL` | `http://localhost:3000` | CORS / Sanctum stateful |
| `L5_SWAGGER_GENERATE_ALWAYS` | `true` | Regenerar spec en local al abrir UI |
| `L5_SWAGGER_CONST_HOST` | `http://localhost:8000` | Host por defecto en anotaciones |
| `DB_*` | SQL Server / MySQL según producto | PedidosWeb: `sqlsrv` |

### 3.4 Pendiente en slices (no bloquea scaffold)

- Middleware tenant (400 `tenant.invalid`) — parcialmente implementado en productos avanzados
- Capas Services / Repositories por dominio
- Seed seguridad (`TR-GEN-02-modelo-roles-permisos-seed`)

---

## 4) Frontend — React + Vite + stack transversal

### 4.1 Instalación base

Desde `frontend/`:

```powershell
npm install
```

Si el proyecto **no** tiene aún `package.json` (repo nuevo):

```powershell
npm create vite@latest . -- --template react-ts
npm install
```

### 4.2 Dependencias transversales MVP (instalar tras Vite)

Alineado a `docs/_base/00-inicio-arquitectura.md` §2:

```powershell
npm install react react-dom
npm install react-router-dom
npm install i18next react-i18next
npm install devextreme devextreme-react
npm install -D @playwright/test vitest @types/react @types/react-dom typescript vite
```

| Paquete | Uso |
|---------|-----|
| `react` / `react-dom` | UI |
| `react-router-dom` | Login, shell, rutas protegidas |
| `i18next` / `react-i18next` | Claves `auth.*`, UI transversal |
| `devextreme` / `devextreme-react` | Grillas, formularios, temas |
| `vitest` | Unit tests en `src/` |
| `@playwright/test` | E2E en `tests/e2e/` |

**Licencia DevExtreme:** `VITE_DEVEXTREME_LICENSE` en `.env` / `frontend/.env` (ver `docs/frontend/devextreme-norms.md` en producto).

### 4.3 Estructura de carpetas objetivo

```text
frontend/src/
  app/           # App, router, providers
  features/      # auth, i18n, …
  shared/
    http/        # client.ts + interceptors (Bearer, X-Paq-Cliente)
    ui/          # wrappers DevExtreme (DataGridDX, …)
  layouts/       # shell post-login (TR experiencia base)
```

### 4.4 Variables frontend (`.env` / `.env.example` en raíz o `frontend/`)

| Variable | Ejemplo |
|----------|---------|
| `VITE_API_BASE_URL` | `http://localhost:8000/api/v1` |
| `VITE_APP_VERSION` | desde `VERSION` en raíz |
| `VITE_DEVEXTREME_LICENSE` | clave comercial |

### 4.5 Proxy desarrollo (`vite.config.ts`)

Proxy `/api` → `http://localhost:8000` para evitar CORS en local.

---

## 5) Verificación post-instalación

| Check | Comando | Esperado |
|-------|---------|----------|
| Backend tests | `cd backend && php artisan test` | verde (health + ApiResponse) |
| Health HTTP | `GET http://localhost:8000/api/v1/health` | `error: 0`, `resultado.status: up` |
| OpenAPI UI | `GET http://localhost:8000/api/documentation` | Swagger UI 200 |
| OpenAPI spec | `cd backend && composer openapi` | `storage/api-docs/api-docs.json` actualizado |
| Frontend dev | `cd frontend && npm run dev` | `http://localhost:3000` |
| Frontend tests | `cd frontend && npm run test:all` | unit + E2E smoke |

---

## 6) Trazabilidad

| Tema | Documento |
|------|-----------|
| Envelope API | [`envelope-respuestas.md`](./00-arquitectura-api/envelope-respuestas.md) |
| Login / sesión | [`02-acceso-y-seguridad/login-y-sesion.md`](../02-acceso-y-seguridad/login-y-sesion.md) |
| Normas TR | `docs/04-tareas/_NORMAS-TRANSVERSALES-TR.md` (producto) |
| Manual programador | `docs/_base/_MANUAL-PROGRAMADOR.MD` §4 |

---

*Última actualización: 2026-05-30 — L5-Swagger en scaffold backend.*

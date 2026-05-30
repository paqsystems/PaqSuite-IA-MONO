# Host, tenant y SQL en proyectos MONO

Especificación compartida en **`docs/_base/resolucion-host-cliente-sql-mono.md`** (symlink `docs/_base` → PaqSuite-IA-BASE).

## Patrón único (todos los productos MONO)

| Rol | URL |
|-----|-----|
| Entrada | `{cliente}.{proyecto}.paqsystems.com` |
| Frontend | `frontend.{proyecto}.paqsystems.com` |
| Backend | `backend.{proyecto}.paqsystems.com` |

Flujo: entrada → redirect a **frontend** con contexto **`{cliente}`** → API en **backend** con `X-Paq-Cliente` → SQL vía `EMPRESAS_CONEXION`.

Regla Cursor: **`.cursor/rules/15-host-subdominio-base-datos-y-branding.md`** (logo, Tailscale).

## En el OpenSpec de cada producto

Solo constantes: slug `{proyecto}`, convención de nombre de BD, tenant de desarrollo (`demo` / `desarrollo`).

## MULTI ERP

`X-Company-Id` y selector en UI: `docs/_base/regla-cursor-multitenant-paqsuite.md` — no es MONO.

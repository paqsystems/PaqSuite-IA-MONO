# Envelope JSON de respuestas API (MONO)

Contrato **obligatorio** para todas las respuestas HTTP de la API REST (`/api/v1/*`), compartido por web, mobile e integraciones.

**Implementación técnica (reglas Cursor):** `.cursor/rules/mono/03-api-contract.md`  
**Código Laravel:** `App\Http\Responses\ApiResponse` — ver [`00-instalacion-scaffold-fullstack.md`](../00-instalacion-scaffold-fullstack.md) §3.  
**Normas TR por slice:** `docs/04-tareas/_NORMAS-TRANSVERSALES-TR.md` §2 (en cada producto MONO).

---

## 1) Formato estándar (SIEMPRE)

Todas las respuestas — éxito o error — son JSON con **exactamente** estas tres propiedades de primer nivel:

```json
{
  "error": 0,
  "respuesta": "mensaje para UI",
  "resultado": {}
}
```

| Campo | Tipo | Regla |
|-------|------|--------|
| **`error`** | **entero** | `0` = OK; `≠ 0` = error controlado (validación, negocio, autorización, etc.). **No usar booleano.** |
| **`respuesta`** | string | Mensaje para la UI. Ver §4 (texto vs clave i18n). |
| **`resultado`** | **objeto JSON** | Payload de datos. **Siempre presente.** **Nunca `null` ni ausente.** Usar `{}` cuando no hay datos. |

El frontend **no** debe aceptar formatos alternativos.

---

## 2) Códigos HTTP y cuerpo

El **status HTTP** indica la categoría; el **cuerpo** mantiene siempre el envelope:

| HTTP | Uso típico |
|------|------------|
| 200 / 201 | OK |
| 400 | Request inválido |
| 401 | No autenticado |
| 403 | No autorizado |
| 404 | No encontrado |
| 409 | Conflicto |
| 422 | Validación (Laravel) |
| 429 | Rate limit |
| 500 | Error inesperado |

Ejemplo error 403:

```json
{
  "error": 3001,
  "respuesta": "auth.noCommercialProfile",
  "resultado": {}
}
```

---

## 3) Catálogo mínimo de códigos `error`

| Rango | Dominio |
|-------|---------|
| `0` | OK |
| `1000–1999` | Validación (request / DTO) |
| `2000–2999` | Reglas de negocio |
| `3000–3999` | Autorización / permisos |
| `4000–4999` | Not found / conflictos |
| `9000–9999` | Infraestructura / errores inesperados |

Los slices pueden documentar códigos concretos en TR; el rango debe respetarse.

---

## 4) Campo `respuesta` e i18n

| Caso | Valor de `respuesta` | UI |
|------|----------------------|-----|
| Éxito sin mensaje | `"ok"` o cadena vacía `""` | Opcional |
| Error traducible | **Clave i18n** (`auth.invalidCredentials`, `validation.required`, …) | Cliente traduce con `t(respuesta)` |
| Error no traducible / legacy | Texto literal en idioma del servidor | Mostrar tal cual (evitar en MVP salvo excepción) |

**Convención MVP:** preferir claves i18n en `respuesta` para errores mostrados al usuario; coordinar catálogo con `docs/00-contexto/_mono/01-experiencia-base/idioma-multilingual.md`.

---

## 5) Paginación dentro de `resultado`

Listados paginados incluyen la estructura estándar **dentro** de `resultado`:

```json
{
  "error": 0,
  "respuesta": "ok",
  "resultado": {
    "items": [],
    "page": 1,
    "page_size": 20,
    "total": 123,
    "total_pages": 7
  }
}
```

Query params habituales: `page`, `page_size`, `sort`, `sort_dir`, filtros por querystring (whitelist).

---

## 6) OpenAPI

- Todas las operaciones `/api/v1/*` documentan request/response con este envelope.
- Rutas protegidas: `security` Bearer + header `X-Paq-Cliente` donde aplique tenancy MONO.
- Spec generada: `{backend}/api/documentation` (L5-Swagger).

---

## 7) Ejemplos

**Health / ping:**

```json
{
  "error": 0,
  "respuesta": "ok",
  "resultado": {
    "serviceName": "PaqSuite-IA-PedidosWeb",
    "status": "up"
  }
}
```

**Login 200 (payload en `resultado`):**

```json
{
  "error": 0,
  "respuesta": "ok",
  "resultado": {
    "token": "1|…",
    "user": { "id": 1, "displayName": "…", "login": "…" },
    "functionalProfile": "vendedor",
    "locale": "es",
    "theme": "generic.light"
  }
}
```

**Credenciales inválidas (401):**

```json
{
  "error": 3002,
  "respuesta": "auth.invalidCredentials",
  "resultado": {}
}
```

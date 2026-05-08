# 01 — Contrato Base de API MONOEMPRESA (obligatorio)

## 0) API agnóstica de cliente (obligatorio)

La API REST debe servir a todos los frontends del sistema.

* Web y mobile consumen los mismos endpoints.
* Cualquier nuevo frontend debe poder usar la misma API sin cambios en backend.
* El contrato es único y compartido.

## 1) Formato estándar de respuesta (SIEMPRE)

Todas las respuestas HTTP deben ser JSON con:

```json
{
  "error": 0,
  "respuesta": "mensaje para UI",
  "resultado": {}
}
```

* `error`

  * `0` = OK
  * `!= 0` = error controlado

* `respuesta`

  * texto legible para usuario

* `resultado`

  * siempre objeto JSON
  * nunca null
  * usar `{}` cuando no existan datos

## 2) Códigos HTTP + `error`

* 200/201 OK
* 400 Request inválido
* 401 No autenticado
* 403 No autorizado
* 404 No encontrado
* 409 Conflicto
* 422 Validación
* 429 Rate limit
* 500 Error inesperado

El cuerpo SIEMPRE mantiene:

```json
{
  "error": 0,
  "respuesta": "",
  "resultado": {}
}
```

## 3) Catálogo mínimo de códigos `error`

* 0: OK
* 1000–1999: Validación
* 2000–2999: Negocio
* 3000–3999: Autorización
* 4000–4999: Not found/conflictos
* 9000–9999: Infraestructura/errores inesperados

## 4) Paginación, orden y filtros

Listados deben soportar:

* `page`
* `page_size`
* `sort`
* `sort_dir`
* filtros por querystring

Respuesta estándar:

```json
{
  "items": [],
  "page": 1,
  "page_size": 20,
  "total": 123,
  "total_pages": 7
}
```

### Rendimiento y volumen

* Mantener paginación obligatoria.
* Evitar colecciones completas salvo excepción explícita.
* Evitar consultas N+1.
* Mantener nombres de rutas estables para logs/performance.

## 5) Idempotencia y concurrencia

* PUT/PATCH/DELETE deben ser idempotentes.
* Contemplar control optimista cuando corresponda.

## 6) Versionado

* `/api/v1/...`
* Breaking changes:
  `/api/v2/...`

## 7) Contenido y formatos

* JSON UTF-8
* Fechas ISO-8601
* Montos decimal
* Sort/filter solo whitelist

## 8) Documentación OpenAPI

### Requisito obligatorio

* Todas las APIs deben documentarse mediante Swagger/OpenAPI.
* Generación automática desde código.

### Estándar

* OpenAPI 3.x
* L5-Swagger para Laravel

### Alcance

* Todos los endpoints `/api/v1/*`
* Públicos y protegidos
* Ejemplos request/response

### Seguridad

* Bearer Token (Sanctum)

### Envelope estándar

Las respuestas documentadas deben reflejar:

```json
{
  "error": 0,
  "respuesta": "",
  "resultado": {}
}
```

### Versionado

* `/api/v1`
* Nuevas versiones → nueva spec OpenAPI

### Sincronización

* OpenAPI es fuente de verdad.
* Mantener specs alineadas.

### Ubicación

* `backend/storage/api-docs/api-docs.json`
* Swagger UI:
  `{URL_BASE_BACKEND}/api/documentation`
* JSON:
  `{URL_BASE_BACKEND}/docs`

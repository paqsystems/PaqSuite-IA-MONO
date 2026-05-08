# 00 — Política general para Backend MONOEMPRESA

## Objetivo

Definir reglas obligatorias para el BackEnd de todo el proyecto usando **Laravel + PHP** con **Laravel Sanctum** para autenticación.

## Alcance

Estas reglas aplican a:

* Todas las APIs (GET/POST/PUT/PATCH/DELETE)
* Servicios, validaciones, manejo de errores, seguridad, logging/auditoría
* ORM (Eloquent) y consideraciones para consultas SQL complejas

## Principios

* Consistencia > creatividad.
* Seguridad por defecto.
* Validar inputs antes de persistir.
* No exponer detalles internos (stacktrace, SQL, credenciales).
* Mantener contrato API estable (evitar breaking changes).
* Listados y respuestas con FKs: incluir texto de presentación (código, nombre, descripción o label) para la UI.
* Los IDs no son etiquetas para usuario final.

## Ordenamiento en listados API (`sort` / `dir`)

* Laravel 10: `orderBy()` solo acepta dirección `asc` o `desc`.
* No combinar `input('dir', 'default')` con `input('dir')` sin default.
* Mantener validaciones consistentes en parámetros de ordenamiento.

## Rendimiento y obtención de datos

* Evitar consultas N+1.
* Utilizar eager loading cuando corresponda.
* Mantener paginación en listados.
* Optimizar consultas complejas.
* Medir performance antes de optimizar prematuramente.
* Registrar tiempos de ejecución relevantes.

## Seguridad

* Autenticación mediante Laravel Sanctum.
* Validar permisos antes de operaciones sensibles.
* Nunca confiar en datos provenientes del frontend.
* Centralizar validaciones y autorización.

## Arquitectura

* Mantener arquitectura por capas:

  * Controller
  * Service
  * Domain
  * Repository

* Evitar lógica de negocio en controllers.

* Mantener Services reutilizables.

* Mantener repositories desacoplados.

## APIs

* Mantener contratos API consistentes.
* Usar respuestas JSON homogéneas.
* Manejar errores de forma centralizada.
* Mantener versionado `/api/v1/`.

## Referencias

* `.cursor/rules/03-api-contract.md`
* `.cursor/rules/04-security-sessions-tokens.md`
* `.cursor/rules/14-obtencion-datos-performance.md`
* `docs/01-arquitectura/01-arquitectura-proyecto.md`
* `docs/backend/PLAYBOOK_BACKEND_LARAVEL.md`
* `docs/domain/DATA_MODEL.md`

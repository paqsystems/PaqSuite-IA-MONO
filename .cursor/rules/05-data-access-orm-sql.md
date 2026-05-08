# 03 — Acceso a datos MONOEMPRESA: ORM (Eloquent) + Queries complejas

## 0) Herramientas nativas de Laravel

* Eloquent ORM
* Laravel Migrations
* Seeders y Factories
* Query Builder

NO usar Prisma/PSL.

---

## 1) Migrations

* Todas las estructuras mediante Laravel Migrations.
* Toda migration debe ser reversible.
* No modificar migrations ya aplicadas en producción.

### Convenciones

* nombres descriptivos
* control de versiones obligatorio
* `migrate:fresh` solo desarrollo

---

## 2) Modelos Eloquent

### Reglas

* `$fillable` explícito
* `$casts` obligatorio
* relaciones públicas
* usar Resources/DTOs

### Relaciones

* `hasMany`
* `belongsTo`
* `hasOne`
* `belongsToMany`

### Performance

* usar eager loading
* evitar N+1

### Scopes

Crear scopes reutilizables.

---

## 3) Seeders y Factories

### Seeders

* catálogos
* usuarios iniciales
* parámetros base

### Factories

* testing
* desarrollo
* datos fake

---

## 4) Convenciones de nombres físicos

### USERS

La tabla USERS reside en la base operativa local.

### Prefijos

* tablas proyecto:
  `PQ_`

### No afecta

* entidades lógicas
* DTOs
* endpoints
* clases

### Sí afecta

* tablas físicas
* vistas
* procedures
* funciones SQL

---

## 5) ORM estándar

* CRUD mediante Eloquent.
* No devolver modelos crudos.

---

## 6) Queries complejas

Permitido:

* Query Builder
* vistas
* stored procedures
* SQL parametrizado

Cuando existan:

* reportes
* agregaciones
* joins complejos
* CTE/window functions

---

## 7) Anti-patrones SQL

### Prohibido

Subqueries en WHERE para verificar existencia.

### Obligatorio

Usar LEFT JOIN + NULL verification.

---

## 8) Anti-SQL Injection

* bindings obligatorios
* whitelists
* validar `page_size`
* prohibido concatenar SQL

---

## 9) Performance básica

* evitar N+1
* usar paginación
* índices en filtros
* auditoría timestamps
* tracking usuario cuando corresponda

---

## Referencias

* `.cursor/rules/02-backend-policy.md`
* `docs/backend/PLAYBOOK_BACKEND_LARAVEL.md`
* `docs/domain/DATA_MODEL.md`

# 23 — Versionado y Actualización de Base de Datos en Deploy (MONOEMPRESA)

## 0) Propósito

Esta regla define:

* versionado de aplicación,
* actualización de esquema,
* actualización de datos fijos,
* metodología de deploy,

para proyectos MONOEMPRESA.

---

## 1) Versionado SemVer

Formato:

```text id="0t7l7r"
MAJOR.MINOR.PATCH
```

Ejemplo:

```text id="6wb1rx"
1.2.3
```

### Reglas

| Tipo  | Cuándo                             |
| ----- | ---------------------------------- |
| MAJOR | breaking changes                   |
| MINOR | funcionalidades nuevas compatibles |
| PATCH | correcciones y ajustes             |

---

## 2) Fuente de verdad

Archivo raíz:

```text id="6z8p8n"
VERSION
```

Contenido:

```text id="wjcrvu"
1.0.0
```

---

## 3) Migrations

### Reglas

* migrations reversibles
* no modificar migrations productivas
* estrategia forward-only
* rollback solo mediante nueva migration correctiva

---

## 4) Seeders

### Obligatorio

Todos los seeders deben ser:

```text id="jlwmr5"
idempotentes
```

### Uso

* catálogos
* menús
* configuraciones
* parámetros generales
* usuarios iniciales

---

## 5) Deploy MONOEMPRESA

### Modelo

Existe:

```text id="vutv7m"
UNA única base operativa
```

No existe:

* Dictionary DB
* Company DB
* tenant
* iteración de empresas

---

## 6) Orden de deploy

### En cada deploy

1. migrations
2. seeds

---

## 7) Instalación nueva

Flujo:

```text id="08prze"
php artisan migrate
php artisan db:seed
```

---

## 8) Política de rollback

No se ejecutan rollback de migrations en producción.

Correcciones:

```text id="h52vo7"
nueva migration correctiva
```

---

## 9) Workflow push / PR / CI

Antes de push/PR:

* revisar cambios
* sugerir bump versión
* justificar
* esperar confirmación usuario

---

## 10) Resumen

| Criterio       | Decisión             |
| -------------- | -------------------- |
| Versionado     | SemVer               |
| Fuente versión | archivo VERSION      |
| Seeds          | siempre en deploy    |
| Base de datos  | única base operativa |
| Rollback       | forward-only         |

---

## Referencias

* `.cursor/rules/multi/05-data-access-orm-sql.md` ó `.cursor/rules/mono/05-data-access-orm-sql.md`
* `docs/06-operacion/deploy-infraestructura.md`

# Regla: Plan de tareas del HU-007 (Parámetros generales) MONOEMPRESA

## Objetivo

HU-007 define el proceso general de mantenimiento de parámetros generales.

El proceso:

* se implementa una sola vez,
* es reutilizado por todos los módulos,
* filtra por `Programa`.

---

## Alcance del plan de tareas

---

## 1. Base de datos

### Implementar

* tabla `PQ_PARAMETROS_GRAL`
* seeds iniciales
* índices necesarios
* persistencia local

Referencia:

```text id="e2m8qj"
docs/modelo-datos/pq-parametros-gral.md
```

---

## 2. Backend

### Endpoints

```text id="09v9ga"
GET /api/v1/parametros-gral?programa={programa}
```

y actualización:

```text id="5mwd3z"
PATCH / PUT
```

---

## Reglas

* solo editar `Valor_*`
* no permitir altas
* no permitir bajas
* filtrar por `Programa`
* comparación case-insensitive
* invalidar caché relacionada

---

## Seguridad

Validar:

* autenticación
* permisos funcionales

---

## Tests

Implementar tests:

* GET
* actualización
* validaciones
* permisos

---

## 3. Frontend

### Implementar

* listado readonly
* edición por modal
* control dinámico por tipo
* labels i18n
* tooltips
* captions

---

## Reglas UI

* un único control según `tipo_valor`
* no edición inline genérica
* modal preferido

---

## 4. Integración con menú

Cada menú debe enviar:

```text id="k1q9nm"
procedimiento => programa
```

---

## 5. Deploy / seeds

Los seeds deben:

* ejecutarse en deploy
* ser idempotentes
* sincronizar parámetros por módulo

---

## 6. JSON fuente de verdad

Archivo:

```text id="n4g8wt"
PQ_PARAMETROS_GRAL.seed.json
```

---

## Seeder

Implementar:

```text id="k2s0ze"
PqParametrosGralSeeder
```

con:

* upsert por Programa + Clave
* sincronización caption
* sincronización tooltip
* sincronización tipo

---

## Reglas

Cuando una HU agregue parámetros:

1. actualizar documentación
2. actualizar JSON
3. ejecutar seed en desarrollo

---

## Persistencia MONOEMPRESA

Los parámetros residen en:

```text id="n7n0rh"
base operativa local
```

No existe:

* tenant
* X-Company-Id
* company db
* dictionary db

---

## Referencias

* `.cursor/rules/multi/13-parametros-generales-ui-listado-y-edicion-por-tipo.md` ó `.cursor/rules/mono/13-parametros-generales-ui-listado-y-edicion-por-tipo.md`
* `docs/03-historias-usuario/000-Generalidades/HU-007-Parametros-generales.md`

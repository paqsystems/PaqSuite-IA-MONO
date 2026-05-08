# 25 — Tareas con grillas: habilitar layouts (HU-001) MONOEMPRESA

## Regla

Cada vez que una tarea incluya una o más grillas:

* DataGrid
* listados
* ABMs
* pantallas con tablas

debe habilitarse el sistema de layouts persistentes.

---

## Obligatorio por cada grilla

### Implementar:

* guardar layout
* recuperar layout
* eliminar layout
* restaurar último layout utilizado

---

## Identificación de grillas

Cada grilla debe poseer:

| Propiedad | Descripción               |
| --------- | ------------------------- |
| proceso   | nombre lógico del proceso |
| grid_id   | identificador interno     |

Ejemplos:

```text id="hqf4dn"
Clientes
Pedidos
Empleados
```

Si existe una sola grilla:

```text id="2f5c2m"
grid_id = default
```

---

## Persistencia

Los layouts deben almacenarse en:

```text id="h5n6ba"
base operativa local
```

No existe:

* diccionario central
* company db
* empresa activa
* tenant

---

## Comportamiento obligatorio

Al abrir la pantalla:

* recuperar último layout usado
* permitir seleccionar otro
* permitir guardar nuevos layouts
* permitir eliminar layouts propios

---

## Alcance

Aplicar en:

* nuevas pantallas
* nuevas grillas
* ampliaciones de ABMs
* listados existentes modificados

---

## Relación con estándar DevExtreme

Esta regla complementa:

```text
.cursor/rules/multi/08-devextreme-grid-standards.md ó .cursor/rules/mono/08-devextreme-grid-standards.md
```

No reemplaza el estándar general de grillas.

---

## Referencias

* `.cursor/rules/multi/08-devextreme-grid-standards.md` ó `.cursor/rules/mono/08-devextreme-grid-standards.md`
* `docs/03-historias-usuario/000-Generalidades/HU-001-layouts-grilla.md`

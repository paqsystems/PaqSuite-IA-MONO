# 24 — Estándar de Grillas DevExtreme (DataGrid) MONOEMPRESA

## 0) Propósito

Esta regla define el estándar obligatorio para todas las grillas (DataGrid) del proyecto MONOEMPRESA utilizando DevExtreme.

---

## 1) Características Obligatorias

Toda grilla debe incluir:

* CRUD por fila
* Column chooser
* Ordenamiento
* Filtrado
* Reordenamiento
* Agrupación
* Totalizadores
* Selección múltiple
* Pager nativo
* Layouts persistentes

---

## 1.1 CRUD por fila

### Alta

Cuando el proceso permita alta desde listado:

* usar únicamente botón nativo `addRowButton`
* no duplicar botones “Nuevo/Crear”

### Edición y eliminación

* usar íconos nativos DevExtreme
* no usar columna textual “Acciones”

---

## 1.2 Columnas

* todas las columnas disponibles
* algunas visibles por defecto
* column chooser habilitado

### Columnas críticas

Usar:

```tsx
allowHiding={false}
critical={true}
```

para impedir ocultamiento.

---

## 1.3 Ordenamiento

```tsx
allowSorting={true}
```

---

## 1.4 Filtrado

```tsx
filterRow={{ visible: true }}
```

---

## 1.5 Reordenamiento

```tsx
allowColumnReordering={true}
```

---

## 1.6 Agrupación

```tsx
groupPanel={{ visible: true }}
```

---

## 1.7 Totalizadores

Usar:

```tsx
<Summary>
```

con:

* sum
* count
* avg
* min
* max

---

## 1.8 Selección múltiple

```tsx
selection={{ mode: 'multiple' }}
```

---

## 1.9 Paginación

Usar exclusivamente pager nativo DevExtreme.

### Tamaños estándar

```text
10 / 25 / 50
```

### Server paging

Usar:

```tsx
serverPaging
```

cuando backend pagine.

---

## 1.10 Hooks React

No hacer:

```tsx
return <Loading />
```

antes de definir hooks.

---

## 1.11 Layouts persistentes

Toda grilla debe soportar:

* guardar layout
* recuperar layout
* eliminar layout

---

## 2) Identificación de grillas

En MONOEMPRESA:

| Propiedad | Origen                    |
| --------- | ------------------------- |
| proceso   | nombre lógico del proceso |
| grid_id   | identificador interno     |

Ejemplos:

```text
Clientes
Empleados
Pedidos
```

Si existe una sola grilla:

```text
grid_id = default
```

---

## 3) Persistencia de layouts

Los layouts se almacenan en:

```text
base operativa local
```

No existe:

* diccionario central
* empresa activa
* company db
* tenant

---

## 4) data-testid obligatorio

Formato:

```text
grid.{proceso}.{grid_id}
```

Ejemplos:

```text
grid.clientes.default
grid.pedidos.master
```

---

## 5) Configuración de referencia

```tsx
<DataGrid
  dataSource={dataSource}
  showBorders={true}
  allowColumnReordering={true}
  allowColumnResizing={true}
  columnAutoWidth={true}
  filterRow={{ visible: true }}
  groupPanel={{ visible: true }}
  selection={{ mode: 'multiple' }}
  headerFilter={{ visible: true }}
  columnChooser={{ mode: 'select' }}
  searchPanel={{ visible: true }}
>
```

---

## 6) Referencias

* `.cursor/rules/09-tareas-grillas-habilitar-layouts-hu001.md`
* `docs/frontend/ui-layer-wrappers.md`
* `docs/frontend/devextreme-norms.md`

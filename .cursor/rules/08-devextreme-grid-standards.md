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
* cada ícono con **tooltip** (`hint`) e i18n — regla base: `.cursor/rules/base/20-frontend/28-ui-grilla-acciones-iconos-tooltip.md`

### Acciones custom por fila (detalle, duplicar, etc.)

* mismo patrón: **ícono DevExtreme + tooltip**, no botones con texto
* ver `.cursor/rules/base/20-frontend/28-ui-grilla-acciones-iconos-tooltip.md`

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

### Selector de layouts

* La opción **Plantilla del sistema** (`layoutId: null`) restaura el estado original de columnas del proceso (`DataGrid.state(null)`).
* Los layouts **propios** del usuario (`isOwner: true`) se distinguen en el selector con sufijo **` (*)`** al final del nombre (i18n `gridLayout.ownerMarker`).
* Con plantilla del sistema activa, **Guardar** abre siempre **Guardar como** (no altera la plantilla base).

### Implementación de referencia

* `frontend/src/features/gridLayouts/hooks/useGridLayouts.tsx`
* `frontend/src/features/gridLayouts/components/GridLayoutToolbar.tsx`
* `frontend/src/shared/ui/grids/DataGridDx.tsx` — `applyState(null)` debe resetear la grilla

### Totalizadores de pie en layouts

* Al **guardar** un layout, `captureState` debe incluir los totalizadores reales del footer (`summary.totalItems`), excluyendo el placeholder técnico `paqSummaryFooterPlaceholder`.
* Al **cargar** un layout, `applyState` debe restaurar esos totalizadores vía `instance.option('summary.totalItems', …)` además del `state()` de columnas/filtros.
* Clave de persistencia en el JSON del layout: `paqSummaryTotalItems` (`PAQ_SUMMARY_TOTAL_ITEMS_STATE_KEY` en `dataGridSummaryFooter.ts`).

---

## 1.13 Exportación Excel — básica vs formateada

DevExtreme `exportDataGrid` aplica `numFmt` y negrita de columnas **antes** de `customizeCell`. La modalidad **básica** debe limpiarlos explícitamente (`autoFilterEnabled: false`); si no, ambas modalidades se ven iguales.

### Básica

* Sin `autoFilter`, sin `numFmt`, sin negrita en encabezados, sin fondo gris.
* Valores crudos (fechas como ISO string en fallback).

### Formateada (default)

Modalidad **formateada** debe diferenciarse de la básica:

| Tipo | Regla Excel |
|------|-------------|
| Fecha / datetime | `numFmt` según locale activo (i18n) |
| Entero | Numérico sin decimales (`0`) |
| Decimal | Según `column.format`; fallback `0.00` |
| Booleano | Texto i18n (`gridExport.boolean.true` / `false`; es: VERDADERO/FALSO) |
| Encabezados | Negrita + fondo gris (`#D9D9D9`) |
| Totales de pie | Incluir `totalFooter` / `groupFooter` con formato numérico y negrita |

Implementación: `frontend/src/shared/ui/gridExport/exportDataGridExcel.ts`, `excelExportFormatting.ts`.

---

## 1.12 Procesos tipo Informes — ícono Actualizar

Aplica a grillas de **solo lectura** que muestran datos obtenidos del servidor (consultas, informes comerciales, listados de reporte). No aplica a ABM ni a pantallas de carga/edición.

### Obligatorio

* Incluir ícono **Actualizar** en la toolbar externa de `DataGridDx` (`toolbarEnd`).
* Ubicación: **antes** de layouts y exportación — lo más cerca posible del **column chooser** nativo de la grilla.
* Control: DevExtreme `Button` con `icon="refresh"` (no HTML nativo).
* Tooltip: i18n obligatorio — clave **`grid.refresh`** en los 5 locales.
* `data-testid` estable: **`gridRefresh`** (`elementAttr` en el botón).
* Acción: **volver a obtener** los datos del origen (API / `loadData`); no solo refrescar el `dataSource` local sin re-fetch.

### Orden `toolbarEnd` (Informes)

```text
[actualizar] → [layouts] → [export] → [extras del proceso]
```

Coherente con TR-GEN-03 (`R-C1-02`); el ícono Actualizar es el primer slot de `toolbarEnd` en procesos Informes.

### Implementación de referencia

* Componente reutilizable: `frontend/src/features/consultas/components/GridRefreshButton.tsx`
* Patrón página: `ConsultaGridPage.tsx` — `refreshToken` / `useEffect` que re-ejecuta `loadData`
* Ejemplos: consultas comprobantes, detalle pedidos, informes deuda/cheques/stock/historial

### Criterio de cumplimiento

Al crear o tocar una grilla Informes: verificar toolbar con Actualizar, tooltip i18n, re-fetch servidor y testid `gridRefresh`.

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

* `.cursor/rules/multi/09-tareas-grillas-habilitar-layouts-hu001.md` ó `.cursor/rules/mono/09-tareas-grillas-habilitar-layouts-hu001.md`
* `docs/04-tareas/001-Generaliddes/TR-GEN-03-grillas-listados.md` — convención `toolbarEnd`
* `docs/03-historias-usuario/001-Generaliddes/HU-GEN-03-grillas-listados.md`
* `frontend/src/features/consultas/components/GridRefreshButton.tsx`
* `docs/frontend/ui-layer-wrappers.md`
* `docs/frontend/devextreme-norms.md`

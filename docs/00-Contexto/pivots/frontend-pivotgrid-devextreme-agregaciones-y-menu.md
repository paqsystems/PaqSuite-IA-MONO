# PivotGrid (DevExtreme) — agregaciones, menú contextual y tipos

**Proyecto:** PaqSuite  
**Ámbito:** frontend compartido (`PivotGridBlock`, convivencia con `DataGridDX`)  
**Versión:** 1.1  
**Fecha:** 2026-06-18  

---

## 1) Objetivo

Dejar registro de **cómo** se implementó la vista pivot basada en **DevExtreme PivotGrid** respecto a:

- elección de **agregación** (suma, promedio, mínimo, máximo, conteo) por campo en el área de **valores**;
- coherencia de **tipos** (`dataType`) cuando los datos vienen como string pero representan números o fechas;
- alineación con la **documentación oficial** (comportamiento no incluido “de fábrica” tal cual Excel/Tango);
- **paridad con grillas (GEN-03):** diseños guardados, plantilla inicial, ícono Actualizar e **i18n** completo del PivotGrid.

---

## 2) Contexto oficial (DevExtreme)

1. **`summaryType` por defecto** en la definición de campos del `PivotGridDataSource` es **`count`** si no se fija otro valor explícito. Esto explica conteos inesperados al mover un campo al área de datos sin configuración previa.  
   Referencia: [PivotGridDataSource — fields — summaryType](https://js.devexpress.com/React/Documentation/ApiReference/Data_Layer/PivotGridDataSource/Configuration/fields/).

2. La guía de **Summaries** indica usar **`dataType`** cuando el valor llega en formato ambiguo (p. ej. número o fecha como string).  
   Referencia: [PivotGrid — Summaries](https://js.devexpress.com/React/Documentation/Guide/UI_Components/PivotGrid/Summaries/).

3. El cambio de tipo de resumen **en tiempo de ejecución** (menú estilo “Suma / Promedio…”) **no viene implementado por defecto**; la guía lo describe en **Runtime Summary Type Selection** mediante **`onContextMenuPreparing`** y actualización del campo vía `dataSource.field(...)` + `load()`.  
   Referencia: [PivotGrid — Summaries — Runtime Summary Type Selection](https://js.devexpress.com/React/Documentation/Guide/UI_Components/PivotGrid/Summaries/#Runtime_Summary_Type_Selection).

4. En el evento **`onContextMenuPreparing`** del **PivotGrid**, el objeto **`field`** está disponible para interacciones desde el **Field panel** (pastillas de áreas).  
   Referencia: [PivotGrid — Configuration — onContextMenuPreparing](https://js.devexpress.com/React/Documentation/ApiReference/UI_Components/dxPivotGrid/Configuration/).

---

## 3) Implementación en el repositorio

| Elemento | Ubicación | Notas |
|----------|-----------|--------|
| Bloque de pivot reutilizable | `frontend/src/shared/components/PivotGridBlock.tsx` | Metadata API, diseños guardados, `PivotGrid` + `FieldPanel`, `fieldChooser` y `PivotGridDataSource`. |
| Contrato de campos guardados / API | `frontend/src/shared/services/pivotApi.ts` | `PivotFieldConfig` incluye `dataField`, `area`, `summaryType` opcional; `dataType` opcional solo como apoyo al contrato si se persiste en el futuro. |
| Toggle grilla / pivot y `fallbackFields` | `frontend/src/shared/ui/DataGridDX/DataGridDX.tsx` | Define campos iniciales cuando no hay `pivotBase` en catálogo. |
| Informe piloto (textos i18n) | `frontend/src/features/partesProduccion/components/InformesGestionPivotBlock.tsx` | Pasa `consultaId` y textos al bloque. |

### 3.1 Menú contextual de agregación (clic derecho)

- Se implementa **`onContextMenuPreparing`** en **`PivotGrid`** con la lógica recomendada por la guía: si `e.field?.area === 'data'`, se añaden ítems al menú (Sumar, Promediar, Mínimo, Máximo, Contar) filtrados según el **tipo de valor** inferido o según **`tipoDato`** del catálogo `pq_pivots_campos` expuesto en metadata (`nombreTecnico` / `tipoDato`).
- Para que el mismo comportamiento aplique al **Field Chooser** integrado, la API del `PivotGrid` expone la configuración anidada **`fieldChooser`**. El componente hijo React **`<FieldChooser />`** no declara `onContextMenuPreparing` en sus tipos; por eso se usa **`fieldChooser={{ enabled, allowSearch, onContextMenuPreparing }}`** en las props del `PivotGrid` en lugar de duplicar el hijo con una prop no tipada.
- **Resolución de índice:** en algunos eventos `e.field.index` puede no venir informado; se añadió **`resolvePivotDataFieldIndex`** buscando el campo en `dataSource.fields()` por `dataField` y `area === 'data'`.

### 3.2 Menú contextual de encabezados (filas y columnas)

Implementación: `frontend/src/shared/pivot/utils/pivotHeaderContextMenu.ts` (`buildPivotHeaderContextMenuItems`).

| Layout filas | Área | Ítems adicionales |
|--------------|------|-------------------|
| **`standard`** (PedidosWeb) | **Columna** | **Expandir todo** / **Contraer todo** sobre la dimensión del encabezado (`dataSource.expandAll` / `collapseAll` + `reload`) |
| **`standard`** | **Fila** | **Misma paridad** que columnas (desde CC PQ #7, 2026-06-18) |
| **`standard`** | Fila (dimensión con detalle) | **Incluir detalle** (`includePivotFieldInRowArea`) cuando aplica (`codCliente` → `razonSocial`, etc.) |
| **`tree`** | Fila / columna | Expandir / contraer ítem + expandir/contraer todo |

Claves i18n: `pivot.dx.expandAll`, `pivot.dx.collapseAll` (5 locales + overrides `dxPivotGrid-*`).

### 3.3 Tipos (`dataType`) y `onFieldsPrepared`

- Al construir el `PivotGridDataSource`, para campos en área **`data`** se asigna **`dataType`** (`number` | `date` | `string`) a partir del catálogo o de una muestra de filas (incluye heurística para **strings con aspecto numérico**).
- Se usa **`onFieldsPrepared`** del `PivotGridDataSource` para **reconciliar** `dataType` cuando DevExtreme o el store autogeneran campos (`retrieveFields` por defecto `true`) y la inferencia inicial difiere, sin tocar el `summaryType` elegido por el usuario (evita pisar un “Contar” intencional solo por corregir tipo).

### 3.3 Qué **no** se expone en la UI actual

- No hay **selector global único** de operación para todos los valores: la agregación es **por campo de datos**, vía menú contextual (equivalente funcional al patrón oficial). Cualquier texto legacy en traducciones que hable de “operación para todos” puede ignorarse o limpiarse en una pasada de i18n.

---

## 4) Uso para el usuario final

1. Clic derecho sobre la **pastilla del campo** en el área **Valores** (field panel) o sobre el mismo campo en la zona de **Valores** del **Field Chooser**.
2. Elegir **Sumar**, **Promediar**, **Mínimo**, **Máximo** o **Contar** según las opciones mostradas (dependen del tipo inferido o del catálogo).
3. **Guardar** el diseño con los botones del toolbar del bloque (persistencia vía API de pivots guardados), si la pantalla lo habilita.

---

## 5) Pruebas y `data-testid`

Los `data-testid` del bloque siguen el prefijo configurable (`testIdPrefix`, p. ej. `pivot.{testId}`). Tras cambios en menús o toolbar, revisar E2E que apunten a selectores estables.

### Estables obligatorios (toolbar y acciones)

| Control | `data-testid` |
|---------|----------------|
| Selector de diseños | `pivotLayoutSelect` |
| Guardar | `pivotLayoutSave` |
| Guardar como | `pivotLayoutSaveAs` |
| Eliminar | `pivotLayoutDelete` |
| Diálogo nombre (Guardar como) | `pivotLayoutSaveAsDialog` |
| Actualizar datos | `pivotRefresh` |

---

## 6) Diseños guardados — paridad con layouts de grilla

Misma lógica funcional que [HU-GEN-03-layouts-grilla](../../../03-historias-usuario/001-Generaliddes/HU-GEN-03-layouts-grilla.md) y `GridLayoutToolbar`, adaptada a pivots (`pq_pivots_config` / API de diseños guardados).

### 6.1 Diseños propios con sufijo ` (*)`

- La API debe devolver `isOwner: true` cuando el usuario autenticado es el creador del diseño.
- En el **SelectBox** del toolbar, los diseños propios muestran el nombre seguido del sufijo visual **` (*)`**.
- Clave i18n: **`pivotLayout.ownerMarker`** (valor `" (*)"` en los 5 locales).
- El sufijo **no se persiste** en el nombre guardado en BD (`nombre` / `layout_name`).

### 6.2 Plantilla inicial → pivot vacía

- La opción **Plantilla inicial** del selector corresponde a **`configId: null`** (sin fila en `pq_pivots_config` activa).
- Al elegirla, el `PivotGridDataSource` debe **resetear** el diseño: sin campos en filas, columnas, filtros internos ni valores (pivot vacía lista para diseñar).
- Clave i18n del ítem: **`pivotLayout.initialTemplate`**.
- La **pivot base** de metadata (`pivotBase` en la definición de consulta) sigue existiendo como referencia del analista; la plantilla inicial en UI es el estado “sin diseño persistido ni campos asignados”, análogo a la **plantilla del sistema** en grillas (`layoutId: null`).

### 6.3 Guardar desde pivot vacía = Guardar como

- Con **plantilla inicial** activa (`configId: null`), el botón **Guardar** debe abrir siempre el flujo de **Guardar como** (diálogo de nombre + POST).
- No existe registro previo que actualizar; no se debe alterar la pivot base de metadata ni diseños ajenos.
- Misma regla si el usuario partió de plantilla inicial, asignó campos y aún no guardó.

### 6.4 Operaciones estándar

| Acción | Regla |
|--------|--------|
| **Guardar** | Solo si hay `configId` y `isOwner`; PUT del JSON de diseño |
| **Guardar como** | POST; nombre único por `consulta_id`; error i18n si duplicado |
| **Cargar** | Aplicar diseño seleccionado; restaurar último usado al montar si existe |
| **Eliminar** | Solo creador; borrado lógico |

Implementación de referencia (grilla): `frontend/src/features/gridLayouts/components/GridLayoutToolbar.tsx`, `useGridLayouts.tsx`.

---

## 7) Ícono Actualizar (re-fetch de datos)

Paridad con [08-devextreme-grid-standards.md §1.12](../../../.cursor/rules/mono/08-devextreme-grid-standards.md) y `GridRefreshButton.tsx`.

### Obligatorio en consultas pivotables tipo Informes

- **Button** DevExtreme con `icon="refresh"` en la toolbar del bloque (`toolbarEnd`).
- Ubicación: **antes** de diseños guardados y exportación.
- Tooltip i18n: **`pivot.refresh`** (5 locales).
- `data-testid`: **`pivotRefresh`** (`elementAttr` en el botón).
- Acción: **volver a obtener** el dataset del servidor con los **filtros generales y parámetros vigentes**; no limitarse a `dataSource.reload()` local si los datos dependen de una API externa.

### Orden `toolbarEnd` sugerido

```text
[actualizar] → [diseños guardados] → [export] → [extras del proceso]
```

Patrón de página: token de refresco (`refreshToken`) + `useEffect` que re-ejecuta la carga de datos, igual que `ConsultaGridPage.tsx`.

---

## 8) i18n del PivotGrid DevExtreme

Todo caption, etiqueta de field panel, Field Chooser, menú de agregación y texto de toolbar debe respetar el criterio i18n del proyecto (5 idiomas).

Documento normativo completo: **[patron-i18n-pivot-devextreme.md](patron-i18n-pivot-devextreme.md)**.

Resumen:

- Prefijos `pivot.*`, `pivot.dx.*`, `pivotLayout.*` en los 5 `locales/*.json`.
- Overrides `dxPivotGrid-*` vía `syncDevExtremeLocale` (misma regla `loadMessages` que la grilla).
- Remount del `PivotGrid` con `key` que incluya `locale` al cambiar idioma.

---

## 9) Referencias cruzadas

- Arquitectura del motor (metadata, API, persistencia): [arquitectura_motor_pivots_y_flujo.md](arquitectura_motor_pivots_y_flujo.md)
- i18n PivotGrid: [patron-i18n-pivot-devextreme.md](patron-i18n-pivot-devextreme.md)
- i18n grilla (patrón base): [patron-i18n-grilla-devextreme.md](../03-ui-transversal/patron-i18n-grilla-devextreme.md)
- Tarea vista pivot informes: `docs/04-tareas/102-InformesGestion/TR-022-vista-pivot-informes.md`
- Regla Cursor DevExtreme: `.cursor/rules/devextreme-frontend.mdc`

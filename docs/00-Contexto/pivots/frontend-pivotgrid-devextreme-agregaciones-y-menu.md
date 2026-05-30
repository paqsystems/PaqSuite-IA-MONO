# PivotGrid (DevExtreme) — agregaciones, menú contextual y tipos

**Proyecto:** PaqSuite  
**Ámbito:** frontend compartido (`PivotGridBlock`, convivencia con `DataGridDX`)  
**Versión:** 1.0  
**Fecha:** 2026-05-04  

---

## 1) Objetivo

Dejar registro de **cómo** se implementó la vista pivot basada en **DevExtreme PivotGrid** respecto a:

- elección de **agregación** (suma, promedio, mínimo, máximo, conteo) por campo en el área de **valores**;
- coherencia de **tipos** (`dataType`) cuando los datos vienen como string pero representan números o fechas;
- alineación con la **documentación oficial** (comportamiento no incluido “de fábrica” tal cual Excel/Tango).

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

### 3.2 Tipos (`dataType`) y `onFieldsPrepared`

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

---

## 6) Referencias cruzadas

- Arquitectura del motor (metadata, API, persistencia): `docs/01-arquitectura/pivots/arquitectura_motor_pivots_y_flujo.md`
- Tarea vista pivot informes: `docs/04-tareas/102-InformesGestion/TR-022-vista-pivot-informes.md`
- Normas generales DevExtreme en el repo: `docs/frontend/devextreme-norms.md`

# Patrón i18n — Grilla DevExtreme (`DataGridDx`)

| Campo | Valor |
|-------|--------|
| **Ámbito** | Proyectos MONO con `DataGridDx` transversal (SPEC-001-03 / GEN-03) |
| **Última actualización** | 2026-06-01 |
| **Implementación de referencia** | `frontend/src/shared/ui/grids/DataGridDx.tsx` |

Documento **normativo** para no repetir correcciones de i18n en grillas DevExtreme en proyectos futuros. Complementa [TR-GEN-01-idioma](../../../04-tareas/001-Generaliddes/TR-GEN-01-idioma.md) (checklist §4) y [TR-GEN-03-grillas-listados](../../../04-tareas/001-Generaliddes/TR-GEN-03-grillas-listados.md).

---

## 1) Principio

**No asumir** que DevExtreme traduce solo con `locale('es')`. Parte del texto sale del bundle DX, parte de props del componente y parte de menús contextuales internos. Toda cadena visible en la grilla transversal debe:

1. Existir en los **5 locales** de la app (`es`, `en`, `pt`, `fr`, `it`) bajo prefijo `grid.*` / `grid.dx.*` / `grid.summary.*`.
2. Aplicarse por el mecanismo correcto (prop explícita, override `dxDataGrid-*`, o menú custom).
3. Re-sincronizarse al **cambiar idioma** sin recargar la página.

---

## 2) Arquitectura (tres capas)

```
react-i18next (grid.*)
        │
        ├─► Props explícitas en DataGridDx (FilterRow, GroupPanel, ColumnChooser)
        │
        ├─► getGridDevExtremeMessageOverrides()  →  claves dxDataGrid-*
        │         (gridDevExtremeMessages.ts)
        │
        └─► syncDevExtremeLocale()
              ├─ loadMessages(bundle oficial DX por idioma)
              ├─ loadMessages(overrides app)
              └─ locale(código)
```

### Archivos obligatorios (referencia PaqSuite)

| Archivo | Responsabilidad |
|---------|-----------------|
| `frontend/src/features/i18n/syncDevExtremeLocale.ts` | Carga bundles DX + overrides; `locale()` |
| `frontend/src/features/i18n/gridDevExtremeMessages.ts` | Mapa `grid.dx.*` → `dxDataGrid-*` |
| `frontend/src/features/i18n/LocaleProvider.tsx` | Llama `syncDevExtremeLocale` al cambiar idioma |
| `frontend/src/features/i18n/i18n.ts` | Sync inicial al bootstrap |
| `frontend/src/shared/ui/grids/hooks/useDataGridDevExtremeTexts.ts` | Props DX derivadas de `t()` + sync en `useEffect` |
| `frontend/src/shared/ui/grids/DataGridDx.tsx` | Wrapper: props, `key={gridId-locale}`, summary, CSS pie |
| `frontend/src/shared/ui/grids/utils/dataGridSummaryContextMenu.ts` | Menú totalizadores (pie, por columna) |
| `frontend/src/shared/ui/grids/utils/dataGridSummaryFooter.ts` | Placeholder de fila de pie visible |
| `frontend/src/locales/*.json` | Catálogo `grid.*` |

---

## 3) Regla crítica: `loadMessages` (error frecuente)

Los JSON de DevExtreme ya vienen con forma `{ "es": { "dxDataGrid-…": "…" } }`.

```ts
// ❌ INCORRECTO — anida el locale dos veces; DX cae a inglés
import esMessages from 'devextreme/localization/messages/es.json';
loadMessages({ es: esMessages });

// ✅ CORRECTO
loadMessages(esMessages);
```

**Síntoma:** menú contextual de encabezado («Sort Ascending», «Group by This Column»), paginador o textos DX genéricos en **inglés** aunque la app esté en español.

**Prueba de regresión:** `frontend/src/features/i18n/syncDevExtremeLocale.test.ts`.

---

## 4) Inventario de superficies de la grilla

Cada fila indica **cómo** traducir y **qué validar en QA**.

| # | Superficie UI | Mecanismo | Claves i18n (app) | Notas |
|---|---------------|-----------|-------------------|--------|
| 1 | Operadores fila de filtro | Prop `FilterRow.operationDescriptions` | `grid.dx.filter.*` | No depender solo del bundle DX |
| 2 | Panel de agrupación (vacío) | Prop `GroupPanel.emptyPanelText` | `grid.dx.groupPanelEmpty` | |
| 3 | Selector de columnas | Props `ColumnChooser.title`, `emptyPanelText` | `grid.dx.columnChooserTitle`, `grid.dx.columnChooserEmpty` | |
| 4 | **Menú contextual de encabezado** (ordenar, agrupar, mover columna) | Override `dxDataGrid-*` vía `getGridDevExtremeMessageOverrides` | `grid.dx.sort.*`, `grid.dx.group.*`, `grid.dx.column.move*` | Clic derecho en cabecera de columna |
| 5 | Paginador / textos DX del pager | Bundle DX + `locale()` correcto | (bundle) | Verificar tras arreglar `loadMessages` |
| 6 | Formatos de total en pie (Suma, Prom, etc.) | Override `dxDataGrid-summary*` | `grid.dx.summary.*` | |
| 7 | **Menú totalizadores (pie)** | Menú custom `onContextMenuPreparing` (`target === 'footer'`) | `grid.summary.*` | **Un totalizador por columna**; clic derecho en celda de pie de esa columna |
| 8 | Cabeceras de columnas de negocio | `caption={t('grid.column.…')}` en cada `Column` | `grid.column.*` | |
| 9 | Hints acciones por fila | `hint` / `hintKey` en botones DX | `grid.action.*` | Sin texto visible en botón |
| 10 | Columna acciones (caption) | `caption` en columna `type="buttons"` | `grid.column.actions` | Evita ítem vacío en Column Chooser |
| 11 | Vacío / carga / error | Props `emptyMessageKey`, mensajes wrapper | `grid.empty`, `grid.loading`, `grid.error.load` | |
| 12 | Remount al cambiar idioma | `key={\`${gridId}-${locale}\`}` en `DataGrid` | — | Fuerza re-render de textos internos DX |

### Claves DX ↔ app (menú de encabezado)

| Clave DevExtreme | Clave app |
|------------------|-----------|
| `dxDataGrid-sortingAscendingText` | `grid.dx.sort.ascending` |
| `dxDataGrid-sortingDescendingText` | `grid.dx.sort.descending` |
| `dxDataGrid-sortingClearText` | `grid.dx.sort.clear` |
| `dxDataGrid-groupHeaderText` | `grid.dx.group.byColumn` |
| `dxDataGrid-ungroupHeaderText` | `grid.dx.group.ungroup` |
| `dxDataGrid-ungroupAllText` | `grid.dx.group.ungroupAll` |
| `dxDataGrid-moveColumnToTheLeft` | `grid.dx.column.moveLeft` |
| `dxDataGrid-moveColumnToTheRight` | `grid.dx.column.moveRight` |

Al añadir un texto DX nuevo: **1)** clave en los 5 `locales/*.json`, **2)** entrada en `gridDevExtremeMessages.ts`, **3)** si existe prop en el componente React, también en `useDataGridDevExtremeTexts.ts`.

---

## 5) Pie de grilla (totalizadores)

| Tema | Decisión |
|------|----------|
| Alcance | Un totalizador **por columna** (no uno global para toda la grilla) |
| Interacción | Clic derecho en la celda del **pie** de la columna → menú según tipo de dato |
| Fila visible | DevExtreme **no dibuja** pie si `summary.totalItems` está vacío → placeholder `custom` en `dataGridSummaryFooter.ts` |
| UX | Separadores visuales entre columnas en el pie (`dataGridDx.css` + `showColumnLines`) |
| Anti-patrón | `TotalItem` fijo con texto «N registros» para toda la grilla |

---

## 6) Checklist QA por idioma (grilla)

**Última ejecución manual:** 2026-06-01 (dashboard + `/demo/abm`, locale `es`) — ítems validados; ver [F-GEN-03-cierre-formal](../../../04-tareas/001-Generaliddes/F-GEN-03-cierre-formal.md).

Ejecutar con al menos **es** e **it** (o el idioma nuevo):

- [x] Operadores de `FilterRow` en idioma activo
- [x] Panel de agrupación vacío traducido
- [x] Column Chooser (título y panel vacío)
- [x] Menú contextual de **encabezado**: ordenar, agrupar, mover columna
- [x] Paginador (si aplica)
- [x] Clic derecho en pie de columna numérica → Sumar / Promedio / etc.
- [x] Dos columnas con totalizadores distintos visibles **en su columna**
- [x] Cambiar idioma en runtime → grilla actualiza textos (sin F5)
- [x] Paridad de claves `grid.dx.*` y `grid.summary.*` en los 5 JSON

---

## 7) Checklist al incorporar un proyecto nuevo

- [ ] Copiar o reutilizar `syncDevExtremeLocale`, `gridDevExtremeMessages`, `useDataGridDevExtremeTexts`
- [ ] `LocaleProvider` / bootstrap llaman `syncDevExtremeLocale` en el mismo ciclo que `i18n.changeLanguage`
- [ ] Toda pantalla tabular usa **`DataGridDx`**, no `DataGrid` suelto sin i18n
- [ ] Tests: `syncDevExtremeLocale.test.ts`, `gridDevExtremeMessages.test.ts`
- [ ] E2E smoke: login → grilla → caption / filtro en idioma elegido
- [ ] Leer §3 (`loadMessages`) antes del primer QA

---

## 8) Anti-patrones

| Anti-patrón | Por qué falla |
|-------------|----------------|
| Solo `locale('es')` sin overrides ni props | Menús DX y cadenas del bundle quedan en inglés o mezcladas |
| `loadMessages({ es: esMessages })` | Doble anidación; mensajes DX no aplican |
| Textos hardcodeados en `TotalItem` / `displayFormat` | No i18n; acopla idioma |
| Grilla sin `key` al cambiar locale | Textos internos DX obsoletos hasta F5 |
| Asumir un solo totalizador global | Requisito de negocio: **por columna** |
| Traducir solo en `es.json` | Incumple SPEC-001-01 (5 idiomas) |

---

## 9) Historial de hallazgos (GEN-03, 2026-06-01)

| Hallazgo | Corrección |
|----------|------------|
| Filtros en inglés | Props `FilterRow` + overrides + `loadMessages` correcto |
| Panel agrupación / Column Chooser en inglés | Props + `grid.dx.*` |
| Menú encabezado en inglés | Fix `loadMessages` + claves `grid.dx.sort/group/column` |
| Pie sin fila o un solo bloque | Placeholder footer + CSS separadores por columna |
| Totalizador «único» percibido | UX: separadores; lógica ya era por columna |

Detalle de implementación: [TR-GEN-03-grillas-listados §10](../../../04-tareas/001-Generaliddes/TR-GEN-03-grillas-listados.md).

---

## 10) Referencias

- SPEC: [SPEC-001-03-ui-transversal](../../../05-open-spec/001-Generaliddes/SPEC-001-03-ui-transversal.md)
- Regla Cursor (checklist QA global): [`.cursor/rules/base/40-i18n/41-i18n-and-testid.md`](../../../.cursor/rules/base/40-i18n/41-i18n-and-testid.md) — subsección **Grilla DevExtreme (`DataGridDx`)**
- Regla Cursor (implementación DX): `.cursor/rules/devextreme-frontend.mdc`
- Checklist idioma ítems 21–28: [TR-GEN-01-idioma §4](../../../04-tareas/001-Generaliddes/TR-GEN-01-idioma.md)

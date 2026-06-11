# Patrón i18n — Pivot DevExtreme (`PivotGrid`)

| Campo | Valor |
|-------|--------|
| **Ámbito** | Proyectos MONO con `PivotGridBlock` / consultas pivotables (SPEC-001-08) |
| **Última actualización** | 2026-06-10 |
| **Paridad con grillas** | Mismo criterio que [patron-i18n-grilla-devextreme.md](../03-ui-transversal/patron-i18n-grilla-devextreme.md) (SPEC-001-03 / GEN-03) |

Documento **normativo** para que captions, menús, tooltips y textos internos del **PivotGrid** DevExtreme respeten i18n en los **5 locales** (`es`, `en`, `pt`, `fr`, `it`).

---

## 1) Principio

**No asumir** que DevExtreme traduce solo con `locale('es')`. El PivotGrid expone textos en:

- **Field panel** (Filas, Columnas, Filtros, Valores)
- **Field Chooser**
- Menús contextuales de agregación y del propio componente
- Exportación integrada (si se habilita)
- Toolbar del bloque (Guardar, Actualizar, etc.)

Toda cadena visible debe:

1. Existir en los **5 locales** bajo prefijos `pivot.*`, `pivot.dx.*` y `pivotLayout.*`.
2. Aplicarse por el mecanismo correcto (prop explícita, override `dxPivotGrid-*`, menú custom o `t()` en toolbar).
3. Re-sincronizarse al **cambiar idioma** sin recargar la página.

---

## 2) Arquitectura (tres capas)

```
react-i18next (pivot.* / pivotLayout.*)
        │
        ├─► Props explícitas en PivotGridBlock (FieldPanel, FieldChooser, toolbar)
        │
        ├─► getPivotDevExtremeMessageOverrides()  →  claves dxPivotGrid-*
        │         (pivotDevExtremeMessages.ts — previsto)
        │
        └─► syncDevExtremeLocale()
              ├─ loadMessages(bundle oficial DX por idioma)
              ├─ loadMessages(overrides app)
              └─ locale(código)
```

### Archivos de referencia (implementación prevista / paridad grilla)

| Archivo | Responsabilidad |
|---------|-----------------|
| `frontend/src/features/i18n/syncDevExtremeLocale.ts` | Carga bundles DX + overrides; `locale()` |
| `frontend/src/features/i18n/pivotDevExtremeMessages.ts` | Mapa `pivot.dx.*` → `dxPivotGrid-*` |
| `frontend/src/shared/components/PivotGridBlock.tsx` | Wrapper: props, toolbar, `key` por locale |
| `frontend/src/locales/*.json` | Catálogo `pivot.*` y `pivotLayout.*` |

---

## 3) Regla crítica: `loadMessages`

Idéntica a la grilla — ver [patron-i18n-grilla-devextreme.md §3](../03-ui-transversal/patron-i18n-grilla-devextreme.md).

```ts
// ❌ INCORRECTO — anida el locale dos veces
loadMessages({ es: esMessages });

// ✅ CORRECTO
loadMessages(esMessages);
```

**Síntoma:** Field panel, Field Chooser o menús DX en **inglés** con app en español.

---

## 4) Inventario de superficies del pivot

| # | Superficie UI | Mecanismo | Claves i18n (app) | Notas |
|---|---------------|-----------|-------------------|--------|
| 1 | Áreas del field panel (Filas, Columnas, Filtros, Valores) | Override `dxPivotGrid-*` y/o props `texts` del `FieldPanel` | `pivot.dx.fieldPanel.*` | No depender solo del bundle DX |
| 2 | Field Chooser (título, vacío, búsqueda) | Props `fieldChooser` + overrides | `pivot.dx.fieldChooser.*` | |
| 3 | Menú contextual de agregación (Sumar, Promedio…) | Ítems custom en `onContextMenuPreparing` | `pivot.summary.*` | Paridad con menú de totalizadores de grilla |
| 4 | Toolbar — Guardar / Guardar como / Eliminar | `Button` DevExtreme + `t()` | `pivotLayout.save`, `pivotLayout.saveAs`, `pivotLayout.delete` | |
| 5 | Toolbar — selector de diseños | `SelectBox` + sufijo propios | `pivotLayout.initialTemplate`, `pivotLayout.ownerMarker` | `ownerMarker` = ` (*)` |
| 6 | Toolbar — ícono Actualizar | `Button` `icon="refresh"` | `pivot.refresh` | Tooltip obligatorio |
| 7 | Captions de campos pivotables | Metadata `nombreVisible` + i18n de proceso si aplica | `pivot.field.*` o catálogo por consulta | Nunca `nombreTecnico` en UI |
| 8 | Vacío / carga / error del bloque | Props del wrapper | `pivot.empty`, `pivot.loading`, `pivot.error.load` | |
| 9 | Remount al cambiar idioma | `key={\`${consultaId}-${locale}\`}` en `PivotGrid` | — | Fuerza re-render de textos internos DX |

### Claves DX ↔ app (ejemplos habituales)

| Clave DevExtreme | Clave app sugerida |
|------------------|-------------------|
| `dxPivotGrid-rowFields` | `pivot.dx.fieldPanel.rows` |
| `dxPivotGrid-columnFields` | `pivot.dx.fieldPanel.columns` |
| `dxPivotGrid-filterFields` | `pivot.dx.fieldPanel.filters` |
| `dxPivotGrid-dataFields` | `pivot.dx.fieldPanel.values` |
| `dxPivotGrid-allFields` | `pivot.dx.fieldChooser.allFields` |

Al añadir un texto DX nuevo: **1)** clave en los 5 `locales/*.json`, **2)** entrada en `pivotDevExtremeMessages.ts`, **3)** prop explícita en el componente si existe.

---

## 5) Checklist QA por idioma (pivot)

Ejecutar con al menos **es** e **it**:

- [ ] Field panel: nombres de áreas en idioma activo
- [ ] Field Chooser: título, vacío y búsqueda traducidos
- [ ] Menú contextual de agregación en área Valores
- [ ] Toolbar: Guardar, Guardar como, Eliminar, selector de diseños
- [ ] Sufijo ` (*)` en diseños propios (`pivotLayout.ownerMarker`)
- [ ] Opción **Plantilla inicial** (`pivotLayout.initialTemplate`) traducida
- [ ] Ícono Actualizar: tooltip `pivot.refresh`
- [ ] Cambiar idioma en runtime → pivot actualiza textos (sin F5)
- [ ] Paridad de claves `pivot.dx.*` y `pivotLayout.*` en los 5 JSON

---

## 6) Anti-patrones

| Anti-patrón | Por qué falla |
|-------------|----------------|
| Solo `locale('es')` sin overrides ni props | Field panel y menús DX en inglés o mezclados |
| Textos hardcodeados en menú de agregación | No i18n; incumple SPEC-001-01 |
| Traducir solo en `es.json` | Incumple los 5 idiomas obligatorios |
| Pivot sin `key` al cambiar locale | Textos internos DX obsoletos hasta F5 |
| Mostrar `nombreTecnico` del catálogo | Incumple norma conceptual de pivots |

---

## 7) Referencias

- UI toolbar y diseños guardados: [frontend-pivotgrid-devextreme-agregaciones-y-menu.md §7–9](frontend-pivotgrid-devextreme-agregaciones-y-menu.md)
- Paridad layouts grilla: [HU-GEN-03-layouts-grilla](../../../03-historias-usuario/001-Generaliddes/HU-GEN-03-layouts-grilla.md)
- Ícono Actualizar (criterio Informes): [08-devextreme-grid-standards.md §1.12](../../../.cursor/rules/mono/08-devextreme-grid-standards.md)
- Regla Cursor: `.cursor/rules/devextreme-frontend.mdc`

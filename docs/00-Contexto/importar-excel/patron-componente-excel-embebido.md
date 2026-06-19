# Patrón UI — componente Excel embebido en pantalla host

| Campo | Valor |
|-------|--------|
| **Epic** | SPEC-001-07 Importar Excel |
| **Estado** | Decisiones UX cerradas (2026-06-16) |
| **Alcance** | Componente **genérico** reutilizable en cualquier pantalla host |
| **Fuera de alcance** | Uso concreto en un proceso de negocio (ej. carga/edición pedidos → SPEC dedicado del proceso) |

**Relacionado:** motor backend GEN-07 (`PQ_EXCEL_*`, APIs `/api/v1/excel-import/*`), TR-GEN-07-*, TR-GEN-03-exportaciones (`saveExcelWithPicker`).

---

## 1) Principio

El componente es **cerrado**: encapsula exportar plantilla, importar, validar, staging, procesamiento según política del catálogo y presentación de errores. Su **salida** hacia el host es un **payload de datos válidos** (y metadatos del lote). **Qué hacer con ese payload** (grabar pedido, actualizar grilla ABM, etc.) es responsabilidad exclusiva del proceso invocador.

No acoplar lógica de negocio del host dentro del componente.

---

## 2) Superficie UI (solo web)

Dos acciones en toolbar del host (o slot equivalente):

| Acción | Comportamiento |
|--------|----------------|
| **Exportar planilla base** | Descarga `.xlsx` generada por backend. Nombre por defecto fijo: `{codigoProceso}_plantilla.xlsx` (sanitizado). Usar `saveExcelWithPicker` cuando el navegador lo permita; si no, descarga silenciosa (TR-GEN-03). |
| **Importar planilla** | Abre **modal** DevExtreme (`Popup`). |

### Modal — fase carga

- Botón **Examinar** → selector de **archivo** `.xlsx` (sin campo “carpeta”; solo web).
- **SelectBox** de hoja (tras leer hojas vía API).
- Acción **Validar** / **Importar** → `POST` lote + validación staging.

### Modal — error estructural

- Mensaje i18n explícito (columna faltante, duplicado, etc.).
- **Sin grilla** de filas.
- Botones: **Cerrar**, **Reintentar** (resetea selección archivo/hoja).

---

## 3) Dos ejes ortogonales

Tras validación exitosa en estructura, distinguir siempre:

| Eje | Pregunta | UI |
|-----|----------|-----|
| **A — Mostrar errores** | ¿Hubo filas con error? | Si **sí** → ampliar modal y mostrar grilla **solo filas con error** (paginación server-side). Si **no** → no mostrar grilla. |
| **B — Retorno al host** | ¿Se entregan datos válidos al proceso invocador? | Ver matriz §4. Independiente de si el usuario revisó la grilla de errores. |

Cuando hay errores **y** el proceso admite parcial: primero eje A (mostrar errores), luego eje B (retornar filas válidas procesadas).

---

## 4) Matriz de resultados

`permiteProcesamientoParcial` = flag del proceso en `PQ_EXCEL_PROCESOS`.

| Errores por fila | Parcial | Grilla errores (solo con error) | Procesar backend | Payload al host (`validRows`) |
|------------------|---------|-----------------------------------|------------------|-------------------------------|
| No | — | No | Todo el lote | Todas las filas válidas |
| Sí | `false` | Sí | **No** | **Vacío** `[]` |
| Sí | `true` | Sí (antes de cerrar) | Solo filas válidas | Solo filas válidas procesadas |

Estados de lote alineados a GEN-07: `procesada`, `procesada_parcial`, `lista_para_procesar` (bloqueado si parcial false y hay errores).

---

## 5) Contrato de salida (host)

### Evento `onComplete`

El host registra un callback invocado al **finalizar** el flujo con resultado definitivo (incluye cierre tras ver errores cuando corresponde retorno vacío o parcial).

```typescript
type ExcelImportHostResult = {
  guidImportacion: string;
  codigoProceso: string;
  validRows: Array<Record<string, unknown>>;
  meta: {
    totalFilas: number;
    filasValidas: number;
    filasConError: number;
    permiteProcesamientoParcial: boolean;
    estadoImportacion: string;
    nombreArchivoOriginal: string;
  };
};
```

| Campo | Regla |
|-------|--------|
| `validRows` | Filas **ya procesadas** por el handler (datos listos para uso del host). Vacío si hubo errores y parcial = false. Solo válidas si parcial = true con mezcla. Todas si cero errores. |
| Claves de cada fila | Campos internos del catálogo (`NombreCampoInterno` / equivalente API columnas), no necesariamente títulos Excel. |

### Evento `onCancel` (opcional)

Cierre del modal **sin** concluir importación (ej. usuario cancela antes de validar, o cierra en error estructural). No invoca `onComplete`.

### Evento `onClose` con errores sin parcial

Si el usuario cierra el modal tras ver grilla de errores (parcial false): `onComplete` con `validRows: []` y `meta.filasConError > 0`, **o** solo `onCancel` según implementación — **preferencia documentada:** `onComplete` con payload vacío para que el host pueda auditar `guidImportacion` si lo necesita.

---

## 6) Grilla de errores en modal

Cuando eje A aplica (≥ 1 fila con error):

- Modal se **amplía**; grilla `DataGridDx` bajo controles de archivo/hoja.
- **Solo filas con error** (`tieneError = true`), API paginada server-side.
- Columnas: definición de plantilla del proceso + columna **Errores** + **Número fila Excel**.
- Resumen en toolbar: totales válidas / con error.
- **Paginación:** server-side (misma API `GET .../filas` con filtro `soloErrores=true` o equivalente).

### Acciones en modal (errores)

| Control | Acción |
|---------|--------|
| **Exportar errores** | Ver §7 |
| **Reintentar** | Nueva selección de archivo (nuevo lote al confirmar) |
| **Cerrar** | Cierra modal; lote permanece en **historial** |
| **Continuar** (solo si parcial true y hay válidas) | Tras revisar errores → `onComplete` con `validRows` |

Si parcial **false**: no hay **Continuar** con datos; solo Cerrar / Reintentar / Exportar.

---

## 7) Exportar filas con error

| Aspecto | Valor |
|---------|--------|
| Contenido | Columnas plantilla + Errores + número fila Excel |
| Alcance | Solo filas con error del lote actual |
| Formato | `.xlsx` |
| Nombre sugerido | `{nombreArchivoOriginal_sinExtension}_errores_YYYYMMDDhhmmss.xlsx` |
| Mecanismo | Generación backend dedicada o export cliente vía API filtrada; guardado con `saveExcelWithPicker` / descarga |

Corrección fuera de la app → **nueva importación** = **nuevo lote** (v1, sin reproceso del mismo lote).

---

## 8) Flujo resumido (mermaid)

```mermaid
flowchart TD
  T[Toolbar host: Exportar | Importar] --> M[Modal: archivo + hoja]
  M --> V{Validación}
  V -->|Estructural falla| E[Mensaje + Cerrar/Reintentar]
  V -->|OK| R{Errores por fila?}
  R -->|No| P[Procesar todo]
  P --> H1[onComplete: todas las filas]
  R -->|Sí| G[Grilla solo errores]
  G --> P2{Parcial?}
  P2 -->|No| H2[onComplete: validRows vacío]
  P2 -->|Sí| P3[Procesar válidas]
  P3 --> G
  G --> C[Usuario: Exportar / Reintentar / Continuar]
  C --> H3[onComplete: solo válidas]
```

---

## 9) Historial

Todo lote creado queda en `PQ_EXCEL_IMPORTACIONES` y es consultable en historial transversal (`pw_historialimportexcel`), incluso si el usuario cierra el modal con errores sin reimportar.

---

## 10) Props sugeridas (implementación FE)

```typescript
type ExcelImportHostToolbarProps = {
  codigoProceso: string;
  disabled?: boolean;
  onComplete: (result: ExcelImportHostResult) => void;
  onCancel?: () => void;
};
```

- `codigoProceso` resuelve catálogo, plantilla, política parcial y `HandlerBackend`.
- El host **no** pasa columnas ni reglas; todo sale del catálogo.

---

## 11) Diferencias respecto al D1 actual (GEN-07)

| Tema | D1 actual | Patrón embebido |
|------|-----------|-----------------|
| Ubicación UI | Rutas `/excel-import/procesos/:codigo` y `/lotes/:guid` | Modal + toolbar en pantalla host |
| Grilla éxito | Página completa siempre tras upload | Sin grilla si cero errores |
| Grilla errores | Todas las filas | Solo filas con error |
| Salida | Usuario procesa en grilla dedicada | `onComplete` con payload al host |
| Export staging | Deshabilitado | Export errores `.xlsx` en modal |

Migración: el componente embebido **reutiliza** las mismas APIs; cambia composición UI y contrato hacia el host.

---

## 12) Procesos de negocio (ej. Pedidos Web)

La integración en **carga/edición de pedidos** (individual vs masiva, volcar renglones al pedido abierto, etc.) se documentará en un **SPEC del proceso** que referencie este patrón y defina el handler `onComplete` concreto. No forma parte de este documento.

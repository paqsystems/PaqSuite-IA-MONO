# Patrón — Formato decimal PivotGrid DevExtreme

| Campo | Valor |
|-------|--------|
| **Estado** | Vigente |
| **Alcance** | PivotGrid transversal (SPEC-001-08) |
| **Paridad** | Grillas consulta `#,##0.00` (GEN-03) |

## Regla (fuente de verdad)

Todo campo con **`tipoDato` numérico** (`number`, `numeric`, `decimal`) en metadata pivot debe mostrarse con:

| Propiedad | Valor |
|-----------|--------|
| DevExtreme `field.format` | `#,##0.00` |
| Separador miles | Sí (`,` locale-aware vía DX) |
| Decimales | **2** fijos |

Aplica a:

- Valores en área **data**
- **Subtotales** y **totales generales** del pivot
- Export Excel básico / tabla dinámica (hereda formato del componente)

## Implementación

| Capa | Artefacto |
|------|-----------|
| Backend metadata API | `PivotCampoFormatPolicy::resolveFormato()` en `PivotMetadataResolver` |
| Plantilla métrica seed | `PLANTILLA_METRICA_NUM` → `formato.format = #,##0.00` |
| Frontend campos | `resolvePivotDecimalFormat.ts` → `applyPivotNumberFieldFormat` en `onFieldsPrepared` |
| Catálogo / sintético | `mapMetadataToPivotFields`, `resolvePivotCampoForField` |

**Constantes:**

- PHP: `PivotCampoFormatPolicy::DECIMAL_DX_FORMAT`
- TS: `pivotDecimalDxFormat` en `frontend/src/shared/pivot/utils/resolvePivotDecimalFormat.ts`

## Excepciones

- Campos **fecha** (`date` / `datetime`): patrón `resolvePivotDateFormat` (i18n locale).
- Campos **texto** en valores (count/min/max): sin formato decimal.
- **No** usar formatos distintos por campo numérico salvo cambio explícito de este patrón.

## QA

1. Arrastrar métrica numérica (p. ej. cantidad, saldo, importe) a Valores.
2. Verificar celdas y fila/columna **Total** con miles y 2 decimales.
3. Cambiar agregación (sum/avg/min/max) — formato se mantiene.

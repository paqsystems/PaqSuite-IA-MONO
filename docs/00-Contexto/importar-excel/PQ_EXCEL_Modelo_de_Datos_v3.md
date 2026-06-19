# Modelo de Datos - Importación Excel

## 1. Objetivo
Definir la base conceptual del modelo de datos para soportar procesos de importación Excel en entorno multiusuario, con staging persistente, auditoría, historial y reglas estructurales fijas por proceso.

## 2. Regla definitiva de encabezado
- La fila de encabezado es siempre la fila 1.
- No se almacena parametrización de fila de encabezado.
- El parser asume fila 1 como fuente única de nombres de columnas.

## 3. Regla definitiva de nombres de columnas
La definición de columnas esperadas por proceso debe almacenar el nombre visible exacto que se exporta en la plantilla y contra el cual se valida el archivo importado.

### 3.1 Restricción del nombre visible
Se permiten únicamente:
- letras sin tildes
- números
- espacios

No se permiten:
- tildes
- puntuación
- símbolos especiales
- guiones bajos
- separadores especiales

## 4. Consecuencia para el modelo
Cada campo esperado por proceso debe contemplar dos conceptos:
- nombre visible Excel
- nombre interno técnico

Ejemplo conceptual:
- `NombreColumnaExcel`: `Fecha Emision`
- `NombreCampoInterno`: `fecha_emision`

## 5. Entidades conceptuales principales
- Proceso de importación
- Campo definido por proceso
- Lote de importación
- Fila importada
- Error de fila
- Historial de importaciones

## 6. Tablas previstas
Se propondrá una estructura con prefijo `PQ_EXCEL_`, incluyendo:
- configuración de procesos
- definición de campos
- cabecera de importaciones
- detalle de filas importadas
- errores por fila si se decide separar
- historial consultable

## 7. Impacto de las definiciones adoptadas
Las reglas ya cerradas producen estas simplificaciones de diseño:

### 7.1 Encabezado fijo
- evita configuración adicional
- simplifica el parser
- reduce ambigüedad

### 7.2 Coincidencia exacta de columnas
- evita alias
- evita lógica difusa
- facilita la validación estructural

### 7.3 Encabezados sin tildes ni símbolos
- reduce problemas de encoding
- simplifica comparaciones
- facilita interoperabilidad con APIs, JSON y SQL

### 7.4 Separación entre nombre visible y nombre interno
- mejora experiencia de usuario en Excel
- mantiene orden técnico interno
- permite evolucionar procesos sin exponer nombres técnicos

## 8. Política de procesamiento ante filas con error

En **`PQ_EXCEL_PROCESOS`**, el atributo **`PermiteProcesamientoParcial`** fija, **por proceso**, si la presencia de **al menos una fila con error** en el lote permite igualmente procesar las filas válidas restantes.

| `PermiteProcesamientoParcial` | Efecto |
|-------------------------------|--------|
| `0` (default) | Procesamiento final **bloqueado** mientras exista ≥ 1 fila con `TieneError = 1` o estado de fila erróneo. |
| `1` | Procesamiento final **permitido** sobre filas válidas; filas con error no se aplican al destino; lote puede quedar en `procesada_parcial`. |

Esta regla es **independiente por proceso**: dos procesos distintos pueden adoptar políticas distintas según criticidad del dato o conveniencia operativa.

Errores **estructurales** del archivo (previos al staging) no entran en esta política: el lote no alcanza estado `lista_para_procesar`.

**Casos borde:** si **todas** las filas tienen error → no se habilita procesamiento (aunque `PermiteProcesamientoParcial = 1`). Si **cero** filas tienen error → se procesa todo el lote; estado `procesada` (no `procesada_parcial`).

**Fila ajustada:** `FilaAjustadaAutomaticamente` por trim o limpieza de caracteres (ver documento conceptual §7); solo auditoría en BD, **sin indicación en UI** en esta etapa.

## 9. Decisiones operativas que impactan el modelo
- cada importación debe tener un identificador único
- el staging debe ser persistente
- el historial debe quedar disponible desde la primera etapa
- debe registrarse si una fila fue ajustada automáticamente
- debe registrarse el estado de cada lote y de cada fila
- debe registrarse la hoja elegida y el archivo original

## 10. Atributos de `PQ_EXCEL_PROCESOS_CAMPOS` usados en plantilla modelo

La exportación de plantilla (documento conceptual §12) consume:

| Atributo | Uso en plantilla |
|----------|------------------|
| `OrdenCampo` | Orden de columnas en fila 1 |
| `NombreColumnaExcel` | Texto visible del encabezado |
| `TipoDato` | Formato de columna y validación Excel |
| `LargoMaximo` | Validación de longitud (`texto`, `codigo`) |
| `CantidadDecimales` | Formato y validación `decimal` |
| `EsColumnaObligatoriaEstructural` | Línea `OBLIGATORIO` en comentario del encabezado |
| `Observaciones` | Texto adicional en comentario del encabezado |
| `Activo` | Solo `Activo = 1` se exporta |

En `PQ_EXCEL_PROCESOS`:

| Atributo | Uso en plantilla |
|----------|------------------|
| `GeneraPlantilla` | Si `0`, no se muestra botón ni endpoint de descarga |
| `FormatoBooleanoPlantilla` | Lista de validación para columnas `booleano` |
| `CodigoProceso` | Nombre sugerido del archivo `.xlsx` |

## 11. Nota de evolución futura

En etapas futuras podría analizarse:
- múltiples hojas por proceso
- archivos múltiples
- mayor detalle de auditoría por ajuste automático
- separación específica de errores en tabla independiente

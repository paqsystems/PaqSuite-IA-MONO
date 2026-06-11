# Especificación Técnica de Consultas Pivotables
**Proyecto:** Paqsuite  
**Documento:** Especificación técnica para Cursor  
**Versión:** 1.0  
**Fecha:** 2026-03-16

---

# 1. Objetivo

Definir el contrato técnico estándar que debe cumplir toda consulta pivotable del sistema, de forma que pueda ser interpretado de manera homogénea por backend, frontend, validaciones, exportación y persistencia de pivots guardados.

Este documento está orientado a:

- Cursor
- programadores backend
- programadores frontend
- responsables de integración
- validadores funcionales y técnicos

---

# 2. Principio de diseño

Toda consulta pivotable debe construirse sobre dos niveles:

## Nivel A — Definiciones globales reutilizables
Contiene reglas estándar del sistema:

- desagregaciones temporales
- formatos
- categorías
- agregaciones por tipo de métrica
- reglas de persistencia
- exportación estándar
- restricciones por defecto

## Nivel B — Definición específica por consulta
Declara únicamente:

- metadatos de la consulta
- fuente de datos
- pivot base
- campos habilitados
- filtros generales
- overrides o restricciones particulares

> Regla obligatoria: toda consulta pivotable debe usar primero las definiciones globales estándar y solo declarar overrides cuando exista una razón funcional concreta.

---

# 3. Formato sugerido de definición

Se recomienda definir una consulta pivotable en un archivo JSON por consulta.

Ejemplos:

```txt
ventas.pivot-definition.json
compras.pivot-definition.json
tareas.pivot-definition.json
```

También puede existir un archivo complementario legible por humanos:

```txt
ventas.pivot-definition.md
```

---

# 4. Estructura general obligatoria

Toda definición técnica de consulta pivotable debe seguir esta estructura raíz:

```json
{
  "consultaId": "ventas",
  "consultaNombre": "Ventas",
  "consultaDescripcion": "Consulta analítica de ventas",
  "pivotHabilitado": true,
  "version": 1,
  "origenDatos": {},
  "configuracionGeneral": {},
  "pivotBase": {},
  "campos": [],
  "filtrosGenerales": [],
  "exportacion": {},
  "persistencia": {},
  "restricciones": {}
}
```

---

# 5. Definiciones globales reutilizables

## 5.1 Plantilla temporal estándar

Todo campo fecha pivotable debe exponer, cuando corresponda, estas desagregaciones:

- Fecha
- Año
- Semestre
- Trimestre
- Mes
- Año-Mes
- Semana
- Día
- Día de semana

### Reglas
- mismos nombres visibles en todas las consultas
- mismo orden natural
- misma semántica
- mismo formato
- mismas posibilidades de uso en filas, columnas y filtros

---

## 5.2 Categorías estándar

Se recomienda usar estas categorías globales:

- Tiempo
- Organización
- Cliente / Proveedor
- Producto
- Comercial
- Estado
- Importes
- Cantidades
- Indicadores

---

## 5.3 Formatos estándar

Valores permitidos para `formato`:

- `texto`
- `numero`
- `moneda`
- `porcentaje`
- `fecha`
- `fechaHora`
- `booleano`

---

## 5.4 Agregaciones estándar por tipo de métrica

### Métricas monetarias
- `SUMA`
- `PROMEDIO`
- `MAX`
- `MIN`

### Métricas de cantidad
- `SUMA`
- `PROMEDIO`
- `MAX`
- `MIN`

### Conteos
- `CONTEO`
- `CONTEO_DISTINTO`

### Porcentajes
- `PROMEDIO`
- `MAX`
- `MIN`

### Precios unitarios
- `PROMEDIO`
- `MAX`
- `MIN`

> Regla: una consulta no debe redefinir estas agregaciones salvo excepción documentada.

---

# 6. Metadatos de la consulta

```json
{
  "consultaId": "ventas",
  "consultaNombre": "Ventas",
  "consultaDescripcion": "Consulta analítica de ventas por cliente, artículo, vendedor y período",
  "pivotHabilitado": true,
  "version": 1
}
```

## Reglas
- `consultaId`: identificador técnico único y estable
- `consultaNombre`: nombre funcional visible
- `consultaDescripcion`: descripción breve
- `pivotHabilitado`: habilita o no el pivot
- `version`: versión de definición de consulta

---

# 7. Origen de datos

```json
{
  "origenDatos": {
    "fuenteTipo": "view",
    "fuenteNombre": "VW_VENTAS_ANALITICA",
    "claveRegistro": "IdRegistro",
    "admiteDrillDown": true
  }
}
```

## Valores posibles
- `fuenteTipo`: `view`, `table`, `sp`, `api`
- `fuenteNombre`: nombre técnico de la fuente
- `claveRegistro`: clave técnica del registro base
- `admiteDrillDown`: indica si una celda puede abrir detalle

## Regla
La fuente debe ser analítica, consistente y apta para agregación.

---

# 8. Configuración general

```json
{
  "configuracionGeneral": {
    "nombrePivotBase": "Ventas por cliente por mes",
    "mostrarGrillaYPivot": true,
    "vistaInicial": "grilla",
    "permiteCambiarAVistaPivot": true,
    "camposAgrupadosPorCategoria": true,
    "mostrarSubtotalesPorDefecto": true,
    "mostrarTotalesGeneralesPorDefecto": true
  }
}
```

## Reglas
- `vistaInicial`: `grilla` o `pivot`
- `nombrePivotBase`: nombre funcional de la vista inicial
- `mostrarGrillaYPivot`: indica convivencia entre ambas vistas

---

# 9. Pivot base

```json
{
  "pivotBase": {
    "filas": ["Cliente"],
    "columnas": ["Mes"],
    "valores": [
      {
        "campo": "ImporteNeto",
        "agregacion": "SUMA"
      }
    ],
    "filtrosInternos": [],
    "ordenamiento": [
      {
        "campo": "Cliente",
        "direccion": "asc"
      }
    ],
    "mostrarSubtotales": true,
    "mostrarTotalesGenerales": true,
    "expandido": true
  }
}
```

## Reglas
- toda consulta pivotable debe tener exactamente una pivot base
- la pivot base no debe quedar vacía
- debe representar un caso útil y comprensible
- debe usar campos válidos y roles compatibles

---

# 10. Campos

Toda consulta pivotable debe declarar sus campos pivotables.

## Estructura estándar de campo

```json
{
  "campoId": "Cliente",
  "nombreTecnico": "CLI_NOMBRE",
  "nombreVisible": "Cliente",
  "descripcion": "Cliente asociado al comprobante",
  "tipoCampo": "dimension",
  "tipoDato": "string",
  "categoria": "Cliente / Proveedor",
  "rolesPermitidos": ["fila", "columna", "filtro"],
  "visible": true,
  "ordenSugerido": 10,
  "ordenNatural": null,
  "formato": "texto",
  "anchoSugerido": 180,
  "pivotable": true,
  "drillable": true
}
```

## Propiedades obligatorias
- `campoId`
- `nombreTecnico`
- `nombreVisible`
- `tipoCampo`
- `tipoDato`
- `categoria`
- `rolesPermitidos`
- `visible`
- `formato`
- `pivotable`

## Propiedades recomendadas
- `descripcion`
- `ordenSugerido`
- `ordenNatural`
- `anchoSugerido`
- `drillable`

---

# 11. Tipos de campo

Valores permitidos para `tipoCampo`:

- `dimension`
- `metrica`
- `atributo`

## Reglas
### `dimension`
Puede participar en:
- filas
- columnas
- filtros

### `metrica`
Puede participar en:
- valores

### `atributo`
Normalmente no debe participar en pivot.
Puede existir para grilla o drill-down.

---

# 12. Tipos de dato

Valores sugeridos para `tipoDato`:

- `string`
- `int`
- `decimal`
- `date`
- `datetime`
- `bool`

---

# 13. Roles permitidos

Valores permitidos para `rolesPermitidos`:

- `fila`
- `columna`
- `valor`
- `filtro`

## Regla
Un campo solo puede usarse en los roles declarados.

---

# 14. Métricas

Las métricas deben incorporar propiedades adicionales.

## Ejemplo

```json
{
  "campoId": "ImporteNeto",
  "nombreTecnico": "IMP_NETO",
  "nombreVisible": "Importe Neto",
  "descripcion": "Importe neto del comprobante",
  "tipoCampo": "metrica",
  "tipoDato": "decimal",
  "categoria": "Importes",
  "rolesPermitidos": ["valor"],
  "visible": true,
  "ordenSugerido": 100,
  "ordenNatural": null,
  "formato": "moneda",
  "anchoSugerido": 120,
  "pivotable": true,
  "drillable": false,
  "agregacionDefault": "SUMA",
  "agregacionesPermitidas": ["SUMA", "PROMEDIO", "MAX", "MIN"],
  "decimales": 2,
  "permiteTotales": true,
  "esMetricaPrincipal": true
}
```

## Propiedades obligatorias para métricas
- `agregacionDefault`
- `agregacionesPermitidas`

## Propiedades recomendadas
- `decimales`
- `permiteTotales`
- `esMetricaPrincipal`

---

# 15. Filtros generales

```json
{
  "filtrosGenerales": [
    {
      "campo": "Empresa",
      "obligatorio": true,
      "visible": true,
      "tipoControl": "select"
    },
    {
      "campo": "Fecha",
      "obligatorio": false,
      "visible": true,
      "tipoControl": "dateRange"
    }
  ]
}
```

## Valores posibles de `tipoControl`
- `select`
- `multiSelect`
- `text`
- `date`
- `dateRange`
- `numberRange`
- `check`

## Reglas
- los filtros generales afectan grilla y pivot
- una consulta puede exigir filtros obligatorios
- la validación debe ocurrir antes de ejecutar la consulta base

---

# 16. Exportación

```json
{
  "exportacion": {
    "excelBasicoHabilitado": true,
    "excelFormateadoHabilitado": true,
    "pdfHabilitado": false,
    "incluirFiltrosAplicados": true,
    "incluirMetadatos": true
  }
}
```

## Reglas
- `excelBasicoHabilitado`: exporta solo datos
- `excelFormateadoHabilitado`: exporta la estructura pivot
- `incluirFiltrosAplicados`: agrega metadatos de filtros
- `incluirMetadatos`: agrega usuario, fecha, consulta, pivot usado

---

# 17. Persistencia

```json
{
  "persistencia": {
    "permiteGuardar": true,
    "permiteGuardarComo": true,
    "permiteEliminar": true,
    "visibilidad": "global",
    "soloCreadorModifica": true,
    "soloCreadorElimina": true,
    "restaurarUltimoDiseno": true,
    "plantillaInicialPivotVacia": true
  }
}
```

## Reglas
- todos los pivots guardados son visibles para todos los usuarios con acceso a la consulta
- solo el creador modifica mediante `Guardar`
- cualquier usuario puede crear otro mediante `Guardar como`
- solo el creador elimina
- diseños propios (`isOwner`) muestran sufijo **` (*)`** en selector (i18n `pivotLayout.ownerMarker`); no se persiste en el nombre
- **Plantilla inicial** (`configId: null`) restablece pivot vacía (sin campos en áreas); i18n `pivotLayout.initialTemplate`
- con plantilla inicial activa, **Guardar** equivale a **Guardar como**
- al montar la pantalla, restaurar último diseño usado por el usuario si `restaurarUltimoDiseno` es true
- **`nombre` único por `consulta_id`** (paridad grilla); duplicado en Guardar como → `pivotLayout.duplicateName`

---

# 17.1 UI transversal del PivotGrid (paridad GEN-03)

Toda consulta pivotable con persistencia habilitada debe cumplir:

| # | Requisito | Referencia |
|---|-----------|------------|
| 1 | Sufijo ` (*)` en diseños propios | `pivotLayout.ownerMarker` |
| 2 | Plantilla inicial → pivot vacía | `configId: null`, reset field panel |
| 3 | Guardar desde plantilla inicial = Guardar como | POST con nombre |
| 4 | Ícono Actualizar en toolbar | `pivot.refresh`, `data-testid="pivotRefresh"`, re-fetch servidor |
| 5 | i18n completo DevExtreme | `patron-i18n-pivot-devextreme.md` |

Documento de implementación: `frontend-pivotgrid-devextreme-agregaciones-y-menu.md` §6–8.

---

# 18. Restricciones

```json
{
  "restricciones": {
    "maximoCamposEnFilas": 5,
    "maximoCamposEnColumnas": 3,
    "maximoMetricas": 5,
    "maximoRegistrosBase": 50000,
    "requiereFiltroPrevio": true,
    "camposObligatoriosEnFiltro": ["Empresa"],
    "bloquearSiExcedeVolumen": true,
    "permiteColumnasSinValores": false
  }
}
```

## Reglas
- las restricciones pueden ser globales o por consulta
- si se declara un override, debe justificarse funcionalmente
- el sistema debe impedir combinaciones inválidas antes de ejecutar el pivot

---

# 19. Validaciones mínimas obligatorias

Toda definición técnica debe validar:

1. que `consultaId` exista y sea único
2. que `pivotHabilitado = true` para permitir pivot
3. que exista `pivotBase`
4. que todos los campos referenciados en `pivotBase` existan en `campos`
5. que las métricas usen agregaciones permitidas
6. que cada campo tenga `nombreVisible`
7. que ningún nombre visible exponga nombres técnicos al usuario
8. que los filtros obligatorios estén declarados
9. que no existan roles incompatibles
10. que la definición sea compatible con la versión esperada

---

# 20. Compatibilidad y versionado

## Reglas
- toda consulta pivotable debe declarar `version`
- toda configuración guardada de pivot debe registrar la versión de definición usada al momento del guardado
- si cambia la definición, el sistema debe poder:
  - validar compatibilidad
  - migrar configuración si corresponde
  - informar incompatibilidades si no es posible reutilizarla

---

# 21. Ejemplo completo

```json
{
  "consultaId": "ventas",
  "consultaNombre": "Ventas",
  "consultaDescripcion": "Consulta analítica de ventas",
  "pivotHabilitado": true,
  "version": 1,
  "origenDatos": {
    "fuenteTipo": "view",
    "fuenteNombre": "VW_VENTAS_ANALITICA",
    "claveRegistro": "IdRegistro",
    "admiteDrillDown": true
  },
  "configuracionGeneral": {
    "nombrePivotBase": "Ventas por cliente por mes",
    "mostrarGrillaYPivot": true,
    "vistaInicial": "grilla",
    "permiteCambiarAVistaPivot": true,
    "camposAgrupadosPorCategoria": true,
    "mostrarSubtotalesPorDefecto": true,
    "mostrarTotalesGeneralesPorDefecto": true
  },
  "pivotBase": {
    "filas": ["Cliente"],
    "columnas": ["Mes"],
    "valores": [
      {
        "campo": "ImporteNeto",
        "agregacion": "SUMA"
      }
    ],
    "filtrosInternos": [],
    "ordenamiento": [
      {
        "campo": "Cliente",
        "direccion": "asc"
      }
    ],
    "mostrarSubtotales": true,
    "mostrarTotalesGenerales": true,
    "expandido": true
  },
  "campos": [
    {
      "campoId": "Cliente",
      "nombreTecnico": "CLI_NOMBRE",
      "nombreVisible": "Cliente",
      "descripcion": "Cliente asociado al comprobante",
      "tipoCampo": "dimension",
      "tipoDato": "string",
      "categoria": "Cliente / Proveedor",
      "rolesPermitidos": ["fila", "columna", "filtro"],
      "visible": true,
      "ordenSugerido": 10,
      "ordenNatural": null,
      "formato": "texto",
      "anchoSugerido": 180,
      "pivotable": true,
      "drillable": true
    },
    {
      "campoId": "Mes",
      "nombreTecnico": "MES_NOMBRE",
      "nombreVisible": "Mes",
      "descripcion": "Mes del comprobante",
      "tipoCampo": "dimension",
      "tipoDato": "string",
      "categoria": "Tiempo",
      "rolesPermitidos": ["fila", "columna", "filtro"],
      "visible": true,
      "ordenSugerido": 20,
      "ordenNatural": "calendarioMes",
      "formato": "texto",
      "anchoSugerido": 100,
      "pivotable": true,
      "drillable": false
    },
    {
      "campoId": "ImporteNeto",
      "nombreTecnico": "IMP_NETO",
      "nombreVisible": "Importe Neto",
      "descripcion": "Importe neto del comprobante",
      "tipoCampo": "metrica",
      "tipoDato": "decimal",
      "categoria": "Importes",
      "rolesPermitidos": ["valor"],
      "visible": true,
      "ordenSugerido": 100,
      "ordenNatural": null,
      "formato": "moneda",
      "anchoSugerido": 120,
      "pivotable": true,
      "drillable": false,
      "agregacionDefault": "SUMA",
      "agregacionesPermitidas": ["SUMA", "PROMEDIO", "MAX", "MIN"],
      "decimales": 2,
      "permiteTotales": true,
      "esMetricaPrincipal": true
    }
  ],
  "filtrosGenerales": [
    {
      "campo": "Empresa",
      "obligatorio": true,
      "visible": true,
      "tipoControl": "select"
    },
    {
      "campo": "Fecha",
      "obligatorio": false,
      "visible": true,
      "tipoControl": "dateRange"
    }
  ],
  "exportacion": {
    "excelBasicoHabilitado": true,
    "excelFormateadoHabilitado": true,
    "pdfHabilitado": false,
    "incluirFiltrosAplicados": true,
    "incluirMetadatos": true
  },
  "persistencia": {
    "permiteGuardar": true,
    "permiteGuardarComo": true,
    "permiteEliminar": true,
    "visibilidad": "global",
    "soloCreadorModifica": true,
    "soloCreadorElimina": true
  },
  "restricciones": {
    "maximoCamposEnFilas": 5,
    "maximoCamposEnColumnas": 3,
    "maximoMetricas": 5,
    "maximoRegistrosBase": 50000,
    "requiereFiltroPrevio": true,
    "camposObligatoriosEnFiltro": ["Empresa"],
    "bloquearSiExcedeVolumen": true,
    "permiteColumnasSinValores": false
  }
}
```

---

# 22. Resumen normativo

1. Toda consulta pivotable debe tener una definición formal.
2. Debe existir exactamente una pivot base.
3. Deben usarse reglas globales estándar antes que overrides locales.
4. Toda métrica debe declarar agregación por defecto y agregaciones permitidas.
5. Toda fecha pivotable debe poder usar la plantilla temporal estándar.
6. La definición debe ser versionable y compatible con pivots guardados.
7. Las restricciones deben validarse antes de ejecutar la consulta.
8. El usuario nunca debe ver nombres técnicos en la UI.
9. La UI del pivot debe alinear persistencia y toolbar con grillas: ` (*)` en propios, plantilla inicial → pivot vacía, Guardar = Guardar como, Actualizar e i18n DevExtreme.

---

# Fin del documento

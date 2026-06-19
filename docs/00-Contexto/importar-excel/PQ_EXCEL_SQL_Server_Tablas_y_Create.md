# PQ_EXCEL - Tablas SQL Server y comandos de creación

## Criterios generales adoptados

- Prefijo de tablas: `PQ_EXCEL_`
- Base objetivo: SQL Server
- Diseño multiusuario
- Staging persistente por lote
- Historial desde primera etapa
- Encabezado siempre en fila 1
- Procesos con diseño fijo por proceso
- Coincidencia exacta de encabezados
- Encabezados sin tildes ni símbolos especiales
- Validaciones de negocio fuera del importador genérico

---

## 1. Catálogos y configuración

### Tabla: `PQ_EXCEL_PROCESOS`

```sql
CREATE TABLE dbo.PQ_EXCEL_PROCESOS (
    IdProceso                INT IDENTITY(1,1) NOT NULL,
    CodigoProceso            VARCHAR(50) NOT NULL,
    NombreProceso            VARCHAR(150) NOT NULL,
    Descripcion              VARCHAR(500) NULL,
    NombreHojaDefault        VARCHAR(100) NULL,
    PermiteProcesamientoParcial BIT NOT NULL CONSTRAINT DF_PQ_EXCEL_PROCESOS_PermiteProcesamientoParcial DEFAULT (0),
    PermiteSoloValidar       BIT NOT NULL CONSTRAINT DF_PQ_EXCEL_PROCESOS_PermiteSoloValidar DEFAULT (1),
    GeneraPlantilla          BIT NOT NULL CONSTRAINT DF_PQ_EXCEL_PROCESOS_GeneraPlantilla DEFAULT (1),
    MantenerEspaciosEnBlancoDefault BIT NOT NULL CONSTRAINT DF_PQ_EXCEL_PROCESOS_MantenerEspacios DEFAULT (0),
    MantenerCaracteresEspecialesDefault BIT NOT NULL CONSTRAINT DF_PQ_EXCEL_PROCESOS_MantenerCaracteres DEFAULT (0),
    HandlerBackend           VARCHAR(200) NULL,
    Activo                   BIT NOT NULL CONSTRAINT DF_PQ_EXCEL_PROCESOS_Activo DEFAULT (1),
    FechaAlta                DATETIME2(0) NOT NULL CONSTRAINT DF_PQ_EXCEL_PROCESOS_FechaAlta DEFAULT (SYSDATETIME()),
    UsuarioAlta              VARCHAR(100) NOT NULL,
    FechaModificacion        DATETIME2(0) NULL,
    UsuarioModificacion      VARCHAR(100) NULL,
    CONSTRAINT PK_PQ_EXCEL_PROCESOS PRIMARY KEY CLUSTERED (IdProceso),
    CONSTRAINT UQ_PQ_EXCEL_PROCESOS_CodigoProceso UNIQUE (CodigoProceso)
);
GO
```

**`PermiteProcesamientoParcial`** (por proceso): si es `0` (default), la existencia de **≥ 1 fila con error** en staging **impide** el procesamiento final del lote. Si es `1`, el usuario puede procesar solo las filas válidas y el lote puede cerrar en `procesada_parcial`. No aplica a errores estructurales del archivo (columnas, encabezados, etc.).

### Tabla: `PQ_EXCEL_PROCESOS_CAMPOS`

```sql
CREATE TABLE dbo.PQ_EXCEL_PROCESOS_CAMPOS (
    IdProcesoCampo           INT IDENTITY(1,1) NOT NULL,
    IdProceso                INT NOT NULL,
    OrdenCampo               INT NOT NULL,
    NombreColumnaExcel       VARCHAR(150) NOT NULL,
    NombreCampoInterno       VARCHAR(100) NOT NULL,
    TipoDato                 VARCHAR(30) NOT NULL,
    LargoMaximo              INT NULL,
    CantidadDecimales        INT NULL,
    EsColumnaObligatoriaEstructural BIT NOT NULL CONSTRAINT DF_PQ_EXCEL_PROCESOS_CAMPOS_Obligatoria DEFAULT (0),
    EsCampoCodigo            BIT NOT NULL CONSTRAINT DF_PQ_EXCEL_PROCESOS_CAMPOS_EsCampoCodigo DEFAULT (0),
    Activo                   BIT NOT NULL CONSTRAINT DF_PQ_EXCEL_PROCESOS_CAMPOS_Activo DEFAULT (1),
    Observaciones            VARCHAR(500) NULL,
    CONSTRAINT PK_PQ_EXCEL_PROCESOS_CAMPOS PRIMARY KEY CLUSTERED (IdProcesoCampo),
    CONSTRAINT FK_PQ_EXCEL_PROCESOS_CAMPOS_Proceso FOREIGN KEY (IdProceso)
        REFERENCES dbo.PQ_EXCEL_PROCESOS (IdProceso),
    CONSTRAINT UQ_PQ_EXCEL_PROCESOS_CAMPOS_Proceso_NombreExcel UNIQUE (IdProceso, NombreColumnaExcel),
    CONSTRAINT UQ_PQ_EXCEL_PROCESOS_CAMPOS_Proceso_CampoInterno UNIQUE (IdProceso, NombreCampoInterno),
    CONSTRAINT CK_PQ_EXCEL_PROCESOS_CAMPOS_TipoDato CHECK (TipoDato IN ('texto','entero','decimal','fecha','booleano','codigo'))
);
GO

CREATE INDEX IX_PQ_EXCEL_PROCESOS_CAMPOS_IdProceso
    ON dbo.PQ_EXCEL_PROCESOS_CAMPOS (IdProceso, OrdenCampo);
GO
```

---

## 2. Lotes de importación

### Tabla: `PQ_EXCEL_IMPORTACIONES`

```sql
CREATE TABLE dbo.PQ_EXCEL_IMPORTACIONES (
    IdImportacion            BIGINT IDENTITY(1,1) NOT NULL,
    GuidImportacion          UNIQUEIDENTIFIER NOT NULL CONSTRAINT DF_PQ_EXCEL_IMPORTACIONES_Guid DEFAULT (NEWID()),
    IdProceso                INT NOT NULL,
    UsuarioEjecucion         VARCHAR(100) NOT NULL,
    TerminalEjecucion        VARCHAR(100) NULL,
    ArchivoOriginalNombre    VARCHAR(260) NOT NULL,
    ArchivoOriginalExtension VARCHAR(10) NOT NULL,
    HojaSeleccionada         VARCHAR(150) NOT NULL,
    MantenerEspaciosEnBlanco BIT NOT NULL CONSTRAINT DF_PQ_EXCEL_IMPORTACIONES_MantenerEspacios DEFAULT (0),
    MantenerCaracteresEspeciales BIT NOT NULL CONSTRAINT DF_PQ_EXCEL_IMPORTACIONES_MantenerCaracteres DEFAULT (0),
    EstadoImportacion        VARCHAR(30) NOT NULL,
    EsAsincronica            BIT NOT NULL CONSTRAINT DF_PQ_EXCEL_IMPORTACIONES_EsAsincronica DEFAULT (0),
    FechaInicio              DATETIME2(0) NOT NULL CONSTRAINT DF_PQ_EXCEL_IMPORTACIONES_FechaInicio DEFAULT (SYSDATETIME()),
    FechaFin                 DATETIME2(0) NULL,
    CantidadFilasLeidas      INT NOT NULL CONSTRAINT DF_PQ_EXCEL_IMPORTACIONES_FilasLeidas DEFAULT (0),
    CantidadFilasDescartadas INT NOT NULL CONSTRAINT DF_PQ_EXCEL_IMPORTACIONES_FilasDescartadas DEFAULT (0),
    CantidadFilasValidas     INT NOT NULL CONSTRAINT DF_PQ_EXCEL_IMPORTACIONES_FilasValidas DEFAULT (0),
    CantidadFilasConError    INT NOT NULL CONSTRAINT DF_PQ_EXCEL_IMPORTACIONES_FilasConError DEFAULT (0),
    CantidadFilasProcesadas  INT NOT NULL CONSTRAINT DF_PQ_EXCEL_IMPORTACIONES_FilasProcesadas DEFAULT (0),
    MensajeResultado         VARCHAR(1000) NULL,
    PuedeCancelar            BIT NOT NULL CONSTRAINT DF_PQ_EXCEL_IMPORTACIONES_PuedeCancelar DEFAULT (1),
    CONSTRAINT PK_PQ_EXCEL_IMPORTACIONES PRIMARY KEY CLUSTERED (IdImportacion),
    CONSTRAINT UQ_PQ_EXCEL_IMPORTACIONES_Guid UNIQUE (GuidImportacion),
    CONSTRAINT FK_PQ_EXCEL_IMPORTACIONES_Proceso FOREIGN KEY (IdProceso)
        REFERENCES dbo.PQ_EXCEL_PROCESOS (IdProceso),
    CONSTRAINT CK_PQ_EXCEL_IMPORTACIONES_Estado CHECK (EstadoImportacion IN (
        'pendiente',
        'validando',
        'validada',
        'con_error_estructura',
        'lista_para_procesar',
        'procesando',
        'procesada',
        'procesada_parcial',
        'cancelada'
    ))
);
GO

CREATE INDEX IX_PQ_EXCEL_IMPORTACIONES_Proceso_Fecha
    ON dbo.PQ_EXCEL_IMPORTACIONES (IdProceso, FechaInicio DESC);
GO

CREATE INDEX IX_PQ_EXCEL_IMPORTACIONES_Usuario_Fecha
    ON dbo.PQ_EXCEL_IMPORTACIONES (UsuarioEjecucion, FechaInicio DESC);
GO
```

---

## 3. Staging de filas

### Tabla: `PQ_EXCEL_IMPORTACIONES_FILAS`

```sql
CREATE TABLE dbo.PQ_EXCEL_IMPORTACIONES_FILAS (
    IdImportacionFila        BIGINT IDENTITY(1,1) NOT NULL,
    IdImportacion            BIGINT NOT NULL,
    NumeroFilaExcel          INT NOT NULL,
    EstadoFila               VARCHAR(20) NOT NULL,
    FilaAjustadaAutomaticamente BIT NOT NULL CONSTRAINT DF_PQ_EXCEL_IMPORTACIONES_FILAS_Ajustada DEFAULT (0),
    TieneError               BIT NOT NULL CONSTRAINT DF_PQ_EXCEL_IMPORTACIONES_FILAS_TieneError DEFAULT (0),
    ErrorImportacion         VARCHAR(MAX) NULL,
    DatosOriginalesJson      NVARCHAR(MAX) NULL,
    DatosNormalizadosJson    NVARCHAR(MAX) NULL,
    FechaAlta                DATETIME2(0) NOT NULL CONSTRAINT DF_PQ_EXCEL_IMPORTACIONES_FILAS_FechaAlta DEFAULT (SYSDATETIME()),
    CONSTRAINT PK_PQ_EXCEL_IMPORTACIONES_FILAS PRIMARY KEY CLUSTERED (IdImportacionFila),
    CONSTRAINT FK_PQ_EXCEL_IMPORTACIONES_FILAS_Importacion FOREIGN KEY (IdImportacion)
        REFERENCES dbo.PQ_EXCEL_IMPORTACIONES (IdImportacion),
    CONSTRAINT UQ_PQ_EXCEL_IMPORTACIONES_FILAS_Importacion_Fila UNIQUE (IdImportacion, NumeroFilaExcel),
    CONSTRAINT CK_PQ_EXCEL_IMPORTACIONES_FILAS_Estado CHECK (EstadoFila IN (
        'pendiente',
        'valida',
        'con_error',
        'procesada',
        'rechazada'
    ))
);
GO

CREATE INDEX IX_PQ_EXCEL_IMPORTACIONES_FILAS_IdImportacion
    ON dbo.PQ_EXCEL_IMPORTACIONES_FILAS (IdImportacion, NumeroFilaExcel);
GO

CREATE INDEX IX_PQ_EXCEL_IMPORTACIONES_FILAS_IdImportacion_Estado
    ON dbo.PQ_EXCEL_IMPORTACIONES_FILAS (IdImportacion, EstadoFila);
GO
```

---

## 4. Errores detallados por fila

> Aunque funcionalmente la grilla usa una sola columna concatenada, esta tabla permite auditoría más fina, soporte y trazabilidad.

### Tabla: `PQ_EXCEL_IMPORTACIONES_FILAS_ERRORES`

```sql
CREATE TABLE dbo.PQ_EXCEL_IMPORTACIONES_FILAS_ERRORES (
    IdFilaError              BIGINT IDENTITY(1,1) NOT NULL,
    IdImportacionFila        BIGINT NOT NULL,
    SecuenciaError           INT NOT NULL,
    CodigoError              VARCHAR(50) NULL,
    TipoError                VARCHAR(20) NOT NULL,
    NombreCampoInterno       VARCHAR(100) NULL,
    NombreColumnaExcel       VARCHAR(150) NULL,
    MensajeError             VARCHAR(1000) NOT NULL,
    CONSTRAINT PK_PQ_EXCEL_IMPORTACIONES_FILAS_ERRORES PRIMARY KEY CLUSTERED (IdFilaError),
    CONSTRAINT FK_PQ_EXCEL_IMPORTACIONES_FILAS_ERRORES_Fila FOREIGN KEY (IdImportacionFila)
        REFERENCES dbo.PQ_EXCEL_IMPORTACIONES_FILAS (IdImportacionFila),
    CONSTRAINT UQ_PQ_EXCEL_IMPORTACIONES_FILAS_ERRORES_Secuencia UNIQUE (IdImportacionFila, SecuenciaError),
    CONSTRAINT CK_PQ_EXCEL_IMPORTACIONES_FILAS_ERRORES_Tipo CHECK (TipoError IN ('estructura','formato','negocio','sistema'))
);
GO

CREATE INDEX IX_PQ_EXCEL_IMPORTACIONES_FILAS_ERRORES_IdImportacionFila
    ON dbo.PQ_EXCEL_IMPORTACIONES_FILAS_ERRORES (IdImportacionFila, SecuenciaError);
GO
```

---

## 5. Notificaciones internas del proceso

### Tabla: `PQ_EXCEL_IMPORTACIONES_NOTIFICACIONES`

```sql
CREATE TABLE dbo.PQ_EXCEL_IMPORTACIONES_NOTIFICACIONES (
    IdNotificacion           BIGINT IDENTITY(1,1) NOT NULL,
    IdImportacion            BIGINT NOT NULL,
    UsuarioDestino           VARCHAR(100) NOT NULL,
    TipoNotificacion         VARCHAR(30) NOT NULL,
    FechaGeneracion          DATETIME2(0) NOT NULL CONSTRAINT DF_PQ_EXCEL_IMPORTACIONES_NOTIF_Fecha DEFAULT (SYSDATETIME()),
    FechaLeida               DATETIME2(0) NULL,
    Titulo                   VARCHAR(200) NOT NULL,
    Mensaje                  VARCHAR(1000) NOT NULL,
    Leida                    BIT NOT NULL CONSTRAINT DF_PQ_EXCEL_IMPORTACIONES_NOTIF_Leida DEFAULT (0),
    CONSTRAINT PK_PQ_EXCEL_IMPORTACIONES_NOTIFICACIONES PRIMARY KEY CLUSTERED (IdNotificacion),
    CONSTRAINT FK_PQ_EXCEL_IMPORTACIONES_NOTIFICACIONES_Importacion FOREIGN KEY (IdImportacion)
        REFERENCES dbo.PQ_EXCEL_IMPORTACIONES (IdImportacion),
    CONSTRAINT CK_PQ_EXCEL_IMPORTACIONES_NOTIFICACIONES_Tipo CHECK (TipoNotificacion IN ('toast','bandeja','resultado'))
);
GO

CREATE INDEX IX_PQ_EXCEL_IMPORTACIONES_NOTIFICACIONES_Usuario_Leida
    ON dbo.PQ_EXCEL_IMPORTACIONES_NOTIFICACIONES (UsuarioDestino, Leida, FechaGeneracion DESC);
GO
```

---

## 6. Vista sugerida para historial

```sql
CREATE VIEW dbo.PQ_EXCEL_VW_HISTORIAL_IMPORTACIONES
AS
SELECT
    I.IdImportacion,
    I.GuidImportacion,
    P.CodigoProceso,
    P.NombreProceso,
    I.UsuarioEjecucion,
    I.TerminalEjecucion,
    I.ArchivoOriginalNombre,
    I.HojaSeleccionada,
    I.EstadoImportacion,
    I.EsAsincronica,
    I.FechaInicio,
    I.FechaFin,
    I.CantidadFilasLeidas,
    I.CantidadFilasDescartadas,
    I.CantidadFilasValidas,
    I.CantidadFilasConError,
    I.CantidadFilasProcesadas,
    I.MensajeResultado
FROM dbo.PQ_EXCEL_IMPORTACIONES I
INNER JOIN dbo.PQ_EXCEL_PROCESOS P
    ON P.IdProceso = I.IdProceso;
GO
```

---

## 7. Datos semilla sugeridos

### Ejemplo de alta de proceso

```sql
INSERT INTO dbo.PQ_EXCEL_PROCESOS (
    CodigoProceso,
    NombreProceso,
    Descripcion,
    NombreHojaDefault,
    PermiteProcesamientoParcial,
    PermiteSoloValidar,
    GeneraPlantilla,
    MantenerEspaciosEnBlancoDefault,
    MantenerCaracteresEspecialesDefault,
    HandlerBackend,
    Activo,
    UsuarioAlta
)
VALUES (
    'ARTICULOS_ALTA',
    'Importacion de Articulos',
    'Importacion de articulos desde plantilla Excel',
    NULL,
    0,
    1,
    1,
    0,
    0,
    'Importacion.Articulos.AltaHandler',
    1,
    'system'
);
GO
```

### Ejemplo de campos del proceso

```sql
DECLARE @IdProceso INT;

SELECT @IdProceso = IdProceso
FROM dbo.PQ_EXCEL_PROCESOS
WHERE CodigoProceso = 'ARTICULOS_ALTA';

INSERT INTO dbo.PQ_EXCEL_PROCESOS_CAMPOS (
    IdProceso,
    OrdenCampo,
    NombreColumnaExcel,
    NombreCampoInterno,
    TipoDato,
    LargoMaximo,
    CantidadDecimales,
    EsColumnaObligatoriaEstructural,
    EsCampoCodigo,
    Activo,
    Observaciones
)
VALUES
(@IdProceso, 1, 'Codigo',      'codigo',      'codigo',  50, NULL, 1, 1, 1, 'Debe venir como texto'),
(@IdProceso, 2, 'Descripcion', 'descripcion', 'texto', 255, NULL, 1, 0, 1, NULL),
(@IdProceso, 3, 'Rubro',       'rubro',       'texto', 100, NULL, 0, 0, 1, NULL),
(@IdProceso, 4, 'Precio',      'precio',      'decimal', NULL, 2, 0, 0, 1, NULL),
(@IdProceso, 5, 'Fecha Alta',  'fecha_alta',  'fecha', NULL, NULL, 0, 0, 1, NULL);
GO
```

---

## 8. Observaciones de diseño

- `PQ_EXCEL_PROCESOS` define la configuración fija por proceso, incluida **`PermiteProcesamientoParcial`** (§6.1 documento conceptual).
- `PQ_EXCEL_PROCESOS_CAMPOS` define la estructura esperada de cada plantilla y alimenta la **generación de plantilla modelo** (documento conceptual **§12**: encabezados, comentarios `OBLIGATORIO`/`Observaciones`, formato por `TipoDato`).
- `PQ_EXCEL_IMPORTACIONES` representa el lote o sesión de importación.
- `PQ_EXCEL_IMPORTACIONES_FILAS` guarda el staging por fila.
- `PQ_EXCEL_IMPORTACIONES_FILAS_ERRORES` permite auditoría detallada sin ensuciar la grilla.
- `PQ_EXCEL_IMPORTACIONES_NOTIFICACIONES` cubre bandeja interna y toast.
- La columna concatenada `ErrorImportacion` se mantiene en staging por practicidad de UI.
- Los JSON permiten flexibilidad en primera etapa y evitan sobrediseñar staging específico por proceso.

## 9. Evoluciones posibles

- tabla específica de archivos binarios o metadatos hash
- versionado de plantillas por proceso
- staging tipado por proceso
- reproceso de filas válidas
- archivado o purga por política de antigüedad

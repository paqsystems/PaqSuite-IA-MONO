# PQ_EXCEL - Diagrama Mermaid

```mermaid
erDiagram
    PQ_EXCEL_PROCESOS {
        int IdProceso PK
        varchar CodigoProceso UK
        varchar NombreProceso
        varchar Descripcion
        varchar NombreHojaDefault
        bit PermiteProcesamientoParcial
        bit PermiteSoloValidar
        bit GeneraPlantilla
        bit MantenerEspaciosEnBlancoDefault
        bit MantenerCaracteresEspecialesDefault
        varchar HandlerBackend
        bit Activo
        datetime FechaAlta
        varchar UsuarioAlta
        datetime FechaModificacion
        varchar UsuarioModificacion
    }

    PQ_EXCEL_PROCESOS_CAMPOS {
        int IdProcesoCampo PK
        int IdProceso FK
        int OrdenCampo
        varchar NombreColumnaExcel
        varchar NombreCampoInterno
        varchar TipoDato
        int LargoMaximo
        int CantidadDecimales
        bit EsColumnaObligatoriaEstructural
        bit EsCampoCodigo
        bit Activo
        varchar Observaciones
    }

    PQ_EXCEL_IMPORTACIONES {
        bigint IdImportacion PK
        uniqueidentifier GuidImportacion UK
        int IdProceso FK
        varchar UsuarioEjecucion
        varchar TerminalEjecucion
        varchar ArchivoOriginalNombre
        varchar ArchivoOriginalExtension
        varchar HojaSeleccionada
        bit MantenerEspaciosEnBlanco
        bit MantenerCaracteresEspeciales
        varchar EstadoImportacion
        bit EsAsincronica
        datetime FechaInicio
        datetime FechaFin
        int CantidadFilasLeidas
        int CantidadFilasDescartadas
        int CantidadFilasValidas
        int CantidadFilasConError
        int CantidadFilasProcesadas
        varchar MensajeResultado
        bit PuedeCancelar
    }

    PQ_EXCEL_IMPORTACIONES_FILAS {
        bigint IdImportacionFila PK
        bigint IdImportacion FK
        int NumeroFilaExcel
        varchar EstadoFila
        bit FilaAjustadaAutomaticamente
        bit TieneError
        varchar ErrorImportacion
        nvarchar DatosOriginalesJson
        nvarchar DatosNormalizadosJson
        datetime FechaAlta
    }

    PQ_EXCEL_IMPORTACIONES_FILAS_ERRORES {
        bigint IdFilaError PK
        bigint IdImportacionFila FK
        int SecuenciaError
        varchar CodigoError
        varchar TipoError
        varchar NombreCampoInterno
        varchar NombreColumnaExcel
        varchar MensajeError
    }

    PQ_EXCEL_IMPORTACIONES_NOTIFICACIONES {
        bigint IdNotificacion PK
        bigint IdImportacion FK
        varchar UsuarioDestino
        varchar TipoNotificacion
        datetime FechaGeneracion
        datetime FechaLeida
        varchar Titulo
        varchar Mensaje
        bit Leida
    }

    PQ_EXCEL_PROCESOS ||--o{ PQ_EXCEL_PROCESOS_CAMPOS : define
    PQ_EXCEL_PROCESOS ||--o{ PQ_EXCEL_IMPORTACIONES : ejecuta
    PQ_EXCEL_IMPORTACIONES ||--o{ PQ_EXCEL_IMPORTACIONES_FILAS : contiene
    PQ_EXCEL_IMPORTACIONES_FILAS ||--o{ PQ_EXCEL_IMPORTACIONES_FILAS_ERRORES : detalla
    PQ_EXCEL_IMPORTACIONES ||--o{ PQ_EXCEL_IMPORTACIONES_NOTIFICACIONES : genera
```

## Lectura del diagrama

- **`PermiteProcesamientoParcial`** en `PQ_EXCEL_PROCESOS`: por cada proceso, define si ≥ 1 fila con error permite o no procesar el resto de filas válidas (ver documento conceptual §6.1).
- Un proceso puede tener muchos campos definidos.
- Un proceso puede tener muchas importaciones ejecutadas.
- Cada importación tiene muchas filas en staging.
- Cada fila puede tener muchos errores detallados.
- Cada importación puede generar varias notificaciones internas.

## Sugerencia visual

Si después querés, puedo generarte una segunda versión del Mermaid:
- más ejecutiva y resumida
- o una más técnica, con foco en staging y auditoría.

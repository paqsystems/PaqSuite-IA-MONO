# Modelo de Datos del Subsistema de Emisión

Este diagrama representa una propuesta de modelo lógico para persistir:

- Procesos emisibles
- Schemas de dataset
- Reportes
- Plantillas de mail
- Variables permitidas
- Formatos de integración
- Claves de segmentación
- Configuración del proceso
- Historial de emisiones
- Detalle por segmento
- Transferencias/copias entre empresas

---

```mermaid
erDiagram

    PQ_OUT_PROCESOS {
        varchar IdProceso PK
        varchar Nombre
        varchar Descripcion
        varchar TipoProceso
        bit PermiteConsolidado
        bit PermiteSegmentado
        bit CanalSoportaImpresion
        bit CanalSoportaPdf
        bit CanalSoportaExcel
        bit CanalSoportaCsv
        bit CanalSoportaMail
        bit CanalSoportaZip
        bit CanalSoportaIntegracion
        bit Activo
    }

    PQ_OUT_DATASET_SCHEMAS {
        varchar IdDatasetSchema PK
        varchar IdProceso FK
        varchar Nombre
        varchar Descripcion
        varchar TipoEstructura
        nvarchar DefinicionJson
        bit Activo
    }

    PQ_OUT_PROCESO_SEGMENTOS {
        bigint IdProcesoSegmento PK
        varchar IdProceso FK
        varchar CodigoSegmento
        varchar Nombre
        varchar CampoClave
        varchar CampoDescripcion
        varchar CampoMail
        bit Activo
    }

    PQ_OUT_REPORTES {
        bigint IdReporte PK
        varchar IdProceso FK
        varchar IdDatasetSchema FK
        varchar CodigoReporte
        varchar Nombre
        varchar Descripcion
        varchar OrigenTipo
        bigint IdReporteOrigen FK
        bit EsEstandar
        bit EsPrincipal
        bit EsPrivado
        bit VisibleEnMobile
        varchar ModoMobile
        bit Activo
        varbinary LayoutDefinicion
        varchar LayoutMimeType
        varchar IdUsuarioCreador
        datetime FechaCreacion
        varchar IdUsuarioModificador
        datetime FechaModificacion
        datetime FechaEliminacion
        varchar IdUsuarioEliminador
    }

    PQ_OUT_REPORTE_PERMISOS {
        bigint IdReportePermiso PK
        bigint IdReporte FK
        varchar IdUsuario
        bit PuedeUsar
        bit PuedeEditar
        bit PuedeEliminar
        bit PuedeCompartir
    }

    PQ_OUT_MAIL_PLANTILLAS {
        bigint IdPlantillaMail PK
        varchar IdProceso FK
        varchar CodigoPlantilla
        varchar Nombre
        varchar Descripcion
        nvarchar AsuntoTemplate
        nvarchar BodyHtmlTemplate
        bit EsPrincipal
        bit EsEstandar
        bit VisibleEnMobile
        varchar ModoMobile
        bit Activo
        varchar IdUsuarioCreador
        datetime FechaCreacion
        varchar IdUsuarioModificador
        datetime FechaModificacion
    }

    PQ_OUT_MAIL_VARIABLES {
        bigint IdMailVariable PK
        varchar IdProceso FK
        varchar CodigoVariable
        varchar Nombre
        varchar Descripcion
        varchar EjemploValor
        bit Activo
    }

    PQ_OUT_FORMATOS_INTEGRACION {
        bigint IdFormatoIntegracion PK
        varchar IdProceso FK
        varchar CodigoFormato
        varchar Nombre
        varchar Descripcion
        varchar TipoArchivo
        varchar Extension
        varchar Encoding
        varchar GeneradorTipo
        varchar GeneradorReferencia
        bit EsPrincipal
        bit VisibleEnMobile
        varchar ModoMobile
        bit Activo
        varchar IdUsuarioCreador
        datetime FechaCreacion
        varchar IdUsuarioModificador
        datetime FechaModificacion
    }

    PQ_OUT_PROCESO_CONFIG {
        varchar IdProceso PK, FK
        bigint IdReportePrincipal FK
        bigint IdPlantillaMailPrincipal FK
        bigint IdFormatoIntegracionPrincipal FK
        bit RequiereVistaPrevia
        bit PermiteEnvioMasivo
        bit PermiteZip
        varchar DestinatarioModoDefault
        bit VisibleEnMobile
        varchar ModoMobile
    }

    PQ_OUT_EMISIONES {
        bigint IdEmision PK
        varchar IdProceso FK
        bigint IdReporte FK
        bigint IdPlantillaMail FK
        bigint IdFormatoIntegracion FK
        varchar ModoEmision
        varchar CanalSalida
        varchar CodigoSegmento
        int CantidadSegmentos
        varchar IdUsuarioEmisor
        datetime FechaEmision
        varchar Resultado
        nvarchar MensajeResultado
        varchar ArchivoGeneradoNombre
        nvarchar ArchivoGeneradoRuta
    }

    PQ_OUT_EMISIONES_DET {
        bigint IdEmisionDet PK
        bigint IdEmision FK
        varchar ValorClaveSegmento
        varchar DescripcionSegmento
        varchar MailDestino
        varchar Resultado
        nvarchar MensajeResultado
        varchar ArchivoGeneradoNombre
    }

    PQ_OUT_TRANSFERENCIAS {
        bigint IdTransferencia PK
        varchar TipoObjeto
        bigint IdObjetoOrigen
        varchar EmpresaOrigen
        varchar EmpresaDestino
        varchar IdUsuario
        datetime FechaTransferencia
        varchar Resultado
        nvarchar MensajeResultado
    }

    PQ_OUT_PROCESOS ||--o{ PQ_OUT_DATASET_SCHEMAS : define
    PQ_OUT_PROCESOS ||--o{ PQ_OUT_PROCESO_SEGMENTOS : permite
    PQ_OUT_PROCESOS ||--o{ PQ_OUT_REPORTES : tiene
    PQ_OUT_PROCESOS ||--o{ PQ_OUT_MAIL_PLANTILLAS : tiene
    PQ_OUT_PROCESOS ||--o{ PQ_OUT_MAIL_VARIABLES : expone
    PQ_OUT_PROCESOS ||--o{ PQ_OUT_FORMATOS_INTEGRACION : tiene
    PQ_OUT_PROCESOS ||--|| PQ_OUT_PROCESO_CONFIG : configura
    PQ_OUT_PROCESOS ||--o{ PQ_OUT_EMISIONES : registra

    PQ_OUT_DATASET_SCHEMAS ||--o{ PQ_OUT_REPORTES : soporta

    PQ_OUT_REPORTES ||--o{ PQ_OUT_REPORTE_PERMISOS : permisos
    PQ_OUT_REPORTES ||--o| PQ_OUT_REPORTES : origen_copia

    PQ_OUT_REPORTES ||--o{ PQ_OUT_EMISIONES : usado_en
    PQ_OUT_MAIL_PLANTILLAS ||--o{ PQ_OUT_EMISIONES : usado_en
    PQ_OUT_FORMATOS_INTEGRACION ||--o{ PQ_OUT_EMISIONES : usado_en

    PQ_OUT_EMISIONES ||--o{ PQ_OUT_EMISIONES_DET : detalle

    PQ_OUT_REPORTES o|--|| PQ_OUT_PROCESO_CONFIG : principal
    PQ_OUT_MAIL_PLANTILLAS o|--|| PQ_OUT_PROCESO_CONFIG : principal
    PQ_OUT_FORMATOS_INTEGRACION o|--|| PQ_OUT_PROCESO_CONFIG : principal
# C4 Nivel 2 – Sistema Completo con Subsistema de Emisión

Este diagrama representa la arquitectura general del sistema, integrando:

- Frontend Web
- Frontend Mobile
- Backend / API
- Dictionary DB
- Company DB
- Motor de Emisión
- Motor de Reportes
- Motor de Mail
- Motor de Integración
- Storage de archivos
- Sistemas externos

El objetivo es mostrar cómo se ubica el subsistema de emisión dentro de la arquitectura general del ERP.

---

```mermaid
flowchart LR

    %% Actores
    USER[Usuario]
    ADMIN[Administrador / Diseñador]
    EXT[Aplicaciones Externas<br/>Bancos / AFIP / Otros Sistemas]
    SMTP[Servidor de Correo]

    %% Frontends
    subgraph FRONT["Frontends"]
        WEB[Frontend Web<br/>Desktop / Tablet]
        MOB[Frontend Mobile<br/>Responsive Web]
    end

    %% Backend principal
    subgraph BACK["Backend / API ERP"]
        AUTH[Autenticación y Autorización]
        PROC[Procesos de Negocio]
        DATASET[Resolución de Dataset por Proceso]
        OUTAPI[API de Emisión]
    end

    %% Subsistema de emisión
    subgraph EMIT["Subsistema de Emisión"]
        ORCH[Orquestador de Emisión]
        RULES[Reglas de Emisión]
        DEST[Resolución de Destinatarios]
        REPENG[Report Engine<br/>DevExtreme]
        EXPENG[Export Engine<br/>Excel / CSV]
        MAILENG[Mail Engine]
        INTENG[Integration File Engine<br/>TXT / CSV / XLSX específicos]
        ZIPENG[Zip / Batch Engine]
        AUDIT[Audit / History Engine]
    end

    %% Persistencia
    subgraph DATA["Persistencia"]
        DDB[(Dictionary DB)]
        CDB[(Company DB)]
        FILES[(File Storage / Temporales / Adjuntos)]
    end

    %% Relaciones actores
    USER --> WEB
    USER --> MOB
    ADMIN --> WEB

    %% Relaciones front-back
    WEB --> AUTH
    WEB --> PROC
    WEB --> OUTAPI

    MOB --> AUTH
    MOB --> PROC
    MOB --> OUTAPI

    %% Backend core
    PROC --> DATASET
    OUTAPI --> ORCH
    DATASET --> ORCH
    AUTH --> ORCH

    %% Reglas y resolución
    ORCH --> RULES
    ORCH --> DEST

    %% Motores
    ORCH --> REPENG
    ORCH --> EXPENG
    ORCH --> MAILENG
    ORCH --> INTENG
    ORCH --> ZIPENG
    ORCH --> AUDIT

    %% DBs
    AUTH --> DDB
    PROC --> DDB
    PROC --> CDB
    DATASET --> CDB
    RULES --> CDB
    DEST --> CDB
    AUDIT --> CDB

    %% Archivos
    REPENG --> FILES
    EXPENG --> FILES
    INTENG --> FILES
    ZIPENG --> FILES
    MAILENG --> FILES

    %% Externos
    MAILENG --> SMTP
    INTENG --> EXT

    %% Lectura de configuración
    ORCH --> CDB
    ORCH --> DDB
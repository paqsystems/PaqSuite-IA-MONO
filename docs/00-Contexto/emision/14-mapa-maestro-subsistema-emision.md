# 14-mapa-maestro-subsistema-emision

Este diagrama representa el **Mapa Maestro del Subsistema de Emisión**.

Integra en una sola vista:

- procesos
- datasets
- reglas de emisión
- reportes
- plantillas de mail
- formatos de integración
- modos de emisión
- canales de salida
- desktop vs mobile
- historial y auditoría
- almacenamiento
- sistemas externos

Su objetivo es servir como **GPS arquitectónico** del subsistema para análisis, desarrollo y mantenimiento.

---

```mermaid
flowchart TB

    %% =========================
    %% ACTORES Y CANALES
    %% =========================
    U[Usuario Operativo]
    A[Administrador / Diseñador]
    X[Aplicaciones Externas<br/>Bancos / AFIP / Otros]
    S[Servidor SMTP]

    U --> WEB
    U --> MOB
    A --> WEB

    %% =========================
    %% FRONTENDS
    %% =========================
    subgraph FRONT["Frontends"]
        WEB[Frontend Web Desktop / Tablet]
        MOB[Frontend Mobile Responsivo]
    end

    %% =========================
    %% EXPERIENCIA DESKTOP / MOBILE
    %% =========================
    subgraph UX["Experiencia de Uso"]
        WEB --> WEB_EMIT[Emitir información]
        WEB --> WEB_REP[Diseñar / Administrar reportes]
        WEB --> WEB_MAIL[Diseñar / Administrar plantillas mail]
        WEB --> WEB_CFG[Administración avanzada]
        WEB --> WEB_AUD[Consulta de historial y auditoría]

        MOB --> MOB_EMIT[Emitir información simplificada]
        MOB --> MOB_VIEW[Ver / Compartir PDF]
        MOB --> MOB_MAIL[Enviar mail]
        MOB --> MOB_REPRINT[Reimprimir / Reenviar]
        MOB --> MOB_INT[Generar archivo integración simple]

        MOB -. no soportado .-> MOB_NO_REP[Diseño reportes]
        MOB -. no soportado .-> MOB_NO_MAIL[Diseño plantillas]
        MOB -. no soportado .-> MOB_NO_CFG[Administración avanzada]
    end

    %% =========================
    %% BACKEND ERP
    %% =========================
    subgraph ERP["Backend / API ERP"]
        AUTH[Autenticación / Autorización]
        PROC[Procesos de negocio]
        API_OUT[API de Emisión]
        DATASET_RES[Resolución de Dataset por Proceso]
    end

    WEB_EMIT --> API_OUT
    WEB_REP --> API_OUT
    WEB_MAIL --> API_OUT
    WEB_CFG --> API_OUT
    WEB_AUD --> API_OUT

    MOB_EMIT --> API_OUT
    MOB_VIEW --> API_OUT
    MOB_MAIL --> API_OUT
    MOB_REPRINT --> API_OUT
    MOB_INT --> API_OUT

    WEB --> AUTH
    MOB --> AUTH

    PROC --> DATASET_RES
    API_OUT --> EMIT

    %% =========================
    %% SUBSISTEMA DE EMISIÓN
    %% =========================
    subgraph EMIT["Subsistema de Emisión"]
        ORCH[Orquestador de Emisión]
        RULES[Reglas del Proceso]
        DEST[Resolución de Destinatarios]
        MODE[Resolución de Modo de Emisión]
        CHAN[Resolución de Canal de Salida]
        MOBILE_RULES[Políticas de Mobile]
        AUD[Motor de Auditoría]
    end

    API_OUT --> ORCH
    DATASET_RES --> ORCH
    AUTH --> ORCH

    ORCH --> RULES
    ORCH --> DEST
    ORCH --> MODE
    ORCH --> CHAN
    ORCH --> MOBILE_RULES
    ORCH --> AUD

    %% =========================
    %% NÚCLEO FUNCIONAL
    %% =========================
    subgraph CORE["Núcleo Funcional del Subsistema"]
        P[Proceso]
        DS[Dataset Schema]
        SEG[Claves de Segmentación]
        CFG[Configuración del Proceso]
        REP[Reportes]
        MAILTPL[Plantillas de Mail]
        VARS[Variables de Mail]
        INTFMT[Formatos de Integración]
    end

    RULES --> P
    RULES --> CFG
    DATASET_RES --> DS
    RULES --> SEG
    RULES --> REP
    RULES --> MAILTPL
    RULES --> INTFMT
    MAILTPL --> VARS

    P --> DS
    P --> REP
    P --> MAILTPL
    P --> INTFMT
    P --> SEG
    P --> CFG

    %% =========================
    %% MODOS DE EMISIÓN
    %% =========================
    subgraph MODES["Modos de Emisión"]
        CONS[Consolidado]
        SPLIT[Segmentado por clave]
    end

    MODE --> CONS
    MODE --> SPLIT
    SPLIT --> SEG

    %% =========================
    %% CANALES DE SALIDA
    %% =========================
    subgraph CHANNELS["Canales de Salida"]
        C_PRINT[Print]
        C_PDF[PDF]
        C_EXCEL[Excel]
        C_CSV[CSV]
        C_MAIL[Mail]
        C_ZIP[ZIP / Lote]
        C_INT[Archivo de Integración]
    end

    CHAN --> C_PRINT
    CHAN --> C_PDF
    CHAN --> C_EXCEL
    CHAN --> C_CSV
    CHAN --> C_MAIL
    CHAN --> C_ZIP
    CHAN --> C_INT

    %% =========================
    %% MOTORES ESPECIALIZADOS
    %% =========================
    subgraph ENGINES["Motores Especializados"]
        REPENG[DevExtreme Report Engine]
        EXPENG[Export Engine]
        MAILENG[Mail Engine]
        ZIPENG[Zip / Batch Engine]
        INTENG[Integration File Engine]
    end

    C_PRINT --> REPENG
    C_PDF --> REPENG
    C_EXCEL --> EXPENG
    C_CSV --> EXPENG
    C_MAIL --> MAILENG
    C_ZIP --> ZIPENG
    C_INT --> INTENG

    REP --> REPENG
    MAILTPL --> MAILENG
    INTFMT --> INTENG

    %% =========================
    %% LÓGICA MAIL
    %% =========================
    subgraph MAILLOGIC["Lógica de Envío por Mail"]
        DM1[Destino manual]
        DM2[Destino fijo parametrizado]
        DM3[Mail único desde dataset]
        DM4[Mail resuelto por regla]
        DM5[Un mail por entidad]
        PDFDOC[PDF adjunto como documento]
        BODY[Mail como comunicación]
    end

    DEST --> DM1
    DEST --> DM2
    DEST --> DM3
    DEST --> DM4
    DEST --> DM5

    MAILENG --> PDFDOC
    MAILENG --> BODY
    REPENG --> PDFDOC
    MAILENG --> S

    %% =========================
    %% EMISIÓN SEGMENTADA
    %% =========================
    subgraph SPLITLOGIC["Lógica de Emisión Segmentada"]
        GROUP[Grouping por clave]
        DOCS[Generar salida por segmento]
        ZIPLOT[Armar lote ZIP]
        MAILLOT[Enviar por entidad]
        DETAUD[Registrar detalle por segmento]
    end

    SPLIT --> GROUP
    GROUP --> DOCS
    DOCS --> ZIPLOT
    DOCS --> MAILLOT
    DOCS --> DETAUD

    ZIPLOT --> ZIPENG
    MAILLOT --> MAILENG
    DETAUD --> AUD

    %% =========================
    %% SALIDAS DE PRESENTACIÓN VS INTEGRACIÓN
    %% =========================
    subgraph OUTPUT_TYPES["Familias de Salida"]
        PRESENT[Salidas de Presentación]
        INTEG[Salidas de Integración]
    end

    PRESENT --> C_PRINT
    PRESENT --> C_PDF
    PRESENT --> C_EXCEL
    PRESENT --> C_CSV
    PRESENT --> C_MAIL
    PRESENT --> C_ZIP

    INTEG --> C_INT

    %% =========================
    %% REGLAS DE DISEÑO
    %% =========================
    subgraph GOVERNANCE["Reglas de Diseño y Gobierno"]
        G1[Reportes diseñables por usuario autorizado]
        G2[Plantillas mail diseñables por usuario autorizado]
        G3[Formatos integración NO diseñables por usuario]
        G4[Mail = comunicación]
        G5[PDF = documento]
        G6[Mobile ejecuta]
        G7[Desktop administra y diseña]
    end

    WEB_REP --> G1
    WEB_MAIL --> G2
    WEB_CFG --> G3

    G1 --> REP
    G2 --> MAILTPL
    G3 --> INTFMT
    G4 --> MAILENG
    G5 --> REPENG
    G6 --> MOBILE_RULES
    G7 --> WEB

    %% =========================
    %% PERSISTENCIA
    %% =========================
    subgraph DATA["Persistencia"]
        DDB[(Dictionary DB)]
        CDB[(Company DB)]
        FS[(File Storage)]
    end

    AUTH --> DDB
    PROC --> DDB

    PROC --> CDB
    RULES --> CDB
    DEST --> CDB
    CFG --> CDB
    REP --> CDB
    MAILTPL --> CDB
    INTFMT --> CDB
    AUD --> CDB

    REPENG --> FS
    EXPENG --> FS
    ZIPENG --> FS
    INTENG --> FS
    MAILENG --> FS

    %% =========================
    %% HISTORIAL Y AUDITORÍA
    %% =========================
    subgraph HISTORY["Historial y Auditoría"]
        H1[Registrar emisión]
        H2[Registrar usuario / fecha]
        H3[Registrar reporte usado]
        H4[Registrar plantilla usada]
        H5[Registrar formato integración usado]
        H6[Registrar archivo generado]
        H7[Registrar resultado]
        H8[Registrar detalle por segmento]
    end

    AUD --> H1
    AUD --> H2
    AUD --> H3
    AUD --> H4
    AUD --> H5
    AUD --> H6
    AUD --> H7
    AUD --> H8

    H1 --> CDB
    H2 --> CDB
    H3 --> CDB
    H4 --> CDB
    H5 --> CDB
    H6 --> CDB
    H7 --> CDB
    H8 --> CDB

    %% =========================
    %% INTEGRACIONES EXTERNAS
    %% =========================
    INTENG --> X
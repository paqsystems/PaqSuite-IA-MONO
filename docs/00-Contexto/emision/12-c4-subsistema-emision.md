# C4 - Subsistema emisión
---

## Versión con foco más “C4 narrativo”
Si querés acompañarlo con un texto abajo, te dejo este bloque:

## Descripción de contenedores

### Frontend Web
Interfaz principal para operación administrativa, diseño de reportes, diseño de plantillas y administración avanzada.

### Frontend Mobile
Interfaz responsiva simplificada para ejecución rápida de procesos de emisión, reimpresión, envío por mail y generación simple de archivos.

### Backend / API ERP
Expone los procesos de negocio, resuelve autenticación, datasets por proceso y delega la emisión al subsistema especializado.

### Subsistema de Emisión
Orquesta todos los tipos de salida del sistema:
- documentales
- exportables
- mails
- archivos técnicos de integración
- emisiones segmentadas
- lotes ZIP
- auditoría

### Dictionary DB
Contiene metadatos globales del sistema, definición de procesos, menús, permisos y configuraciones comunes.

### Company DB
Contiene parametrización operativa por empresa:
- reportes
- plantillas de mail
- formatos de integración
- configuraciones del proceso
- historial de emisiones

### File Storage
Almacena archivos generados temporal o permanentemente:
- PDFs
- Excels
- ZIPs
- TXT/CSV/XLSX de integración
- adjuntos de mail

### Sistemas Externos
Aplicaciones destino de archivos técnicos:
- bancos
- AFIP
- otros sistemas administrativos o financieros

### Servidor de Correo
Responsable del envío efectivo de mails emitidos por el sistema.

# C4 Complementario – Foco en Subsistema de Emisión

```mermaid
flowchart TB

    ERP[ERP Backend] --> OUTAPI[API Emisión]

    subgraph EMIT["Subsistema de Emisión"]
        ORCH[Orquestador]
        RULES[Reglas]
        DEST[Destinatarios]
        REP[DevExtreme Reports]
        EXP[Export Engine]
        MAIL[Mail Engine]
        INT[Integration Engine]
        ZIP[Zip Engine]
        AUD[Audit Engine]
    end

    OUTAPI --> ORCH
    ORCH --> RULES
    ORCH --> DEST
    ORCH --> REP
    ORCH --> EXP
    ORCH --> MAIL
    ORCH --> INT
    ORCH --> ZIP
    ORCH --> AUD

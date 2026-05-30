# Estrategias de Salida

El motor utiliza un patrón de estrategia según el tipo de salida.

---

```mermaid
flowchart TB

EM[Motor de Emisión]

EM --> TS{Tipo de salida}


TS -->|PRINT| R1[ReportOutputStrategy]
TS -->|PDF| R1

TS -->|EXCEL| R2[ExportOutputStrategy]
TS -->|CSV| R2

TS -->|MAIL| R3[MailOutputStrategy]

TS -->|ZIP| R4[BatchZipOutputStrategy]

TS -->|INTEGRATION_FILE| R5[IntegrationFileOutputStrategy]


R1 --> DX[DevExtreme Engine]

R2 --> EX[Export Engine]

R3 --> ML[Mail Engine]
R3 --> DX

R4 --> ZIP[Zip Engine]
R4 --> DX
R4 --> EX
R4 --> IF[Integration File Engine]

R5 --> IF


DX --> OUT[Archivo final]
EX --> OUT
ML --> OUT
ZIP --> OUT
IF --> OUT

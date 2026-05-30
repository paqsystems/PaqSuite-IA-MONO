# Arquitectura General del Motor de Emisión

El subsistema de emisión permite generar salidas desde cualquier proceso del sistema.

Cada proceso define:

- Dataset
- Reportes disponibles
- Plantillas de mail
- Formatos de integración
- Modos de emisión (consolidado / segmentado)
- Canales de salida
- Comportamiento en mobile

La arquitectura utiliza un **Motor de Emisión central** que orquesta motores especializados.

---

```mermaid
flowchart LR

    U[Usuario] --> W[Frontend Web]
    U --> M[Frontend Mobile]

    W --> API[Backend API]
    M --> API

    API --> EM[Motor de Emisión]

    EM --> REP[Report Engine<br/>DevExtreme]
    EM --> EXP[Export Engine<br/>Excel/CSV]
    EM --> MAIL[Mail Engine]
    EM --> INT[Integration File Engine]
    EM --> ZIP[Zip/Lote Engine]
    EM --> AUD[Audit / History Engine]

    API --> CFG[(Company DB)]
    API --> DIC[(Dictionary DB)]

    REP --> FS[(File Storage)]
    EXP --> FS
    INT --> FS
    ZIP --> FS

    AUD --> CFG
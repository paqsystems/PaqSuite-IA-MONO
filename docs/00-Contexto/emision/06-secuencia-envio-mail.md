# Secuencia de Envío por Mail

---

```mermaid
sequenceDiagram

participant U as Usuario
participant UI as Frontend
participant API as Backend
participant EM as Motor Emisión
participant REP as DevExtreme Engine
participant MAIL as Mail Engine
participant AUD as Auditoría

U->>UI: Emitir por mail

UI->>API: proceso + reporte + plantilla

API->>EM: solicitar emisión

EM->>REP: generar PDF

REP-->>EM: PDF generado

EM->>MAIL: enviar mail + adjunto

MAIL-->>EM: resultado envío

EM->>AUD: registrar emisión

EM-->>API: resultado

API-->>UI: confirmación
# Secuencia Generación Archivo de Integración

---

```mermaid
sequenceDiagram

participant U as Usuario
participant UI as Frontend
participant API as Backend
participant EM as Motor Emisión
participant INT as Integration Engine
participant AUD as Auditoría

U->>UI: Generar archivo integración

UI->>API: proceso + formato

API->>EM: solicitar generación

EM->>INT: construir archivo

INT-->>EM: archivo TXT/CSV/XLSX

EM->>AUD: registrar generación

EM-->>API: archivo generado

API-->>UI: descarga archivo
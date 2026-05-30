# Flujo General de Emisión

El flujo describe cómo se genera cualquier salida del sistema.

---

```mermaid
flowchart TD

A[Usuario inicia emisión]

A --> B[Seleccionar proceso]

B --> C[Resolver dataset]

C --> D[Validar reglas del proceso]

D --> E{Modo de emisión}

E -->|Consolidado| F[Salida única]

E -->|Segmentado| G[Agrupar por entidad]

G --> H[Generar salida por segmento]

F --> I[Seleccionar canal]
H --> I

I --> J{Tipo de salida}

J -->|PDF / Print| K[Motor DevExtreme]

J -->|Excel / CSV| L[Motor Export]

J -->|Mail| M[Motor Mail + PDF]

J -->|ZIP| N[Motor Lote]

J -->|Archivo Integración| O[Motor Integración]

K --> P[Archivo final]
L --> P
M --> P
N --> P
O --> P

P --> Q[Registrar auditoría]

Q --> R[Resultado al usuario]
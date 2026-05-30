# 10-matriz-decisiones-funcionales-subsistema-emision

Este documento define una **matriz de decisiones funcionales** para el subsistema de emisión.

Su objetivo es evitar ambigüedades al diseñar o programar:

- qué tipo de salida corresponde en cada caso
- qué opciones son configurables por usuario
- qué opciones son solo técnicas/programadas
- qué capacidades se permiten en mobile
- qué capacidades quedan limitadas a desktop
- cuándo usar reportes DevExtreme y cuándo no
- cuándo corresponde emisión consolidada o segmentada

---

## 1. Principios rectores

### 1.1. Centralidad del proceso
Toda emisión se define **por proceso**.

Cada proceso debe declarar:

- dataset
- modos de emisión soportados
- canales de salida soportados
- reportes disponibles
- plantillas de mail disponibles
- formatos de integración disponibles
- comportamiento mobile

---

### 1.2. Separación entre presentación e integración
Existen dos familias de salida:

#### A. Salidas de presentación
Pensadas para lectura humana.

Ejemplos:
- impresión
- PDF
- Excel
- CSV
- mail con PDF adjunto
- ZIP documental

Estas salidas pueden usar:
- reportes DevExtreme
- exportadores genéricos
- plantillas de mail

#### B. Salidas de integración
Pensadas para otras aplicaciones o plataformas.

Ejemplos:
- TXT para banco
- CSV con formato exacto
- XLSX para importación externa
- interfaces contables / financieras

Estas salidas:
- **no** son reportes DevExtreme
- **no** son diseñables por usuario final
- requieren lógica programada específica

---

### 1.3. El mail es comunicación, el PDF es documento
En el envío por mail:

- el cuerpo del mail debe ser breve y parametrizable
- la información formal debe viajar en un PDF adjunto
- el cuerpo del mail **no** debe contener datasets tabulares extensos

Regla base:
- **Mail = comunicación**
- **PDF = documento**

---

### 1.4. Mobile ejecuta; Desktop administra
En mobile se privilegia:

- rapidez
- simplicidad
- uso operacional

En desktop se concentran:

- diseño de reportes
- diseño de plantillas
- administración avanzada
- parametrización técnica
- análisis de lotes y auditoría extensa

---

## 2. Matriz principal de decisiones

| Tema                         | Decisión |
|------------------------------|----------|
| Unidad funcional base        | Proceso |
| Dataset                      | Definido por proceso |
| Reportes                     | Asociados a proceso y dataset |
| Reporte habitual             | Definido por proceso |
| Plantilla mail habitual      | Definida por proceso |
| Formato integración habitual | Definido por proceso |
| Modo de emisión              | Consolidado y/o segmentado por clave |
| Motor presentación           | DevExtreme + exportadores |
| Motor integración            | Generadores programados |
| Mobile                       | Ejecución simplificada |
| Desktop                      | Ejecución + diseño + administración |

---

## 3. Matriz por tipo de salida

| Tipo de salida | Usa DevExtreme | Diseñable por usuario | Requiere plantilla mail | Soporta mobile | Observaciones |
|---|---:|---:|---:|---:|---|
| Impresión | Sí | Sí | No | Parcial | En mobile suele resolverse como PDF + compartir/imprimir |
| PDF | Sí | Sí | No | Sí | Es la salida documental principal |
| Excel genérico | No necesariamente | No en general | No | Parcial | Puede salir por exportador |
| CSV genérico | No necesariamente | No | No | Parcial | Exportación simple |
| Mail con PDF adjunto | Sí para el PDF | Plantilla sí, reporte sí | Sí | Sí | El cuerpo del mail no reemplaza al documento |
| ZIP de salidas | Depende del contenido | No | No | Parcial | Útil para emisión segmentada |
| Archivo integración TXT | No | No | No | Parcial | Programado específicamente |
| Archivo integración CSV | No | No | No | Parcial | No confundir con CSV genérico |
| Archivo integración XLSX | No | No | No | Parcial | Formato exacto para terceros |

---

## 4. Matriz sobre configurabilidad

### 4.1. Elementos configurables por usuario autorizado

| Elemento | Configurable por usuario | Observaciones |
|---|---:|---|
| Reporte DevExtreme | Sí | Solo si tiene permisos |
| Copia de reporte | Sí | Muy recomendado |
| Reporte principal | Sí | Uno por proceso |
| Plantilla de mail | Sí | Solo por usuarios autorizados |
| Asunto de mail | Sí | Con placeholders simples |
| Cuerpo HTML de mail | Sí | Con placeholders simples |
| Plantilla principal | Sí | Una por proceso |
| Selección de reporte al emitir | Sí | Preseleccionar habitual |
| Selección de plantilla al emitir | Sí | Si el proceso lo permite |

### 4.2. Elementos NO configurables por usuario final

| Elemento | Configurable por usuario | Motivo |
|---|---:|---|
| Formato bancario TXT | No | Requiere exactitud técnica |
| Formato integración CSV específico | No | Riesgo alto de incompatibilidad |
| Formato integración XLSX exacto | No | Debe responder al sistema destino |
| Reglas de armado del archivo integración | No | Deben programarse |
| Validaciones técnicas de integración | No | Deben ser controladas por sistema |

---

## 5. Matriz de emisión consolidada vs segmentada

| Criterio | Consolidada | Segmentada |
|---|---|---|
| Cantidad de salidas | Una | Una por entidad/clave |
| Uso típico | Gestión interna, auditoría, supervisión | Comunicación externa o sectorizada |
| Ejemplo | Listado total de deuda | Un resumen por cliente |
| Mail | Un solo mail | Un mail por entidad |
| PDF | Un solo PDF | Un PDF por entidad |
| ZIP | Opcional | Muy útil |
| Historial | Un registro general | Registro general + detalle por segmento |

### Regla funcional
Usar **emisión consolidada** cuando:
- el destinatario es único
- el objetivo es análisis interno
- no se requiere separar por entidad

Usar **emisión segmentada** cuando:
- cada entidad debe recibir su propio documento
- se debe generar un archivo por cliente/proveedor/empleado
- se necesita envío individualizado

---

## 6. Matriz de resolución de destinatarios

| Modo destinatario | Descripción | Caso de uso |
|---|---|---|
| Manual | Usuario ingresa el mail | Emisión puntual |
| Fijo parametrizado | Mail fijo definido para el proceso | Áreas internas |
| Dataset único | El dataset trae un único mail | Factura o recibo individual |
| Regla de negocio | Se resuelve según la entidad | Cliente / proveedor / empleado |
| Por segmento | Un mail por cada clave agrupada | Cobranzas por cliente |

### Regla base
Cuando exista un **documento formal**, el destinatario debe recibir:
- mail breve
- PDF adjunto

---

## 7. Matriz de reportes

| Decisión | Regla |
|---|---|
| Asociación | El reporte pertenece a un proceso |
| Compatibilidad | El reporte usa un único dataset/schema |
| Estándar | No se elimina ni se modifica directamente |
| Personalizado | Puede editarse según permisos |
| Copia | Debe poder crearse desde uno existente |
| Principal | Solo uno por proceso |
| Mobile | Puede usarse, pero no diseñarse |

### Regla fuerte
Los reportes **sí son diseñables por usuario autorizado**, pero siempre sobre el dataset del proceso.

---

## 8. Matriz de plantillas de mail

| Decisión | Regla |
|---|---|
| Asociación | La plantilla pertenece a un proceso |
| Tipo de contenido | Asunto + cuerpo HTML |
| Variables | Placeholders simples |
| Documento | PDF adjunto desde reporte |
| Principal | Solo una por proceso |
| Mobile | Puede usarse, no diseñarse |
| Editor | Solo desktop |

### Regla fuerte
La plantilla mail **no reemplaza** al documento emitido.

---

## 9. Matriz de formatos de integración

| Decisión           | Regla                             |
|--------------------|-----------------------------------|
| Asociación         | El formato pertenece a un proceso |
| Diseño por usuario | No permitido                      |
| Tipo archivo       | TXT / CSV / XLSX / otros          |
| Generación         | Programada                        |
| Validación         | Técnica y obligatoria             |
| Historial          | Debe registrarse                  |
| Mobile             | Solo ejecución simple, no diseño  |

### Regla fuerte
Un formato de integración debe modelarse como **salida técnica**, no como reporte.

---

## 10. Matriz Desktop vs Mobile

| Capacidad                          | Desktop | Mobile |
|------------------------------------|--------:|---:|
| Emitir PDF                         | Sí      | Sí |
| Enviar mail                        | Sí      | Sí |
| Compartir archivo                  | Sí      | Sí |
| Reimprimir comprobante             | Sí      | Sí |
| Elegir reporte existente           | Sí      | Sí |
| Elegir plantilla existente         | Sí      | Parcial |
| Emisión segmentada simple          | Sí      | Parcial |
| ZIP por entidad                    | Sí      | Parcial |
| Generar archivo integración simple | Sí      | Sí, si el flujo es corto |
| Diseñar reportes                   | Sí      | No |
| Diseñar plantillas mail            | Sí      | No |
| Copiar reportes entre empresas     | Sí      | No |
| Administración avanzada            | Sí      | No |
| Auditoría extensa                  | Sí      | No |

### Regla base Mobile
Mobile debe permitir:
- ejecución rápida
- pocas decisiones
- formularios breves
- acceso a reportes existentes
- compartir o enviar

No debe incluir:
- diseño
- parametrización avanzada
- mantenimiento técnico

---

## 11. Matriz de decisión para Cursor / IDE

### 11.1. Cuándo usar DevExtreme
Usar DevExtreme cuando:
- la salida es documental o visual
- se requiere PDF o impresión
- se necesita un layout adaptable para humanos
- el usuario debe poder diseñar o copiar formatos

No usar DevExtreme cuando:
- el archivo es para otra aplicación
- el formato exige estructura exacta
- el archivo tiene header/detalle/trailer técnico
- el banco o sistema externo exige formato rígido

---

### 11.2. Cuándo usar generación técnica programada
Usar generador técnico cuando:
- el archivo va a un banco
- el archivo se importa en otro sistema
- el formato tiene delimitadores, padding o secuencias rígidas
- la validación es técnica y no visual

---

### 11.3. Cuándo permitir diseño por usuario
Permitir diseño por usuario cuando:
- el objetivo es presentación documental
- el proceso expone un dataset estable
- el usuario tiene permisos
- el diseño afecta solo el layout

No permitir diseño por usuario cuando:
- el objetivo es integración externa
- el formato debe cumplir especificación rígida
- una variación puede romper la importación destino

---

### 11.4. Cuándo ofrecer la opción en mobile
Ofrecer opción en mobile cuando:
- la acción es frecuente
- la interacción puede resolverse en pocos pasos
- no requiere análisis visual extenso
- el usuario busca ejecutar, no administrar

No ofrecer en mobile cuando:
- requiere diseñador
- requiere grids complejas
- exige mucha parametrización
- necesita debugging o auditoría detallada

---

## 12. Reglas finales obligatorias

1. Toda salida pertenece a un proceso.
2. Todo proceso define su dataset.
3. Todo reporte pertenece a un proceso y a un dataset.
4. Toda plantilla de mail pertenece a un proceso.
5. Todo formato de integración pertenece a un proceso.
6. El reporte habitual es por proceso.
7. La plantilla habitual es por proceso.
8. El formato de integración habitual es por proceso.
9. Los reportes sí son diseñables por usuarios autorizados.
10. Los formatos de integración no son diseñables por usuarios finales.
11. El mail transporta comunicación; el PDF transporta el documento.
12. Mobile ejecuta; desktop administra y diseña.
13. La emisión segmentada debe registrar detalle por segmento.
14. Toda emisión debe registrarse en historial.
15. Toda salida en mobile debe pasar por una evaluación explícita de costo/beneficio funcional.

---

## 13. Resumen ejecutivo para la IDE

La IDE debe asumir que el subsistema de emisión:

- está centrado en procesos
- distingue entre presentación e integración
- usa DevExtreme solo para salidas documentales
- usa generadores programados para formatos técnicos
- maneja emisión consolidada y segmentada
- utiliza plantillas de mail con PDF adjunto
- soporta mobile solo en modo ejecución simplificada
- reserva diseño y parametrización avanzada para desktop
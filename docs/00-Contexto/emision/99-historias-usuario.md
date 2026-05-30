# Historias de Usuario – Subsistema de Emisión

Este documento enumera las **épicas** e **historias de usuario** derivadas de la documentación de arquitectura del subsistema de emisión en `docs/01-arquitectura/emision/`.

**Documentos de referencia analizados:**
- 01-arquitectura-general-motor-emision.md
- 02-contenedores-motor-emision.md
- 03-flujo-general-emision.md
- 04-estrategias-salida.md
- 05-desktop-vs-mobile.md
- 06-secuencia-envio-mail.md
- 07-secuencia-archivo-integracion.md
- 08-modelo-conceptual-subsistema-emision.md
- 09-modelo-datos-subsistema-emision.md
- 10-matriz-decisiones-funcionales-subsistema-emision.md

---

## Épica EM-01 – Catálogo y configuración de procesos de emisión

**Objetivo:** Permitir definir y mantener los procesos emisibles que orquestan dataset, reportes, plantillas de mail, formatos de integración y comportamiento por canal.

| ID | Historia de usuario | Clasificación | Resumen |
|----|---------------------|---------------|---------|
| HU-EM-001 | Listado de procesos de emisión | MUST-HAVE | Como usuario autorizado quiero ver el listado de procesos de emisión (activos/inactivos) para gestionar el catálogo. Incluye filtros por tipo, estado y búsqueda por nombre. |
| HU-EM-002 | Alta de proceso de emisión | MUST-HAVE | Como usuario autorizado quiero crear un proceso de emisión definiendo nombre, descripción, tipo, modos permitidos (consolidado/segmentado) y canales de salida soportados (impresión, PDF, Excel, CSV, mail, ZIP, integración). |
| HU-EM-003 | Edición de proceso de emisión | MUST-HAVE | Como usuario autorizado quiero editar un proceso de emisión existente (datos básicos, flags de canales, modos) sin alterar el historial de emisiones ya registrado. |
| HU-EM-004 | Configuración por proceso (reporte/plantilla/formato principales) | MUST-HAVE | Como usuario autorizado quiero definir para cada proceso el reporte principal, la plantilla de mail principal y el formato de integración principal, así como si requiere vista previa, permite envío masivo, ZIP y visibilidad en mobile. |
| HU-EM-005 | Definición de dataset por proceso | MUST-HAVE | Como usuario autorizado quiero asociar uno o más esquemas de dataset a un proceso (nombre, tipo de estructura, definición JSON) para que los reportes y salidas usen esos datos. |
| HU-EM-006 | Baja lógica / activación de proceso | SHOULD-HAVE | Como usuario autorizado quiero desactivar o reactivar un proceso de emisión para que no aparezca en las opciones de emisión sin borrar datos históricos. |

---

## Épica EM-02 – Reportes

**Objetivo:** Gestionar reportes asociados a procesos y datasets; permitir diseño en desktop y uso en emisión (desktop y mobile según configuración).

| ID | Historia de usuario | Clasificación | Resumen |
|----|---------------------|---------------|---------|
| HU-EM-007 | Listado de reportes de un proceso | MUST-HAVE | Como usuario quiero ver los reportes disponibles para un proceso (código, nombre, estándar/principal/privado, visible en mobile) con filtros y orden. |
| HU-EM-008 | Creación de reporte (asociado a proceso y dataset) | MUST-HAVE | Como usuario autorizado quiero crear un reporte vinculado a un proceso y a un dataset schema, con código, nombre, tipo de origen (nuevo/copia) y definición de layout. |
| HU-EM-009 | Edición de reporte (diseño layout) | MUST-HAVE | Como usuario autorizado quiero editar el diseño (layout) de un reporte no estándar en desktop, manteniendo compatibilidad con el dataset del proceso. |
| HU-EM-010 | Copia de reporte (entre procesos o como variante) | MUST-HAVE | Como usuario autorizado quiero crear una copia de un reporte existente como base para uno nuevo (mismo proceso u otro), preservando layout y adaptando referencias. |
| HU-EM-011 | Asignación de reporte principal del proceso | MUST-HAVE | Como usuario autorizado quiero marcar un reporte como principal del proceso para que se preseleccione por defecto al emitir. Solo uno por proceso. |
| HU-EM-012 | Permisos por reporte (usar, editar, eliminar, compartir) | SHOULD-HAVE | Como usuario autorizado quiero asignar permisos por usuario (o rol) sobre cada reporte (puede usar, editar, eliminar, compartir) para control de acceso. |
| HU-EM-013 | Vista previa de reporte antes de emitir | MUST-HAVE | Como usuario quiero previsualizar el resultado de un reporte con los datos del proceso actual antes de elegir canal (impresión, PDF, mail, etc.). |
| HU-EM-014 | Reportes visibles en mobile (solo uso, sin diseño) | MUST-HAVE | Como usuario mobile quiero ver y elegir solo los reportes habilitados para mobile del proceso, sin opción de diseñar ni editar. |

---

## Épica EM-03 – Plantillas de mail

**Objetivo:** Gestionar plantillas de correo por proceso (asunto + cuerpo HTML con variables); diseño solo en desktop; uso en emisión con adjunto PDF.

| ID | Historia de usuario | Clasificación | Resumen |
|----|---------------------|---------------|---------|
| HU-EM-015 | Listado de plantillas de mail de un proceso | MUST-HAVE | Como usuario autorizado quiero ver las plantillas de mail del proceso (código, nombre, principal, estándar, visible en mobile). |
| HU-EM-016 | Creación de plantilla de mail | MUST-HAVE | Como usuario autorizado quiero crear una plantilla con asunto y cuerpo HTML, usando placeholders de variables definidas para el proceso. |
| HU-EM-017 | Edición de plantilla de mail | MUST-HAVE | Como usuario autorizado quiero editar asunto y cuerpo de una plantilla no estándar en desktop. El documento formal debe ir en PDF adjunto, no en el cuerpo. |
| HU-EM-018 | Variables de mail del proceso (catálogo) | MUST-HAVE | Como usuario autorizado quiero consultar y mantener el catálogo de variables disponibles para las plantillas del proceso (código, nombre, descripción, ejemplo). |
| HU-EM-019 | Plantilla principal del proceso | MUST-HAVE | Como usuario autorizado quiero marcar una plantilla como principal del proceso para preselección al enviar por mail. Solo una por proceso. |
| HU-EM-020 | Uso de plantilla en mobile (solo selección) | MUST-HAVE | Como usuario mobile quiero elegir una plantilla existente habilitada para mobile al enviar por mail, sin editar contenido. |

---

## Épica EM-04 – Formatos de integración

**Objetivo:** Gestionar el catálogo de formatos de integración por proceso; la generación es programada (no diseñable por usuario); registrar uso en auditoría.

| ID | Historia de usuario | Clasificación | Resumen |
|----|---------------------|---------------|---------|
| HU-EM-021 | Listado de formatos de integración de un proceso | MUST-HAVE | Como usuario autorizado quiero ver los formatos de integración del proceso (código, nombre, tipo archivo, extensión, principal, visible en mobile). |
| HU-EM-022 | Alta de formato de integración (metadatos) | MUST-HAVE | Como usuario técnico quiero registrar un formato de integración (código, nombre, tipo/extension, encoding, generador tipo/referencia) asociado a un proceso. La lógica de generación es programada. |
| HU-EM-023 | Edición de metadatos de formato de integración | MUST-HAVE | Como usuario técnico quiero editar metadatos del formato (nombre, descripción, activo, visible en mobile) sin cambiar la lógica de generación. |
| HU-EM-024 | Formato principal de integración del proceso | MUST-HAVE | Como usuario autorizado quiero marcar un formato como principal del proceso para preselección al generar archivo de integración. Solo uno por proceso. |
| HU-EM-025 | Generación de archivo de integración (flujo) | MUST-HAVE | Como usuario quiero solicitar la generación de un archivo de integración para un proceso y formato; el sistema ejecuta el generador programado, registra la emisión y me permite descargar el archivo (TXT/CSV/XLSX según formato). |
| HU-EM-026 | Generación de archivo de integración desde mobile (flujo simple) | SHOULD-HAVE | Como usuario mobile quiero generar un archivo de integración simple (proceso + formato preseleccionado, pocos pasos) y descargar o compartir el resultado. |

---

## Épica EM-05 – Segmentación y modos de emisión

**Objetivo:** Definir claves de segmentación por proceso y soportar emisión consolidada vs segmentada (una salida total vs una por entidad).

| ID | Historia de usuario | Clasificación | Resumen |
|----|---------------------|---------------|---------|
| HU-EM-027 | Definición de claves de segmentación por proceso | MUST-HAVE | Como usuario autorizado quiero definir claves de segmentación (código, nombre, campo clave, campo descripción, campo mail) para procesos que permitan emisión segmentada. |
| HU-EM-028 | Emisión en modo consolidado | MUST-HAVE | Como usuario quiero emitir en modo consolidado cuando el proceso lo permita: una única salida (PDF, Excel, CSV, etc.) con todos los datos, para uso interno o auditoría. |
| HU-EM-029 | Emisión en modo segmentado | MUST-HAVE | Como usuario quiero emitir en modo segmentado: el sistema agrupa por la clave configurada y genera una salida por segmento (p. ej. un PDF por cliente); opción de descargar como ZIP. |
| HU-EM-030 | Resolución de destinatarios (manual, fijo, por dataset, por segmento) | MUST-HAVE | Como usuario quiero que al enviar por mail el sistema resuelva destinatarios según la configuración del proceso: manual (ingreso mail), fijo parametrizado, único del dataset, o un mail por segmento. |
| HU-EM-031 | Registro de detalle por segmento en emisión segmentada | MUST-HAVE | Como usuario autorizado quiero que cada emisión segmentada registre en historial el detalle por segmento (valor clave, descripción, mail destino, resultado y archivo generado por segmento). |

---

## Épica EM-06 – Emisión por canal (PDF, impresión, Excel, CSV)

**Objetivo:** Ejecutar la emisión según el canal elegido (impresión, PDF, Excel, CSV) usando el motor correspondiente y registrando en auditoría.

| ID | Historia de usuario | Clasificación | Resumen |
|----|---------------------|---------------|---------|
| HU-EM-032 | Selección de proceso y reporte para emitir | MUST-HAVE | Como usuario quiero elegir un proceso de emisión y un reporte (con reporte principal preseleccionado), y opcionalmente vista previa, antes de elegir el canal. |
| HU-EM-033 | Emisión a PDF | MUST-HAVE | Como usuario quiero generar un PDF a partir del reporte seleccionado con los datos del proceso y descargarlo o abrirlo; el sistema usa el motor DevExtreme y registra la emisión. |
| HU-EM-034 | Emisión a impresión | MUST-HAVE | Como usuario desktop quiero enviar la salida del reporte a impresión (flujo que puede resolverse como generación de PDF + diálogo de impresión). |
| HU-EM-035 | Emisión a Excel / CSV (exportador) | MUST-HAVE | Como usuario quiero generar una salida en Excel o CSV según el proceso y el motor de exportación configurado, y descargar el archivo; se registra en historial. |
| HU-EM-036 | Descarga de archivo generado (PDF, Excel, CSV) | MUST-HAVE | Como usuario quiero recibir el archivo generado (nombre, tipo correcto) por descarga en el navegador tras la emisión exitosa. |

---

## Épica EM-07 – Envío por mail

**Objetivo:** Enviar correo con plantilla (asunto + cuerpo) y adjunto PDF generado desde reporte; registrar emisión y opcionalmente detalle por segmento.

| ID | Historia de usuario | Clasificación | Resumen |
|----|---------------------|---------------|---------|
| HU-EM-037 | Envío por mail con plantilla y adjunto PDF | MUST-HAVE | Como usuario quiero enviar un mail eligiendo proceso, reporte y plantilla; el sistema genera el PDF con el reporte, arma el mail con asunto y cuerpo (variables reemplazadas) y adjunta el PDF; se registra la emisión. |
| HU-EM-038 | Selección de destinatarios (manual o según regla) | MUST-HAVE | Como usuario quiero indicar destinatarios manualmente o que el sistema los resuelva según la configuración del proceso (fijo, dataset, por segmento). |
| HU-EM-039 | Envío por mail en emisión segmentada (un mail por segmento) | MUST-HAVE | Como usuario quiero que en emisión segmentada por mail se envíe un correo por cada segmento, con el PDF correspondiente a ese segmento y destinatario resuelto (p. ej. por campo mail del segmento). |
| HU-EM-040 | Envío por mail desde mobile | MUST-HAVE | Como usuario mobile quiero enviar por mail usando reporte y plantilla existentes, con flujo simplificado (pocos pasos, sin diseño). |

---

## Épica EM-08 – Emisión en lote (ZIP)

**Objetivo:** Generar múltiples salidas (p. ej. por segmento) y entregarlas en un único ZIP descargable.

| ID | Historia de usuario | Clasificación | Resumen |
|----|---------------------|---------------|---------|
| HU-EM-041 | Generación de ZIP con salidas segmentadas | MUST-HAVE | Como usuario quiero que al elegir emisión segmentada con canal ZIP el sistema genere un archivo por segmento (PDF u otro) y los empaquete en un único ZIP para descarga. |
| HU-EM-042 | Descarga de ZIP generado | MUST-HAVE | Como usuario quiero descargar el archivo ZIP generado con nombre identificable (proceso, fecha, etc.) y que la emisión quede registrada en historial. |

---

## Épica EM-09 – Historial y auditoría

**Objetivo:** Consultar el historial de emisiones, con detalle por segmento cuando aplique, y soportar auditoría.

| ID | Historia de usuario | Clasificación | Resumen |
|----|---------------------|---------------|---------|
| HU-EM-043 | Consulta de historial de emisiones | MUST-HAVE | Como usuario autorizado quiero consultar el historial de emisiones filtrado por proceso, rango de fechas, usuario emisor, canal y resultado (éxito/error). |
| HU-EM-044 | Detalle de una emisión (cabecera + detalle por segmento) | MUST-HAVE | Como usuario autorizado quiero ver el detalle de una emisión: proceso, reporte/plantilla/formato usados, modo, canal, fecha, usuario, resultado y, si es segmentada, lista de segmentos con valor, mail y archivo generado. |
| HU-EM-045 | Reenvío o reimpresión desde historial | SHOULD-HAVE | Como usuario quiero desde el historial reenviar por mail o reimprimir/descargar de nuevo el documento de una emisión ya realizada (mismo reporte y datos registrados). |
| HU-EM-046 | Auditoría extensa (desktop) | SHOULD-HAVE | Como usuario desktop quiero acceder a vistas de auditoría más detalladas (filtros avanzados, exportación de listado) para análisis y control. |

---

## Épica EM-10 – Experiencia mobile

**Objetivo:** Soportar en mobile solo ejecución simplificada: emitir con reporte/plantilla existentes, ver/compartir PDF, enviar mail, generar archivo de integración simple; sin diseño ni administración.

| ID | Historia de usuario | Clasificación | Resumen |
|----|---------------------|---------------|---------|
| HU-EM-047 | Emisión simplificada desde mobile | MUST-HAVE | Como usuario mobile quiero iniciar una emisión con pocos pasos: elegir proceso (solo visibles en mobile), reporte/plantilla/formato preseleccionados o elegibles desde lista reducida. |
| HU-EM-048 | Ver o compartir PDF desde mobile | MUST-HAVE | Como usuario mobile quiero ver el PDF generado y compartirlo (sistema nativo) o imprimirlo sin acceder al diseñador de reportes. |
| HU-EM-049 | Reimprimir o reenviar comprobante desde mobile | SHOULD-HAVE | Como usuario mobile quiero reimprimir o reenviar por mail un comprobante ya emitido desde una vista simplificada de historial. |
| HU-EM-050 | Restricción: sin diseño ni administración en mobile | MUST-HAVE | Como usuario mobile acepto que en la app mobile no estén disponibles el diseño de reportes, el diseño de plantillas de mail, la administración avanzada de procesos ni la auditoría extensa (solo ejecución y historial básico). |

---

## Épica EM-11 – Transferencias y copias entre empresas (opcional)

**Objetivo:** Soportar transferencia o copia de objetos (reportes, plantillas, etc.) entre empresas según modelo de datos.

| ID | Historia de usuario | Clasificación | Resumen |
|----|---------------------|---------------|---------|
| HU-EM-051 | Registro de transferencia de objetos entre empresas | SHOULD-HAVE | Como usuario autorizado quiero que cuando se copie o transfiera un objeto (reporte, plantilla) entre empresa origen y destino quede registrado en historial de transferencias (tipo objeto, id, empresas, usuario, fecha, resultado). |
| HU-EM-052 | Copia de reporte entre empresas | SHOULD-HAVE | Como usuario autorizado quiero copiar un reporte de una empresa a otra manteniendo layout y referencias adaptadas al proceso destino, con registro de transferencia. |

---

## Resumen por épica

| Épica | Descripción | HUs |
|-------|-------------|-----|
| EM-01 | Catálogo y configuración de procesos | 6 |
| EM-02 | Reportes | 8 |
| EM-03 | Plantillas de mail | 6 |
| EM-04 | Formatos de integración | 6 |
| EM-05 | Segmentación y modos de emisión | 5 |
| EM-06 | Emisión por canal (PDF, impresión, Excel, CSV) | 5 |
| EM-07 | Envío por mail | 4 |
| EM-08 | Emisión en lote (ZIP) | 2 |
| EM-09 | Historial y auditoría | 4 |
| EM-10 | Experiencia mobile | 4 |
| EM-11 | Transferencias entre empresas | 2 |
| **Total** | | **52** |

---

## Reglas de negocio transversales (resumen)

- Toda emisión pertenece a un **proceso**.
- Todo proceso define su **dataset**; reportes y salidas se basan en él.
- **Reportes** son diseñables por usuarios autorizados (desktop); **formatos de integración** no.
- **Mail** = comunicación (asunto + cuerpo); **PDF** = documento (siempre adjunto cuando hay documento formal).
- **Mobile**: solo ejecución simplificada; diseño y administración en **desktop**.
- Emisión **segmentada** debe registrar **detalle por segmento** en historial.
- Toda emisión debe registrarse en **auditoría/historial**.

---

## Referencias

- `docs/01-arquitectura/emision/` – Documentación de arquitectura del subsistema.
- `docs/01-arquitectura/emision/09-modelo-datos-subsistema-emision.md` – Modelo de datos (tablas `PQ_OUT_*`).
- `docs/01-arquitectura/emision/10-matriz-decisiones-funcionales-subsistema-emision.md` – Decisiones funcionales y reglas para Cursor/IDE.

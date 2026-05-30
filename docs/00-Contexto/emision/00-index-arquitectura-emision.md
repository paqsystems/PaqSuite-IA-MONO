# Índice – Arquitectura del Subsistema de Emisión

Este índice enumera todos los documentos de la carpeta `docs/01-arquitectura/emision/` que describen el subsistema de emisión (motor central, reportes, mail, archivos de integración, desktop vs mobile).

---

## Documentos (orden recomendado de lectura)

| # | Documento | Descripción |
|---|-----------|-------------|
| 01 | [Arquitectura General del Motor de Emisión](01-arquitectura-general-motor-emision.md) | Visión general: dataset, reportes, plantillas mail, formatos de integración, modos y canales de salida. |
| 02 | [Contenedores del Subsistema de Emisión](02-contenedores-motor-emision.md) | Capas: Frontend, Aplicación, motores especializados, persistencia. |
| 03 | [Flujo General de Emisión](03-flujo-general-emision.md) | Cómo se genera cualquier salida: selección de proceso, dataset, tipo de salida. |
| 04 | [Estrategias de Salida](04-estrategias-salida.md) | Patrón estrategia por tipo de salida (Print, Mail, Integration). |
| 05 | [Capacidades Desktop vs Mobile](05-desktop-vs-mobile.md) | Diferencias funcionales: desktop para configuración, mobile para ejecución rápida. |
| 06 | [Secuencia de Envío por Mail](06-secuencia-envio-mail.md) | Diagrama de secuencia: usuario → frontend → API → motor emisión → mail. |
| 07 | [Secuencia Generación Archivo de Integración](07-secuencia-archivo-integracion.md) | Diagrama de secuencia: generación de archivo de integración. |
| 08 | [Modelo Conceptual del Subsistema de Emisión](08-modelo-conceptual-subsistema-emision.md) | Entidades: Proceso, Dataset, Reportes, Plantillas Mail, Formatos Integración, Historial. |
| 09 | [Modelo de Datos del Subsistema de Emisión](09-modelo-datos-subsistema-emision.md) | Propuesta de modelo lógico: procesos emisibles, schemas, reportes, plantillas, historial. |
| 10 | [Matriz de Decisiones Funcionales](10-matriz-decisiones-funcionales-subsistema-emision.md) | Decisiones: tipo de salida, opciones configurables, desktop vs mobile, consolidado vs segmentado. |
| 11 | [C4 Nivel 2 – Sistema Completo con Subsistema de Emisión](11-c4-nivel2-sistema-completo-emision.md) | Vista C4: Frontend Web/Mobile, Backend, DBs, motores (Emisión, Reportes, Mail, Integración). |
| 12 | [C4 – Subsistema Emisión](12-c4-subsistema-emision.md) | Descripción de contenedores del subsistema emisión (narrativo C4). |
| 13 | [Integraciones Externas Emisión](13-integraciones-externas-emision.md) | Relaciones con sistemas externos: SMTP, bancos, AFIP, portales, storage. |
| 99 | [Historias de Usuario – Subsistema de Emisión](99-historias-usuario.md) | Épicas e historias de usuario derivadas de esta arquitectura. |

---

## Resumen por tema

- **Visión y flujo:** 01, 02, 03  
- **Estrategias y secuencias:** 04, 06, 07  
- **Desktop / Mobile:** 05  
- **Modelo:** 08, 09  
- **Decisiones funcionales:** 10  
- **Vistas C4 e integraciones:** 11, 12, 13  
- **Trazabilidad a HU:** 99  

# Sentry en PaqSuite IA — Guía para el equipo

Este documento explica **para qué sirve Sentry** en el contexto de nuestro producto y cómo encaja con un modelo de **una sola instalación** (un despliegue del producto por entorno típico, sin la complejidad de correlacionar múltiples clientes ni múltiples instalaciones aisladas del mismo código).

---

## ¿Qué es Sentry?

**Sentry** es una plataforma de **observabilidad orientada a errores y rendimiento** en aplicaciones. Su función principal es:

- **Capturar excepciones y fallos** que ocurren en backend y frontend en tiempo real.
- **Agruparlos en issues** (incidencias) para que no revisemos el mismo error miles de veces como eventos sueltos.
- **Mostrar stack traces** con contexto (release, entorno, URL, tags, breadcrumbs) para **reducir el tiempo de diagnóstico**.
- Opcionalmente, **performance monitoring** (transacciones lentas, cuellos de botella) según plan y configuración.

En la práctica: cuando algo rompe en producción, Sentry es donde **ves qué falló, dónde en el código y qué versión está desplegada**, sin depender solo de capturas de pantalla o logs dispersos.

Sentry **no sustituye** logs de aplicación ni métricas de infraestructura; **complementa** el diagnóstico de bugs con **eventos ricos y correlacionados por release y entorno**.

---

## ¿Para qué lo usamos en este proyecto?

Stack habitual: **Laravel (API/backend)** + **React (Vite)** + **MySQL**. Los proyectos guiados por este documento tienen **una instalación única**: no existe el trabajo de reunir errores dispersos entre muchos despliegues de distintos clientes.

Implicaciones:

- **`environment`** suele bastar para distinguir `production`, `staging`, `development` (o el esquema de nombres que acordemos).
- **`release`** identifica **qué build o versión** está en ese despliegue; es suficiente para priorizar regresiones y comparar después de cada deploy.
- No hace falta un **plan de etiquetado multi-instalación** ni comparar vistas por “cliente”: el foco es **salud del único sistema en marcha**.

Objetivos típicos para el equipo:

1. **Detectar regresiones** tras un despliegue (comparando releases).
2. **Priorizar** por frecuencia e impacto (usuarios o eventos afectados).
3. **Enriquecer el contexto** con tags seguros solo cuando aporten valor **local** al producto (p. ej. módulo, feature flag), siempre sin PII innecesaria.
4. **Facilitar diagnóstico**: enlazar una issue con el cambio en código y verificar en la siguiente release.

---

## Conceptos que conviene conocer

| Concepto | Descripción breve |
|----------|-------------------|
| **DSN** | URL/clave del proyecto en Sentry donde se envían los eventos. Suele ir por variable de entorno; **no commitear** valores reales. |
| **Issue** | Agrupación de eventos similares (mismo tipo de error); es la unidad de trabajo típica (“arreglar esta incidencia”). |
| **Release** | Identificador de la versión del software desplegado (semver, tag Git, build). Permite preguntas del tipo “¿en qué release apareció?”. |
| **Environment** | Etiqueta de entorno (`production`, `staging`, `local`, …). En instalación única no se usa para discriminar instalaciones paralelas del mismo código. |
| **Tags / contexto** | Pares clave-valor opcionales para filtrar en la UI. Evitar PII; usar identificadores técnicos acordados. |
| **Source maps** | En frontend, permiten ver el stack trace en **TypeScript/original** y no solo en el bundle minificado. |

---

## Modelo recomendado (instalación única)

Con **una sola línea productiva** (y quizá staging), lo habitual es:

- **Un proyecto Sentry** por producto o por repositorio (según cómo organizéis la cuenta).
- **Un DSN por entorno** (o mismo DSN y `environment` distinto desde `.env`; ambos patrones son válidos mientras las variables estén claras en documentación).

No hace falta decidir entre “proyecto por cliente” vs “tags por cliente”: **ese árbol de decisión aquí no aplica**. Si en el futuro el mismo código se instalara muchas veces en sitios distintos, revisar esa estrategia con el equipo antes de exponer métricas mezcladas.

---

## Arranque: proyecto, variables y validación

### 1. Crear el proyecto en Sentry

1. En [sentry.io](https://sentry.io), dentro de vuestra organización, crear **un proyecto** para esta línea de producto (por ejemplo `paqsuite-ia`).
2. Elegir la plataforma principal si os lo pide el asistente (Laravel/React según corresponda sirve sobre todo para defaults).
3. Copiar el **DSN** y guardarlo solo como valor de entorno (`SENTRY_LARAVEL_DSN`, equivalente en frontend): **no** subirlo al repositorio.

### 2. Convenciones mínimas

| Concepto | Convención sugerida | Ejemplo |
|----------|---------------------|---------|
| **Environment** | Nombres estables por fase (`production`, `staging`, …) | `production`, `staging` |
| **Release** | Versión del artefacto desplegado | `2.4.1`, `2.4.1+build20260509` |
| **Tags (opcional)** | Solo si ayudan al diagnóstico interno, sin datos personales | `module:facturacion` |

Sin obligación de `installationId` ni `clientSlug`: no hay que segmentar errores entre instalaciones distintas.

### 3. Variables en el despliegue (`.env`)

1. **`SENTRY_*` / DSN** según Laravel y frontend.
2. **`SENTRY_ENVIRONMENT`** (o mapeo claro desde `APP_ENV`).
3. **`SENTRY_RELEASE`** alineado con la versión desplegada (CI, tag Git o proceso de release único).

### 4. Panel de Sentry: alertas

1. Crear alertas enlazadas a `environment` (p. ej. solo `production` para prioridad alta).
2. Revisar **Data scrubbing** (Security → proyecto) para campos sensibles (cookies, headers, payload).

### 5. Integración en código (resumen)

- **Laravel:** `environment` y `release` desde `.env`; tags opcionales vía middleware o scopes si el dominio lo requiere.
- **React:** misma **`release`** (y mismo criterio de `environment`) vía variables de build o runtime según cómo empaquetéis; **no hardcodear** secretos en el cliente.

### 6. Validación antes de dar por cerrado el arranque

1. En **staging** (y luego producción cuando toque): forzar un error controlado en API y otro en SPA.
2. Comprobar en Sentry **`environment`** y **`release`** esperados.
3. Ajustar umbrales de alertas según volumen real.

---

## Pasos de implementación en el proyecto (aplicación)

Integración pensada para **Laravel + React (Vite)**; ejecutar cuando el equipo priorice la tarea.

### Backend (Laravel)

- SDK oficial **`sentry/sentry-laravel`** y publicar configuración si aplica.
- Opcional: activar solo fuera de `local` para reducir ruido (según `.env`).
- Variables típicas del despliegue: `SENTRY_LARAVEL_DSN`, `SENTRY_ENVIRONMENT` (o criterio claro con `APP_ENV`), `SENTRY_RELEASE`.

### Frontend (React + Vite)

- **`@sentry/react`** integrado con el router del proyecto.
- **Source maps** en build (plugin de Vite o CLI de Sentry) para stacks legibles.
- **DSN** y **`release`** vía variables de entorno del build pipeline.

### Versionado (`release`)

- En cada deploy: actualizar **`SENTRY_RELEASE`** con la versión acordada.
- Las preguntas del tipo “¿en qué release se introdujo?” se resuelven dentro de esa **única línea temporal** del producto.

---

## Operativa en instalación única

1. **`.env`** (o gestor de secretos) del servidor con **DSN**, **environment** y **release**.
2. **Política de PII**: scrubbing en Sentry, campos prohibidos en contexto manual y opción de **desactivar** el SDK si compliance lo exige.

El **MCP de Sentry en Cursor** es solo para el equipo de desarrollo en el IDE; **no forma parte** del artefacto desplegado en el servidor de la instalación.

---

## Orden recomendado (resumen)

1. Crear **proyecto Sentry** y fijar nombres de **environment**.
2. **Integrar** Laravel y React documentando variables en `.env.example` (sin secretos).
3. **Definir** cómo se genera **`SENTRY_RELEASE`** en CI o en el proceso de deploy.
4. **Probar** en staging errores backend y frontend y revisar agrupación en issues.
5. Activar **alertas** coherentes para producción.

---

## MCP de Sentry (solo desarrollo)

El **Model Context Protocol** de Sentry en Cursor conecta el IDE con vuestra cuenta de Sentry para que los asistentes puedan consultar issues y contexto.

- Es una herramienta **para programadores**, configurada en el entorno local de Cursor.
- **No** forma parte del deploy ni de la instalación única del producto.

---

## Herramientas del MCP (referencia)

Las herramientas las expone el servidor MCP integrado en Cursor; los nombres pueden variar ligeramente según la versión del conector. Resumen alineado con el comportamiento descrito por el propio MCP:

| Herramienta | Función |
|-------------|---------|
| **`whoami`** | Indica **qué usuario de Sentry** está autenticado en la sesión (identidad útil para comprobar acceso antes de consultar datos). |
| **`find_organizations`** | **Lista organizaciones** a las que tenés acceso y permite filtrar por nombre o **slug** (hasta ~25 resultados; sirve para obtener `organizationSlug` y `regionUrl` en otras llamadas). |
| **`find_teams`** | Lista **equipos** dentro de una organización; devuelve slug e ID para encadenar con otros pedidos. |
| **`find_projects`** | Lista **proyectos** de una organización (slug, búsqueda por nombre/slug); base para acotar issues, releases o eventos. |
| **`find_releases`** | Lista **releases** recientes o busca por texto en la versión; sirve para correlacionar errores con despliegues y entornos. |
| **`get_issue_tag_values`** | Obtiene la **distribución de valores de un tag** para una issue concreta (p. ej. `url`, `browser`, `environment`, `release`): cuántas veces aparece cada valor y qué facetas predominan. |
| **`get_replay_details`** | Resume una **Session Replay** por URL o por `replayId` (qué ocurrió en la sesión y pistas para seguir con issues o trazas). |
| **`get_event_attachment`** | **Lista o descarga adjuntos** de un evento (capturas, logs subidos, etc.); con `attachmentId` descarga uno; sin él, lista metadatos y enlaces. |
| **`search_events`** | **Busca eventos o replays** en datasets de Sentry (errores, logs, spans, métricas, perfiles, replays): agregaciones (conteos) o filas con timestamp. **No** sustituye el listado de issues agrupados → usar **`search_issues`**. |
| **`analyze_issue_with_seer`** | Pide análisis con **Seer** sobre una issue: **causa raíz**, ubicación en código y **sugerencias de fix** (IA de Sentry); conviene cuando el usuario lo pide explícitamente o el detalle de la issue no alcanza. |
| **`search_issues`** | Devuelve una **lista de issues agrupadas** (título, estado, usuarios afectados, etc.) con búsqueda en lenguaje natural o sintaxis de issues. **No** usar para conteos agregados globales → **`search_events`**. |
| **`search_issue_events`** | Lista **eventos dentro de una issue** concreta (filtrá por entorno, release, usuario, trace, etc.); el contexto queda acotado a esa agrupación. |
| **`get_profile_details`** | Inspecciona un **perfil de rendimiento** (transacción o continuo): resumen, marcos calientes, muestras; acepta URL de perfil o IDs + rango temporal para perfiles continuos. |
| **`get_sentry_resource`** | **Recuperación genérica** por URL o por tipo + ID: issues, eventos, trazas, spans, breadcrumbs, replays; pieza central para “abrir” un recurso cuando ya tenés el enlace o los identificadores. |

---

## Referencia rápida

Para prompts de ejemplo orientados al MCP en el IDE, ver `prompts/sentry.md`.

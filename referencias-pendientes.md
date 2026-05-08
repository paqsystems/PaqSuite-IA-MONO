# Referencias pendientes (archivos citados pero no encontrados)

Este archivo lista rutas mencionadas en la documentación del repositorio (principalmente `.cursor/rules/`) que **no están presentes** en el estado actual del árbol de trabajo.

**Alcance de la revisión:** raíz del repo `PaqSuite-IA-MONO` sin carpeta `docs/`, ni `backend/`, ni otros proyectos enlazados; solo se verifica existencia en este clon/workspace.

---

## Documentación esperada (`docs/`)

Las siguientes rutas aparecen como referencias; conviene ubicarlas en el mono-repo/documentación oficial o crear el contenido.

| Ruta | Citada desde |
|------|----------------|
| `docs/00-contexto/00-contexto-global-erp.md` | `.cursor/rules/01-project-context.md` |
| `docs/01-arquitectura/README.md` | `.cursor/rules/01-project-context.md` |
| `docs/01-arquitectura/01-arquitectura-proyecto.md` | `.cursor/rules/02-backend-policy.md` |
| `docs/03-historias-usuario/000-Generalidades/HU-001-layouts-grilla.md` | `.cursor/rules/09-tareas-grillas-habilitar-layouts-hu001.md` |
| `docs/03-historias-usuario/000-Generalidades/HU-007-Parametros-generales.md` | `.cursor/rules/11-parametros-generales-por-modulo.md`, `.cursor/rules/12-plan-tareas-hu-parametros-generales.md` |
| `docs/06-operacion/deploy-infraestructura.md` | `.cursor/rules/07-versioning-and-deploy.md` |
| `docs/backend/PLAYBOOK_BACKEND_LARAVEL.md` | `.cursor/rules/02-backend-policy.md`, `.cursor/rules/05-data-access-orm-sql.md` |
| `docs/design/paqsystems-main-shell-design.md` | `.cursor/rules/10-dashboard-indicadores-por-modulo.md` |
| `docs/domain/DATA_MODEL.md` | `.cursor/rules/02-backend-policy.md`, `.cursor/rules/05-data-access-orm-sql.md` |
| `docs/frontend/devextreme-norms.md` | `.cursor/rules/08-devextreme-grid-standards.md` |
| `docs/frontend/ui-layer-wrappers.md` | `.cursor/rules/08-devextreme-grid-standards.md` |
| `docs/modelo-datos/pq-parametros-gral.md` | `.cursor/rules/12-plan-tareas-hu-parametros-generales.md` |

---

## Otros artefactos citados (fuera de `docs/`)

| Ruta | Citada desde |
|------|----------------|
| `backend/storage/api-docs/api-docs.json` | `.cursor/rules/03-api-contract.md` |

---

## Reglas Cursor (`.cursor/rules/`) referenciadas en el pasado y no presentes

Durante la renumeración correlativa (**01–14**) se omitieron enlaces a archivos de reglas que **no estaban en el workspace**. Si existen en otra rama o repo, conviene restaurarlas como archivos numerados nuevos y volver a enlazar desde las reglas relacionadas:

| Archivo citado antes | Contexto breve |
|----------------------|----------------|
| `.cursor/rules/07-frontend-norms.md` | Citada desde `14-obtencion-datos-performance.md`; enlace quitado por inexistencia |
| `.cursor/rules/13-user-story-to-task-breakdown.md` | Citada desde dashboard por módulo; enlace quitado |
| `.cursor/rules/24-devextreme-grid-standards-mono.md` | Nombre anterior **no coincide** con el archivo real renombrado a `08-devextreme-grid-standards.md` |
| `.cursor/rules/29-ui-catalogos-fk-codigo-descripcion.md` | Referencias en política backend; entrada quitada |
| `.cursor/rules/30-ui-abm-grilla-alta-edicion-modal.md` | Referencias en estándares de grilla / parámetros; entrada quitada |
| `.cursor/rules/31-estado-hu-tr.md` | Referencia textual en `06-task-execution-traceability.md`; bloque quitado |

---

## Notas

* Si el contenido vivo del proyecto vive fuera del root visible de Cursor (submódulos, otro volumen), las rutas anteriores podrían resolverse al abrir ese árbol o al sincronizar el mono-repo completo.

* Actualizar esta lista cuando existan los archivos esperados conviene hacerlo **aquí con el mismo formato** para no dispersar rutas fantasmas en las reglas.

# Documentación mono (_mono)

Carpeta de **reingeniería inversa**: especificaciones en lenguaje natural reconstruidas a partir de historias de usuario en:

- `../hu-anteriores/` — épica 000 Generalidades
- `../001-Seguridad/` — épica 001 Seguridad y Acceso

Estos documentos son la fuente para volver a generar HUs, specs técnicas y tareas. El despliegue objetivo es **monoempresa** salvo bloques explícitos comentados para versión **MULTI**.

## Índice por tema

### Acceso y sesión

| Archivo | Contenido | Origen principal |
|---------|-----------|------------------|
| [Login.md](./Login.md) | Login Sanctum, recuperación/cambio contraseña, logout, post-login MONO | 001: HU-001, HU-003, HU-004, HU-005 |
| [idioma.md](./idioma.md) | i18n, selector, banderas, `users.locale` | 000: HU-004, HU-008 |
| [menu-avatar.md](./menu-avatar.md) | Perfil, apariencia, nueva pestaña, Asistente IA, logout | 001 + 000 |
| [apariencia-temas.md](./apariencia-temas.md) | Temas DevExtreme por usuario (menú avatar) | 000: HU-011 UI; no tema por empresa en MONO |

### Menú y seguridad

| Archivo | Contenido | Origen principal |
|---------|-----------|------------------|
| [Menu-general.md](./Menu-general.md) | Seed `pq_menus`, API menú, sidebar dinámico, iconos | 001: HU-015–HU-018 |
| [seguridad-permisos.md](./seguridad-permisos.md) | Roles, permisos, atributos, resolución de menú | 001: HU-012–HU-014, HU-016 |
| [administracion-seguridad.md](./administracion-seguridad.md) | ABM usuarios, roles, permisos, atributos | 001: HU-010, HU-012–HU-014; HU-011 MULTI |

### Datos y UI transversal

| Archivo | Contenido | Origen principal |
|---------|-----------|------------------|
| [grillas.md](./grillas.md) | Estándar grilla, layouts, export Excel, eliminar ABM | 000: HU-001, HU-006, HU-009 |
| [pivots.md](./pivots.md) | PivotGrid y export tabla dinámica | 000: HU-006 |
| [parametros-generales.md](./parametros-generales.md) | `PQ_PARAMETROS_GRAL` por módulo | 000: HU-007 |

## Flujo recomendado

1. Leer el documento mono del dominio.
2. Derivar o actualizar historias de usuario (OpenSpec / `docs/03-historias-usuario/`).
3. Bajar a specs de arquitectura, API y UI.
4. Implementar y validar contra criterios de aceptación.

## Convenciones MONO / MULTI

| Tema | MONO (este proyecto) | MULTI (futuro, bloques `<!-- -->`) |
|------|----------------------|-------------------------------------|
| Empresa activa | Una sola; sin selector ni `X-Company-Id` | HU-002, menú avatar, `Pq_Permiso` multi |
| Apariencia | `users.theme` en menú avatar | `PQ_Empresa.Theme` por tenant |
| Parámetros / API datos | Company DB única | Resolución por `X-Company-Id` |
| Admin empresas | No catálogo multi-tenant | HU-011 completa |

## Notas de numeración

- **HU-011** en `hu-anteriores` = UI full DevExtreme; **HU-011** en `001-Seguridad` = administración de empresas (MULTI).
- Al regenerar HUs mono, usar numeración de épica PedidosWeb/OpenSpec, no copiar tal cual la del repo legacy.

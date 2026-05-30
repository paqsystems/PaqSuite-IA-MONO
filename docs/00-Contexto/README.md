# Documentación mono (`_mono`)

Carpeta de **contexto funcional compartido** para proyectos **monoempresa**. Reúne la definición conceptual del framework en lenguaje humano y funciona como fuente para regenerar HUs, OpenSpec, tareas técnicas y arquitectura.

El alcance vigente del repositorio es **MONO**. Las referencias a **MULTI** se conservan solo como variante futura y deben leerse con ese criterio.

## Propósito de la carpeta

Esta carpeta no describe una implementación técnica puntual ni un producto específico. Su función es capturar el marco conceptual común del sistema para después derivar:

1. historias de usuario en `docs/03-historias-usuario/`,
2. tareas técnicas en `docs/04-tareas/`,
3. especificaciones formales en `docs/05-open-spec/`,
4. decisiones de implementación en `docs/01-arquitectura/`.

## Estructura por dominios

### `00-arquitectura-api`

| Archivo | Contenido |
|---------|-----------|
| [envelope-respuestas.md](./00-arquitectura-api/envelope-respuestas.md) | Formato JSON obligatorio: `error`, `respuesta`, `resultado` |
| [00-instalacion-scaffold-fullstack.md](./00-instalacion-scaffold-fullstack.md) | **Instalación inicial** Laravel 10 + envelope + React/Vite/DevExtreme |

### `01-experiencia-base`

| Archivo | Contenido |
|---------|-----------|
| [estructura-sitio.md](./01-experiencia-base/estructura-sitio.md) | Mapa funcional del sitio y tipologías de pantalla |
| [shell-layout.md](./01-experiencia-base/shell-layout.md) | Aterrizaje MONO del shell post-login |
| [menu-general.md](./01-experiencia-base/menu-general.md) | Sidebar, seed `pq_menus`, API y navegación |
| [menu-avatar.md](./01-experiencia-base/menu-avatar.md) | Perfil, apariencia, nueva pestaña, ayuda y logout |
| [navegacion-pestanas.md](./01-experiencia-base/navegacion-pestanas.md) | Preferencia de apertura en misma o nueva pestaña |
| [idioma-multilingual.md](./01-experiencia-base/idioma-multilingual.md) | Idioma, i18n y alcance transversal |
| [apariencia-temas.md](./01-experiencia-base/apariencia-temas.md) | Temas DevExtreme por usuario y estándar visual |
| [ayuda-externa-asistente.md](./01-experiencia-base/ayuda-externa-asistente.md) | Acceso global a ayuda externa o Asistente IA |

### `02-acceso-y-seguridad`

| Archivo | Contenido |
|---------|-----------|
| [login-y-sesion.md](./02-acceso-y-seguridad/login-y-sesion.md) | Login, recuperación, cambio de contraseña y logout |
| [usuarios-roles-permisos-resumen.md](./02-acceso-y-seguridad/usuarios-roles-permisos-resumen.md) | Mapa conceptual rápido de identidad, roles y permisos |
| [seguridad-permisos.md](./02-acceso-y-seguridad/seguridad-permisos.md) | Roles, permisos y autorización |
| [administracion-seguridad.md](./02-acceso-y-seguridad/administracion-seguridad.md) | ABM de usuarios, roles, permisos y atributos |
| [menu-y-autorizacion.md](./02-acceso-y-seguridad/menu-y-autorizacion.md) | Relación entre `pq_menus`, roles y sidebar |

### `03-ui-transversal`

| Archivo | Contenido |
|---------|-----------|
| [grillas.md](./03-ui-transversal/grillas.md) | Estándar de grillas, layouts, exportación y baja |
| [pivots.md](./03-ui-transversal/pivots.md) | Tablas dinámicas y exportación pivot |
| [exportaciones.md](./03-ui-transversal/exportaciones.md) | Criterio común de exportación para grillas y pivots |
| [plantillas.md](./03-ui-transversal/plantillas.md) | Tipologías reutilizables de pantalla |
| [patrones-abm.md](./03-ui-transversal/patrones-abm.md) | Comportamientos comunes de alta, baja y modificación |

### `04-configuracion-global`

| Archivo | Contenido |
|---------|-----------|
| [parametros-generales.md](./04-configuracion-global/parametros-generales.md) | Proceso transversal de `PQ_PARAMETROS_GRAL` |
| [configuracion-funcional-por-modulo.md](./04-configuracion-global/configuracion-funcional-por-modulo.md) | Criterio de qué corresponde parametrizar por módulo |

### `05-variantes-y-alcance`

| Archivo | Contenido |
|---------|-----------|
| [mono-vs-multi-referencias.md](./05-variantes-y-alcance/mono-vs-multi-referencias.md) | Diferencias de alcance entre MONO y MULTI |

## Diseño compartido en `docs/_base`

Algunos activos visuales y lineamientos base siguen viviendo en `docs/_base`:

| Archivo | Contenido |
|---------|-----------|
| `docs/_base/shell-layout-principal.md` | Shell post-login compartido entre MONO y MULTI |
| `docs/_base/Bosquejo-pantalla-principal.jpg` | Referencia visual del layout |

## Flujo recomendado

1. Leer el documento de contexto del dominio correspondiente.
2. Derivar o actualizar HUs y OpenSpec.
3. Bajar a especificaciones de arquitectura, API y UI.
4. Implementar y validar contra criterios de aceptación.

## Criterio MONO vs MULTI

Resumen rápido:

| Tema | MONO | MULTI |
|------|------|-------|
| Empresa activa | Fija por despliegue | Seleccionable por usuario |
| Apariencia | `users.theme` por usuario | Frecuentemente por empresa |
| API de datos | Sin `X-Company-Id` | Con `X-Company-Id` |
| Administración de empresas | No aplica | Puede existir |

Detalle ampliado: `05-variantes-y-alcance/mono-vs-multi-referencias.md`.

# Shell y layout principal en MONO

Documento de contexto del armazón post-login para soluciones **monoempresa**. Complementa la especificación visual compartida en `docs/_base/shell-layout-principal.md` y la aterriza al modo MONO de este repositorio.

## Objetivo

Definir el marco de navegación y trabajo común del sistema para que todas las pantallas partan de la misma estructura mental:

- header persistente,
- sidebar de procesos,
- área principal de trabajo,
- footer con contexto de sesión.

## Alcance

Aplica a toda pantalla **posterior al login** en frontend web. No define el detalle técnico de componentes ni estilos; ese nivel se baja luego en `docs/01-arquitectura/` y en las specs derivadas.

## Estructura base

La composición general del shell es la misma que la documentada en `docs/_base/shell-layout-principal.md`:

1. **Header** con marca, selector de idioma y acceso al menú avatar.
2. **Sidebar** con menú general filtrado por permisos.
3. **Área principal** para dashboard, grillas, formularios, parámetros e informes.
4. **Footer** con versión, identidad de sesión y leyenda institucional.

## Desglose funcional por zona

### Header

- **Grupo de tres controles del menú** (izquierda, antes o junto al logo): hamburguesa (panel sidebar), expandir/contraer árbol, vista todas las ramas / solo operativas. Definición en `menu-general.md`; persistencia **por usuario o terminal**, nunca por empresa ni global.
- Marca o logo del cliente.
- Selector de idioma visible fuera del menú avatar.
- Avatar del usuario en el extremo derecho.
- En MONO no muestra selector de empresa.

### Sidebar

- Menú lateral jerárquico construido desde `pq_menus`.
- Debe permitir identificar claramente módulo, submódulo y proceso.
- El nodo activo y su rama deben permanecer reconocibles durante la navegación.

### Área principal

- Aloja el dashboard inicial o el proceso activo.
- Debe soportar la convivencia de grillas, pivots, formularios, parámetros y ayudas sin alterar el shell.
- Puede implementar navegación interna por rutas o por pestañas, según defina el producto.

### Footer

- Expone contexto de sesión legible para el usuario.
- Muestra versión y leyenda institucional.
- No reemplaza mensajes de error, confirmaciones ni indicadores de carga.

## Reglas funcionales

- Tras login exitoso el usuario entra directamente al shell principal; en MONO **no** existe paso intermedio de selección de empresa.
- El header mantiene visibles los controles globales del sistema y no cambia entre procesos.
- El sidebar refleja lo autorizado por menú y permisos; no se hardcodea por perfil en el cliente.
- El área principal puede mostrar la ruta de inicio del producto o el proceso abierto desde el menú.
- El footer permanece visible como barra de estado liviana, sin reemplazar mensajes operativos ni toasts.
- La experiencia del shell debe ser estable aunque cambie el módulo o tipo de proceso.
- La responsividad del shell no cambia la semántica de sus zonas; solo adapta su presentación.

## Particularidades MONO

- No hay selector de empresa activa en header ni en menú avatar.
- La preferencia distintiva del avatar es **Apariencia**, persistida por usuario.
- La resolución de datos no depende de `X-Company-Id`; existe una única Company DB por despliegue.

## Relación con otros temas

- Menú lateral: `menu-general.md`
- Menú avatar: `menu-avatar.md`
- Idioma: `idioma-multilingual.md`
- Login y post-login: `../02-acceso-y-seguridad/login-y-sesion.md`
- Variantes MONO vs MULTI: `../05-variantes-y-alcance/mono-vs-multi-referencias.md`

## Derivaciones esperables

Este documento alimenta specs e historias sobre:

- layout principal,
- navegación global,
- comportamiento post-login,
- responsive del shell,
- distribución de controles globales (incl. tres toggles de menú),
- dashboard inicial y permanencia del marco de navegación.

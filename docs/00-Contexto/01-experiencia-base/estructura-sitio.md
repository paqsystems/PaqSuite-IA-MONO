# Estructura del sitio

Documento de contexto de la organización funcional del frontend web en modo **MONO**. Describe cómo se distribuyen los espacios y tipos de navegación del sitio sin entrar todavía en detalle técnico de rutas o componentes.

## Objetivo

Dar una visión simple y compartida de cómo se recorre la aplicación:

- desde el login,
- hacia el shell principal,
- desde el shell hacia módulos y procesos,
- y desde allí hacia operaciones concretas.

## Mapa conceptual

El sitio se entiende en tres niveles:

1. **Acceso**: login y recuperación de credenciales.
2. **Navegación global**: shell principal, menú general, menú avatar, idioma.
3. **Procesos**: dashboards, ABM, consultas, parámetros, informes y ayudas.

## Flujo general del sitio

1. El usuario entra por login.
2. El sistema valida credenciales, permisos y preferencias.
3. Se accede al shell principal.
4. Desde el menú general o la ruta de inicio se ingresa a procesos concretos.
5. Cada proceso se ejecuta sin romper el marco común del shell.
6. El usuario puede volver al inicio, cambiar preferencias o cerrar sesión desde controles globales.

## Zonas principales del sitio

### Antes de autenticarse

- Pantalla de login.
- Selector de idioma visible.
- Acceso a recuperación de contraseña.

### Después de autenticarse

- Shell principal común en toda la navegación.
- Ruta de inicio con dashboard o vista principal del producto.
- Menú lateral para abrir procesos autorizados.
- Menú avatar para acciones personales y de sesión.

## Tipos de navegación

- **Navegación global**: idioma, avatar, logout, ayudas y preferencias.
- **Navegación por procesos**: opciones del sidebar o accesos equivalentes.
- **Navegación contextual**: acciones dentro de un proceso, como abrir modal, detalle o cambiar de vista grilla/pivot.

## Tipos de pantalla esperados

- **Inicio**: dashboard o resumen inicial.
- **ABM**: listados con alta, edición, baja y detalle.
- **Consultas e informes**: grillas, pivots y exportaciones.
- **Configuración**: parámetros generales y otras preferencias funcionales.
- **Pantallas de seguridad**: usuarios, roles, permisos, atributos.

## Reglas funcionales

- El usuario debe poder volver al inicio sin perder la estructura global del shell.
- Las opciones visibles de navegación se resuelven por menú y permisos, no por convenciones implícitas.
- Los procesos mantienen una experiencia homogénea en caption, toolbar, modales y acciones.
- La estructura general debe seguir siendo reconocible aunque cambie el producto concreto que se monte sobre el framework.
- El sitio debe distinguir claramente entre acciones globales del usuario y acciones operativas del proceso activo.
- La operatoria principal no debe depender de accesos ocultos o no visibles dentro del marco general.

## Particularidades MONO

- No existe cambio de empresa dentro del sitio.
- Las preferencias globales son por usuario, no por empresa.
- El sitio opera contra una instalación única, aunque se mantenga referencia documental a variantes MULTI.

## Relación con otros temas

- Shell: `shell-layout.md`
- Menú general: `menu-general.md`
- Menú avatar: `menu-avatar.md`
- Navegación por pestañas: `navegacion-pestanas.md`
- UI transversal: `../03-ui-transversal/grillas.md`

## Derivaciones esperables

Este documento sirve de base para especificar:

- layout y navegación global,
- rutas y comportamiento post-login,
- tipologías de pantalla,
- consistencia de UX entre módulos,
- separación entre navegación global y navegación operativa.

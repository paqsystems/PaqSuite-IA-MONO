# Menú y autorización

Documento de contexto que explica cómo se relacionan el árbol de `pq_menus`, los roles, los atributos y la API de menú del usuario.

## Objetivo

Dejar explícita la lógica conceptual que conecta:

- la definición estructural del menú,
- las asignaciones de usuario y rol,
- la autorización por opción,
- y la construcción del sidebar visible en frontend.

## Fuente de verdad

La estructura del menú vive en `pq_menus` y se carga por seed versionado. El menú visible para un usuario no es ese árbol completo, sino una proyección filtrada por seguridad.

## Flujo conceptual

1. El usuario inicia sesión.
2. El sistema obtiene las asignaciones del usuario en `Pq_Permiso`.
3. Para cada rol aplicable, evalúa si tiene `AccesoTotal`.
4. Si no tiene acceso total, consulta `PQ_RolAtributo` para determinar qué opciones de menú quedan autorizadas.
5. El backend devuelve el árbol resultante mediante la API de menú del usuario.
6. El frontend construye el sidebar a partir de esa respuesta, sin recalcular permisos localmente.

## Reglas de autorización

- `pq_menus` define qué existe y cómo se ordena.
- `enabled` indica si un ítem puede formar parte del menú visible.
- `AccesoTotal` otorga acceso a todas las opciones habilitadas.
- Si un rol no tiene `AccesoTotal`, basta con que exista al menos un atributo válido para que la opción aparezca en el menú.
- Si el usuario tiene varios roles, la autorización efectiva es la unión de permisos.
- Un ítem no visible en el menú no debería quedar operativamente accesible por la UI estándar del sistema.

## Reglas de presentación derivadas

- El sidebar debe respetar `parentId` y `orden`.
- Los nodos visibles deben conservar su jerarquía completa.
- El frontend no debe usar flags hardcodeadas como `esAdmin` para decidir visibilidad de secciones.
- Si la API falla o no devuelve opciones, debe existir estrategia de menú mínimo o mensaje controlado.
- La respuesta del menú debe ser suficiente para reconstruir el árbol visible sin necesitar reglas implícitas por módulo.

## MONO

En MONO no interviene una empresa seleccionada por el usuario en tiempo real. La autorización se resuelve sobre la instalación única del despliegue.

## Relación con otros temas

- Menú general: `../01-experiencia-base/menu-general.md`
- Seguridad y permisos: `seguridad-permisos.md`
- Login y sesión: `login-y-sesion.md`
- Administración de seguridad: `administracion-seguridad.md`
- Mantenimiento de roles y permisos: `mantenimiento-roles-permisos.md`

## Derivaciones esperables

Este documento sirve de base para:

- API de menú del usuario,
- sidebar dinámico,
- validación de visibilidad de opciones,
- coherencia entre permisos de menú y acciones en pantalla,
- relación entre seed del menú, roles y atributos.

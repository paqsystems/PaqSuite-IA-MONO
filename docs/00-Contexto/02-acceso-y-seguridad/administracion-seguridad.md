# Administración de seguridad

Documento de reingeniería de los procesos de mantenimiento de usuarios, roles, permisos y atributos de rol.

Solo usuarios con perfil administrador, o con permisos explícitos sobre las opciones de administración, acceden a estas pantallas. La implementación se alinea con los patrones de ABM en `../03-ui-transversal/patrones-abm.md`.

---

## Administración de usuarios

Gestión de la tabla `users`.

> **PedidosWeb:** este ABM **no aplica** en el producto; los usuarios se proveen desde el ERP. En permisos solo se usan como catálogo de lookup. Ver `mantenimiento-roles-permisos.md` §3.

### Capacidades

- Listar con filtros por código, nombre, email y estado.
- Alta de usuario con datos básicos y contraseña inicial.
- Edición de datos permitidos.
- Baja lógica mediante `inhabilitado = true`.
- Cuando un módulo lo requiera, exponer atributos funcionales asociados a la operatoria del producto, por ejemplo un indicador `supervisor`.

### Reglas

- Código y email deben ser únicos.
- La contraseña siempre se almacena con hash.
- Las asignaciones de usuario y rol se gestionan aparte.
- La baja es lógica; un usuario inhabilitado no debe poder volver a autenticarse mientras permanezca en ese estado.
- Si existe un atributo funcional como `supervisor`, su administración debe quedar claramente diferenciada de los roles técnicos del framework: puede coexistir con ellos, pero no los reemplaza.
- Los atributos funcionales de usuario que pertenezcan a un módulo deben documentarse también en la documentación de producto correspondiente.

---

## Administración de roles

Gestión de `Pq_Rol`.

- Alta, edición y listado de nombre, descripción y `AccesoTotal`.
- Eliminación solo si la política de integridad lo permite.
- Los atributos granulares del rol se administran en proceso separado.
- Un rol con `AccesoTotal` no necesita mantenimiento fino de atributos para operar a nivel de menú.

---

## Administración de permisos

Gestión de `Pq_Permiso`, es decir, de la asignación de acceso.

- Alta y edición de combinaciones usuario y rol.
- **Asignación individual** (un usuario, un rol) y **asignación masiva** (por usuario con varios roles, o por rol con varios usuarios). Detalle de flujos, validaciones y API en `mantenimiento-roles-permisos.md`.
- En MONO la empresa del despliegue es fija y puede no mostrarse en la UI.
- Eliminar una asignación equivale a quitar acceso al usuario.
- La combinación gestionada en MONO es funcionalmente `usuario + rol` sobre la instalación actual.
- Si por compatibilidad de esquema legado existiera una columna como `IDEmpresa`, en MONO no forma parte de la regla de negocio visible y puede mantenerse solo con un valor fijo de instalación hasta normalizar el modelo físico.

### PedidosWeb

- Los usuarios no se dan de alta desde este módulo; provienen del ERP / sincronización. El ABM de permisos solo **asigna roles** a usuarios ya existentes.
- No existe modo masivo «por empresa» (referencia Tango TR-013 update 03); en MONO aplican solo los modos **por usuario** y **por rol**.

---

## Administración de atributos de rol

Gestión de `PQ_RolAtributo` cuando el rol no tiene `AccesoTotal`.

- Permite marcar alta, baja, modificación y consulta por opción de menú.
- La combinación rol y opción de menú debe ser única.
- Estos permisos gobiernan visibilidad de menú y acciones en pantallas.

---

## Criterios comunes de estas pantallas

- Son mantenimientos administrativos del sistema, no procesos de negocio del usuario final.
- Deben seguir patrón ABM consistente con el resto del framework.
- La terminología visible debe ser funcional y comprensible para administradores.
- Los cambios en estas pantallas repercuten en acceso, visibilidad de procesos y acciones disponibles.

---

## Administración de empresas

No aplica en MONO como proceso operativo. La gestión de múltiples empresas queda solo como referencia documental para una variante MULTI.

---

## Relación con menú del sistema

Las opciones de administración deben existir en `pq_menus` con `routeName` y `enabled` correctos para que el sidebar dinámico pueda exponerlas. Ver `../01-experiencia-base/menu-general.md`.

## Relación con otros temas

- SPEC ([A1 cerrada](../../../05-open-spec/001-Generaliddes/SPEC-001-02-admin-mantenimiento-roles-permisos.md)): `SPEC-001-02-admin-mantenimiento-roles-permisos.md`
- Mantenimiento de roles y permisos (flujos detallados): `mantenimiento-roles-permisos.md`
- Seguridad y permisos: `seguridad-permisos.md`
- Menú y autorización: `menu-y-autorizacion.md`
- Grillas ABM: `../03-ui-transversal/grillas.md`
- Patrones ABM: `../03-ui-transversal/patrones-abm.md`

## Derivaciones esperables

Este documento debería permitir regenerar:

- administración de usuarios,
- administración de roles,
- administración de permisos,
- administración de atributos de rol,
- relación entre mantenimiento de seguridad y menú del sistema.

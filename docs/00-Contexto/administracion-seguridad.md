# Administración de tablas de seguridad

Documento de reingeniería de los procesos de **mantenimiento** de usuarios, roles, permisos y (en MULTI) empresas. Épica Seguridad.

Solo usuarios con perfil **administrador** (o permisos explícitos sobre las opciones de menú de administración) acceden a estas pantallas. Implementación alineada a **grillas ABM DevExtreme** (`grillas.md`).

---

## Administración de usuarios

Gestión de la tabla **`users`**.

### Capacidades

- Listar con filtros: código, nombre, email, activo, inhabilitado.
- **Alta:** código (único), nombre, email (único), contraseña inicial, supervisor, activo, inhabilitado.
- **Edición:** datos del usuario (el código puede ser inmutable si es clave de integración).
- **Baja lógica:** `inhabilitado = true` (no borrado físico). Usuario inhabilitado no puede hacer login.

### Reglas

- Contraseña inicial y cambios administrativos en hash.
- Validar formatos (email, longitud mínima de contraseña).
- Las asignaciones usuario–empresa–rol se gestionan en **Administración de permisos**, no en esta pantalla.

---

## Administración de roles

Gestión de **`Pq_Rol`**.

- Listar: nombre, descripción, `AccesoTotal`.
- Alta/edición: `NombreRol`, `DescripcionRol`, `AccesoTotal`.
- Eliminar rol solo si no tiene permisos asignados (o según política de cascada definida en spec).
- Atributos granulares por menú: pantalla separada (sección siguiente).

---

## Administración de permisos (asignaciones)

Gestión de **`Pq_Permiso`**: vincula usuario, empresa y rol.

- Listar con filtros por usuario, empresa, rol.
- Alta: seleccionar usuario, empresa, rol; combinación única.
- Edición: cambiar rol de una asignación existente.
- Eliminar: quita acceso del usuario a esa empresa (MULTI).

En **MONO**: la empresa es la única del despliegue (campo fijo u oculto en UI); el administrador asigna **usuario + rol**.

---

## Administración de atributos de rol

Gestión de **`PQ_RolAtributo`** cuando el rol **no** tiene `AccesoTotal`.

- Por cada opción de `pq_menus` relevante: marcar `Permiso_Alta`, `Permiso_Baja`, `Permiso_Modi`, `Permiso_Repo`.
- Si `AccesoTotal = true` en el rol, no hace falta cargar atributos (acceso implícito total).
- Combinación única por rol y opción de menú.

Estos permisos gobiernan visibilidad en menú (vía API) y acciones en pantallas (ej. eliminar en ABM).

---

## Administración de empresas (MONO / MULTI)

En **monoempresa** no se administra un catálogo multi-tenant: existe **una** configuración de empresa/base para el despliegue. El ABM completo de empresas (alta de tenants, `NombreBD` múltiples, theme por empresa) aplica solo a versión **MULTI** (bloque comentado abajo).

<!--
Versión MULTI — Administración de empresas (HU-011)

- Listar / filtrar PQ_Empresa (nombre, habilitada).
- Alta: NombreEmpresa, NombreBD (único, Company DB), Habilita, imagen, theme DevExtreme.
- Validar NombreBD (o código Tango) no duplicado; referencia lectura tabla EMPRESA (Tango).
- Edición y habilitar/inhabilitar; empresa inhabilitada no aparece en selector de usuarios.
- theme en PQ_Empresa aplica al seleccionar empresa (no aplica en MONO; ver apariencia-temas.md).
- Creación de BD física según política de infraestructura.
-->

---

## Relación con menú del sistema

Las opciones de administración (Usuarios, Roles, Permisos, Atributos de rol, etc.) deben existir en `pq_menus` con `routeName` y `enabled` correctos para el sidebar dinámico. Ver `Menu-general.md`.

-
# Seguridad: permisos, roles y autorización

Documento de reingeniería del modelo de acceso del sistema. Complementa `login-y-sesion.md` y la resolución de menú documentada en `menu-y-autorizacion.md`.

## Tablas conceptuales

| Tabla | Propósito |
|-------|-----------|
| `users` | Identidad y autenticación |
| `Pq_Rol` | Roles reutilizables |
| `Pq_Permiso` | Asignación usuario, empresa y rol |
| `PQ_RolAtributo` | Permisos granulares por rol y opción de menú |
| `pq_menus` | Árbol de opciones del sistema |

`users_identities` no aplica por el momento.

---

## Flujo general de autorización

1. Login exitoso del usuario.
2. Resolver contexto de instalación vigente.
3. Obtener rol o roles desde `Pq_Permiso`.
4. Si el rol tiene `AccesoTotal`, habilitar todas las opciones de menú permitidas por `pq_menus.enabled`.
5. Si no, filtrar por `PQ_RolAtributo`.
6. El menú y las pantallas deben respetar esos permisos.

En MONO no existe selección operativa de empresa: el contexto de instalación es fijo y único para todo el despliegue.

## Principios de autorización

- La autorización no se limita a mostrar u ocultar menú; también gobierna acciones dentro de cada proceso.
- El menú visible y las capacidades operativas deben responder al mismo criterio de seguridad.
- La ausencia de permiso válido equivale a ausencia de acceso operativo.

---

## Roles (`Pq_Rol`)

| Campo | Uso |
|-------|-----|
| `IDRol` | Clave |
| `NombreRol`, `DescripcionRol` | Identificación |
| `AccesoTotal` | Acceso total a todas las opciones habilitadas |

Un rol con `AccesoTotal` equivale a un perfil funcional amplio.

---

## Permisos o asignaciones (`Pq_Permiso`)

Vincula usuario, empresa y rol.

- En MULTI un usuario puede tener varias filas, una por empresa.
- En MONO suele existir una sola asignación por usuario porque la empresa del despliegue es única.
- Si el esquema físico heredado conserva una columna como `IDEmpresa`, en MONO puede quedar con valor constante por compatibilidad, sin convertirse en dimensión funcional visible.
- Sin permisos válidos, el usuario no debe poder operar después del login.
- Si un usuario posee más de un rol aplicable, el resultado funcional se interpreta como union de permisos.

---

## Atributos de rol (`PQ_RolAtributo`)

Cuando `AccesoTotal = false`, cada opción de menú puede tener permisos explícitos:

| Permiso | Significado habitual |
|---------|----------------------|
| `Permiso_Alta` | Crear |
| `Permiso_Baja` | Eliminar |
| `Permiso_Modi` | Modificar |
| `Permiso_Repo` | Consultar o informar |

La autorización de acciones en pantalla debe estar alineada con estos atributos.

Esto implica, por ejemplo:

- mostrar u ocultar acciones de alta,
- habilitar o no edición,
- permitir o no baja,
- permitir acceso a consultas o reportes.

---

## Acceso a pantallas de administración

Los ABM de usuarios, roles, permisos y atributos requieren rol de administrador del sistema o permisos equivalentes. Ver `administracion-seguridad.md`.

## Relación con otros temas

- Login y sesión: `login-y-sesion.md`
- Menú y autorización: `menu-y-autorizacion.md`
- Menú general: `../01-experiencia-base/menu-general.md`
- Grillas y acciones por permiso: `../03-ui-transversal/grillas.md`

## Referencia MULTI

En MULTI aparecen selección de empresa activa y validación por tenant. Ver `../05-variantes-y-alcance/mono-vs-multi-referencias.md`.

## Derivaciones esperables

Este documento debería permitir regenerar:

- administración de roles,
- administración de permisos,
- administración de atributos de rol,
- reglas de autorización en menú y pantallas,
- relación entre acceso total y permisos granulares.

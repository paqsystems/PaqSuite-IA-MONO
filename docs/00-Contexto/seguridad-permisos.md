# Seguridad: permisos, roles y autorización

Documento de reingeniería del modelo de acceso (épica Seguridad). Complementa `Login.md` y `Menu-general.md`.

## Tablas (Dictionary DB)

| Tabla | Propósito |
|-------|-----------|
| `users` | Identidad y autenticación |
| `Pq_Rol` | Roles reutilizables (conjuntos de permisos) |
| `Pq_Permiso` | Asignación **usuario → empresa → rol** |
| `PQ_RolAtributo` | Permisos granulares por rol y opción de menú |
| `pq_menus` | Árbol de opciones del sistema |
| `PQ_Empresa` | Empresas / tenant (en MONO: instancia única del despliegue) |

`users_identities` no aplica por el momento.

---

## Flujo de autorización (visión general)

```
Login (users + Sanctum)
    → Resolver empresa activa (MONO: fija; MULTI: selector / X-Company-Id)
    → Pq_Permiso: rol del usuario en esa empresa
    → Si rol.AccesoTotal → menú completo (pq_menus enabled)
    → Si no → filtrar por PQ_RolAtributo (Alta, Baja, Modi, Repo por opción)
    → API menú + pantallas respetan esos permisos
```

---

## Roles (`Pq_Rol`)

| Campo | Uso |
|-------|-----|
| `IDRol` | Clave |
| `NombreRol`, `DescripcionRol` | Identificación |
| `AccesoTotal` | Si `true`, el rol accede a **todas** las opciones de menú habilitadas sin definir atributos por ítem |

Administración: ver `administracion-seguridad.md` (ABM roles).

Un rol con `AccesoTotal` equivale a perfil supervisor/administrador funcional amplio.

---

## Permisos / asignaciones (`Pq_Permiso`)

Vincula **usuario + empresa + rol**.

- Clave compuesta única: `(IDRol, IDEmpresa, IDUsuario)`.
- Un usuario puede tener **varias** filas en MULTI (una por empresa).
- En **MONO**: típicamente una sola fila por usuario (empresa única del despliegue).
- Sin ningún permiso válido: el usuario **no** puede operar tras el login.

Administración: ABM de asignaciones (HU-013).

---

## Atributos de rol (`PQ_RolAtributo`)

Cuando `AccesoTotal = false`, cada opción de menú (`pq_menus`) puede tener permisos explícitos por rol:

| Permiso | Significado habitual |
|---------|----------------------|
| `Permiso_Alta` | Crear / agregar registros |
| `Permiso_Baja` | Eliminar (ej. acción en grilla ABM) |
| `Permiso_Modi` | Modificar |
| `Permiso_Repo` | Consultar / informes |

- Combinación única: `(IDRol, IDOpcionMenu, IDAtributo)`.
- Si el usuario tiene **varios roles** en la misma empresa (MULTI), la autorización de menú es **unión**: basta un rol que autorice la opción.

Administración: ABM atributos de rol (HU-014).

---

## Resolución del menú del usuario

Endpoint protegido (ej. `GET /api/v1/user/menu`):

1. Usuario autenticado (sesión/token).
2. En MONO: empresa implícita única. En MULTI: **empresa activa** obligatoria.
3. Obtener rol(es) desde `Pq_Permiso`.
4. Si `AccesoTotal` → todas las filas de `pq_menus` con `enabled = 1` y `text` no vacío.
5. Si no → solo ítems con al menos un permiso en `PQ_RolAtributo` para ese rol.
6. Respuesta jerárquica ordenada: `id`, `text`, `parentId`, `orden`, `routeName` / `procedimiento`.
7. Sin permisos o sin empresa (MULTI): lista vacía o menú mínimo (Inicio, Perfil) según diseño.

Detalle de sidebar y seed: `Menu-general.md`.

---

## Acceso a pantallas de administración

Los ABM de usuarios, roles, permisos y atributos (ver `administracion-seguridad.md`) requieren rol de **administrador del sistema** (o equivalente con `AccesoTotal` / permisos explícitos sobre esas opciones de menú).

---

## MONO vs MULTI

| Aspecto | MONO | MULTI |
|---------|------|-------|
| Empresa activa | Fija en despliegue | Selector + `X-Company-Id` |
| `Pq_Permiso` | Una empresa por instalación | Varias empresas por usuario |
| Validación API gestión | Sin header tenant; Company DB única | `X-Company-Id` obligatorio |

---

## Historias de usuario de origen

`001-Seguridad`: HU-012 (roles), HU-013 (permisos), HU-014 (atributos rol), HU-016 (API menú). HU-002 y validación tenant en MULTI.

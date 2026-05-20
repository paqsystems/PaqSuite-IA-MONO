# Login, sesión y contraseña

Documento de reingeniería del acceso al sistema y flujos de credenciales (épica Seguridad).

## Modelo de usuario (Dictionary DB)

Tabla **`users`** (sin uso de `users_identities` por ahora; dejar previsto en modelo para login social futuro):

| Campo | Reglas |
|-------|--------|
| `id` | Autoincremental |
| `codigo` | Único, no repetible (código de usuario para login) |
| `name` | Nombre de usuario |
| `email` | Único, no repetible |
| `password_hash` | Contraseña con **hash** (bcrypt o equivalente) |
| `activo` | Solo `true` puede autenticarse |
| `inhabilitado` | Si `true`, no puede acceder (baja lógica administrativa) |
| `first_login` | Si `true`, debe cambiar contraseña antes de usar el resto del sistema |

Autenticación API: **Laravel Sanctum**. El token se guarda en el frontend (localStorage o sessionStorage) e incluye datos para resolver permisos (`user_id`, código de usuario, etc.).

Campos de preferencia en la misma tabla (ver otros documentos mono):

- `locale` — idioma (ver `idioma.md`)
- `theme` — apariencia DevExtreme (ver `apariencia-temas.md`, menú avatar)
- `menu_abrir_nueva_pestana` — apertura de procesos en nueva pestaña (ver `menu-avatar.md`)

---

## Pantalla de login

### Controles visibles

- **Selector de idioma** disponible antes de autenticarse (ver `idioma.md`).
- Campos: **código de usuario** y **contraseña** (ambos obligatorios antes de enviar).
- Acción de ingreso.
- Enlace **¿Olvidaste tu contraseña?** (recuperación; ver más abajo).

### Validación de ingreso

1. Código y contraseña no vacíos.
2. Usuario existe en `users`.
3. `activo = true` e `inhabilitado = false`.
4. Contraseña coincide con `password_hash`.
5. Usuario tiene **al menos una asignación** en `Pq_Permiso` (en MONO: permiso sobre la única empresa del despliegue).
6. Si todo es válido: generar token Sanctum, cargar preferencias (`locale`, `theme`, `menu_abrir_nueva_pestana`, etc.) y preparar menú según permisos.

Ante rechazo: **mensaje genérico** sin revelar si el usuario existe o cuál fue el fallo.

Si no tiene permisos habilitados: mensaje específico de falta de acceso (sin detalles sensibles de credenciales).

### Flujo post-login (MONO)

En **monoempresa** no hay pantalla intermedia de selección de empresa: tras login válido se redirige al **layout principal** con la empresa única del despliegue ya implícita en contexto.

<!--
Versión MULTI — Post-login con varias empresas (HU-002)

- Una sola empresa en Pq_Permiso → layout principal con esa empresa activa.
- Varias empresas → pantalla selector de empresa; luego layout con X-Company-Id.
- Ninguna empresa → impedir acceso con mensaje apropiado.
-->

---

## Recuperación de contraseña (olvidé mi contraseña)

Flujo desde la pantalla de login (usuario **no** autenticado):

1. Enlace visible: *¿Olvidaste tu contraseña?*
2. Formulario para ingresar **email** registrado en `users`.
3. Si el email existe: generar token de restablecimiento con validez limitada (ej. 60 minutos) y enviar correo con enlace (requiere `MAIL_*` configurado; en desarrollo puede simularse).
4. Mensaje siempre **genérico** al usuario: *Si el email existe, recibirás instrucciones* (no revelar si el email está registrado).
5. El enlace abre formulario: contraseña nueva + confirmación.
6. Éxito: actualizar `password_hash`, invalidar token, redirigir a login con mensaje de éxito.
7. Token expirado o inválido: error claro sin exponer detalles internos.

Correo en el **idioma activo** de la interfaz en el momento de la solicitud.

---

## Cambio de contraseña (usuario autenticado)

Accesible desde menú avatar → **Cambiar contraseña** (popup/modal DevExtreme):

| Campo | Validación |
|-------|------------|
| Contraseña actual | Debe ser correcta |
| Contraseña nueva | Política de longitud/complejidad del producto |
| Confirmación | Debe coincidir con la nueva |

- Tras guardar: actualizar `password_hash` y `first_login = false`.
- Mensaje de éxito y cierre del modal.
- Error en contraseña actual: mensaje claro.

Si `first_login = true`, el usuario **debe** completar este flujo antes de acceder al resto del sistema (bloqueo de navegación hasta cambio exitoso).

---

## Cerrar sesión (logout)

Desde menú avatar → **Cerrar sesión** (acción inmediata, sin confirmación):

- Invalidar token en backend cuando aplique (Sanctum).
- Eliminar token y contexto de sesión en el cliente.
- En MONO: no hay contexto de empresa activa que limpiar más allá de la sesión global.
- Redirigir a pantalla de login.
- Peticiones posteriores con el token anterior → **401 Unauthorized**.

<!--
Versión MULTI — Logout

- Además, limpiar empresa activa persistida (localStorage) y X-Company-Id en cliente.
-->

---

## Post-login (layout principal)

Tras acceso exitoso:

- **Sidebar** con menú filtrado por permisos (ver `Menu-general.md`, `seguridad-permisos.md`).
- Header: selector de **idioma**, menú **avatar** (sin selector de empresa en MONO).
- Área de contenido (pestañas de procesos en desktop/tablet según shell).
- Tema DevExtreme según `users.theme` (ver `apariencia-temas.md`).

---

## Historias de usuario de origen

`001-Seguridad`: HU-001 (login), HU-003 (logout), HU-004 (cambio contraseña), HU-005 (recuperación). HU-002 (selección empresa) solo MULTI.

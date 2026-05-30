# Login, sesión y contraseña

Documento de reingeniería del acceso al sistema y de los flujos de credenciales en modo MONO.

## Modelo de usuario

Tabla `users` en Dictionary DB:

| Campo | Reglas |
|-------|--------|
| `id` | Identificador |
| `codigo` | Único, usado para login |
| `name` | Nombre legible |
| `email` | Único |
| `password_hash` | Contraseña hasheada |
| `activo` | Solo `true` puede autenticarse |
| `inhabilitado` | Si `true`, no puede acceder |
| `first_login` | Obliga cambio de contraseña inicial |

Autenticación API: Laravel Sanctum o equivalente.

Campos de preferencia asociados al usuario:

- `locale`: ver `../01-experiencia-base/idioma-multilingual.md`
- `theme`: ver `../01-experiencia-base/apariencia-temas.md`
- `menu_abrir_nueva_pestana`: ver `../01-experiencia-base/navegacion-pestanas.md`

---

## Pantalla de login

### Controles visibles

- Selector de idioma antes de autenticarse.
- Código de usuario.
- Contraseña.
- Acción de ingreso.
- Enlace "¿Olvidaste tu contraseña?".

### Resultado esperado del login exitoso

- Crear la sesión autenticada.
- Cargar preferencias del usuario.
- Resolver permisos y menú disponible.
- Redirigir al shell principal sin pasos intermedios de empresa en MONO.

### Validación de ingreso

1. Código y contraseña no vacíos.
2. Usuario existente.
3. `activo = true` e `inhabilitado = false`.
4. Contraseña válida.
5. El usuario posee al menos una asignación válida en `Pq_Permiso`.
6. Si todo es correcto, se genera sesión y se cargan preferencias y menú.

Ante rechazo de credenciales debe mostrarse mensaje genérico. Si el problema es ausencia de permisos, puede mostrarse un mensaje específico de falta de acceso.

### Flujo post-login

En MONO no existe selección intermedia de empresa: el usuario entra directamente al shell principal.

---

## Recuperación de contraseña

Flujo para usuario no autenticado:

1. Enlace visible desde login.
2. Formulario con email registrado.
3. Si el email existe, se genera token temporal y se envía correo.
4. El mensaje visible siempre es genérico.
5. El enlace permite definir nueva contraseña.
6. Al completar con éxito, se actualiza `password_hash` y se vuelve a login.

El correo debe salir en el idioma activo al momento de la solicitud.

---

## Cambio de contraseña

Accesible desde menú avatar:

| Campo | Validación |
|-------|------------|
| Contraseña actual | Debe ser correcta |
| Contraseña nueva | Debe cumplir política vigente |
| Confirmación | Debe coincidir |

- Éxito: actualizar `password_hash` y `first_login = false`.
- Error: mensaje claro y sin inconsistencias.
- Si `first_login = true`, el usuario debe completar este flujo antes de operar normalmente.

La política exacta de complejidad puede variar por producto, pero el flujo conceptual debe permanecer común.

---

## Logout

Desde el menú avatar:

- Invalidar token en backend cuando aplique.
- Limpiar token y contexto local.
- Redirigir a login.
- Cualquier uso posterior del token previo debe dar `401`.

En MONO no existe empresa activa adicional que limpiar.

---

## Reglas conceptuales de sesión

- La sesión autenticada habilita el acceso al shell y a los procesos autorizados.
- Las preferencias del usuario forman parte del estado funcional de la sesión.
- La pérdida de autenticación debe devolver al usuario a login sin dejar el shell en estado ambiguo.
- El cambio de contraseña, logout y recuperación son capacidades del mismo dominio funcional de acceso.

## Post-login y shell principal

Tras acceso exitoso el usuario entra al shell estándar:

- Header con logo, selector de idioma y menú avatar.
- Sidebar con menú filtrado por permisos.
- Área principal con dashboard o proceso.
- Footer con contexto de sesión y versión.

Detalle de layout: `../01-experiencia-base/shell-layout.md`.

## Referencia MULTI

La selección de empresa, el uso de `X-Company-Id` y la limpieza de contexto por tenant quedan fuera de MONO. Ver `../05-variantes-y-alcance/mono-vs-multi-referencias.md`.

## Derivaciones esperables

Este documento debería permitir regenerar:

- login de usuario,
- logout,
- recuperación de contraseña,
- cambio de contraseña,
- reglas de carga inicial de sesión y preferencias.

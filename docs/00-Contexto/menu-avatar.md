# Menú de usuario (avatar)

Documento de reingeniería: menú desplegable bajo el **avatar del usuario** en el extremo derecho del header principal.

## Presentación

- Ubicación: extremo derecho del frame superior.
- Avatar: foto de la ficha del usuario si existe; si no, imagen genérica de rostro humano.
- Al hacer clic se despliega el menú de acciones personales y de sesión.

El selector de **idioma** no forma parte de este menú; es un control independiente en la barra superior (ver `idioma.md`).

---

## Opciones del menú

### Perfil

Muestra y permite editar los datos del usuario autenticado (nombre, mail, etc.) según permisos de edición de perfil.

### Cambiar empresa activa

En sistemas **mono** no hay más de una empresa por usuario: esta opción queda **fuera de alcance** en la versión actual. El detalle siguiente se conserva comentado para la versión **MULTI**.

<!--
Versión MULTI — Cambiar empresa activa

Para usuarios con acceso a **más de una empresa**:

- Listar solo empresas con permiso (`Pq_Permiso`: `IDUsuario` + `IDEmpresa`).
- Mostrar nombre legible desde `PQ_Empresa`.
- Al elegir otra empresa: actualizar contexto global, reflejar nombre en header y enviar la nueva empresa en peticiones API (ej. header `X-Company-Id`).
- El backend valida permiso antes de operar con esa empresa.
- El token de sesión **no** se invalida; el usuario sigue autenticado.
- La vista actual se recarga o ajusta al nuevo contexto.

Si el usuario tiene una sola empresa, el selector puede ocultarse o mostrarse en solo lectura.
-->

### Apariencia

Permite elegir la **estética global** del sistema entre las apariencias predefinidas de **DevExtreme**.

- Acceso desde el menú avatar, etiqueta **Apariencia**.
- Presentación: **listbox** (o control equivalente) con todas las opciones de tema disponibles y nombre legible (ej. *Material Blue Claro*, *Generic Oscuro*).
- Al elegir una opción, el cambio se aplica **de inmediato** a toda la interfaz (shell, grillas, formularios, modales), sin redeploy.
- Catálogo cerrado de temas y reglas técnicas: ver `apariencia-temas.md`.
- Persistencia **por usuario** en `users.theme` (Dictionary DB), con el mismo criterio de conservación entre sesiones y dispositivos que otras preferencias del menú avatar.
- Si no hay tema guardado, se usa el tema por defecto del producto hasta que el usuario elija uno.

En **monoempresa** no existe configuración de apariencia por empresa ni en administración de empresas; esta opción del menú avatar es la **única** vía de personalización visual.

### Abrir en nueva pestaña

**Toggle** que define cómo se abren las opciones del **menú de navegación** (procesos):

| Valor | Comportamiento |
|-------|----------------|
| Desactivado (por defecto) | Navegación en la misma pestaña (SPA estándar) |
| Activado | Cada proceso se abre en nueva pestaña del navegador |

- Preferencia **por usuario**, no por empresa.
- Persistencia en Dictionary DB: `users.menu_abrir_nueva_pestana` (0 = misma pestaña, 1 = nueva pestaña).
- Debe conservarse entre sesiones, navegadores y equipos (server-side).
- **Solo aplica al frontend web**; en app móvil o PWA mobile no se ofrece.
- Al abrir nueva pestaña, la sesión (token) debe estar disponible sin re-login. En MULTI, también empresa activa (ver bloque comentado *Cambiar empresa*).

### Asistente IA

Acceso a ayuda operativa externa:

- Etiqueta visible: **Asistente IA**, con ícono distintivo.
- Al activar: abre el recurso en **nueva pestaña o ventana**; la pantalla del sistema permanece abierta.
- La URL destino **no** va hardcodeada en el componente: se resuelve por ruta o mecanismo interno de redirección y configuración backend/parámetros.
- Valor inicial previsto: chat compartido con documentación de `docs/99-Manual-Usuario`.
- Si no hay URL configurada o es inválida: ocultar la opción o informar que la ayuda no está disponible, sin navegación rota.

### Cambiar contraseña

Abre el flujo de cambio de contraseña (contraseña actual + nueva dos veces). Ver `Login.md`.

### Cerrar sesión

Cierra la sesión de forma segura (ver `Login.md`):

- Acción inmediata, sin modal de confirmación.
- Invalidar token en backend (Sanctum) si aplica.
- Limpiar almacenamiento de token en el cliente.
- Redirigir a login; peticiones con token previo → 401.

---

## Historias de usuario de origen

`001-Seguridad`: HU-003 (logout), HU-004 (cambio contraseña vía menú). `hu-anteriores` / generalidades: HU-003 (nueva pestaña), HU-010 (Asistente IA), apariencia mono. HU-002 (cambio empresa) solo MULTI.


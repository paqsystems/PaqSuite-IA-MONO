# Menú de usuario (avatar)

Documento de reingeniería del menú desplegable bajo el **avatar del usuario** en el extremo derecho del header principal.

**Layout:** ubicación y marco general en `shell-layout.md` y en `docs/_base/shell-layout-principal.md`.

**MONO vs MULTI (menú avatar):** en MONO la preferencia distintiva es **Apariencia**. En MULTI la distintiva es **Cambiar empresa activa**; la referencia se conserva en `../05-variantes-y-alcance/mono-vs-multi-referencias.md`.

## Presentación

- Ubicación: extremo derecho del frame superior.
- Avatar: foto de la ficha del usuario si existe; si no, avatar genérico.
- Al hacer clic se despliega el menú de acciones personales y de sesión.

El selector de idioma **no** forma parte de este menú; es un control independiente en la barra superior. Ver `idioma-multilingual.md`.

---

## Opciones del menú

### Perfil

Muestra y permite editar los datos del usuario autenticado según permisos de edición de perfil.

### Cambiar empresa activa

En MONO no aplica porque cada despliegue trabaja con una única empresa. La variante MULTI se conserva documentada solo como referencia.

### Apariencia

Permite elegir la estética global del sistema entre las apariencias predefinidas de DevExtreme.

- Acceso desde el menú avatar.
- Presentación sugerida: listbox o control equivalente con opciones legibles.
- El cambio se aplica de inmediato a toda la interfaz.
- Persistencia por usuario en `users.theme`.
- Reglas técnicas y catálogo cerrado: `apariencia-temas.md`.

En MONO esta es la vía distintiva de personalización visual; no existe tema por empresa.

### Abrir en nueva pestaña

Toggle que define cómo se abren las opciones del menú general:

| Valor | Comportamiento |
|-------|----------------|
| Desactivado | Navegación en la misma pestaña |
| Activado | Cada proceso se abre en nueva pestaña |

- Preferencia por usuario.
- Persistencia en `users.menu_abrir_nueva_pestana`.
- Solo aplica al frontend web.
- Detalle funcional: `navegacion-pestanas.md`.

### Asistente IA

Acceso a ayuda operativa externa:

- Abre el recurso en nueva pestaña o ventana.
- La URL no va hardcodeada en el componente.
- Si no hay recurso configurado, la acción se oculta o informa indisponibilidad.
- Detalle funcional: `ayuda-externa-asistente.md`.

### Cambiar contraseña

Abre el flujo de cambio de contraseña. Ver `../02-acceso-y-seguridad/login-y-sesion.md`.

### Cerrar sesión

Cierra la sesión de forma segura. Ver `../02-acceso-y-seguridad/login-y-sesion.md`.

- Acción inmediata, sin modal de confirmación.
- Invalidar token en backend cuando corresponda.
- Limpiar sesión en cliente.
- Redirigir a login.

---

## Orden recomendado de opciones

Sin fijar una implementación rígida, el menú avatar debería mantener una lógica reconocible:

1. perfil y datos del usuario,
2. preferencias personales de uso o apariencia,
3. accesos globales de ayuda,
4. acciones de seguridad y cierre de sesión.

## Persistencia de preferencias

Las opciones persistibles del menú avatar deben vivir como preferencias del usuario y no del proceso activo:

- idioma cuando la arquitectura lo trate como preferencia,
- apariencia,
- apertura en nueva pestaña,
- otras preferencias personales que el framework incorpore más adelante.

## Reglas de diseño funcional

- Las acciones del menú avatar son personales, de sesión o de ayuda global; no reemplazan al menú general de procesos.
- Debe mantenerse pequeño, reconocible y estable entre productos que usen el framework.
- El orden de opciones puede ajustarse por producto, pero el núcleo conceptual debe mantenerse.
- Las opciones distintivas de MONO no deben mezclarse con conceptos propios de MULTI dentro del mismo flujo visual.

## Derivaciones esperables

Este documento debería alcanzar para regenerar:

- HUs de preferencias de usuario,
- HUs de apertura en nueva pestaña,
- HUs de ayuda externa,
- flujo de logout y cambio de contraseña desde el avatar.

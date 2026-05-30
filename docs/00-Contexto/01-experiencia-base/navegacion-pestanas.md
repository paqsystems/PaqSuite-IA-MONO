# Navegación en misma o nueva pestaña

Documento de contexto de la preferencia de usuario para abrir procesos del menú general en la misma pestaña del navegador o en una nueva.

## Objetivo

Permitir que cada usuario adapte la navegación del sistema a su forma de trabajo, especialmente cuando necesita mantener varios procesos abiertos en paralelo.

## Alcance

Aplica al **frontend web** y a las opciones del **menú general** que navegan hacia procesos. No aplica a mobile ni al acceso a recursos externos como ayuda o documentación.

## Regla principal

Cada usuario dispone de una preferencia persistente:

| Valor | Comportamiento |
|-------|----------------|
| Desactivado | Abrir procesos en la misma pestaña |
| Activado | Abrir procesos en una nueva pestaña del navegador |

## Ubicación de la preferencia

- La configuración se expone en el **menú avatar**.
- Se presenta como toggle o control equivalente con redacción clara.
- La preferencia es personal y persiste entre sesiones.

## Persistencia

- Campo previsto: `users.menu_abrir_nueva_pestana`.
- La persistencia es **server-side** para que se conserve entre navegadores y equipos.
- El valor debe cargarse junto con la sesión o mediante endpoint de preferencias.

## Reglas funcionales

- La preferencia aplica al hacer clic en opciones del menú general que abren procesos.
- En modo "misma pestaña", la navegación sigue el comportamiento estándar de la SPA.
- En modo "nueva pestaña", la nueva pestaña debe reutilizar la sesión vigente sin exigir nuevo login.
- La preferencia no debe romper el comportamiento de expandir o colapsar nodos del sidebar.
- Esta capacidad no reemplaza posibles pestañas internas dentro del área principal; ambas estrategias pueden coexistir.

## Fuera de alcance

- Aplicaciones mobile o PWA en experiencia mobile.
- Accesos externos como **Asistente IA**, que siempre abren fuera del flujo principal.

## Relación con otros temas

- Menú avatar: `menu-avatar.md`
- Menú general: `menu-general.md`
- Login y sesión: `../02-acceso-y-seguridad/login-y-sesion.md`
- Shell principal: `shell-layout.md`

## Derivaciones esperables

Este documento alimenta specs y tareas sobre:

- preferencia de usuario,
- navegación del sidebar,
- conservación de sesión entre pestañas,
- pruebas E2E de apertura de procesos.

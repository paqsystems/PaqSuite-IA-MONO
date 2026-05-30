# Usuarios, roles y permisos: resumen conceptual

Documento de contexto de alto nivel para resumir cómo se encadenan identidad, roles, permisos y autorización en el framework MONO.

## Objetivo

Ofrecer una vista rápida y autosuficiente del modelo de seguridad para que, al regenerar historias, specs o tareas, exista un punto de partida conceptual único antes de bajar al detalle de cada proceso administrativo.

## Cadena conceptual

1. **Usuario**: representa la identidad autenticable del sistema.
2. **Rol**: representa un perfil funcional reutilizable.
3. **Permiso**: vincula usuario y rol dentro del alcance de la instalación.
4. **Atributo de rol**: define acciones permitidas por opción de menú cuando no existe acceso total.
5. **Menú visible y acciones disponibles**: resultado operativo de todo lo anterior.

## Entidades principales

| Entidad | Pregunta que responde |
|---------|------------------------|
| `users` | Quién accede |
| `Pq_Rol` | Qué perfil funcional tiene |
| `Pq_Permiso` | Qué rol se le asigna |
| `PQ_RolAtributo` | Qué puede hacer por opción |
| `pq_menus` | Qué procesos y opciones existen |

## Reglas clave en MONO

- La instalación trabaja sobre una única empresa operativa.
- El usuario necesita al menos una asignación válida para poder operar.
- `AccesoTotal` simplifica la autorización de menú.
- Sin `AccesoTotal`, la capacidad se determina por atributos del rol.
- Si hay varios roles aplicables, la autorización efectiva es la unión de permisos.

## Qué habilita este modelo

- construir el menú visible;
- decidir acceso a pantallas;
- habilitar o bloquear acciones como alta, baja, modificación o consulta;
- separar administración de seguridad de la operatoria del negocio.

## Relación con otros temas

- Login y sesión: `login-y-sesion.md`
- Seguridad y permisos: `seguridad-permisos.md`
- Menú y autorización: `menu-y-autorizacion.md`
- Administración de seguridad: `administracion-seguridad.md`

## Derivaciones esperables

Este documento sirve para regenerar:

- mapa conceptual de seguridad,
- historias de administración de usuarios, roles y permisos,
- specs de autorización y de menú dinámico.

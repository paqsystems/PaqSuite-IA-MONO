# MONO vs MULTI: referencias de alcance

Documento de contexto para separar con claridad qué reglas aplican al framework **monoempresa** y cuáles se conservan solo como referencia para una posible variante **multiempresa**.

## Objetivo

Evitar mezclar en cada documento el alcance vigente de MONO con comportamientos futuros de MULTI, manteniendo a la vez una referencia común para no perder trazabilidad conceptual.

## Regla de lectura

- En este repositorio, el alcance operativo es **MONO**.
- Toda mención a MULTI debe entenderse como referencia futura, no como requisito vigente salvo que una spec lo indique de forma explícita.

## Diferencias clave

| Tema | MONO | MULTI |
|------|------|-------|
| Empresa activa | Fija por despliegue | Seleccionable por usuario |
| Header y avatar | Sin selector de empresa | Puede mostrar cambio de empresa |
| API de negocio | Sin `X-Company-Id` | Con `X-Company-Id` o equivalente |
| Resolución de base | Una sola Company DB | Dictionary DB + Company DB por empresa |
| `Pq_Permiso` | Normalmente una asignación por usuario | Varias asignaciones por usuario |
| Apariencia | `users.theme` por usuario | Frecuentemente tema por empresa |
| Administración de empresas | No aplica | Puede existir ABM de empresas |

## Cómo documentar

- Los documentos de `_mono` deben expresar la regla vigente para MONO.
- Si hace falta conservar una variante MULTI, debe quedar claramente marcada como referencia y no como alcance actual.
- Cuando la diferencia sea amplia, conviene remitir a este documento en lugar de insertar largos bloques comentados en cada archivo.

## Temas donde la diferencia es más sensible

- login y post-login;
- menú avatar;
- menú y autorización;
- parámetros generales;
- apariencia y branding;
- resolución de base de datos.

## Relación con otros temas

- Shell: `../01-experiencia-base/shell-layout.md`
- Menú avatar: `../01-experiencia-base/menu-avatar.md`
- Login y sesión: `../02-acceso-y-seguridad/login-y-sesion.md`
- Menú y autorización: `../02-acceso-y-seguridad/menu-y-autorizacion.md`
- Parámetros generales: `../04-configuracion-global/parametros-generales.md`

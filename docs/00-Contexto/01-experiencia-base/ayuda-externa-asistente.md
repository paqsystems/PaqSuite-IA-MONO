# Ayuda externa y Asistente IA

Documento de contexto del acceso global a ayuda operativa externa desde el menú avatar.

## Objetivo

Dar al usuario una vía rápida y visible para consultar asistencia funcional, manuales o un chat compartido sin interrumpir el trabajo que está realizando dentro del sistema.

## Alcance

Aplica como acceso **global** del sistema, disponible desde cualquier pantalla post-login a través del menú avatar.

## Regla funcional

- La opción visible para el usuario se denomina **Asistente IA**.
- Se ubica en el **menú avatar**.
- Al activarla, abre el recurso externo en una nueva pestaña o ventana.
- La pantalla actual del sistema permanece abierta y la sesión principal no se altera.

## Configuración

- La URL destino no debe quedar hardcodeada en el componente visual.
- La navegación debe pasar por una ruta o mecanismo interno de redirección.
- El recurso debe poder reemplazarse sin modificar el frontend.
- El alcance inicial previsto es un chat compartido asociado a la documentación operativa del sistema.

La parametrización del destino puede resolverse de distintas formas según arquitectura:

- configuración global de la instalación cuando el recurso sea único para todo el sistema;
- configuración por módulo cuando el producto necesite ayudas diferenciadas;
- otro mecanismo controlado del backend o del framework, siempre que la UI no dependa de un valor fijo embebido.

Si existe un valor inicial concreto de despliegue, debe interpretarse como configuración inicial de la instalación y no como parte rígida del framework compartido.

## Comportamiento ante error o falta de configuración

- Si no hay URL configurada, la acción debe ocultarse o informar que la ayuda no está disponible.
- Si la URL es inválida o el recurso falla, la aplicación no debe dejar una navegación rota ni bloquear el flujo principal.

## Criterios de experiencia

- El ítem debe ser fácil de identificar y diferenciarse de acciones de perfil o cierre de sesión.
- Puede incluir un icono distintivo de ayuda o asistencia.
- Debe poder automatizarse mediante `data-testid` o convención equivalente.

## Relación con otros temas

- Menú avatar: `menu-avatar.md`
- Navegación por pestañas: `navegacion-pestanas.md`
- Parámetros generales, si se decide parametrizar el destino: `../04-configuracion-global/parametros-generales.md`
- Manuales o documentación operativa del producto, cuando el recurso de ayuda actúe como puerta de entrada a ese corpus documental.

## Derivaciones esperables

Este documento sirve de base para:

- spec de acceso a ayuda externa,
- definición del mecanismo de redirección,
- parametrización del recurso de ayuda,
- pruebas de navegación no disruptiva.

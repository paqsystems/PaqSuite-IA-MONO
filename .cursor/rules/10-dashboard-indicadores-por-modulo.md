# Regla: Indicadores de Dashboard en Historias de Módulo (MONOEMPRESA)

## Objetivo

Cada vez que se generen las historias de usuario de un módulo, deben incluirse indicadores/KPIs para el dashboard principal.

---

## Alcance

Aplicar cuando se generen HUs de:

* ventas
* stock
* producción
* cobranzas
* pedidos
* CRM
* logística
* etc.

---

## Reglas de indicadores

### 1. Disponibilidad

Los indicadores solo estarán visibles si:

* el módulo está instalado
* el módulo está habilitado en la aplicación

---

### 2. Alcance por rol

| Rol            | Alcance                |
| -------------- | ---------------------- |
| Supervisor     | todos los datos        |
| Usuario normal | solo sus propios datos |

---

### 3. Dashboard principal

Las historias del módulo deben incluir:

* KPIs
* cards
* métricas
* indicadores resumidos
* variaciones cuando corresponda

---

## Ejemplos

* pedidos pendientes
* ventas del período
* tareas vencidas
* producción del día
* cobranzas pendientes
* clientes activos

---

## Criterios mínimos

* indicadores visibles solo si el módulo está habilitado
* supervisor ve global
* usuario normal ve sus datos
* soportar actualización periódica
* soportar filtros de período cuando aplique

---

## Referencias

* `docs/_base/shell-layout-principal.md` (layout post-login; área principal / dashboard de inicio)

# Regla: Parámetros generales al generar HUs de módulo (MONOEMPRESA)

## Objetivo

Cada vez que se generen todas las historias de usuario de un módulo, debe incluirse:

* definición de parámetros generales,
* HU de mantenimiento de parámetros,
* claves configurables del módulo.

---

## Alcance

Aplica a módulos como:

* Ventas
* Stock
* Producción
* CRM
* Compras
* Logística
* Pedidos
* etc.

---

## Nombre del módulo

Definir:

```text id="g5w2tr"
PROGRAMA
```

sin espacios.

Ejemplos:

```text id="x95q0n"
Ventas
Stock
PartesProduccion
Pedidos
```

---

## Tabla de parámetros

Definir:

| CLAVE | TIPO_VALOR | CAPTION | TOOLTIP |
| ----- | ---------- | ------- | ------- |

Tipos:

| Tipo | Significado |
| ---- | ----------- |
| S    | string      |
| T    | text        |
| I    | integer     |
| D    | datetime    |
| B    | boolean     |
| N    | numeric     |

---

## Criterios obligatorios

* el módulo debe invocar el proceso general HU-007
* solo editar valores
* no permitir alta/baja libre
* UI homogénea por tipo
* soporte tooltip y caption

---

## Persistencia

Los parámetros residen en:

```text id="1o4fsu"
base operativa local
```

No existe:

* tenant
* company db
* dictionary db
* X-Company-Id

---

## Reglas de implementación

### Campo Programa

Debe coincidir exactamente entre:

* seeds
* menú
* backend
* documentación
* filtros

---

## Ejemplo

```markdown id="7kntnl"
## Parámetros módulo Ventas

PROGRAMA: Ventas

| CLAVE | TIPO | Descripción |
|---|---|---|
| requiere_aprobacion | B | Requiere aprobación |
| dias_edicion | I | Días permitidos |
| monto_maximo | N | Monto máximo |
```

---

## Referencias

* `.cursor/rules/multi/13-parametros-generales-ui-listado-y-edicion-por-tipo.md` ó `.cursor/rules/mono/13-parametros-generales-ui-listado-y-edicion-por-tipo.md`
* `.cursor/rules/multi/10-dashboard-indicadores-por-modulo.md` ó `.cursor/rules/mono/10-dashboard-indicadores-por-modulo.md`
* `docs/03-historias-usuario/000-Generalidades/HU-007-Parametros-generales.md`

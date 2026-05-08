# 32 — Parámetros generales (HU-007): listado homogéneo y edición por tipo MONOEMPRESA

## Objetivo

Separar:

* consulta,
* visualización,
* edición,

de parámetros generales usando controles específicos según `tipo_valor`.

---

## Alcance

Aplica a todos los módulos que reutilicen HU-007.

Ejemplo:

```text id="6m0pah"
/parametros/Ventas
/parametros/Stock
/parametros/Pedidos
```

---

## Columnas soportadas

| Código | Tipo        | Columna        |
| ------ | ----------- | -------------- |
| S      | string      | Valor_String   |
| T      | texto largo | Valor_Text     |
| I      | entero      | Valor_Int      |
| D      | fecha       | Valor_DateTime |
| B      | boolean     | Valor_Bool     |
| N      | decimal     | Valor_Decimal  |

---

## Listado

La grilla debe mostrar:

* caption
* clave
* valor visible
* tooltip

---

## Reglas de visualización

### Booleanos

Mostrar:

```text id="h5o2ui"
Sí / No
```

mediante i18n.

No mostrar:

```text id="rhtx1u"
true / false
```

---

## Otros tipos

* fechas formateadas
* números localizados
* vacíos homogéneos

---

## Restricciones

No permitir:

* edición inline genérica
* input universal texto
* mezcla de controles incompatibles

---

## Edición por tipo

Al editar:

| Tipo | Control    |
| ---- | ---------- |
| B    | checkbox   |
| I    | entero     |
| N    | decimal    |
| S    | texto      |
| T    | textarea   |
| D    | fecha/hora |

---

## Modal obligatorio

La edición debe realizarse mediante:

```text id="gr5azg"
modal
```

o panel equivalente en misma ruta.

---

## Flujo

* editar
* guardar
* cancelar
* refrescar listado

---

## Persistencia MONO

Los parámetros residen en:

```text id="2lt7gt"
base operativa local
```

No existe:

* tenant
* X-Company-Id
* company db
* dictionary db

---

## Tests

### data-testid

```text id="dypv0r"
parametrosGral.grid
parametrosGral.editar.{clave}
parametrosGral.editor.{clave}
```

---

## Referencias

* `.cursor/rules/11-parametros-generales-por-modulo.md`
* `.cursor/rules/12-plan-tareas-hu-parametros-generales.md`

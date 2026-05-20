# Grillas pivot (tablas dinámicas)

Documento de reingeniería para vistas de datos agrupados en formato pivot, complementario a `grillas.md`.

## Qué es un pivot en PaqSystems

Un **pivot** es una grilla orientada al análisis: filas, columnas y valores se reorganizan para resumir datos (por cliente, período, empleado, etc.). En web se implementa con **DevExtreme PivotGrid** (o vista equivalente marcada como pivot en el proceso).

Comparte con las grillas tabulares:

- Identificación por `proceso` + `grid_id`
- Permisos y filtros del proceso
- Estándar visual DevExtreme

Las particularidades de pivot se documentan aquí.

---

## Comportamiento esperado

- El usuario puede arrastrar campos a zonas de fila, columna, valor y filtro según el modelo del pivot.
- Los totales y subtotales respetan el tipo de dato (mismas reglas que grillas: no sumar strings, etc.).
- La exportación a Excel ofrece la modalidad **Tabla dinámica** (ver `grillas.md`): el archivo XLSX conserva estructura pivot para expandir/colapsar y filtrar en Excel.
- Esa modalidad **solo** aparece en procesos marcados como pivot; en grillas planas no se muestra.

---

## Exportación pivot → Excel

| Aspecto | Detalle |
|---------|---------|
| Formato destino | XLSX con tabla dinámica de Excel |
| Datos | Vista actual (filtros y disposición pivot aplicados) |
| Permisos | Igual que la grilla en pantalla |
| Límite de filas | Mismo criterio que exportación general (ej. 10.000) |

---

## Totalizadores

los datos que se definen en la sección datos, deben poder asignarseles las operaciones de acumulación conforme a las propiedades que ofrece DevExtreme (con botón derecho sobre el dato seleccionado). considerar las posibilidades de acumulación según el tipo de dato:
numéricos : sumar, contar, máximo, minimo, promedio, etc.
string : contar, maximo, minimo
fecha : maximo, minimo, contar
etc. 

---
## Relación con layouts

Si el proceso pivot admite layouts persistentes (`pq_grid_layouts`), deben persistir la disposición de campos en zonas pivot además de filtros y totales, en la medida que el control lo soporte.

## Layouts y Excel vinculado visualmente a la grilla

sigue las mismas reglas que se define en este tópico en el documento docs/00_contexto/_mono/grillas.md

## Intercalación entre grilla y pivots en los informes

sigue las mismas reglas que se define en este tópico en el documento docs/00_contexto/_mono/grillas.md

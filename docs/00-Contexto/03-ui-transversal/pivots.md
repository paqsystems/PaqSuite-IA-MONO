# Grillas pivot (tablas dinámicas)

Documento de reingeniería para vistas de datos agrupados en formato pivot, complementario a `grillas.md`.

## Qué es un pivot

Un pivot es una vista orientada al análisis: reorganiza datos en filas, columnas y valores para resumir información por cliente, período, empleado u otras dimensiones.

En web se implementa con DevExtreme PivotGrid o control equivalente marcado como pivot dentro del proceso.

El pivot no reemplaza a la grilla de detalle. Ambos modos deben poder convivir cuando el proceso lo justifique.

## Capacidades esperadas

- Arrastrar campos a zonas de fila, columna, valor y filtro.
- Aplicar totales y subtotales coherentes con el tipo de dato.
- Exportar a Excel preservando la estructura pivot cuando el proceso lo permita.
- Mantener el mismo estándar visual y de permisos del proceso base.
- Partir de una pivot base util y no de una configuracion vacia cuando el proceso se abra por primera vez.

## Totalizadores

Los campos de datos deben ofrecer operaciones compatibles con su tipo:

- numéricos: sumar, contar, máximo, mínimo, promedio;
- string: contar, máximo, mínimo;
- fecha: máximo, mínimo, contar.

## Exportación pivot a Excel

La accion de exportacion debe ubicarse en la zona superior inmediata del pivot, junto con layouts y demas acciones generales del bloque analitico.

### Modalidades

| Modalidad | Contenido |
|-----------|-----------|
| Basico | Exporta los datos resultantes en forma simple |
| Tabla dinamica | Exporta la estructura pivot para seguir analizandola en Excel |

### Regla para la modalidad basica

- Exporta la matriz resultante visible, con encabezados y totales de la vista actual.
- Prioriza el contenido de datos por sobre la estructura interactiva del pivot.
- Sirve para compartir resultados o reutilizarlos fuera del sistema sin conservar el comportamiento dinamico completo.

### Regla para la modalidad tabla dinamica

- Preserva la estructura del pivot.
- Mantiene jerarquia de filas y columnas, subtotales y totales.
- Permite expandir, colapsar y seguir filtrando en Excel cuando la tecnologia elegida lo soporte.
- Solo debe ofrecerse cuando el proceso este efectivamente en modo pivot.

| Aspecto | Detalle |
|---------|---------|
| Formato destino | XLSX con estructura pivot |
| Datos | Vista actual con filtros aplicados |
| Permisos | Los mismos de la pantalla |
| Límites | Mismo criterio que exportación general |

## Relación con layouts

Si el proceso admite layouts persistentes, deben guardarse tambien:

- disposicion de campos en zonas pivot,
- filtros aplicados,
- totalizadores y agregaciones seleccionadas,
- configuracion funcional de la vista analitica que el control permita persistir.

### Operaciones esperadas

| Accion | Comportamiento |
|--------|----------------|
| Guardar | Actualiza el layout pivot seleccionado |
| Guardar como | Crea un layout pivot nuevo a partir del vigente |
| Cargar | Aplica un layout disponible para esa consulta |
| Eliminar | Solo permitido para el creador |

### Reglas de comparticion

- Todos los layouts o pivots guardados son visibles y utilizables por todos los usuarios autorizados al proceso.
- Solo el creador puede modificar o eliminar uno existente.
- Cualquier usuario puede basarse en uno compartido para crear uno propio.
- Los controles de layouts deben convivir visualmente con la exportacion en la parte superior inmediata del bloque pivot.

## Alternancia con grilla

Cuando convivan grilla y pivot en un mismo informe, se respeta la regla común documentada en `grillas.md`: el usuario puede alternar entre ambas vistas y la inicial es siempre la grilla.

## Relación con otros temas

- Grillas: `grillas.md`
- Exportaciones: `exportaciones.md`
- Apariencia: `../01-experiencia-base/apariencia-temas.md`

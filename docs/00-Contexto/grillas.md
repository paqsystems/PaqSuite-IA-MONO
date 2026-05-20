# Grillas (listados y ABM)

Documento de reingeniería: describe el comportamiento esperado de las grillas en PaqSystems. Sirve como fuente en lenguaje natural para volver a derivar historias de usuario y especificaciones técnicas.

## Principio general

Toda pantalla que muestre datos tabulares debe utilizar **incondicionalmente** el control de grilla de **DevExtreme**.

| Plataforma | Presentación |
|------------|--------------|
| Web (desktop/tablet) | Formato tabular, similar a una planilla Excel |
| Mobile | Formato de etiquetas (kardex), no tabla densa |

El estándar visual y de comportamiento debe alinearse con la regla de producto *full DevExtreme* (ver `apariencia-temas.md`).

---

## Características obligatorias de toda grilla

Estas capacidades aplican a **todas** las grillas, salvo que una definición de proceso las restrinja explícitamente.

### Columnas y datos

a) Poner a disposición **todos los campos posibles** de la tabla o del proceso mostrado. Si la definición del proceso no indica columnas, se muestran por defecto todos los campos disponibles; en cualquier caso el usuario puede **agregar o quitar columnas** desde el selector de columnas de la grilla.

b) Permitir **ordenar** haciendo clic en el caption de cualquier columna.

c) Permitir **reubicar columnas** arrastrando el título de una columna a otra posición.

### Agrupación y totalización

d) Presentar siempre la **barra superior de agrupación**: el usuario puede arrastrar una columna a esa zona para agrupar filas.

e) Presentar siempre la **barra inferior de totalización**: sobre cualquier columna se pueden aplicar funciones según el tipo de dato (contar, sumar, promediar, mínimo, máximo, etc.). Las operaciones inválidas para el tipo no se ofrecen (por ejemplo, no se suman campos string, fechas ni booleanos).

### Procesos ABM

f) En procesos de alta-baja-modificación, habilitar el botón **Agregar** (`+`) nativo de la grilla DevExtreme.

g) Las acciones sobre cada registro (eliminar, modificar, ver detalle, etc.) se realizan con **íconos de acción de DevExtreme** ubicados a la derecha de la fila. Evitar botones de texto y columnas estáticas dedicadas solo a acciones.

h) En grillas ABM debe existir la acción **Eliminar** por registro (ver sección *Eliminación en ABM*).

i) Los botones por fila deben estar siempre a la vista del usuario. dejarlos fijos o inmovil, si la cantidad de columnas obliga a un scrolleo a la derecha.

---

## Identificación de una grilla

Cada instancia de grilla en el sistema se identifica de forma única mediante:

| Clave | Origen | Uso |
|-------|--------|-----|
| `proceso` | Valor de `pq_menus.procedimiento` del ítem de menú que abre la pantalla | Agrupa layouts, exportación y telemetría |
| `grid_id` | Identificador lógico dentro de la pantalla | Distingue varias grillas en el mismo proceso (ej. `default`, `master`, `detalle`) |
| `layout_name` | Nombre elegido por el usuario al guardar un formato | Nombre legible del layout guardado |

Los layouts y preferencias de grilla se filtran por **proceso + grid_id**.

---

## Layouts persistentes (formatos guardados)

Los usuarios que trabajan habitualmente con grillas deben poder **guardar y recuperar formatos personalizados** sin reconfigurar la vista en cada acceso.

### Qué incluye un layout

Un layout guardado persiste, como mínimo:

- Columnas visibles y su orden
- Filtros aplicados
- Agrupaciones
- Ordenamiento
- Totalizadores configurados en la barra inferior

### Operaciones sobre layouts

| Acción | Comportamiento |
|--------|----------------|
| **Guardar** | Si hay un layout seleccionado, actualiza ese layout. Si está seleccionada la plantilla original del sistema, actúa como *Guardar como…* |
| **Guardar como…** | Crea un layout nuevo a partir del estado actual, con otro nombre |
| **Cargar** | Al abrir la pantalla, se aplica el último layout usado por ese usuario (si existe). El usuario puede elegir otro layout de la lista |
| **Eliminar** | Solo el usuario que creó el layout puede eliminarlo. Los layouts de otros usuarios no muestran eliminar (o aparece deshabilitado) |

### Compartición

Los layouts son **compartidos entre usuarios**: cualquiera puede ver y aplicar un layout definido por otro. Solo el **creador** puede modificarlo o eliminarlo.

### Persistencia

Los datos se almacenan en la tabla `pq_grid_layouts` (Dictionary DB): `id`, `user_id`, `proceso`, `grid_id`, `layout_name`, `layout_data`, `is_default`, `created_at`, `updated_at`.

El “último layout usado” por usuario puede resolverse con registro de uso, campo en preferencias o mecanismo equivalente definido en implementación.

---

## Exportación a Excel

Toda grilla debe ofrecer en su toolbar la acción **Exportar a Excel**.

### Disponibilidad

- El botón está habilitado solo si hay datos exportables.
- Sin datos: botón deshabilitado y mensaje *No hay datos para exportar*.
- Convención de prueba automatizada: `grid.{proceso}.{grid_id}.exportExcel`.

### Modalidades

El usuario elige la modalidad antes de exportar (menú o diálogo). Por defecto: **planilla formateada**.

| Modalidad | Contenido | Uso típico |
|-----------|-----------|----------|
| **Planilla básica** | Encabezados + filas sin formato de celda, sin totales; datos crudos | Importaciones masivas, procesamiento externo |
| **Planilla formateada** | Encabezados resaltados, fechas y números según locale, totales en pie, anchos ajustados | Informes para usuarios de negocio |
| **Tabla dinámica** | Solo en vistas pivot; estructura pivot preservada en Excel | Análisis interactivo en Excel |

Opcionalmente: exportar solo filas seleccionadas si la grilla admite selección múltiple.

### Alcance de datos exportados

- Se respetan filtros, orden y agrupación de la vista actual.
- Con paginación: opción de exportar página actual o todas las filas (con límite razonable, ej. 10.000 filas; informar si se supera).
- Los datos exportados obedecen los **mismos permisos** que la grilla en pantalla.
- Nombre de archivo descriptivo: `{proceso}_{fecha}.xlsx` o similar.
- Formato: **XLSX**.

La exportación se realiza en el cliente con los datos ya cargados, salvo que la grilla use paginación servidor sin dataset completo en memoria; en ese caso puede requerirse un endpoint de exportación por proceso.

---

## Eliminación en grillas ABM

En pantallas ABM, cada fila expone la acción **Eliminar** (ícono en columna de acciones).

### Permisos y reglas

- Visible/habilitada solo si el usuario tiene **permiso de baja** (`Permiso_Baja` del ítem de menú del ABM).
- Si el registro no es eliminable por estado o dependencias, el control queda deshabilitado u oculto según el caso.
- La eliminación afecta al registro de **esa fila** (no a una selección ambigua).

### Flujo de usuario

1. Clic en Eliminar → **modal de confirmación** con identificador del registro (código, nombre, etc.).
2. Cancelar cierra sin cambios.
3. Confirmar → llamada al endpoint de baja del recurso.
4. Éxito → refresco de grilla o remoción de fila + mensaje de confirmación.
5. Error (403, 422 por dependencias, etc.) → mensaje claro; la grilla no debe quedar en estado inconsistente.

La política de baja (lógica `activo=0` vs borrado físico) la define cada recurso en backend.

---

## Relación con otros documentos mono

| Tema | Documento |
|------|-----------|
| Tablas dinámicas / pivot | `pivots.md` |
| Apariencia y temas DevExtreme | `apariencia-temas.md` |
| Menú y procesos | `Menu-general.md` |
| Permisos (baja, etc.) | `seguridad-permisos.md` |
| Parámetros por módulo | `parametros-generales.md` |

## Layouts y Excel vinculado visualmente a la grilla

Ambas características se deben mostrar en la parte superior inmediata de la grilla o pivot. no ubicarla en ninguna otra sección de la pantalla.

## Intercalación entre grilla y pivots en los informes

Cuando coexisten grila y pivot, debe haber un control que permita el intercanbio de un control a otro. puede ser un botón que cambie de caption para pasar a la otra modalidad cada vez que se presiona, o un toggle donde "false" es "grilla"y "true" es pivot. Iniciar siempre como grilla

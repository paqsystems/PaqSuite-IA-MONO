# Grillas (listados y ABM)

Documento de reingeniería del comportamiento esperado de las grillas en PaqSuite. Sirve como fuente en lenguaje natural para derivar historias y especificaciones técnicas.

## Principio general

Toda pantalla que muestre datos tabulares debe utilizar la grilla de DevExtreme.

| Plataforma | Presentación |
|------------|--------------|
| Web | Formato tabular |
| Mobile | Formato de etiquetas o kardex |

El estándar visual debe alinearse con `../01-experiencia-base/apariencia-temas.md`.

---

## Capacidades obligatorias

### Columnas y datos

- Mostrar todos los campos disponibles del proceso, salvo restricción explícita.
- Permitir agregar o quitar columnas desde el selector de columnas.
- Permitir ordenar por cualquier caption.
- Permitir reubicar columnas.

### Agrupación y totalización

- Barra superior de agrupación disponible.
- Barra inferior de totalización siempre disponible.
- Las funciones ofrecidas dependen del tipo de dato.

### Procesos ABM

- Botón **Agregar** nativo de la grilla en procesos ABM.
- Acciones por fila con iconos DevExtreme, ubicadas a la derecha.
- Acción **Eliminar** disponible según permisos y reglas del recurso.
- Las acciones por fila deben quedar visibles aun con scroll horizontal.

---

## Identificación de una grilla

Cada instancia de grilla se identifica por:

| Clave | Origen | Uso |
|-------|--------|-----|
| `proceso` | `pq_menus.procedimiento` | Agrupa layouts, exportación y telemetría |
| `grid_id` | Identificador lógico en la pantalla | Distingue múltiples grillas |
| `layout_name` | Nombre del formato guardado | Identifica un layout |

Los layouts y preferencias se filtran por `proceso + grid_id`.

---

## Layouts persistentes

El usuario debe poder guardar y recuperar formatos personalizados.

Los controles de layouts deben estar visibles en la zona superior inmediata de la grilla, junto con las acciones propias del listado. No corresponde ubicar estas acciones en otra sección de la pantalla.

### Qué incluye un layout

- Columnas visibles y orden.
- Filtros.
- Agrupaciones.
- Ordenamiento.
- Totalizadores.
- Anchos, visibilidad y demás propiedades del formato vigente que el control permita persistir sin ambiguedad funcional.

### Operaciones

| Acción | Comportamiento |
|--------|----------------|
| Guardar | Actualiza el layout seleccionado |
| Guardar como | Crea un layout nuevo |
| Cargar | Aplica el layout elegido o el último usado |
| Eliminar | Solo disponible para el creador |

### Reglas de uso

- Cada grilla se identifica por `proceso + grid_id`.
- Todos los usuarios pueden ver y aplicar layouts existentes del mismo proceso y grilla.
- Solo el creador puede modificar o eliminar un layout guardado por ese mismo usuario.
- Los layouts **propios** del usuario se distinguen en el selector con sufijo **` (*)`** al final del nombre.
- Un usuario puede partir de un layout ajeno y generar uno nuevo propio mediante **Guardar como**.
- **Plantilla del sistema** (`layoutId: null`) restaura la grilla original del proceso (columnas por defecto); no persiste fila en BD.
- Si el formato visible corresponde a la plantilla original del sistema, la acción **Guardar** se interpreta como **Guardar como** (no altera la plantilla base).
- Al abrir la pantalla, el sistema debería restaurar el ultimo layout utilizado por el usuario si existe.

### Compartición

Los layouts son compartidos entre usuarios, pero solo el creador puede modificarlos o eliminarlos.

Persistencia prevista en `pq_grid_layouts`.

---

## Exportación a Excel

Toda grilla debe ofrecer la acción **Exportar a Excel**.

La acción debe estar disponible en la toolbar superior inmediata de la grilla, en convivencia con layouts y otras acciones generales del listado.

### Disponibilidad

- Habilitada solo si hay datos exportables.
- Sin datos, el botón se deshabilita y se informa al usuario.
- La exportación toma como base el formato vigente de la vista al momento de ejecutarse.

### Modalidades

| Modalidad | Contenido | Uso típico |
|-----------|-----------|------------|
| Planilla basica | Datos crudos tal como se ven en la vista, sin formatos avanzados | Procesamiento externo |
| Planilla formateada | Encabezados, formatos por tipo de dato, anchos razonables y totales | Uso de negocio |
| Tabla dinamica | Solo para vistas pivot | Analisis interactivo |

### Regla para la modalidad basica

- Exporta la grilla "asi como esta" en cuanto a columnas, filtros, orden y agrupaciones visibles.
- No agrega formato especial de Excel.
- Los valores se trasladan en forma simple, priorizando fidelidad del dato por sobre la presentacion.

### Regla para la modalidad formateada

- Mantiene el formato vigente de la vista y ademas aplica formato de Excel segun el tipo de dato.
- Las fechas se exportan en formato legible acorde al **locale activo** (i18n).
- Los campos enteros se exportan sin decimales.
- Los campos numericos decimales conservan la cantidad de decimales del campo (`column.format`; fallback 2 decimales).
- Los booleanos se exportan como texto **VERDADERO** / **FALSO** (o equivalente i18n del locale).
- Los importes, cantidades y porcentajes deben respetar su categoria funcional cuando la definicion del proceso la conozca.
- Incluye encabezados resaltados (negrita + fondo gris), anchos de columna razonables y **totalizadores de pie** (`totalFooter` / `groupFooter`) cuando correspondan.

### Alcance

- Respeta filtros, orden y agrupación.
- Puede ofrecer exportación de página actual o de todo el dataset con límites razonables.
- Debe obedecer los mismos permisos que la grilla visible.
- El nombre del archivo debe ser descriptivo usando como base el `proceso` y la fecha.

---

## Eliminación en ABM

La acción **Eliminar** se expone por fila.

### Reglas

- Visible solo con `Permiso_Baja`.
- Puede deshabilitarse por estado o dependencias del registro.
- Aplica únicamente a la fila seleccionada.

### Flujo

1. El usuario hace clic en eliminar.
2. Se muestra confirmación con identificación del registro.
3. Si confirma, se ejecuta la baja.
4. La grilla se refresca y se informa el resultado.

La política de baja lógica o física depende del recurso.

---

## Informes con grilla y pivot

Cuando un proceso soporte ambas vistas, debe existir un control claro para alternar entre grilla y pivot. La vista inicial es siempre la grilla.

## Relación con otros temas

- Pivots: `pivots.md`
- Exportaciones: `exportaciones.md`
- Apariencia: `../01-experiencia-base/apariencia-temas.md`
- Menú general: `../01-experiencia-base/menu-general.md`
- Permisos: `../02-acceso-y-seguridad/seguridad-permisos.md`
- Patrones ABM: `patrones-abm.md`

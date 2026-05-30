# Parámetros generales por módulo

Documento de reingeniería del proceso transversal de configuración en `PQ_PARAMETROS_GRAL`.

## Objetivo

Permitir que usuarios con permiso de configuración adapten el comportamiento del sistema por módulo sin modificar código, editando solo valores de parámetros previamente definidos.

---

## Invocación desde el menú

- Existe un proceso general reutilizable de mantenimiento de parámetros.
- Cada módulo que tenga parámetros expone un ítem de menú cuyo `pq_menus.procedimiento` coincide con el valor de `Programa`.
- Al abrir desde ese ítem, la pantalla muestra solo filas del módulo correspondiente.

En MONO, la coincidencia entre `Programa` y `pq_menus.procedimiento` debe resolverse con el mismo valor canónico de módulo. Para evitar desajustes por diferencias de casing en seeds, backend o SQL Server, la resolución funcional puede tratar esa comparación de forma insensible a mayúsculas y minúsculas, sin cambiar por ello el nombre canónico documentado del módulo.

Cada módulo documenta sus claves, tipos y defaults en su propia spec; este documento define el marco común del proceso.

## Alcance conceptual del proceso

Este mantenimiento existe para variar comportamiento del sistema sin cambiar código fuente, pero siempre dentro de un conjunto de claves previamente definidas.

Por lo tanto:

- la pantalla edita configuraciones existentes;
- no crea nuevas claves libremente;
- no redefine la estructura del módulo;
- no reemplaza seeds ni definiciones funcionales del producto.

---

## Base de datos

Tabla `PQ_PARAMETROS_GRAL` en la Company DB del despliegue.

| Campo | Uso |
|-------|-----|
| `Programa` | Módulo |
| `Clave` | Identificador del parámetro |
| `tipo_valor` | Define qué columna `Valor_*` aplica |
| `CAPTION` | Etiqueta breve del parámetro |
| `TOOLTIP` | Ayuda por fila |
| `Valor_*` | Valor persistido según tipo |

Los registros iniciales se cargan por seed. La pantalla no permite alta ni baja de filas; solo edición de valores.

---

## Interfaz de usuario

### Listado

- Columnas: `Clave`, caption y valor mostrado como texto homogéneo.
- Booleanos con etiquetas localizadas.
- Tooltip por fila cuando exista.
- Implementación alineada a grilla o listado DevExtreme.

### Edición por fila

- Botón **Editar** por fila.
- Modal o panel con un solo control acorde al tipo de dato.
- Guardar valida tipo y rango.
- Cancelar descarta cambios.

Los textos del marco de pantalla deben seguir las reglas de i18n.

### Reglas de presentación

- El valor en el listado se muestra como texto homogéneo, independientemente del tipo físico de almacenamiento.
- La edición debe respetar el tipo funcional del parámetro y no usar un control genérico para todo.
- `CAPTION` y `TOOLTIP` forman parte de la experiencia visible y deben resolverse según idioma activo.
- Los textos marco de la pantalla y sus acciones deben seguir claves de i18n del framework; esto no convierte al proceso en un catálogo libre de etiquetas editables por usuario.
- Cuando `CAPTION` o `TOOLTIP` existan como metadatos del parámetro, se interpretan como definición controlada por seed o especificación del módulo, no como texto improvisado desde la UI.

---

## Acceso y permisos

En MONO no existe `X-Company-Id`: siempre se trabaja contra la única Company DB del despliegue.

Solo usuarios con permiso de configuración del módulo pueden acceder a este proceso.

## Reglas funcionales

- Cada fila representa una capacidad configurable del módulo.
- Los valores deben ser validados antes de persistirse.
- La edición de un parámetro no debería requerir entender el tipo físico de base de datos.
- Si un módulo no tiene parámetros definidos, no corresponde inventarlos desde la UI.
- Las claves, captions y tooltips deberían surgir de definiciones controladas por seed o por la especificación funcional del módulo.
- El proceso reutilizable filtra por módulo, pero no redefine la semántica de cada clave: esa definición sigue perteneciendo a la documentación del módulo que invoca este mantenimiento.

---

## Relación con otros temas

- Menú general y `procedimiento`: `../01-experiencia-base/menu-general.md`
- Idioma: `../01-experiencia-base/idioma-multilingual.md`
- Permisos: `../02-acceso-y-seguridad/seguridad-permisos.md`
- Estándar visual: `../01-experiencia-base/apariencia-temas.md`, `../03-ui-transversal/grillas.md`

## Referencia MULTI

La resolución por empresa activa y el uso de `X-Company-Id` quedan fuera de MONO. Ver `../05-variantes-y-alcance/mono-vs-multi-referencias.md`.

## Derivaciones esperables

Este documento debería permitir regenerar:

- HU del proceso general de parámetros,
- tasks de listado y edición por tipo,
- contratos API de lectura y actualización,
- reglas de caption/tooltip traducibles,
- relación entre menú y `Programa`.

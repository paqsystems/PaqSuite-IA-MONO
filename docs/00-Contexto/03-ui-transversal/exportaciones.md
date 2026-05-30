# Exportaciones de grillas y pivots

Documento de contexto del criterio comun de exportacion para vistas tabulares y analiticas del framework.

## Objetivo

Definir en un solo lugar las reglas conceptuales de exportacion para que, al regenerar HUs, TR y OpenSpec, no queden repartidas entre historias de grillas, pivots o procesos particulares.

## Alcance

Aplica a:

- grillas operativas y de consulta;
- consultas con alternancia entre grilla y pivot;
- vistas pivot propiamente dichas.

No cubre exportaciones batch, programadas ni otros formatos como PDF.

## Regla general

Toda vista exportable debe ofrecer un boton o accion visible en la zona superior inmediata del control principal.

La exportacion debe tomar como base la vista vigente:

- columnas o estructura activa,
- filtros aplicados,
- ordenamiento,
- agrupaciones,
- permisos efectivos del usuario.

## Modalidades conceptuales

### Grillas

| Modalidad | Significado |
|-----------|-------------|
| Basica | Exporta los datos en forma simple, "asi como estan" |
| Formateada | Exporta los datos aplicando formatos de Excel segun el tipo de dato |

Salvo que una especificacion de producto indique otra cosa, la modalidad sugerida por defecto para grillas es **Formateada**, porque suele responder mejor al uso operativo de negocio y lectura humana.

### Pivots

| Modalidad | Significado |
|-----------|-------------|
| Basica | Exporta el resultado visible como matriz de datos |
| Tabla dinamica | Exporta la estructura pivot para continuar el analisis en Excel |

## Regla de formato

Cuando la modalidad sea formateada o estructurada, la exportacion debe reflejar la semantica del dato:

- fechas como fecha legible;
- enteros sin decimales;
- decimales con su precision funcional;
- importes, cantidades y porcentajes con el formato que les corresponda;
- encabezados y totales con presentacion clara para lectura de negocio.

## Regla de disponibilidad

- Sin datos exportables, la accion debe quedar deshabilitada o informar que no hay datos para exportar.
- La modalidad solo debe ofrecer opciones coherentes con la vista actual.
- En vistas pivot, la opcion **Tabla dinamica** solo aparece cuando el control activo realmente es pivot.
- Si la vista permite seleccion multiple, puede ofrecerse una variante de exportacion de **solo filas seleccionadas**, siempre que no contradiga la semantica de la vista activa.
- La exportacion visible para el usuario debe respetar exactamente los filtros, ordenamientos, agrupaciones y permisos que gobiernan la vista actual.

## Volumen y estrategia

- La exportacion debe privilegiar una experiencia util y segura antes que intentar descargar volumen ilimitado sin control.
- Cuando la vista actual represente un volumen muy grande, la especificacion del producto o del proceso puede fijar un limite razonable o exigir una estrategia alternativa para evitar timeouts, archivos excesivos o bloqueos del navegador.
- Si existe una opcion entre exportar la pagina actual o el conjunto completo, esa decision debe ser consistente con la obtencion de datos del proceso y con las restricciones de performance del framework.

## Relacion con layouts

La exportacion convive funcionalmente con la gestion de layouts:

- ambos controles deben vivir cerca del control principal;
- el usuario exporta lo que ve en ese momento;
- cambiar layout o pivot guardado cambia tambien la base de exportacion.

## Relacion con otros temas

- Grillas: `grillas.md`
- Pivots: `pivots.md`
- Apariencia e idioma: `../01-experiencia-base/apariencia-temas.md`, `../01-experiencia-base/idioma-multilingual.md`

## Derivaciones esperables

Este documento sirve para regenerar:

- HUs de exportacion;
- tareas tecnicas de utilidades XLSX;
- specs de toolbar y modalidades;
- criterios de pruebas funcionales y E2E.

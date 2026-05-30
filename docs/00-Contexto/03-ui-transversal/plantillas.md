# Plantillas de pantalla

Documento de contexto de las tipologías de pantalla reutilizables del framework.

## Objetivo

Definir moldes funcionales repetibles para que los módulos del sistema compartan una experiencia consistente y no rediseñen desde cero cada pantalla.

## Tipos de plantilla

### Listado con acciones

Pantalla centrada en una grilla con toolbar y acciones por fila. Es la base natural para consultas, catálogos y ABM.

### Listado con editor modal

Variante del listado donde alta y edición ocurren en modal o panel sobre la misma ruta, sin abandonar el contexto de la grilla.

### Formulario de carga o edición

Pantalla orientada a capturar o modificar datos cuando la complejidad supera lo razonable para edición inline o modal simple.

### Consulta con alternancia grilla/pivot

Pantalla de análisis que permite pasar de una vista tabular a una vista pivot sin perder el contexto del proceso.

### Configuración por filas

Plantilla propia de parámetros generales: listado solo lectura más edición focalizada por fila y por tipo de dato.

### Inicio o dashboard

Pantalla inicial del producto dentro del shell, con indicadores, resúmenes o accesos prioritarios.

## Reglas funcionales

- Cada proceso debería identificar tempranamente qué plantilla lo representa mejor.
- La elección de plantilla debe priorizar consistencia y simplicidad antes que diseños ad hoc.
- La misma plantilla puede tener variantes por complejidad, pero sin romper el patrón principal.
- Los captions, botones, ayudas y mensajes deben seguir las reglas transversales de idioma, apariencia y permisos.

## Relación con otros temas

- Shell y estructura del sitio: `../01-experiencia-base/shell-layout.md`, `../01-experiencia-base/estructura-sitio.md`
- Grillas: `grillas.md`
- Patrones ABM: `patrones-abm.md`
- Parámetros generales: `../04-configuracion-global/parametros-generales.md`

## Derivaciones esperables

Este documento sirve de base para:

- elegir UI inicial de un proceso nuevo,
- homogeneizar screens de distintos módulos,
- derivar specs de layout de pantalla por tipo de proceso.

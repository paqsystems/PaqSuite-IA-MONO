# Configuración funcional por módulo

Documento de contexto para distinguir qué decisiones funcionales de un módulo deben resolverse mediante configuración y cuáles deben permanecer definidas por diseño o seed.

## Objetivo

Evitar que la pantalla de parámetros generales se use como un contenedor ambiguo de cualquier necesidad futura y dejar claro qué tipo de comportamiento sí corresponde parametrizar por módulo.

## Qué se considera configuración funcional

Corresponde parametrizar cuando el módulo necesita variar un comportamiento sin alterar su estructura base, por ejemplo:

- activación o desactivación de una capacidad prevista;
- valores por defecto;
- límites, tolerancias o umbrales;
- URLs, textos auxiliares o referencias externas controladas;
- selecciones de modo de operación ya contempladas por el diseño.

## Qué no corresponde parametrizar

No corresponde usar parámetros generales para:

- crear nuevas entidades o procesos no definidos;
- reemplazar reglas centrales del dominio sin análisis funcional;
- ocultar decisiones de arquitectura;
- introducir estructuras arbitrarias de datos que deberían tener tabla propia;
- convertir la UI de parámetros en un editor libre de metadata.

## Relación con `PQ_PARAMETROS_GRAL`

La tabla `PQ_PARAMETROS_GRAL` materializa esta configuración funcional:

- `Programa` identifica el módulo;
- `Clave` identifica el parámetro;
- `tipo_valor` determina cómo se edita y valida;
- `CAPTION` y `TOOLTIP` describen la experiencia visible.

La existencia de una fila supone que la capacidad configurable ya fue definida conceptualmente para ese módulo.

## Regla de gobernanza funcional

Antes de crear un parámetro nuevo debería quedar claro:

1. qué comportamiento modifica;
2. por qué corresponde parametrizarlo y no fijarlo por diseño;
3. qué tipo funcional de dato necesita;
4. cómo se mostrará y validará en la UI;
5. qué impacto tiene en el módulo.

## Relación con otros temas

- Proceso general de parámetros: `parametros-generales.md`
- Menú y procedimiento: `../01-experiencia-base/menu-general.md`
- Idioma: `../01-experiencia-base/idioma-multilingual.md`

## Derivaciones esperables

Este documento sirve para regenerar:

- criterios para definir nuevos parámetros por módulo,
- reglas funcionales de seeds de parámetros,
- decisiones sobre qué va a `PQ_PARAMETROS_GRAL` y qué requiere otro diseño.

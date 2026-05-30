# Patrones ABM

Documento de contexto de los comportamientos comunes para pantallas de alta, baja y modificación.

## Objetivo

Uniformar cómo se presentan y resuelven los procesos ABM dentro del framework, de manera que usuarios y desarrolladores reconozcan una lógica repetible.

## Patrón base

El patrón preferido es:

1. listado inicial en grilla;
2. acciones por fila visibles;
3. alta y edición en modal o panel superpuesto;
4. confirmación explícita para eliminar.

## Reglas funcionales

### Listado

- La grilla es la entrada natural al proceso.
- La toolbar debe agrupar acciones globales como agregar, exportar, filtros o layouts.
- El usuario debe conservar contexto de búsqueda y orden al volver desde una edición.

### Alta

- Se inicia desde botón **Agregar**.
- Usa formulario alineado al tipo de dato y a la complejidad del recurso.
- Las validaciones ocurren antes de guardar y también en backend.

### Edición

- Se dispara desde icono de acción por fila.
- Debe mostrar valores vigentes del registro.
- Al guardar con éxito, el listado refleja el cambio sin perder coherencia de estado.

### Baja

- Se ejecuta desde acción por fila.
- Requiere confirmación explícita.
- Debe respetar permisos y restricciones de negocio.

### Detalle

- Cuando el proceso necesita solo consulta, puede existir acción de detalle reutilizando el mismo patrón de modal o panel, pero sin habilitar edición.

## Relación con permisos

- La visibilidad de agregar, editar o eliminar debe responder a permisos del menú y atributos del rol.
- La ausencia de permiso no debe resolverse solo con ocultamiento visual; también debe existir validación en backend.

## Relación con otros temas

- Grillas: `grillas.md`
- Administración de seguridad: `../02-acceso-y-seguridad/administracion-seguridad.md`
- Plantillas: `plantillas.md`

## Derivaciones esperables

Este documento sirve de base para:

- HU de catálogos y mantenimientos,
- specs de UI modal sobre listado,
- criterios de regresión funcional para ABM.

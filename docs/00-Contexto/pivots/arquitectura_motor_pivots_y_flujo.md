# Arquitectura del Motor de Pivots y Flujo de Ejecución
**Proyecto:** Paqsuite  
**Base de datos:** Dictionary DB  
**Prefijo de tablas:** pq_pivots_  
**Documento:** Arquitectura funcional y técnica del motor de pivots  
**Versión:** 1.0  
**Fecha:** 2026-04-07

---

# 1. Objetivo

Definir la arquitectura general del motor de pivots del sistema y describir el flujo de ejecución completo desde que un usuario accede a una consulta pivotable hasta que:

- visualiza un pivot base
- modifica su estructura
- valida la configuración
- ejecuta el análisis
- exporta resultados
- guarda o elimina pivots personalizados
- realiza drill-down al detalle

Este documento busca servir como referencia para:

- programadores backend
- programadores frontend
- Cursor
- analistas funcionales
- soporte técnico

---

# 2. Alcance

El documento cubre:

1. componentes del motor
2. tablas y metadatos involucrados
3. responsabilidades por capa
4. flujo de construcción de un pivot
5. flujo de validación
6. flujo de persistencia
7. flujo de exportación
8. flujo de drill-down
9. manejo de compatibilidad y versiones
10. recomendaciones de implementación

No cubre en detalle el diseño visual de pantalla ni el SQL específico de cada consulta de negocio.

---

# 3. Visión general del motor

El motor de pivots se apoya en dos conceptos centrales:

## 3.1 Definición estructural por metadata
La consulta, sus campos, sus plantillas, sus restricciones y sus pivots guardados no deben depender exclusivamente del código, sino de metadata persistida en Dictionary DB.

## 3.2 Ejecución dinámica controlada
El usuario puede reorganizar el pivot dinámicamente, pero siempre dentro de reglas validadas por el motor.

---

# 4. Componentes del motor

## 4.1 Catálogo de consultas pivotables
Tabla:

```txt
pq_pivots_consultas
```

Responsabilidad:
- registrar qué consultas admiten pivot
- definir su fuente técnica
- indicar si admiten drill-down
- gestionar versión de definición

---

## 4.2 Catálogo de campos
Tabla:

```txt
pq_pivots_campos
```

Responsabilidad:
- definir campos disponibles por consulta
- identificar dimensiones, métricas y atributos
- registrar nombres visibles
- definir roles permitidos
- declarar agregaciones y formatos

---

## 4.3 Catálogo de plantillas globales
Tablas:

```txt
pq_pivots_plantillas
pq_pivots_plantillas_det
```

Responsabilidad:
- centralizar comportamientos estándar
- evitar redefinir reglas por consulta
- permitir máxima generalización

---

## 4.4 Catálogo de validaciones
Tabla:

```txt
pq_pivots_validaciones
```

Responsabilidad:
- controlar restricciones
- limitar combinaciones inválidas
- proteger performance
- exigir filtros obligatorios
- impedir configuraciones inconsistentes

---

## 4.5 Pivots guardados
Tabla:

```txt
pq_pivots_config
```

Responsabilidad:
- persistir configuraciones de usuario
- permitir Guardar / Guardar como / Eliminar
- mantener autoría
- registrar versión de definición usada

---

## 4.6 Auditoría opcional
Tabla:

```txt
pq_pivots_aud
```

Responsabilidad:
- registrar creación, modificación, eliminación y clonación
- mejorar trazabilidad y soporte técnico

---

# 5. Capas lógicas recomendadas

## 5.1 Capa de metadata
Responsable de leer y resolver:

- consulta pivotable
- campos
- plantillas
- validaciones
- pivots guardados

## 5.2 Capa de resolución de definición
Responsable de combinar:

- definición del catálogo de campos
- herencia desde plantillas globales
- overrides locales
- restricciones aplicables

Resultado:
- definición efectiva de la consulta en tiempo de ejecución

## 5.3 Capa de validación
Responsable de evaluar:

- reglas generales
- reglas por consulta
- roles permitidos
- límites de dimensiones y métricas
- compatibilidad de agregaciones
- filtros obligatorios
- combinaciones no permitidas

## 5.4 Capa de ejecución de consulta
Responsable de:

- recibir filtros generales
- obtener dataset base
- respetar límites de volumen
- devolver datos estructurados para pivot

## 5.5 Capa de transformación pivot
Responsable de:

- aplicar filas
- aplicar columnas
- aplicar valores
- agrupar
- agregar
- ordenar
- subtotalizar
- totalizar

## 5.6 Capa de presentación
Responsable de:

- mostrar selector de campos
- mostrar pivot resultante
- permitir cambios de estructura
- ofrecer exportaciones
- exponer acciones de guardado

## 5.7 Capa de persistencia
Responsable de:

- guardar configuración
- clonar pivots
- eliminar lógicamente
- validar permisos sobre pivots existentes

---

# 6. Flujo general de ejecución

## Paso 1 – Apertura de una consulta
El usuario ingresa a una consulta que potencialmente admite pivot.

El sistema debe:

1. identificar `consulta_id`
2. leer `pq_pivots_consultas`
3. validar que:
   - la consulta esté activa
   - `pivot_habilitado = 1`

Si no admite pivot:
- se muestra solo la grilla

Si admite pivot:
- se habilita alternancia grilla / pivot

---

## Paso 2 – Resolución de metadata
El motor debe cargar:

1. consulta desde `pq_pivots_consultas`
2. campos desde `pq_pivots_campos`
3. plantillas referenciadas desde `pq_pivots_plantillas`
4. detalle de plantillas desde `pq_pivots_plantillas_det`
5. validaciones desde `pq_pivots_validaciones`

Resultado:
- definición efectiva de la consulta pivotable

---

## Paso 3 – Construcción de la definición efectiva
Para cada campo:

1. leer definición local en `pq_pivots_campos`
2. si tiene `plantilla_global`, cargar propiedades de la plantilla
3. aplicar herencia
4. sobrescribir con valores locales cuando estén informados

Resultado:
- campo final listo para UI y validación

> Regla: la plantilla aporta defaults; el campo local solo sobrescribe por necesidad concreta.

---

## Paso 4 – Determinación del pivot inicial
El sistema debe decidir qué configuración usar al abrir el pivot.

Orden recomendado:

### Opción A
Pivot del sistema marcado como:
- `es_pivot_sistema = 1`
- `es_default = 1`

### Opción B
Pivot base definido por metadata técnica de la consulta

### Opción C
Último pivot usado por el usuario, si más adelante se implementara esa lógica

Si no existe pivot guardado:
- usar pivot base estándar

---

## Paso 5 – Carga de filtros generales
Antes de ejecutar el pivot, el sistema debe cargar los filtros generales de la consulta.

Debe validar:

- filtros visibles
- filtros obligatorios
- tipo de control
- valores compatibles

En esta instancia también deben aplicarse validaciones del tipo:

- `filtro_obligatorio`
- `limite_tiempo`
- `campo_requiere`

---

## Paso 6 – Obtención del dataset base
Con los filtros validados, el backend ejecuta la consulta analítica base:

- view
- table
- procedure
- api

El dataset debe ser:

- coherente con los campos definidos en metadata
- apto para agregación
- limitado por restricciones de volumen

Aquí deben aplicarse controles como:

- rango máximo de fechas
- máximo de registros base
- necesidad de filtros previos

---

## Paso 7 – Construcción del pivot
Con el dataset base y la configuración seleccionada, el motor debe:

1. validar filas
2. validar columnas
3. validar métricas
4. validar agregaciones
5. agrupar dataset
6. calcular agregaciones
7. aplicar subtotales
8. aplicar totales generales
9. ordenar resultado según reglas

---

## Paso 8 – Presentación del resultado
El frontend debe renderizar:

- estructura de filas
- estructura de columnas
- celdas agregadas
- subtotales
- totales
- encabezados visibles
- formato numérico y textual

Además, debe mostrar:

- selector de campos
- configuración actual
- acciones de exportación
- acciones de guardado

---

# 7. Flujo de validación

Las validaciones deben ejecutarse en varias etapas.

## 7.1 Validación de metadata
Se ejecuta al cargar la consulta.

Controla:
- existencia de consulta
- campos activos
- integridad de plantillas
- coherencia de roles
- agregaciones válidas

## 7.2 Validación de filtros
Se ejecuta antes de obtener dataset.

Controla:
- obligatoriedad
- formatos
- compatibilidad entre filtros

## 7.3 Validación de estructura pivot
Se ejecuta antes de construir el resultado.

Controla:
- máximo de dimensiones
- máximo de columnas
- máximo de métricas
- campos incompatibles
- campos requeridos
- métricas no válidas
- agregaciones incompatibles

## 7.4 Validación de volumen
Se ejecuta antes y/o después de consultar datos.

Controla:
- registros base
- tamaño del pivot
- cantidad potencial de celdas
- necesidad de advertencias o bloqueo

---

# 8. Flujo de pivots guardados

## 8.1 Listado
Al abrir una consulta, el sistema debe poder listar pivots disponibles de:

```txt
pq_pivots_config
```

Filtrando por:
- `consulta_id`
- `activo = 1`
- `eliminado = 0`

Y mostrando:
- nombre
- creador
- fecha de creación
- fecha de última modificación
- si es pivot del sistema
- si es default

---

## 8.2 Guardar
Cuando el usuario modifica un pivot existente y elige Guardar:

Validaciones:
- el pivot existe
- no está eliminado
- el usuario actual es el creador
- la configuración actual es válida

Acción:
- actualizar `configuracion_json`
- actualizar usuario y fecha de última modificación

---

## 8.3 Guardar como
Cuando el usuario crea una variante nueva:

Validaciones:
- nombre informado
- configuración válida
- consulta compatible

Acción:
- insertar nuevo registro
- registrar `pivot_origen_id` si partió de otro
- registrar creador y fechas

---

## 8.4 Eliminar
Cuando el usuario elimina un pivot propio:

Validaciones:
- el pivot existe
- es del usuario
- no es un pivot del sistema

Acción:
- borrado lógico:
  - `eliminado = 1`
  - `activo = 0`

---

# 9. Flujo de exportación

El motor debe soportar dos modos:

## 9.1 Excel básico
Objetivo:
- exportar datos resultantes sin reconstrucción visual avanzada

Debe incluir:
- encabezados
- datos agregados
- subtotales si aplican
- totales si aplican
- filtros aplicados
- metadatos de consulta y pivot

## 9.2 Excel formateado
Objetivo:
- reproducir la estructura visual del pivot

Debe incluir:
- jerarquías
- layout de filas y columnas
- subtotales
- totales
- formato numérico
- presentación coherente con el pivot visual

> Regla: el Excel formateado replica la estructura del pivot, no necesariamente su interactividad.

---

# 10. Flujo de drill-down

Si la consulta tiene:

```txt
admite_drilldown = 1
```

y la celda seleccionada es drillable, el sistema debe:

1. identificar la intersección exacta:
   - filas
   - columnas
   - filtros aplicados
   - métrica seleccionada
2. reconstruir el criterio de detalle
3. ejecutar una consulta de detalle
4. mostrar grilla o visor de detalle

El drill-down debe respetar:
- filtros generales
- filtros internos del pivot
- intersección específica de la celda
- permisos del usuario

---

# 11. Compatibilidad y versionado

Cada pivot guardado debe registrar:

```txt
version_definicion_consulta
```

Al reutilizar un pivot guardado, el sistema debe:

1. comparar versión guardada vs versión actual de la consulta
2. verificar si los campos siguen existiendo
3. verificar si las agregaciones siguen siendo válidas
4. verificar si las plantillas mantienen compatibilidad

Resultado posible:

## Caso A – Compatible
Se usa normalmente.

## Caso B – Compatible con ajuste menor
Se corrige automáticamente y se informa si corresponde.

## Caso C – Incompatible
Se rechaza y se informa que el pivot debe recrearse o guardarse como nuevo.

---

# 12. Recomendación de responsabilidades por backend y frontend

## Backend
Debe resolver:
- metadata
- herencia de plantillas
- validaciones
- ejecución de consulta
- transformación pivot
- exportación
- persistencia

## Frontend
Debe resolver:
- experiencia de armado
- selector de campos
- acciones del usuario
- visualización
- mensajes de validación
- interacción de exportación
- interacción de guardado

> Regla recomendada: la lógica crítica debe vivir en backend, no en frontend.

---

# 13. Flujo resumido extremo a extremo

## Escenario completo
1. Usuario abre consulta
2. Sistema carga metadata de consulta
3. Sistema carga campos
4. Sistema resuelve plantillas
5. Sistema carga validaciones
6. Usuario elige pivot base o guardado
7. Usuario define filtros
8. Sistema valida filtros
9. Sistema ejecuta consulta base
10. Sistema valida volumen y compatibilidad
11. Sistema construye pivot
12. Frontend renderiza resultado
13. Usuario modifica filas / columnas / valores
14. Sistema revalida
15. Sistema recalcula pivot
16. Usuario exporta o guarda
17. Sistema persiste o genera archivo

---

# 14. Recomendaciones de implementación

## 14.1 Resolver metadata en un servicio específico
Se recomienda un servicio dedicado, por ejemplo:

```txt
PivotMetadataService
```

## 14.2 Separar transformación pivot de consulta base
Se recomienda distinguir claramente:
- servicio de obtención de dataset
- servicio de armado pivot

## 14.3 Centralizar validaciones
Se recomienda un servicio dedicado, por ejemplo:

```txt
PivotValidationService
```

## 14.4 Centralizar persistencia
Se recomienda un servicio dedicado, por ejemplo:

```txt
PivotConfigService
```

## 14.5 Evitar lógica hardcodeada
Toda regla posible debe salir de metadata.

## 14.6 Mantener trazabilidad
Toda operación relevante debería poder auditarse.

---

# 15. Riesgos a evitar

1. construir pivots sin metadata consolidada
2. permitir campos técnicos en UI
3. no validar antes de ejecutar
4. cargar demasiados datos sin filtros
5. mezclar reglas de negocio con renderizado visual
6. romper compatibilidad de pivots guardados sin control de versión
7. depender de demasiada lógica en frontend

---

# 16. Mapa lógico final del motor

```txt
pq_pivots_consultas
    ↓
pq_pivots_campos
    ↓
pq_pivots_plantillas
    ↓
pq_pivots_plantillas_det
    ↓
pq_pivots_validaciones
    ↓
pq_pivots_config
    ↓
pq_pivots_aud (opcional)
```

Interpretación:

- `consultas` define qué existe
- `campos` define con qué se construye
- `plantillas` define estándares reutilizables
- `validaciones` define límites y restricciones
- `config` define personalizaciones guardadas
- `aud` define trazabilidad

---

# 17. Resumen normativo

1. El motor de pivots debe basarse en metadata persistida en Dictionary DB.
2. La definición efectiva de una consulta surge de combinar consulta, campos, plantillas y validaciones.
3. Toda configuración debe validarse antes de ejecutarse.
4. La lógica crítica del motor debe residir en backend.
5. Los pivots guardados deben ser versionables y compatibles con la definición vigente.
6. La exportación, persistencia y drill-down deben integrarse al mismo flujo general del motor.
7. Debe evitarse toda lógica innecesariamente hardcodeada cuando pueda definirse por metadata.

---

# Fin del documento

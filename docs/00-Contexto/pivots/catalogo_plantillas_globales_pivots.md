# Catálogo de Plantillas Globales de Pivots
**Proyecto:** Paqsuite  
**Base de datos:** Dictionary DB  
**Prefijo de tablas:** pq_pivots_  
**Documento:** Diseño del catálogo de plantillas globales  
**Versión:** 1.0  
**Fecha:** 2026-03-16

---

# 1. Objetivo

Definir un catálogo centralizado de **plantillas globales** para el motor de pivots, con el fin de reutilizar reglas estándar y evitar redefinir en cada consulta comportamientos ya conocidos.

Este catálogo permite formalizar y reutilizar:

- plantillas temporales
- plantillas de métricas monetarias
- plantillas de cantidades
- plantillas de porcentajes
- plantillas de precios unitarios
- reglas de formato
- órdenes naturales
- roles permitidos
- agregaciones por defecto
- agregaciones permitidas

---

# 2. Principio rector

> Todo comportamiento que pueda estandarizarse a nivel global debe definirse una sola vez y reutilizarse desde las consultas y los campos.

Esto evita:

- inconsistencias entre consultas
- duplicación de criterios
- complejidad innecesaria
- errores de mantenimiento
- diferencias arbitrarias en la experiencia de usuario

---

# 3. Rol de las plantillas globales

Las plantillas globales permiten que un campo registrado en `pq_pivots_campos` herede propiedades estándar sin redefinirlas localmente.

Ejemplos:

- un campo `Importe Neto` puede usar la plantilla `metrica_monetaria`
- un campo `Cantidad` puede usar la plantilla `metrica_cantidad`
- un campo `Fecha` o sus derivados pueden usar `temporal_estandar`
- un campo `Precio Unitario` puede usar `precio_unitario`
- un campo `Estado` puede usar `estado_estandar`

---

# 4. Estructura general propuesta

Se propone un modelo de dos tablas:

1. `pq_pivots_plantillas`
2. `pq_pivots_plantillas_det`

---

# 5. Tabla cabecera de plantillas

## Nombre

```txt
pq_pivots_plantillas
```

## Objetivo

Registrar la identidad funcional de cada plantilla global.

---

## Estructura propuesta

| Campo | Tipo sugerido | Nulo | Descripción |
|---|---|---:|---|
| plantilla_id | bigint identity | No | Identificador interno |
| codigo | varchar(100) | No | Código único de plantilla |
| nombre | varchar(200) | No | Nombre visible |
| descripcion | varchar(500) | Sí | Descripción funcional |
| tipo_plantilla | varchar(50) | No | temporal / metrica / dimension / orden / formato / estado |
| activo | bit | No | Activa o no |
| usuario_creacion | varchar(100) | No | Usuario creador |
| fecha_creacion | datetime2 | No | Fecha de creación |
| usuario_ult_mod | varchar(100) | No | Última modificación |
| fecha_ult_mod | datetime2 | No | Fecha última modificación |

---

## Clave primaria

```txt
plantilla_id
```

## Restricción lógica

```txt
codigo
```

debe ser único.

---

# 6. Tabla detalle de propiedades de plantilla

## Nombre

```txt
pq_pivots_plantillas_det
```

## Objetivo

Guardar las propiedades concretas que una plantilla aporta al campo o al comportamiento del motor de pivots.

---

## Estructura propuesta

| Campo | Tipo sugerido | Nulo | Descripción |
|---|---|---:|---|
| plantilla_det_id | bigint identity | No | Identificador interno |
| plantilla_id | bigint | No | Plantilla cabecera |
| propiedad | varchar(100) | No | Nombre de propiedad |
| valor | varchar(500) | Sí | Valor de propiedad |
| orden | int | No | Orden de aplicación o lectura |

---

## Ejemplos de propiedad

- formato
- orden_natural
- roles_permitidos
- agregacion_default
- agregaciones_permitidas
- categoria
- decimales
- permite_totales
- visible_por_defecto
- tipo_dato
- comportamiento_temporal
- derivados_temporales

---

# 7. Relaciones

```txt
pq_pivots_plantillas_det.plantilla_id
→ pq_pivots_plantillas.plantilla_id
```

Y desde el catálogo de campos:

```txt
pq_pivots_campos.plantilla_global
→ pq_pivots_plantillas.codigo
```

> En una primera versión, `pq_pivots_campos.plantilla_global` puede vincular por código de plantilla.  
> Más adelante, si preferís máxima integridad, podría pasar a `plantilla_id`.

---

# 8. SQL conceptual propuesto

## Tabla cabecera

```sql
CREATE TABLE pq_pivots_plantillas
(
    plantilla_id bigint IDENTITY(1,1) NOT NULL,
    codigo varchar(100) NOT NULL,
    nombre varchar(200) NOT NULL,
    descripcion varchar(500) NULL,
    tipo_plantilla varchar(50) NOT NULL,
    activo bit NOT NULL CONSTRAINT DF_pq_pivots_plantillas_activo DEFAULT (1),
    usuario_creacion varchar(100) NOT NULL,
    fecha_creacion datetime2 NOT NULL,
    usuario_ult_mod varchar(100) NOT NULL,
    fecha_ult_mod datetime2 NOT NULL,
    CONSTRAINT PK_pq_pivots_plantillas PRIMARY KEY (plantilla_id),
    CONSTRAINT UQ_pq_pivots_plantillas_codigo UNIQUE (codigo)
);
GO

CREATE INDEX IX_pq_pivots_plantillas_tipo_activo
    ON pq_pivots_plantillas (tipo_plantilla, activo);
GO
```

---

## Tabla detalle

```sql
CREATE TABLE pq_pivots_plantillas_det
(
    plantilla_det_id bigint IDENTITY(1,1) NOT NULL,
    plantilla_id bigint NOT NULL,
    propiedad varchar(100) NOT NULL,
    valor varchar(500) NULL,
    orden int NOT NULL CONSTRAINT DF_pq_pivots_plantillas_det_orden DEFAULT (0),
    CONSTRAINT PK_pq_pivots_plantillas_det PRIMARY KEY (plantilla_det_id),
    CONSTRAINT FK_pq_pivots_plantillas_det_plantilla
        FOREIGN KEY (plantilla_id)
        REFERENCES pq_pivots_plantillas (plantilla_id)
);
GO

CREATE INDEX IX_pq_pivots_plantillas_det_plantilla
    ON pq_pivots_plantillas_det (plantilla_id, orden);
GO

CREATE INDEX IX_pq_pivots_plantillas_det_propiedad
    ON pq_pivots_plantillas_det (propiedad);
GO
```

---

# 9. Plantillas globales sugeridas

## 9.1 `temporal_estandar`

### Objetivo
Definir el comportamiento general de campos temporales y sus derivados.

### Propiedades sugeridas
- categoria = Tiempo
- roles_permitidos = fila,columna,filtro
- formato = fecha
- comportamiento_temporal = estandar
- derivados_temporales = Fecha,Anio,Semestre,Trimestre,Mes,AnioMes,Semana,Dia,DiaSemana

---

## 9.2 `metrica_monetaria`

### Objetivo
Definir reglas estándar para importes monetarios.

### Propiedades sugeridas
- categoria = Importes
- tipo_dato = decimal
- formato = moneda
- roles_permitidos = valor
- agregacion_default = SUMA
- agregaciones_permitidas = SUMA,PROMEDIO,MAX,MIN
- decimales = 2
- permite_totales = 1

---

## 9.3 `metrica_cantidad`

### Objetivo
Definir reglas estándar para cantidades.

### Propiedades sugeridas
- categoria = Cantidades
- tipo_dato = decimal
- formato = numero
- roles_permitidos = valor
- agregacion_default = SUMA
- agregaciones_permitidas = SUMA,PROMEDIO,MAX,MIN
- decimales = 2
- permite_totales = 1

---

## 9.4 `metrica_porcentaje`

### Objetivo
Definir reglas estándar para porcentajes.

### Propiedades sugeridas
- categoria = Indicadores
- tipo_dato = decimal
- formato = porcentaje
- roles_permitidos = valor
- agregacion_default = PROMEDIO
- agregaciones_permitidas = PROMEDIO,MAX,MIN
- decimales = 2
- permite_totales = 0

---

## 9.5 `precio_unitario`

### Objetivo
Definir reglas estándar para precios unitarios.

### Propiedades sugeridas
- categoria = Importes
- tipo_dato = decimal
- formato = moneda
- roles_permitidos = valor
- agregacion_default = PROMEDIO
- agregaciones_permitidas = PROMEDIO,MAX,MIN
- decimales = 2
- permite_totales = 0

---

## 9.6 `conteo_estandar`

### Objetivo
Definir comportamiento para métricas de conteo.

### Propiedades sugeridas
- categoria = Indicadores
- tipo_dato = int
- formato = numero
- roles_permitidos = valor
- agregacion_default = CONTEO
- agregaciones_permitidas = CONTEO,CONTEO_DISTINTO
- decimales = 0
- permite_totales = 1

---

## 9.7 `estado_estandar`

### Objetivo
Definir un tratamiento uniforme para campos de estado.

### Propiedades sugeridas
- categoria = Estado
- tipo_dato = string
- formato = texto
- roles_permitidos = fila,columna,filtro
- orden_natural = asc
- visible_por_defecto = 1

---

## 9.8 `dimension_texto_estandar`

### Objetivo
Definir reglas generales para dimensiones textuales.

### Propiedades sugeridas
- tipo_dato = string
- formato = texto
- roles_permitidos = fila,columna,filtro
- visible_por_defecto = 1
- orden_natural = asc

---

# 10. Ejemplos de registros en cabecera

| codigo | nombre | tipo_plantilla | descripcion |
|---|---|---|---|
| temporal_estandar | Temporal estándar | temporal | Reglas comunes para fechas y derivados |
| metrica_monetaria | Métrica monetaria | metrica | Reglas comunes para importes |
| metrica_cantidad | Métrica cantidad | metrica | Reglas comunes para cantidades |
| metrica_porcentaje | Métrica porcentaje | metrica | Reglas comunes para porcentajes |
| precio_unitario | Precio unitario | metrica | Reglas comunes para precios unitarios |
| conteo_estandar | Conteo estándar | metrica | Reglas comunes para conteos |
| estado_estandar | Estado estándar | dimension | Reglas comunes para estados |

---

# 11. Ejemplo de detalle para `metrica_monetaria`

## Cabecera

```txt
codigo = metrica_monetaria
```

## Detalle

| propiedad | valor | orden |
|---|---|---:|
| categoria | Importes | 10 |
| tipo_dato | decimal | 20 |
| formato | moneda | 30 |
| roles_permitidos | valor | 40 |
| agregacion_default | SUMA | 50 |
| agregaciones_permitidas | SUMA,PROMEDIO,MAX,MIN | 60 |
| decimales | 2 | 70 |
| permite_totales | 1 | 80 |

---

# 12. Ejemplo de detalle para `temporal_estandar`

| propiedad | valor | orden |
|---|---|---:|
| categoria | Tiempo | 10 |
| roles_permitidos | fila,columna,filtro | 20 |
| comportamiento_temporal | estandar | 30 |
| derivados_temporales | Fecha,Anio,Semestre,Trimestre,Mes,AnioMes,Semana,Dia,DiaSemana | 40 |
| formato | fecha | 50 |

---

# 13. Cómo se aplican las plantillas

La lógica recomendada de aplicación es:

## Paso 1
Cargar definición del campo desde `pq_pivots_campos`

## Paso 2
Si el campo tiene `plantilla_global`, cargar la plantilla correspondiente

## Paso 3
Aplicar propiedades de plantilla como valores por defecto

## Paso 4
Si el campo declara un valor explícito en `pq_pivots_campos`, ese valor local sobrescribe al de la plantilla

> Regla: la plantilla define el estándar; el campo solo debería sobrescribir cuando exista una razón funcional concreta.

---

# 14. Beneficios del modelo

Implementar `pq_pivots_plantillas` y `pq_pivots_plantillas_det` aporta:

1. máxima generalización posible
2. menor duplicación
3. mayor consistencia entre consultas
4. mejor mantenibilidad
5. facilidad para agregar nuevos tipos de métricas o dimensiones
6. simplificación del contrato técnico por consulta
7. mejor compatibilidad futura con un motor de pivots más dinámico

---

# 15. Orden lógico del motor de pivots

Con este catálogo, el modelo queda así:

1. `pq_pivots_consultas`  
   Catálogo de consultas pivotables

2. `pq_pivots_campos`  
   Catálogo de campos de cada consulta

3. `pq_pivots_plantillas`  
   Catálogo de plantillas globales

4. `pq_pivots_plantillas_det`  
   Propiedades de cada plantilla

5. `pq_pivots_config`  
   Configuraciones guardadas de pivots

6. `pq_pivots_aud`  
   Auditoría opcional

---

# 16. Próxima mejora recomendada

La siguiente pieza útil, si querés llevar esto a un nivel todavía más robusto, sería definir un documento o catálogo de:

```txt
pq_pivots_validaciones
```

o incluso reglas técnicas por consulta para:

- límites de combinaciones
- campos incompatibles
- filtros obligatorios
- restricciones de volumen
- overrides permitidos

---

# 17. Resumen normativo

1. Las plantillas globales deben residir en Dictionary DB.
2. El prefijo debe ser `pq_pivots_`.
3. Toda plantilla debe tener cabecera y detalle.
4. El código de plantilla debe ser único.
5. Las propiedades de plantilla deben poder heredarse desde `pq_pivots_campos`.
6. El campo puede sobrescribir propiedades de plantilla solo en casos justificados.
7. Las plantillas deben utilizarse para maximizar la generalización del sistema.

---

# Fin del documento

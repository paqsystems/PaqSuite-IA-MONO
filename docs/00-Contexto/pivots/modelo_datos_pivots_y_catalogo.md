# Modelo de Datos -- Sistema de Pivots

**Proyecto:** Paqsuite\
**Base de datos:** Dictionary DB\
**Prefijo de tablas:** pq_pivots\_\
**Versión:** 1.0\
**Fecha:** 2026-03-16

------------------------------------------------------------------------

# 1. Objetivo

Definir el modelo de datos para el almacenamiento de:

1.  **Pivots personalizados de usuarios**
2.  **Catálogo de consultas pivotables del sistema**

Este diseño busca:

-   centralizar configuraciones de pivots
-   permitir reutilización entre usuarios
-   mantener trazabilidad
-   separar definición técnica de consultas del código
-   permitir evolución del motor de pivots

------------------------------------------------------------------------

# 2. Principios de diseño

El modelo sigue estas reglas:

1.  Todas las tablas residen en **Dictionary DB**
2.  Prefijo obligatorio **pq_pivots\_**
3.  Configuración del pivot almacenada en **JSON**
4.  Cabecera relacional para búsquedas y control
5.  Borrado **lógico**
6.  Trazabilidad de creación y modificación
7.  Compatibilidad con versiones de definición de consultas

------------------------------------------------------------------------

# 3. Entidades principales

El modelo se compone de dos entidades centrales.

## 3.1 Catálogo de consultas pivotables

Tabla:

    pq_pivots_consultas

Define qué consultas del sistema admiten pivot.

------------------------------------------------------------------------

## 3.2 Pivots personalizados

Tabla:

    pq_pivots_config

Almacena las configuraciones guardadas por usuarios.

------------------------------------------------------------------------

# 4. Catálogo de consultas pivotables

Tabla:

    pq_pivots_consultas

Esta tabla registra todas las consultas del sistema que soportan pivots.

------------------------------------------------------------------------

## Estructura

  ----------------------------------------------------------------------------------
  Campo                Tipo sugerido              Nulo       Descripción
  -------------------- -------------------------- ---------- -----------------------
  consulta_id          varchar(100)               No         Identificador técnico
                                                             de la consulta

  nombre               varchar(200)               No         Nombre funcional

  descripcion          varchar(500)               Sí         Descripción

  fuente_tipo          varchar(50)                No         view / table / sp / api

  fuente_nombre        varchar(200)               No         Nombre de la fuente

  version_definicion   int                        No         Versión del contrato
                                                             técnico

  pivot_habilitado     bit                        No         Indica si la consulta
                                                             admite pivot

  admite_drilldown     bit                        No         Permite abrir detalle

  activo               bit                        No         Consulta activa

  fecha_creacion       datetime2                  No         Fecha creación

  usuario_creacion     varchar(100)               No         Usuario creador
  ----------------------------------------------------------------------------------

------------------------------------------------------------------------

## Clave primaria

    consulta_id

------------------------------------------------------------------------

# 5. Pivots personalizados

Tabla:

    pq_pivots_config

Almacena cada pivot guardado por usuarios.

------------------------------------------------------------------------

## Estructura

  Campo                         Tipo sugerido     Nulo   Descripción
  ----------------------------- ----------------- ------ ----------------------------
  pivot_id                      bigint identity   No     Identificador del pivot
  consulta_id                   varchar(100)      No     Consulta asociada
  nombre                        varchar(200)      No     Nombre visible
  descripcion                   varchar(500)      Sí     Descripción
  configuracion_json            nvarchar(max)     No     Configuración pivot
  version_definicion_consulta   int               No     Versión de consulta usada
  usuario_creador_id            varchar(100)      No     Usuario creador
  fecha_creacion                datetime2         No     Fecha creación
  usuario_ult_mod_id            varchar(100)      No     Última modificación
  fecha_ult_mod                 datetime2         No     Fecha modificación
  pivot_origen_id               bigint            Sí     Pivot clonado
  eliminado                     bit               No     Borrado lógico
  fecha_eliminacion             datetime2         Sí     Fecha eliminación
  usuario_eliminacion_id        varchar(100)      Sí     Usuario eliminación
  activo                        bit               No     Pivot activo
  es_pivot_sistema              bit               No     Pivot definido por sistema
  es_default                    bit               No     Pivot predeterminado

------------------------------------------------------------------------

# 6. Relaciones

    pq_pivots_config.consulta_id
    → pq_pivots_consultas.consulta_id

    pq_pivots_config.pivot_origen_id
    → pq_pivots_config.pivot_id

------------------------------------------------------------------------

# 7. Índices recomendados

### Índice por consulta

    IX_pq_pivots_config_consulta
    (consulta_id, activo, eliminado)

### Índice por creador

    IX_pq_pivots_config_creador
    (usuario_creador_id)

### Índice por nombre

    IX_pq_pivots_config_nombre
    (consulta_id, nombre)

### Índice por origen

    IX_pq_pivots_config_origen
    (pivot_origen_id)

------------------------------------------------------------------------

# 8. SQL conceptual

## Tabla catálogo

``` sql
CREATE TABLE pq_pivots_consultas
(
 consulta_id varchar(100) NOT NULL,
 nombre varchar(200) NOT NULL,
 descripcion varchar(500) NULL,
 fuente_tipo varchar(50) NOT NULL,
 fuente_nombre varchar(200) NOT NULL,
 version_definicion int NOT NULL,
 pivot_habilitado bit NOT NULL,
 admite_drilldown bit NOT NULL,
 activo bit NOT NULL DEFAULT 1,
 fecha_creacion datetime2 NOT NULL,
 usuario_creacion varchar(100) NOT NULL,
 CONSTRAINT pk_pq_pivots_consultas PRIMARY KEY (consulta_id)
);
```

------------------------------------------------------------------------

## Tabla pivots

``` sql
CREATE TABLE pq_pivots_config
(
 pivot_id bigint IDENTITY(1,1) NOT NULL,
 consulta_id varchar(100) NOT NULL,
 nombre varchar(200) NOT NULL,
 descripcion varchar(500) NULL,
 configuracion_json nvarchar(max) NOT NULL,
 version_definicion_consulta int NOT NULL,
 usuario_creador_id varchar(100) NOT NULL,
 fecha_creacion datetime2 NOT NULL,
 usuario_ult_mod_id varchar(100) NOT NULL,
 fecha_ult_mod datetime2 NOT NULL,
 pivot_origen_id bigint NULL,
 eliminado bit NOT NULL DEFAULT 0,
 fecha_eliminacion datetime2 NULL,
 usuario_eliminacion_id varchar(100) NULL,
 activo bit NOT NULL DEFAULT 1,
 es_pivot_sistema bit NOT NULL DEFAULT 0,
 es_default bit NOT NULL DEFAULT 0,
 CONSTRAINT pk_pq_pivots_config PRIMARY KEY (pivot_id),
 CONSTRAINT fk_pq_pivots_config_consulta
     FOREIGN KEY (consulta_id)
     REFERENCES pq_pivots_consultas(consulta_id),
 CONSTRAINT fk_pq_pivots_config_origen
     FOREIGN KEY (pivot_origen_id)
     REFERENCES pq_pivots_config(pivot_id)
);
```

------------------------------------------------------------------------

# 9. Configuración JSON del pivot

La configuración pivot se guarda en:

    configuracion_json

Ejemplo:

``` json
{
  "filas": ["Cliente"],
  "columnas": ["Mes"],
  "valores": [
    {
      "campo": "ImporteNeto",
      "agregacion": "SUMA"
    }
  ],
  "filtrosInternos": [],
  "ordenamiento": [
    {
      "campo": "Cliente",
      "direccion": "asc"
    }
  ],
  "mostrarSubtotales": true,
  "mostrarTotalesGenerales": true
}
```

------------------------------------------------------------------------

# 10. Operaciones soportadas

## Guardar

Actualiza un pivot existente.

Validación:

-   usuario actual = creador

------------------------------------------------------------------------

## Guardar como

Inserta nuevo registro:

-   copia configuración
-   registra pivot_origen_id

------------------------------------------------------------------------

## Eliminar

Borrado lógico:

    eliminado = 1
    activo = 0

------------------------------------------------------------------------

# 11. Reglas funcionales

1.  Todos los pivots son visibles para usuarios con acceso a la
    consulta.
2.  Solo el creador puede modificar un pivot.
3.  Solo el creador puede eliminarlo.
4.  Cualquier usuario puede clonar un pivot existente.
5.  El sistema puede definir pivots base mediante **es_pivot_sistema**.

------------------------------------------------------------------------

# 12. Posibles extensiones futuras

El modelo permite agregar:

-   pivots favoritos
-   pivots por rol
-   métricas de uso
-   historial de cambios
-   pivots predeterminados por usuario

------------------------------------------------------------------------

# Fin del documento

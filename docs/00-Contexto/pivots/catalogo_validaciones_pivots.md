# Catálogo de Validaciones del Motor de Pivots

**Proyecto:** Paqsuite\
**Base de datos:** Dictionary DB\
**Prefijo de tablas:** pq_pivots\_\
**Documento:** Reglas y validaciones del motor de pivots\
**Versión:** 1.0\
**Fecha:** 2026-03-16

------------------------------------------------------------------------

# 1. Objetivo

Definir un mecanismo formal para registrar **validaciones y
restricciones** del motor de pivots.

Estas validaciones permiten controlar:

-   combinaciones no permitidas de campos
-   límites de volumen de datos
-   filtros obligatorios
-   reglas de compatibilidad
-   reglas de rendimiento
-   excepciones permitidas

La idea es que el **motor de pivots sea configurable desde metadata**,
evitando lógica rígida en el código.

------------------------------------------------------------------------

# 2. Principio de diseño

> Toda regla que limite o condicione la construcción de un pivot debe
> poder definirse mediante metadata.

Esto permite:

-   evolucionar reglas sin recompilar el sistema
-   mantener coherencia entre consultas
-   mejorar soporte técnico
-   facilitar diagnóstico de errores

------------------------------------------------------------------------

# 3. Tipos de validaciones

Se proponen los siguientes tipos principales.

  Tipo                          Descripción
  ----------------------------- -------------------------------------
  filtro_obligatorio            exige que exista al menos un filtro
  campo_incompatible            impide combinación de campos
  campo_requiere                campo requiere otro campo
  limite_filas                  limita cantidad máxima de filas
  limite_columnas               limita cantidad máxima de columnas
  limite_celdas                 limita tamaño del pivot
  limite_tiempo                 exige filtro temporal
  metrica_unica                 solo permite una métrica
  dimension_unica               solo permite una dimensión
  dimension_maxima              límite de dimensiones
  metrica_maxima                límite de métricas
  requiere_agregacion           exige agregación válida
  incompatibilidad_agregacion   agregaciones incompatibles

------------------------------------------------------------------------

# 4. Tabla de validaciones

## Nombre

    pq_pivots_validaciones

------------------------------------------------------------------------

## Estructura

  Campo               Tipo sugerido       Nulo Descripción
  ------------------- ----------------- ------ ---------------------
  validacion_id       bigint identity       No Identificador
  consulta_id         varchar(100)          Sí Consulta afectada
  tipo_validacion     varchar(50)           No Tipo de regla
  campo_codigo        varchar(100)          Sí Campo afectado
  campo_relacionado   varchar(100)          Sí Campo relacionado
  valor_parametro     varchar(200)          Sí Parámetro de regla
  mensaje_error       varchar(500)          Sí Mensaje a mostrar
  severidad           varchar(20)           No error / warning
  activo              bit                   No Regla activa
  usuario_creacion    varchar(100)          No Usuario creador
  fecha_creacion      datetime2             No Fecha creación
  usuario_ult_mod     varchar(100)          No Última modificación
  fecha_ult_mod       datetime2             No Fecha modificación

------------------------------------------------------------------------

# 5. SQL conceptual

``` sql
CREATE TABLE pq_pivots_validaciones
(
 validacion_id bigint IDENTITY(1,1) NOT NULL,
 consulta_id varchar(100) NULL,
 tipo_validacion varchar(50) NOT NULL,
 campo_codigo varchar(100) NULL,
 campo_relacionado varchar(100) NULL,
 valor_parametro varchar(200) NULL,
 mensaje_error varchar(500) NULL,
 severidad varchar(20) NOT NULL,
 activo bit NOT NULL DEFAULT 1,
 usuario_creacion varchar(100) NOT NULL,
 fecha_creacion datetime2 NOT NULL,
 usuario_ult_mod varchar(100) NOT NULL,
 fecha_ult_mod datetime2 NOT NULL,
 CONSTRAINT PK_pq_pivots_validaciones PRIMARY KEY (validacion_id)
);
```

------------------------------------------------------------------------

# 6. Ejemplos de validaciones

## 6.1 Filtro temporal obligatorio

  tipo_validacion   campo_codigo   valor_parametro
  ----------------- -------------- -----------------
  limite_tiempo     Fecha          requerido

Mensaje:

    Debe seleccionar un filtro de fecha para ejecutar esta consulta.

------------------------------------------------------------------------

## 6.2 Dimensiones máximas

  tipo_validacion    valor_parametro
  ------------------ -----------------
  dimension_maxima   3

Mensaje:

    El pivot no puede tener más de 3 dimensiones.

------------------------------------------------------------------------

## 6.3 Métrica única

  tipo_validacion
  -----------------
  metrica_unica

Mensaje:

    Esta consulta solo permite una métrica.

------------------------------------------------------------------------

## 6.4 Campos incompatibles

  campo_codigo   campo_relacionado
  -------------- -------------------
  Cliente        Vendedor

Mensaje:

    Cliente y Vendedor no pueden usarse juntos en esta consulta.

------------------------------------------------------------------------

# 7. Flujo de validación

El motor de pivots debe aplicar validaciones en este orden:

1.  Validaciones globales
2.  Validaciones por consulta
3.  Validaciones por campo
4.  Validaciones por combinación

------------------------------------------------------------------------

# 8. Integración con el motor de pivots

El sistema completo queda estructurado así:

1.  pq_pivots_consultas\
2.  pq_pivots_campos\
3.  pq_pivots_plantillas\
4.  pq_pivots_plantillas_det\
5.  pq_pivots_validaciones\
6.  pq_pivots_config

------------------------------------------------------------------------

# 9. Beneficios

Implementar un catálogo de validaciones permite:

-   proteger el rendimiento del sistema
-   evitar consultas incorrectas
-   mejorar la experiencia de usuario
-   facilitar soporte técnico
-   permitir evolución del motor

------------------------------------------------------------------------

# 10. Resumen normativo

1.  Toda validación debe registrarse en `pq_pivots_validaciones`.
2.  Las reglas deben poder aplicarse a nivel global o por consulta.
3.  Las validaciones deben ejecutarse antes de generar el pivot.
4.  Las reglas deben devolver mensajes claros al usuario.
5.  Las validaciones deben poder activarse o desactivarse.

------------------------------------------------------------------------

# Fin del documento

# Norma Conceptual del Sistema de Pivots

**Proyecto:** Paqsuite\
**Documento:** Norma general conceptual para uso de pivots en informes y
consultas\
**Versión:** 1.0\
**Fecha:** 2026-03-16

------------------------------------------------------------------------

# 1. Objetivo

Definir las normas conceptuales para la utilización de **vistas Pivot**
dentro del sistema, estableciendo criterios homogéneos para:

-   diseño de consultas pivotables
-   comportamiento funcional del pivot
-   consistencia entre módulos
-   interacción con grillas
-   exportación de datos
-   almacenamiento de pivots personalizados

Este documento está orientado a:

-   programadores
-   analistas funcionales
-   soporte técnico
-   responsables de mantenimiento del sistema

No define detalles de implementación técnica.\
Ese objetivo corresponde al **documento técnico para Cursor**.

------------------------------------------------------------------------

# 2. Definición de Pivot

En el sistema se entiende por **Pivot** una vista analítica que permite
reorganizar datos mediante:

-   **filas**
-   **columnas**
-   **valores agregados**
-   **filtros**

El objetivo del pivot es permitir:

-   análisis multidimensional
-   comparación entre dimensiones
-   detección de tendencias
-   identificación de concentraciones
-   resumen de grandes volúmenes de datos

------------------------------------------------------------------------

# 3. Diferencia entre Grilla y Pivot

## Grilla

La grilla representa **datos de detalle**.

Características:

-   muestra registros individuales
-   permite ordenar y filtrar registros
-   puede permitir acciones sobre filas
-   se utiliza para tareas operativas

La grilla responde preguntas como:

> ¿Qué ocurrió exactamente?

------------------------------------------------------------------------

## Pivot

El pivot representa **datos resumidos**.

Características:

-   agrupa registros
-   calcula totales y subtotales
-   cruza dimensiones
-   permite reorganizar la información

El pivot responde preguntas como:

> ¿Cómo se distribuyen los datos?

------------------------------------------------------------------------

# 4. Principios generales del sistema de pivots

## Principio 1

El pivot **no reemplaza a la grilla**.

Ambos modos de visualización deben convivir.

------------------------------------------------------------------------

## Principio 2

Las consultas pivotables deben definirse explícitamente.

No todas las consultas deben permitir pivot.

------------------------------------------------------------------------

## Principio 3

Toda consulta pivotable debe tener una **pivot base**.

Esto evita que el usuario vea un pivot vacío.

------------------------------------------------------------------------

## Principio 4

Los pivots deben usar **nombres funcionales**, nunca nombres técnicos.

Correcto:

-   Cliente
-   Artículo
-   Importe Neto
-   Año-Mes

Incorrecto:

-   cli_id
-   art_cod
-   imp_neto

------------------------------------------------------------------------

# 5. Generalizaciones del sistema

Para asegurar consistencia, ciertas reglas deben aplicarse a **todas las
consultas**.

------------------------------------------------------------------------

## 5.1 Tratamiento de fechas

Todo campo de tipo fecha pivotable debe poder desagregarse en:

-   Año
-   Semestre
-   Trimestre
-   Mes
-   Año-Mes
-   Semana
-   Día
-   Día de semana

Estas dimensiones deben mantener:

-   nomenclatura uniforme
-   orden cronológico natural
-   formato consistente

------------------------------------------------------------------------

## 5.2 Categorías estándar de campos

Los campos pivotables deben clasificarse en categorías.

Categorías recomendadas:

-   Tiempo
-   Organización
-   Cliente / Proveedor
-   Producto
-   Comercial
-   Estado
-   Importes
-   Cantidades
-   Indicadores

------------------------------------------------------------------------

## 5.3 Formatos estándar

Los formatos de presentación deben ser uniformes.

Formatos disponibles:

-   texto
-   número
-   moneda
-   porcentaje
-   fecha
-   fecha-hora
-   booleano

------------------------------------------------------------------------

## 5.4 Agregaciones estándar

Dependiendo del tipo de métrica.

### Importes

-   suma
-   promedio
-   máximo
-   mínimo

### Cantidades

-   suma
-   promedio
-   máximo
-   mínimo

### Conteos

-   conteo
-   conteo distinto

### Porcentajes

-   promedio
-   máximo
-   mínimo

------------------------------------------------------------------------

# 6. Definición de una consulta pivotable

Toda consulta pivotable debe declarar:

-   pivot base
-   dimensiones disponibles
-   métricas disponibles
-   filtros generales
-   restricciones de volumen

------------------------------------------------------------------------

# 7. Pivot base

La pivot base define la estructura inicial.

Debe indicar:

-   filas
-   columnas
-   métricas
-   agregaciones

Ejemplo:

**Ventas por cliente por mes**

Filas:

Cliente

Columnas:

Mes

Valores:

Importe Neto (Suma)

------------------------------------------------------------------------

# 8. Dimensiones

Las dimensiones representan campos que permiten **agrupar información**.

Ejemplos:

-   Cliente
-   Proveedor
-   Artículo
-   Sucursal
-   Vendedor
-   Estado
-   Año
-   Mes

Las dimensiones pueden utilizarse en:

-   filas
-   columnas
-   filtros

------------------------------------------------------------------------

# 9. Métricas

Las métricas representan valores que pueden **agregarse**.

Ejemplos:

-   Importe Neto
-   Importe Total
-   Cantidad
-   Costo
-   Margen
-   Horas

Las métricas deben definir:

-   agregación por defecto
-   agregaciones permitidas

------------------------------------------------------------------------

# 10. Filtros generales

Los filtros generales restringen el conjunto de datos antes de generar
la vista.

Ejemplos:

-   Empresa
-   Fecha
-   Cliente
-   Sucursal
-   Vendedor

Los filtros afectan tanto a:

-   la grilla
-   el pivot

------------------------------------------------------------------------

# 11. Exportación

Las consultas pivotables deben permitir exportación.

Se definen dos modalidades.

## Excel básico

Exporta únicamente los datos resultantes.

Incluye:

-   matriz de datos
-   encabezados
-   totales

------------------------------------------------------------------------

## Excel formateado

Exporta la estructura del pivot.

Incluye:

-   jerarquía de filas
-   subtotales
-   totales
-   formato visual

------------------------------------------------------------------------

# 12. Pivots guardados

Los usuarios pueden guardar configuraciones de pivot.

Operaciones permitidas:

-   Guardar
-   Guardar como
-   Eliminar

Reglas:

-   todos los pivots son visibles para todos los usuarios
-   solo el creador puede modificar un pivot existente
-   solo el creador puede eliminarlo
-   cualquier usuario puede crear uno nuevo basado en otro

------------------------------------------------------------------------

# 13. Drill‑down

Cuando la consulta lo permita, debe existir la posibilidad de abrir el
detalle que compone una celda del pivot.

Esto permite:

-   detectar anomalías
-   investigar datos
-   conectar análisis con operación

------------------------------------------------------------------------

# 14. Restricciones y performance

Las consultas pivotables deben establecer límites razonables para evitar
problemas de rendimiento.

Ejemplos:

-   límite de registros
-   filtros obligatorios
-   advertencias de volumen

------------------------------------------------------------------------

# 15. Buenas prácticas

Se recomienda:

-   utilizar pivots solo cuando el análisis lo justifique
-   definir pivots base útiles
-   mantener consistencia de nombres
-   evitar dimensiones irrelevantes
-   limitar combinaciones excesivas

------------------------------------------------------------------------

# 16. Casos donde no corresponde pivot

No se recomienda pivot cuando:

-   el objetivo es edición de datos
-   el volumen es mínimo
-   no existen métricas agregables
-   la consulta es puramente operativa

------------------------------------------------------------------------

# 17. Documentos complementarios

Este documento se complementa con:

1.  **Especificación técnica de consultas pivotables (para Cursor)**\
2.  **Modelo de datos para almacenamiento de pivots personalizados**

------------------------------------------------------------------------

# Fin del documento

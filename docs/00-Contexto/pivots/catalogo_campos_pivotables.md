# Catálogo de Campos Pivotables
**Proyecto:** Paqsuite  
**Base de datos:** Dictionary DB  
**Prefijo de tablas:** pq_pivots_  
**Documento:** Diseño del catálogo de campos pivotables  
**Versión:** 1.0  
**Fecha:** 2026-03-16

---

# 1. Objetivo

Definir un catálogo de campos pivotables que permita desacoplar del código la metadata funcional y técnica de cada campo disponible en consultas con soporte de pivot.

Este catálogo convierte al sistema en un motor de pivots más dinámico, permitiendo administrar de manera centralizada:

- nombre visible de los campos
- nombre técnico de origen
- tipo de campo
- tipo de dato
- categoría funcional
- roles permitidos
- formato
- orden natural
- agregaciones permitidas
- comportamiento temporal
- visibilidad
- capacidad de drill-down

---

# 2. Objetivos específicos

El catálogo debe permitir:

1. registrar todos los campos disponibles por consulta pivotable
2. evitar nombres técnicos en la interfaz
3. estandarizar métricas, dimensiones y atributos
4. permitir generalizaciones máximas posibles
5. reutilizar reglas estándar entre consultas
6. facilitar validación de pivots guardados
7. facilitar la evolución del sistema sin depender solo del código fuente

---

# 3. Ubicación y convención

- **Base de datos:** Dictionary DB
- **Prefijo:** `pq_pivots_`

Tabla principal propuesta:

```txt
pq_pivots_campos
```

---

# 4. Relación con el catálogo de consultas

Cada campo pivotable pertenece a una consulta registrada en:

```txt
pq_pivots_consultas
```

Relación:

```txt
pq_pivots_campos.consulta_id
→ pq_pivots_consultas.consulta_id
```

---

# 5. Rol del catálogo de campos

El catálogo de campos actúa como diccionario central del motor de pivots.

Debe responder preguntas como:

- ¿Qué campos puede usar esta consulta?
- ¿Cómo se llama cada campo para el usuario?
- ¿Es dimensión o métrica?
- ¿Puede ir en filas, columnas, filtros o valores?
- ¿Qué formato debe mostrar?
- ¿Qué agregaciones permite?
- ¿Es un campo temporal derivado?
- ¿Tiene orden natural?
- ¿Se puede usar para drill-down?
- ¿Está visible o solo reservado para uso interno?

---

# 6. Tabla propuesta

## Nombre

```txt
pq_pivots_campos
```

---

# 7. Estructura propuesta

| Campo | Tipo sugerido | Nulo | Descripción |
|---|---|---:|---|
| campo_id | bigint identity | No | Identificador interno |
| consulta_id | varchar(100) | No | Consulta a la que pertenece |
| campo_codigo | varchar(100) | No | Código lógico estable del campo |
| nombre_tecnico | varchar(200) | No | Nombre técnico en fuente de datos |
| nombre_visible | varchar(200) | No | Nombre visible en UI |
| descripcion | varchar(500) | Sí | Descripción funcional |
| tipo_campo | varchar(30) | No | dimension / metrica / atributo |
| tipo_dato | varchar(30) | No | string / int / decimal / date / datetime / bool |
| categoria | varchar(100) | No | Categoría funcional |
| roles_permitidos | varchar(100) | No | fila,columna,filtro,valor |
| visible | bit | No | Visible en selector UI |
| pivotable | bit | No | Participa en pivot |
| drillable | bit | No | Permite drill-down |
| orden_sugerido | int | No | Orden en selector |
| orden_natural | varchar(50) | Sí | asc / fecha / calendarioMes / periodo / etc. |
| formato | varchar(50) | No | texto / numero / moneda / porcentaje / fecha / fechaHora / booleano |
| ancho_sugerido | int | Sí | Ancho sugerido para UI |
| plantilla_global | varchar(100) | Sí | Plantilla estándar aplicada |
| es_derivado_temporal | bit | No | Indica si deriva de un campo fecha |
| campo_origen_derivado | varchar(100) | Sí | Campo base del derivado temporal |
| agregacion_default | varchar(50) | Sí | Solo para métricas |
| agregaciones_permitidas | varchar(200) | Sí | Solo para métricas |
| decimales | int | Sí | Decimales sugeridos |
| permite_totales | bit | No | Permite totales |
| es_metrica_principal | bit | No | Métrica principal de la consulta |
| visible_por_defecto | bit | No | Visible inicialmente |
| activo | bit | No | Campo activo |
| usuario_creacion | varchar(100) | No | Usuario creador |
| fecha_creacion | datetime2 | No | Fecha creación |
| usuario_ult_mod | varchar(100) | No | Usuario última modificación |
| fecha_ult_mod | datetime2 | No | Fecha última modificación |

---

# 8. Claves y relaciones

## Clave primaria

```txt
campo_id
```

## Clave foránea

```txt
consulta_id → pq_pivots_consultas.consulta_id
```

## Restricción lógica recomendada

Dentro de una misma consulta, el valor de:

```txt
campo_codigo
```

debe ser único.

---

# 9. Campos clave del modelo

## 9.1 `campo_codigo`

Es el identificador lógico estable del campo dentro de la consulta.

Ejemplos:

- Cliente
- Mes
- AnioMes
- ImporteNeto
- Cantidad
- Estado

No debe cambiar arbitrariamente, porque sirve para:

- pivot base
- pivots guardados
- exportación
- validación

---

## 9.2 `nombre_tecnico`

Es el nombre real del campo en la fuente de datos.

Ejemplos:

- CLI_NOMBRE
- MES_NOMBRE
- IMP_NETO
- ESTADO_COMPROBANTE

No debe mostrarse al usuario.

---

## 9.3 `nombre_visible`

Es el nombre mostrado en UI.

Debe ser claro, funcional y no técnico.

Correctos:

- Cliente
- Mes
- Año-Mes
- Importe Neto

Incorrectos:

- cli_nombre
- imp_neto
- fec_doc

---

## 9.4 `tipo_campo`

Valores permitidos:

- `dimension`
- `metrica`
- `atributo`

### Reglas

#### `dimension`
Puede participar en:
- filas
- columnas
- filtros

#### `metrica`
Puede participar en:
- valores

#### `atributo`
Generalmente no participa en pivot.  
Puede usarse en grilla o drill-down.

---

## 9.5 `roles_permitidos`

Se propone almacenar una lista simple delimitada por comas.

Ejemplo:

```txt
fila,columna,filtro
```

o

```txt
valor
```

### Alternativa futura
Si más adelante se quisiera máxima normalización, podría existir una tabla detalle:

```txt
pq_pivots_campos_roles
```

En una primera etapa no parece necesario.

---

# 10. Generalizaciones máximas

## 10.1 Plantillas globales

La columna:

```txt
plantilla_global
```

permite aplicar reglas estándar predefinidas.

Ejemplos:

- temporal_estandar
- metrica_monetaria
- metrica_cantidad
- metrica_porcentaje
- precio_unitario
- estado_estandar

Esto permite que una consulta reutilice definiciones generales sin repetir lógica.

---

## 10.2 Derivados temporales

El catálogo debe soportar la idea de que un campo fecha genere múltiples derivados estándar.

Ejemplo:

Campo base:

- Fecha

Derivados:

- Año
- Semestre
- Trimestre
- Mes
- Año-Mes
- Semana
- Día
- Día de semana

Para eso se usan:

- `es_derivado_temporal`
- `campo_origen_derivado`

Ejemplo:

- `es_derivado_temporal = 1`
- `campo_origen_derivado = Fecha`

---

# 11. Agregaciones

Para métricas, el catálogo debe definir:

- `agregacion_default`
- `agregaciones_permitidas`

Ejemplos:

## Importe Neto
- agregacion_default = SUMA
- agregaciones_permitidas = SUMA,PROMEDIO,MAX,MIN

## Cantidad
- agregacion_default = SUMA
- agregaciones_permitidas = SUMA,PROMEDIO,MAX,MIN

## Precio Unitario
- agregacion_default = PROMEDIO
- agregaciones_permitidas = PROMEDIO,MAX,MIN

## Clientes Únicos
- agregacion_default = CONTEO_DISTINTO
- agregaciones_permitidas = CONTEO,CONTEO_DISTINTO

---

# 12. Formatos y orden natural

## `formato`
Valores sugeridos:

- texto
- numero
- moneda
- porcentaje
- fecha
- fechaHora
- booleano

## `orden_natural`
Valores sugeridos:

- asc
- desc
- fecha
- periodo
- calendarioMes
- diaSemana

Esto permite ordenar correctamente campos como:

- Mes
- Año-Mes
- Día de semana

---

# 13. Visibilidad y comportamiento

## `visible`
Indica si se muestra al usuario en el selector del pivot.

## `visible_por_defecto`
Indica si aparece inicialmente destacado o propuesto.

## `pivotable`
Indica si el campo participa en la construcción del pivot.

## `drillable`
Indica si el campo tiene utilidad en la apertura de detalle.

---

# 14. Métrica principal

La columna:

```txt
es_metrica_principal
```

permite identificar la métrica principal de una consulta.

Ejemplos:

- en ventas: Importe Neto
- en horas: Horas
- en inventario: Cantidad

Sirve para:

- pivot base
- sugerencias automáticas
- gráficos
- resúmenes

---

# 15. SQL conceptual propuesto

```sql
CREATE TABLE pq_pivots_campos
(
    campo_id bigint IDENTITY(1,1) NOT NULL,
    consulta_id varchar(100) NOT NULL,
    campo_codigo varchar(100) NOT NULL,
    nombre_tecnico varchar(200) NOT NULL,
    nombre_visible varchar(200) NOT NULL,
    descripcion varchar(500) NULL,
    tipo_campo varchar(30) NOT NULL,
    tipo_dato varchar(30) NOT NULL,
    categoria varchar(100) NOT NULL,
    roles_permitidos varchar(100) NOT NULL,
    visible bit NOT NULL CONSTRAINT DF_pq_pivots_campos_visible DEFAULT (1),
    pivotable bit NOT NULL CONSTRAINT DF_pq_pivots_campos_pivotable DEFAULT (1),
    drillable bit NOT NULL CONSTRAINT DF_pq_pivots_campos_drillable DEFAULT (0),
    orden_sugerido int NOT NULL CONSTRAINT DF_pq_pivots_campos_orden_sugerido DEFAULT (0),
    orden_natural varchar(50) NULL,
    formato varchar(50) NOT NULL,
    ancho_sugerido int NULL,
    plantilla_global varchar(100) NULL,
    es_derivado_temporal bit NOT NULL CONSTRAINT DF_pq_pivots_campos_es_derivado_temporal DEFAULT (0),
    campo_origen_derivado varchar(100) NULL,
    agregacion_default varchar(50) NULL,
    agregaciones_permitidas varchar(200) NULL,
    decimales int NULL,
    permite_totales bit NOT NULL CONSTRAINT DF_pq_pivots_campos_permite_totales DEFAULT (1),
    es_metrica_principal bit NOT NULL CONSTRAINT DF_pq_pivots_campos_es_metrica_principal DEFAULT (0),
    visible_por_defecto bit NOT NULL CONSTRAINT DF_pq_pivots_campos_visible_por_defecto DEFAULT (1),
    activo bit NOT NULL CONSTRAINT DF_pq_pivots_campos_activo DEFAULT (1),
    usuario_creacion varchar(100) NOT NULL,
    fecha_creacion datetime2 NOT NULL,
    usuario_ult_mod varchar(100) NOT NULL,
    fecha_ult_mod datetime2 NOT NULL,
    CONSTRAINT PK_pq_pivots_campos PRIMARY KEY (campo_id),
    CONSTRAINT FK_pq_pivots_campos_consulta
        FOREIGN KEY (consulta_id)
        REFERENCES pq_pivots_consultas (consulta_id)
);
GO

CREATE UNIQUE INDEX UX_pq_pivots_campos_consulta_codigo
    ON pq_pivots_campos (consulta_id, campo_codigo);
GO

CREATE INDEX IX_pq_pivots_campos_consulta_activo
    ON pq_pivots_campos (consulta_id, activo);
GO

CREATE INDEX IX_pq_pivots_campos_consulta_categoria
    ON pq_pivots_campos (consulta_id, categoria);
GO

CREATE INDEX IX_pq_pivots_campos_consulta_orden
    ON pq_pivots_campos (consulta_id, orden_sugerido);
GO
```

---

# 16. Ejemplos de registros

## 16.1 Campo dimensión

| consulta_id | campo_codigo | nombre_tecnico | nombre_visible | tipo_campo | tipo_dato | categoria | roles_permitidos | formato |
|---|---|---|---|---|---|---|---|---|
| ventas | Cliente | CLI_NOMBRE | Cliente | dimension | string | Cliente / Proveedor | fila,columna,filtro | texto |

## 16.2 Campo métrica

| consulta_id | campo_codigo | nombre_tecnico | nombre_visible | tipo_campo | tipo_dato | categoria | roles_permitidos | formato |
|---|---|---|---|---|---|---|---|---|
| ventas | ImporteNeto | IMP_NETO | Importe Neto | metrica | decimal | Importes | valor | moneda |

## 16.3 Campo temporal derivado

| consulta_id | campo_codigo | nombre_tecnico | nombre_visible | tipo_campo | tipo_dato | categoria | es_derivado_temporal | campo_origen_derivado |
|---|---|---|---|---|---|---|---:|---|
| ventas | AnioMes | ANIO_MES | Año-Mes | dimension | string | Tiempo | 1 | Fecha |

---

# 17. Beneficios del catálogo

Implementar `pq_pivots_campos` aporta estas ventajas:

1. desacopla metadata del código
2. unifica nombres visibles
3. facilita motor de pivots genérico
4. mejora validación de pivots guardados
5. permite reutilizar plantillas globales
6. mejora exportación y documentación
7. facilita soporte y mantenimiento

---

# 18. Orden lógico del modelo completo

Con este agregado, el motor de pivots queda estructurado así:

1. `pq_pivots_consultas`  
   Catálogo de consultas pivotables

2. `pq_pivots_campos`  
   Catálogo de campos disponibles por consulta

3. `pq_pivots_config`  
   Pivots guardados por usuarios

4. `pq_pivots_aud`  
   Auditoría opcional

---

# 19. Próxima mejora recomendada

La siguiente pieza muy útil sería una tabla de plantillas o reglas globales, por ejemplo:

```txt
pq_pivots_plantillas
```

y eventualmente:

```txt
pq_pivots_plantillas_det
```

Eso permitiría registrar formalmente:

- temporal_estandar
- metrica_monetaria
- metrica_cantidad
- metrica_porcentaje
- ordenes naturales
- formatos estándar

y llevar todavía más lejos la generalización que planteaste.

---

# 20. Resumen normativo

1. Toda consulta pivotable debe tener sus campos registrados en `pq_pivots_campos`.
2. `campo_codigo` debe ser estable y único dentro de la consulta.
3. `nombre_visible` nunca debe ser técnico.
4. Las métricas deben registrar agregaciones permitidas.
5. Los campos temporales derivados deben identificarse explícitamente.
6. El catálogo debe residir en Dictionary DB.
7. El prefijo debe ser `pq_pivots_`.

---

# Fin del documento

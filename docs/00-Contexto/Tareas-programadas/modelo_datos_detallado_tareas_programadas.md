# Modelo de datos detallado — Tareas automáticas programadas

## 1. Objetivo

Definir el modelo de datos detallado para soportar el subsistema de tareas automáticas programadas, contemplando:

- catálogo de procesos programables,
- definición funcional de tareas,
- asociación por empresa,
- parámetros configurables,
- estado interno técnico,
- ejecuciones,
- snapshot de configuración por corrida,
- logs estructurados,
- retención histórica,
- y soporte para ejecución manual, automática y simulación.

Este modelo toma como base el diseño conceptual previamente definido y corrige una limitación importante del modelo actual: la mezcla entre configuración de usuario, estado interno y resultado de ejecución.

---

## 2. Principios de modelado

## 2.1 Separación obligatoria de responsabilidades

El modelo debe separar físicamente las siguientes entidades:

1. **Catálogo de procesos programables**
2. **Definición de tarea programada**
3. **Empresas alcanzadas por la tarea**
4. **Parámetros configurables de la tarea**
5. **Estado interno técnico por tarea/empresa**
6. **Ejecuciones**
7. **Snapshot de parámetros por ejecución**
8. **Logs de ejecución**

## 2.2 Una tarea, muchas empresas, muchas ejecuciones

Una definición funcional de tarea puede estar asociada a varias empresas.

Cada disparo real genera una ejecución hija por empresa.

## 2.3 Toda ejecución debe quedar registrada

Debe registrarse toda corrida o intento relevante, incluso si termina:

- omitida,
- fallida en validación,
- abortada por solapamiento,
- en simulación,
- o sin registros para procesar.

## 2.4 El semáforo no se persiste

El color de UI no forma parte del modelo. Se deriva del estado final.

---

## 3. Entidades principales

El modelo propuesto se organiza en nueve grupos:

1. `PQ_TASK_PROCESS`
2. `PQ_TASK_PROCESS_PARAMETER`
3. `PQ_TASK_DEFINITION`
4. `PQ_TASK_DEFINITION_COMPANY`
5. `PQ_TASK_DEFINITION_PARAMETER`
6. `PQ_TASK_RUNTIME_STATE`
7. `PQ_TASK_EXECUTION_BATCH`
8. `PQ_TASK_EXECUTION`
9. `PQ_TASK_EXECUTION_PARAMETER`
10. `PQ_TASK_EXECUTION_LOG`

Se agregan además tablas opcionales recomendadas para futuro:

11. `PQ_TASK_PROCESS_STATE_FIELD`
12. `PQ_TASK_DEFINITION_AUDIT`

---

## 4. Catálogo de procesos programables

## 4.1 Tabla `PQ_TASK_PROCESS`

Representa cada proceso que el sistema puede ofrecer para automatización.

### Campos propuestos

- `process_id` (PK, bigint identity)
- `process_code` (varchar(100), unique, not null)
- `process_name` (varchar(200), not null)
- `process_description` (varchar(max), null)
- `module_code` (varchar(100), null)
- `module_name` (varchar(200), null)
- `handler_class` (varchar(300), not null)
- `is_active` (bit, not null, default 1)
- `supports_manual_execution` (bit, not null, default 1)
- `supports_scheduled_execution` (bit, not null, default 1)
- `supports_simulation` (bit, not null, default 0)
- `works_by_company` (bit, not null, default 1)
- `selection_strategy_auto` (varchar(50), not null)
- `selection_strategy_manual` (varchar(50), null)
- `idempotency_strategy` (varchar(50), null)
- `validation_handler` (varchar(300), null)
- `notes` (varchar(max), null)
- `created_at` (datetime2, not null)
- `updated_at` (datetime2, not null)
- `created_by` (varchar(100), null)
- `updated_by` (varchar(100), null)

### Reglas

- `process_code` debe ser estable y único.
- `handler_class` identifica el componente backend que implementa la lógica.
- `selection_strategy_auto` documenta cómo arma el lote automático.
- `selection_strategy_manual` documenta cómo se arma el lote manual.
- `idempotency_strategy` documenta la política de re-ejecución.

### Valores sugeridos

Para `selection_strategy_auto`:
- `ALL_AVAILABLE`
- `ALL_PENDING`
- `ALL_UNPROCESSED`
- `BY_CRITERIA`
- `CUSTOM`

Para `selection_strategy_manual`:
- `USER_SELECTION`
- `FILTER_AND_SELECT`
- `PERIOD_AND_SELECT`
- `NOT_APPLICABLE`
- `CUSTOM`

Para `idempotency_strategy`:
- `SAFE_REEXECUTION`
- `CONTROLLED_BY_MARKS`
- `CONTROLLED_BY_VALIDATION`
- `NOT_IDEMPOTENT`
- `CUSTOM`

---

## 4.2 Tabla `PQ_TASK_PROCESS_PARAMETER`

Define el catálogo de parámetros configurables admitidos por cada proceso programable.

### Campos propuestos

- `process_parameter_id` (PK, bigint identity)
- `process_id` (FK -> `PQ_TASK_PROCESS.process_id`, not null)
- `parameter_code` (varchar(100), not null)
- `parameter_name` (varchar(200), not null)
- `parameter_description` (varchar(max), null)
- `data_type` (varchar(30), not null)
- `ui_control_type` (varchar(50), null)
- `is_required` (bit, not null, default 0)
- `default_value_text` (varchar(max), null)
- `allowed_values_source` (varchar(200), null)
- `validation_rule` (varchar(max), null)
- `display_order` (int, not null, default 0)
- `is_user_editable` (bit, not null, default 1)
- `is_multivalue` (bit, not null, default 0)
- `example_value` (varchar(500), null)
- `created_at` (datetime2, not null)
- `updated_at` (datetime2, not null)

### Tipos permitidos sugeridos

- `STRING`
- `INT`
- `DECIMAL`
- `BOOL`
- `DATETIME`

### Controles UI sugeridos

- `TEXTBOX`
- `TEXTAREA`
- `CHECKBOX`
- `DATEPICKER`
- `NUMBER`
- `EMAIL_LIST`
- `PATH_TEXTBOX`
- `SELECT`
- `MULTISELECT`

### Reglas

- Un proceso puede no tener parámetros configurables.
- `parameter_code` debe ser único por `process_id`.
- `is_multivalue = 1` permite modelar valores como lista separada por comas, por ejemplo emails.

---

## 4.3 Tabla opcional `PQ_TASK_PROCESS_STATE_FIELD`

Documenta campos de estado interno técnico que el proceso utiliza, pero que no debe editar el usuario.

### Campos propuestos

- `process_state_field_id` (PK, bigint identity)
- `process_id` (FK -> `PQ_TASK_PROCESS.process_id`, not null)
- `field_code` (varchar(100), not null)
- `field_name` (varchar(200), not null)
- `field_description` (varchar(max), null)
- `data_type` (varchar(30), not null)
- `is_persisted` (bit, not null, default 1)
- `notes` (varchar(max), null)

### Uso

Sirve para documentar y estructurar estado interno como:

- último ID procesado,
- última fecha,
- último aviso emitido,
- última cantidad de errores,
- otros equivalentes.

---

## 5. Definición funcional de tarea

## 5.1 Tabla `PQ_TASK_DEFINITION`

Representa la configuración funcional creada por el usuario a partir de un proceso del catálogo.

### Campos propuestos

- `task_definition_id` (PK, bigint identity)
- `process_id` (FK -> `PQ_TASK_PROCESS.process_id`, not null)
- `task_code` (varchar(100), unique, null)
- `task_title` (varchar(200), not null)
- `task_description` (varchar(max), null)
- `is_active` (bit, not null, default 1)
- `frequency_type` (varchar(30), not null)
- `frequency_interval` (int, null)
- `frequency_days_mask` (varchar(20), null)
- `base_date` (date, null)
- `base_time` (time, null)
- `next_run_at` (datetime2, null)
- `last_run_at` (datetime2, null)
- `last_status_code` (varchar(30), null)
- `allow_manual_execution` (bit, not null, default 1)
- `allow_simulation` (bit, not null, default 0)
- `version_no` (int, not null, default 1)
- `notes` (varchar(max), null)
- `created_at` (datetime2, not null)
- `updated_at` (datetime2, not null)
- `created_by` (varchar(100), null)
- `updated_by` (varchar(100), null)

### Reglas

- `allow_manual_execution` y `allow_simulation` no pueden violar lo permitido por el catálogo del proceso.
- `next_run_at` expresa la próxima ejecución planificada del encabezado.
- `last_status_code` es informativo del último resultado agregado; no reemplaza el historial.

### Valores sugeridos

Para `frequency_type`:
- `HOURLY_INTERVAL`
- `DAILY`
- `WEEKLY`
- `MONTHLY`
- `CUSTOM`

Para `last_status_code`:
- `EN_EJECUCION`
- `EXITOSA`
- `EXITOSA_CON_OBSERVACIONES`
- `FALLIDA`
- `OMITIDA`

---

## 5.2 Tabla `PQ_TASK_DEFINITION_COMPANY`

Asocia una tarea a una o varias empresas.

### Campos propuestos

- `task_definition_company_id` (PK, bigint identity)
- `task_definition_id` (FK -> `PQ_TASK_DEFINITION.task_definition_id`, not null)
- `company_code` (varchar(50), not null)
- `company_name` (varchar(200), null)
- `is_active` (bit, not null, default 1)
- `created_at` (datetime2, not null)
- `updated_at` (datetime2, not null)

### Reglas

- Debe existir unicidad por (`task_definition_id`, `company_code`).
- Una tarea sin empresas asociadas no debe ejecutar.

---

## 5.3 Tabla `PQ_TASK_DEFINITION_PARAMETER`

Guarda los parámetros configurados por el usuario para una definición de tarea.

### Campos propuestos

- `task_definition_parameter_id` (PK, bigint identity)
- `task_definition_id` (FK -> `PQ_TASK_DEFINITION.task_definition_id`, not null)
- `process_parameter_id` (FK -> `PQ_TASK_PROCESS_PARAMETER.process_parameter_id`, not null)
- `parameter_code` (varchar(100), not null)
- `value_string` (varchar(max), null)
- `value_int` (int, null)
- `value_decimal` (decimal(18,4), null)
- `value_bool` (bit, null)
- `value_datetime` (datetime2, null)
- `value_json` (nvarchar(max), null)
- `created_at` (datetime2, not null)
- `updated_at` (datetime2, not null)

### Reglas

- Debe existir unicidad por (`task_definition_id`, `parameter_code`).
- El tipo efectivo debe coincidir con el definido en `PQ_TASK_PROCESS_PARAMETER`.
- `value_json` se reserva para crecimiento futuro o estructuras complejas.

---

## 5.4 Tabla opcional `PQ_TASK_DEFINITION_AUDIT`

Permite guardar historial de cambios de la definición.

### Campos propuestos

- `task_definition_audit_id` (PK, bigint identity)
- `task_definition_id` (FK -> `PQ_TASK_DEFINITION.task_definition_id`, not null)
- `version_no` (int, not null)
- `change_type` (varchar(30), not null)
- `changed_at` (datetime2, not null)
- `changed_by` (varchar(100), null)
- `summary` (varchar(max), null)
- `snapshot_json` (nvarchar(max), not null)

### Uso

No es imprescindible para la primera versión, pero sí recomendable si se quiere trazabilidad completa de cambios de configuración.

---

## 6. Estado interno técnico por tarea y empresa

## 6.1 Tabla `PQ_TASK_RUNTIME_STATE`

Guarda estado técnico interno persistente por combinación de tarea y empresa.

### Objetivo

Separar claramente el estado interno del proceso de los parámetros visibles al usuario.

### Campos propuestos

- `task_runtime_state_id` (PK, bigint identity)
- `task_definition_id` (FK -> `PQ_TASK_DEFINITION.task_definition_id`, not null)
- `company_code` (varchar(50), not null)
- `state_key` (varchar(100), not null)
- `state_data_type` (varchar(30), not null)
- `value_string` (varchar(max), null)
- `value_int` (int, null)
- `value_decimal` (decimal(18,4), null)
- `value_bool` (bit, null)
- `value_datetime` (datetime2, null)
- `value_json` (nvarchar(max), null)
- `updated_at` (datetime2, not null)
- `updated_by_execution_id` (bigint, null)

### Reglas

- Debe existir unicidad por (`task_definition_id`, `company_code`, `state_key`).
- Esta tabla no debe editarse desde la pantalla de definición.
- El proceso puede leer y actualizar estos valores durante la ejecución.

### Ejemplos de uso

- `ULTIMO_ID`
- `ULTIMA_FECHA`
- `ULTIMO_EMAIL`
- `ULTIMA_CANTIDAD_ERRORES`

---

## 7. Modelo de ejecución

## 7.1 Tabla `PQ_TASK_EXECUTION_BATCH`

Representa un disparo general de una tarea, que luego se descompone en una ejecución por empresa.

### Objetivo

Tener un encabezado agrupador cuando una misma tarea se dispara para varias empresas al mismo tiempo.

### Campos propuestos

- `execution_batch_id` (PK, bigint identity)
- `task_definition_id` (FK -> `PQ_TASK_DEFINITION.task_definition_id`, not null)
- `trigger_type` (varchar(20), not null)
- `run_mode` (varchar(20), not null)
- `requested_by` (varchar(100), null)
- `requested_at` (datetime2, not null)
- `scheduler_reference` (varchar(200), null)
- `version_no_used` (int, not null)
- `status_code` (varchar(30), not null)
- `started_at` (datetime2, null)
- `finished_at` (datetime2, null)
- `summary_message` (varchar(max), null)

### Valores sugeridos

Para `trigger_type`:
- `PROGRAMADA`
- `MANUAL`

Para `run_mode`:
- `REAL`
- `SIMULACION`

### Reglas

- Sirve como agrupador transversal.
- Si la tarea corre para una sola empresa, igual puede existir por consistencia.

---

## 7.2 Tabla `PQ_TASK_EXECUTION`

Representa la ejecución concreta de una tarea para una empresa determinada.

### Campos propuestos

- `execution_id` (PK, bigint identity)
- `execution_batch_id` (FK -> `PQ_TASK_EXECUTION_BATCH.execution_batch_id`, not null)
- `task_definition_id` (FK -> `PQ_TASK_DEFINITION.task_definition_id`, not null)
- `process_id` (FK -> `PQ_TASK_PROCESS.process_id`, not null)
- `company_code` (varchar(50), not null)
- `trigger_type` (varchar(20), not null)
- `run_mode` (varchar(20), not null)
- `status_code` (varchar(30), not null)
- `started_at` (datetime2, not null)
- `finished_at` (datetime2, null)
- `requested_by` (varchar(100), null)
- `version_no_used` (int, not null)
- `scheduled_for` (datetime2, null)
- `summary_message` (varchar(max), null)
- `error_code` (varchar(100), null)
- `is_overlapped_skip` (bit, not null, default 0)
- `created_at` (datetime2, not null)

### Reglas

- Debe crearse al inicio con estado `EN_EJECUCION`.
- Debe actualizarse al final con el estado definitivo.
- `is_overlapped_skip = 1` identifica omisión por solapamiento.
- El usuario debe poder distinguir si fue manual o programada.
- Debe guardar si la corrida fue real o simulada.

### Estados permitidos sugeridos

- `EN_EJECUCION`
- `EXITOSA`
- `EXITOSA_CON_OBSERVACIONES`
- `FALLIDA`
- `OMITIDA`
- `CANCELADA`

---

## 7.3 Tabla `PQ_TASK_EXECUTION_PARAMETER`

Guarda snapshot congelado de los parámetros usados en una ejecución.

### Campos propuestos

- `execution_parameter_id` (PK, bigint identity)
- `execution_id` (FK -> `PQ_TASK_EXECUTION.execution_id`, not null)
- `parameter_code` (varchar(100), not null)
- `parameter_name` (varchar(200), null)
- `data_type` (varchar(30), not null)
- `value_string` (varchar(max), null)
- `value_int` (int, null)
- `value_decimal` (decimal(18,4), null)
- `value_bool` (bit, null)
- `value_datetime` (datetime2, null)
- `value_json` (nvarchar(max), null)

### Reglas

- Debe copiarse desde la definición al inicio de cada ejecución.
- Puede incluir también valores técnicos resueltos al momento de correr, si se considera útil para diagnóstico.
- Debe existir unicidad por (`execution_id`, `parameter_code`).

---

## 7.4 Tabla `PQ_TASK_EXECUTION_LOG`

Guarda el detalle cronológico de mensajes de una ejecución.

### Campos propuestos

- `execution_log_id` (PK, bigint identity)
- `execution_id` (FK -> `PQ_TASK_EXECUTION.execution_id`, not null)
- `log_timestamp` (datetime2, not null)
- `log_level` (varchar(20), not null)
- `log_source` (varchar(200), null)
- `message_text` (varchar(max), not null)
- `message_code` (varchar(100), null)
- `context_json` (nvarchar(max), null)
- `sequence_no` (int, not null)

### Valores sugeridos

Para `log_level`:
- `INFO`
- `WARNING`
- `ERROR`

### Reglas

- Debe haber índice por `execution_id` y `sequence_no`.
- Debe poder filtrarse por nivel.
- `context_json` permite anexar datos técnicos o funcionales adicionales.

---

## 8. Relaciones entre tablas

## 8.1 Relación general

- Un `PQ_TASK_PROCESS` tiene muchos `PQ_TASK_PROCESS_PARAMETER`.
- Un `PQ_TASK_PROCESS` puede tener muchos `PQ_TASK_PROCESS_STATE_FIELD`.
- Un `PQ_TASK_PROCESS` puede originar muchas `PQ_TASK_DEFINITION`.
- Una `PQ_TASK_DEFINITION` tiene muchas `PQ_TASK_DEFINITION_COMPANY`.
- Una `PQ_TASK_DEFINITION` tiene muchos `PQ_TASK_DEFINITION_PARAMETER`.
- Una `PQ_TASK_DEFINITION` tiene muchos `PQ_TASK_RUNTIME_STATE`.
- Una `PQ_TASK_DEFINITION` tiene muchos `PQ_TASK_EXECUTION_BATCH`.
- Un `PQ_TASK_EXECUTION_BATCH` tiene muchas `PQ_TASK_EXECUTION`.
- Una `PQ_TASK_EXECUTION` tiene muchos `PQ_TASK_EXECUTION_PARAMETER`.
- Una `PQ_TASK_EXECUTION` tiene muchos `PQ_TASK_EXECUTION_LOG`.

---

## 9. Reglas de integridad y comportamiento

## 9.1 Validación previa obligatoria

Antes de crear o lanzar una ejecución, el motor debe validar:

- tarea activa,
- empresas asociadas activas,
- parámetros obligatorios completos,
- no solapamiento,
- elegibilidad del modo (manual/simulación),
- elegibilidad del calendario en caso automático.

## 9.2 Política de solapamiento

Si existe una ejecución en curso para la misma tarea y empresa:

- no se crea una segunda ejecución concurrente real,
- debe quedar registro con `status_code = OMITIDA`,
- `is_overlapped_skip = 1`,
- y log de error por solapamiento.

## 9.3 Política de manual vs programada

La ejecución manual:
- se registra con `trigger_type = MANUAL`,
- corre inmediatamente,
- no recalcula calendario.

La automática:
- se registra con `trigger_type = PROGRAMADA`,
- usa `scheduled_for` para conservar el momento previsto.

## 9.4 Política de simulación

Si el proceso soporta simulación y el usuario la solicita:

- `run_mode = SIMULACION`,
- la ejecución corre con logs normales,
- no debe materializar efectos definitivos.

---

## 10. Índices recomendados

## 10.1 `PQ_TASK_PROCESS`

- unique index en `process_code`
- index en `module_code`
- index en `is_active`

## 10.2 `PQ_TASK_PROCESS_PARAMETER`

- unique index en (`process_id`, `parameter_code`)
- index en `display_order`

## 10.3 `PQ_TASK_DEFINITION`

- unique index en `task_code` si se usa
- index en `process_id`
- index en `is_active`
- index en `next_run_at`
- index en (`is_active`, `next_run_at`)

## 10.4 `PQ_TASK_DEFINITION_COMPANY`

- unique index en (`task_definition_id`, `company_code`)
- index en `company_code`

## 10.5 `PQ_TASK_DEFINITION_PARAMETER`

- unique index en (`task_definition_id`, `parameter_code`)

## 10.6 `PQ_TASK_RUNTIME_STATE`

- unique index en (`task_definition_id`, `company_code`, `state_key`)

## 10.7 `PQ_TASK_EXECUTION_BATCH`

- index en `task_definition_id`
- index en `requested_at`
- index en `status_code`

## 10.8 `PQ_TASK_EXECUTION`

- index en `task_definition_id`
- index en `company_code`
- index en `status_code`
- index en `started_at`
- index en (`task_definition_id`, `company_code`, `status_code`)
- index filtrado o equivalente para `status_code = EN_EJECUCION`

## 10.9 `PQ_TASK_EXECUTION_PARAMETER`

- unique index en (`execution_id`, `parameter_code`)

## 10.10 `PQ_TASK_EXECUTION_LOG`

- index en (`execution_id`, `sequence_no`)
- index en (`execution_id`, `log_level`)
- index en `log_timestamp`

---

## 11. Política de retención

La retención se define mediante configuración global del sistema: cantidad de meses a conservar.

### Alcance

Debe aplicarse a:

- `PQ_TASK_EXECUTION_LOG`
- `PQ_TASK_EXECUTION_PARAMETER`
- `PQ_TASK_EXECUTION`
- `PQ_TASK_EXECUTION_BATCH` cuando ya no tenga hijas vigentes

### Regla sugerida

- primero depurar logs,
- luego snapshot de parámetros,
- luego ejecuciones hijas,
- por último batches huérfanos.

La definición de tarea, empresas, parámetros vigentes y estado interno no deberían depurarse por esta política.

---

## 12. Casos de uso mapeados al modelo

## 12.1 Autorización automática de pedidos

### En catálogo

`PQ_TASK_PROCESS`
- `process_code = AUTORIZACION_PEDIDOS_AUTO`
- `supports_manual_execution = 1`
- `supports_scheduled_execution = 1`
- `supports_simulation = 1` (recomendado)

### Parámetros de proceso

En `PQ_TASK_PROCESS_PARAMETER`:
- `TALONARIO_PEDIDO` (INT)
- `DIAS_TOLERANCIA` (INT)
- `NO_PENDIENTES_PARCIALES` (BOOL)
- `EMAILS` (STRING, `is_multivalue = 1`)
- `ASUNTO` (STRING)
- eventualmente `MONTO_TOLERANCIA` si sigue vigente

### Ejecución

- una definición funcional,
- varias empresas asociadas,
- batch por disparo,
- ejecución por empresa,
- log detallado,
- snapshot congelado de parámetros usados.

## 12.2 Envío automático de mails con control incremental

### Configuración visible

- destinatarios,
- asunto,
- mensaje,
- fecha de inicio.

### Estado interno técnico

En `PQ_TASK_RUNTIME_STATE`:
- `ULTIMO_ID`
- `ULTIMA_FECHA`

De esta forma se evita mezclar el avance técnico del proceso con la configuración funcional.

## 12.3 Egresos automáticos a sucursales

### Configuración visible

- `FECHA_INICIO`
- `COMPROBANTE_STOCK`
- `MAIL_AVISOS`
- `TIPO_SELECCION_SUCURSAL`
- `REMITO_COMPLETO_A_SUCURSAL`

### Estado interno técnico

- `ULTIMO_EMAIL`
- `ULTIMA_CANTIDAD_ERRORES`

---

## 13. Decisiones de diseño ya adoptadas y reflejadas en el modelo

Este modelo incorpora explícitamente las siguientes decisiones ya definidas:

- una tarea puede tener múltiples empresas;
- una ejecución real se registra por empresa;
- la ejecución manual corre inmediatamente;
- la ejecución manual no altera el calendario;
- no hay reintentos automáticos;
- el solapamiento genera omisión con log de error;
- los logs tienen niveles;
- el estado `EN_EJECUCION` debe persistirse al inicio;
- no hace falta mostrar avance en vivo;
- la consulta tendrá botón de actualizar;
- el historial se conserva según configuración global;
- el snapshot de parámetros queda congelado por ejecución;
- el modelo debe soportar simulación;
- la mayoría de tareas pueden no tener parámetros adicionales;
- mails multivalor pueden seguir guardándose como string separado por comas, validado por lógica de aplicación;
- rutas se consideran rutas locales del servidor y deben tratarse como texto parametrizado.

---

## 14. Recomendaciones de implementación para Cursor

1. Crear primero el catálogo de procesos y parámetros.
2. Separar totalmente parámetros visibles de estado interno.
3. Implementar el motor de ejecución sobre `PQ_TASK_EXECUTION_BATCH` + `PQ_TASK_EXECUTION`.
4. Persistir `EN_EJECUCION` al inicio, incluso antes del procesamiento real.
5. Copiar snapshot de parámetros al comenzar cada ejecución.
6. Exponer logs por nivel y en orden secuencial.
7. No usar el campo “último estado” como fuente principal de verdad; usar siempre historial de ejecuciones.
8. Mantener `next_run_at` como dato operativo del scheduler, pero no como reemplazo del historial.
9. Implementar validación previa común antes de invocar cualquier proceso.
10. Diseñar la UI de parámetros desde el catálogo (`PQ_TASK_PROCESS_PARAMETER`).

---

## 15. Próximo paso sugerido

A partir de este modelo de datos detallado, el siguiente documento natural debería ser uno de estos dos:

1. **modelo relacional técnico con SQL Server**
   - tipos concretos,
   - PK/FK,
   - índices,
   - constraints,
   - scripts de creación.

2. **documentación técnica para Cursor**
   - contratos backend,
   - servicios,
   - flujo de scheduler,
   - flujo manual,
   - endpoints,
   - y comportamiento esperado por pantalla.

Lo ideal es generar ambos, en ese orden.


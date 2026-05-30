# Modelo relacional técnico SQL Server — Tareas automáticas programadas

## 1. Objetivo

Definir el modelo relacional técnico para SQL Server del subsistema de tareas automáticas programadas, incluyendo:

- tablas,
- claves primarias,
- claves foráneas,
- restricciones,
- índices,
- convenciones de nombres,
- y scripts base `CREATE TABLE`.

Este documento materializa el modelo de datos detallado previamente definido.

---

## 2. Convenciones adoptadas

### 2.1 Prefijo

Todas las tablas del subsistema usan el prefijo:

- `PQ_TASK_`

### 2.2 Tipos generales

- claves primarias: `BIGINT IDENTITY(1,1)`
- fechas con hora: `DATETIME2(0)`
- fechas puras: `DATE`
- horas puras: `TIME(0)`
- textos cortos: `VARCHAR(n)`
- textos largos: `VARCHAR(MAX)`
- estructuras extensibles: `NVARCHAR(MAX)`
- booleanos: `BIT`
- importes: `DECIMAL(18,4)`

### 2.3 Auditoría mínima

Siempre que aplique, se incluyen:

- `created_at`
- `updated_at`
- `created_by`
- `updated_by`

### 2.4 Catálogos lógicos

Por simplicidad y flexibilidad inicial, varios estados/códigos se modelan como `VARCHAR`, no como tablas de dominio separadas.

Esto permite evolucionar reglas sin sobrediseñar en la primera etapa.

---

## 3. Orden sugerido de creación

1. `PQ_TASK_PROCESS`
2. `PQ_TASK_PROCESS_PARAMETER`
3. `PQ_TASK_PROCESS_STATE_FIELD`
4. `PQ_TASK_DEFINITION`
5. `PQ_TASK_DEFINITION_COMPANY`
6. `PQ_TASK_DEFINITION_PARAMETER`
7. `PQ_TASK_DEFINITION_AUDIT`
8. `PQ_TASK_RUNTIME_STATE`
9. `PQ_TASK_EXECUTION_BATCH`
10. `PQ_TASK_EXECUTION`
11. `PQ_TASK_EXECUTION_PARAMETER`
12. `PQ_TASK_EXECUTION_LOG`

---

## 4. Scripts SQL

## 4.1 `PQ_TASK_PROCESS`

```sql
CREATE TABLE PQ_TASK_PROCESS (
    process_id BIGINT IDENTITY(1,1) NOT NULL,
    process_code VARCHAR(100) NOT NULL,
    process_name VARCHAR(200) NOT NULL,
    process_description VARCHAR(MAX) NULL,
    module_code VARCHAR(100) NULL,
    module_name VARCHAR(200) NULL,
    handler_class VARCHAR(300) NOT NULL,
    is_active BIT NOT NULL CONSTRAINT DF_PQ_TASK_PROCESS_is_active DEFAULT (1),
    supports_manual_execution BIT NOT NULL CONSTRAINT DF_PQ_TASK_PROCESS_supports_manual DEFAULT (1),
    supports_scheduled_execution BIT NOT NULL CONSTRAINT DF_PQ_TASK_PROCESS_supports_scheduled DEFAULT (1),
    supports_simulation BIT NOT NULL CONSTRAINT DF_PQ_TASK_PROCESS_supports_simulation DEFAULT (0),
    works_by_company BIT NOT NULL CONSTRAINT DF_PQ_TASK_PROCESS_works_by_company DEFAULT (1),
    selection_strategy_auto VARCHAR(50) NOT NULL,
    selection_strategy_manual VARCHAR(50) NULL,
    idempotency_strategy VARCHAR(50) NULL,
    validation_handler VARCHAR(300) NULL,
    notes VARCHAR(MAX) NULL,
    created_at DATETIME2(0) NOT NULL,
    updated_at DATETIME2(0) NOT NULL,
    created_by VARCHAR(100) NULL,
    updated_by VARCHAR(100) NULL,
    CONSTRAINT PK_PQ_TASK_PROCESS PRIMARY KEY (process_id),
    CONSTRAINT UQ_PQ_TASK_PROCESS_process_code UNIQUE (process_code)
);
GO
```

## 4.2 `PQ_TASK_PROCESS_PARAMETER`

```sql
CREATE TABLE PQ_TASK_PROCESS_PARAMETER (
    process_parameter_id BIGINT IDENTITY(1,1) NOT NULL,
    process_id BIGINT NOT NULL,
    parameter_code VARCHAR(100) NOT NULL,
    parameter_name VARCHAR(200) NOT NULL,
    parameter_description VARCHAR(MAX) NULL,
    data_type VARCHAR(30) NOT NULL,
    ui_control_type VARCHAR(50) NULL,
    is_required BIT NOT NULL CONSTRAINT DF_PQ_TASK_PROCESS_PARAMETER_is_required DEFAULT (0),
    default_value_text VARCHAR(MAX) NULL,
    allowed_values_source VARCHAR(200) NULL,
    validation_rule VARCHAR(MAX) NULL,
    display_order INT NOT NULL CONSTRAINT DF_PQ_TASK_PROCESS_PARAMETER_display_order DEFAULT (0),
    is_user_editable BIT NOT NULL CONSTRAINT DF_PQ_TASK_PROCESS_PARAMETER_is_user_editable DEFAULT (1),
    is_multivalue BIT NOT NULL CONSTRAINT DF_PQ_TASK_PROCESS_PARAMETER_is_multivalue DEFAULT (0),
    example_value VARCHAR(500) NULL,
    created_at DATETIME2(0) NOT NULL,
    updated_at DATETIME2(0) NOT NULL,
    CONSTRAINT PK_PQ_TASK_PROCESS_PARAMETER PRIMARY KEY (process_parameter_id),
    CONSTRAINT FK_PQ_TASK_PROCESS_PARAMETER_process FOREIGN KEY (process_id)
        REFERENCES PQ_TASK_PROCESS(process_id),
    CONSTRAINT UQ_PQ_TASK_PROCESS_PARAMETER_process_code UNIQUE (process_id, parameter_code)
);
GO
```

## 4.3 `PQ_TASK_PROCESS_STATE_FIELD`

```sql
CREATE TABLE PQ_TASK_PROCESS_STATE_FIELD (
    process_state_field_id BIGINT IDENTITY(1,1) NOT NULL,
    process_id BIGINT NOT NULL,
    field_code VARCHAR(100) NOT NULL,
    field_name VARCHAR(200) NOT NULL,
    field_description VARCHAR(MAX) NULL,
    data_type VARCHAR(30) NOT NULL,
    is_persisted BIT NOT NULL CONSTRAINT DF_PQ_TASK_PROCESS_STATE_FIELD_is_persisted DEFAULT (1),
    notes VARCHAR(MAX) NULL,
    CONSTRAINT PK_PQ_TASK_PROCESS_STATE_FIELD PRIMARY KEY (process_state_field_id),
    CONSTRAINT FK_PQ_TASK_PROCESS_STATE_FIELD_process FOREIGN KEY (process_id)
        REFERENCES PQ_TASK_PROCESS(process_id),
    CONSTRAINT UQ_PQ_TASK_PROCESS_STATE_FIELD_process_code UNIQUE (process_id, field_code)
);
GO
```

## 4.4 `PQ_TASK_DEFINITION`

```sql
CREATE TABLE PQ_TASK_DEFINITION (
    task_definition_id BIGINT IDENTITY(1,1) NOT NULL,
    process_id BIGINT NOT NULL,
    task_code VARCHAR(100) NULL,
    task_title VARCHAR(200) NOT NULL,
    task_description VARCHAR(MAX) NULL,
    is_active BIT NOT NULL CONSTRAINT DF_PQ_TASK_DEFINITION_is_active DEFAULT (1),
    frequency_type VARCHAR(30) NOT NULL,
    frequency_interval INT NULL,
    frequency_days_mask VARCHAR(20) NULL,
    base_date DATE NULL,
    base_time TIME(0) NULL,
    next_run_at DATETIME2(0) NULL,
    last_run_at DATETIME2(0) NULL,
    last_status_code VARCHAR(30) NULL,
    allow_manual_execution BIT NOT NULL CONSTRAINT DF_PQ_TASK_DEFINITION_allow_manual DEFAULT (1),
    allow_simulation BIT NOT NULL CONSTRAINT DF_PQ_TASK_DEFINITION_allow_simulation DEFAULT (0),
    version_no INT NOT NULL CONSTRAINT DF_PQ_TASK_DEFINITION_version_no DEFAULT (1),
    notes VARCHAR(MAX) NULL,
    created_at DATETIME2(0) NOT NULL,
    updated_at DATETIME2(0) NOT NULL,
    created_by VARCHAR(100) NULL,
    updated_by VARCHAR(100) NULL,
    CONSTRAINT PK_PQ_TASK_DEFINITION PRIMARY KEY (task_definition_id),
    CONSTRAINT FK_PQ_TASK_DEFINITION_process FOREIGN KEY (process_id)
        REFERENCES PQ_TASK_PROCESS(process_id),
    CONSTRAINT UQ_PQ_TASK_DEFINITION_task_code UNIQUE (task_code)
);
GO
```

## 4.5 `PQ_TASK_DEFINITION_COMPANY`

```sql
CREATE TABLE PQ_TASK_DEFINITION_COMPANY (
    task_definition_company_id BIGINT IDENTITY(1,1) NOT NULL,
    task_definition_id BIGINT NOT NULL,
    company_code VARCHAR(50) NOT NULL,
    company_name VARCHAR(200) NULL,
    is_active BIT NOT NULL CONSTRAINT DF_PQ_TASK_DEFINITION_COMPANY_is_active DEFAULT (1),
    created_at DATETIME2(0) NOT NULL,
    updated_at DATETIME2(0) NOT NULL,
    CONSTRAINT PK_PQ_TASK_DEFINITION_COMPANY PRIMARY KEY (task_definition_company_id),
    CONSTRAINT FK_PQ_TASK_DEFINITION_COMPANY_definition FOREIGN KEY (task_definition_id)
        REFERENCES PQ_TASK_DEFINITION(task_definition_id),
    CONSTRAINT UQ_PQ_TASK_DEFINITION_COMPANY UNIQUE (task_definition_id, company_code)
);
GO
```

## 4.6 `PQ_TASK_DEFINITION_PARAMETER`

```sql
CREATE TABLE PQ_TASK_DEFINITION_PARAMETER (
    task_definition_parameter_id BIGINT IDENTITY(1,1) NOT NULL,
    task_definition_id BIGINT NOT NULL,
    process_parameter_id BIGINT NOT NULL,
    parameter_code VARCHAR(100) NOT NULL,
    value_string VARCHAR(MAX) NULL,
    value_int INT NULL,
    value_decimal DECIMAL(18,4) NULL,
    value_bool BIT NULL,
    value_datetime DATETIME2(0) NULL,
    value_json NVARCHAR(MAX) NULL,
    created_at DATETIME2(0) NOT NULL,
    updated_at DATETIME2(0) NOT NULL,
    CONSTRAINT PK_PQ_TASK_DEFINITION_PARAMETER PRIMARY KEY (task_definition_parameter_id),
    CONSTRAINT FK_PQ_TASK_DEFINITION_PARAMETER_definition FOREIGN KEY (task_definition_id)
        REFERENCES PQ_TASK_DEFINITION(task_definition_id),
    CONSTRAINT FK_PQ_TASK_DEFINITION_PARAMETER_process_parameter FOREIGN KEY (process_parameter_id)
        REFERENCES PQ_TASK_PROCESS_PARAMETER(process_parameter_id),
    CONSTRAINT UQ_PQ_TASK_DEFINITION_PARAMETER UNIQUE (task_definition_id, parameter_code)
);
GO
```

## 4.7 `PQ_TASK_DEFINITION_AUDIT`

```sql
CREATE TABLE PQ_TASK_DEFINITION_AUDIT (
    task_definition_audit_id BIGINT IDENTITY(1,1) NOT NULL,
    task_definition_id BIGINT NOT NULL,
    version_no INT NOT NULL,
    change_type VARCHAR(30) NOT NULL,
    changed_at DATETIME2(0) NOT NULL,
    changed_by VARCHAR(100) NULL,
    summary VARCHAR(MAX) NULL,
    snapshot_json NVARCHAR(MAX) NOT NULL,
    CONSTRAINT PK_PQ_TASK_DEFINITION_AUDIT PRIMARY KEY (task_definition_audit_id),
    CONSTRAINT FK_PQ_TASK_DEFINITION_AUDIT_definition FOREIGN KEY (task_definition_id)
        REFERENCES PQ_TASK_DEFINITION(task_definition_id)
);
GO
```

## 4.8 `PQ_TASK_RUNTIME_STATE`

```sql
CREATE TABLE PQ_TASK_RUNTIME_STATE (
    task_runtime_state_id BIGINT IDENTITY(1,1) NOT NULL,
    task_definition_id BIGINT NOT NULL,
    company_code VARCHAR(50) NOT NULL,
    state_key VARCHAR(100) NOT NULL,
    state_data_type VARCHAR(30) NOT NULL,
    value_string VARCHAR(MAX) NULL,
    value_int INT NULL,
    value_decimal DECIMAL(18,4) NULL,
    value_bool BIT NULL,
    value_datetime DATETIME2(0) NULL,
    value_json NVARCHAR(MAX) NULL,
    updated_at DATETIME2(0) NOT NULL,
    updated_by_execution_id BIGINT NULL,
    CONSTRAINT PK_PQ_TASK_RUNTIME_STATE PRIMARY KEY (task_runtime_state_id),
    CONSTRAINT FK_PQ_TASK_RUNTIME_STATE_definition FOREIGN KEY (task_definition_id)
        REFERENCES PQ_TASK_DEFINITION(task_definition_id),
    CONSTRAINT UQ_PQ_TASK_RUNTIME_STATE UNIQUE (task_definition_id, company_code, state_key)
);
GO
```

## 4.9 `PQ_TASK_EXECUTION_BATCH`

```sql
CREATE TABLE PQ_TASK_EXECUTION_BATCH (
    execution_batch_id BIGINT IDENTITY(1,1) NOT NULL,
    task_definition_id BIGINT NOT NULL,
    trigger_type VARCHAR(20) NOT NULL,
    run_mode VARCHAR(20) NOT NULL,
    requested_by VARCHAR(100) NULL,
    requested_at DATETIME2(0) NOT NULL,
    scheduler_reference VARCHAR(200) NULL,
    version_no_used INT NOT NULL,
    status_code VARCHAR(30) NOT NULL,
    started_at DATETIME2(0) NULL,
    finished_at DATETIME2(0) NULL,
    summary_message VARCHAR(MAX) NULL,
    CONSTRAINT PK_PQ_TASK_EXECUTION_BATCH PRIMARY KEY (execution_batch_id),
    CONSTRAINT FK_PQ_TASK_EXECUTION_BATCH_definition FOREIGN KEY (task_definition_id)
        REFERENCES PQ_TASK_DEFINITION(task_definition_id)
);
GO
```

## 4.10 `PQ_TASK_EXECUTION`

```sql
CREATE TABLE PQ_TASK_EXECUTION (
    execution_id BIGINT IDENTITY(1,1) NOT NULL,
    execution_batch_id BIGINT NOT NULL,
    task_definition_id BIGINT NOT NULL,
    process_id BIGINT NOT NULL,
    company_code VARCHAR(50) NOT NULL,
    trigger_type VARCHAR(20) NOT NULL,
    run_mode VARCHAR(20) NOT NULL,
    status_code VARCHAR(30) NOT NULL,
    started_at DATETIME2(0) NOT NULL,
    finished_at DATETIME2(0) NULL,
    requested_by VARCHAR(100) NULL,
    version_no_used INT NOT NULL,
    scheduled_for DATETIME2(0) NULL,
    summary_message VARCHAR(MAX) NULL,
    error_code VARCHAR(100) NULL,
    is_overlapped_skip BIT NOT NULL CONSTRAINT DF_PQ_TASK_EXECUTION_is_overlapped_skip DEFAULT (0),
    created_at DATETIME2(0) NOT NULL,
    CONSTRAINT PK_PQ_TASK_EXECUTION PRIMARY KEY (execution_id),
    CONSTRAINT FK_PQ_TASK_EXECUTION_batch FOREIGN KEY (execution_batch_id)
        REFERENCES PQ_TASK_EXECUTION_BATCH(execution_batch_id),
    CONSTRAINT FK_PQ_TASK_EXECUTION_definition FOREIGN KEY (task_definition_id)
        REFERENCES PQ_TASK_DEFINITION(task_definition_id),
    CONSTRAINT FK_PQ_TASK_EXECUTION_process FOREIGN KEY (process_id)
        REFERENCES PQ_TASK_PROCESS(process_id)
);
GO
```

## 4.11 `PQ_TASK_EXECUTION_PARAMETER`

```sql
CREATE TABLE PQ_TASK_EXECUTION_PARAMETER (
    execution_parameter_id BIGINT IDENTITY(1,1) NOT NULL,
    execution_id BIGINT NOT NULL,
    parameter_code VARCHAR(100) NOT NULL,
    parameter_name VARCHAR(200) NULL,
    data_type VARCHAR(30) NOT NULL,
    value_string VARCHAR(MAX) NULL,
    value_int INT NULL,
    value_decimal DECIMAL(18,4) NULL,
    value_bool BIT NULL,
    value_datetime DATETIME2(0) NULL,
    value_json NVARCHAR(MAX) NULL,
    CONSTRAINT PK_PQ_TASK_EXECUTION_PARAMETER PRIMARY KEY (execution_parameter_id),
    CONSTRAINT FK_PQ_TASK_EXECUTION_PARAMETER_execution FOREIGN KEY (execution_id)
        REFERENCES PQ_TASK_EXECUTION(execution_id),
    CONSTRAINT UQ_PQ_TASK_EXECUTION_PARAMETER UNIQUE (execution_id, parameter_code)
);
GO
```

## 4.12 `PQ_TASK_EXECUTION_LOG`

```sql
CREATE TABLE PQ_TASK_EXECUTION_LOG (
    execution_log_id BIGINT IDENTITY(1,1) NOT NULL,
    execution_id BIGINT NOT NULL,
    log_timestamp DATETIME2(0) NOT NULL,
    log_level VARCHAR(20) NOT NULL,
    log_source VARCHAR(200) NULL,
    message_text VARCHAR(MAX) NOT NULL,
    message_code VARCHAR(100) NULL,
    context_json NVARCHAR(MAX) NULL,
    sequence_no INT NOT NULL,
    CONSTRAINT PK_PQ_TASK_EXECUTION_LOG PRIMARY KEY (execution_log_id),
    CONSTRAINT FK_PQ_TASK_EXECUTION_LOG_execution FOREIGN KEY (execution_id)
        REFERENCES PQ_TASK_EXECUTION(execution_id)
);
GO
```

---

## 5. Restricciones lógicas recomendadas (`CHECK`)

## 5.1 Tabla `PQ_TASK_PROCESS_PARAMETER`

```sql
ALTER TABLE PQ_TASK_PROCESS_PARAMETER
ADD CONSTRAINT CK_PQ_TASK_PROCESS_PARAMETER_data_type
CHECK (data_type IN ('STRING','INT','DECIMAL','BOOL','DATETIME'));
GO
```

## 5.2 Tabla `PQ_TASK_DEFINITION`

```sql
ALTER TABLE PQ_TASK_DEFINITION
ADD CONSTRAINT CK_PQ_TASK_DEFINITION_frequency_type
CHECK (frequency_type IN ('HOURLY_INTERVAL','DAILY','WEEKLY','MONTHLY','CUSTOM'));
GO

ALTER TABLE PQ_TASK_DEFINITION
ADD CONSTRAINT CK_PQ_TASK_DEFINITION_last_status_code
CHECK (last_status_code IS NULL OR last_status_code IN (
    'EN_EJECUCION','EXITOSA','EXITOSA_CON_OBSERVACIONES','FALLIDA','OMITIDA','CANCELADA'
));
GO
```

## 5.3 Tabla `PQ_TASK_RUNTIME_STATE`

```sql
ALTER TABLE PQ_TASK_RUNTIME_STATE
ADD CONSTRAINT CK_PQ_TASK_RUNTIME_STATE_data_type
CHECK (state_data_type IN ('STRING','INT','DECIMAL','BOOL','DATETIME','JSON'));
GO
```

## 5.4 Tabla `PQ_TASK_EXECUTION_BATCH`

```sql
ALTER TABLE PQ_TASK_EXECUTION_BATCH
ADD CONSTRAINT CK_PQ_TASK_EXECUTION_BATCH_trigger_type
CHECK (trigger_type IN ('PROGRAMADA','MANUAL'));
GO

ALTER TABLE PQ_TASK_EXECUTION_BATCH
ADD CONSTRAINT CK_PQ_TASK_EXECUTION_BATCH_run_mode
CHECK (run_mode IN ('REAL','SIMULACION'));
GO

ALTER TABLE PQ_TASK_EXECUTION_BATCH
ADD CONSTRAINT CK_PQ_TASK_EXECUTION_BATCH_status_code
CHECK (status_code IN ('EN_EJECUCION','EXITOSA','EXITOSA_CON_OBSERVACIONES','FALLIDA','OMITIDA','CANCELADA'));
GO
```

## 5.5 Tabla `PQ_TASK_EXECUTION`

```sql
ALTER TABLE PQ_TASK_EXECUTION
ADD CONSTRAINT CK_PQ_TASK_EXECUTION_trigger_type
CHECK (trigger_type IN ('PROGRAMADA','MANUAL'));
GO

ALTER TABLE PQ_TASK_EXECUTION
ADD CONSTRAINT CK_PQ_TASK_EXECUTION_run_mode
CHECK (run_mode IN ('REAL','SIMULACION'));
GO

ALTER TABLE PQ_TASK_EXECUTION
ADD CONSTRAINT CK_PQ_TASK_EXECUTION_status_code
CHECK (status_code IN ('EN_EJECUCION','EXITOSA','EXITOSA_CON_OBSERVACIONES','FALLIDA','OMITIDA','CANCELADA'));
GO
```

## 5.6 Tabla `PQ_TASK_EXECUTION_PARAMETER`

```sql
ALTER TABLE PQ_TASK_EXECUTION_PARAMETER
ADD CONSTRAINT CK_PQ_TASK_EXECUTION_PARAMETER_data_type
CHECK (data_type IN ('STRING','INT','DECIMAL','BOOL','DATETIME','JSON'));
GO
```

## 5.7 Tabla `PQ_TASK_EXECUTION_LOG`

```sql
ALTER TABLE PQ_TASK_EXECUTION_LOG
ADD CONSTRAINT CK_PQ_TASK_EXECUTION_LOG_log_level
CHECK (log_level IN ('INFO','WARNING','ERROR'));
GO
```

---

## 6. Índices recomendados

```sql
CREATE INDEX IX_PQ_TASK_PROCESS_module_code
    ON PQ_TASK_PROCESS(module_code);
GO

CREATE INDEX IX_PQ_TASK_PROCESS_is_active
    ON PQ_TASK_PROCESS(is_active);
GO

CREATE INDEX IX_PQ_TASK_PROCESS_PARAMETER_display_order
    ON PQ_TASK_PROCESS_PARAMETER(process_id, display_order);
GO

CREATE INDEX IX_PQ_TASK_DEFINITION_process_id
    ON PQ_TASK_DEFINITION(process_id);
GO

CREATE INDEX IX_PQ_TASK_DEFINITION_is_active
    ON PQ_TASK_DEFINITION(is_active);
GO

CREATE INDEX IX_PQ_TASK_DEFINITION_next_run_at
    ON PQ_TASK_DEFINITION(next_run_at);
GO

CREATE INDEX IX_PQ_TASK_DEFINITION_active_next_run
    ON PQ_TASK_DEFINITION(is_active, next_run_at);
GO

CREATE INDEX IX_PQ_TASK_DEFINITION_COMPANY_company_code
    ON PQ_TASK_DEFINITION_COMPANY(company_code);
GO

CREATE INDEX IX_PQ_TASK_EXECUTION_BATCH_task_definition_id
    ON PQ_TASK_EXECUTION_BATCH(task_definition_id);
GO

CREATE INDEX IX_PQ_TASK_EXECUTION_BATCH_requested_at
    ON PQ_TASK_EXECUTION_BATCH(requested_at);
GO

CREATE INDEX IX_PQ_TASK_EXECUTION_BATCH_status_code
    ON PQ_TASK_EXECUTION_BATCH(status_code);
GO

CREATE INDEX IX_PQ_TASK_EXECUTION_task_definition_id
    ON PQ_TASK_EXECUTION(task_definition_id);
GO

CREATE INDEX IX_PQ_TASK_EXECUTION_company_code
    ON PQ_TASK_EXECUTION(company_code);
GO

CREATE INDEX IX_PQ_TASK_EXECUTION_status_code
    ON PQ_TASK_EXECUTION(status_code);
GO

CREATE INDEX IX_PQ_TASK_EXECUTION_started_at
    ON PQ_TASK_EXECUTION(started_at);
GO

CREATE INDEX IX_PQ_TASK_EXECUTION_task_company_status
    ON PQ_TASK_EXECUTION(task_definition_id, company_code, status_code);
GO

CREATE INDEX IX_PQ_TASK_EXECUTION_LOG_execution_sequence
    ON PQ_TASK_EXECUTION_LOG(execution_id, sequence_no);
GO

CREATE INDEX IX_PQ_TASK_EXECUTION_LOG_execution_level
    ON PQ_TASK_EXECUTION_LOG(execution_id, log_level);
GO

CREATE INDEX IX_PQ_TASK_EXECUTION_LOG_timestamp
    ON PQ_TASK_EXECUTION_LOG(log_timestamp);
GO
```

### Índice filtrado recomendado para solapamiento

```sql
CREATE INDEX IX_PQ_TASK_EXECUTION_en_ejecucion
    ON PQ_TASK_EXECUTION(task_definition_id, company_code, started_at)
    WHERE status_code = 'EN_EJECUCION';
GO
```

---

## 7. Reglas técnicas de uso

### 7.1 Inicio de ejecución

Al comenzar una corrida:

1. crear registro en `PQ_TASK_EXECUTION_BATCH`,
2. crear una `PQ_TASK_EXECUTION` por empresa alcanzada,
3. marcar cada ejecución con `status_code = 'EN_EJECUCION'`,
4. copiar snapshot de parámetros a `PQ_TASK_EXECUTION_PARAMETER`,
5. empezar a escribir logs en `PQ_TASK_EXECUTION_LOG`.

### 7.2 Fin de ejecución

Al finalizar:

- actualizar `status_code`,
- completar `finished_at`,
- guardar `summary_message`,
- actualizar `last_run_at`, `last_status_code` y `next_run_at` en `PQ_TASK_DEFINITION`.

### 7.3 Solapamiento

Si existe ejecución activa de la misma tarea y empresa:

- no lanzar nueva ejecución real,
- registrar ejecución omitida con:
  - `status_code = 'OMITIDA'`
  - `is_overlapped_skip = 1`
- loguear error por solapamiento.

### 7.4 Estado interno técnico

`PQ_TASK_RUNTIME_STATE` debe usarse exclusivamente para variables internas del motor/proceso, por ejemplo:

- último ID procesado,
- última fecha,
- último aviso enviado,
- contadores internos.

No debe usarse para parámetros configurables por usuario.

---

## 8. Consideraciones específicas ya definidas por negocio

Este modelo refleja decisiones ya acordadas:

- una tarea puede agrupar varias empresas;
- cada empresa genera su propia ejecución hija;
- la ejecución manual corre inmediatamente;
- la ejecución manual no altera el calendario automático;
- no hay reintentos automáticos;
- el solapamiento se omite y se loguea como error;
- los logs tienen niveles `INFO`, `WARNING`, `ERROR`;
- debe persistirse `EN_EJECUCION` al iniciar;
- se soporta simulación;
- la mayoría de procesos no requieren parámetros adicionales;
- mails multivalor se guardan como string separado por comas;
- rutas locales del servidor se almacenan como texto parametrizado.

---

## 9. Próximo paso recomendado

Con este documento ya definido, el siguiente paso ideal sería generar la **documentación técnica para Cursor**, incluyendo:

- contratos backend,
- servicios y handlers,
- flujo de scheduler,
- endpoints,
- reglas de validación,
- comportamiento de la consulta,
- ejecución manual y simulación,
- y lineamientos de implementación en Laravel.


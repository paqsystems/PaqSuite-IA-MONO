# Diagrama Mermaid — Modelo de datos de tareas automáticas programadas

```mermaid
erDiagram
    PQ_TASK_PROCESS {
        bigint process_id PK
        varchar process_code UK
        varchar process_name
        varchar module_code
        varchar module_name
        varchar handler_class
        bit is_active
        bit supports_manual_execution
        bit supports_scheduled_execution
        bit supports_simulation
        bit works_by_company
        varchar selection_strategy_auto
        varchar selection_strategy_manual
        varchar idempotency_strategy
        varchar validation_handler
    }

    PQ_TASK_PROCESS_PARAMETER {
        bigint process_parameter_id PK
        bigint process_id FK
        varchar parameter_code
        varchar parameter_name
        varchar data_type
        varchar ui_control_type
        bit is_required
        bit is_user_editable
        bit is_multivalue
        int display_order
    }

    PQ_TASK_PROCESS_STATE_FIELD {
        bigint process_state_field_id PK
        bigint process_id FK
        varchar field_code
        varchar field_name
        varchar data_type
        bit is_persisted
    }

    PQ_TASK_DEFINITION {
        bigint task_definition_id PK
        bigint process_id FK
        varchar task_code UK
        varchar task_title
        bit is_active
        varchar frequency_type
        int frequency_interval
        varchar frequency_days_mask
        date base_date
        time base_time
        datetime next_run_at
        datetime last_run_at
        varchar last_status_code
        bit allow_manual_execution
        bit allow_simulation
        int version_no
    }

    PQ_TASK_DEFINITION_COMPANY {
        bigint task_definition_company_id PK
        bigint task_definition_id FK
        varchar company_code
        varchar company_name
        bit is_active
    }

    PQ_TASK_DEFINITION_PARAMETER {
        bigint task_definition_parameter_id PK
        bigint task_definition_id FK
        bigint process_parameter_id FK
        varchar parameter_code
        varchar value_string
        int value_int
        decimal value_decimal
        bit value_bool
        datetime value_datetime
        nvarchar value_json
    }

    PQ_TASK_DEFINITION_AUDIT {
        bigint task_definition_audit_id PK
        bigint task_definition_id FK
        int version_no
        varchar change_type
        datetime changed_at
        varchar changed_by
        nvarchar snapshot_json
    }

    PQ_TASK_RUNTIME_STATE {
        bigint task_runtime_state_id PK
        bigint task_definition_id FK
        varchar company_code
        varchar state_key
        varchar state_data_type
        varchar value_string
        int value_int
        decimal value_decimal
        bit value_bool
        datetime value_datetime
        nvarchar value_json
        bigint updated_by_execution_id
    }

    PQ_TASK_EXECUTION_BATCH {
        bigint execution_batch_id PK
        bigint task_definition_id FK
        varchar trigger_type
        varchar run_mode
        varchar requested_by
        datetime requested_at
        varchar scheduler_reference
        int version_no_used
        varchar status_code
        datetime started_at
        datetime finished_at
    }

    PQ_TASK_EXECUTION {
        bigint execution_id PK
        bigint execution_batch_id FK
        bigint task_definition_id FK
        bigint process_id FK
        varchar company_code
        varchar trigger_type
        varchar run_mode
        varchar status_code
        datetime started_at
        datetime finished_at
        varchar requested_by
        int version_no_used
        datetime scheduled_for
        varchar error_code
        bit is_overlapped_skip
    }

    PQ_TASK_EXECUTION_PARAMETER {
        bigint execution_parameter_id PK
        bigint execution_id FK
        varchar parameter_code
        varchar parameter_name
        varchar data_type
        varchar value_string
        int value_int
        decimal value_decimal
        bit value_bool
        datetime value_datetime
        nvarchar value_json
    }

    PQ_TASK_EXECUTION_LOG {
        bigint execution_log_id PK
        bigint execution_id FK
        datetime log_timestamp
        varchar log_level
        varchar log_source
        varchar message_code
        int sequence_no
    }

    PQ_TASK_PROCESS ||--o{ PQ_TASK_PROCESS_PARAMETER : defines
    PQ_TASK_PROCESS ||--o{ PQ_TASK_PROCESS_STATE_FIELD : documents
    PQ_TASK_PROCESS ||--o{ PQ_TASK_DEFINITION : templates
    PQ_TASK_PROCESS_PARAMETER ||--o{ PQ_TASK_DEFINITION_PARAMETER : instantiates

    PQ_TASK_DEFINITION ||--o{ PQ_TASK_DEFINITION_COMPANY : applies_to
    PQ_TASK_DEFINITION ||--o{ PQ_TASK_DEFINITION_PARAMETER : configures
    PQ_TASK_DEFINITION ||--o{ PQ_TASK_DEFINITION_AUDIT : versions
    PQ_TASK_DEFINITION ||--o{ PQ_TASK_RUNTIME_STATE : maintains
    PQ_TASK_DEFINITION ||--o{ PQ_TASK_EXECUTION_BATCH : triggers
    PQ_TASK_DEFINITION ||--o{ PQ_TASK_EXECUTION : executes

    PQ_TASK_EXECUTION_BATCH ||--o{ PQ_TASK_EXECUTION : groups
    PQ_TASK_EXECUTION ||--o{ PQ_TASK_EXECUTION_PARAMETER : snapshots
    PQ_TASK_EXECUTION ||--o{ PQ_TASK_EXECUTION_LOG : logs

    PQ_TASK_PROCESS ||--o{ PQ_TASK_EXECUTION : implemented_by
```

## Lectura rápida del diagrama

- `PQ_TASK_PROCESS` representa el catálogo técnico de procesos programables.
- `PQ_TASK_DEFINITION` representa la tarea configurada por el usuario.
- `PQ_TASK_DEFINITION_COMPANY` permite asociar varias empresas a una misma tarea.
- `PQ_TASK_RUNTIME_STATE` separa el estado interno técnico de los parámetros visibles.
- `PQ_TASK_EXECUTION_BATCH` agrupa un disparo general.
- `PQ_TASK_EXECUTION` guarda la corrida concreta por empresa.
- `PQ_TASK_EXECUTION_PARAMETER` congela los parámetros usados en esa corrida.
- `PQ_TASK_EXECUTION_LOG` guarda el detalle cronológico de mensajes.

## Sugerencia de uso

Este archivo sirve para:

- validar visualmente la arquitectura,
- detectar redundancias o faltantes,
- discutir ajustes antes de cerrar SQL definitivo,
- y luego usarlo como apoyo para la documentación técnica de Cursor.

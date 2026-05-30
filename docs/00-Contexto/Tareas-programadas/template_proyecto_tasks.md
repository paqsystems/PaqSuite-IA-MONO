# 🧱 Template Base Proyecto — Laravel + Tasks

## 1. Estructura Recomendada

app/
├── Models/
├── Services/
│ ├── TaskExecutionService.php
│ ├── TaskSchedulerService.php
│ └── TaskLogger.php
├── Processes/
│ ├── Contracts/
│ │ └── TaskProcessInterface.php
│ ├── Base/
│ │ └── BaseTaskProcess.php
│ └── Implementations/
├── Console/
│ └── Kernel.php


---

## 2. Contrato de proceso

```php
interface TaskProcessInterface
{
    public function validate(TaskExecutionContext $context): void;
    public function execute(TaskExecutionContext $context): TaskResult;
}
```

## 3. Base de Proceso

abstract class BaseTaskProcess implements TaskProcessInterface
{
    protected TaskLogger $logger;

    public function __construct(TaskLogger $logger)
    {
        $this->logger = $logger;
    }

    protected function logInfo($msg)
    {
        $this->logger->info($msg);
    }

    protected function logError($msg)
    {
        $this->logger->error($msg);
    }
}

---

## 4. Ejemplo real de Proceso

class AutorizarPedidosProcess extends BaseTaskProcess
{
    public function execute(TaskExecutionContext $context): TaskResult
    {
        $this->logInfo("Inicio proceso");

        if ($context->mode === 'SIMULACION') {
            $this->logInfo("Modo simulación");
        }

        // lógica

        return new TaskResult('EXITOSA', 'Proceso finalizado');
    }
}

---

## 5. Motor de Ejecución

class TaskExecutionService
{
    public function runTask($task, $trigger, $mode)
    {
        // crear batch
        // crear ejecución
        // snapshot parámetros
        // ejecutar proceso
        // guardar logs
        // actualizar estado
    }
}

---

## 6. Logger

class TaskLogger
{
    public function info($msg) {}
    public function warning($msg) {}
    public function error($msg) {}
}

---

## 7. Scheduler 

``` PHP
$schedule->call(function () {
    app(TaskSchedulerService::class)->run();
})->everyMinute();
```

## 8. Scheduler Service

class TaskSchedulerService
{
    public function run()
    {
        // buscar tareas activas
        // evaluar next_run_at
        // ejecutar
    }
}

---

## 9. Reglas técnicas obligatorias
Un solo motor de ejecución
No lógica en controllers
Logs siempre
Snapshot de parámetros
Manejo de errores controlado

---

## 10. Simulación

``` PHP
if ($mode === 'SIMULACION') {
    // no persistir efectos
}
```

---

## 11. Validación estándar
tarea activa
parámetros válidos
empresas definidas
no solapamiento

---

##  12. Uso esperado

Este template sirve para:
- arrancar rápido
- mantener consistencia
- evitar errores estructurales

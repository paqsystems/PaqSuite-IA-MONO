# 📘 Manual de Arquitectura + Onboarding --- Subsistema de Tareas Programadas

## 1. Objetivo

Este documento tiene como objetivo:

-   Alinear al equipo técnico en la arquitectura del subsistema
-   Definir cómo desarrollar nuevos procesos programables
-   Establecer buenas prácticas
-   Facilitar el onboarding de nuevos desarrolladores

------------------------------------------------------------------------

## 2. Visión General del Sistema

El subsistema permite:

-   Automatizar procesos de negocio
-   Ejecutarlos manualmente o por scheduler
-   Registrar resultados, logs e historial

### Componentes principales

1.  Catálogo de procesos
2.  Definición de tareas
3.  Motor de ejecución
4.  Scheduler
5.  Logs e historial

------------------------------------------------------------------------

## 3. Arquitectura General

### Capas

-   Models (Eloquent)
-   Services
-   Process Handlers
-   Execution Engine
-   Scheduler

------------------------------------------------------------------------

## 4. Principios de Diseño

### Separación de responsabilidades

-   Configuración ≠ Ejecución ≠ Estado ≠ Logs

### Idempotencia

Cada proceso debe definir su comportamiento ante re-ejecución

### Trazabilidad

Toda ejecución debe quedar registrada

### Reutilización

Un proceso sirve para: - ejecución automática - ejecución manual

------------------------------------------------------------------------

## 5. Flujo de Ejecución

1.  Scheduler o usuario dispara tarea
2.  Se crea un batch
3.  Se crean ejecuciones por empresa
4.  Se guardan parámetros (snapshot)
5.  Se ejecuta proceso
6.  Se registran logs
7.  Se actualiza estado final

------------------------------------------------------------------------

## 6. Estructura del Código

### Ubicación sugerida

    app/
     ├── Models/
     ├── Services/
     ├── Processes/
     ├── Jobs/
     └── Console/

------------------------------------------------------------------------

## 7. Cómo Crear un Nuevo Proceso Programable

### Paso 1: Registrar en catálogo

-   process_code
-   nombre
-   parámetros

### Paso 2: Crear clase

``` php
class MiProceso implements TaskProcessInterface
```

### Paso 3: Implementar lógica

``` php
public function execute($context)
```

### Paso 4: Registrar parámetros

### Paso 5: Probar manualmente

### Paso 6: Activar scheduler

------------------------------------------------------------------------

## 8. Buenas Prácticas

-   No escribir lógica en controllers
-   Usar servicios
-   Loguear siempre
-   Validar antes de ejecutar
-   No mezclar estado interno con parámetros

------------------------------------------------------------------------

## 9. Logs

Siempre usar:

-   INFO → flujo normal
-   WARNING → observaciones
-   ERROR → fallos

------------------------------------------------------------------------

## 10. Simulación

-   No ejecutar efectos reales
-   Sí generar logs
-   Usar para testing

------------------------------------------------------------------------

## 11. Errores Comunes

❌ No registrar logs\
❌ Duplicar lógica manual/automática\
❌ No controlar solapamiento\
❌ No validar parámetros

------------------------------------------------------------------------

## 12. Onboarding rápido (nuevo desarrollador)

1.  Leer este documento
2.  Revisar modelo de datos
3.  Ejecutar una tarea manual
4.  Crear un proceso simple de prueba
5.  Revisar logs e historial

------------------------------------------------------------------------

## 13. Checklist antes de subir código

-   [ ] Proceso registrado
-   [ ] Parámetros definidos
-   [ ] Logs implementados
-   [ ] Validaciones hechas
-   [ ] Simulación funciona
-   [ ] No hay efectos duplicados

------------------------------------------------------------------------

## 14. Futuro del sistema

-   Integración con colas
-   Escalabilidad horizontal
-   Monitoreo en tiempo real
-   Dashboard operativo

------------------------------------------------------------------------

## ✔ Resultado

Este manual permite que cualquier desarrollador:

-   entienda la arquitectura
-   cree nuevos procesos
-   mantenga el sistema
-   trabaje de forma consistente

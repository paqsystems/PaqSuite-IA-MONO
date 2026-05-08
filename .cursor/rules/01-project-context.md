## alwaysApply: true

# description: Contexto del Proyecto – ERP PaqSuite MONOEMPRESA

## Propósito

Este documento proporciona el contexto general del proyecto para que el agente IA comprenda el dominio, objetivos y restricciones.

## Contexto General

Este proyecto es una **plataforma ERP MONOEMPRESA** en evolución.

Actualmente incluye:

* módulos operativos administrativos,
* arquitectura web moderna,
* autenticación y permisos,
* funcionalidades reutilizables para aplicaciones empresariales.

## Objetivo General

* Registrar y gestionar operaciones empresariales de forma simple y eficiente
* Centralizar procesos administrativos
* Mantener una arquitectura reutilizable y escalable
* Entregar flujos E2E completos y funcionales

## Qué SÍ es este proyecto

* Una aplicación ERP MONOEMPRESA
* Una aplicación web con backend (Laravel), frontend (React), tests y deploy
* Un sistema con autenticación, roles y permisos
* Una arquitectura modular reutilizable

## Qué NO es este proyecto

* No es un sistema multiempresa
* No utiliza Dictionary DB
* No utiliza tenant por header
* No separa bases por empresa
* No utiliza contexto empresa activo

## Usuarios

* Existe un rol Supervisor con acceso total.
* El resto de los alcances y limitaciones se definen mediante permisos.

## Flujo E2E Prioritario

**Login → Operación principal → Visualización / Consulta**

Todo el desarrollo debe soportar este flujo.

## Entidades Clave

a nivel base operativa única:

* **Usuario** (tabla USERS)
* roles
* permisos
* opciones de menú
* parámetros generales
* tareas programadas
* procesos automáticos

## Principios de Diseño

* Simplicidad sobre sofisticación
* No sobre–ingeniería
* Validaciones claras
* Trazabilidad del trabajo
* Testing enfocado en el flujo principal

## Alcance Técnico

* Arquitectura por capas (Controller → Service → Domain → Repository)
* API REST (`/api/v1/`)
* Autenticación Sanctum
* Tests unitarios, integración y E2E

## Referencias

* `docs/00-contexto/00-contexto-global-erp.md`
* `docs/01-arquitectura/README.md`

# 📘 Playbook Operativo — Subsistema de Tareas Programadas

## 1. Objetivo
Este playbook define cómo trabajamos en el equipo para:

- desarrollar tareas automatizadas
- mantener consistencia técnica
- evitar errores operativos
- facilitar soporte y evolución

Equipo objetivo: 1–3 desarrolladores.

---

## 2. Principio central del equipo

👉 "Todo proceso automatizado debe ser predecible, trazable y seguro"

Esto implica:
- siempre loguear
- evitar duplicaciones
- poder ejecutar manualmente
- poder simular antes de ejecutar

---

## 3. Roles (livianos)

### 👨‍💻 Responsable funcional/técnico (vos)
- define reglas de negocio
- valida diseño
- aprueba implementación

### 👨‍💻 Desarrollador
- implementa procesos
- respeta contratos
- documenta lo mínimo necesario

---

## 4. Flujo de trabajo REAL

### Paso 1 — Definición funcional
Se documenta:
- qué hace el proceso
- qué datos toma
- qué resultado produce
- qué parámetros necesita

---

### Paso 2 — Diseño técnico
Se define:
- proceso programable
- parámetros
- validaciones
- idempotencia

---

### Paso 3 — Implementación
Se desarrolla:
- clase del proceso
- integración con motor
- logs

---

### Paso 4 — Testing (CRÍTICO)
Siempre probar:

✔ ejecución manual  
✔ ejecución automática  
✔ simulación  
✔ re-ejecución  

---

### Paso 5 — Deploy
Se activa:
- tarea programada
- monitoreo inicial

---

### Paso 6 — Monitoreo
Primeros días:
- revisar logs
- validar comportamiento real

---

## 5. Reglas obligatorias del equipo

### 🔴 Regla 1 — Logs obligatorios
Todo proceso debe registrar:
- inicio
- pasos importantes
- errores
- resultado final

---

### 🔴 Regla 2 — No duplicar lógica
Manual y automático deben compartir lógica.

---

### 🔴 Regla 3 — Idempotencia
Siempre definir:
👉 qué pasa si se ejecuta 2 veces

---

### 🔴 Regla 4 — No solapamiento
Nunca permitir doble ejecución simultánea.

---

### 🔴 Regla 5 — Simulación cuando aplique
Procesos críticos deben poder simularse.

---

## 6. Manejo de incidentes

Cuando algo falla:

1. Ver ejecución
2. Leer logs
3. Reproducir en simulación
4. Corregir
5. volver a ejecutar manual

---

## 7. Checklist antes de cerrar un desarrollo

- [ ] Logs completos
- [ ] Validaciones hechas
- [ ] No duplica efectos
- [ ] Simulación funciona
- [ ] Manual y automático alineados
- [ ] Parámetros claros

---

## 8. Errores típicos a evitar

❌ No loguear  
❌ Mezclar estado interno con parámetros  
❌ Hardcodear valores  
❌ No controlar duplicación  
❌ No probar simulación  

---

## 9. Regla de oro

👉 Si no podés explicar qué hace el proceso en 2 minutos,
el diseño está mal.

---

## 10. Evolución futura

Cuando el sistema crezca:
- colas (queues)
- workers
- dashboard en tiempo real

Pero NO ahora.

---

## ✔ Resultado esperado

Con este playbook:
- cualquier dev puede trabajar ordenado
- el sistema es mantenible
- se minimizan errores operativos
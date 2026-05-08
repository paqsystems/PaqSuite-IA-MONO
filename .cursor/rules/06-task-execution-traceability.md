# Regla: Trazabilidad de ejecución de Tareas (TR) MONOEMPRESA

## Objetivo

Cada vez que Cursor implemente una TR (TR-*.md), debe dejar evidencia clara de:

* qué archivos creó/modificó,
* qué comandos ejecutó,
* qué decisiones técnicas tomó,
* y qué quedó pendiente.

---

## Reglas obligatorias

### 1)

Al finalizar la implementación de una TR, actualizar el mismo archivo TR agregando:

* `## Archivos creados/modificados`
* `## Comandos ejecutados`
* `## Notas y decisiones`
* `## Pendientes / follow-ups`

---

### 2)

En `Archivos creados/modificados`, listar paths relativos agrupados por:

* Backend
* Frontend
* Database/Migrations/Seeds
* Tests
* Docs/CI

---

### 3)

En `Comandos ejecutados`, listar:

* comando ejecutado,
* objetivo,
* resultado relevante si aplica.

---

### 4)

Si durante la implementación se modifica:

* estructura,
* rutas,
* nombres,
* arquitectura,

documentar el cambio en:

```text
## Notas y decisiones
```

y actualizar documentación relacionada.

---

### 5)

Nunca borrar ejecuciones anteriores.

Si se reimplementa una TR:

* agregar subsección con timestamp,
* mantener historial.

---

### 6)

Actualizar el campo:

```text
Estado = Pendiente de Revisión
```

al finalizar implementación y trazabilidad.

---

### 7)

Si la TR modifica:

* listados,
* grillas,
* exportaciones,
* endpoints pesados,
* consultas complejas,

documentar alineación con:

```text
.cursor/rules/multi/14-obtencion-datos-performance.md ó .cursor/rules/mono/14-obtencion-datos-performance.md
```

y registrar:

* métricas,
* optimizaciones,
* logs `perf.*`,
* decisiones de performance.

# Regla: Optimización de obtención de datos (MONOEMPRESA)

## Objetivo

Evitar degradación de performance en:

* backend
* API
* SQL
* frontend
* DevExtreme

---

## Principios

1. medir antes de optimizar
2. un cambio a la vez
3. identificar cuello real
4. documentar mejoras

---

## 1) Backend Laravel

### Reglas generales

* evitar N+1
* usar eager loading
* usar paginación real
* evitar consultas repetidas
* limitar payloads innecesarios

---

## 2) ORM y SQL

### Permitido

* Eloquent
* Query Builder
* SQL parametrizado
* vistas
* stored procedures

---

## 3) Paginación

Implementar:

```text id="gh0t9w"
page
page_size
total
total_pages
```

---

## 4) Índices

Evaluar índices cuando:

* existan filtros frecuentes
* exista ordenamiento repetitivo
* existan consultas pesadas

---

## 5) Payloads

No enviar:

* grafos completos
* relaciones innecesarias
* datos redundantes

Sí enviar:

* labels
* códigos
* descripciones
* datos necesarios para UI

---

## 6) API

Mantener contrato homogéneo.

Respetar:

* envelope
* paginación
* sort
* dir

---

## 7) Frontend React

### Reglas

* evitar loops en useEffect
* evitar doble fetch
* reutilizar apiFetch
* reutilizar stores
* evitar renders innecesarios

---

## 8) DevExtreme

Usar:

* DataGridDX
* remote operations
* loadPanel
* paginación nativa

Evitar:

* recarga completa innecesaria
* múltiples fetch simultáneos
* grillas custom paralelas

---

## 9) Observabilidad

Cuando aplique registrar:

```text id="zjlwmf"
query_time_ms
transform_time_ms
total_time_ms
payload_bytes
```

---

## 10) Orden sugerido de optimización

1. middleware repetitivo
2. metadata repetida
3. paginación
4. SQL / índices
5. payload

---

## Persistencia MONOEMPRESA

Existe:

```text id="1l29fc"
una única base operativa
```

No existe:

* company db
* dictionary db
* tenant
* X-Company-Id

---

## Referencias

* `.cursor/rules/multi/02-backend-policy.md` ó `.cursor/rules/mono/02-backend-policy.md`
* `.cursor/rules/multi/03-api-contract.md` ó `.cursor/rules/mono/03-api-contract.md`
* `.cursor/rules/multi/08-devextreme-grid-standards.md` ó `.cursor/rules/mono/08-devextreme-grid-standards.md`

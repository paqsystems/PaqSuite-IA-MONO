# Parámetros generales por módulo

Documento de reingeniería del proceso transversal de configuración en `PQ_PARAMETROS_GRAL`.

## Objetivo

Permitir que usuarios con permiso de configuración adapten el comportamiento del sistema **por módulo** sin modificar código: solo editan valores de parámetros ya definidos en seeds/deploy.

---

## Invocación desde el menú

- Existe un **proceso general** reutilizable de mantenimiento de parámetros.
- Cada módulo que tenga parámetros expone un **ítem de menú** cuyo `pq_menus.procedimiento` coincide con el nombre clave del módulo (campo `Programa` en datos), por ejemplo `PartesProduccion`.
- Al abrir desde ese ítem, la pantalla muestra **solo** filas cuyo `Programa` coincide con ese procedimiento (comparación canónica; backend puede ignorar mayúsculas/minúsculas para evitar desajustes con SQL Server).

Cada módulo documenta en su propia spec las claves, tipos, valores por defecto, y los textos (o claves i18n) de **CAPTION** y **TOOLTIP** por parámetro; este documento define el **marco común** de la pantalla.

---

## Base de datos

Tabla **`PQ_PARAMETROS_GRAL`** en la **Company DB** del despliegue (en MONO, una sola instancia por instalación).

| Campo relevante | Uso |
|-----------------|-----|
| `Programa` | Módulo; mismo literal que `procedimiento` del menú |
| `Clave` | Identificador del parámetro |
| `tipo_valor` | Define qué columna `Valor_*` aplica |
| `CAPTION` | Etiqueta breve del parámetro en el listado (traducida; ver `idioma.md`) |
| `TOOLTIP` | Ayuda por fila (traducida; ver `idioma.md`) |
| `Valor_String`, `Valor_Text`, `Valor_Int`, `Valor_DateTime`, `Valor_Bool`, `Valor_Decimal` | Almacenamiento según tipo |

Registros iniciales: **seeds** en deploy (como `PQ_MENUS`). La pantalla **no permite alta ni baja** de filas; solo edición de valores.

---

## Interfaz de usuario

### Parte 1 — Listado (solo lectura)

- Columnas: **Clave**, descripción breve desde **`CAPTION`** (mostrado en el idioma activo), **valor mostrado como texto homogéneo** (todos los tipos convertidos a string legible).
- Booleanos: etiquetas localizadas (ej. Sí/No), no `true`/`false` crudos; NULL se interpreta como negativo.
- Ayuda por fila desde **`TOOLTIP`** (traducido; si viene vacío, sin tooltip).
- `CAPTION` y `TOOLTIP` se cargan con cada registro; su traducción es obligatoria según `idioma.md` (alcance sin excepción).
- Implementación alineada a **grilla/listado DevExtreme** del resto del sistema.

### Parte 2 — Edición por fila

- Botón **Editar** en cada fila.
- Abre **modal** o vista de edición con **un solo control** acorde a `tipo_valor` (checkbox, entero, decimal, texto, texto largo, fecha/hora).
- No usar cajas de texto genéricas para todos los tipos en el listado.
- **Guardar** valida tipo y rangos; **Cancelar** descarta cambios.

Textos de marco de pantalla bajo claves `parametrosGral.*` en locales del frontend.

---

## Acceso a datos y API (MONO)

En **monoempresa** no aplica resolución de base por empresa activa ni el header **`X-Company-Id`**: existe una única Company DB del despliegue. Lectura y escritura de `PQ_PARAMETROS_GRAL` se realizan siempre contra esa base. El usuario debe tener permiso de configuración del módulo (ver sección *Permisos*).

<!--
Versión MULTI — Multi-tenant y API (no aplicar en MONO)

- Lectura y escritura deben usar la Company DB resuelta por X-Company-Id (empresa activa).
- El cliente envía siempre ese header al cargar y guardar.
- Usuario con una sola empresa: el cliente puede fijarla automáticamente como activa.
- Sin header correcto, la lista puede aparecer vacía aunque existan datos en otra base.
- Normalizar pq_empresa.NombreBD (trim) en middleware si hubiera espacios.

El usuario debe tener permiso sobre la empresa activa.
-->

---

## Permisos

Solo usuarios con permiso de **configuración** del módulo (según rol y menú) acceden al proceso.

---

## Relación con otros documentos mono

| Tema | Documento |
|------|-----------|
| Menú y `procedimiento` | `Menu-general.md` |
| Permisos de acceso | `seguridad-permisos.md` |
| Empresa activa (solo MULTI) | `menu-avatar.md` |
| Estándar visual | `apariencia-temas.md`, `grillas.md` |

---

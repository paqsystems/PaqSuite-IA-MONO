# 15 — Host, base de datos y branding por cliente (subdominio)

## 0) Objetivo

Definir la convención **obligatoria** para:

* derivar el **nombre de la base de datos** a partir del **proyecto** y del **host** en producción, y de un modelo fijo en desarrollo;
* usar el mismo identificador de **cliente** para ubicar la **imagen del logo** en carpeta `images/` y exponerla en la UI.

## 1) Alcance

Apunta a proyectos PaqSystems desplegados con URLs de tipo subdominio bajo `paqsystems.com`, con Laravel y frontend en el mismo criterio de identificación por host.

### 1.1) Proyectos MONO (frontend + backend por proyecto)

En productos **MONO**, la resolución de host, redirect y SQL **no** usa la tabla de la sección 5 como única verdad: ver **`docs/_base/resolucion-host-cliente-sql-mono.md`**.

- **Entrada:** `{cliente}.{proyecto}.paqsystems.com`
- **Redirect a:** `frontend.{proyecto}.paqsystems.com` (conservando `{cliente}`)
- **API:** `backend.{proyecto}.paqsystems.com` con header **`X-Paq-Cliente`**
- **SQL:** fila en `EMPRESAS_CONEXION` por `{proyecto}` + `{cliente}`
- **Desarrollo:** `cliente = demo` (salvo otro slug en OpenSpec del producto)

Un artefacto de frontend y un artefacto de backend **por `{proyecto}`**; **no** deploy por cliente ni selector de empresa en UI (MULTI ERP).

Esta regla **15** aplica en MONO **logo** (mismo `{cliente}`) y **conectividad SQL privada** (sección 6). Índice: `docs/_mono/README-host-y-tenant.md`.

### 1.2) Proyectos MULTI o modelo clásico por host

Las secciones 2–5 siguientes describen resolución directa `paqsystems_{proyecto}_{cliente}` desde el host sin redirect MONO (multi-empresa ERP / productos que no usen el patrón `frontend.{proyecto}` / `backend.{proyecto}`).

---

## 2) Forma del host en producción

Las invocaciones en **producción** deben usar la sintaxis:

```text
https://{cliente}.{proyecto}.paqsystems.com
```

Donde:

* **`{cliente}`** — primer subdominio (segmento más a la izquierda del nombre de host antes de `{proyecto}`). Sirve como **identificador estable del cliente** en todo el sistema (base de datos, assets de marca).
* **`{proyecto}`** — segundo subdominio, identifica la **instancia vertical / producto** del ecosistema (nombre del proyecto en convención de despliegue).

El **primer nodo útil del host para multicliente es `{cliente}`**; combinado con `{proyecto}` determina la base de datos aplicable en esa petición.

---

## 3) Convención de nombre de base de datos

### 3.1 Producción

Base de datos a utilizar:

```text
paqsystems_{proyecto}_{cliente}
```

* `{proyecto}` y `{cliente}` deben resolver a **tokens seguros para identificadores SQL** (por convención: minúsculas, sin espacios, caracteres `[a-z0-9_]`, acordados con operación/hosting).

### 3.2 Desarrollo

En **desarrollo**, la base de datos es **siempre**:

```text
paqsystems_{proyecto}_demo
```

* `{cliente}` **no** se toma del host en desarrollo: el sufijo fijo **`demo`** sustituye al cliente para un único esquema de trabajo local o de integración según equipo.
* `{proyecto}` debe obtenerse del **nombre de proyecto configurado para el repo** (variable de configuración `.env` o equivalente acordado), para que todas las aplicaciones locales del mismo producto compartan esta regla sin depender de un host igual al de producción.

### 3.3 Implementación esperada

* **Backend:** resolver `cliente` y `proyecto` desde el header `Host` (o infraestructura equivalente detrás del proxy), **solo** cuando el host cumpla el patrón esperado para producción; en entorno local/staging usar la regla **`_demo`** y el `proyecto` configurado.
* **No confiar solo en headers sin validar forma del host:** rechazar patrones inconsistentes cuando la seguridad o el modelo de datos lo requieran.
* Mantener coherentes migrations y seeds entre `paqsystems_{proyecto}_{cliente}` (prod) y `paqsystems_{proyecto}_demo` (dev).

Referencias relacionadas: `.cursor/rules/05-data-access-orm-sql.md`, `.cursor/rules/07-versioning-and-deploy.md`.

---

## 4) Logo corporativo (`images/` + cliente)

El identificador **`{cliente}`** del host de producción (mismo slug usado en el nombre de la base tras el proyecto) debe usarse para localizar los recursos del **logo de la empresa** dentro de una subcarpeta bajo **`images/`** (ruta exacta dentro de `images/` puede ser `images/{cliente}/` o `images/brands/{cliente}/`; lo importante es que **todo el código que resuelva el logo use el mismo esquema** acordado en el repositorio).

### 4.1 Ubicación en la interfaz

* **Login:** logo a la **izquierda** del área de inicio de sesión (orden visual claro cuando hay layout centrado — el branding del cliente debe anclarse al lado izquierdo del bloque principal o según shell del proyecto, manteniendo “izquierda del login”).
* **Aplicación en ejecución:** logo en la parte **izquierda del frame superior** (cabecera / top bar persistente durante el uso).

### 4.2 Resolución ante fallbacks

Si el archivo no existe, definir comportamiento degradado único por producto (imagen neutra o logo PaqSystems) sin romper layout; documentar esa decisión una sola vez en el proyecto.

---

## 5) Resumen operativo

| Contexto           | Fuente `{proyecto}`      | Fuente `{cliente}`        | Base de datos                        |
|--------------------|---------------------------|---------------------------|---------------------------------------|
| Producción (`*.paqsystems.com` patrón acordado) | Segundo subdominio        | Primer subdominio         | `paqsystems_{proyecto}_{cliente}`     |
| Desarrollo         | Config repo / `.env`      | Sufijo literal `demo` (no usar subdominio de cliente) | `paqsystems_{proyecto}_demo`          |

| Uso branding       | Clave consistente |
|-------------------|-------------------|
| Logo empresa      | Mismo `{cliente}` que en BD producción |

---

## 6) Conectividad SQL privada (Tailscale) — MONO y multi-cliente

Los SQL de cada `{cliente}` **no** se publican en Internet ni se abren puertos del firewall del cliente hacia AWS. El backend del deploy único se conecta por **red privada Tailscale**.

Documentación operativa extendida: `docs/_base/_Tailscape.md`.  
Patrón ERP equivalente (tabla `EMPRESAS_CONEXION`, header tenant): `docs/_base/regla-cursor-multitenant-paqsuite.md`.

### 6.1 Arquitectura de conexión

```text
Usuario → {cliente}.{proyecto}.paqsystems.com → frontend.{proyecto}.paqsystems.com
       → Frontend (AWS) → backend.{proyecto}.paqsystems.com
       → lookup (proyecto, cliente) en registro de conexiones
       → HOST_TAILSCALE → SQL Server del cliente (sin IP pública)
```

**Prohibido:** `Frontend → SQL Server` directo. Siempre `Frontend → Backend → SQL`.

### 6.2 Registro de asociación (tabla central)

En el SQL **central del deploy** (o config cifrada equivalente) mantener filas por `(proyecto, cliente)`. Nombre sugerido alineado al ERP: **`EMPRESAS_CONEXION`** (o `CLIENTES_CONEXION` si el producto lo renombra).

| Campo | Uso |
|-------|-----|
| `CODIGO_TENANT` / `cliente` | Mismo slug que `{cliente}` del host y logo (`acme`, `demo`) |
| `proyecto` | Slug del producto (`pedidosweb`, …) |
| `DOMINIO` | Host de entrada esperado (ej. `acme.pedidosweb.paqsystems.com`) |
| `HOST_TAILSCALE` | Hostname Tailscale del SQL (ej. `acme.tailnet.ts.net`) — **no** IP pública del cliente |
| `SQL_DATABASE` | Nombre de base (ej. `paqsystems_pedidosweb_acme`) |
| `SQL_INSTANCE` | Instancia nombrada (opcional) |
| `SQL_USER` | Usuario dedicado (ej. `paqsuite_api`), **nunca** `sa` |
| `SQL_PASSWORD_ENCRYPTED` | Secreto cifrado; no en repo ni `.env` commiteado |
| `ACTIVO` | Habilita o no el cliente |

Ejemplo mínimo de esquema (adaptar tipos al motor central):

```sql
-- Referencia; motor y prefijos según producto
CREATE TABLE EMPRESAS_CONEXION (
    ID              INT PRIMARY KEY IDENTITY,
    CODIGO_TENANT   VARCHAR(50) NOT NULL,
    PROYECTO        VARCHAR(50) NOT NULL,
    DOMINIO         VARCHAR(150) NOT NULL,
    HOST_TAILSCALE  VARCHAR(200) NOT NULL,
    SQL_DATABASE    VARCHAR(100) NOT NULL,
    SQL_USER        VARCHAR(100) NOT NULL,
    SQL_PASSWORD_ENCRYPTED VARBINARY(MAX) NOT NULL,
    ACTIVO          BIT NOT NULL DEFAULT 1,
    UNIQUE (PROYECTO, CODIGO_TENANT)
);
```

### 6.3 Identificación del cliente en API (header)

Convención **MONO / productos nuevos:**

```http
X-Paq-Cliente: acme
```

Equivalente conceptual al `X-Tenant` del documento multitenant ERP. El proxy que redirige `{cliente}.{proyecto}` → `frontend.{proyecto}` puede **inyectar** `X-Paq-Cliente` hacia `backend.{proyecto}`.

**Backend:** leer header, validar fila en `EMPRESAS_CONEXION` (`ACTIVO = 1`), construir connection string dinámico hacia `HOST_TAILSCALE`. **No** confiar solo en el frontend: validar host/origen cuando aplique.

**Frontend:** centralizar resolución de `cliente` (helper / interceptor Axios); en desarrollo `cliente = demo` (`localhost`, IP LAN, `VITE_TENANT_OVERRIDE` opcional con prioridad documentada en el producto).

### 6.4 Desarrollo (`demo`)

- `cliente` / tenant forzado = **`demo`**.
- Fila `demo` en `EMPRESAS_CONEXION` apunta al SQL de desarrollo (Tailscale o red local acordada).
- Base sugerida: `paqsystems_{proyecto}_demo`.

### 6.5 Cache, logging y seguridad

| Tema | Regla |
|------|--------|
| Cache | Cachear filas de `EMPRESAS_CONEXION` en backend (TTL sugerido **5 min**); invalidar al actualizar registro. |
| Logging | Registrar `cliente`, `proyecto`, hostname, endpoint, usuario, fallback `demo`, errores SQL/Tailscale (sin passwords). |
| SQL | Usuario mínimo privilegio (`paqsuite_api`); no `sa`. |
| Red | ACLs Tailscale por cliente; cada cliente con nodo SQL en su tailnet. |

### 6.6 DNS (producción MONO)

- Wildcard o CNAME: `{cliente}.{proyecto}` → redirect a `frontend.{proyecto}`; API en `backend.{proyecto}` (ver `resolucion-host-cliente-sql-mono.md`).

---

## 7) Referencias

* `docs/_base/resolucion-host-cliente-sql-mono.md` — deploy único, redirect, header `cliente`
* `docs/_base/regla-cursor-multitenant-paqsuite.md` — tenant, `X-Tenant`, flujo ERP
* `docs/_base/_Tailscape.md` — implementación Tailscale
* `.cursor/rules/02-backend-policy.md`
* `.cursor/rules/05-data-access-orm-sql.md`
* `.cursor/rules/07-versioning-and-deploy.md`

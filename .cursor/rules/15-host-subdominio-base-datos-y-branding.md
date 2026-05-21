# 15 — Host, base de datos y branding por cliente (subdominio)

## 0) Objetivo

Definir la convención **obligatoria** para:

* derivar el **nombre de la base de datos** a partir del **proyecto** y del **host** en producción, y de un modelo fijo en desarrollo;
* usar el mismo identificador de **cliente** para ubicar la **imagen del logo** en carpeta `images/` y exponerla en la UI.

## 1) Alcance

Apunta a proyectos PaqSystems desplegados con URLs de tipo subdominio bajo `paqsystems.com`, con Laravel y frontend en el mismo criterio de identificación por host.

### 1.1) Proyectos MONO (deploy único)

En productos **MONO**, la resolución de host, redirect y SQL **no** usa la tabla de la sección 5 como única verdad: ver **`docs/_base/resolucion-host-cliente-sql-mono.md`**:

- Deploy canónico: `demo.{proyecto}.paqsystems.com`
- Entrada usuario: `{cliente}.{proyecto}.paqsystems.com` → redirect a `demo` + contexto `{cliente}`
- SQL: registro de asociación por `cliente` (host, instancia, BD, credenciales)
- Desarrollo: `cliente = demo`

Esta regla **15** sigue aplicando en MONO la parte de **logo** (`{cliente}` = mismo slug que en la asociación SQL).

### 1.2) Proyectos MULTI o modelo clásico por host

Las secciones 2–5 siguientes describen resolución directa `paqsystems_{proyecto}_{cliente}` desde el host sin redirect a `demo` (multi-empresa ERP / productos que no usen deploy único MONO).

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

## 6) Referencias

* `.cursor/rules/02-backend-policy.md`
* `.cursor/rules/05-data-access-orm-sql.md`
* `.cursor/rules/07-versioning-and-deploy.md`

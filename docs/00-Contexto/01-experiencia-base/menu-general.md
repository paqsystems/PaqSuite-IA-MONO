# Menú general de navegación

Documento de reingeniería del menú de **procesos y módulos** (sidebar), distinto del menú avatar del usuario. Reúne lineamientos de seguridad y de navegación general del framework.

## Rol en la aplicación

El menú general es la puerta de entrada a los procesos. Cada ítem habilitado para el usuario proviene de **`pq_menus`**, filtrado por roles y atributos definidos en `../02-acceso-y-seguridad/seguridad-permisos.md`.

Tras el login, el frontend obtiene el menú desde la **API del usuario**; no debe estar hardcodeado por permisos en el cliente.

---

## Fuente de verdad: seed versionado (`pq_menus`)

El árbol de menú del sistema se define en el repositorio y se sincroniza a base mediante seed idempotente:

- Archivo versionado, por ejemplo `PQ_MENUS.seed.json`.
- Script o seeder que hace upsert en `pq_menus` sin duplicar al ejecutar dos veces.
- Integrado en deploy o post-migración.
- Orden determinístico por padre y `orden`, con constraints de integridad del árbol.

**Fuera de alcance inicial:** ABM de edición del menú en producción. Los cambios estructurales se realizan por seed y deploy.

### Campos relevantes de `pq_menus`

| Campo | Uso |
|-------|-----|
| `id` | Identificador del nodo |
| `text` | Etiqueta visible |
| `Idparent` / `parentId` | Jerarquía |
| `order` / `orden` | Orden entre hermanos |
| `procedimiento` | Clave del proceso |
| `routeName` | Ruta SPA; puede ser null en nodos agrupadores |
| `enabled` | `1` visible, `0` oculto |
| `tipo`, `expanded`, `estructura` | Comportamiento de árbol según diseño |

Convención: `routeName` debe coincidir con el router del frontend.

## Reglas conceptuales del árbol

- El menú representa la estructura funcional visible del sistema.
- Un nodo puede ser agrupador o navegable.
- La jerarquía del árbol debe ser estable entre entornos, porque también ordena la experiencia del usuario y no solo la persistencia de datos.
- El orden entre hermanos debe ser determinístico y significativo para la operatoria.

---

## API de menú del usuario

Endpoint protegido, por ejemplo `GET /api/v1/user/menu`:

- Usuario autenticado.
- En MONO, empresa única implícita.
- Filtra `pq_menus` según `Pq_Permiso`, `Pq_Rol` y `PQ_RolAtributo`.
- Devuelve árbol ordenado con `id`, `text`, `parentId`, `orden`, `routeName` y/o `procedimiento`.
- Si falla o queda vacío, el frontend debe poder mostrar un menú mínimo sin romper layout.

La resolución detallada entre menú y permisos se describe en `../02-acceso-y-seguridad/menu-y-autorizacion.md`.

---

## Tres controles del menú en el header (MONO)

Norma transversal para **todos los proyectos mono**: tres acciones **independientes** en la barra superior, a la **izquierda** (antes o junto al logo). Son preferencias de **presentación** del usuario; **no** filtran permisos ni sustituyen la API de menú.

| # | Control | Icono sugerido | Qué afecta | Qué **no** afecta |
|---|---------|----------------|------------|-------------------|
| 1 | **Mostrar / ocultar sidebar** | Hamburguesa (☰) | Visibilidad del **panel** lateral (ancho del frame o drawer en móvil). El área principal se redimensiona. | Ramas del árbol, permisos, ítems autorizados |
| 2 | **Expandir / contraer árbol** | Doble chevron / “expand all” | Estado **expandido/contraído** de **todas** las ramas del TreeView, con el sidebar visible | Visibilidad del panel (control 1), permisos |
| 3 | **Vista del menú** | Lista ↔ árbol | Modo de **listado** del árbol autorizado (véase modos abajo) | Permisos; no oculta procesos no autorizados |

### Cuándo se muestran

| Fase | Menú lateral de procesos | Controles 1–3 |
|------|---------------------------|---------------|
| Login | No | No (o solo branding; sin árbol de procesos) |
| Post-login (shell) | Sí, en sidebar | Sí, en header |

### Reglas de interacción

1. **Independencia:** cada control tiene su propio estado; activar uno no implica el estado de los otros.
2. **Sidebar oculto (control 1):** los controles 2 y 3 siguen en el header; al volver a mostrar el sidebar se restauran el último estado de expansión y modo de vista.
3. **Ruta activa:** en modo `allBranches`, al navegar a un proceso los **padres del ítem activo** quedan expandidos aunque el árbol estuviera contraído globalmente (el usuario debe ver dónde está).
4. **Móvil:** el control 1 suele abrir/cerrar un **drawer**; 2 y 3 aplican al árbol dentro del drawer o permanecen en header según diseño del producto.
5. **Seguridad:** cambiar vista o expansión **nunca** debe mostrar un proceso que la API no autorizó.

### Alcance de persistencia (obligatorio)

El estado de los **tres controles** debe recordarse **por usuario** o **por terminal/navegador** (implementación más simple). En ambos casos rige lo mismo:

| Permitido | Prohibido |
|-----------|-----------|
| Preferencia **del usuario autenticado** (p. ej. columna en `users` o claves scoped por `userId`) | Preferencia **por empresa** / tenant / `IDEmpresa` / `X-Company-Id` |
| Preferencia **del navegador** en ese equipo (`localStorage` / `sessionStorage`), idealmente con **sufijo de `userId`** para no mezclar usuarios en la misma PC | Parámetro **global** del sistema (`PQ_PARAMETROS_GRAL`, config de despliegue, default único para todos) |
| Defaults de producto solo al **primer uso** de un usuario/terminal sin valor guardado | Compartir el mismo estado entre **distintos usuarios** que inician sesión |

**Reglas:**

1. **Nunca por empresa:** en MONO no aplica selector de empresa; en MULTI futuro, **cambiar empresa activa no altera** sidebar visible, expansión del árbol ni modo de vista (no persistir ni leer por empresa).
2. **Nunca global:** administración, seed ni parámetros generales **no** definen estos tres estados para todos los usuarios.
3. **Por usuario (recomendado si hay sync multi-dispositivo):** persistir en preferencias del usuario (`users` o `GET/PATCH /users/me/preferences`), keyed por `userId`, sin `empresaId`.
4. **Por terminal (MVP habitual):** `localStorage` con claves `{appId}.{userId}.menu.*` (o equivalente). Al **logout**, no reutilizar claves de otro usuario en el mismo navegador.
5. **Logout / otro usuario:** al iniciar sesión con otro usuario, cargar **su** estado guardado o defaults; no heredar el del usuario anterior.

### Persistencia (convención técnica)

Implementación MVP habitual: `localStorage` **por terminal**, scoped por usuario:

| Clave sugerida | Tipo | Default desktop | Descripción |
|----------------|------|-----------------|-------------|
| `{appId}.{userId}.menu.sidebarVisible` | `boolean` | `true` | Panel lateral visible |
| `{appId}.{userId}.menu.treeExpanded` | `boolean` | `true` | Todas las ramas expandidas (`true`) o contraídas (`false`) |
| `{appId}.{userId}.menu.displayMode` | `allBranches` \| `operationalOnly` | `allBranches` | Modo de vista del árbol |

Alternativa equivalente: persistir los tres valores en **preferencias de usuario** en servidor (misma regla: por `userId`, sin dimensión empresa). Elegir una vía por producto; no mezclar scopes (empresa/global).

**Prohibido explícitamente:** `PQ_Empresa`, `PQ_PARAMETROS_GRAL`, flags de tenant o “default de instalación” como fuente de estos estados.

### Modos de vista (control 3)

#### `allBranches` — Todas las ramas

- Renderiza el árbol **como lo entrega la API** (agrupadores + procesos hijos).
- Muestra nodos **agrupadores** (carpetas sin ruta) e ítems **operativos** (procesos navegables).

#### `operationalOnly` — Solo opciones operativas

- Lista únicamente nodos **operativos** (abren un proceso en el área principal).
- Los **agrupadores** sin ruta **no** aparecen como fila; sus hijos autorizados se muestran en lista **plana**, ordenados por `orden` / `order` del payload (mismo orden relativo que en el árbol).
- Si un nodo tiene **ruta y hijos**, se trata como operativo y navegable.

**Clasificación de nodos (seed / API):**

| Tipo | Regla | Campo opcional API |
|------|--------|-------------------|
| Agrupador | Sin `routeName` / `routePath`, puede tener hijos | `nodeType: group` |
| Operativo | Con `routeName` / `routePath` hacia un proceso | `nodeType: process` |

Si la API no envía `nodeType`, el frontend infiere: `!routePath && children.length > 0` ⇒ agrupador; con `routePath` ⇒ operativo.

En despliegues con menú **plano** (solo hojas operativas), ambos modos pueden verse iguales hasta que el seed incluya agrupadores; el control 3 debe implementarse igual para paridad con ERP legacy (PaqSystems).

### Implementación frontend (referencia)

- Controles: componente `MenuToolbarControls` (o equivalente) montado en el **header** del shell; lógica acoplada al sidebar / TreeView (DevExtreme).
- Control 2: `expandAll()` / `collapseAll()` del TreeView, o estado centralizado de `expanded` por nodo.
- Control 3: transformación del árbol en cliente (`flattenOperationalNodes`) **después** de recibir la respuesta autorizada; sin llamadas API adicionales.
- Accesibilidad: `aria-label`, tooltip i18n y foco por teclado en cada botón.
- Tests: `data-testid` sugeridos `menuToggleSidebar`, `menuToggleExpandAll`, `menuToggleDisplayMode`.

### Ubicación en el shell

Detalle de zonas del header: `shell-layout.md` (misma carpeta).

---

## Sidebar dinámico (frontend)

El sidebar consume la API al montar el layout principal:

- Sin lógica hardcodeada de permisos.
- Respeta jerarquía por `parentId` y `orden`.
- Cada ítem navegable usa `routeName`.
- Implementa los **tres controles del header** descritos arriba (sidebar, expandir/contraer árbol, vista operativa).
- Mantiene overlay en móvil y preferencia de apertura en pestaña definida en `navegacion-pestanas.md`.
- Resalta el ítem activo por coincidencia más específica de ruta.
- Mantiene expandidos los padres del ítem activo.
- El clic para navegar no debe interferir con expandir o colapsar nodos.
- Los textos visibles surgen de `text`; si se usa i18n por claves, debe respetarse el idioma activo.

Ítems como **Inicio** y **Perfil** pueden venir del seed o resolverse como mínimo de frontend, pero la decisión debe quedar formalizada en specs.

## Reglas de experiencia de usuario

- El usuario debe entender claramente dónde está parado dentro del árbol de navegación.
- Un cambio de permisos debe reflejarse en el menú visible sin requerir redefinir el frontend por perfil.
- Las opciones visibles deben responder a lenguaje funcional y no a nombres técnicos.
- El menú no debe duplicar acciones personales del avatar ni acciones internas propias de la toolbar del proceso.

---

## Iconos y presentación

- Cada módulo puede tener icono distintivo.
- Los nodos intermedios pueden usar iconos opcionales.
- Los procesos operativos pueden diferenciarse visualmente según tipo: ABM, carga, informe, importación o exportación.

---

## Relación con procesos

- Los procesos de parámetros generales reutilizan `procedimiento = Programa` de `PQ_PARAMETROS_GRAL`; ver `../04-configuracion-global/parametros-generales.md`.
- Los mantenimientos de seguridad cuelgan del mismo esquema de menú; ver `../02-acceso-y-seguridad/administracion-seguridad.md`.
- `procedimiento` también se usa para identificar layouts de grilla y exportaciones; ver `../03-ui-transversal/grillas.md`.

## Derivaciones esperables

Este documento debería permitir regenerar:

- HUs de seed y versionado del menú,
- API de menú del usuario,
- sidebar dinámico,
- **tres controles de presentación** en header (MONO),
- iconografía y jerarquía funcional,
- relación entre `routeName`, `procedimiento` y procesos del sistema.

---

## MONO vs MULTI

| Aspecto | MONO | MULTI |
|---------|------|-------|
| Recarga menú al cambiar empresa | No aplica | Sí, al cambiar empresa activa |
| API menú | Rol en empresa única | Rol según empresa activa |
| Estado 3 controles header (sidebar / expandir / vista) | Por **usuario** o **terminal** | Igual: **nunca** por empresa ni global |

Referencia ampliada: `../05-variantes-y-alcance/mono-vs-multi-referencias.md`.

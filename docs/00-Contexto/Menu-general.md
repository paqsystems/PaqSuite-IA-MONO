# Menú general de navegación

Documento de reingeniería del menú de **procesos y módulos** (sidebar), distinto del menú avatar del usuario. Épica Seguridad + generalidades de navegación.

## Rol en la aplicación

El menú general es la puerta de entrada a los procesos. Cada ítem habilitado para el usuario proviene de **`pq_menus`**, filtrado por **roles y atributos** (`seguridad-permisos.md`).

Tras el login, el frontend obtiene el menú desde la **API del usuario**; no debe estar hardcodeado por permisos en el cliente.

---

## Fuente de verdad: seed versionado (`pq_menus`)

El árbol de menú del sistema se define en el **repositorio** y se sincroniza a base mediante seed **idempotente**:

- Archivo versionado (ej. `PQ_MENUS.seed.json` o `PQ_MENUS.seed.v2.json`).
- Script/seeder que hace upsert en `pq_menus` sin duplicar al ejecutar dos veces.
- Integrado en deploy / post-migración.
- Orden determinístico por padre (`parent` + `order`); constraints de integridad del árbol.

**Fuera de alcance inicial:** ABM en UI para editar el menú en producción (cambios vía seed y deploy).

### Campos relevantes de `pq_menus`

| Campo | Uso |
|-------|-----|
| `id` | Identificador del nodo |
| `text` | Etiqueta visible (puede enlazar con i18n) |
| `Idparent` / `parentId` | Jerarquía |
| `order` / `orden` | Orden entre hermanos |
| `procedimiento` | Clave del proceso (grillas, parámetros, rutas legacy) |
| `routeName` | Ruta SPA (ej. `/admin/usuarios`, `/perfil`); nullable en nodos solo agrupadores |
| `enabled` | `1` = visible en menú del usuario; `0` = excluido |
| `tipo`, `expanded`, `estructura` | Comportamiento de árbol según diseño |

Convención: `routeName` debe coincidir con el **router del frontend**. Opciones de administración de seguridad llevan `enabled = 1` y `routeName` definido en el seed.

---

## API de menú del usuario

Endpoint protegido (ej. `GET /api/v1/user/menu`):

- Usuario autenticado; en MONO empresa única implícita; en MULTI requiere empresa activa.
- Filtra `pq_menus` según `Pq_Permiso` + `Pq_Rol` + `PQ_RolAtributo` (ver `seguridad-permisos.md`).
- Devuelve árbol ordenado con `id`, `text`, `parentId`, `orden`, `routeName` (y/o `procedimiento`).
- Fallo o lista vacía: menú mínimo (Inicio, Perfil) o mensaje sin romper layout.

---

## Sidebar dinámico (frontend)

El componente **sidebar** consume la API al montar el layout principal:

- **Sin menú hardcodeado** de permisos (`esAdmin` / flags locales no deciden secciones).
- Respeta jerarquía: secciones (nivel 0), subnodos (nivel 1, 2…) según `parentId` y `orden`.
- Cada ítem navegable usa `routeName` para construir la ruta.
- Comportamientos existentes: colapsar sidebar, preferencia *Abrir en nueva pestaña* (`menu-avatar.md`), overlay en móvil.
- **Ítem activo:** resaltar la ruta que mejor coincide con `location.pathname` (match más específico en rutas anidadas).
- **Rama expandida:** al cargar o cambiar ruta, mantener expandidos los padres del ítem activo.
- Clic en enlace no debe interferir con expandir/colapsar del árbol (`stopPropagation` donde corresponda).
- Textos desde `text` de la API; i18n si existe clave en locales.
- Estilos e indentación por nivel (`sidebar-item-level0`, `level1`, …) y tema del usuario (`apariencia-temas.md`).

Ítems fijos **Inicio** y **Perfil** pueden venir del seed (`routeName` `/` y `/perfil`) o definirse como mínimo en frontend si no están en `pq_menus` (documentar decisión en spec).

### Navegación al hacer clic

- Preferencia *misma pestaña* → SPA / TabPanel de procesos.
- Preferencia *nueva pestaña* → `target="_blank"` o equivalente con sesión preservada.

---

## Iconos en cada opción

- Rama principal (módulo): icono distintivo del módulo.
- Nodos intermedios: iconos opcionales.
- Ítems que invocan procesos: icono según tipo — ABM, carga, informe/consulta, importación, exportación.

---

## Expansión y contracción del menú

- Control para modo **siempre expandido** (útil con pocas opciones habilitadas).
- Modo **siempre contraído**: solo permanece expandida la rama del proceso en ejecución.

---

## Procesos especiales invocados desde menú

- **Parámetros generales** de un módulo: `procedimiento` = `Programa` en `PQ_PARAMETROS_GRAL` (`parametros-generales.md`).
- Mantenimientos de seguridad: usuarios, roles, permisos (`administracion-seguridad.md`).
- Resto de módulos de negocio según sus seeds de menú.

---

## Relación con grillas y permisos

- `procedimiento` enlaza con `proceso` en layouts de grilla y exportación (`grillas.md`).
- `Permiso_Baja` y demás atributos gobiernan acciones en la pantalla destino (ej. eliminar en ABM).

---

## MONO vs MULTI

| Aspecto | MONO | MULTI |
|---------|------|-------|
| Recarga menú al cambiar empresa | No aplica | Sí, al cambiar empresa activa |
| API menú | Rol en empresa única | Rol según `X-Company-Id` |

---

## Historias de usuario de origen

`001-Seguridad`: HU-015 (seed menú), HU-016 (API menú), HU-017 (sidebar dinámico), HU-018 (`routeName` / `enabled` en seed). `hu-anteriores` HU-003 (nueva pestaña).

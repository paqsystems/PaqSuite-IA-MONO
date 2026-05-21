# Idioma e internacionalización (i18n)

Documento de reingeniería: cómo el usuario elige el idioma de la interfaz y cómo el sistema lo aplica.

## Objetivo

Permitir que cada usuario use la aplicación en su idioma preferido, con cambio inmediato de textos y formatos, persistido entre dispositivos y sesiones.

El idioma afecta **labels, mensajes y formatos** de fecha/número según locale. **No** traduce automáticamente datos de negocio (nombres de clientes, descripciones de artículos, etc.).

---

## Idiomas soportados (catálogo inicial)

Solo se ofrecen idiomas con archivos de traducción completos en frontend (`react-i18next` o equivalente):

| Código | Idioma (etiqueta) | Indicador visual sugerido |
|--------|-------------------|---------------------------|
| `es` | Español | Bandera Argentina 🇦🇷 |
| `en` | English | Bandera Reino Unido 🇬🇧 |
| `pt` | Português | Bandera Brasil 🇧🇷 |
| `fr` | Français | Bandera Francia 🇫🇷 |
| `it` | Italiano | Bandera Italia 🇮🇹 |

Ampliar el catálogo implica agregar locale JSON y registrar el idioma en configuración i18n.

---

## Selector de idioma

### Ubicación

- **Pantalla de login** (antes de autenticarse).
- **Header del layout principal** tras el login.
- **No** dentro del menú avatar; es control **independiente** en la barra superior.

### Forma de interacción

Puede implementarse como:

- Grupo de botones con banderas,
- Dropdown con bandera + nombre del idioma en su propio idioma, o
- Lista con ambos.

La opción activa debe distinguirse visualmente (resaltado, check, etc.).

### Aplicación del cambio

Al elegir idioma, la UI se actualiza **sin recargar la página completa**. Todos los textos que usan la función de traducción (`t()`) deben reflejar el cambio al instante.

---

## Idioma inicial al cargar la app

| Situación | Regla |
|-----------|--------|
| Usuario con `users.locale` definido | Usar ese valor |
| `users.locale` NULL o vacío | Usar `navigator.language` del navegador |
| Idioma del navegador no soportado | Fallback a **español** (o idioma por defecto de producto) |

---

## Persistencia

| Estado del usuario | Dónde se guarda |
|--------------------|----------------|
| No autenticado | Temporal en cliente (ej. localStorage); al login se envía al backend |
| Autenticado | `users.locale` en Dictionary DB (varchar ~10, ej. `es`, `en`) |

La preferencia es **por usuario**, no por empresa.

API: lectura/actualización vía payload de login, perfil o endpoint de preferencias.

---

## Activación técnica en frontend

- i18n configurado globalmente.
- **Todos** los textos de UI deben usar claves de traducción (no literales fijos en componentes).
- Archivos por idioma bajo la carpeta de locales del proyecto (ej. `es.json`, `en.json`, …).
- Formatos de fecha y número alineados al locale activo.

Correos transaccionales (ej. recuperación de contraseña) deben enviarse en el idioma activo en el momento del envío.

---

## Alcance obligatorio de i18n (sin excepción)

**Regla:** todo texto que el sistema **genera o muestra en la interfaz** (y no es dato de negocio almacenado por el usuario) debe provenir de claves de traducción (`t()` / locales), nunca de literales fijos en código. Esto incluye captions, placeholders, tooltips, mensajes y etiquetas de acciones.

### Controles y formularios (DevExtreme y shell)

- `caption`, `label`, `placeholder` y textos de estado vacío de cualquier control (ej. *Seleccione una opción*, *Ingrese mail*).
- `hint`, `tooltip` y textos de ayuda junto a controles.
- Botones, toggles, títulos de popup/modal y pies de diálogo (Aceptar, Cancelar, etc.).
- Mensajes de validación en cliente y textos de error devueltos por la API cuando se muestran al usuario (mapear códigos a claves i18n).

### Mensajes transitorios y bloqueantes

- Modales de advertencia, confirmación, error y éxito.
- Toasts, snackbars y notificaciones.
- Indicadores de carga o procesamiento (*Cargando…*, *Guardando…*, etc.).

### Navegación y menús

- Ítems del **menú general** (sidebar): etiquetas visibles; si el texto viene del seed `pq_menus`, debe resolverse por clave i18n o convención documentada equivalente.
- Opciones del **menú avatar** (Perfil, Apariencia, Cerrar sesión, etc.).
- Títulos de sección, breadcrumbs y pestañas del layout cuando sean texto de sistema.

### Login y contraseña

- Pantalla de login: captions, placeholders y enlaces (*¿Olvidaste tu contraseña?*).
- Flujos de recuperación y cambio de contraseña: todos los controles y mensajes en pantalla.
- **Correos** de recuperación o notificación: cuerpo y asunto en el idioma activo al momento del envío.

### Parámetros generales

- Marco de la pantalla: títulos, botones *Editar* / *Guardar*, mensajes de validación (`parametrosGral.*`).
- Atributos **`CAPTION`** y **`TOOLTIP`** de cada fila en `PQ_PARAMETROS_GRAL` (cargados con el parámetro): forman parte de lo que se **expone** en el listado y en la ayuda por fila; **deben traducirse sin excepción** al idioma activo, igual que cualquier otro texto de UI.
  - **CAPTION** → etiqueta breve visible en el listado (columna de descripción).
  - **TOOLTIP** → texto de ayuda al pasar el cursor o equivalente en la fila.
- Al cambiar el idioma en el selector, esos textos deben actualizarse junto con el resto de la interfaz (sin depender de un solo idioma fijo en base de datos).
- Implementación habitual: en seed, `CAPTION` / `TOOLTIP` almacenan **claves** i18n (ej. `parametrosGral.{Programa}.{Clave}.caption`) y la UI los resuelve con `t()`; los archivos de locale (`es`, `en`, `pt`, `fr`, `it`) contienen el texto en cada idioma para cada parámetro del módulo. Si se usa otro mecanismo, debe cumplir el mismo resultado: usuario ve caption y tooltip en su idioma.

### Grillas y pivots (DevExtreme)

**Grilla:** títulos de columnas definidos por la app, tooltips de columna, textos del panel de filtro, agrupación y totalización al pie, nombres de funciones de total (suma, cuenta, promedio, etc.), toolbar (exportar, layouts, agregar), mensajes de grilla vacía o sin datos.

**Pivot:** captions de zonas fila/columna/valor, totalizadores, lista de funciones disponibles, asistente (*wizard*) de armado, tooltips y mensajes del control.

### Lo que no entra en esta obligación

- Datos de negocio: nombres de clientes, descripciones de artículos, textos libres cargados por usuarios en Company DB.
- Valores de parámetros (`Valor_*`) mostrados como dato configurado, no como etiqueta de UI.


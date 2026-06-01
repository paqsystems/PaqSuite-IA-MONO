# Idioma e internacionalización (i18n)

Documento de reingeniería de cómo el usuario elige el idioma de la interfaz y cómo el sistema lo aplica.

## Objetivo

Permitir que cada usuario use la aplicación en su idioma preferido, con cambio inmediato de textos y formatos, persistido entre dispositivos y sesiones.

El idioma afecta labels, mensajes y formatos de fecha o número. No traduce automáticamente datos de negocio cargados por usuarios.

---

## Idiomas soportados

Solo se ofrecen idiomas con archivos de traducción completos en frontend:

| Código | Idioma | Indicador visual sugerido |
|--------|--------|---------------------------|
| `es` | Español | Bandera argentina |
| `en` | English | Bandera Reino Unido |
| `pt` | Português | Bandera Brasil |
| `fr` | Français | Bandera Francia |
| `it` | Italiano | Bandera Italia |

Ampliar el catálogo implica agregar locales y registrarlos en la configuración i18n.

---

## Selector de idioma

### Ubicación

- Pantalla de login.
- Header del layout principal post-login.
- No forma parte del menú avatar.

### Interacción

Puede implementarse como grupo de banderas, dropdown o lista con nombre nativo del idioma. La opción activa debe distinguirse visualmente.

### Aplicación del cambio

Al elegir idioma, la UI se actualiza sin recarga completa. Todo texto resuelto por claves de traducción debe reflejar el cambio al instante.

---

## Idioma inicial

| Situación | Regla |
|-----------|--------|
| Usuario con `users.locale` definido | Usar ese valor |
| `users.locale` vacío | Usar `navigator.language` |
| Idioma no soportado | Fallback al idioma por defecto del producto |

---

## Persistencia

| Estado del usuario | Dónde se guarda |
|--------------------|----------------|
| No autenticado | Temporal en cliente |
| Autenticado | `users.locale` en Dictionary DB |

La preferencia es por usuario, no por empresa.

---

## Regla transversal de alcance

Todo texto que el sistema genera o muestra en la interfaz, y que no es dato de negocio ingresado por el usuario, debe provenir de claves de traducción.

Esto incluye:

- captions, placeholders, hints y tooltips,
- botones, toggles y títulos de popup,
- validaciones, errores y toasts,
- ítems del menú general y menú avatar,
- textos de login, recuperación y mail de link a cambio de contraseña
- captions y tooltips de parámetros generales,
- toolbar, mensajes vacíos y totalizadores de grillas y pivots.
- apariencias de temas DevExtreme.

No entra en esta obligación el dato de negocio en sí, como nombres de clientes, descripciones libres o valores cargados por usuarios.

---

## Relación con otros temas

- Shell: `shell-layout.md`
- Menú avatar: `menu-avatar.md`
- Login y sesión: `../02-acceso-y-seguridad/login-y-sesion.md`
- Parámetros generales: `../04-configuracion-global/parametros-generales.md`
- Grillas: `../03-ui-transversal/grillas.md`
- Pivots: `../03-ui-transversal/pivots.md`

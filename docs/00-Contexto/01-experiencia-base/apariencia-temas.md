# Apariencia, temas DevExtreme y UI homogénea

Documento de reingeniería del look and feel del sistema y del estándar visual **full DevExtreme**.

## Principio de producto

La interfaz web debe ser homogénea: shell, formularios, modales, grillas y controles de catálogo usan componentes y theming nativos de DevExtreme. Se evita introducir paletas CSS paralelas que compitan con el tema activo.

Al cambiar la apariencia, toda la superficie visible debe reflejarla de inmediato, sin islas visuales con colores fijos ajenos al tema elegido.

---

## Apariencia en MONO

En monoempresa no existe configuración de tema por tenant ni por administración de empresas. La estética la define cada usuario desde el menú avatar.

Esta es la preferencia distintiva MONO frente a MULTI.

### Dónde se elige

- Opción **Apariencia** en el menú avatar.
- Presentación sugerida: listbox o dropdown equivalente con todas las apariencias disponibles.
- El cambio se aplica al instante a shell, grillas, formularios y modales.

### Alcance del cambio

Un único tema activo afecta toda la UI web del producto para ese usuario. No depende de empresa activa ni de pantallas de administración.

### Lista cerrada de temas

Solo se ofrecen temas precompilados de DevExtreme:

| Valor técnico | Descripción legible |
|---------------|---------------------|
| `generic.light` | Generic Claro |
| `generic.dark` | Generic Oscuro |
| `generic.light.compact` | Generic Claro Compacto |
| `generic.dark.compact` | Generic Oscuro Compacto |
| `material.blue.light` / `.dark` | Material Blue |
| `material.teal.light` / `.dark` | Material Teal |
| `material.purple.light` / `.dark` | Material Purple |
| `material.orange.light` / `.dark` | Material Orange |
| `fluent.blue.light` / `.dark` | Fluent Blue |
| `fluent.saas.light` / `.dark` | Fluent SaaS |

### Persistencia

- Preferencia por usuario.
- Campo previsto: `users.theme`.
- Valor nulo o inválido: fallback al tema por defecto del producto.
- Debe conservarse entre sesiones, navegadores y equipos.

---

## Estándar UI full DevExtreme

### Alcance

- Shell principal.
- Formularios de alta y edición.
- Popups y modales.
- Botones primarios y secundarios.
- Selectores y listas de catálogo.
- Grillas y pivots.

### Reglas

- Nuevas pantallas y refactors usan controles DevExtreme para estas superficies.
- El CSS custom de color se limita a branding mínimo o ajustes puntuales.
- No deben existir variables globales que pisen la paleta del tema activo.
- Las excepciones deben quedar justificadas en specs o tareas.

---

## Fuera de alcance en la fase MONO actual

- ThemeBuilder o temas arbitrarios por cliente.
- Tema configurado en administración de empresas.
- Apariencia distinta por empresa dentro del mismo despliegue.

## Referencia MULTI

En MULTI el tema suele resolverse por empresa activa y no por preferencia personal del avatar. Ver `../05-variantes-y-alcance/mono-vs-multi-referencias.md`.

## Relación con otros temas

- Menú avatar: `menu-avatar.md`
- Shell principal: `shell-layout.md`
- Grillas: `../03-ui-transversal/grillas.md`
- Pivots: `../03-ui-transversal/pivots.md`

# Apariencia, temas DevExtreme y UI homogénea

Documento de reingeniería: look & feel del sistema y estándar visual *full DevExtreme*.

## Principio de producto

La interfaz web debe ser **homogénea**: shell (header, sidebar, footer), formularios, modales, grillas y controles de catálogo usan componentes y theming **nativos de DevExtreme**. Evitar paletas CSS custom (`--paq-*` u otras) que compitan con el tema activo.

Al cambiar la apariencia, **toda** la superficie visible debe reflejarla de inmediato, sin “islas” con colores fijos ajenos al tema elegido.

---

## Apariencia en versión MONO (alcance actual)

En **monoempresa** no hay configuración de tema por tenant ni por administración de empresas. La estética del sistema la define **cada usuario** desde el menú avatar.

### Dónde se elige

- Opción **Apariencia** en el menú de usuario (avatar). Ver `menu-avatar.md`.
- Presentación sugerida: **listbox** (o dropdown equivalente DevExtreme) con todas las apariencias disponibles.
- Al seleccionar una opción, el cambio se aplica **al instante** a toda la aplicación (shell, grillas, formularios, modales).

### Alcance del cambio

Un único tema activo afecta **toda** la UI web del producto para ese usuario en esa sesión. No depende de empresa activa ni de pantallas de administración.

### Lista cerrada de temas

Solo se ofrecen temas **precompilados** de DevExtreme (sin ThemeBuilder ni valores arbitrarios):

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

- Preferencia **por usuario** (análoga a `locale` o `menu_abrir_nueva_pestana`).
- Campo previsto en Dictionary DB: `users.theme` (varchar ~50, nullable).
- NULL o valor inválido → tema por defecto del producto (ej. `generic.light` o `material.blue.light`).
- Debe conservarse entre sesiones, navegadores y equipos (persistencia server-side vía login o API de preferencias).

### Carga en runtime

1. Tras login (o al restaurar sesión), cargar `theme` del usuario desde backend.
2. Sustituir el `<link>` del CSS DevExtreme en `<head>` por `devextreme/dist/css/dx.{theme}.css`.
3. Evitar *flash* de estilos: aplicar tema antes de montar el layout principal cuando sea posible.
4. Si el CSS del tema no existe o falla la carga → fallback al tema por defecto y registro de advertencia.

Validación backend: solo valores de la lista cerrada o null.

---

## Estándar UI full DevExtreme

### Alcance de homogeneización

- Shell principal (header, sidebar, footer, contenedores).
- Formularios de alta/edición.
- Popups y modales.
- Botones de acción primaria/secundaria.
- Selectores y listas de catálogo.
- Grillas y pivots (ver `grillas.md`, `pivots.md`).

### Reglas

- Nuevas pantallas y refactors usan controles DevExtreme para lo anterior.
- CSS custom de color limitado a branding mínimo (logos, espaciados puntuales).
- No variables globales de color que pisen la paleta del tema activo.
- Excepciones al estándar deben quedar justificadas y trazadas en specs/tareas.

### Migración

La transición desde componentes legacy se planifica por módulos con tareas de regresión visual y funcional (E2E).

### Compatibilidad

El listado del selector de apariencia debe corresponder a los archivos CSS DevExtreme instalados en el build frontend.

---

## Fuera de alcance (fase MONO actual)

- ThemeBuilder o temas con logo/colores corporativos custom.
- Tema configurado en administración de empresas o por tenant.
- Apariencia distinta por empresa dentro del mismo despliegue mono.

---

## Versión MULTI (referencia futura, no aplicar en MONO)

<!--
Versión MULTI — Apariencia por empresa (derivado de HU-005 en proyecto multi)

- Cada empresa (tenant) puede tener un tema en `PQ_Empresa.Theme`.
- Se configura en administración de empresas, no en menú avatar.
- Al cambiar empresa activa, se carga el tema de esa empresa.
- Ver historias de usuario multiempresa al redefinir lineamientos MULTI.
-->

--
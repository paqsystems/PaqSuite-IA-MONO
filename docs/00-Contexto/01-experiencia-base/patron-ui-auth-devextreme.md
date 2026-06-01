# Patrón UI Auth DevExtreme (MONO)

**Estética y tokens compartidos (BASE):** [`docs/_base/00-modelo-estetica-ui-base.md`](../../../_base/00-modelo-estetica-ui-base.md) — gradiente de marca, card, CTA, breakpoints y variables `--app-shell-*` en shell.

## Objetivo

Definir un contrato reusable para pantallas públicas/autenticadas de credenciales, idioma y apariencia, de modo que futuras TR de proyectos MONO no vuelvan a reabrir decisiones ya cerradas de UX, i18n, DevExtreme y testabilidad.

## Alcance

Aplica a:

- login público;
- `forgot-password` / `reset-password`;
- cambio de contraseña autenticado (`/change-password`);
- selector de idioma;
- selector de apariencia / temas;
- cualquier otra superficie auth similar que reutilice shell, popup, formularios o catálogos DX.

## Reglas base

### 1. Controles

- La UI final debe usar **DevExtreme** cuando exista equivalente razonable (`TextBox`, `Button`, `SelectBox`, `Popup`, `List`, etc.).
- No dejar controles HTML nativos finales (`<input>`, `<button>`, `<select>`, radios, modales custom) si el caso ya está cubierto por DX.

### 2. i18n obligatorio

- Todo texto visible debe salir del locale activo.
- Incluye: títulos, labels, placeholders, ayudas, botones, mensajes, nombres de temas, textos de popup y variantes de error/éxito.
- No hardcodear literales de UI en JSX o CSS.

### 3. Licencia DevExtreme

- `VITE_DEVEXTREME_LICENSE` debe existir en `frontend/.env` o `frontend/.env.local` reales.
- `frontend/.env.example` es solo plantilla.
- Cada cambio de licencia requiere reiniciar Vite.
- Verificación mínima: abrir una pantalla pública y confirmar ausencia del banner `For evaluation purposes only`.

### 4. Patrones UX cerrados

- Login / forgot / reset: card centrada sobre gradiente, selector de idioma integrado y estilo visual consistente entre pantallas públicas.
- Reset password: reutilizar el mismo patrón visual de `forgot-password` con `TextBox` password, CTA primario DX y link secundario de retorno al login dentro de la card.
- Change password: ventana/card modal-like, con gate `firstLogin` y textos 100% i18n.
- Apariencia:
  - `Aplicar` = preview sin cerrar.
  - `Confirmar` = persistir y cerrar.
  - `Cancelar` o `X` = revertir preview no confirmado.
  - Los botones deben seguir visibles después de cada preview.
  - El shell autenticado (header, sidebar, footer, avatar-menu y overlays propios) debe heredar una paleta derivada del **tema DevExtreme activo**, no limitarse a un único esquema genérico `light/dark`.
  - En temas oscuros debe preservarse contraste legible en textos, íconos, selección y hover; no aceptar menús con texto apenas visible.

### 5. Responsive

- Los títulos de popup no deben truncarse por idioma.
- Las acciones deben permanecer visibles dentro de la ventana y aceptar wrap si el texto crece.
- Las listas largas deben conservar selección y una navegación usable aunque el catálogo sea amplio.

### 6. Testabilidad

- Preservar `data-testid` públicos y estables.
- No acoplar pruebas al DOM interno que genera DevExtreme.
- Cada slice reusable debe tener al menos una verificación E2E en un locale distinto de `es`.

## data-testid canónicos

- Login: `login-form`, `login-submit`, `login-forgot-password`
- Forgot password: `localeSelectorForgotPassword`, `forgot-password-form`, `forgotPasswordEmail`, `forgotPasswordSubmit`, `forgotPasswordBackToLogin`
- Reset password: `localeSelectorResetPassword`, `reset-password-form`, `resetPasswordNew`, `resetPasswordConfirm`, `resetPasswordSubmit`, `resetPasswordBackToLogin`
- Change password: `changePasswordCurrent`, `changePasswordNew`, `changePasswordConfirm`, `changePasswordSubmit`, `changePasswordError`, `first-login-gate`
- Apariencia: `themeOption-{themeKey}`, `themeCancelButton`, `themeApplyButton`, `themeConfirmButton`, `themeCurrentValue`

## Consumo desde TR

Toda TR futura que toque estas superficies debe referenciar este documento y declarar explícitamente si:

- lo cumple sin cambios; o
- extiende el patrón con una variación puntual justificada.

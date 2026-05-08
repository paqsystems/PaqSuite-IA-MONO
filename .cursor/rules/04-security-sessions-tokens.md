# 02 — Seguridad (Sanctum) MONOEMPRESA — obligatorio, OWASP-aligned

## 0) Modelo de seguridad MONOEMPRESA

El ERP opera sobre una única base operativa.

* No existe tenant.
* No existe `X-Company-Id`.
* No existe Dictionary DB.
* Usuarios, roles y permisos residen en la misma base operativa.

## Regla

Nunca confiar únicamente en que un usuario esté autenticado.

Toda acción debe validar:

* autenticación,
* permisos,
* acceso funcional correspondiente.

---

## 1) Estándares adoptados

* OWASP API Security Top 10
* OWASP ASVS
* NIST 800-63
* Seguridad por diseño

---

## 2) Autenticación con Laravel Sanctum

### Modalidades

* Tokens personales
* SPA con cookies

### Regla del proyecto

* Todas las APIs requieren autenticación salvo endpoints públicos explícitos.
* Toda request autenticada debe poseer usuario válido.
* Los tokens deben poder revocarse.

### Requisitos mínimos

* Política de expiración de tokens.
* Abilities/scopes mínimos.
* Nunca confiar en frontend para permisos.

---

## 2.1) Gestión de Contraseñas

### Regla fundamental

Todas las contraseñas deben almacenarse hasheadas.

### Algoritmo

* bcrypt
* configuración estándar Laravel

### Implementación obligatoria

#### Crear/actualizar contraseña

```php id="y24y8d"
$passwordHash = Hash::make($password);
```

#### Validar contraseña

```php id="k3h5z4"
Hash::check($password, $user->password_hash)
```

### Prohibido

* texto plano
* md5
* sha1
* sha256 para passwords

### Requisitos mínimos

* mínimo 8 caracteres
* validación obligatoria backend

---

## 3) Autorización

* Policies/Gates obligatorios.
* Nunca autorizar solo por login.
* Validar permisos funcionales.

---

## 4) Validación y sanitización

* Validar antes de persistir.
* Limitar tamaños.
* Sanitizar HTML/markdown.

---

## 5) Anti-inyección

* Nunca concatenar SQL.
* Usar bindings.
* `whereRaw` solo parametrizado.
* Sort/filter solo whitelist.

---

## 6) Rate limiting

* Rate limiting en login y endpoints sensibles.
* Responder 429 consistente.

---

## 7) Logs seguros

Nunca loguear:

* passwords,
* tokens,
* secrets,
* connection strings.

Usar correlation ID.

---

## 8) TLS y headers

TLS obligatorio en producción.

Headers mínimos:

```text id="sjr4di"
X-Content-Type-Options
X-Frame-Options
HSTS
CSP
```

---

## 9) Adjuntos

* Validar MIME.
* Limitar tamaño.
* Guardar fuera del webroot.

---

## 10) CSRF

Aplicar CSRF cuando se use modo cookie/session.

---

## 11) Secretos y credenciales

* Nunca hardcodear secretos.
* Usar `.env`.
* `.env` fuera de Git.
* `.env.example` obligatorio.

---

## 12) Protección contra ataques

### XSS

* Escapar output.
* Sanitizar HTML.

### CSRF

* Proteger sesiones cookie.

### Clickjacking

* `X-Frame-Options`.

### Command Injection

* Nunca ejecutar comandos sin whitelist.

---

## 13) Headers de seguridad

Obligatorios en producción:

```text id="d06s0y"
X-Content-Type-Options
X-Frame-Options
X-XSS-Protection
Strict-Transport-Security
Content-Security-Policy
Referrer-Policy
```

---

## 14) Logging seguro

### Nunca loguear

* passwords
* tokens
* API keys
* datos sensibles

### Sí loguear

* usuario
* timestamps
* acciones
* errores
* correlation ids

---

## 15) Gestión de sesiones y tokens

### Tokens Sanctum

* expiración 24 horas
* revocación obligatoria
* múltiples dispositivos permitidos

### Regla obligatoria

Al cambiar contraseña:

```php id="xy2qf7"
$user->tokens()->delete();
```

---

## 16) Auditoría

### Auditar

* login
* logout
* cambios password
* operaciones críticas
* cambios permisos

### Retención

* 1 año

---

## 17) Producción

### Checklist mínimo

* HTTPS
* APP_DEBUG=false
* rate limiting
* validaciones activas
* headers seguridad
* logs seguros
* passwords hasheados

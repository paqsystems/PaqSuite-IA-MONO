# Chat Asistente IA con BYOK

Documento de contexto para incorporar un chat asistente de ayuda operativa en el portal, con consumo de inferencia a cargo del cliente o del usuario mediante esquema **BYOK** (`bring your own key`).

Este documento no reemplaza a `ayuda-externa-asistente.md`. Lo complementa como evolución funcional: de un acceso externo simple a un chat asistente propio del producto, abierto en nueva pestaña y contextual al uso del sistema.

## Objetivo

Dar al usuario una vía integrada para consultar ayuda funcional, interpretar pantallas, analizar imágenes y recuperar respuestas apoyadas en documentación propia del sistema, sin asociar el costo de uso del modelo a la cuenta del proveedor del producto.

La experiencia buscada es de **asistencia operativa orientativa**, no de automatización autónoma ni de soporte técnico definitivo.

---

## Alcance

Aplica como capacidad global post-login, disponible desde cualquier pantalla del portal a través de un acceso visible y consistente de ayuda o asistencia.

El asistente debe poder:

- responder consultas textuales sobre uso del sistema;
- utilizar como base documental el manual de usuario y otras fuentes aprobadas del producto;
- aceptar imágenes como apoyo a la consulta cuando el proveedor configurado soporte capacidad multimodal;
- orientar al usuario con pasos sugeridos, aclaraciones y referencias a documentación fuente;
- degradar de forma controlada cuando no exista configuración válida o cuando el proveedor no soporte una capacidad solicitada.

---

## Principio de costos y tenencia

La inferencia del asistente no debe consumirse con credenciales del proveedor del producto.

El principio operativo es:

- cada instalación, tenant o usuario aporta su propia credencial de acceso al proveedor de IA;
- el backend del portal actúa como mediador técnico de la llamada;
- el consumo y facturación quedan asociados a la cuenta externa configurada por quien aporta la credencial.

Esto permite ofrecer una experiencia integrada sin asumir costo directo de tokens desde la cuenta del fabricante del sistema.

---

## Modalidades de credencial admitidas

La arquitectura debe permitir, como mínimo, estas variantes:

### Credencial por cliente o tenant

Modo recomendado para operación habitual.

- Un administrador del cliente configura el proveedor, endpoint, modelo y credencial.
- Todos los usuarios habilitados del tenant consumen sobre esa cuenta externa.
- Simplifica adopción y soporte.

### Credencial por usuario

Modo alternativo para escenarios donde cada usuario deba operar con su propia cuenta externa.

- Cada usuario aporta su propia credencial.
- El costo queda individualizado.
- La experiencia de onboarding y soporte es más exigente.
- La persistencia no debe resolverse en `users`; debe existir una tabla separada para la configuración sensible del asistente.

### Credencial efímera por sesión

Modo opcional para escenarios de mayor sensibilidad.

- La credencial se informa solo para la sesión activa.
- No se persiste en el sistema.
- Reduce responsabilidad de almacenamiento, pero empeora usabilidad.

---

## Configuración funcional mínima

La capacidad debe poder resolverse desde backend o configuración central, sin hardcodear valores en frontend.

Configuración conceptual mínima:

| Clave | Propósito |
|------|-----------|
| `providerId` | Identifica el proveedor o familia de integración |
| `baseUrl` | Endpoint base del proveedor |
| `apiKey` | Credencial externa aportada por cliente o usuario |
| `modelId` | Modelo recomendado o seleccionado |
| `supportsVision` | Indica si acepta imágenes |
| `isEnabled` | Habilita o deshabilita el asistente |

La UI de configuración debe ayudar al operador a completar estos valores y, como mínimo, ofrecer un acceso claro a documentación externa o interna para saber dónde obtener cada dato requerido.

El producto no debe asumir que un operador conoce por sí mismo:

- qué proveedor elegir;
- qué endpoint usar;
- dónde crear la credencial;
- qué modelo conviene seleccionar;
- si el plan habilita imágenes.

Por eso, cada proveedor soportado debería contar al menos con una referencia visible a una URL explicativa de onboarding.

La URL de onboarding (`supportUrl`) debe formar parte del catálogo de proveedores y no de la configuración sensible del usuario.

Además, conviene mantener un catálogo editable de proveedores soportados dentro del proyecto para desacoplar:

- la definición funcional del proveedor;
- las URLs de documentación y onboarding;
- las capacidades declaradas por defecto;
- la activación real que cada despliegue decida habilitar.

Referencia propuesta para ese catálogo:

- `asistente-ia-proveedores.md`

## Persistencia y protección de credenciales

Cuando la modalidad elegida requiera persistencia, las credenciales deben almacenarse cifradas en una tabla separada de `users`.

### Decisión de diseño

La configuración sensible del asistente no debe formar parte de la tabla de usuarios.

Se define una tabla dedicada, siguiendo la convención del producto:

`pq_pedidosweb_asistente_ia_credenciales`

Propósito conceptual:

- aislar secretos y configuración sensible del perfil general del usuario;
- permitir endurecer permisos de lectura y escritura;
- facilitar rotación, revocación y auditoría;
- evitar exposición accidental en consultas o exportaciones de `users`.

### Campos conceptuales mínimos

| Campo | Tipo sugerido | Descripción |
|---|---|---|
| `id_credencial` | bigint identity PK | Identificador técnico |
| `user_id` | bigint | Usuario del portal al que pertenece la configuración |
| `provider_id` | varchar(50) | `providerId` seleccionado por el usuario |
| `base_url` | varchar(255) | Endpoint base configurado |
| `api_key_encrypted` | nvarchar(max) | Credencial cifrada |
| `model_id` | varchar(120) | Modelo configurado |
| `supports_vision` | bit | Si la configuración admite imágenes |
| `is_enabled` | bit | Habilitación lógica |
| `created_at` | datetime | Alta |
| `updated_at` | datetime | Última modificación |

Reglas iniciales:

- una configuración activa por usuario;
- la credencial no se guarda nunca en texto plano;
- el backend cifra antes de persistir;
- el backend descifra solo al momento de invocar al proveedor;
- la clave o mecanismo de descifrado debe quedar fuera de la base de datos;
- la credencial completa no debe exponerse en UI, logs, respuestas API ni mensajes de error.

### Evolución posible

Si más adelante se necesitara configuración compartida por tenant o múltiples proveedores por usuario, esa evolución debe construirse sobre esta tabla dedicada o sobre una variante equivalente, no sobre columnas agregadas a `users`.

---

## Proveedores y modelo recomendado

La solución debe ser **agnóstica de proveedor** en su diseño.

Puede soportar distintas opciones compatibles con chat textual o multimodal, por ejemplo:

- proveedores comerciales con API pública;
- endpoints compatibles tipo OpenAI;
- despliegues privados del cliente;
- runtimes locales o infra administrada por el cliente.

### Catálogo inicial recomendado

Puede inicializarse una tabla de proveedores del asistente con un conjunto corto de opciones conocidas y editables por el proyecto.

Propuesta inicial:

- Ollama
- OpenAI
- Anthropic
- Google Gemini
- Azure OpenAI
- OpenRouter
- Groq
- Mistral

La información funcional editable de este catálogo puede mantenerse en:

- `asistente-ia-proveedores.md`

### Recomendación inicial

Como recomendación de arranque, conviene priorizar integración con **Ollama** cuando el cliente disponga de una instalación propia o administrada que permita operar modelos de la familia Llama u otros compatibles sin depender de una cuenta del fabricante del portal.

Esta recomendación no implica costo cero universal. El costo real depende de la infraestructura elegida por el cliente, del modelo utilizado y de la operación necesaria para sostener ese servicio.

---

## Experiencia de usuario esperada

El asistente debe presentarse como ayuda contextual y operativa del sistema.

Comportamiento esperado:

- acceso desde un punto estable y reconocible de la UI;
- acceso desde menú avatar con la etiqueta **Chat Asistente IA**;
- apertura en nueva pestaña del portal;
- conservación de la pantalla actual del usuario;
- posibilidad de formular consultas libres;
- posibilidad de adjuntar imágenes cuando la configuración lo permita;
- posibilidad de ver referencias o citas a la documentación usada para responder;
- mensaje claro cuando una capacidad no esté disponible.

El sistema debe evitar prometer diagnóstico infalible. La respuesta debe posicionarse como orientación operativa y, cuando corresponda, sugerir revisión manual o derivación a soporte.

### Mensajes editables de la experiencia

El saludo inicial y la frase de cierre orientada a soporte no deberían quedar hardcodeados en código.

Conviene resolverlos desde archivos Markdown editables dentro del proyecto, de modo que producto o implementación puedan ajustarlos sin tocar lógica de negocio.

Archivos propuestos:

- `asistente-ia-mensaje-inicial.md`
- `asistente-ia-mensaje-cierre-soporte.md`

Uso esperado:

- el mensaje inicial se muestra al abrir el asistente o al iniciar una conversación nueva;
- el mensaje final se agrega solo cuando la IA no tenga confianza suficiente en la respuesta;
- el contenido puede evolucionar sin alterar contratos de API.

---

## Fuentes de conocimiento

La base documental principal debe ser el corpus mantenido por el producto.

Fuentes previstas en Fase 1:

- `99-manual-usuario`;
- documentación operativa funcional estable aprobada por el equipo.

Fuentes excluidas en Fase 1:

- SPEC, HU y TR;
- documentación técnica de implementación;
- borradores o documentos no aprobados;
- documentos conceptuales amplios de producto.

La respuesta ideal debe poder citar o referenciar la fuente documental utilizada, en lugar de responder solo desde conocimiento general del modelo.

La ampliación del corpus a importación Excel, documentos funcionales de negocio u otras fuentes debe abrirse en fases posteriores mediante decisión explícita.

---

## Consulta con imágenes

El asistente puede admitir imágenes como soporte de una consulta siempre que `supportsVision` sea verdadero para la configuración activa.

Usos previstos:

- capturas de pantallas del sistema;
- imágenes exportadas o capturadas de planillas Excel usadas por el sistema;
- documentos del negocio que el producto admita como material válido de consulta;
- mensajes visuales de error o advertencia que ayuden a contextualizar la duda.

### Límites iniciales recomendados

Para la primera versión, conviene fijar límites cerrados y no parametrizables por usuario final:

- formatos permitidos: `png`, `jpg`, `jpeg`, `webp`;
- tamaño máximo por archivo: `5 MB`;
- cantidad máxima por consulta: `4` imágenes.

Las consultas pueden enviarse con texto, solo imágenes o combinación de texto e imágenes dentro de esos límites, siempre que el proveedor/modelo activo lo soporte.

La decisión de no exponer estos límites como preferencia editable al usuario reduce complejidad funcional, soporte y combinatoria de validaciones. Si en el futuro aparece una necesidad real, puede abrirse parametrización por tenant o instalación.

### Comportamiento esperado con imágenes

- Si el proveedor no soporta visión, la UI debe informar que los adjuntos no están disponibles para la configuración actual.
- Si un archivo excede límites o tiene formato inválido, la consulta no debe romperse; debe mostrarse un error controlado.
- El usuario debe entender que la imagen se envía al proveedor externo únicamente para el análisis de esa consulta.

---

## Privacidad y ciclo de vida de archivos

Reglas funcionales acordadas para imágenes o adjuntos:

- la imagen no se guarda en el sistema;
- el archivo se usa solo para la consulta en curso;
- se envía al proveedor externo configurado;
- se elimina o descarta luego del análisis;
- no se persiste como histórico del portal.

Esto reduce exposición local de datos, aunque no elimina la necesidad de advertir que el contenido será procesado por un proveedor externo elegido por el cliente o el usuario.

---

## Seguridad y transparencia

El asistente debe operar con reglas claras de transparencia:

- informar cuando responde en base a documentación del sistema;
- informar cuando una respuesta depende del proveedor externo configurado;
- informar cuando una capacidad no está disponible por configuración o plan;
- no exponer credenciales en frontend ni en mensajes de error;
- validar y controlar el uso desde backend.

Cuando existan roles o permisos diferenciados, la disponibilidad del asistente y de sus capacidades puede quedar sujeta a autorización explícita.

---

## Resultado esperado de las respuestas

El resultado esperado no es una "resolución garantizada", sino una orientación útil y controlada.

Las respuestas deberían priorizar:

- explicación funcional del problema observado;
- pasos sugeridos para continuar;
- aclaración de campos, validaciones o restricciones;
- referencia al manual o documento aplicable;
- recomendación de escalar a soporte cuando la evidencia sea insuficiente.

No debe asumirse que la IA reemplaza procesos de soporte, validación funcional o decisión de negocio.

---

## Fases de evolución sugeridas

### Fase 1

Chat en nueva pestaña con consultas textuales y base documental propia.

### Fase 2

Adjuntos de imágenes como apoyo a la consulta, condicionados por `supportsVision`.

### Fase 3

Asistencia contextual enriquecida, combinando consulta, pantalla actual, documentación relevante y adjuntos.

---

## Relación con otros temas

- Ayuda externa simple: `ayuda-externa-asistente.md`
- Menú avatar: `menu-avatar.md`
- Shell principal: `shell-layout.md`
- Parámetros generales: `../04-configuracion-global/parametros-generales.md`
- Importación Excel: `../../05-open-spec/001-Generaliddes/SPEC-001-07-importar-excel.md`

---

## Derivaciones esperables

Este documento debería alcanzar para abrir luego:

- un SPEC específico de chat asistente IA;
- decisiones de configuración global para `BYOK`;
- HU de onboarding/configuración por proveedor;
- HU de consulta textual;
- HU de adjuntos de imágenes;
- TR de backend, frontend, seguridad, límites y privacidad.

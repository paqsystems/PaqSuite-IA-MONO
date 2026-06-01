# Catálogo inicial de proveedores para Asistente IA

Documento editable para inicializar el catálogo funcional de proveedores del asistente IA.

Su objetivo es desacoplar del código:

- el nombre visible del proveedor;
- las URLs de documentación y onboarding;
- las capacidades declaradas por defecto;
- el tipo de integración esperada.

Este catálogo no reemplaza la validación técnica real de cada integración. Sirve como base inicial para configuración y UX.

En la primera HU del catálogo, estos proveedores se consideran soportados como alcance inicial.

## Proveedores iniciales recomendados

| providerId | Nombre visible | Tipo de integración | Soporta BYOK | Soporta imágenes | Requiere `baseUrl` editable | URL documentación | `supportUrl` / onboarding | Notas |
|---|---|---|---|---|---|---|---|---|
| `ollama` | Ollama | Runtime local o administrado por el cliente | Sí | Sí, según modelo | Sí | [Ollama Docs](https://docs.ollama.com/) | [Ollama Download](https://ollama.com/download) | Recomendado para despliegues controlados por el cliente; el costo depende de su propia infraestructura |
| `openai` | OpenAI | API pública | Sí | Sí, según modelo | No | [OpenAI Platform Docs](https://platform.openai.com/docs/overview) | [OpenAI API Keys](https://platform.openai.com/api-keys) | Opción estándar para integración directa |
| `anthropic` | Anthropic | API pública | Sí | Sí, según modelo | No | [Anthropic Docs](https://docs.anthropic.com/) | [Anthropic Console](https://console.anthropic.com/) | Conveniente para flujos conversacionales largos |
| `googleGemini` | Google Gemini | API pública | Sí | Sí, según modelo | No | [Gemini API Docs](https://ai.google.dev/gemini-api/docs) | [Google AI Studio](https://aistudio.google.com/) | Puede requerir proyecto o habilitación específica según cuenta |
| `azureOpenAi` | Azure OpenAI | API administrada en Azure | Sí | Sí, según despliegue/modelo | Sí | [Azure OpenAI Docs](https://learn.microsoft.com/azure/ai-services/openai/) | [Azure Portal](https://portal.azure.com/) | Requiere endpoint y despliegue específicos del cliente |
| `openRouter` | OpenRouter | Agregador compatible | Sí | Sí, según modelo | No | [OpenRouter Docs](https://openrouter.ai/docs) | [OpenRouter Keys](https://openrouter.ai/settings/keys) | Útil para dar variedad de modelos con una sola cuenta |
| `groq` | Groq | API pública | Sí | Sí, según modelo | No | [Groq Docs](https://console.groq.com/docs/overview) | [Groq Console](https://console.groq.com/keys) | Suele priorizar baja latencia |
| `mistral` | Mistral | API pública | Sí | Sí, según modelo | No | [Mistral Docs](https://docs.mistral.ai/) | [Mistral Console](https://console.mistral.ai/) | Alternativa comercial conocida para texto y visión |

## Criterios para admitir un proveedor

- debe ofrecer un mecanismo claro para que el cliente o usuario aporte su propia credencial;
- debe contar con documentación pública accesible para onboarding;
- debe permitir explicar con claridad qué valor usar para `apiKey`, `baseUrl` y `modelId`;
- si soporta imágenes, debe indicarse que la capacidad depende del modelo configurado;
- la habilitación en producto no debe asumirse automática solo por estar listado aquí.

## Notas de mantenimiento

- Este archivo es editable y actúa como fuente funcional inicial para sembrar la tabla de proveedores.
- Las URLs de documentación pueden cambiar con el tiempo; conviene revisarlas al cerrar la TR técnica.
- Si un proveedor deja de ser prioritario, puede marcarse como inactivo en la tabla sin eliminar su histórico.

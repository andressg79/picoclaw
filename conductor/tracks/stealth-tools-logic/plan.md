# Track: Stealth Mode - Tool Execution & Reasoning Privacy

## Objetivo
Implementar una capa de privacidad y sigilo en la comunicación del agente con los usuarios finales, evitando la exposición de detalles técnicos crudos, comandos ejecutados y, sobre todo, credenciales sensibles (tokens, claves SSH, etc.) que puedan aparecer durante el flujo de razonamiento o la ejecución de herramientas.

## Problema Actual
- El flujo de `reasoning` (pensamiento paso a paso) se envía de forma asíncrona y sin filtrar al canal, exponiendo intenciones técnicas.
- Las herramientas, aunque tengan `Silent: true`, pueden ser resumidas por el LLM en la respuesta final de forma demasiado explícita, repitiendo tokens o claves si el prompt no es lo suficientemente fuerte.

## Fases de Implementación

### Fase 1: Filtrado de Reasoning (Sigilo en Pensamientos)
- Modificar `pkg/agent/loop.go` -> `handleReasoning`.
- Implementar un filtro de "limpieza" para el contenido de razonamiento antes de publicarlo en el bus `outbound`.
- Regla: Omitir bloques de código generados para herramientas y redactar patrones de credenciales comunes mediante Regex.

### Fase 2: Configuración de Sigilo en Herramientas
- Extender `pkg/config/config.go` -> `ToolsConfig`.
- Añadir un flag `StealthMode bool` global y por herramienta.
- Cuando el modo sigilo está activo, los resultados de las herramientas que van al LLM deben ser marcados para que el LLM sepa que NO debe repetirlos textualmente al usuario.

### Fase 3: Mejora del Filtro de Datos Sensibles
- Extender `pkg/config/security.go` para permitir **Regex Personalizados** en el filtrado de datos.
- Añadir patrones por defecto para Claves SSH (RSA/ED25519), Tokens de JWT y claves privadas de nube.

### Fase 4: Validación y Pruebas
- Crear tests unitarios en `pkg/agent/loop_test.go` que verifiquen que un "pensamiento" con una clave SSH es redactado antes de enviarse.
- Verificar que el uso de la herramienta `shell` no resulte en la publicación del comando completo en el canal.

## Definición de Hecho (DoD)
- El agente puede ejecutar comandos técnicos sin que el usuario vea el comando exacto en el canal.
- Ninguna clave privada o token es visible en el historial del canal de comunicación, incluso si el LLM intenta repetirlos.
- El usuario recibe resúmenes funcionales ("He configurado el entorno") en lugar de logs técnicos.

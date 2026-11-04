# Mapa del curso

## Estado de publicación

Los seis capítulos del **bloque I — Fundamentos** están publicados. Los bloques
II a V y el proyecto final describen el recorrido previsto y todavía no tienen
capítulos publicados. El mapa se irá actualizando conforme se añada material.

[Volver a la presentación del curso](../README.md).

## Método

Cada unidad sigue cuatro capas:

1. **Modelo mental:** qué problema existe y qué pieza lo resuelve.
2. **Protocolo observable:** mensajes, eventos, ficheros y procesos que podemos inspeccionar.
3. **Implementación en PI:** cómo materializa PI esa pieza.
4. **Laboratorio:** una prueba que permite confirmar o refutar el modelo mental.

No daremos por entendido un concepto porque sepamos utilizar su comando. Tampoco
confundiremos el comportamiento actual de PI con una ley universal de los agentes.

## Bloque I — Fundamentos

### 1. [Harness y bucle agente](01-fundamentos/01-harness-y-bucle-agente.md)

- Diferencia entre modelo, proveedor, cliente y harness.
- El modelo propone; el proceso anfitrión ejecuta.
- Bucle `mensaje -> inferencia -> tool call -> resultado -> inferencia`.
- Estado, efectos externos y condiciones de parada.

### 2. [La frontera HTTP/JSON](01-fundamentos/02-http-json-y-streaming.md)

- Cliente y servidor, petición y respuesta, cabeceras y cuerpo.
- JSON como representación, no como transporte ni como programa.
- Streaming y eventos incrementales.
- Errores de red, proveedor, protocolo, herramienta y aplicación.

### 3. [Mensajes, tokens y contexto](01-fundamentos/03-mensajes-tokens-y-contexto.md)

- Roles, bloques de contenido y resultados de herramientas.
- System prompt, historial activo, definiciones de tools y skills.
- Ventana de contexto, coste, caché, límites y pérdida de información.
- Por qué contexto no equivale a memoria.

### 4. [Tool calling](01-fundamentos/04-tool-calling.md)

- Esquema de entrada, identificador de llamada y validación.
- Dispatcher, ejecución, captura de salida y correlación del resultado.
- Herramientas secuenciales y paralelas.
- Timeouts, idempotencia, reintentos y efectos irreversibles.

### 5. [Estado, sesiones y compaction](01-fundamentos/05-estado-sesiones-y-compaction.md)

- Log append-only y árbol de ramas de PI.
- Continuar, ramificar, clonar y bifurcar.
- Reconstrucción del contexto activo.
- Resumir no es comprimir sin pérdida.

### 6. [Seguridad y confianza](01-fundamentos/06-seguridad-y-confianza.md)

- Autoridad del modelo frente a autoridad del proceso.
- Permisos Unix, sandbox, contenedor y aislamiento de red.
- Prompt injection, exfiltración y cadena de suministro.
- Modos interactivo y desatendido.

## Bloque II — Anatomía de PI

### 7. Arranque y configuración

- Directorio de trabajo, descubrimiento y precedencia.
- `AGENTS.md`, settings globales/locales y project trust.
- Modelos, proveedores, autenticación y selección.

### 8. Recursos declarativos

- Prompt templates, skills, temas y paquetes.
- Cuándo basta una instrucción y cuándo hace falta código.
- Descubrimiento y carga bajo demanda.

### 9. Extensiones ejecutables

- Módulos TypeScript y ciclo de vida.
- Tools, comandos, hooks, renderers y componentes TUI.
- Fronteras de confianza y manejo de errores.

### 10. Integraciones

- MCP y su relación con las tools nativas.
- Print mode, eventos JSONL, RPC y SDK.
- Integración como proceso frente a integración en proceso.

## Bloque III — PI personal y profesional

### 11. Configuración mínima de esta máquina

- OpenAI Codex mediante OAuth.
- Modelo y nivel de razonamiento por defecto.
- Context files, skills y comandos reutilizables.
- Separación entre configuración personal y configuración de proyecto.

### 12. Perfiles operativos

- Exploración de solo lectura.
- Desarrollo supervisado.
- Automatización contenida.
- Trabajo con repositorios o datos no confiables.

### 13. Observabilidad y evaluación

- Sesiones como trazas y límites de esa analogía.
- Latencia, tokens, coste, llamadas y fallos.
- Tests de tools y evaluaciones end-to-end.
- Cómo comparar resultados sin confundir modelo y harness.

## Bloque IV — De PI a otros vendors

### 14. Matriz comparativa

- PI, Codex, Claude Code y otros agentes relevantes en 2026.
- Misma taxonomía: loop, tools, contexto, estado, seguridad y extensibilidad.
- Qué pertenece al producto y qué pertenece al modelo/proveedor.

### 15. Patrones avanzados

- Subagentes, concurrencia y coordinación.
- Planificación explícita, verificación y bucles de reparación.
- Recuperación de contexto y memoria externa.
- Human-in-the-loop y políticas de aprobación.

## Bloque V — Construir un harness desde cero

### 16. Harness mínimo

- Cliente de modelo.
- Historial de mensajes.
- Una tool segura y el dispatcher.
- Bucle y condición de salida.

### 17. Convertirlo en un sistema utilizable

- Persistencia, streaming, cancelación y control de errores.
- Presupuesto de contexto y compaction.
- Permisos, aislamiento y auditoría.
- CLI, tests y empaquetado.

## Proyecto final

Construiremos un harness pequeño para una tarea real, con requisitos explícitos,
amenazas, pruebas y comparación razonada con PI. Cada componente deberá existir por
una necesidad demostrada; no por imitar la lista de funciones de otro producto.

# Glosario vivo

## Agente

Sistema que persigue un objetivo mediante decisiones y acciones sucesivas. En este
curso incluye modelo, harness, tools, contexto, estado y entorno; no es sinónimo de LLM.

## API

Contrato mediante el que un componente solicita operaciones a otro. Puede exponerse
por HTTP, una librería, un proceso local u otros mecanismos.

## Contexto

Información incluida en una petición concreta al modelo. Puede contener
instrucciones, mensajes, resultados de tools y descripciones de recursos. No equivale
a toda la información almacenada por el sistema.

## Compaction

Sustitución de una parte antigua del contexto por un resumen para liberar presupuesto.
Es una transformación con pérdida: las entradas originales pueden seguir persistidas
aunque ya no se envíen literalmente al modelo.

## Harness

Programa anfitrión que orquesta peticiones al modelo, contexto, tools, estado,
permisos, interfaz y condiciones de parada. PI es un harness.

## HTTP

Protocolo de petición y respuesta utilizado habitualmente entre clientes y servidores.
Define método, destino, cabeceras, cuerpo y códigos de estado; no define por sí mismo
el significado de los datos de una API de modelos.

## JSON

Formato textual para representar valores estructurados: objetos, arrays, strings,
números, booleanos y `null`. No ejecuta acciones y no implica por sí mismo HTTP.

## JSONL

Formato donde cada línea es un valor JSON independiente. Facilita logs append-only y
flujos de eventos incrementales.

## Petición y respuesta

En HTTP, el cliente envía una petición y el servidor devuelve una respuesta. Una
respuesta HTTP correcta no garantiza que la tarea de nivel superior haya terminado con
éxito.

## Serialización

Conversión de estructuras en memoria a una representación transportable o persistible,
como texto JSON codificado en UTF-8. La deserialización realiza el recorrido inverso.

## Streaming

Entrega incremental de una respuesta mediante eventos o fragmentos. Los deltas
intermedios no deben confundirse con el mensaje final autoritativo.

## Token

Unidad producida por el tokenizador de un modelo. No equivale necesariamente a una
palabra, carácter o byte. Mensajes, instrucciones y definiciones de tools consumen
tokens cuando forman parte de la entrada.

## LLM o modelo

Modelo de lenguaje que, dados unos datos de entrada, genera una salida. El modelo no
hereda automáticamente acceso al sistema donde corre el cliente.

## Prompt caching

Reutilización por el proveedor de computación asociada a un prefijo de entrada. Puede
reducir latencia o coste, pero no es memoria semántica ni recupera información ausente.

## Proveedor

Servicio o infraestructura mediante el que se accede a un modelo. Modelo y proveedor
no son necesariamente una relación uno a uno.

## Run

Ejecución iniciada a partir de una entrada y que puede requerir varias consultas al
modelo antes de terminar, fallar o ser cancelada.

## Sesión

Registro persistente de la conversación y sus eventos. En PI es un árbol guardado en
un archivo JSONL; una de sus ramas es la activa.

## Tool

Operación que el harness ofrece al modelo mediante un contrato estructurado. La
solicitud la genera el modelo; la implementación la ejecuta el harness o un sistema conectado.

## Dispatcher

Componente del harness que resuelve una tool call, valida y autoriza sus argumentos,
invoca la implementación correspondiente y construye el resultado correlacionado.

## Idempotencia

Propiedad por la que repetir una operación con la misma intención no acumula un efecto
adicional relevante. Facilita reintentos, pero no sustituye la autorización ni permite
suponer que una implementación concreta cumple esa propiedad.

## Tool call

Solicitud estructurada generada por el modelo. Contiene al menos el nombre de la tool,
sus argumentos y un identificador que permite correlacionarla con su resultado.

## Tool result

Representación que el ejecutor devuelve al harness después de aceptar, bloquear,
completar o fallar una tool call. No equivale necesariamente al efecto real producido.

## Turn

En PI, ciclo que incluye una respuesta del asistente, la ejecución de sus tool calls y
el registro de los resultados. Un run puede contener varias turns.

## Ventana de contexto

Capacidad limitada de información que puede participar en una inferencia, incluyendo
la entrada y el espacio que el runtime deba reservar para la salida según el protocolo
y modelo utilizados.

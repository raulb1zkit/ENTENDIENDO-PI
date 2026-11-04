# 5. Estado, sesiones y compaction

## Objetivo del capítulo

Puedes cerrar PI y continuar mañana. También puedes volver a un mensaje anterior,
probar otra solución y compactar una conversación larga. Estas acciones parecen
variantes de «recordar», pero cambian piezas diferentes del sistema.

Al terminar deberías poder:

- distinguir estado del mundo, sesión persistida y contexto enviado al modelo;
- leer las relaciones entre entradas de una sesión sin saber JavaScript;
- elegir entre continuar, ramificar, bifurcar y clonar;
- explicar qué conserva la compaction y qué información puede perder;
- comprobar lo ocurrido sin confundir el stream de eventos con el archivo de sesión.

Los ejemplos corresponden a PI 1.3.0. La parte de lectura y el primer laboratorio
pueden hacerse sin proveedor; conversar y generar resúmenes requiere un modelo accesible.

## 1. Hay varios estados a la vez

Imagina que pides a PI modificar un archivo y ejecutar una prueba. Al terminar hay
al menos tres estados distintos:

| Estado | Qué contiene | Dónde vive |
|---|---|---|
| Mundo externo | Archivo modificado, procesos, cambios en servicios | Disco, sistema operativo y servicios |
| Sesión | Mensajes, llamadas, resultados y metadatos registrados | Archivo mantenido por PI |
| Contexto activo | Información que se prepara para la próxima inferencia | Petición construida por el harness |

El archivo puede seguir modificado aunque borres la sesión. Una respuesta puede
seguir guardada en la sesión aunque haya dejado de enviarse al modelo. El modelo
puede leer un resultado antiguo que ya no representa el contenido actual del disco.

Por tanto, «PI lo recuerda» necesita una pregunta adicional: ¿está guardado, está
incluido en la siguiente petición o sigue siendo cierto en el mundo real?

## 2. Persistir una sesión no entrena al modelo

En el bucle que hemos estudiado, el harness reúne mensajes y los envía al proveedor.
Cuando reanudas una sesión, recupera datos guardados y construye contexto con ellos.
No necesita modificar los pesos del modelo para continuar la conversación.

La continuidad depende de lo que se vuelva a presentar. Si una restricción quedó
fuera del contexto, no basta con que exista en algún archivo de sesión para que el
modelo la tenga presente.

Esto conecta con el capítulo 3: almacenamiento y contexto tienen capacidades y
costes diferentes. Puedes guardar una conversación enorme y enviar solo una parte.

## 3. JSONL: una entrada por línea

PI guarda sesiones como archivos JSONL. Cada línea contiene un objeto JSON. En el
uso normal, añade entradas para registrar mensajes y otros cambios: es un registro
*append-only*, es decir, al que se va agregando información al final.

Es un modelo de persistencia, no una garantía de que el archivo sea inmutable: un
usuario puede editarlo o borrarlo, y una migración de formato puede transformarlo.

Este ejemplo didáctico muestra solo los campos relevantes:

```json
{"type":"message","id":"a","parentId":null,"message":{"role":"user","content":"Explora dos opciones"}}
{"type":"message","id":"b","parentId":"a","message":{"role":"assistant","content":[{"type":"text","text":"Podemos usar A o B"}]}}
{"type":"message","id":"c","parentId":"b","message":{"role":"user","content":"Desarrolla A"}}
{"type":"message","id":"d","parentId":"b","message":{"role":"user","content":"Mejor desarrolla B"}}
```

No hace falta ejecutar ese JSON. Se lee así:

- `type` dice qué clase de entrada es.
- `id` identifica una entrada.
- `parentId` apunta a la entrada anterior **dentro de esa rama**.
- `null` indica que la entrada no tiene padre.
- `message` contiene el mensaje y su rol.

Aunque `d` esté escrita después de `c`, su padre es `b`. Por eso ambas representan
alternativas que salen del mismo punto:

```text
a → b ─┬→ c: desarrollar A
       └→ d: desarrollar B
```

El orden de las líneas cuenta cuándo se añadieron las entradas. Los enlaces entre
ellas cuentan qué camino de conversación forman. Son relaciones distintas.

El archivo real tiene además una cabecera de sesión, fechas y otros campos. La
cabecera usa `type: "session"` y su propio identificador de sesión; no es un nodo
del árbol con un `parentId`.

## 4. Rama activa y reconstrucción del contexto

La posición actual en el árbol suele llamarse *leaf*, u hoja. Para recuperar su
historia, PI sigue los padres hasta la raíz y obtiene el camino correspondiente.

Si la hoja del ejemplo es `d`, la historia seleccionada pasa por `a`, `b` y `d`.
La opción `c` sigue almacenada, pero no se incluye automáticamente como conversación
activa. Un resumen de rama puede trasladar información de ella, como veremos después.

La construcción del contexto tiene más pasos que seleccionar el camino:

1. Localizar la rama activa y sus entradas.
2. Aplicar la compaction pertinente, si existe.
3. Aplicar las ediciones de contexto de esa rama.
4. Reconstruir los mensajes, el prompt y las tools correspondientes.
5. Adaptar la información al protocolo del proveedor.

PI registra también metadatos, como cambios de modelo y entradas de extensiones.
No todas las líneas del archivo se convierten en texto enviado al modelo. Por
ejemplo, una entrada `custom` guarda estado de extensión sin participar por sí
misma en el contexto; una `custom_message` sí puede aportar un mensaje.

La referencia de formato describe además `context_edit`: una entrada que omite o
reemplaza contenido para futuras inferencias sin borrar el registro original. Esto
refuerza la diferencia entre historial guardado y contexto efectivo.

## 5. Continuar, ramificar, bifurcar y clonar

Estas son las operaciones de PI 1.3.0 descritas en su documentación de sesiones:

| Acción | Comando | Resultado |
|---|---|---|
| Continuar la sesión más reciente del directorio | `pi --continue` | Reabre trabajo guardado |
| Elegir una sesión | `pi --resume` o `/resume` | Abre un selector |
| Iniciar una sesión nueva | `/new` | Empieza otra conversación |
| Navegar por el árbol | `/tree` | Cambia de posición en la misma sesión |
| Bifurcar desde un mensaje de usuario anterior | `/fork` | Crea otra sesión para esa alternativa |
| Clonar la rama activa | `/clone` | Copia esa rama a una sesión nueva |

Con `/tree`, seleccionar un mensaje de usuario devuelve su texto al editor. Puedes
cambiarlo y enviarlo para crear otra rama. Seleccionar una respuesta u otra entrada
permite continuar después de ella con el editor vacío.

Usa `/session` para comprobar la ruta del archivo, el identificador y las métricas.
`/name` permite asignar un nombre reconocible.

## 6. Volver atrás no deshace el mundo

Supón que una rama ejecutó estas acciones:

```text
Editar config.json → enviar una petición a un servicio → registrar los resultados
```

Volver con `/tree` a un mensaje anterior cambia el camino de conversación. El
archivo editado y la petición enviada siguen existiendo. Bifurcar o clonar tampoco
crea una copia independiente del disco ni revierte esos efectos.

Esto puede producir una situación engañosa: el modelo continúa desde una historia
anterior, pero trabaja sobre archivos que ya cambiaron en la rama abandonada.

Antes de comparar alternativas que modifican código, comprueba el estado real con
Git y usa commits o worktrees cuando necesites aislar cambios. Git ayuda a recuperar
archivos versionados; tampoco desenvía correos ni revierte operaciones remotas.

## 7. Qué hace la compaction

Cuando el contexto crece demasiado, PI puede sustituir parte de la historia enviada
por un resumen y conservar mensajes recientes. También puedes solicitarlo con
`/compact` y añadir instrucciones para enfocar el resumen.

```text
Antes, en el contexto:
  instrucciones + conversación antigua + mensajes recientes

Después, en el contexto:
  instrucciones + resumen de lo antiguo + mensajes recientes

En el archivo de sesión:
  entradas originales + nueva entrada de compaction
```

Se añade una entrada `compaction` con el resumen y una referencia llamada
`firstKeptEntryId`: el identificador de la primera entrada anterior que se conserva
en el contexto. El harness usa esa frontera al reconstruir la siguiente petición.
La documentación contempla también el caso en que no se conserva ninguna entrada
anterior, representado mediante el identificador de la propia compaction.

Los mensajes originales permanecen en el archivo. Eso permite inspeccionarlos,
pero no implica que el modelo los esté recibiendo en la siguiente inferencia.

## 8. Resumir implica seleccionar y perder detalles

Comprimir un archivo con un formato sin pérdida permite recuperar exactamente sus
bytes. Resumir una conversación mediante un modelo produce una representación
selectiva; puede omitir condiciones o interpretar mal una decisión.

Considera esta conversación:

```text
Usuario: Puedes modificar el código, pero no publicar todavía.
...
Usuario: Cuando pasen las pruebas, prepara una propuesta para revisión.
```

Un resumen defectuoso como «implementar y publicar cuando pasen los tests» cambia
la autorización. La compaction puede influir en acciones posteriores si se pierde
una condición importante.

Un resumen útil conserva al menos:

- objetivo vigente y restricciones explícitas;
- decisiones tomadas y su motivo;
- trabajo completado y pendiente;
- fallos, incertidumbres y comprobaciones necesarias;
- referencias a archivos y evidencia que se pueda volver a consultar.

Cuando un dato importe para una acción, conviene comprobar su fuente actual.
«Las pruebas pasaron» describe una ejecución concreta, no todas las versiones
futuras del código.

## 9. Cuándo compacta PI y qué necesita

La referencia de PI 1.3.0 describe una condición de umbral aproximada:

```text
contexto estimado > ventana del modelo − reserva para la respuesta
```

La configuración permite ajustar la reserva y cuánta historia reciente mantener.
No necesitas memorizar los valores para entender el mecanismo: debe quedar espacio
para que el modelo pueda contestar y continuar el bucle.

La compaction predeterminada realiza una inferencia para resumir. Puede consumir
cuota y fallar si el proveedor no está disponible o rechaza la petición. No hace
falta llenar la ventana para probarla: `/compact` permite solicitarla manualmente.

Al cambiar de rama con `/tree`, PI también puede resumir el camino que abandonas.
Ese resumen se registra como `branch_summary`. Su propósito es transportar trabajo
relevante entre ramas, mientras que la compaction busca reducir el contexto activo.
Ambos mecanismos dependen de la calidad del resumen.

## 10. Archivo de sesión y stream de eventos

En el laboratorio anterior guardaste la salida de `pi --mode json`. Ese stream
incluye eventos de ejecución como `message_update` o `tool_execution_end`.

El archivo persistido de sesión usa tipos como `message`, `compaction` y
`branch_summary`. Ambos pueden ser JSONL, pero tienen esquemas y propósitos distintos.

Buscar `tool_execution_end` en un archivo de sesión puede no devolver nada aunque
la herramienta se haya ejecutado. Buscar `compaction` en un archivo que solo contiene
un registro de ejemplo tampoco demuestra nada sobre una sesión real.

Antes de filtrar, mira qué tipos hay:

```bash
jq -r '.type' archivo.jsonl | sort | uniq -c
```

`jq -r` muestra cada tipo como texto; `sort` junta los iguales y `uniq -c` los cuenta.
Un filtro vacío significa que no encontró coincidencias. Hay que investigar por qué.

## Laboratorio A: observar un árbol sin modelo

Puedes hacerlo sin proveedor y sin modificar ninguna sesión real.

### Paso 1: crear un ejemplo separado

```bash
cat > /tmp/pi-arbol-ejemplo.jsonl <<'JSONL'
{"type":"message","id":"a","parentId":null,"message":{"role":"user","content":"Explora dos opciones"}}
{"type":"message","id":"b","parentId":"a","message":{"role":"assistant","content":[{"type":"text","text":"A o B"}]}}
{"type":"message","id":"c","parentId":"b","message":{"role":"user","content":"Desarrolla A"}}
{"type":"message","id":"d","parentId":"b","message":{"role":"user","content":"Desarrolla B"}}
JSONL
```

`cat` escribe el bloque hasta la marca `JSONL` en el archivo indicado. Es una muestra
didáctica incompleta: no la uses como sesión importable de PI.

### Paso 2: mostrar las relaciones

```bash
jq -c '{id, parentId, role: .message.role}' /tmp/pi-arbol-ejemplo.jsonl
```

`{id, parentId, ...}` construye un objeto pequeño con esos campos. Observa que `c`
y `d` tienen el mismo padre. Dibuja los caminos desde cada uno hasta la raíz.

### Paso 3: comprobar los hijos de un nodo

```bash
jq -c 'select(.parentId == "b") | {id, texto: .message.content}' \
  /tmp/pi-arbol-ejemplo.jsonl
```

`select` conserva solo entradas cuyo padre sea `b`. Deben aparecer `c` y `d`.
Si `d` es la hoja activa, ¿por qué no basta con enviar todas las líneas al modelo?

## Laboratorio B: sesión real, rama y resumen

Este laboratorio sí necesita un proveedor funcional y puede consumir cuota. Usa
una conversación de prueba, sin pedir modificaciones de archivos.

1. Abre `pi`, inicia `/new` y ponle nombre con `/name`.
2. Envía: «Estamos comparando viajar en tren o autobús. Restricción: llegar antes
   de las 18:00. No consultes herramientas; solo organiza las opciones sin inventar horarios».
3. Añade otro mensaje: «Desarrolla la opción del tren». Espera la respuesta.
4. Usa `/session` y apunta la ruta exacta del archivo de esta sesión.
5. Usa `/tree`, selecciona el último mensaje de usuario y cambia tren por autobús.
   Envía la variante y observa ambas ramas en `/tree`. Si PI ofrece un resumen de
   la rama abandonada, fíjate en la decisión que eliges.
6. Solicita `/compact Conserva la restricción de llegar antes de las 18:00 y la opción activa`.
   Si falla, revisa el error del proveedor; no interpretes el fallo como un resumen correcto.

En otra terminal, consulta el archivo indicado por `/session`, sustituyendo la ruta:

```bash
jq -c 'select(.type == "message") | {id, parentId, role: .message.role}' \
  /ruta/a/la/sesion.jsonl
```

Después consulta los resúmenes:

```bash
jq -c 'select(.type == "compaction" or .type == "branch_summary")
  | {type, id, parentId, firstKeptEntryId, summary}' \
  /ruta/a/la/sesion.jsonl
```

Un campo `null` puede ser normal: `branch_summary` no usa `firstKeptEntryId`.
Comprueba que los mensajes antiguos siguen guardados tras compactar y revisa si el
resumen conservó la restricción. La presencia del registro prueba que se guardó un
resumen; evaluar su calidad requiere leerlo.

## Comprobación de comprensión

1. ¿Qué diferencia hay entre un dato guardado y un dato incluido en el contexto?
2. ¿Por qué dos líneas consecutivas pueden pertenecer a ramas diferentes?
3. ¿Qué aporta `parentId` que no aporta la posición de una línea?
4. ¿Cuándo usarías `/tree`, `/fork` y `/clone`?
5. Si vuelves a un mensaje anterior, ¿qué ocurre con los archivos ya modificados?
6. ¿Qué conserva PI en disco después de una compaction y qué envía al modelo?
7. ¿Por qué un resumen puede cambiar accidentalmente una autorización?
8. ¿Qué comprobarías antes de concluir que un filtro `jq` vacío revela un fallo?

## Respuestas revisadas

Comprobación de comprensión revisada el 4 de noviembre de 2026. Siguiente unidad:
[06 — Seguridad y confianza](06-seguridad-y-confianza.md).

### 1. Dato guardado frente a dato incluido en el contexto

Un dato guardado existe en el archivo de sesión. Un dato incluido en el contexto
se presenta al modelo en una petición concreta y puede influir en su respuesta.
Una instrucción puede seguir almacenada en una rama abandonada sin llegar al modelo
cuando continuamos por otra. Estar guardado no equivale a estar disponible para
esa inferencia.

### 2. Líneas consecutivas y ramas diferentes

El orden de las líneas refleja cuándo se añadieron las entradas. Los enlaces
`parentId` indican cómo se conectan en la conversación. Si `c` y `d` tienen como
padre a `b`, son alternativas que parten del mismo punto: aunque `d` se escriba
justo después de `c`, no continúa desde ella.

```text
a → b ─┬→ c
       └→ d
```

### 3. Qué aporta parentId

`parentId` identifica el padre directo de una entrada. Siguiendo esos enlaces
podemos reconstruir su camino hasta la raíz, algo que la posición de la línea no
permite hacer. El padre puede estar muchas líneas antes.

Matiz sobre la respuesta inicial: no identifica por sí solo una rama completa;
varias ramas pueden compartir antepasados. La historia seleccionada depende de la
hoja activa y de su cadena de padres.

### 4. Cuándo usar /tree, /fork y /clone

- `/tree`: para volver a un punto o explorar otra alternativa dentro de la misma
  sesión.
- `/fork`: para crear otra sesión desde un mensaje de usuario anterior y
  desarrollar esa alternativa por separado.
- `/clone`: para copiar la rama activa a una sesión nueva y continuar desde esa
  copia.

Estas operaciones no crean una copia independiente de los archivos del proyecto.

### 5. Qué ocurre con los archivos al volver atrás

Los archivos siguen modificados. Cambiar el camino de conversación no revierte
los efectos de las herramientas. Git puede ayudar a recuperar archivos versionados
y los worktrees a aislar alternativas, pero no desenvía correos ni revierte por sí
solo operaciones remotas.

Matiz sobre la respuesta inicial: el modelo sí puede saber que un correo se envió
si recibe el resultado de la herramienta o consulta evidencia del envío. Si esa
información queda fuera del contexto, puede desconocerlo. Un resultado antiguo
tampoco sustituye una comprobación actual cuando esta sea necesaria.

### 6. Disco y contexto después de una compaction

PI conserva en disco las entradas originales y añade una entrada de compaction
con el resumen y `firstKeptEntryId`, que marca la frontera de las entradas que se
conservan en el contexto.

Para la siguiente inferencia construye el contexto con las instrucciones, el
resumen y los mensajes que corresponda conservar, incluidos los nuevos mensajes.
Los originales antiguos siguen guardados, pero no se envían íntegros como historia
activa.

### 7. Cómo puede cambiar una autorización al resumir

Resumir selecciona información y puede perder detalles. Si «puedes editar, pero
no publicar» se transforma en «puedes editar y publicar», el resumen representa
un permiso que el usuario no concedió. Omitir una negación, una condición o quién
debe aprobar puede alterar las acciones posteriores.

Matiz sobre la respuesta inicial: el problema no depende solo de que el modelo
sea no determinista. Incluso un resumen determinista puede perder información.
Conviene conservar objetivos, restricciones, autorizaciones y condiciones, además
del trabajo pendiente, y comprobar la fuente cuando una acción dependa de ello.

### 8. Qué comprobar ante un filtro jq vacío

- Que se está leyendo el archivo y la sesión correctos.
- Si el archivo es una sesión persistida, un stream de eventos o una muestra
  didáctica: sus esquemas y tipos pueden ser diferentes.
- Que existen el tipo y los campos que busca el filtro.
- Que la acción esperada llegó a ocurrir correctamente; por ejemplo, que la
  compaction no falló por un error del proveedor.

Una primera inspección útil es:

```bash
jq -r '.type' archivo.jsonl | sort | uniq -c
```

Un resultado vacío solo demuestra que el filtro no encontró coincidencias.
Por ejemplo, buscar `tool_execution_end` en una sesión persistida puede no devolver
nada aunque la herramienta sí se haya ejecutado: ese tipo pertenece al stream de
eventos.

## Fuentes primarias

La explicación de PI se basa en la documentación incluida con PI 1.3.0. Las páginas
`latest` pueden evolucionar respecto de esa copia.

- [Pi: Sessions and Context](https://pi.dev/docs/latest/sessions)
- [Pi: Session File Format](https://pi.dev/docs/latest/session-format)
- [Pi: Compaction Reference](https://pi.dev/docs/latest/compaction)
- [Pi: How Pi Works](https://pi.dev/docs/latest/how-pi-works)

## Siguiente capítulo

[Seguridad y confianza](06-seguridad-y-confianza.md)

# Entendiendo los agentes de código con PI

**Curso práctico en español para entender cómo funcionan los agentes de código:
modelos, herramientas, contexto, sesiones y seguridad.**

¿Qué ocurre entre pedir «arregla este error» y encontrar un archivo modificado?
Aquí seguimos ese recorrido paso a paso, con explicaciones, ejemplos observables
y laboratorios. Usamos [PI](https://pi.dev/) como sistema de referencia para estudiar
las piezas de un agente y aprender a reconocerlas en otras herramientas.

El material es público para que cualquiera pueda aprender a su ritmo y aportar
correcciones. Es un proyecto educativo independiente y en desarrollo.

**[Empieza por el capítulo 1](curso/01-fundamentos/01-harness-y-bucle-agente.md)**
· [Mapa completo](curso/00-mapa-del-curso.md)
· [Glosario](curso/glosario.md)
· [Cómo contribuir](CONTRIBUTING.md)

## Para quién es

Para personas que usan agentes de código o quieren empezar a usarlos y entender
qué ejecutan, qué información reciben y qué permisos tienen.

Conviene manejar archivos y comandos básicos de terminal. No necesitas experiencia
previa construyendo agentes ni saber TypeScript: los fragmentos de código se usan
para explicar el funcionamiento y se acompañan de comentarios.

## Qué vas a aprender

- Distinguir el modelo, el proveedor y el **harness**, el programa que organiza
  la conversación y ejecuta las herramientas.
- Seguir el bucle de petición, inferencia, llamada a herramienta, resultado y
  continuación.
- Leer JSON, JSONL y eventos para comprobar qué ocurrió.
- Separar historial guardado, contexto enviado al modelo y estado real de archivos
  y servicios.
- Entender los límites de los resúmenes, los permisos y las fronteras de aislamiento.

Los bloques posteriores se orientan a configurar PI, comparar otros agentes y
construir un harness pequeño desde cero.

## Capítulos disponibles

El **bloque I, Fundamentos, está publicado**. Puedes leer sus seis capítulos en orden:

| Capítulo | Qué estudias |
|---|---|
| [01 · Harness y bucle agente](curso/01-fundamentos/01-harness-y-bucle-agente.md) | Cómo una petición llega a convertirse en una acción |
| [02 · HTTP, JSON y streaming](curso/01-fundamentos/02-http-json-y-streaming.md) | Cómo viajan las peticiones, respuestas y eventos |
| [03 · Mensajes, tokens y contexto](curso/01-fundamentos/03-mensajes-tokens-y-contexto.md) | Qué recibe el modelo y qué límites tiene esa entrada |
| [04 · Tool calling](curso/01-fundamentos/04-tool-calling.md) | Cómo se solicitan, validan y ejecutan herramientas |
| [05 · Estado, sesiones y compaction](curso/01-fundamentos/05-estado-sesiones-y-compaction.md) | Ramas de conversación, persistencia y pérdida de detalles al resumir |
| [06 · Seguridad y confianza](curso/01-fundamentos/06-seguridad-y-confianza.md) | Permisos, aislamiento, prompt injection y extensiones |

El [mapa del curso](curso/00-mapa-del-curso.md) también recoge los bloques
**planificados, todavía sin capítulos publicados**: anatomía de PI, uso personal
y profesional, comparación entre agentes y construcción de un harness, con un
proyecto final.

## Cómo seguir el curso

1. Lee el capítulo y consulta el glosario cuando aparezca un término nuevo.
2. Realiza el laboratorio cuando lo haya y observa sus resultados.
3. Responde las preguntas de comprensión antes de consultar las soluciones o
   respuestas revisadas disponibles.
4. Si algo no encaja, vuelve a la evidencia: mensajes, archivos, eventos y permisos.

Puedes leer todo desde GitHub. Si prefieres trabajar con una copia local:

```bash
git clone https://github.com/raulb1zkit/ENTENDIENDO-PI.git
cd ENTENDIENDO-PI
```

## Qué necesitas para los laboratorios

Los ejemplos están preparados para **Linux y Bash**, con PI **1.3.0** como versión
de referencia. El entorno original usa Omarchy, pero no necesitas ese escritorio
para seguir los fundamentos.

Según el ejercicio se utilizan `jq`, `curl`, Python 3 o Git. Cada laboratorio
explica qué ejecutar; puedes empezar por los que funcionan sin modelo ni credenciales.

Los ejercicios de conversación real requieren PI y acceso a un proveedor de modelo,
y pueden consumir cuota. La lectura del curso y varios laboratorios locales se
pueden hacer sin ese acceso.

## Fuentes y versiones

Cada capítulo enlaza sus fuentes primarias. El [registro de documentación de PI](investigacion/documentacion-pi.md)
recoge la versión de referencia y el estado de estudio de las fuentes.

Las páginas oficiales marcadas como `latest` pueden cambiar respecto a PI 1.3.0.
Al contrastar un ejemplo, comprueba la versión: una diferencia entre herramientas
o versiones no implica que todas funcionen igual.

## Participa

Las dudas también ayudan a mejorar el curso. Puedes abrir una
[issue](https://github.com/raulb1zkit/ENTENDIENDO-PI/issues) para señalar un error,
un paso difícil de seguir o una explicación que falte, o enviar una pull request.

Consulta la [guía de contribución](CONTRIBUTING.md) para proponer cambios y aportar
evidencia reproducible sin publicar datos personales ni credenciales.

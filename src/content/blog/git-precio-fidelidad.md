---
title: "Git y el precio de la fidelidad"
description: "Por qué tu historial nunca es tan limpio como quisieras, qué lo causa, y cómo podría no ser así."
date: 2026-03-25
tags: ["git", "desarrollo", "open source"]
readingTime: 12
---

Hay una conversación que tarde o temprano tiene todo desarrollador que lleva suficiente tiempo trabajando con Git sobre la que les quiero platicar. No, no es la del merge conflict que arruinó el viernes — esa también, pero no es esa. Me refiero a la conversación sobre el historial.

Específicamente, sobre por qué `git log` en `main` siempre termina siendo un cementerio de `fix typo`, `wip`, `este fix es el bueno, ya de verdad ahora sí` y commits que no le importan a nadie excepto a quien los escribió a las 11 de la noche con una sobredosis de cafeína en las venas.

Hablemos pues sobre esa tensión: de dónde viene, por qué sigue existiendo, y cómo podría no existir. Y sí, al final hay una propuesta. Spoiler: no la voy a implementar yo, no estoy tan loco.

## Un fin de semana de 2005

Git no fue diseñado. Fue destilado.

En abril de 2005, Linus Torvalds — el mismo wey que se aventó el kernel de Linux — se quedó sin herramienta de control de versiones después de un conflicto con la que usaban. Y en lugar de ponerse a buscar alternativas *—como haría una persona sensata—*, simplemente se montó en su macho y escribió Git en aproximadamente diez días *—como haría cualquier desarrollador—*. Sus objetivos eran muy concretos: velocidad, que los datos no se corrompieran, soporte para trabajo distribuido, y que aguantara el volumen brutal que genera el desarrollo del kernel de Linux.

No había documento de diseño. No había equipo de producto. No hubo nadie en una sala de juntas dibujando en una pizarra y hablando de "la visión". Había un problema concreto y un ingeniero con mucho café y pocas ganas de usar lo que ya existía.

Lo fascinante es que la estructura interna que Linus diseñó ese fin de semana sigue siendo esencialmente la misma hoy, veinte años después (con sus retoques aquí y allá). Eso dice mucho de lo bien pensada que estaba. El problema — y eso explica muchas cosas — es que fue pensada para el workflow del kernel de Linux, y el mundo la adoptó para absolutamente todo lo demás sin cuestionarla ni siquiera tantito.

## Cómo funciona Git por dentro (For Dummies)

Antes de hablar del problema es importante entender, aunque sea a muuuuuy grandes rasgos, cómo es que Git guarda el historial. Prometo que no duele... mucho.

### Commits, ramas y el grafo

Cada vez que haces un `git commit`, Git crea lo que internamente llama un objeto — una especie de fotografía del estado de tu proyecto en ese momento. Esa fotografía tiene básicamente tres cosas: el contenido de tus archivos, tu mensaje, y una referencia a la fotografía anterior.

Esa referencia al commit anterior es lo que encadena todo. Imagínate una cadena de fotos donde cada una apunta a la que se tomó antes (como en la intro de Modern Family). Eso es el historial de Git.

Una rama — `main`, `dev`, lo que sea — no es más que un post-it que dice "el commit más reciente está aquí". Nada más. Cuando haces un nuevo commit, el post-it se mueve.

### Qué pasa cuando mergeas

Cuando tienes dos ramas y haces un merge, Git crea un commit especial que tiene dos referencias al pasado en lugar de una: una apunta al último commit de `main` y otra al último commit de la rama que estás integrando.

Eso crea la bifurcación visible que ves en los git graphs — el famoso "nudo". Y aquí empieza el problema: esas dos referencias son completamente iguales para Git. No hay distinción entre "padre principal" y "padre secundario". Son dos padres, los dos iguales, y cuando Git camina por el historial los sigue a los dos sin importarle cuál es cuál.

## Las tres estrategias y sus compromisos

Hoy existen tres formas principales de integrar una rama a `main`, y las tres te obligan a sacrificar algo. Es básicamente escoger qué dedo te cortas.

### Merge `--no-ff`

El merge clásico. Crea el nudo visible en el grafo. La rama mergeada queda conectada. Git sabe que ya fue integrada.

El costo: `git log main` te muestra todos los commits internos de la rama. Cada `fix typo`, cada `wip`, cada `arreglo de nuevo en serio ahora sí este es el bueno lo juro`. El historial es completo y fiel (no como tu ex), pero también es un desastre para leer.

```
main:  A───B───────────M
                      /
feat:      C───D───E─/
```

### Squash merge

Aplasta (de ahí el nombre) todos los commits de la rama en uno solo y lo pone sobre `main`. El historial queda limpio — un commit por feature, legible como changelog, bonito pues.

El costo: la conexión estructural desaparece. La rama queda "suelta" en el grafo, como si nunca hubiera sido mergeada. Git no la reconoce como integrada. Para borrarla tienes que forzar la eliminación porque Git te dice "oye, esto no ha sido mergeado" — aunque claramente sí lo fue. No puedes saber desde qué rama llegó ese commit (a menos que te pongas a leer el commit message, y tienes suerte de que solo fueron algunos commits y no decenas de ellos).

```
main:  A───B───S          ← S contiene todo, sin conexión visible
feat:      C───D───E      ← suelta, sin nudo
```

### Rebase + merge `--no-ff`

La idea es limpiar la rama antes de mergearla. El rebase toma todos tus commits y los "mueve" encima del último estado de `main`, como si la rama hubiera salido de ahí desde un principio. Luego el merge crea el nudo.

El costo: los commits se reescriben. Técnicamente son commits nuevos con contenido idéntico pero identidad diferente — como fotocopias que se ven iguales pero no son el original. Si la rama es tuya y solo tuya, no pasa nada. Si alguien más está trabajando en esa misma rama, te acabas de meter en un agujero del que va a ser difícil salir.

Tres estrategias, tres compromisos distintos, ninguna que te dé historial limpio y nudo visible y sin reescritura al mismo tiempo. Hay que escoger dos de tres — como cuando quieren que te programes algo bueno, rápido y barato.

## El workaround que ya existe: `--first-parent`

Git tiene una respuesta para esto y se llama `--first-parent`.

```bash
git log main --first-parent --oneline
```

Lo que hace es decirle a Git: "al caminar por el historial, solo sigue al primer padre de cada commit". Para un merge commit, el primer padre es siempre el de `main`. El resultado es exactamente la vista limpia que queremos.

```bash
# Con --first-parent ves esto:
M  merge: (feat) integra validación de crédito
B  sync: producción 2026-03-10
A  init

# Sin --first-parent ves esto:
M  merge: (feat) integra validación de crédito
E  fix: caso borde en monto negativo
D  fix typo en mensaje de error
C  wip: validación inicial
B  sync: producción 2026-03-10
A  init
```

Entonces ¿problema resuelto? ¿quién tiene hambre? Espérate tantito. No exactamente, mi querido Watson.

El problema es que esta vista limpia es opt-in (opcional, pues). El default es el historial ruidoso, y `--first-parent` es un flag que tienes que recordar tú (si es que sabes que existe), cada herramienta de CI que uses, cada script de changelog que tengas, cada cliente gráfico en el equipo — y tendrás que configurarlos por tu cuenta... Yay! Si alguien lo olvida, o si usas una herramienta que no lo conoce, vuelves al cementerio de commits.

Y el punto más incómodo: el intent existía en el momento del merge. Sabías que querías un punto de integración limpio. Pero esa información no quedó registrada en ningún lugar que el tooling pueda consumir automáticamente. Simplemente se perdió, *gone with the wind*.

## La propuesta: `symbolic-parent`

Todo lo anterior me llevó a preguntarme: ¿y si el problema no es de visualización sino de modelo de datos? ¿Y si en lugar de arreglarlo al leer, lo arreglamos al escribir? ¿Pa'qué complicarnos la existencia solitos?

Y surgió una idea, y hasta consideré ponerme a implementarla: la idea es introducir un nuevo tipo de referencia en el commit: `symbolic-parent`.

En lugar de dos padres iguales, tendríamos esto:

```
parent <hash>           ← primer padre (main) — se sigue siempre
symbolic-parent <hash>  ← segundo padre (la rama) — solo para el grafo
```

Un `symbolic-parent` le diría a Git: *"esta rama estuvo aquí, este fue el punto de integración, de ahí viene, pero no lo sigas al caminar por el historial por defecto"*.

El resultado en práctica:

| Comando | Comportamiento |
|---|---|
| `git log` | Limpio — ignora el `symbolic-parent` |
| `git log --graph` | Muestra el nudo — dibuja ambos padres |
| `git log --full-history` | Muestra todo — sigue ambos padres |
| `git branch --merged` | Reconoce la rama como mergeada |
| `git bisect` | Limpio — ignora el `symbolic-parent` por defecto |

Y el comando sería tan simple como:

```bash
git merge --symbolic feat
```

Con esto, `git log main` estaría limpio por defecto. El grafo mostraría el nudo. Git reconocería la rama como mergeada. Sin flags opt-in. Sin reescritura de historial. Sin compromisos. Fidelidad y legibilidad al mismo tiempo, que, seamos sinceros, la neta no son conceptos opuestos por naturaleza — solo los tratamos así porque así nos los heredaron.

## Por qué no existe (y por qué tiene sentido que no exista, aunque me duela)

El modelo de objetos de Git tiene una regla invariante muy fuerte desde 2005: todos los padres son iguales. No hay padres de primera y padres de segunda. Esa igualdad simplifica enormemente el razonamiento sobre el historial — todas las herramientas que operan sobre el grafo asumen que si algo es un padre, es un padre real con todas sus consecuencias (no como el Brayan de la Kimberly).

Introducir `symbolic-parent` rompería esa regla y generaría preguntas muy incómodas: ¿cómo se comporta `git revert` sobre un merge con `symbolic-parent`? ¿Cómo calcula la "base común" entre dos ramas cuando hay padres de distintas clases? ¿Qué le pasa a alguien con una versión vieja de Git que encuentra un `symbolic-parent` en el repo — ve un merge normal, o truena?

Ese último punto es el más serio. El mismo repositorio se comportaría diferente dependiendo de la versión del cliente. Lo cual claramente no es ideal, y precisamente Git ha evitado históricamente cualquier situación donde el historial sea ambiguo entre versiones, y tiene sentido.

Hay también una decisión filosófica detrás: Git deliberadamente no quiere privilegiar una interpretación del historial sobre otra. El argumento es que distintos equipos requieren distintas vistas — el equipo de release quiere lo limpio, el arqueólogo de código quiere el grafo completo, el bisect quiere todos los commits. El modelo neutro deja que cada quien elija.

Es una posición coherente, si te pones a pensarlo hasta tiene sentido. Pero pone la carga en el consumidor, no en el productor (*o sea que chin... a mi madre*). El desarrollador que hizo el merge sabía exactamente lo que quería. Esa información — o intención — se perdió. Y ahora cada herramienta tiene que reconstruirla con flags o simplemente adivinar.

Eso también, hay que decirlo, es en parte consecuencia de que Git fue diseñado para el kernel de Linux. Linus nunca necesitó que `git log` fuera legible como changelog para un equipo de producto — necesitaba trazabilidad absoluta de parches, y seamos sinceros, en ese fin de semana quizá ni le pasó por la cabeza. El mundo adoptó la herramienta con su filosofía incluida, y veinte años después hay decisiones de un fin de semana de 2005 que son prácticamente inamovibles porque todo el ecosistema se construyó sobre ellas.

## Las alternativas que sí son aplicables hoy

Si `symbolic-parent` no va a llegar a Git core, ¿qué opciones quedan que no sean un simulacro?

### Trailers estructurados en el commit message

Git permite poner metadatos al final de un mensaje de commit en formato `Clave: valor`. A eso, amigos míos, les llamamos trailers. Ya se usan para cosas como `Signed-off-by` o `Co-authored-by`. Y nada impide definir una convención propia y usarla de forma consistente:

```
merge: (feat) integra validación de crédito

Merge-Type: symbolic
Source-Branch: feat/validacion-credito
Source-Tip: abc1234def5678
```

No cambia el modelo de objetos, no afecta la compatibilidad con nada, y la información queda registrada para cualquier herramienta que quiera leerla.

El pero: un trailer es solo texto hasta que alguien lo interpreta. Si ninguna herramienta popular lo lee, es documentación sofisticada. Útil, pero no resuelve el problema del default ruidoso.

### Una herramienta externa encima de Git

Un CLI que viva encima de Git, que imponga la convención de trailers en cada merge, que tenga su propio comando de log que lea esos trailers y presente la vista limpia por defecto, y que genere changelogs automáticos desde esa metadata. No requiere tocar Git, no hay problemas de compatibilidad, podría empezar como un script de 200 líneas. ¡ES TAN SIMPLE Y HERMOSO! ¿Qué podría malir sal?

Ah sí, solo funciona mientras todo el equipo utilice la herramienta. En el momento que alguien hace `git merge` directamente sin pasar por la herramienta, la convención se rompe y valió.

### Configurar el cliente gráfico

Para el problema específico de visualización, herramientas como Git Graph en VS Code ya ofrecen esto:

```json
"git-graph.onlyFollowFirstParent": true
```

Es la opción más rápida para uso individual o de equipo si todos usan el mismo cliente. Pero solo arregla la vista en esa herramienta específica. `git log` en terminal sigue igual de ruidoso.

## El sistema de versionado ideal: el What if?

¿Y si pudiera diseñar un sistema de versionado desde cero hoy, con todo lo que sabemos después de veinte años de Git, cómo sería?

Mantendría el modelo de objetos casi intacto — el grafo de commits inmutables es genuinamente brillante y no hay mucho que mejorarle. Pero cambiaría algunas cosas.

**Padres con semántica.** El commit soportaría distintos tipos de padre con comportamiento distinto al traversar el historial por defecto. El `symbolic-parent` que describí, pero con espacio para otros tipos según el caso de uso. La semántica estaría en el dato, no en el flag que pasas al momento de leer.

**Defaults sensatos.** `git log` en una rama de integración mostraría la vista limpia por defecto. Para ver el historial completo, un flag explícito. El ruido sería la opción, no el default. Porque los defaults importan más que las opciones avanzadas en la práctica real — la mayoría de la gente nunca cambia los defaults.

**Intención de merge como dato de primera clase.** No en texto libre en el mensaje sino en campos estructurados del objeto, parseables sin regex. El tipo de merge, la rama de origen, el estado al momento de integrar. Metadata que cualquier herramienta pueda leer sin andar de adivino.

**Interfaz humana separada del modelo.** Git mezcla los comandos de bajo nivel (pipe) con los de uso diario (porcelain) de una forma que la línea entre ambos nunca fue del todo clara. El ejemplo clásico: `git checkout` hacía cuatro cosas distintas hasta que en 2019 lo separaron en `git switch` y `git restore`. En el sistema ideal esa separación sería total y explícita desde el día uno. Los comandos de bajo nivel exponen el modelo. Los de alto nivel toman decisiones con defaults razonables. Nunca se mezclan.

**Versionado del formato desde el inicio.** No como extensión retroactiva sino como parte fundamental del diseño. Cada cliente declara qué versiones de objeto entiende. El sistema sirve en consecuencia. Sin esa zona gris de "a ver qué hace un cliente viejo con esto".

## El elefante en la habitación

Nada de esto que propongo va a pasar. No porque sea técnicamente imposible — todo lo que describí es perfectamente implementable (mucha chamba, pero posible). Sino porque Git tiene veinte años de adopción universal, un ecosistema entero de herramientas que asume el modelo actual, y una comunidad de desarrollo que es extremadamente conservadora con cambios al modelo de objetos, por razones muy legítimas, y hasta eso, entendibles.

Pero es importante que dejemos algo en claro. Git no ganó porque sea el mejor diseño posible. Ganó porque GitHub llegó en 2008 y lo convirtió en infraestructura social antes de que el debate técnico pudiera tener lugar. En ese momento el debate terminó.

Y eso está bien. "Suficientemente bueno con adopción universal" le gana a "mejor diseño pero lo ocupo yo y mi compa aún más nerd" aquí y en China, no hay mucho que debatir ahí.

Pero vale la pena nombrar la diferencia entre *"así fue diseñado"* y *"así debe ser"*. La fidelidad histórica y la legibilidad del historial no son mutuamente excluyentes. El hecho de que Git las trate como conceptos en tensión es una decisión heredada de un contexto muy específico — el kernel de Linux en 2005 — y que el mundo adoptó sin cuestionarla demasiado.

Así que quizás alguien, en algún momento, construya algo mejor. Los ingredientes están ahí y la necesidad existe. Porque la neta yo no lo voy a hacer — pero si lo hacen, me avisan.

*Por cierto.... ¿Usan alguna estrategia de merge que no mencioné? ¿Ya encontraron el anillo único "to rule'em all" que yo no vi? Me interesa saberlo, a lo mejor, solo es mi OCD hablando.*
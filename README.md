# skill-detector-control-negativo

**Tu suite de pruebas está verde. La pregunta que nadie hace es si esa suite sabría ponerse roja.
Esta skill cuenta cuántos bancos de tu repositorio comprueban solo que lo bueno pasa, dice cuáles
y por qué. Y se puntúa a sí misma contra un conjunto etiquetado a mano antes de opinar de nada.**

![Python 3.9+](https://img.shields.io/badge/python-3.9%2B-blue)
![sin dependencias](https://img.shields.io/badge/dependencias-ninguna-brightgreen)
![38 casos](https://img.shields.io/badge/banco-38%20casos-brightgreen)
![mutación 6/6](https://img.shields.io/badge/mutaci%C3%B3n-6%2F6-brightgreen)
![licencia MIT](https://img.shields.io/badge/licencia-MIT-lightgrey)

```
/plugin marketplace add jleonceo/skill-detector-control-negativo
/plugin install detector-control-negativo@skill-detector-control-negativo
```

Esto es lo que devuelve: las **32 líneas** que imprime el comando, sin quitar ninguna. Sale del
ejemplo que viaja dentro del repositorio, así que se reproduce con `python
skills/detector-control-negativo/verificar_control_negativo.py --etiquetas
ejemplo/etiquetas_ejemplo.yaml --listar`, y se cuenta con `| wc -l`:

```
==============================================================================
BANCOS SIN CONTROL NEGATIVO
==============================================================================
  bancos con casos propios .. 8
  agregadores ............... 1  (lanzan otros bancos; no tienen casos que juzgar)
  excluidos a proposito ..... 0  (no se ha pasado --excluir)
  criterio del universo ..... versionado en git + nombre test_*/run_tests*/*_test

  con control negativo ...... 6
  SIN control negativo ...... 2
  senal que dispara ......... MARCA 2 | ASSERT_NEG 2 | ESPERA_CERO 2 | NOMBRE_NEG 3 | ETIQUETA_NEG 2

  -- SIN control negativo (revisar a mano antes de creerselo) --
     ejemplo/bancos/test_prosa_con_no_y_sin.py
     ejemplo/bancos/test_solo_positivo.py

  -- con control negativo, y por que --
     ejemplo/bancos/test_assert_negativo.py                     ASSERT_NEG,NOMBRE_NEG
     ejemplo/bancos/test_espera_cero.py                         ESPERA_CERO
     ejemplo/bancos/test_marca_cn.py                            MARCA
     ejemplo/bancos/test_nombre_negativo_aislado.py             NOMBRE_NEG
     ejemplo/bancos/test_veredicto_malo.py                      ETIQUETA_NEG
     skills/detector-control-negativo/test_detector_control_negativo.py MARCA,ASSERT_NEG,ESPERA_CERO,NOMBRE_NEG,ETIQUETA_NEG

------------------------------------------------------------------------------
  EL DETECTOR CONTRA LAS ETIQUETAS  (9 etiquetadas de 9 bancos)
    concuerda ............... 9 de 9

  >> El detector concuerda con todas las etiquetas.
     Bancos sin control negativo hoy: 2 de 8.
     La cifra se REPORTA y no tumba nada: en el arbol donde nacio bajo de 112 a
     ~10 en cinco pasadas, o sea que actuar sobre ella sin anclarla sale caro.
```

Fíjate en las dos últimas líneas antes de fijarte en el recuento: **el código de salida habla del
detector y no del repositorio.** Sale con `0` cuando concuerda con las etiquetas. El número de
bancos se reporta sin tumbar nada. Esa advertencia la imprime el propio programa, en cada ejecución,
para que no dependa de que alguien se acuerde de escribirla. El porqué está más abajo y es la
decisión que da forma al resto.

**Y el «2 de 8» no describe un repositorio de verdad: los ocho bancos de `ejemplo/bancos/` están
plantados a propósito, uno a uno, con su etiqueta al lado.** Dos sin brazo negativo, que son los
dos que el informe señala; cinco con brazo negativo, uno por cada señal; y un agregador, para el
tercer valor de la etiqueta. El noveno banco del universo es el de la propia skill. Es un árbol de
juguete, construido para que cada veredicto se pueda comprobar leyendo el fichero. Sobre tu
repositorio el reparto será otro.

**Eso tiene una consecuencia práctica que conviene saber antes de clonar:** esas fixtures son el
material que el detector MIDE, así que varias invocan símbolos que no existen a propósito y se
caerían si alguien las ejecutara como pruebas. Se llaman `test_*.py` porque el criterio del detector
es el nombre del fichero, y renombrarlas las sacaría de su propia medición. La exclusión vive en
`conftest.py`, con el motivo escrito dentro. Hoy `python -m pytest -q` sobre este repositorio
recoge las 38 pruebas del banco de la skill y sale verde sin tocar una sola fixture.

Este repositorio es solo el paquete instalable. La medición sobre un árbol real, con las cifras,
el método y las cuatro mutaciones que se le pasaron al propio detector, vive aparte:
[jleonceo/detector-control-negativo](https://github.com/jleonceo/detector-control-negativo).

[Español](#español) · [English](#english)

---

## Español

### Qué hace

Cuenta bancos y dice por qué cada uno.

A un banco que solo comprueba el camino bueno se le pasa una entrada correcta, se comprueba que
sale el resultado correcto y ahí acaba. Ese banco da verde y sigue dando verde cuando alguien
rompe la validación de entrada, porque nunca se le pregunta qué hace con una entrada mala. Le
falta el **control negativo**: el caso que afirma lo que la herramienta debe rechazar, bloquear,
dejar vacío o marcar como inválido.

La skill recorre los ficheros de test que git publica en tu repositorio, busca cinco formas de
brazo negativo y devuelve cuántos bancos no tienen ninguna.

Eso es todo. Sin modelo de lenguaje de por medio, sin red, sin dependencias y sin escribir en
ningún sitio.

### Instalación

Tres vías. La primera es la corta y la última funciona siempre.

```bash
# 1. Como plugin de Claude Code
/plugin marketplace add jleonceo/skill-detector-control-negativo
/plugin install detector-control-negativo@skill-detector-control-negativo

# 2. Copiando la carpeta a tus skills personales
git clone https://github.com/jleonceo/skill-detector-control-negativo
cp -r skill-detector-control-negativo/skills/detector-control-negativo ~/.claude/skills/

# 3. Sin instalar nada, ejecutando el script directamente
python skills/detector-control-negativo/verificar_control_negativo.py --raiz .
```

El ejemplo de `ejemplo/` se ejecuta sin que tengas que preparar nada y es de donde sale el bloque
de salida del principio de esta página:

```bash
python skills/detector-control-negativo/verificar_control_negativo.py \
  --etiquetas ejemplo/etiquetas_ejemplo.yaml --listar
```

La tercera vía existe porque el instrumento no necesita a Claude Code. Es un script de biblioteca
estándar que le pregunta el universo a `git ls-files`. Si lo único que quieres es el número, te lo
da sin tocar tu configuración.

### Uso

```bash
cd tu-repositorio
python ~/.claude/skills/detector-control-negativo/verificar_control_negativo.py --etiquetar
```

**La primera ejecución en un repositorio nuevo sale con `2`, y está bien que lo haga.** Todavía no
hay etiquetas. Sin ellas el recuento es una aproximación sin puntuar. El camino es
`--etiquetar`, que imprime la plantilla YAML de los bancos que aún no tienen etiqueta, abrir a
mano los ficheros que salgan y escribir la etiqueta con la línea que la demuestra. Ese rato es el
trabajo y no hay atajo: medir un banco obliga a leerlo.

Estas son las cinco opciones, que son todas:

| Opción | Para qué |
|---|---|
| `--raiz RUTA` | el árbol a medir; por defecto, la raíz git del directorio actual |
| `--etiquetas FICHERO` | la verdad de referencia con la que se puntúa el detector |
| `--excluir SUBCADENA` | deja fuera las rutas que la contengan; repetible y se cuentan aparte |
| `--listar` | cada banco con su veredicto y cada veredicto con la señal que lo disparó |
| `--etiquetar` | plantilla YAML de los bancos que aún no tienen etiqueta |

La primera vez, `--listar`.

Esa vista es la que evita el malentendido más caro. La skill reconoce el brazo negativo por
formas, y las de tu proyecto no tienen por qué parecerse a las de nadie: si tu banco envuelve la
aserción en un helper propio con otro nombre, o marca sus casos negativos con una convención
distinta, `--listar` lo enseña de un vistazo. Cada banco cubierto sale con la señal que lo
disparó, así que se ve si el criterio está reconociendo lo que tú entiendes por brazo negativo o
está contando otra cosa.

### El fichero de etiquetas

```yaml
etiquetas:
  - banco: tests/test_parser.py
    tiene_control_negativo: true
    evidencia: "self.assertRaises(ValueError) en test_rechaza_cabecera_vacia"
    etiquetado_el: 2026-07-30
```

Con eso, una versión nueva del criterio **ya no se juzga por dar un número más pequeño**: se
puntúa. Falsos positivos son los que señala y sí tenían brazo negativo; falsos negativos, los que
da por buenos y no lo tienen. Un criterio que baja el titular subiendo los falsos negativos es
peor, aunque el titular quede mejor.

**Y la etiqueta se escribe antes de tocar el criterio.** Al revés, la etiqueta se acomoda al
instrumento y la medición deja de valer. Si el detector discrepa de una etiqueta tuya, lo que se
arregla es el criterio.

`tiene_control_negativo` tiene un tercer valor, `agregador`, que no es un adorno: hay ficheros que
se llaman como un banco y no tienen un solo caso propio, porque descubren otros y los lanzan.
Contarlos entre los carentes es contar el objeto equivocado.

### Los códigos de salida

| salida | qué significa |
|---|---|
| `0` | el detector concuerda con todas las etiquetas |
| `1` | discrepa de alguna: hay un falso positivo o un falso negativo |
| `2` | no hay etiquetas con las que medirse, así que no se puntúa |
| `3` | no hay universo: ninguna raíz de git, o cero bancos encontrados |

**Ninguno habla del estado de tu repositorio. Es deliberado.** El primer intento de medir esto
dio 112 bancos carentes, luego 77, luego 48, luego 31, y una revisión a mano de tres casos dejó la
cifra de verdad en unos diez. Ninguna versión estaba mal programada: cada afinado medía un objeto
distinto. Actuar sobre una cifra que solo baja conforme se mira mejor es el error que esta
herramienta existe para no repetir, así que el recuento se reporta y quien quiera un umbral que
bloquee lo pone él, sabiendo lo que hace.

El `3` es igual de deliberado. Fuera de un repositorio de git no hay universo, y decir «0 bancos
sin control negativo» ahí sería el falso verde de manual: el mensaje más tranquilizador posible
producido por no haber mirado nada.

### Qué no hace

- No escribe. Ni en tus bancos, ni en tu configuración, ni fuera de la carpeta desde la que se
  ejecuta. No propone parches ni arregla nada.
- No dice si un banco es bueno. Tener una aserción negativa no prueba que discrimine: un banco
  puede llevar `assertFalse` y seguir pasando con el código roto. Eso lo dice mutar el código.
  El paquete trae `mutar.py` para hacérselo a sí mismo.
- No cuenta lo que git no publica. El universo sale de `git ls-files`, así que un banco del
  árbol de trabajo que nadie ha añadido queda fuera. Es un suelo declarado.
- No entra por un nombre que no parezca un banco. El criterio es `test_*.py`,
  `run_tests*.py` o `*_test.py`. Otro esquema de nombres necesita otro criterio.

### Desinstalar

```bash
# si la instalaste como plugin: son DOS comandos, no uno
/plugin uninstall detector-control-negativo@skill-detector-control-negativo
/plugin marketplace remove skill-detector-control-negativo

# si copiaste la carpeta
rm -rf ~/.claude/skills/detector-control-negativo
```

El segundo comando es el que casi nadie documenta. Desinstalar el plugin lo quita de la lista de
plugins, pero deja el marketplace registrado y su copia local en la caché que mantiene Claude Code
en `~/.claude/plugins/cache`, que es suya y no de esta skill.

De la skill en sí no queda nada, porque no escribe nada: ni base de datos, ni fichero de estado,
ni configuración fuera de su carpeta.

### Por qué fiarse

**38 casos. Siete de ellos son controles negativos marcados `CN`.** En un banco cuyo objeto es
cazar bancos sin control negativo, no tenerlos sería la ironía más cara del paquete.

```bash
python skills/detector-control-negativo/test_detector_control_negativo.py
python skills/detector-control-negativo/mutar.py
```

El segundo comando es el que da derecho a presumir del primero. Sabotea el detector a propósito
con **seis sabotajes** y cuenta cuántos caza el banco. Hoy caza los seis sin un hueco. No hay que
creérselo: se ejecuta y se ve. El fichero medido queda restaurado al terminar y también si el
proceso muere a mitad, porque la copia intacta vive en disco y no en la memoria del programa.

Eso hizo falta. Antes de publicar se le pasaron cuatro mutaciones al detector y **dos no rompieron
ni un test**: borrar el literal `_CN_` del reconocimiento de la marca y borrar los nombres `_no_`
y `_sin_` de la señal de nombre salían limpias, porque cada una dependía en exclusiva de un único
banco que no estaba etiquetado. Un banco de 24 casos, verde, con dos líneas del criterio que
cualquiera podía borrar sin que nadie se enterase. Se cerró con dos casos que aíslan cada forma.
El banco pasó de 24 a 26. Los cuatro que faltan hasta 30 los añadió este empaquetado y los dos
últimos, hasta 32, entraron al vigilar el censo de fixtures y el mutador. Cubren lo
que antes no existía o estaba clavado dentro del script: que los bancos plantados a propósito se
cacen, que ningún banco del repositorio se quede sin etiqueta, que la raíz de git no se invente
cuando no hay ninguna y que la exclusión declarada saque las rutas y las cuente aparte.

**Los seis últimos, hasta los 38 de hoy, los trajo una auditoría externa el día de publicar**, con
el banco en 32, la mutación en 6 de 6 y el acuerdo con las etiquetas en 9 de 9. Buscó con material
que no estaba en el banco y encontró dos defectos que ninguno de esos tres verdes podía ver: la
señal de nombre casaba por subcadena, así que «protocolo» disparaba por contener «roto», y el lector
de etiquetas convertía en `false` cualquier valor que no entendiera, incluida su propia plantilla
sin rellenar. La lección va escrita porque cuesta verla: la mutación solo vigila las líneas que ya
existen, y el acuerdo se mide contra un árbol de juguete sin un solo nombre adversarial ni una
etiqueta mal escrita. Un arnés hereda el punto ciego del corpus que lo alimenta.

Y el mutador **falla cerrado**: si el ancla de un sabotaje ya no aparece en el fichero, esa
mutación no se aplica y, en vez de contarla como cazada, sale con ERROR y tumba la ejecución. Un
arnés que aprueba sin haber mordido no avala nada.

### Requisitos

Python 3.9 o superior, biblioteca estándar, sin red y sin nada que instalar. Hace falta `git` en
el PATH, porque el universo de bancos se le pregunta a él. En macOS y en casi todo Linux el
intérprete se llama `python3`, no `python`: los comandos de esta página van con `python` porque se
escribieron en Windows.

Ese «sin red» es de la skill. Hay que apurarlo porque el repositorio no cumple lo mismo que ella.
`verificar_control_negativo.py` y `mutar.py` no abren un socket: lo que necesitan se lo preguntan a
`git` en local. El que sí sale a la red es `guardian_gemelo.py`, en la raíz del repositorio, que
consulta `api.github.com` para comparar este paquete con su gemelo. No se instala con la skill y no
participa en ninguna medición.

Ese 3.9 está **declarado y todavía no certificado**. En local se ha ejecutado con Python 3.10, 3.13
y 3.14, las tres sobre Windows, pero en ninguna versión por debajo de 3.10, que es la que importaba.
Un repaso del código no encontró sintaxis ni biblioteca por encima de 3.9, pero eso
encuentra incompatibilidades y no certifica compatibilidad. Lo que la certifica es la matriz de CI
(3.9, 3.11 y 3.13 sobre Windows, Linux y macOS) dando verde, y hasta el primer push no lo ha dado.
Cuando lo dé, esta nota se corrige y no se borra.

### Piezas hermanas

- **[detector-control-negativo](https://github.com/jleonceo/detector-control-negativo)**: el mismo
  instrumento con el método, las cifras sobre un árbol real y la especificación que se aceptó
  antes de escribir el código.
- **[skill-adherencia-reglas](https://github.com/jleonceo/skill-adherencia-reglas)**: la otra skill
  del mismo laboratorio. Cuenta qué fracción de las reglas que escribiste para tu agente se cumple
  de verdad.
- **[guardianes-verificados-ia](https://github.com/jleonceo/guardianes-verificados-ia)**: quién
  vigila a los guardianes. Un detector que encuentra el problema y no devuelve el código de salida
  que bloquea no protege nada.

---

## English

> **Read this first: the tool speaks Spanish, and this page does not change that.** Every line it
> prints, the five signal names (`MARCA`, `ASSERT_NEG`, `ESPERA_CERO`, `NOMBRE_NEG`,
> `ETIQUETA_NEG`), the five flags and the three YAML keys (`banco`, `tiene_control_negativo`,
> `evidencia`) are Spanish. The label value `agregador` means «aggregator». This page is the
> reference for that vocabulary, and not a localised program.

### What it does

It counts benches, and says why for each one.

A bench that only checks the happy path is fed a valid input, asserts the correct result, and
stops there. It is green. It stays green after somebody breaks input validation, because it is
never asked what happens with a bad input. What it lacks is the **negative control**: the case
asserting what the tool must reject, block, leave empty or flag as invalid.

The skill walks the test files git publishes in your repository, looks for five shapes of negative
arm and returns how many benches have none.

That is all. No language model involved, no network, no dependencies, and it writes nowhere.

### Install

Three ways. The first is the short one and the last always works.

```bash
# 1. As a Claude Code plugin
/plugin marketplace add jleonceo/skill-detector-control-negativo
/plugin install detector-control-negativo@skill-detector-control-negativo

# 2. By copying the folder into your personal skills
git clone https://github.com/jleonceo/skill-detector-control-negativo
cp -r skill-detector-control-negativo/skills/detector-control-negativo ~/.claude/skills/

# 3. Without installing anything, running the script directly
python skills/detector-control-negativo/verificar_control_negativo.py --raiz .
```

The example in `ejemplo/` runs with nothing to prepare. It is where the output block at the top of
this page comes from, all 32 lines of it with none removed:

```bash
python skills/detector-control-negativo/verificar_control_negativo.py \
  --etiquetas ejemplo/etiquetas_ejemplo.yaml --listar
```

**That «2 of 8» describes no real repository: the eight benches in `ejemplo/bancos/` are planted on
purpose, one by one, each with its label beside it.** Two with no negative arm, which are the two
the report flags; five with one, one per signal; and one aggregator, for the label's third value.
The ninth bench of that universe is the skill's own. It is a toy tree, built so every verdict can be
checked by reading the file, and on your repository the split will be a different one.

**That carries a practical consequence worth knowing before you clone.** Those fixtures are the
material the detector MEASURES, so several of them call symbols that do not exist on purpose and
would fail if anybody ran them as tests. They are named `test_*.py` because the detector's criterion
is the filename, and renaming them would drop them out of their own measurement. The exclusion lives
in `conftest.py` with the reason written inside, and today `python -m pytest -q` over this
repository collects 38 tests, all 38 from the skill's own bench, and comes out green without
touching a single fixture.

The third way exists because the instrument does not need Claude Code. It is a standard library
script that asks `git ls-files` for its universe. If all you want is the number, it hands it over
without touching your configuration.

### Usage

```bash
cd your-repository
python ~/.claude/skills/detector-control-negativo/verificar_control_negativo.py --etiquetar
```

**The first run on a new repository exits with `2`, and rightly so.** There are no labels yet, and
without them the count is an unscored approximation. The path is `--etiquetar`, which prints the
YAML template for the benches with no label, then opening those files by hand and writing the
label with the line that proves it. That sitting is the work and there is no shortcut: measuring a
bench means reading it.

These are the five options, which is all of them:

| Option | What for |
|---|---|
| `--raiz PATH` | the tree to measure; by default, the git root of the current directory |
| `--etiquetas FILE` | the reference truth the detector is scored against |
| `--excluir SUBSTRING` | drops paths containing it; repeatable, and counted separately |
| `--listar` | every bench with its verdict, and every verdict with the signal that fired |
| `--etiquetar` | a YAML template for the benches that have no label yet |

Start with `--listar`.

That view heads off the most expensive misunderstanding. The skill recognises the negative arm by
shape, and your project's shapes need not resemble anyone else's: if your bench wraps its
assertion in a helper of its own, or marks its negative cases by a different convention,
`--listar` shows it at a glance. Every covered bench comes out with the signal that fired, so you
can see whether the criterion is recognising what you mean by a negative arm or counting something
else.

### The label file

```yaml
etiquetas:
  - banco: tests/test_parser.py                # bench
    tiene_control_negativo: true               # has a negative control
    evidencia: "self.assertRaises(ValueError) in test_rechaza_cabecera_vacia"
    etiquetado_el: 2026-07-30                  # labelled on
```

With that, a new version of the criterion **stops being judged by returning a smaller number** and
gets scored instead. False positives are the ones it flags that did have a negative arm; false
negatives, the ones it waves through that did not. A criterion that lowers the headline by raising
false negatives is worse, however much better the headline reads.

**And the label is written before the criterion is touched.** The other way round, the label
accommodates the instrument and the measurement stops meaning anything. If the detector disagrees
with a label of yours, what gets fixed is the criterion.

`tiene_control_negativo` takes a third value, `agregador`, which is not decoration: some files are
named like a bench and hold no case of their own, because they discover other benches and launch
them. Counting those among the uncovered counts the wrong object.

### Exit codes

| exit | meaning |
|---|---|
| `0` | the detector agrees with every label |
| `1` | it disagrees with one: there is a false positive or a false negative |
| `2` | there are no labels to be scored against, so no score is invented |
| `3` | no universe: no git root, or zero benches discovered |

**None of them talks about the state of your repository. That is deliberate.** The first
attempt at measuring this returned 112 uncovered benches, then 77, then 48, then 31, and a manual
review of three cases put the real figure at about ten. No version was miscoded: each refinement
measured a different object. Acting on a figure that only falls as you look harder is the mistake
this tool exists to avoid, so the count is reported and anyone wanting a blocking threshold sets
it themselves, knowing what they are doing.

The `3` is just as deliberate. Outside a git repository there is no universe, and saying «0
benches without a negative control» there would be the textbook false green: the most reassuring
message available, produced by having looked at nothing.

### What it does not do

- It does not write. Not to your benches, not to your configuration, not outside the folder it
  runs from. It proposes no patches and fixes nothing.
- It does not say whether a bench is good. Having a negative assertion does not prove it
  discriminates: a bench can carry `assertFalse` and still pass with broken code. Mutation says
  that, and the package ships `mutar.py` to do it to itself.
- It does not count what git does not publish. The universe comes from `git ls-files`, so a
  bench in the working tree that nobody added stays out. A declared floor.
- It does not take in a name that does not look like a bench. The criterion is `test_*.py`,
  `run_tests*.py` or `*_test.py`. Another naming scheme needs another criterion.

### Uninstall

```bash
# if you installed it as a plugin: that is TWO commands, not one
/plugin uninstall detector-control-negativo@skill-detector-control-negativo
/plugin marketplace remove skill-detector-control-negativo

# if you copied the folder
rm -rf ~/.claude/skills/detector-control-negativo
```

The second command is the one almost nobody documents. Uninstalling the plugin drops it from the
plugin list, but leaves the marketplace registered and its local copy in the cache Claude Code
keeps at `~/.claude/plugins/cache`, which belongs to Claude Code and not to this skill.

Of the skill itself nothing is left, because it writes nothing: no database, no state file, no
configuration outside its own folder.

### Why trust it

**38 cases, seven of them negative controls marked `CN`.** In a bench whose whole purpose is
catching benches without negative controls, having none would be the most expensive irony in the
package.

```bash
python skills/detector-control-negativo/test_detector_control_negativo.py
python skills/detector-control-negativo/mutar.py
```

The second command is what earns the right to boast about the first. It sabotages the detector on
purpose, **six sabotages**, and counts how many the bench catches. Today it catches all six with
no gaps. You do not have to take that on faith: you run it and watch. The measured file is
restored when it finishes and also if the process is killed halfway, because the intact copy lives
on disk and not in the program's memory.

It was needed. Before publishing, four mutations were run against the detector and **two broke not
one test**: deleting the `_CN_` literal from marker recognition and deleting the `_no_` and `_sin_`
names from the name signal came out clean, because each depended exclusively on a single bench that
was not labelled. A 24-case bench, green, and two lines of the criterion anyone could delete with
nobody noticing. It was closed with two cases isolating each shape, taking the bench from 24 to 26.
The four more up to 30 were added by this packaging, and they cover what did not exist before or sat
nailed inside the script: that the deliberately planted benches get caught, that no bench of the
repository goes unlabelled, that the git root is not invented where there is none, and that the
declared exclusion drops the paths and counts them separately. Two more, up to 32, came in while
watching the fixture census and the mutator.

**The last six, up to today's 38, were brought in by an outside audit on publication day**, with the
bench at 32, mutation at 6 of 6 and label agreement at 9 of 9. It looked with material that was not
in the bench and found two defects none of those three greens could see: the name signal matched on
substrings, so the Spanish «protocolo» fired by containing «roto» (broken), and the label reader
turned any value it did not understand into `false`, its own unfilled template included. The lesson
is written down because it is hard to see: mutation only guards lines that already exist, and
agreement is measured against a toy tree without a single adversarial name or a single malformed
label. A harness inherits the blind spot of the corpus that feeds it.

And the mutator **fails closed**: if a sabotage's anchor no longer appears in the file, that
mutation is not applied, and instead of counting it as caught the run exits with ERROR. A harness
that passes without having bitten vouches for nothing.

### Requirements

Python 3.9 or newer, standard library, no network and nothing to install. `git` must be on the
PATH, because the bench universe is asked of it. On macOS and most Linux the interpreter is
`python3`, not `python`: the commands on this page say `python` because they were written on
Windows.

That «no network» belongs to the skill. It needs tightening, because the repository does not meet the
same claim. `verificar_control_negativo.py` and `mutar.py` open no socket: what they need they
ask of local `git`. The one that does go out is `guardian_gemelo.py`, at the repository root, which
queries `api.github.com` to compare this package against its twin. It is not installed with the
skill and it takes part in no measurement.

That 3.9 is **declared and not yet certified**. Locally it has been run with Python 3.10, 3.13 and
3.14, all three on Windows, and on nothing below 3.10, which is the version that mattered.
A read-through found no syntax or library above 3.9, but that finds incompatibilities and
does not certify compatibility. What certifies it is the CI matrix (3.9, 3.11 and 3.13 across
Windows, Linux and macOS) going green, and until the first push it has not. When it does, this note
gets corrected and not deleted.

### Sibling repositories

- **[detector-control-negativo](https://github.com/jleonceo/detector-control-negativo)**: the same
  instrument with the method, the figures over a real tree and the specification accepted before a
  line of code was written.
- **[skill-adherencia-reglas](https://github.com/jleonceo/skill-adherencia-reglas)**: the other
  skill from the same workshop. It counts what fraction of the rules you wrote for your agent
  actually gets followed.
- **[guardianes-verificados-ia](https://github.com/jleonceo/guardianes-verificados-ia)**: who
  guards the guards. A detector that finds the problem and fails to return the exit code that
  blocks protects nothing.

---

## Licencia / License

MIT. Ver [LICENSE](LICENSE).

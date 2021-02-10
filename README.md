# SpaCy v.3.0 - Viele Tolle Neuerungen

Dies ist ein Projekt mit dem Ziel neue Features in [spacy v3.0](https://github.com/explosion/spaCy) kennenzulernen und an Beispielen zu verproben.

# Neue Features

## Projects

Eines der auffälligsten Features für mich sind ist ´spacy projects´. Das Feature verspricht eine deutlich bessere Verkettung verschiedener Aufgaben in einem NLP-Projekt (Daten laden, Pipelines trainieren, Export als Python-Package, etc.).

Das Beispielprojekt [tagger_parser_ud](https://spacy.io/usage/projects) zum Training eines POS-Taggers und Dependency Parsers ist ein guter Start in das Verstehen.

### Bestehende Projekte klonen

Die Installation eines Templates ist denkbar einfach. Mit dem Kommando `spacy project` erzeugt einen Unterordner für das Projekt mit allen benötigten Daten.

Der default geht bei `spacy project clone [OPTIONS] NAME [DEST]` auf das [Repository von explosion](https://github.com/explosion/projects). Eigene (auch private) Projekte können via `-r TEXT` bzw. `--repo TEXT` mit der jeweiligen Adresse ebenfalls geklont werden. `NAME` ist das Quellen-Repository, `DEST` kann angegeben werden um den Zielpfad anzupassen (default: Aktuelles Arbeitsverzeichnis).
Die Optionen für das Kommando sind `-b TEXT` bzw. `--branch TEXT` um einen bestimmten Branch zu klonen und `-S` bzw. `--sparse` um nur benötigte Dateien zu klonen. Mehr zu sparse checkouts kann man im [Blog-Eintrag](https://github.blog/2020-01-17-bring-your-monorepo-down-to-size-with-sparse-checkout/) auf github lesen.

Auf geht's:
```commandline
(venv) <work_path> spacy project clone pipelines/tagger_parser_ud
✔ Cloned 'pipelines/tagger_parser_ud' from explosion/projects
Path/To/Project/tagger_parser_ud
✔ Your project is now ready!
To fetch the assets, run:
python -m spacy project assets Path/To/Project/tagger_parser_ud
```
Wir erhalten einen Unterordner mit mehreren Dateien:
```tree
|-tagger_parser_ud
  |-configs
  |  |-default.cfg
  |-.gitignore
  |-project.yml
  |-README.md
  |-test_project_tagger_parser_ud.py
```
Was diese grundlegenden Dateien machen gucken wir uns jetzt an. Insbesondere das Herzstück des Projekts [project.yml](/tagger_parser_ud/project.yml). 

### Assets einsammeln

Was aktuell nämlich noch fehlt, sind Daten. Diese sind in den `assets` definiert. Jedes Asset besteht hat die Keys `dest` als relatives Zielverzeichnis, `url` oder `git` als Quellenangabe. Optional kann eine `checksum` angegeben werden. Als URL werden verschiedene Protokolle unterstützt, unter anderen HTTP, HTTPS, SSH, S3. Wenn ein Repository verwendet wird (wie im Beispiel unten), werden zusätzlich noch Keys für die Repository-Adresse `repo`, den `branch` und Relativpfad im Repository `path`.

```yaml
assets:
  - dest: "assets/${vars.treebank}"
    git:
      repo: "https://github.com/UniversalDependencies/${vars.treebank}"
      branch: "master"
      path: ""
```
Mit dem Kommando `spacy project assets` erhalten wir die genannten Assets:

```commandline
(venv) <work_path> spacy project assets Path/To/Project/tagger_parser_ud
ℹ Fetching 1 asset(s)
✔ Downloaded asset
Path/To/Project/tagger_parser_ud/assets/UD_English-EWT
```
Der Pfad ist optional und kann weggelassen werden, wenn man bereits mit `cd tagger_parser_ud` in den Projekt-Ordner `tagger_parser_ud` navigiert hat.

### Kommandos ausführen

Ebenfalls in der `project.yml` definiert sind die bei spacy, insbesondere für Nicht-Entwickler, beliebten Kommandos. Mit `spacy project run COMMAND` lassen sich die definierten Kommandos ausführen.

Spätestens jetzt müssen wir mit unserer Konsole ins Projekt. Von dort können wir eine der definierten Kommandos ausführen. Mit `spacy project run --help` erhalten wir eine Übersicht über verfügbare Kommandos und ihre Beschreibung. Zusätzlich werden auch Workflows gelistet, die ebenfalls per `run`ausgeführt werden können.

```commandline
(venv) <work_path> spacy project run --help

Part-of-speech Tagging & Dependency Parsing (Universal Dependencies)

Available commands in project.yml
Usage: python -m spacy project run [COMMAND]

convert    Convert the data to spaCy's format
train      Train UD_English-EWT
evaluate   Evaluate on the test data and save the metrics
package    Package the trained model so it can be installed
clean      Remove intermediate files

Available workflows in project.yml
Usage: python -m spacy project run [WORKFLOW]

all   convert -> train -> evaluate -> package
```
> ⚠ **ACHTUNG** ⚠ Das Beispielprojekt funktioniert in der von mir verwendeten Version nicht betriebssystemübergreifend (siehe [issue](https://github.com/explosion/spaCy/issues/6957)). Die Skripte sind mit UNIX-Kommandos gespickt, welche nicht ohne weiteres Zutun unter Windows laufen. Wer im Terminal die Kommandos nicht ausführen kann, kann auf [gitbash](https://gitforwindows.org/) zurückgreifen. 
> 
>Öffnet Git Bash im Projektordner und aktiviert die virtuelle Umgebung mit `source venv/Scripts/activate`.


Nun können wir die spacy Kommandos durchführen. Das ganze machen wir Schritt für Schritt wie im Workflow `all` dargestellt: `convert` -> `train` -> `evaluate` -> `package`. Für die 20 Epochen Training in `train` hat es bei mir sicherlich eine halbe Stunde gebraucht. Also haltet Kaffee oder etwas zu lesen bereit.

##### Convert
```commandline
(venv) spacy project run convert
[i] Grouping every 10 sentences into a document.
[+] Generated output file (1255 documents):
corpus\UD_English-EWT\en_ewt-ud-train.spacy
[i] Grouping every 10 sentences into a document.
[+] Generated output file (201 documents):
corpus\UD_English-EWT\en_ewt-ud-dev.spacy
[i] Grouping every 10 sentences into a document.
[+] Generated output file (208 documents):
corpus\UD_English-EWT\en_ewt-ud-test.spacy
```

Hiermit werden binäre `*.spacy`-Dateien erzeugt, die speicherperformant weiter verwendet werden können. Unter `corpus/UD_English-EWT` liegen nun drei Dateien für Training, Evaluation und (Dev?).

##### Train
```commandline
(venv) spacy project run train
Set up nlp object from config
Pipeline: ['tok2vec', 'tagger', 'morphologizer', 'parser']
Created vocabulary
Finished initializing nlp object
Initialized pipeline components: ['tok2vec', 'tagger', 'morphologizer', 'parser']

[+] Created output directory: training\UD_English-EWT
[i] Using CPU

=========================== Initializing pipeline ===========================
[+] Initialized pipeline

============================= Training pipeline =============================
[i] Pipeline: ['tok2vec', 'tagger', 'morphologizer', 'parser']
[i] Initial learn rate: 0.001
E    #       LOSS TOK2VEC  LOSS TAGGER  LOSS MORPH...  LOSS PARSER  TAG_ACC  POS_ACC  MORPH_ACC  DEP_UAS  DEP_LAS  SENTS_F  SCORE
---  ------  ------------  -----------  -------------  -----------  -------  -------  ---------  -------  -------  -------  ------
  0       0          0.00       148.17         149.29       297.09    21.23    23.72      26.47    20.43     5.51     0.95    0.20
  0     200       3365.40     16954.97       17988.39     29704.21    81.07    83.55      83.70    64.15    53.12    48.90    0.75
  0     400       4923.45      7136.26        8133.20     21205.90    86.60    87.77      88.83    67.24    58.31    41.43    0.80
  0     600       5484.03      5519.39        6554.79     19414.07    88.53    89.87      90.34    73.01    65.52    64.04    0.83
  0     800       5730.21      5121.71        6016.61     18029.11    89.19    90.23      91.42    72.64    65.68    67.80    0.84
  0    1000       5974.90      4570.50        5408.39     17552.41    89.98    90.90      92.37    73.30    66.65    59.97    0.85
  0    1200       6680.42      4646.75        5469.85     18227.79    90.35    91.27      92.38    75.56    69.10    67.63    0.86
  1    1400       6551.16      3690.92        4461.02     16726.41    91.01    91.92      93.12    76.36    70.44    69.14    0.86
  1    1600       8517.97      4255.43        5163.64     20094.18    90.94    91.98      93.02    77.24    71.65    68.46    0.87
  1    1800      10374.21      5310.68        6363.16     24040.57    91.72    92.41      93.42    77.56    72.02    69.21    0.87
  1    2000      13254.34      6786.62        8045.96     30416.89    91.73    92.59      93.66    78.54    73.30    69.27    0.88
  2    2200      15390.22      6284.45        7966.46     32994.87    91.77    92.63      93.69    79.78    74.77    71.47    0.88
  2    2400      20827.21      7886.70        9679.70     41333.49    92.02    92.67      93.65    80.34    75.46    71.62    0.89
  3    2600      25548.15      8224.77       10317.60     48349.38    92.12    92.86      94.05    80.73    76.03    71.54    0.89
  4    2800      30985.91      8975.79       11715.51     54102.55    92.30    93.10      93.94    81.06    76.46    72.45    0.89
  5    3000      33038.40      8131.55       10697.18     54965.00    92.39    93.14      94.14    81.40    76.78    72.47    0.89
  6    3200      33845.76      7625.88       10172.69     51875.30    92.44    93.18      94.02    81.46    76.97    71.60    0.89
  7    3400      33773.34      7160.73        9623.47     48529.81    92.34    93.14      94.09    81.58    77.14    73.83    0.89
  7    3600      34968.95      6580.32        8894.22     47716.01    92.45    93.18      94.04    81.60    77.31    73.47    0.89
  8    3800      34660.18      6016.21        8346.30     45074.63    92.47    93.27      94.18    81.83    77.27    72.65    0.89
  9    4000      35746.07      5715.42        7922.80     44567.01    92.57    93.19      94.17    81.98    77.63    73.08    0.90
 10    4200      35941.34      5351.76        7584.05     43017.20    92.62    93.36      94.14    81.89    77.36    72.54    0.90
 11    4400      36769.47      5107.81        7192.83     41997.42    92.65    93.31      94.26    81.89    77.34    73.75    0.90
 12    4600      37406.21      4783.85        6882.88     40418.95    92.52    93.35      94.19    82.05    77.71    73.97    0.90
 13    4800      39030.43      4769.49        6743.03     40506.64    92.71    93.36      94.15    82.36    77.96    73.41    0.90
 14    5000      38506.06      4421.23        6410.87     38293.99    92.60    93.32      94.14    82.41    77.96    74.21    0.90
 14    5200      39124.72      4338.20        6282.04     37759.94    92.62    93.34      94.25    81.79    77.37    72.14    0.90
 15    5400      40267.21      4018.49        5859.93     37076.61    92.60    93.29      94.13    82.51    77.89    74.93    0.90
 16    5600      39412.30      3897.49        5679.96     35791.14    92.66    93.34      94.19    82.30    77.68    74.21    0.90
 17    5800      40563.62      3799.47        5466.52     35171.70    92.52    93.19      94.19    81.81    77.41    73.67    0.89
 18    6000      41450.40      3847.51        5571.23     34792.24    92.43    93.23      94.11    82.11    77.74    75.21    0.90
 19    6200      39927.39      3494.81        5132.31     33481.89    92.65    93.25      94.09    82.16    77.72    73.74    0.90
 20    6400      41439.48      3472.84        5120.42     32726.51    92.67    93.44      94.20    82.28    77.86    74.09    0.90
[+] Saved pipeline to output directory
training\UD_English-EWT\model-last

=================================== train ===================================
Running command: 'Path\To\Project\venv\scripts\python.exe' -m spacy train configs/default.cfg --output training/UD_English-EWT --gpu-id -1 --paths.train corpus/UD_English-EWT/train.spacy --paths.dev corpus/UD_English-EWT/dev.spacy --nlp.lang=en
```

Im Ordner `training/UD_English-EWT` sind nach der Durchführung das beste und das zuletzt trainierte Modell abgespeichert.


##### Evaluate

```commandline
(venv) spacy project run evaluate
[i] Using CPU

================================== Results ==================================

TOK      99.84
TAG      93.09
POS      93.72
MORPH    94.71
UAS      81.49
LAS      77.41
SENT P   74.49
SENT R   66.78
SENT F   70.42
SPEED    16444


============================== MORPH (per feat) ==============================

                P        R       F
PronType    98.76    98.35   98.56
Number      96.25    95.04   95.64
Mood        94.21    92.84   93.52
Tense       94.56    95.44   95.00
VerbForm    92.54    92.65   92.59
Gender      99.74   100.00   99.87
Person      98.13    98.09   98.11
Poss        98.15    98.76   98.46
Definite    99.93    99.74   99.84
Degree      91.66    90.35   91.00
Case        96.20    96.20   96.20
NumType     93.28    90.91   92.08
Voice       58.97    88.46   70.77
Typo        28.57     4.35    7.55
Abbr        66.67    26.67   38.10
Foreign      0.00     0.00    0.00
Reflex     100.00    75.00   85.71


=============================== LAS (per type) ===============================

                   P       R       F
root           80.97   72.70   76.61
mark           88.78   90.75   89.76
nsubj          90.27   86.02   88.09
advcl          58.56   56.68   57.61
case           86.78   89.46   88.10
obl            70.26   67.93   69.08
nmod:poss      86.22   89.35   87.76
compound       60.71   65.45   62.99
cc             84.95   85.18   85.07
advmod         78.75   77.86   78.31
conj           65.06   64.08   64.57
det            93.42   93.17   93.30
amod           78.70   81.66   80.15
nmod           68.07   71.52   69.75
flat           59.91   50.60   54.86
cop            84.02   83.51   83.76
obl:npmod      40.54   31.25   35.29
acl:relcl      67.19   59.72   63.24
aux            94.35   94.35   94.35
ccomp          60.29   68.91   64.31
obj            83.43   87.53   85.43
nsubj:pass     79.09   86.14   82.46
fixed          76.36   67.74   71.79
parataxis      25.77   24.39   25.06
compound:prt   75.00   77.53   76.24
aux:pass       82.44   95.58   88.52
nummod         64.56   65.95   65.25
xcomp          74.59   78.41   76.45
acl            56.19   62.29   59.08
dep             0.00    0.00    0.00
appos          18.24   29.03   22.41
expl           75.44   70.49   72.88
nmod:tmod      72.73   64.86   68.57
iobj           74.42   78.05   76.19
cc:preconj     50.00   33.33   40.00
obl:tmod       52.05   57.58   54.68
csubj          53.85   30.43   38.89
discourse      72.04   54.92   62.33
nmod:npmod     40.00   15.38   22.22
csubj:pass      0.00    0.00    0.00
list           19.72    5.58    8.70
vocative       29.63   38.10   33.33
reparandum      0.00    0.00    0.00
det:predet     90.00   81.82   85.71
goeswith        0.00    0.00    0.00
orphan          0.00    0.00    0.00
flat:foreign    0.00    0.00    0.00
dislocated      0.00    0.00    0.00

[+] Saved results to metrics\UD_English-EWT.json

================================== evaluate ==================================
Running command: 'Path\To\Project\venv\scripts\python.exe' -m spacy evaluate ./training/UD_English-EWT/model-best ./corpus/UD_English-EWT/test.spacy --output ./metrics/UD_English-EWT.json --gpu-id -1
```

Zusätzlich zur Ausgabe in der Konsole sind im Ordner `metrics` die Ergebnisse der Evaluation gespeichert.

##### Package

```commandline
(venv) spacy project run package
running sdist
running egg_info
creating en_ud_en_ewt.egg-info
writing en_ud_en_ewt.egg-info\PKG-INFO
writing dependency_links to en_ud_en_ewt.egg-info\dependency_links.txt
writing entry points to en_ud_en_ewt.egg-info\entry_points.txt
writing requirements to en_ud_en_ewt.egg-info\requires.txt
writing top-level names to en_ud_en_ewt.egg-info\top_level.txt
writing manifest file 'en_ud_en_ewt.egg-info\SOURCES.txt'
reading manifest file 'en_ud_en_ewt.egg-info\SOURCES.txt'
reading manifest template 'MANIFEST.in'
warning: no files found matching 'LICENSE'
writing manifest file 'en_ud_en_ewt.egg-info\SOURCES.txt'
warning: sdist: standard file not found: should have one of README, README.rst, README.txt, README.md

running check
warning: check: missing required meta-data: url

warning: check: missing meta-data: either (author and author_email) or (maintainer and maintainer_email) should be supplied

creating en_ud_en_ewt-0.0.0
creating en_ud_en_ewt-0.0.0\en_ud_en_ewt
creating en_ud_en_ewt-0.0.0\en_ud_en_ewt.egg-info
creating en_ud_en_ewt-0.0.0\en_ud_en_ewt\en_ud_en_ewt-0.0.0
creating en_ud_en_ewt-0.0.0\en_ud_en_ewt\en_ud_en_ewt-0.0.0\morphologizer
creating en_ud_en_ewt-0.0.0\en_ud_en_ewt\en_ud_en_ewt-0.0.0\parser
creating en_ud_en_ewt-0.0.0\en_ud_en_ewt\en_ud_en_ewt-0.0.0\tagger
creating en_ud_en_ewt-0.0.0\en_ud_en_ewt\en_ud_en_ewt-0.0.0\tok2vec
creating en_ud_en_ewt-0.0.0\en_ud_en_ewt\en_ud_en_ewt-0.0.0\vocab
copying files to en_ud_en_ewt-0.0.0...
copying MANIFEST.in -> en_ud_en_ewt-0.0.0
copying meta.json -> en_ud_en_ewt-0.0.0
copying setup.py -> en_ud_en_ewt-0.0.0
copying en_ud_en_ewt\__init__.py -> en_ud_en_ewt-0.0.0\en_ud_en_ewt
copying en_ud_en_ewt\meta.json -> en_ud_en_ewt-0.0.0\en_ud_en_ewt
copying en_ud_en_ewt.egg-info\PKG-INFO -> en_ud_en_ewt-0.0.0\en_ud_en_ewt.egg-info
copying en_ud_en_ewt.egg-info\SOURCES.txt -> en_ud_en_ewt-0.0.0\en_ud_en_ewt.egg-info
copying en_ud_en_ewt.egg-info\dependency_links.txt -> en_ud_en_ewt-0.0.0\en_ud_en_ewt.egg-info
copying en_ud_en_ewt.egg-info\entry_points.txt -> en_ud_en_ewt-0.0.0\en_ud_en_ewt.egg-info
copying en_ud_en_ewt.egg-info\not-zip-safe -> en_ud_en_ewt-0.0.0\en_ud_en_ewt.egg-info
copying en_ud_en_ewt.egg-info\requires.txt -> en_ud_en_ewt-0.0.0\en_ud_en_ewt.egg-info
copying en_ud_en_ewt.egg-info\top_level.txt -> en_ud_en_ewt-0.0.0\en_ud_en_ewt.egg-info
copying en_ud_en_ewt\en_ud_en_ewt-0.0.0\config.cfg -> en_ud_en_ewt-0.0.0\en_ud_en_ewt\en_ud_en_ewt-0.0.0
copying en_ud_en_ewt\en_ud_en_ewt-0.0.0\meta.json -> en_ud_en_ewt-0.0.0\en_ud_en_ewt\en_ud_en_ewt-0.0.0
copying en_ud_en_ewt\en_ud_en_ewt-0.0.0\tokenizer -> en_ud_en_ewt-0.0.0\en_ud_en_ewt\en_ud_en_ewt-0.0.0
copying en_ud_en_ewt\en_ud_en_ewt-0.0.0\morphologizer\cfg -> en_ud_en_ewt-0.0.0\en_ud_en_ewt\en_ud_en_ewt-0.0.0\morphologizer
copying en_ud_en_ewt\en_ud_en_ewt-0.0.0\morphologizer\model -> en_ud_en_ewt-0.0.0\en_ud_en_ewt\en_ud_en_ewt-0.0.0\morphologizer
copying en_ud_en_ewt\en_ud_en_ewt-0.0.0\parser\cfg -> en_ud_en_ewt-0.0.0\en_ud_en_ewt\en_ud_en_ewt-0.0.0\parser
copying en_ud_en_ewt\en_ud_en_ewt-0.0.0\parser\model -> en_ud_en_ewt-0.0.0\en_ud_en_ewt\en_ud_en_ewt-0.0.0\parser
copying en_ud_en_ewt\en_ud_en_ewt-0.0.0\parser\moves -> en_ud_en_ewt-0.0.0\en_ud_en_ewt\en_ud_en_ewt-0.0.0\parser
copying en_ud_en_ewt\en_ud_en_ewt-0.0.0\tagger\cfg -> en_ud_en_ewt-0.0.0\en_ud_en_ewt\en_ud_en_ewt-0.0.0\tagger
copying en_ud_en_ewt\en_ud_en_ewt-0.0.0\tagger\model -> en_ud_en_ewt-0.0.0\en_ud_en_ewt\en_ud_en_ewt-0.0.0\tagger
copying en_ud_en_ewt\en_ud_en_ewt-0.0.0\tok2vec\cfg -> en_ud_en_ewt-0.0.0\en_ud_en_ewt\en_ud_en_ewt-0.0.0\tok2vec
copying en_ud_en_ewt\en_ud_en_ewt-0.0.0\tok2vec\model -> en_ud_en_ewt-0.0.0\en_ud_en_ewt\en_ud_en_ewt-0.0.0\tok2vec
copying en_ud_en_ewt\en_ud_en_ewt-0.0.0\vocab\key2row -> en_ud_en_ewt-0.0.0\en_ud_en_ewt\en_ud_en_ewt-0.0.0\vocab
copying en_ud_en_ewt\en_ud_en_ewt-0.0.0\vocab\lookups.bin -> en_ud_en_ewt-0.0.0\en_ud_en_ewt\en_ud_en_ewt-0.0.0\vocab
copying en_ud_en_ewt\en_ud_en_ewt-0.0.0\vocab\strings.json -> en_ud_en_ewt-0.0.0\en_ud_en_ewt\en_ud_en_ewt-0.0.0\vocab
copying en_ud_en_ewt\en_ud_en_ewt-0.0.0\vocab\vectors -> en_ud_en_ewt-0.0.0\en_ud_en_ewt\en_ud_en_ewt-0.0.0\vocab
Writing en_ud_en_ewt-0.0.0\setup.cfg
creating dist
Creating tar archive
removing 'en_ud_en_ewt-0.0.0' (and everything under it)
[i] Building package artifacts: sdist
[+] Loaded meta.json from file
training\UD_English-EWT\model-best\meta.json
[+] Successfully created package 'en_ud_en_ewt-0.0.0'
packages\en_ud_en_ewt-0.0.0
[+] Successfully created zipped Python package
packages\en_ud_en_ewt-0.0.0\dist\en_ud_en_ewt-0.0.0.tar.gz

================================== package ==================================
Running command: 'Path\To\Project\venv\scripts\python.exe' -m spacy package training/UD_English-EWT/model-best packages --name ud_en_ewt --version 0.0.0 --force
```

Großartig! Mit `package` haben wir das letzte Kommando im Workflow ausgeführt und haben nun ein vollständiges Projekt zum Deployment oder sonstigen Verwendung im Ordner `packages` abgelegt. In der Konsolenausgabe sind ein paar Warnungen aufgeführt, die wir bei tatsächlicher Weiterverwendung sicherlich noch einmal genauer angucken und lösen sollten. Für's erste haben wir aber hier eine gute Übersicht über die neuen Möglichkeiten von spacy projects erhalten.

Wer direkt aufräumen möchte und bis auf das Package nichts mehr sichten möchte, kann mit `spacy project clean` die Intermediat-Ordner `training/*`, `metrics/*` und `corpus/*` wieder löschen.

### Weitere Features von Projects

Wer weitere Details zu spacy projects nachlesen möchte, findet viele weitere Informationen auf [spacy.io/usage/projects](https://spacy.io/usage/projects). Eine Übersicht über mögliche Variablen in der `project.yml` gibt es [hier](https://spacy.io/usage/projects#project-yml).

# Aufgaben
- [x] Readme aufsetzen
- [x] Ignore-File generieren
- [x] requirements.txt aufsetzen
- [ ] Neue Features von spacy 3.0 sichten und kurz zusammenfassen
  - [ ] `spacy project` zum Verwalten und Teilen von Ende-zu-Ende Workflows
    - [x] Skript-Problem umgehen oder lösen, siehe [issue](https://github.com/explosion/spaCy/issues/6957) -> Gelöst durch ersetzen von cmd.exe mit gitbash
    - [x] Beispiel durchexerzieren
  - [ ] Weitere Aufgaben aus Neuerungen ableiten



# Leitfaden zur Reproduzierung

### Installation
Ich arbeite in Windows gerne mit PyCharm als IDE. Wir wählen eine geeignete Python-Distribution für unsere virtuelle Umgebung. Alle benötigten Packages sind in den [requirements](requirements.txt) notiert.
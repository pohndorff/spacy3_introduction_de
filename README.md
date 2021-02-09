# Beschreibung

Dies ist ein Projekt mit dem Ziel neue Features in [spacy v3.0](https://github.com/explosion/spaCy) kennenzulernen und an Beispielen zu verproben.

# Neue Features

## Projects

Eines der auffälligsten Features für mich sind ist ´spacy projects´. Das Feature verspricht eine deutlich bessere Verkettung verschiedener Aufgaben in einem NLP-Projekt (Daten laden, Pipelines trainieren, Export als Python-Package, etc.).

Das Beispielprojekt [tagger_parser_ud](https://spacy.io/usage/projects) zum Training eines POS-Taggers und Dependency Parsers ist ein guter Start in das Verstehen.

Die Installation ist denkbar einfach

# Aufgaben
- [x] Readme aufsetzen
- [x] Ignore-File generieren
- [x] requirements.txt aufsetzen
- [ ] Neue Features von spacy 3.0 sichten und kurz zusammenfassen
  - [ ] `spacy project` zum Verwalten und Teilen von Ende-zu-Ende Workflows
  - [ ] Weitere Aufgaben aus Neuerungen ableiten


# Leitfaden zur Reproduzierung

### Installation
Ich arbeite gerne mit PyCharm als IDE. Wählt eine geeignete Python-Distribution für eure virtuelle Umgebung. Ich verwende in diesem Projekt Python 3.9. Spacy 3.0 benötigt mindestens Version 3.6. 
```terminal
pip install spacy
```
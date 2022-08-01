# Architektur Kata `CheckYourWork`

Mein Versuch einer Lösung für das Architektur Kata `CheckYourWork`

Aufgabe/kata: siehe kata.md

Lösung/Architektur: siehe architecture.adoc

Die Lösung ist in AsciiDoc (https://asciidoc.org/). Ich habe versucht mich am arc42 (https://arc42.org) zu orientieren.
Diagramme sind mit PlantUML gemacht (https://plantuml.com/)

Dateien/Verzeichnisse|Inhalt
---|---
architecture.adoc|Lösung/Architektur
architecture.pdf|Lösung/Architektur in PDF-Format zum leichteren Lesen
kata.md|Kata/Aufgabenstellung
README.md|die Datei hier
chapter|die Unterkapitel, die in `architecture.adoc` referenziert werden
experimente|Ablage für Experimente, drafts, etc.
images|hier werden Bilder generiert beim Erzeugen des PDF

Wie kann man das anschauen? Im Github sieht man keine PlantUML-Diagramme?

Meine Empfehlung: Auschecken und im IntelliJ anschauen (mit Plugins: https://plugins.jetbrains.com/plugin/7391-asciidoc und
https://plugins.jetbrains.com/plugin/7017-plantuml-integration). Zusätzlich braucht man noch GraphViz lokal installiert: https://plantuml.com/en/starting

PDF:
Das PDF habe ich zum leichteren Lesen eingebunden. Es wird erzeugt, indem man im IntelliJ das 'architecture.adoc' öffnet und  die Option "create PDF from current file" ausführt. 
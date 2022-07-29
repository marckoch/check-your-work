
# Architektur

## Abläufe / UseCases

Spezifizierung der o.g. Requirements

### Student löst Aufgabe

Annahmen: folgende 2 Punkte erfolgen nicht in dieser Anwendung `CheckYourwork`:

- Student schaut Aufgabe im `LMS` an
- Student schreibt Code zur Aufgabe (auf seinem privaten Rechner in der IDE seiner Wahl)

Im `CheckYourWork` nun:

- Student meldet sich an (Abgleich/Delegierung gegen `LMS`)
- Student lädt Code in textueller Form in Anwendung `CheckYourWork` hoch, dabei gibt er an für welche Aufgabe diese Lösung gilt
- Ablehnung falls Abgabedatum schon verstrichen (wird ebenfalls auditierbar festgehalten)
- Code wird auf Plagiat geprüft (gegen andere Lösungen und gegen Fremdsystem `TurnItIn`) (Annahme: das geschieht bevor Benotung, Plagiatsversuch wird ebenfalls auditierbar gespeichert)
- Abbruch bei Plagiatsverdacht
- Code wird in passender Sandbox ausgeführt
- Code wird bewertet
- Ausführung und Ergebnis werden auditierbar gespeichert
- das Ergebnis wird dem Studenten (nicht editierbar) angezeigt: Bewertung, Metriken, Testergebnisse
- Student kann weitere Versuche hochladen (selber Ablauf wie vorher)
- Am Ende sind alle n Versuche des Studenten zu dieser Aufgabe auditierbar festgehalten
- der Student kann einmal hochgeladenen Code nicht mehr ändern/löschen

Annahme: Die Prüfung des Code geht schnell (<60 sek) und der Student bekommt synchron Feedback. So kann er zeitnah Nacharbeiten/Verbesserungen vornehmen und einen erneuten Versuch hochladen

### Professor legt Kriterien fest

Annahmen:

- Aufgaben sind im `LMS` hinterlegt
- Zuordnung Studenten zu Kurs sind im `LMS` hinterlegt
- Zugangsdaten des Professors sind im `LMS` hinterlegt

Ablauf (im neuen System `CheckYourWork`):

- Professor meldet sich an (Abgleich gegen `LMS`)
- Professor legt fest:
  - welche Aufgabe aus dem `LMS`
  - für welchen Kurs (und damit indirekt für welche Studenten)
  - bis wann (Abgabedatum/zeit)
  - mit welchen Abnahmekriterien (Metriken / Tests)
  
  gelöst werden soll

Im Grunde genommen ein Basic CRUD, d.h. Prof kann Daten auch nochmal ändern, etc.

### Verwandte Webseiten

Ähnliche Webseiten gibt es im Internet zugänglich, z.B. Codewars, Hackerrank oder JetBrainsAcademy. Typische Antwortzeiten für einen Roundtrip (Code hochladen/abschicken, Ausführen, Sichten des Ergebnisses) sind im Bereich 30-60 Sekunden.

### Codeausführung

- hochgeladener Code wird auf einer seperaten Infrastruktur ausgeführt (es darf kein vom Studenten hochgeladener Code direkt auf dem Produktivsystem ausgeführt werden), d.h. die Ausführung erfolgt in eigener Sandbox, Container, etc. evtl "dynamisches Provisioning" ala AWS o.ä., diese Sandbox hat keinerlei Verbindung zur Datenbank mit den gespeicherten Ausführungsergebnissen
- diese seperate Infrastruktur sollte gewissen Elastizitäts-Ansprüchen genügen, so dass ein Student sicher sein kann, dass seine Codeausführung binnen 10-20 Sekunden beginnt
- Annahme: es gibt verschiedene Programmiersprachen im `LMS` zu denen Aufgaben gelöst werden müssen. Für jede Programmiersprache muss ein eigenes Sandbox-Template (Basis Container-Image o.ä.) definiert werden

Ablauf:

- Sandbox/Container wird von `CheckYourWork` provisioniert/initialisiert
- hochgeladener Code wird darin compiliert
- hochgeladener Code wird darin ausgeführt
- Tests werden ausgeführt
- Metriken werden angewandt
- `CheckYourWork` liest Ergebnisse aus Container aus
- Beenden der Sandbox

Anschliessend (nicht mehr in der Sandbox):

- Plagiatsprüfung gegen andere Codesnippets anderer Studenten
- Plagiatsprüfung gegen `TurnItIn`
- Bewertung der Arbeit
- Speichern aller Daten (Code, Bewertung, Metriken) in auditierbarer Form in DB

### Plagiatsprüfung

- komplett unklar:
- Es soll gegen andere Lösungen anderer Studenten verglichen werden. Gegen welche Studenten? Nur die des gleichen Kurses? Oder auch gegen die Studenten des Vorjahres, die diesen Kurs besucht haben? Oder gegen den kompletten Lösungspool einer Aufgabe aller Studenten aller Jahre, die jemals diese eine Aufgabe gelöst haben? Je grösser man diesen Pool macht, desto höher die Wahrscheinlichkeit ähnliche Lösungen zu finden, die dann als vermeintliches Plagiat markiert werden.
- Wie soll aber ein Plagiat erkannt werden? Je kleiner die Aufgabe ("simple programming assignments") desto ähnlicher sind alle Lösungen. "Schreibe eine Funktion, die prüft ob eine Zahl eine Primzahl ist" wird bei 300 Studenten eines Jahres sicherlich sehr oft sehr ähnlich gelöst werden. Geht man weitere Jahre zurück steigt die Wahrscheinlichkeit weiter, eine ähnliche Lösung zu finden.
- evtl optional Zukauflösungen evaluieren (das widerspricht aber etwas dem schmalen IT Budget)

--> Risiko: Plagiatsprüfung noch sehr unklar/offen, weitere Klärung nötig

Vorschlag einer denkbaren/kostengünstigen Minimallösung:

- Bei Hochladen wird gegen alle in diesem Jahr zu dieser Aufgabe hochgeladenen Lösungen verglichen. Bei exaktem Match wird die Lösung mit Warning versehen. Hier muss eine manuelle Prüfung/Sichtung durch den Professor erfolgen.

### Auditierbarkeit

- folgende Daten werden bei jedem Lauf eines Codes festgehalten:
  - Zuordnung zum Student (über dessen Id)
  - Zuordnung zur Aufgabe im LMS (über dessen Id, damit klar ist welche Aufgabe dieser Code löst)
  - hochgeladener Code (in textueller Form, kein Compilat o.ä.)
  - Hochladeversuche nach abgelaufener Deadline
  - Ausgaben während der Ausführung (StdOut, StdErr, etc.)
  - Testergebnisse
  - Metriken (Kriterien samt Ergebnis)
  - Plagiatsprüfergebnisse
  - Bewertungen (Benotung)

- diese Daten werden persistent festgehalten in einer regelmässig gesicherten Datenbank
- diese Daten werden beim Speichern aufgelöst und dupliziert gespeichert, d.h. Referenzen werden aufgelöst, etc. so ist sichergestellt, dass nachträgliche Änderungen an z.B. Metriken sich nicht in die persistierten Auditdaten durchschlagen. (Snapshot), d.h. es wird NICHT gespeichert: "Metrik mit Metrik-ID 132 ist verletzt, daher nicht bestanden". Hier ist unklar wie Metrik mit ID 132 zum Zeitpunkt der Codeausführung aussah. Diese Referenz wird aufgelöst und es wird gespeichert: "Metrik mit Metrik-ID 132 (lines of code darf 40 nicht übersteigen) ist verletzt, daher nicht bestanden". So ist auditierbar festgehalten, das die Metrik zum Zeitpunkt der Prüfung loc < 40 beinhaltete, obwohl sie ggf. heute auf loc < 100 geändert wurde.
- Lesenden Zugriff (nur SELECT / READ) auf diesen Daten erhält nur ein entsprechend priviligierter Personenkreis (keine Studenten, aber Professoren und staatliche Behörden schon)
- Schreibenden Zugriff (nur INSERT / CREATE) auf diese Datenbank erhält nur ein technischer User, der ausschliesslich vom `CheckYourWork` verwendet wird

--> vom CRUD auf diese Daten ist das C nur der Anwendung `CheckYourWork` erlaubt und das R nur den Professoren und staatlichen Behörden, U und D sind verboten

TODO: Tabelle

Annahme: Hierfür wird ein einfaches Reporting erstellt
offen: in welchem Format wollen die Auditoren diese Daten haben? (Word? CSV? EXCEL?)
offen: wie lange sollen Daten aufgehoben werden? (für Architektur eher irrelevant, lässt sich durch Speicherplatz in DB lösen)
offen: technische Details: Wie soll Code in DB gespeichert werden? als BLOB? Text? etc.

### Mengengerüste

- 300 User pro Jahr scheint nicht viel, keine globale / überregionale Lösung nötig
- es ist von "simple programming assignments" die Rede, d.h vermutlich <1000 lines of code
- Annahmen:
  - 300 Studenten pro Jahr
  - Jeder Student besucht pro Vorlesungstag 2 Kurse
  - Jeder Kurs stellt pro Vorlesungstag eine Aufgabe (oder anders: Jeder Student erhält in Summe pro Woche 10 Aufgaben)

  --> 300 x 2 x 1 = 600 Codesnippets plus Ausführungen/Plagiatsprüfungen pro Vorlesungstag

- Annahme:
  - Während der Vorlesungszeit (werktags 8-18 Uhr) wenig Aktivität (da sind die Studenten ja in der Vorlesung)
  - Erhöhte Aktivität in der Vorlesungszeit abends, nachts und am Wochenende (dann wenn die Aufgaben gelöst werden)
  - Während der Ferien wenig Aktivität

--> 600 Uploads pro Vorlesungstag --> verteilt auf 10 Stunden vorlesungsfreie Zeit am Tag macht das ca. 60 Uploads pro Stunde, d.h. ca 1 pro Minute

- Vermutung: Vor Deadlines erhöhtes Aufkommen von Anfragen, d.h. Elastizität nicht unwichtig

### Kosten

- IT Budget ist sehr begrenzt, dadurch scheiden kostenintensive Lösungen aus (Oracle, etc.)
- es wird vermehrt auf kostengünstige, Opensource-Alternativen gesetzt (z.B. PostgreSQL statt Oracle)

### Authentisierung / Autorisierung

Annahme: Die Benutzerdaten der Studenten und Professoren sind im `LMS` hinterlegt und können über dessen Integration abgegriffen werden. Die Studenten und Professoren brauchen im `CheckYourWork` kein neues Benutzerkonto anlegen. Die Anmeldung am CheckYourWork delegiert an das `LMS`.

## Schnittstellen

### LMS

- unklar warum hier Integration nötig
- Annahme: `LMS` enthält Studentendaten, Zugangsdaten der Studenten, Zugangsdaten der Professoren, Kursdaten, Zuordnung von Studenten zu Kursen und Aufgaben sowie Lehrmaterialien
- Annahme: Die zu lösenden Aufgaben sind im `LMS` hinterlegt. Es muss eine Verknüpfung erfolgen zwischen Aufgabe im `LMS` und hochgeladenem Code, damit klar ist zu welcher Aufgabe der hochgeladene Code gehört. Diese Zuordnung erfolgt im `CheckYourWork` beim Hochladen des Codes, indem mit dem Schlüssel der gelösten Aufgabe verknüpft wird.
- `LMS` ist eine Mainframe-basierte Lösung, an dem nicht viel geändert werden kann. Daraus folgende Annahme: Änderungen erfolgen dann hauptsächlich auf Seite des `CheckYourWork` (Conformer)

### TurnItIn

- web-based Schnittstelle zur Plagiatserkennung
- Art der Schnittstelle unklar ("web-based"? REST? SOAP? XML?)
- Annahme: synchrone Webschnittstelle
- es wird das Codesnippet ins `TurnItIn` hochgeladen
- das Ergebnis der Plagiatsprüfung liegt unmittelbar vor

OFFEN:

- was wenn Schnittstelle zu `TurnItIn` nicht verfügbar ist? was wenn dadurch Deadline verstreicht? (diese Fälle zumindest auditerbar festhalten, damit evtl Deadline in Ausnahmefällen verlängert wird? o.ä.)

### Sandbox Hoster

wo laufen die Sandboxen? wer pflegt die? wer legt wo fest welche Aufgabe welche Sandbox braucht?

### Risiken

1. Ausfall von `TurnItIn`
2. Plagiatsprüfung unklar
3. 

---
title: "Skalierbare Systeme - Technische Entscheidungsfindung"
topic: "scalable_systems_1_3"
author: "Silas Schnurr"
theme: "metropolis"
fonttheme: "structurebold"
fontsize: 12pt
urlcolor: BrickRed
linkcolor: BrickRed
aspectratio: 169
lang: de-DE
numbersections: true
plantuml-format: svg
toc: true
section-titles: true
...

# Einstieg

## Heute

- Rückblick und offene Themen aus Vorlesung 1 und 2
- Inhalt
- **Aufgabe 2:** noch einmal ein System entwerfen, mit dem Werkzeugkasten von heute

## Lernziele

- Architekturentscheidungen von Implementierungsentscheidungen unterscheiden
- Trade-offs zwischen Qualitätsattributen erkennen und explizit machen
- Stakeholder und Kontextfaktoren identifizieren und in die Entscheidung einbeziehen
- Eine Entscheidung strukturiert herleiten, dokumentieren und kommunizieren
- Eine KI-Unterstützung so einsetzen, dass die Entscheidung bei euch bleibt

# Aufgabe 1

## Aufgabe 1: Anmeldesystem für Lehrveranstaltungen

- Unter www.dhbw-anmeldesystem-lehrveranstaltung.de können sich Studierende für Lehrveranstaltungen anmelden
- Wird für alle neun DHBW-Standorte verwendet, jeder mit eigenem Prüfungsamt und eigenem Stichtag
- Pro Standort rund 6.000 Studierende, die Anmeldung öffnet zu einem festen Zeitpunkt
- In den ersten fünf Minuten melden sich rund 80 % an
- Plätze sind begrenzt, eine Überbuchung ist nicht zulässig; danach kommt eine Bestätigung per E-Mail
- Ein Standort sieht nur seine eigenen Kurse und Studierenden; zwischen den Stichtagen läuft das System fast ohne Last

## Aufgabe 1: Ziel

1. Skizze der Komponenten und des Datenflusses
2. Fünf Entscheidungen, die ihr getroffen habt, **mit Begründung**
3. Was habt ihr bewusst **nicht** gemacht?
4. Welche Annahmen habt ihr getroffen, weil die Aufgabe etwas nicht sagt?

- Nutzung von KI erlaubt
- Zeit: 20 Minuten
- Danach Besprechung / Präsentation

\rightarrow{} So eine Aufgabe könnte im Anwendungsteil der Klausur sein

## Hinweis: Arbeiten mit LLMs

- Ein LLM stellt **nicht alle nötigen Rückfragen**: Was ihr nicht definiert, kann einfach angenommen werden
- Diese Annahmen sind teilweise unsichtbar und können genau die Architekturentscheidungen
- Nützliche Prompts: Kontext und Randbedingungen nennen, Optionen **und** Gegenargumente verlangen, nach Annahmen fragen
- Merkt euch beim Arbeiten: Was habt **ihr** entschieden, was hat das Werkzeug entschieden?

## Aufgabe 1: Besprechung

- Welche Entscheidungen wurden explizit getroffen und diskutiert, welche sind einfach passiert?
- Welche Begründungen tragen, welche sind "best practice"?
- Wo wurde gerechnet, wo geraten?
- Die Rechnung: 6.000 × 80 % in 300 Sekunden sind 16 Anmeldungen pro Sekunde; die Spitze machen die Requests, nicht die Anmeldungen
- Ein System oder neun? Wer hat das entschieden, und mit welcher Begründung?
- Was hat niemand entschieden, obwohl es eine Entscheidung war?

## Aufgabe 1: eine mögliche Lösung

```{.plantuml height=85%}
@startuml
left to right direction
skinparam defaultFontSize 15
skinparam ranksep 25
skinparam nodesep 15
actor "Studierende" as S
[CDN] as CDN
rectangle "Anmeldedienst, zustandslos,\nn Instanzen hinter Load Balancer" {
  [API] as API
}
database "Cache\nKursliste,\nRestplätze" as C
database "PostgreSQL\nPlätze per Constraint,\nStandort als Tenant-ID" as DB
[Outbox-Poller] as P
cloud "Mail-Dienst" as M
S --> CDN : Seite, Skripte
S --> API : anmelden
API --> C : lesen
API --> DB : eine Transaktion
DB -right-> P : liest
P -right-> M : at-least-once
@enduml
```

## Aufgabe 1: die Entscheidungen dahinter

- Lesen skalieren, Schreiben einfach halten: 16 Anmeldungen pro Sekunde brauchen keine Queue, die Seitenaufrufe brauchen CDN und Cache
- Ein System für neun Standorte: Pool-Modell, Standort als Tenant-ID in jeder Tabelle, erzwungen im Tenant Context
- Keine Überbuchung: Constraint in der Datenbank, keine Replizierung der Datenbank
- Bewusst nicht: Silo je Standort, Microservices, NoSQL, Queue vor der Vergabe
- Annahmen: Die Stichtage verteilen sich über Wochen, die Mail darf eine Minute später kommen

# Architektur heißt entscheiden

## Was ist eine Architekturentscheidung?

1. Wir nutzen PostgreSQL statt MongoDB
2. Wir formatieren Code mit Prettier
3. Die Bestätigungsmail wird asynchron verschickt
4. Wir benennen Tabellen im Singular
5. Authentifizierung läuft über einen externen Identity Provider
6. Wir verwenden React statt Vue

## Was ist eine Architekturentscheidung?

- Entscheidungen, die schwer rückgängig zu machen sind
- Sie betreffen Struktur, Qualitätsattribute, Schnittstellen und Technologie mit Lock-in
- Beispiele: Datenbank, Kommunikationsstil, Modulschnitt, Cloud-Anbieter, Programmiersprache
- Gegenbeispiele: Bibliothek für Datumsformatierung, Namenskonventionen
- Die nicht-funktionalen Anforderungen sind der Grund, warum eine Entscheidung architektonisch ist

\rightarrow{} Prüffrage: Was kostet es in einem Jahr, das zurückzudrehen?

## Everything is a trade-off

- Erstes Gesetz der Softwarearchitektur: Alles ist ein Trade-off
- Zweites Gesetz: Das Warum ist wichtiger als das Wie

- "Best Practice" ohne Kontext ist eine Meinung
- Die Aufgabe: Optionen sichtbar machen und bewusst wählen
- Wer keinen Nachteil seiner Lösung nennen kann, hat die Alternativen nicht verstanden

## Reversibilität

- Irreversible Entscheidungen verdienen Zeit, reversible verdienen Tempo
- Last Responsible Moment: so spät wie möglich entscheiden, aber nicht später
- Reversibilität herstellen: Abstraktionen, Standards, Daten exportierbar halten
- Typischer Fehler: alle Entscheidungen gleich schwer nehmen und dadurch gar nicht entscheiden

## Reversibilität und Tragweite

|                      | kleine Tragweite                                          | große Tragweite                                                                             |
| -------------------- | --------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| **leicht umkehrbar** | einfach machen: Bibliothek hinter Interface, Feature-Flag | schnell entscheiden, Rückweg sichern: Managed Service mit Datenexport                       |
| **schwer umkehrbar** | kurz prüfen, dann festlegen: Namensschema der API         | hier gehört die Zeit hin: Datenbank mit 5 TB Bestand, API mit Fremdnutzern, Mandantenmodell |

\rightarrow{} Die Zeit für eine Entscheidung richtet sich nach dem Quadranten, nicht nach der Lautstärke

## Kosten einer Entscheidung

- Direkte Kosten: Zeit, Geld, Lizenzen
- Indirekte Kosten: Komplexität, Lernkurve, Betrieb, Lock-in
- Opportunitätskosten: Was bauen wir dafür nicht?
- Kosten des Nicht-Entscheidens: Drift, implizite Entscheidungen durch den Code
- Jede Entscheidung erzeugt Folgeentscheidungen für andere

- Beispiel Kafka: Lizenz 0 €, aber zwei Wochen Lernkurve, ein Betriebsthema mehr, und nachts muss jemand ran

## Architecture Advice Process (Softwarearchitektur Dezentral leben)

- Wer eine Entscheidung treffen will, darf sie treffen
- Pflicht vorher: alle Betroffenen und erfahrene Personen zum Thema **anhören**
- Der Rat ist verbindlich einzuholen, aber nicht verbindlich zu befolgen
- Die Entscheidung und der eingeholte Rat werden dokumentiert (ADR)

\rightarrow{} schnelle Entscheidungen ohne zentralen Flaschenhals, trotzdem nachvollziehbar

## Entscheiden in der Gruppe

- Konsens: alle sind dafür \rightarrow{} teuer, langsam, oft der kleinste gemeinsame Nenner
- Konsent: niemand hat einen schwerwiegenden Einwand \rightarrow{} meist der bessere Modus
- Disagree and commit: die Entscheidung tragen, auch wenn man anderer Meinung war
- Rollen klären, bevor diskutiert wird: Wer entscheidet, wer berät, wer ist betroffen?

- Antipattern: Design by Committee: jeder bekommt sein Feature in die Architektur, das Ergebnis trägt niemand

## Kognitive Verzerrungen

- **Sunk Cost**: "Wir haben schon so viel investiert" ist kein Argument für die Zukunft
- **Anchoring**: Die erste genannte Option bestimmt den Rahmen der Diskussion
- **Authority Bias**: die Meinung der höchstbezahlten Person im Raum
- **Bandwagon**: Netflix macht das auch (...nur lösen die damit andere Probleme)
- **Confirmation Bias**: wir suchen Belege für die Lösung, die wir schon wollten

- Gegenmaßnahme: Optionen parallel bewerten, Gegenargumente einfordern, Annahmen aufschreiben
- Rückblick zu Aufgabe 1: Welche davon habt ihr in eurer Gruppe erlebt?

# Qualitätsattribute und Trade-offs

## Von der vagen Anforderung zum prüfbaren Kriterium

"Das System muss performant sein" ist keine Anforderung, sondern ein Wunsch

- Die vier Fragen: Wann? Unter welchen Bedingungen? Was genau passiert? Woran messen wir es?
- Was kein Maß hat, lässt sich weder entscheiden noch abnehmen
- Was nicht gemessen wird, wird im Betrieb zufällig erfüllt oder nicht
- Das Team darf die Zahl vorschlagen, bestätigen muss sie der Stakeholder, der mit der Folge lebt

## Quality Attribute Scenarios

Quelle, Auslöser, Artefakt, Umgebung, Reaktion, Maß

- **Vage**: "Die Anmeldung muss schnell sein" und "Es darf nichts schiefgehen"
- **Prüfbar**: "Melden sich 80 % der 6.000 Studierenden in den ersten 5 Minuten an, beantwortet das System 95 % der Anfragen in unter 800 ms"

- Ein Szenario pro Attribut: Performance und Konsistenz getrennt, sonst ist das Maß nicht prüfbar
- Daraus folgen Architekturoptionen: Cache für die Kursliste, Constraint in der Datenbank, Vorabreservierung

## Zwei Szenarien zerlegt

| Teil         | Performance                 | Konsistenz                             |
| ------------ | --------------------------- | -------------------------------------- |
| **Quelle**   | 4.800 Studierende           | zwei Studierende gleichzeitig          |
| **Auslöser** | melden sich in 5 Minuten an | wollen den letzten Platz               |
| **Artefakt** | Anmeldedienst               | Platzvergabe                           |
| **Umgebung** | Stichtag, Normalbetrieb     | parallele Requests                     |
| **Reaktion** | Antwort                     | einer den Platz, der andere die Absage |
| **Maß**      | 95 % unter 800 ms           | 0 Überbuchungen, im Lasttest geprüft   |

\rightarrow{} jede Zelle ist eine Entscheidungsgrundlage

## Fitness Functions

Automatisierte Prüfung von Architektureigenschaften, z. B.

- **Abhängigkeiten**: ArchUnit oder Linter-Regel, "die Domänenschicht importiert nichts aus der Infrastruktur"
- **Performance**: Lasttest in der Pipeline, Build rot bei p95 über 800 ms
- **Sicherheit**: Dependency-Scan, kein Deployment mit bekannter kritischer Lücke
- **Kosten**: Alarm, wenn die monatlichen Cloud-Kosten den Schwellwert überschreiten

\rightarrow{} Die Entscheidung bleibt gültig, auch wenn niemand mehr an sie denkt

## Typische Konflikte

- Konsistenz vs. Verfügbarkeit
- Latenz vs. Durchsatz
- Flexibilität vs. Einfachheit
- Time-to-Market vs. Wartbarkeit
- Kosten vs. Resilienz
- Sicherheit vs. Benutzbarkeit

## Trade-off-Analyse

- **Qualitativ vor quantitativ**: erst die Dimensionen finden (Betrieb, Skills, Latenz), dann messen, was messbar ist
- **Kriterien überschneidungsfrei und vollständig**: "Performance" ist Teil von "Skalierbarkeit"
- **Die Out-of-Context-Falle**: "Kafka ist schneller als RabbitMQ" gilt nur für den gemessenen Fall
- **Relevante Domänenfälle modellieren**: nicht "Queue oder Log", sondern "Bestätigungsmail nach der Anmeldung"
- **Vorsicht vor Evangelisten und Wundermitteln**: Wer nur eine Antwort kennt, hat die Frage nicht gehört

# Stakeholder und Kontext

## Wer entscheidet mit?

- **Business und Produkt**: Ziele, Budget, Zeit
- **Entwicklung**: Umsetzbarkeit, Skills, Motivation
- **Betrieb**: Betreibbarkeit, On-Call, Kosten
- **Security und Recht**: Compliance, Risiko
- **Endnutzer**: die eigentlichen Anforderungen

\rightarrow{} Stakeholder-Map zeigt Einfluss und Interesse

## Stakeholder-Map: das Anmeldesystem

|                      | Interesse niedrig                                       | Interesse hoch                                                   |
| -------------------- | ------------------------------------------------------- | ---------------------------------------------------------------- |
| **Einfluss hoch**    | Rechenzentrum: keine Alarme am Stichtag, geringe Kosten | Prüfungsämter: eng einbinden, keine Überbuchung, Liste am Montag |
| **Einfluss niedrig** | Dozenten: beobachten, Teilnehmerliste reicht            | Studierende: informieren, Fairness, keine Fehlermeldung um 8:00  |

- Wer nicht in der Tabelle steht, meldet sich nach dem Go-live

## Kontextfaktoren

- **Entwicklungsteam**: Größe, Skills, Erfahrung, Fluktuation
- **Organisation**: Art der Zusammenarbeit und Entscheidungswege
- **Bestand**: Legacy-Systeme, Verträge, vorhandene Plattformen
- **Regulatorik und Branche**
- **Zeit und Budget**
- **Zukunft**: erwartetes Wachstum, Lebensdauer des Systems

\rightarrow{} Managed Datenbank? Startup: ja. Bank mit Datenresidenz-Auflage: nein. Konzern mit DB-Team: vielleicht. Dieselbe Frage, andere Antwort

## Build, Buy oder Open Source

- Kernkompetenz vs. Commodity
- Total Cost of Ownership über die Lebensdauer, nicht Lizenzpreis gegen null
- Lock-in bewerten: Ausstiegskosten, Standards, Datenexport
- Managed Services: Betrieb einkaufen, Kontrolle abgeben
- Open Source ist nicht kostenlos, sondern anders bezahlt: Betrieb, Patches, Know-how

## Total Cost of Ownership: Managed Datenbank oder selbst betreiben

Größenordnungen über drei Jahre, eine mittlere Postgres-Instanz

| Posten                         |               Managed |     Selbst betrieben |
| ------------------------------ | --------------------: | -------------------: |
| Instanzen / VMs                |               9.000 € |              5.800 € |
| Einrichtung (PT à 800 €)       |           1 PT: 800 € |        5 PT: 4.000 € |
| Betrieb, Patches, Backup-Tests | 0,2 PT/Monat: 5.800 € | 1 PT/Monat: 28.800 € |
| **Summe**                      |          **15.600 €** |         **38.600 €** |

- Kippt bei vielen Datenbanken, vorhandenem DB-Team oder Datenschutzauflagen

\rightarrow{} Personalkosten sind in fast jeder Make-or-Buy-Rechnung der größte Posten

## Kognitive Last und langweilige Technologie

- Jedes neue Werkzeug kostet dauerhaft Aufmerksamkeit des Teams
- "Choose Boring Technology": Innovationsbudget bewusst einsetzen
- Hype-Zyklen erkennen: CV-Driven Development
- Wann neue Technologie trotzdem richtig ist: wenn sie ein Problem löst, das ihr wirklich habt

# Entscheidungen dokumentieren

## Architecture Decision Records (ADRs)

- Ein kurzes Dokument pro Entscheidung: Kontext, Optionen, Entscheidung, Konsequenzen
- Status und Lebenszyklus: vorgeschlagen, akzeptiert, abgelöst
- Versioniert, in der Sprache des Teams
- Eine Seite, nicht zehn: Was nicht auf eine Seite passt, ist mehrere Entscheidungen
- Wert: Das Warum bleibt erhalten, auch wenn das Team wechselt

## Anti-Patterns beim Entscheiden

- **Covering Your Assets:** aus Angst, falsch zu liegen, wird gar nicht oder zu spät entschieden
  - Gegenmaßnahme: Last Responsible Moment, Entscheidung mit Überprüfungstermin
- **Groundhog Day:** dieselbe Entscheidung wird alle paar Monate neu diskutiert, weil niemand das Warum kennt
  - Gegenmaßnahme: ADR mit Kontext und verworfenen Optionen
- **Email-Driven Architecture:** die Entscheidung steht in einer Mail, die niemand mehr findet
  - Gegenmaßnahme: ein Ort für Entscheidungen, im Repository, in Confluence, verlinkt statt kopiert

## ADR: Aufbau

- **Titel und Nummer:** kurz, als Aussage formuliert
- **Status:** vorgeschlagen, akzeptiert, abgelöst durch ADR-NNNN
- **Kontext:** Problem, Rahmenbedingungen, Anforderungen, Annahmen
- **Betrachtete Optionen:** mit Vor- und Nachteilen, auch die verworfenen
- **Entscheidung:** was wir tun und warum genau das
- **Konsequenzen:** positiv, negativ, offene Punkte, wann wir neu schauen

## ADR-0007: Titel und Kontext

**Bestätigungsmail asynchron versenden** (Status: akzeptiert)

### Kontext

- Stichtag je Standort: 16 Anmeldungen pro Sekunde im Mittel, Requests ein Vielfaches davon
- Externer Mail-Dienst: 99,5 % Verfügbarkeit, p95 800 ms, gelegentliche Timeouts
- Eine Anmeldung darf nicht fehlschlagen, weil der Mailversand klemmt
- Vier Entwickler, kein Kafka-Know-how, PostgreSQL ist gesetzt

## ADR-0007: Betrachtete Optionen

1. Synchroner Aufruf im Request
2. Message Queue (RabbitMQ oder SQS) mit eigenem Worker
3. Transactional Outbox in PostgreSQL mit Polling

## ADR-0007: die drei Optionen als Bild

```{.plantuml height=62%}
@startuml
left to right direction
skinparam defaultFontSize 15
skinparam ranksep 30
skinparam nodesep 15
cloud "Mail-Dienst" as M
rectangle "Option 1: synchron" {
  [API] as A1
  database "DB" as D1
}
rectangle "Option 2: Queue" {
  [API ] as A2
  database "DB " as D2
  queue "Queue" as Q2
  [Worker] as W2
}
rectangle "Option 3: Outbox" {
  [API  ] as A3
  database "DB + Outbox" as D3
  [Poller] as P3
}
A1 --> D1 : 1. Anmeldung
A1 --> M : 2. Mail im Request
A2 --> D2 : 1. Anmeldung
A2 --> Q2 : 2. Nachricht (Dual Write)
Q2 --> W2
W2 --> M
A3 --> D3 : 1. Anmeldung + Outbox, eine Transaktion
D3 --> P3 : 2. Poller liest
P3 --> M : 3. Mail, at-least-once
@enduml
```

- Option 2 ohne Outbox: Commit und Enqueue sind zwei Schritte (Dual Write), dazwischen kann die Nachricht verloren gehen

## ADR-0007: Entscheidung und Konsequenzen

### Entscheidung

Option 3: Die Mail-Aufgabe entsteht in derselben Transaktion wie die Anmeldung, ein Poller
verschickt sie. Kein neues System, kein Mailversand im Request.

### Konsequenzen

- **Positiv:** Anmeldung bleibt stabil, auch wenn der Mailanbieter ausfällt; Retry eingebaut
- **Negativ:** Mail kommt verzögert, der Poller ist Code, der Monitoring braucht
- **Negativ:** At-least-once, im Fehlerfall kommt eine Mail doppelt
- **Offen:** Ab welchem Volumen lohnt eine echte Queue? Prüfung nach dem ersten Stichtag

## Entscheidungen kommunizieren

- Diagramme als Kommunikationsmittel: C4-Modell (Kontext, Container, Komponenten)
- Dokumentation mit Struktur: arc42
- Entscheidung erklären, nicht verkünden: Wer das Warum kennt, kann im Zweifel richtig weiterentscheiden
- Zielgruppengerecht: Management bekommt Wirkung und Kosten, das Team bekommt die Begründung, z. B. bei ADR-0007:
  - fürs Management: "Die Anmeldung bleibt bei Mail-Ausfall nutzbar, Mails kommen bis zu einer Minute später"
  - fürs Team: Outbox, Poller, at-least-once

## KI als Werkzeug im Entscheidungsprozess

- LLMs für Optionsanalyse, ADR-Entwürfe, Recherche, Gegenargumente
- Grenzen: Halluzinationen, Bias zum Mainstream, zunächst kein Wissen über eure Organisation und euren Bestand
- Das Modell kennt den Durchschnitt des Internets, nicht euren Kontext \rightarrow{} Kontext entscheidet
- Die Entscheidung bleibt beim Menschen: Verantwortung und Begründung sind nicht delegierbar
- Gute Praxis: Kontext geben, mehrere Optionen und Gegenargumente verlangen, Quellen prüfen

## Zurück zu Aufgabe 1: Was hat die KI entschieden?

- Welche Annahmen hat das Werkzeug getroffen, ohne zu fragen?
- Welche davon waren Architekturentscheidungen?
- Was hättet ihr in den Prompt schreiben müssen, damit es eure Entscheidung bleibt?
- Was wäre ein guter Prompt für Aufgabe 2?

\rightarrow{} Nicht die KI die Führung übernehmen lassen

# Aufgabe 2

## Aufgabe 2: Fotodokumentation im Außendienst

- Ein Wartungsdienstleister mit 5.000 Monteuren, rund 2.000 Einsätze pro Tag
- Pro Einsatz 10 bis 50 Fotos à 3 MB, hochgeladen meist am Feierabend und oft mit schlechter Mobilfunkverbindung
- Jedes Foto wird verkleinert, auf Schadsoftware geprüft und der Text darauf erkannt, damit das Büro später suchen kann
- Das Büro sucht nach Einsatz, Kunde und Inhalt; Monteure sehen nur ihre eigenen Einsätze; die Fotos zeigen Kundenräume und manchmal Personen
- Aufbewahrung zehn Jahre, danach nachweisbar löschen, ohne dass das System dafür stillsteht

## Aufgabe 2: Ziel

1. Skizze der Komponenten und des Datenflusses
2. Zwei Entscheidungen **mit Begründung**: eine zu den Daten, eine zur Verarbeitung der Fotos
   - Rechnet, bevor ihr wählt: Wie viele Daten sind das nach zehn Jahren?
3. Eine davon als ADR
4. Welche Kontextfaktoren würden eure Entscheidungen zu was ändern?

- Nutzung von KI erlaubt
- Zeit: 20 Minuten
- Danach Besprechung / Präsentation

\rightarrow{} So eine Aufgabe könnte im Anwendungsteil der Klausur sein

## Aufgabe 2: Besprechung

- Was ist gegenüber Aufgabe 1 anders gelaufen?
- Welche Optionen habt ihr diesmal verworfen, und warum?
- Wie viele Daten sind das nach zehn Jahren, und wo liegen sie?
- Wie löscht ihr nach zehn Jahren, ohne das System anzuhalten, und woher wisst ihr, dass alles weg ist?
- Wer darf welches Foto sehen, und wo wird das geprüft?
- Was passiert, wenn alle Monteure um 17 Uhr gleichzeitig hochladen?
- Welcher Kontextfaktor würde eure Entscheidung kippen?

## Aufgabe 2: eine mögliche Lösung

```{.plantuml height=85%}
@startuml
left to right direction
skinparam defaultFontSize 15
skinparam ranksep 25
skinparam nodesep 12
actor "Monteur" as Mo
actor "Büro" as B
[API] as API
database "Object\nStorage" as OS
database "PostgreSQL\nEinsätze, Fotos,\npartitioniert nach Monat" as DB
queue "Ereignis-Log" as Q
[Verkleinern] as R
[Virenscan] as V
[Texterkennung] as O
database "Suchindex" as IX
Mo --> API : Upload je Foto,\nwiederholbar
B --> API : Suche
API --> OS : Datei
API --> DB : Metadaten +\nEreignis (Outbox)
DB --> Q
Q -right-> R
Q -right-> V
Q -right-> O
R -right-> OS : Vorschau
O -right-> IX : Text
API -right-> IX : Rechte geprüft
@enduml
```

## Aufgabe 2: die Entscheidungen dahinter

- Daten: Metadaten in PostgreSQL, Originale und Vorschauen im Object Storage, Volltext im Suchindex; ca. 650 TB gehören nicht in die Datenbank
- Partitionierung jeden Monat: 220 Mio. Zeilen bleiben handhabbar, Löschen nach zehn Jahren heißt Partition entfernen
- Verarbeitung: Log statt Queue, weil drei Konsumenten dasselbe Ereignis brauchen; Upload je Foto und wiederholbar
- Rechte: Monteur sieht nur eigene Einsätze, geprüft in der API, nicht in der App
- Texterkennung austauschbar: erst klassische OCR, ein LLM nur, wenn es gemessen besser ist
- Bewusst nicht: Komprimierung beim Client, Verarbeitung synchron im Upload, Fotos in der Datenbank; Annahme: Fotos verlassen das Rechenzentrum nicht

## Zusammenfassung

- Architektur ist die Summe der schwer umkehrbaren Entscheidungen
- Trade-offs sind unvermeidlich; die Aufgabe ist, sie sichtbar zu machen
- Kontext und Stakeholder bestimmen, was richtig ist, nicht die Technologie
- Was kein Maß hat, ist keine Anforderung, sondern ein Wunsch
- ADRs halten das Warum fest, Experimente ersetzen Meinungen

## Nächste Vorlesung

- Resilienz, Fehlertoleranz & Hochverfügbarkeit (Lukas)

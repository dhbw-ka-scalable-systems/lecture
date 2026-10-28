---
title: "Skalierbare Systeme - Architekturstile"
topic: "scalable_systems_1_5"
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

- Grundbegriffe: Architekturstil, Kopplung, Kohäsion
- Monolithische Stile: Schichten, Modular Monolith
- Verteilte Stile: Services, Microservices, Event-Driven, Serverless
- Stile vergleichen, wählen, migrieren

## Lernziele

- Die wichtigsten Architekturstile
  - Schnitt, Deployment, Daten und Kommunikation unterscheiden
- Den Stil eines bestehenden Systems erkennen und seine Spielregeln einhalten
- Stärken und Schwächen eines Stils gegen Qualitätsattribute bewerten
- Für einen Kontext einen Stil wählen und die Entscheidung begründen (ADR)
- Wissen, wie man von einem Stil zum nächsten kommt und was das kostet

## Warum Architekturstile?

- Ihr steigt in bestehende Systeme ein, selten auf die grüne Wiese
- Jeder Stil hat Spielregeln; wer sie nicht kennt, bricht sie versehentlich
- KI schreibt den Code - Welche Teile, welche Grenzen, welche Kommunikation bleibt eure Entscheidung

## Was wir schon haben

- Vorlesung 1: Monolith vs. Microservices, synchron vs. asynchron, Queues und Logs, API Gateway
- Vorlesung 2: Outbox Pattern, Daten trennen oder zusammenhalten, Mandantenmodelle
- Vorlesung 3: Architekturentscheidungen, Trade-offs, ADRs
- Vorlesung 4: unzuverlässige Netze, Timeouts, Circuit Breaker, Bulkheads

\rightarrow{} Heute werden die Bausteine zum Gesamtbild: Welche Grundstruktur hat ein System, und warum?

# Grundbegriffe

## Was ist ein Architekturstil?

Die Grundstruktur eines Systems: Welche Teile gibt es, wie sind sie geschnitten, wie reden sie miteinander?

- Baustein, Pattern, Stil:
  - **Baustein:** Load Balancer, Queue, Cache (Vorlesung 1)
  - **Pattern:** Lösung für ein Teilproblem: Outbox, Circuit Breaker, Saga
  - **Stil:** Struktur des Ganzen: Schichten, Microservices, Event-Driven
- Reale Systeme mischen Stile; "der Stil" beschreibt den dominanten Aufbau

## Vier Fragen, die einen Stil ausmachen

1. **Schnitt:** Nach welchem Kriterium ist der Code zerlegt: nach Technik oder nach Fachlichkeit?
2. **Deployment:** Wie viele unabhängig deploybare Einheiten gibt es: eine oder viele?
3. **Daten:** Eine gemeinsame Datenbank oder Daten je Teil?
4. **Kommunikation:** Methodenaufruf, synchroner Aufruf über das Netz oder asynchrone Nachricht?

- Die vier Antworten zusammen sind der Stil, der Rest sind Varianten

\rightarrow{} Dieselben vier Fragen helfen, ein fremdes System in der ersten Woche einzuordnen

## Fachlich vs. technisch partitioniert

```{.plantuml height=70%}
@startuml
skinparam defaultFontSize 15
skinparam ranksep 20
skinparam nodesep 15
rectangle "Fachlich partitioniert" {
  rectangle "Kurse\nUI, Logik,\nDaten" as F1
  rectangle "Anmeldung\nUI, Logik,\nDaten" as F2
  rectangle "Benachrichtigung\nLogik, Daten" as F3
  F1 -> F2
  F2 -> F3
}
rectangle "Technisch partitioniert" {
  rectangle "Presentation\nKurs-, Anmelde-, Mail-Controller" as T1
  rectangle "Business\nKurs-, Anmelde-, Mail-Service" as T2
  rectangle "Persistence\nKurs-, Anmelde-, Mail-Repository" as T3
  T1 --> T2
  T2 --> T3
}
@enduml
```

- Fachlich: Eine Änderung bleibt meist in einem Teil
- Technisch: Ein Feature berührt alle Schichten

## Kopplung

Kopplung: Wie viel muss sich an A ändern, wenn B sich ändert, und was passiert mit A, wenn B ausfällt?

- Vier Arten, die im Alltag zählen:
  - **Code:** A importiert B und kennt dessen Datenstrukturen
  - **Daten:** A und B schreiben dieselbe Tabelle
  - **Zeit:** A wartet synchron auf B; ist B langsam, ist A langsam
  - **Deployment:** A und B können nur gemeinsam ausgeliefert werden

\rightarrow{} Kopplung ist nicht schlecht; sie soll dort liegen, wo sie günstig ist: innerhalb einer Fachlichkeit, nicht quer darüber

## Kohäsion

Kohäsion: Wie sehr gehört zusammen, was in einem Teil zusammenliegt?

- Hoch: Das Modul "Anmeldung" enthält alles, was eine Anmeldung ausmacht: Regeln, Plätze, Status
- Niedrig: Ein Modul "Utils" mit Datumsformatierung, PDF-Export und Mailversand

Prüffrage: Wenn sich die x ändert, wie viele Teile muss ich anfassen?

\rightarrow{} Faustregel: hohe Kohäsion innen, lose Kopplung außen

## Zur Erinnerung: das Anmeldesystem (Vorlesung 3)

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

## Aufgabe 1: Kopplung finden

**Arbeitszeit: 5 Minuten**

1. Welche Kopplungen seht ihr in der Skizze? Benennt für jede die Art: Code, Daten, Zeit, Deployment
2. Welche davon ist gewollt, welche würde euch beim Wachsen des Systems stören?

Ergebnis: eine Kopplung, die ihr auflösen würdet, mit Begründung

<!-- Erwartung: API-DB zeitlich und über Daten gekoppelt (gewollt, Transaktion); API-Mail über Outbox zeitlich entkoppelt (gewollt);
API-Cache zeitlich gekoppelt, aber degradierbar; alles ein Deployment. Stoeren wuerde beim Wachsen: alles in einem Prozess, ein Release. -->

## Bewertungsmaßstab

Die Qualitätsattribute aus Vorlesung 3, heute als Maßstab für Stile

- Skalierbarkeit und Elastizität, Fehlertoleranz
- Deployability: Wie schnell und wie risikoarm kommt eine Änderung in Produktion?
- Evolvierbarkeit und Testbarkeit
- Einfachheit und Kosten: Entwicklung, Betrieb, Lernaufwand

# Monolithische Stile

## Der Monolith

- Eine deploybare Einheit: ein Prozess, ein Build, ein Release
- Meist eine Datenbank; Aufrufe sind Methodenaufrufe: schnell, transaktional, einfach zu debuggen
- Die meisten Systeme, in die ihr einsteigt, sind Monolithen, und das ist oft richtig so
- Monolith ist ein Deployment-Begriff, kein Urteil über die innere Struktur
- Innere Struktur: Schichten, fachliche Module oder (Vorsicht) keine

## Schichtenarchitektur (Layered)

```{.plantuml height=62%}
@startuml
skinparam defaultFontSize 15
skinparam ranksep 18
rectangle "Presentation\nController, Views, REST-Endpunkte" as P
rectangle "Business\nFachregeln: Platzvergabe, Stichtage, Mandanten" as B
rectangle "Persistence\nRepositories, SQL" as D
database "Datenbank" as DB
P --> B
B --> D
D --> DB
note right of B
  Aufrufe nur von oben nach unten.
  Jede Schicht kennt nur die darunter.
end note
@enduml
```

## Schichten: Spielregeln

- Geschlossene Schichten: Jede Anfrage durchläuft alle Schichten; offene Schichten dürfen übersprungen werden
- Schicht ist nicht Tier: Schichten sind logisch (Code), Tiers physisch (Deployment); drei Schichten passen auf einen Server
- Fachlogik gehört in die Business-Schicht, nicht in den Controller und nicht ins SQL
- Sinkhole-Antipattern: Schichten reichen nur durch, ohne etwas zu tun; wenn das die Regel ist, sind es zu viele Schichten

## Schichten: Stärken und Schwächen

- **Stärken:** einfach zu verstehen, schnell gestartet, jedes Framework unterstützt es
  - Gut für kleine Teams, erste Versionen und Domänen, die noch nicht verstanden sind

- **Schwächen:** technisch partitioniert: Ein Feature berührt alle Schichten
  - Teams nach Schichten (Frontend, Backend, Datenbank) müssen jede Änderung koordinieren
  - Skalierung und Deployment nur als Ganzes
  - Mit der Zeit wachsen Querverbindungen, aus Schichten wird ein Big Ball of Mud

## Big Ball of Mud

Kein erkennbarer Stil: Jeder Teil kennt jeden, Daten werden überall geändert

- Entsteht nicht durch Absicht, sondern durch tausend kleine Abkürzungen unter Zeitdruck
- Symptome: Angst vor Änderungen, lange Testzyklen, "das weiß nur noch Kollegin X"
- Das Problem ist nicht der Monolith, sondern das Fehlen durchgesetzter Grenzen
- Gilt auch für KI-generierten Code: Ohne Regeln nimmt das Werkzeug jede Abkürzung, die der Code erlaubt

## Modular Monolith (Modulith)

- Fachliche Module mit klaren öffentlichen Schnittstellen, ein Deployment
- Jedes Modul hat eine kleine öffentliche API, der Rest ist privat
- Eine Datenbank, aber ein Schema je Modul; Zugriff nur über das besitzende Modul
- Module reden über Schnittstellen oder In-Process-Events, nicht über Datenbank-Joins

## Modulith: Grenzen durchsetzen

Grenzen, die nur in der Doku stehen, erodieren beim ersten Zeitdruck

- Compiler und Build: getrennte Packages oder Projekte, Sichtbarkeiten (`internal`, package-private)
- Architekturtests: ArchUnit prüft "Anmeldung greift nicht auf Benachrichtigung.intern zu" im CI
- Reviews: "Warum braucht dieses Modul jenes?"
- Nutzen: Grenzen, die später Service-Grenzen werden können, ohne heute den Preis des Netzwerks zu zahlen

## Das Anmeldesystem als Modulith

```{.plantuml height=85%}
@startuml
skinparam defaultFontSize 15
skinparam ranksep 30
skinparam nodesep 25
actor "Studierende" as S
rectangle "Anmeldesystem, ein Deployment" {
  [Kurskatalog] as K
  [Anmeldung] as A
  [Standorte] as T
  [Benachrichtigung] as N
}
database "PostgreSQL\nein Schema je Modul,\nZugriff nur über das eigene Modul" as DB
cloud "Mail-Dienst" as M
S --> K : Kurse ansehen
S --> A : anmelden
A -> K : Kurs lesen (API)
A -> T : Mandant prüfen (API)
A ..> N : Ereignis "Anmeldung bestätigt"\n(in-process)
K --> DB
A --> DB
T --> DB
N --> DB
N --> M : Outbox, at-least-once
@enduml
```

## Ports and Adapters (Hexagonal)

```{.plantuml height=60%}
@startuml
left to right direction
skinparam defaultFontSize 15
skinparam ranksep 30
rectangle "Adapter (eingehend)" {
  [REST-Controller] as R
  [Scheduler / CLI] as C
}
rectangle "Fachkern Anmeldung\nPlatzvergabe, Regeln\nkennt weder HTTP noch SQL" as H
rectangle "Adapter (ausgehend)" {
  [PostgreSQL-Repository] as P
  [In-Memory-Repository (Test)] as T
  [Mail-Adapter] as M
}
R --> H : Port "Anmelden"
C --> H : Port "Anmelden"
H --> P : Port "Plätze"
H --> T : Port "Plätze"
H --> M : Port "Benachrichtigen"
@enduml
```

- Fachlichkeit innen, Technik außen \rightarrow{} testbar ohne Infrastruktur, Technik austauschbar
- Einordnung: Struktur innerhalb eines Moduls oder Services, kein Stil des Gesamtsystems (ebenso Clean und Onion Architecture)

## Den Monolithen skalieren

```{.plantuml height=62%}
@startuml
left to right direction
skinparam defaultFontSize 15
skinparam ranksep 25
skinparam nodesep 10
actor Nutzer as U
[Load Balancer] as LB
rectangle "Anmeldesystem, zustandslos" {
  [Instanz 1] as I1
  [Instanz 2] as I2
  [Instanz n] as I3
}
[Worker\n(gleicher Code)] as W
queue "Queue" as Q
database "Primary" as DB
database "Read Replica" as RR
database "Cache" as C
U --> LB
LB --> I1
LB --> I2
LB --> I3
I1 --> C
I1 --> DB : schreiben
I1 --> RR : lesen
I1 --> Q : langsame Arbeit
Q --> W
DB --> RR : Replikation
@enduml
```

## Grenzen des Monolithen

- **Deployment:** Jedes Release enthält alles; ein Fehler in der Statistik blockiert das Anmelde-Feature
- **Teams:** Ab etwa drei bis vier Teams auf einem Code-Stand steigt der Koordinationsaufwand: Merge, Release, Testlauf
- **Last:** Alle Teile skalieren gemeinsam, auch wenn nur die Anmeldung Last hat
- **Technologie:** eine Sprache, eine Laufzeit, ein Upgrade-Zyklus
- **Fehlerisolation:** Ein Speicherleck legt alles lahm

\rightarrow{} Keiner dieser Punkte ist bei vier Entwicklern und überschaubarer Codebase ein Problem

# Verteilte Stile

## Verteilen heißt: ein Netzwerk dazwischen

- Aus dem Methodenaufruf wird ein Netzwerkaufruf \rightarrow{} langsamer, kann fehlschlagen, kann doppelt ankommen
- Aus einer Transaktion werden mehrere: Konsistenz wird zur Entwurfsfrage
- Fowler: "First Law of Distributed Object Design: Don't distribute your objects"

\rightarrow{} grobe Schnittstellen, wenige Aufrufe, Verteilung nur mit Grund

## Service-basierte Architektur

```{.plantuml height=55%}
@startuml
left to right direction
skinparam defaultFontSize 15
skinparam ranksep 30
skinparam nodesep 10
actor Nutzer as U
[Web-UI] as UI
[Kursverwaltung] as K
[Anmeldung] as A
[Reporting] as R
database "Gemeinsame Datenbank" as DB
U --> UI
UI --> K
UI --> A
UI --> R
K --> DB
A --> DB
R --> DB
@enduml
```

- Wenige, grobe fachliche Services (typisch 4 bis 12), getrennt deploybar, meist mit gemeinsamer Datenbank
- Der pragmatische Mittelweg zwischen Modulith und Microservices

## Service-basiert: Stärken und Schwächen

**Stärken:** Deployment und Skalierung je Service, ohne Daten zu verteilen; Transaktionen bleiben möglich

- Überschaubarer Betrieb, kein Messaging nötig, Schritt vom Modulith ohne großen Umbau
- Fehlerisolation: Reporting kann abstürzen, die Anmeldung läuft weiter

**Schwächen:** Die gemeinsame Datenbank koppelt: Eine Schemaänderung betrifft alle Services

- Die Last aller Services bündelt sich auf einer Datenbank
- Oft die bessere Wahl als Microservices, wird aber selten so genannt

## Microservices

```{.plantuml height=60%}
@startuml
left to right direction
skinparam defaultFontSize 15
skinparam ranksep 30
skinparam nodesep 10
actor Client as C
[API Gateway] as G
[Service A] as A
database "DB A" as DA
[Service B] as B
database "DB B" as DBB
[Service C] as SC
database "DB C" as DC
queue "Broker" as Q
C --> G
G --> A
G --> B
A --> DA
B --> DBB
SC --> DC
A --> B : REST / gRPC
B ..> Q : Event
Q ..> SC
@enduml
```

- Viele kleine Services, jeder mit eigenen Daten, unabhängig deploybar, meist ein Team je Service

## Microservices: Prinzipien

- **Schnitt entlang der Fachlichkeit**: Bounded Context, ein Begriff hat im Service genau eine Bedeutung
- **Datenhoheit**: Nur der Service schreibt seine Daten; andere fragen per API oder hören auf Events
- **Unabhängig** deployen und skalieren; Technologie je Service frei (in der Praxis: wenige erlaubte Stacks)
- **Ein Team** besitzt einen Service, Entwicklung und Betrieb: "you build it, you run it"

Microservices müssen nicht klein sein
\rightarrow{} Größe ist nicht das Kriterium, Unabhängigkeit ist es; der Begriff "micro" ist historisch

## Microservices: Kommunikation

- **Synchron**: REST oder gRPC; einfach, aber zeitliche Kopplung und Aufrufketten
- **Asynchron**: Nachrichten und Ereignisse über einen Broker; entkoppelt, aber Eventual Consistency
- Außen: API Gateway bündelt Auth, Limits, Routing

Faustregel: synchron für Fragen ("Wie viele Plätze?"), asynchron für Tatsachen ("Anmeldung bestätigt")

## Microservices: Daten

- **Keine gemeinsame Datenbank**: API-Aufrufe oder lokale Kopien
- **Transaktionen** über Services: kein 2PC, sondern Saga: Folge lokaler Transaktionen mit Kompensation
- **Datenduplikation** ist der bewusste Preis: Der Anmeldedienst speichert Kurs-ID und Kursname, nicht den ganzen Kurs
- **Eventual Consistency**: Daten sind kurz inkonsistent; das muss der Fachbereich akzeptieren und die UI zeigen

## Saga (Entwurfsmuster): Orchestrierung vs. Choreografie

```{.plantuml height=62%}
@startuml
left to right direction
skinparam defaultFontSize 15
skinparam ranksep 25
skinparam nodesep 10
rectangle "Choreografie" {
  [Bestellung ] as B2
  queue "Events" as Q
  [Zahlung ] as Z2
  [Lager ] as L2
  B2 --> Q : BestellungAngelegt
  Q --> Z2
  Z2 --> Q : ZahlungGebucht
  Q --> L2
  L2 --> Q : NichtLieferbar
  Q --> Z2 : erstatten
}
rectangle "Orchestrierung" {
  [Orchestrator\n"Bestellung abschließen"] as O
  [Bestellung] as B1
  [Zahlung] as Z1
  [Lager] as L1
  O --> B1 : 1. anlegen
  O --> Z1 : 2. abbuchen
  O --> L1 : 3. reservieren
  O ..> Z1 : bei Fehler:\nerstatten
}
@enduml
```

- Orchestrierung: Ablauf an einer Stelle, Koordinator koppelt
- Choreografie: entkoppelt, Ablauf nirgends sichtbar

## Microservices: der Preis

- Verteilte Komplexität: Netzwerk, Teilausfälle, Debugging über Service-Grenzen, Versionierung von Schnittstellen
- Betrieb: Pipeline, Monitoring, Secrets, Security je Service; ohne Plattform-Team nicht tragbar
- Organisation: braucht Teams, die Services besitzen; ein Team von fünf mit zwölf Services besitzt nichts davon
- Verteilter Monolith: viele Services, die nur gemeinsam deploybar sind: alle Nachteile, kein Vorteil
- Wann nicht: kleines Team, unklare Domäne, Start eines Produkts, keine Betriebserfahrung

## Das Anmeldesystem als Microservices

```{.plantuml height=85%}
@startuml
skinparam defaultFontSize 15
skinparam ranksep 30
skinparam nodesep 20
actor "Studierende" as S
[API Gateway\nAuth, Limits, Routing] as G
[Kursdienst] as K
database "Kurse-DB" as KD
[Anmeldedienst] as A
database "Anmeldungen-DB\nPlätze per Constraint" as AD
[Identität und Standorte] as I
database "Standorte-DB" as ID
queue "Broker" as Q
[Benachrichtigungsdienst] as N
cloud "Mail-Dienst" as M
S --> G
G --> K
G --> A
G --> I : Token prüfen
A -> K : Kurs lesen (sync)
A --> AD
K --> KD
I --> ID
A ..> Q : "Anmeldung bestätigt"\n(Outbox)
Q ..> N
N --> M
@enduml
```

## Was wurde schwerer?

- Kurs und Plätze liegen in zwei Services: Was passiert, wenn der Kurs gelöscht wird, während Anmeldungen laufen?
- Vier Deployments, drei Datenbanken, ein Gateway, ein Broker
- Am Stichtag: Aufrufkette Gateway \rightarrow{} Anmeldung \rightarrow{} Kurs; jeder Hop kostet Latenz und kann ausfallen

- Richtig gemacht: Die Platzvergabe bleibt in einem Service mit einer Transaktion, der Überbuchungsschutz braucht keine Saga
- Gewonnen: unabhängige Releases und Skalierung; nötig erst, wenn Teams oder Last das verlangen

\rightarrow{} Prüfsatz: "Wir brauchen unabhängiges Deployment von X, weil ..." Wer ihn nicht vervollständigen kann, braucht keine Microservices

## Pipes and Filters

```{.plantuml height=28%}
@startuml
left to right direction
skinparam defaultFontSize 15
skinparam nodesep 10
[Upload] as U
[Verkleinern] as R
[Virenscan] as V
[Texterkennung] as O
database "Suchindex" as I
U --> R : Foto
R --> V : Foto + Vorschau
V --> O : geprüft
O --> I : Text
@enduml
```

- Daten fließen durch eine Kette unabhängiger Schritte; jeder Filter kennt nur seine Ein- und Ausgabe
- Stärken: Schritte austauschen, umordnen, parallel ausführen
- Schwächen: Zustand und Fehlerbehandlung über Schritte hinweg
- Heute meist als Teil einer Datenpipeline oder eines Event-Driven-Systems

## Event-Driven Architecture (EDA)

```{.plantuml height=45%}
@startuml
left to right direction
skinparam defaultFontSize 15
skinparam ranksep 40
skinparam nodesep 10
[Produzent] as P
queue "Broker\nTopic" as T
[Konsument A] as A
[Konsument B] as B
[Konsument C\n(später dazugekommen)] as C
P --> T : Ereignis
T --> A
T --> B
T ..> C
note bottom of P
  kennt A, B und C nicht
end note
@enduml
```

- Ereignisse statt Aufrufe: "Anmeldung bestätigt" statt "schick eine Mail"
- Neue Funktion heißt neuer Konsument, ohne den Produzenten anzufassen

## EDA: Events, Commands, Queues, Topics

- **Event:** Es ist etwas passiert; Vergangenheitsform, viele Empfänger, keine Antwort erwartet
- **Command:** Tu etwas; ein Empfänger, kann abgelehnt werden
- **Queue:** eine Nachricht, ein Konsument, Arbeit verteilen
- **Topic oder Log:** eine Nachricht, alle Abonnenten; Log zusätzlich mit Historie und Replay

## Topologien: Broker vs. Mediator

```{.plantuml height=72%}
@startuml
left to right direction
skinparam defaultFontSize 15
skinparam ranksep 25
skinparam nodesep 8
rectangle "Mediator-Topologie (Orchestrierung)" {
  [Anmeldung ] as A2
  [Mediator\nAblauf "Anmeldung verarbeiten"] as Me
  [Warteliste ] as W2
  [Mail ] as M2
  [Statistik ] as S2
  A2 --> Me : AnmeldungBestätigt
  Me --> W2 : 1.
  Me --> M2 : 2.
  Me --> S2 : 3.
}
rectangle "Broker-Topologie (Choreografie)" {
  [Anmeldung] as A1
  queue "Topic 1" as Q1
  [Warteliste] as W1
  queue "Topic 2" as Q2
  [Mail] as M1
  [Statistik] as S1
  A1 --> Q1 : AnmeldungBestätigt
  Q1 --> M1
  Q1 --> W1
  W1 --> Q2 : Nachgerückt
  Q2 --> M1
  Q2 --> S1
}
@enduml
```

- Broker: maximal entkoppelt, Gesamtablauf nirgends sichtbar
- Mediator: Ablauf und Fehlerbehandlung an einer Stelle, dafür ein Koppelpunkt

## EDA: Stärken und Schwächen

**Stärken:**

- lose Kopplung: Konsumenten einzeln skalierbar, Lastspitzen werden gepuffert
- Erweiterung ohne Änderung des Produzenten \rightarrow{} passt zu Systemen, die auf Dinge reagieren

**Schwächen:**

- Der Gesamtablauf steht in keinem Code: "Wer reagiert eigentlich auf dieses Event?"
- Reihenfolge, Duplikate, verlorene Events, Weiterentwicklung der Event-Formate
- Debugging: "Was ist mit Anmeldung 4711 passiert?"

## Das Anmeldesystem event-driven

```{.plantuml height=68%}
@startuml
skinparam defaultFontSize 15
skinparam ranksep 24
skinparam nodesep 20
actor "Studierende" as S
rectangle "Kern: synchron, transaktional" {
  [Anmeldung] as A
  database "PostgreSQL\nPlätze + Outbox" as DB
}
[Outbox-Poller] as P
queue "Topic anmeldung.bestaetigt" as T
[Mail] as M
[Warteliste] as W
[Statistik] as ST
[Export Prüfungsamt] as E
S --> A : anmelden, Antwort sofort
A -> DB : eine Transaktion
DB --> P
P --> T
T --> M
T --> W
T --> ST
T ..> E : neu, Kern unverändert
@enduml
```

- Die Platzvergabe bleibt eine Frage mit sofortiger Antwort; alles danach ist Reaktion auf eine Tatsache

## Serverless und Functions as a Service

```{.plantuml height=60%}
@startuml
left to right direction
skinparam defaultFontSize 15
skinparam ranksep 30
skinparam nodesep 10
actor "Studierende" as S
[API Gateway] as G
rectangle "Functions, Laufzeit vom Anbieter betrieben" {
  [anmelden()] as F1
  [kurseListen()] as F2
  [mailSenden()] as F3
  [exportErzeugen()] as F4
}
database "Managed DB" as DB
queue "Event-Bus" as Q
cloud "Mail-Dienst" as M
S --> G
G --> F1
G --> F2
F1 --> DB
F2 --> DB
F1 ..> Q : Event
Q ..> F3
Q ..> F4
F3 --> M
@enduml
```

- Code als Funktionen, der Anbieter betreibt die Laufzeit
- Skalierung von null bis zur Lastspitze
- Bezahlung pro Aufruf
- Typischer Einsatz: Event-Verarbeitung, Glue-Code, sporadische Aufgaben; selten das ganze System
- Mischung ist üblich: z. B. Kern als Container, Mailversand und Exporte als Funktionen

# Stile vergleichen und wählen

## Vergleich

\footnotesize

| Stil               | Einfach | Deploy | Skalierung | Fehlertoleranz | Evolvierbar | Betrieb  |
| :----------------- | :-----: | :----: | :--------: | :------------: | :---------: | :------: |
| Schichten-Monolith |   +++   |   +    |     +      |       +        |      +      | günstig  |
| Modular Monolith   |   ++    |   +    |     +      |       +        |     ++      | günstig  |
| Service-basiert    |   ++    |   ++   |     ++     |       ++       |     ++      |  mittel  |
| Microservices      |    +    |  +++   |    +++     |      +++       |     +++     |  teuer   |
| Event-Driven       |    +    |   ++   |    +++     |      +++       |     +++     |  teuer   |
| Serverless         |   ++    |  +++   |    +++     |       ++       |     ++      | variabel |

\normalsize

## Erste Woche in einem neuen Projekt: den Stil erkennen

- Wie viele Dinge werden deployt: ein Artefakt oder viele mit eigener Pipeline?
- Wie viele Datenbanken, und wer schreibt in welche Tabelle?
- Wie reden die Teile: Methodenaufruf, HTTP, Nachrichten? Gibt es einen Broker?
- Wie sind Packages und Repos geschnitten: Controller/Service/Repository oder Kurse/Anmeldung/Mail?
- Wer darf was ändern, ohne jemanden zu fragen? Das ist die echte Grenze
- Deckt sich das mit dem, was das Team über seine Architektur sagt?

## Schneiden: Wo liegen die Grenzen?

- **Nach Fachlichkeit**, nicht nach Technik und nicht nach Tabellen
- **Bounded Context**: Wo wechselt die Bedeutung eines Begriffs? "Kurs" im Katalog (Beschreibung), in der Anmeldung (Plätze), im Prüfungsamt (Note)
- Was sich gemeinsam ändert, gehört zusammen; was anders skaliert oder anders oft released wird, darf getrennt sein

Ein falscher Schnitt lässt sich im Modulith günstig korrigieren, zwischen Services teuer

## Migrationspfade

```{.plantuml height=30%}
@startuml
left to right direction
skinparam defaultFontSize 15
skinparam nodesep 25
rectangle "Monolith\n(Schichten)" as A
rectangle "Modular\nMonolith" as B
rectangle "Service-\nbasiert" as C
rectangle "Micro-\nservices" as D
A --> B : Grenzen im Code
B --> C : Grenzen im Deployment
C --> D : Grenzen in den Daten
@enduml
```

- Erst Grenzen im Code, dann im Deployment, zuletzt in den Daten (Daten trennen ist der schwierigste Schritt)
- Kein Big Bang: Das neue System muss neben dem alten laufen können

## Strangler Fig

```{.plantuml height=52%}
@startuml
left to right direction
skinparam defaultFontSize 15
skinparam ranksep 30
skinparam nodesep 10
actor Clients as C
[Fassade / Proxy\nRouting je Funktion] as F
rectangle "Altsystem" {
  [Kurse] as K
  [Anmeldung] as A
  [Prüfungen (alt)] as P
}
[Prüfungsdienst (neu)] as PN
database "Prüfungs-DB (neu)" as PD
C --> F
F --> K
F --> A
F ..> P : vorher
F --> PN : nachher
PN --> PD
@enduml
```

- Schrittweise ersetzen: Eine Fassade leitet Funktion für Funktion zum neuen System, das alte stirbt langsam ab
- Jeder Schritt ist rückgängig machbar; die Datenmigration ist der schwierige Teil

## Stil wählen: Kontextfaktoren

- **Team:** Größe, Anzahl Teams, Betriebserfahrung, Plattform vorhanden?
- **Domäne:** verstanden oder noch in Bewegung? Sind die Grenzen klar?
- **Last:** gleichmäßig oder Spitzen, gleich für alle Teile oder sehr unterschiedlich?
- **Änderungen:** Wie oft, wie unabhängig voneinander, wie risikoreich?
- **Vorgaben:** Budget, Compliance, Cloud oder Rechenzentrum, Bestandssystem
- Standardantwort: Modular Monolith; Abweichung mit Begründung, dokumentiert als ADR

## Aufgabe 2: das Anmeldesystem drei Jahre später

- Das System aus Vorlesung 3 läuft seit drei Jahren als Modulith: Kurskatalog, Anmeldung, Benachrichtigung, Standorte, eine PostgreSQL
- Neu gewünscht: Prüfungsanmeldung mit Notenabruf, Wartelisten mit Nachrückern, Export ans Prüfungsamt, Kursbewertungen
- Inzwischen 25 Entwickler in fünf Teams; Releases blockieren sich gegenseitig, der Testlauf dauert 50 Minuten
- 20 Standorte, davon drei im Ausland; Spitze am Stichtag 60 Anmeldungen pro Sekunde, Requests ein Vielfaches
- Das Prüfungsamt verlangt Prüfungsdaten in einem eigenen, auditierbaren System mit zehn Jahren Historie
- Zwei Standorte verlangen den Betrieb im eigenen Rechenzentrum

## Aufgabe 2: Ziel

1. Welchen Stil wählt ihr für das Gesamtsystem, und welche Teile löst ihr heraus (oder nicht)?
2. Skizze: deploybare Einheiten, Datenhaltung, Kommunikation (synchron oder asynchron)
3. Eine Entscheidung als ADR: Kontext, Optionen, Entscheidung, Konsequenzen
4. Welche Anforderung hat euch am stärksten Richtung Verteilung gedrängt, welche dagegen?

- Nutzung von KI erlaubt: Lasst euch Alternativen und Gegenargumente geben, nicht nur einen Vorschlag
- Danach Besprechung

\rightarrow{} So eine Aufgabe könnte im Anwendungsteil der Klausur sein

## Aufgabe 2: Besprechung

- Hat jemand den Modulith behalten? Mit welcher Begründung?
- Welcher Teil wurde zuerst herausgelöst, und warum dieser?
- Wo bleibt die Platzvergabe, und wo liegt ihre Transaktion?
- Wie kommt das Prüfungsamt an seine Daten: Aufruf, Event, Replikation, eigener Service?
- Was hat "eigenes Rechenzentrum" geändert: den Stil, das Deployment oder nur die Konfiguration?
- Was hat die KI vorgeschlagen, und wo habt ihr widersprochen?

## Aufgabe 2: eine mögliche Lösung

```{.plantuml height=85%}
@startuml
skinparam defaultFontSize 15
skinparam ranksep 28
skinparam nodesep 18
actor "Studierende" as S
actor "Prüfungsamt" as PA
[API Gateway] as G
rectangle "Anmeldesystem: Modulith, Teams 1 bis 3" {
  [Kurskatalog] as K
  [Anmeldung + Warteliste] as A
  [Bewertungen] as B
  [Standorte] as T
}
database "PostgreSQL\nSchema je Modul" as DB
rectangle "Prüfungsdienst: Team 4" {
  [Prüfungen] as P
  database "Prüfungs-DB\nEvent Store, 10 Jahre" as PD
}
queue "Topic" as Q
rectangle "Konsumenten: Team 5" {
  [Benachrichtigung] as N
  [Export Prüfungsamt] as E
}
cloud "Mail-Dienst" as M
S --> G
PA --> G
G --> K
G --> A
G --> P
K --> DB
A --> DB
B --> DB
T --> DB
P --> PD
P -> A : Anmeldung prüfen (sync)
A ..> Q : Events (Outbox)
P ..> Q
Q ..> N
Q ..> E
N --> M
@enduml
```

## Aufgabe 2: die Entscheidungen dahinter

- Service-basiert mit drei Einheiten, nicht fünfzehn: Der Modulith bleibt für Kurse, Anmeldung, Warteliste, Bewertungen
- Platzvergabe bleibt eine Transaktion; Prüfungen als eigener Service mit eigener Datenbank: eigener Stakeholder, Audit, Historie, Release-Takt
- Benachrichtigung und Exporte als Event-Konsumenten: entkoppelt, am Stichtag unkritisch
- Release-Problem zuerst im Modulith lösen: Modulgrenzen prüfen, Testpyramide statt 50 Minuten Volllauf
- Eigenes Rechenzentrum: Deployment-Frage (Container, dieselben Artefakte), kein neuer Stil
- Bewusst nicht: ein Service je Modul, Kafka für 60 Anmeldungen pro Sekunde, Serverless für den Kern

# Zusammenfassung

## Zusammenfassung

- Ein Stil ist die Antwort auf vier Fragen: Schnitt, Deployment, Daten, Kommunikation
- Stile unterscheiden sich in Kopplung und Preis, nicht in modern oder alt
- Modular Monolith als Standardstart; Verteilung kostet Netzwerk, Konsistenz und Betrieb und braucht einen Grund
- Unabhängigkeit kann für Komplexität sorgen, Entkopplung kann für Unübersichtlichkeit sorgen
- Der Stil folgt Kontext und Organisation; Grenzen gelten nur, wenn sie durchgesetzt werden
- Erste Aufgabe im neuen Projekt: den Stil erkennen und seine Spielregeln einhalten

## Nächste Vorlesung

- Betrieb, Observability & Deployment (Lukas)

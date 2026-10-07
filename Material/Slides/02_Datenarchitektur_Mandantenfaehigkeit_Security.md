---
title: "Skalierbare Systeme - Datenarchitektur, Mandantenfähigkeit & System-Security"
topic: "scalable_systems_1_2"
author: "Lukas Panni"
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

# Organisatorisches

## Heute

- **Datenarchitektur**: Datenmodell, Transaktionen, Replikation und Partitionierung
- **Mandantenfähigkeit**: Isolation ist eine Architekturentscheidung
- **System-Security**: Grenzen, Identitäten und Schutzschichten
- **Durchgängiges Beispiel**: Cal.diy, erweitert zum Mandanten-Teaching-Model
- **AI Engineering**: Daten und Sicherheit für KI-Systeme

## Vorstellung

### Dozent: Lukas Panni

- Per Du
- E-Mail-Adresse: <lukas.panni@outlook.de>
- Seit 2018 bei SEW-EURODRIVE in Bruchsal
  - 2023: _M.Sc._ Informatik - HKA
  - 2021: _B.Sc._ Informatik - DHBW Karlsruhe

## Lernziele

- Datenmodell und Zugriffswege gemeinsam entwerfen
- Replikation, Partitionierung und Konsistenz passend einordnen
- drei grundlegende Mandantenmodelle vergleichen
- Tenant-Isolation über den ganzen Request erzwingen
- Bedrohungen an Vertrauensgrenzen finden und Schutzmaßnahmen ableiten

## Unser reales Beispiel: Cal.com/Cal.diy

- Community-getriebene, Open-Source Next.js basierte Terminplanungsplattform: [github.com/calcom/cal.diy](https://github.com/calcom/cal.diy)
- Selbst gehostete Community-Edition von [Cal.com](https://cal.com)
- Terminarten, Verfügbarkeiten und Buchungen
- Integrationen mit externen Kalendern und Videodiensten

# Datenarchitektur

## Datenarchitektur - Grundlagen (1)

Wikipedia:

> Datenarchitektur ist eine Teildisziplin der IT-Architektur, die sich mit grundlegenden Strukturen und Prinzipien zu Daten und Informationen, ihrer Konstruktion, Nutzung und Weiterentwicklung befasst.

## Datenarchitektur - Grundlagen (2)

- Welche Daten gibt es, wie sind die Beziehungen zwischen ihnen?
- Wem gehören die Daten, wer steuert den Zugriff?
- Wer liest und schreibt wann, wie oft und mit welchen weiteren Anforderungen?
- Wo werden Daten gespeichert, wie lange und mit welchen Anforderungen an Verfügbarkeit und Konsistenz?

## Vom Anwendungsfall zum Datenmodell - Use-Case Terminbuchung (1)

1. Gast öffnet eine Buchungsseite
2. System zeigt freie Zeiten
3. Gast wählt einen Zeitslot
4. System legt eine Buchung an
5. Externer Kalender wird aktualisiert (z.B. via Webhook)
6. Bestätigung wird versendet

## Vom Anwendungsfall zum Datenmodell - Use-Case Terminbuchung (2)

- Welche Informationen werden benötigt?
- Wer greift zu? lesend? schreibend?
- Zugriffshäufigkeiten?
- Anforderungen an Konsistenz? Transaktionen?

## Datenmodell: drei Ebenen

1. **Konzeptuell:** Modellierung der Realität. Daten und ihre Beziehungen.
2. **Logisch:** Wie bildet das gewählte DBMS diese Dinge ab?
3. **Physisch:** Wie werden sie technisch gespeichert und schnell erreichbar gemacht?

Entspricht strukturiertem Vorgehen: erst Daten erfassen + modellieren, dann Entscheidung für Datenbanksystem treffen, dann technische Details festlegen.

## Konzeptuelles Datenmodell: Cal.diy

```plantuml
@startuml
hide circle
skinparam linetype ortho
entity User
entity EventType
entity Availability
entity Room
entity Booking

User ||--o{ EventType
User ||--o{ Availability
EventType ||--o{ Booking
Room ||--o{ Booking
@enduml
```

## Logisches Datenmodell: relational

- Relational: Daten liegen in **Relationen** (Tabellen)
- Jede Relation ist eine Menge von **Tupeln** (Datensätzen) desselben Typs
- Beziehungen werden etwa über Fremdschlüssel abgebildet; Constraints machen Regeln prüfbar
- Für Buchungen mit Beziehungen und Transaktionen ist das ein guter Start

## Logische Datenmodelle: weitere Optionen "NoSQL"

- **Dokumentenorientiert**: zusammengehörige Daten als Dokument; flexibel, gut für Aggregate
- **Key-Value Store:** Schlüssel führt direkt zu einem Wert; gut für Cache oder Sessions
- **Graphdatenbank:** Knoten und Kanten, gut für viele Beziehungsabfragen
- **Objektorientierte Datenbank:** persistiert Objekte direkt; heute eher Nische

## Logisches Datenmodell: Auswahl

- Kein perfektes System für alle Anwendungsfälle
- Alle Arten haben Vor- und Nachteile
- Entscheidung sollte von Zugriffsmustern und Domänenregeln abhängen
  - Anforderungen an Konsistenz
  - Zugriffshäufigkeiten
  - Art der Beziehungen
- Für viele Anwendungen ist eine relationale Datenbank ein guter Startpunkt, erweitern bei Bedarf

## Zugriffsmuster im Beispiel

Typische Abfragen:

- Freie Slots einer Terminart anzeigen
- Eine eindeutige Buchung laden
- Buchungen auflisten
- Kollision mit bestehender Buchung prüfen
- Externe Services abfragen oder aktualisieren

## Physisches Datenmodell

- Ergänzt das logische Modell um technische Details der Speicherung und Zugriffsoptimierung
- Beispiele: Indexstrukturen, Partitionierung, Replikation, Speicherformat und Query-Pläne
  - z.B. für schnelle Abfragen auf freie Slots, Constraints für Konsistenz und Einhaltung logischer Regeln
- Datenbanknutzer sehen diese Details meist nicht; die Latenz und Kosten aber schon

## Polyglot Persistence

- Ein System nutzt bewusst mehrere Speichermodelle für verschiedene Aufgaben
  - z.B. wenn ein relationales Modell nicht alle Anforderungen erfüllt
  - Beispiel: PostgreSQL für Buchungen, Key-Value für Sessions, Suchindex für Volltext
- Jedes zusätzliche System kostet Betrieb, Konsistenzarbeit und Wissen
- Deshalb: erst ein passendes System, weitere nur bei messbarem Bedarf

## Wiederholung Transaktionen

> Folge von Operationen, die als logische Einheit betrachtet werden und entweder vollständig ausgeführt oder vollständig zurückgesetzt werden.

## Transaktionen bei Buchungen

1. Slot prüfen
2. Buchung anlegen
3. Status und Teilnehmer speichern

Nicht Teil derselben Datenbanktransaktion:

- externer Kalender
- E-Mail-Versand
- Webhooks

## Wiederholung Transaktionen - ACID

- **Atomicity:** ganz oder gar nicht
- **Consistency:** Datenregeln bleiben erfüllt
- **Isolation:** parallele Transaktionen sehen keinen unkontrollierten Zwischenstand
- **Durability:** bestätigte Daten bleiben gespeichert

## Aufgabe: Doppelbuchung

**Arbeitszeit: 6 Minuten — 2 allein, 2 zu zweit, 2 im Plenum**

Zwei Requests prüfen "gleichzeitig" denselben freien Slot und wollen beide buchen.

1. Wo genau ist die Race Condition?
2. Welche Regel muss unabhängig vom Anwendungscode gelten?
3. Was darf bei der zweiten Anfrage passieren?

## Doppelbuchung Ergebnis (1)

![Doppelbuchung Fehler](media/doppelbuchung-fehler.png){width=65%}

## Lösung mit Transaktionen? (1)

Warum lösen Transaktionen das Problem der Doppelbuchung nicht garantiert?

## Lösung mit Transaktionen? (2)

- Beide Requests müssen entweder komplett ausgeführt oder komplett zurückgesetzt werden
- Annahme: Isolation Level Read Committed o. ä.
  - Solange bei Start beider Transaktionen der Slot noch frei ist können beide ohne Verletzung von Konsistenzregeln schreiben!
- Lösungsmöglichkeit: Höheres Isolationslevel, _aber_ Performance kann leiden

## Wiederholung: Isolation Levels (1)

Isolation bestimmt, welche Auswirkungen **parallel laufender Transaktionen** sichtbar werden.

| Level                      | Zusage                                                                                                              |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| Read Committed             | Transaktion kann nur bestätigte (commitete) Änderungen sehen; zwischen zwei Operationen kann sich der Stand ändern. |
| Repeatable Read / Snapshot | Transaktion sieht nur Änderungen, die vor Start der Transaktion commitet wurden.                                    |
| Serializable               | Transaktionen wirken als wären sie nacheinander ausgeführt worden, keine phantom reads möglich                      |


## Wiederholung: Isolation Levels (2)

- Höhere Isolation bedeutet mehr Koordination, mögliche Abbrüche und Retries
- Details und konkrete Namen unterscheiden sich je nach DBMS

## Alternative Lösung: Unique Constraint (1)

- z.B. Unique Index auf `room_id` und `start_time` für bestätigte Buchungen
  - Reicht noch nicht aus für flexible Zeitintervalle, die überlappen können

```sql
-- Für feste Slots, etwa immer 30 Minuten:
CREATE UNIQUE INDEX booking_one_per_fixed_slot
ON booking (room_id, start_time)
WHERE status = 'CONFIRMED';
```

- Beispiel für Teil des physischen Datenmodells, reines Implementierungsdetail

## Alternative Lösung: Unique Constraint (2)

![Doppelbuchung Unique Index](media/doppelbuchung-unique-index.png){width=68%}

## Warum Konsistenzregeln in der Datenbank?

- Invarianten im Datenmodell (z.B. nur eine bestätigte Buchung pro Raum und Zeit) sollte nah bei den Daten abgesichert werden
  - Regeln die unabhängig vom Use-Case gelten sollen
- Bei Umsetzung in Datenbank:
  - Regeln gelten unabhängig über welchen Weg die Daten geändert werden (parallele Requests, Batch-Jobs, Admin-Tools, neue Code-Pfade ...)
  - Einfach wartbar
  - Schützt vor Inkonsistenzen bei parallelen Anfragen

## Warum Daten verteilen?

Eine einzelne Datenbank kann an Grenzen stoßen:

- Viele gleichzeitige Zugriffe erhöhen Wartezeiten
- Große Datenmengen übersteigen Speicher- oder Rechenkapazität
- Weit entfernte Nutzer warten länger auf Antworten
- Single Point of Failure

## Replikation (1)

> **Replikation**: Dieselben Daten verteilt auf mehrere Server.

- Löst Probleme mit Single Point of Failure und Latenzen
- Keine Lösung für große Datenmengen -> gleiche Daten müssen auf allen Servern vorgehalten werden
- Neues Problem: Konsistenz bei parallelen Schreibzugriffen auf verschiedene Server

## Replikation (2)

- **Starke Konsistenz:** Nach bestätigtem Schreiben liefert eine Leseanfrage den neuen Stand oder schlägt fehl
- **Eventual Consistency:** Eine Leseanfrage kann zunächst noch den alten Stand liefern, später gleichen sich die Kopien an
- **CAP** = Consistency, Availability, Partition Tolerance (ein verteiltes System kann immer nur zwei der drei Eigenschaften gleichzeitig erfüllen)
  - Bei Netzwerkunterbrechung zwischen Konsistenz und Verfügbarkeit wählen

## CAP

![CAP](media/cap.png){width=70%}

> [CAP Theorem](https://medium.com/@anupchakole/understanding-the-cap-theorem-why-your-system-cant-have-it-all-4004c25e021f)

## BASE

**B**asically **A**vailable, **S**oft State, **E**ventual Consistency

- Alternatives Konsistenzmodell zu ACID in verteilten Systemen
- Hohe Verfügbarkeit, gute Skalierbarkeit
- Tradeoff: geringere Konsistenzgarantien
- In NoSQL-Datenbanken in verteilten Systemen verbreitet

## Replikation Modelle (1)

**Leader-Follower Modell**

- Ein Server ist der Leader, alle anderen sind Follower
- Schreibzugriffe gehen an den Leader, Lesezugriffe sind bei allen möglich
- Aktualisierungen werden vom Leader an die Follower weitergegeben
  - synchron oder asynchron möglich
  - bei asynchroner Replikation nur eventual consistency möglich
- **Vorteil:** Einfaches Konsistenzmodell, leicht zu verstehen und zu implementieren
- **Nachteil:** Single Point of Failure beim Leader, mögliche Performance-Engpässe beim Leader

## Replikation Modelle (2)

![Replikation Lag](media/Replication_Lag.pdf){height=120%}

## Replikation Modelle (3)

- Anpassung: Multi-Leader
  - Mehrere Leader erlauben parallele Schreibzugriffe
  - Konflikte müssen aufgelöst werden
  - Oder Kombination mit Sharding
- Komplexer: Leaderless mit Quoren
  - Jeder Read/Write Zugriff auf mehrere Replikate
  - Konsistenz wird durch Quoren sichergestellt (Mehrheit der Replikate muss zustimmen)

## Partitionierung / Sharding

> **Sharding / Partitionierung**: Die Daten werden auf mehrere Server verteilt, sodass jeder Server nur einen Teil der Daten hält.

- Vorteile
  - Reduziert die Datenmenge pro Server
  - Parallele Schreib- und Lesezugriffe ohne zusätzliche Koordination
  - bei Geografischer Verteilung oft gut: z.B. deutsche User nutzen häufig deutsche Daten, andere weniger häufig

## Sharding Umsetzung (1)

- Shard Key: Alle Daten eines bestimmten Schlüssels liegen auf demselben Shard
  - z.B. user_id -> Daten werden nach User-ID aufgeteilt: gleiche user_id = gleicher Shard
- Zentraler Koordinator / Router mit Kenntnis der Shard-Zuordnung notwendig
  - kann selbst wiederum repliziert sein
- Technische Umsetzung
  - **Range Sharding**: Aufteilung nach Wertebereich
    - Beispiel: user_id 1-1000 auf Shard 1, 1001-2000 auf Shard 2
    - Hotspot Risiko größer
  - **Hash Sharding**: Aufteilung nach Hashwert des Shard Keys
    - Ziel: gleichmäßige Verteilung der Daten auf die Shards
    - Schlecht wenn häufig ganze Wertebereiche abgefragt werden

## Sharding Umsetzung (2)

![Sharding Umsetzung](media/Sharding_Shard_Key.pdf){width=120%}

## Sharding Umsetzung (3)

![Sharding Umsetzung](media/Sharding_Router.pdf){width=120%}

## Sharding Herausforderungen

- Aufteilung der Daten ist Anwendungsfallabhängig
  - **Ziel**: gleichmäßige Verteilung der Last
  - Last kann sich über Zeit verschieben, Rebalancing notwendig
- Größerer Aufwand bei Cross-Shard-Operationen: z.B. Joins über mehrere Shards, globale Abfragen, Transaktionen
- Abhilfe durch Denormalisierung möglich, aber Aufwand um Konsistenz sicherzustellen

## Verteilte Transaktionen

- Transaktionen zwischen verschiedenen Shards oder verschiedenen Systemen erfordern spezielle Koordination
  - Datenbanksystem selbst kann ACID nicht mehr ohne weiteres garantieren

- **Two-Phase Commit (2PC)**
  - _Phase 1_: Prepare
    - Alle Systeme prüfen, ob sie die Transaktion ausführen können
  - _Phase 2_: Commit
    - Wenn alle Systeme bereit sind, wird die Transaktion ausgeführt
    - Andernfalls wird sie abgebrochen
    - Rollback, wenn ein System die erfolgreiche Ausführung der Transaktion nicht bestätigen kann

## Verteilte Transaktionen (2)

- 2PC Nachteile
  - Koordinator notwendig
  - hohe Latenz, geringer Durchsatz, viel Kommunikationsoverhead
  - Problematisch bei System/Kommunikations-ausfällen
- Bei Aktualisierung von externen Systemen: Outbox Pattern
  - Aktualisierung von internem Anwendungszustand per Transaktion
  - In gleicher Transaktion wird ein Aktualisierungs-Event in der DB gespeichert
  - Events werden nachgelagert an externe Systeme gesendet
  - Idempotenz bei Event-Empfänger notwendig!

## Outbox Pattern Beispiel

![Outbox Pattern Beispiel](media/Outbox_Pattern.png)

## Daten aufteilen? Data Disintegrators

- **Änderungen:** Auswirkungen von Änderungen an Daten auf Services
  - Trennung erleichtert Änderungen an getrennten Services
  - Trennung verringert Koordinationsaufwand über verschiedene Entwicklungs-Teams
  - Klare Datenhoheit wichtig (auch in monolithischen DBs): mehrere Services sollten nicht gleichzeitig die gleichen Daten ändern
- **Betrieb + Skalierung:**
  - Trennung ermöglicht unabhängige Skalierung und Wartung
  - Trennung erleichtert Verbindungsmanagement und Auslastung von Connection Pools
  - Trennung erleichtert Fehlertoleranz für _Teile_ der Anwendung

## Daten zusammenhalten? Data Integrators

- **Beziehungen:**
  - Fremdschlüssel und Constraints sichern Integrität, koppeln aber zusammengehörige Daten
  - Konsistenz wird durch geteilte Daten erschwert
- **Transaktionen:**
  - Gemeinsame ACID-Transaktionen sind über getrennte Datenbanken nicht ohne Weiteres möglich
  - Verteilte Transaktionen bringen Koordination, Latenz und Ausfallrisiken mit sich

# Mandantenfähigkeit

## Ausgangssituation

![Ausgangssituation](media/Ausganssituation_non_SaaS.pdf)

## Was ist ein Mandant?

- engl. "tenant" -> "Multi-Tenancy"
- organisatorisch abgegrenzte Kundengruppe in einem gemeinsamen Produkt
- eigene Daten, Einstellungen, Benutzer und Berechtigungen

## Multi-Tenancy - Definition + Motivation

> Ein System bedient mehrere Mandanten gleichzeitig, wobei die Daten und Aktionen der Mandanten logisch getrennt bleiben.

- Ökonomischere Ressourcennutzung, spart Betriebskosten
- Zentrale Weiterentwicklung
- Skalierbarer Betrieb

## Zielbild

![Zielbild](media/Zielbild_SaaS.pdf){height=80%}

## Von Single-Tenancy zu Multi-Tenancy (1)

- Viele Anwendungen starten nicht mit Multi-Tenancy als Anforderung
  - Siehe Cal.diy Beispiel
  - Was fehlt uns für Multi-Tenancy? (Ziel: Untersützung für mehrere Organisationen)

## Von Single-Tenancy zu Multi-Tenancy (2)

- Anpassungen im Datenmodell: Alle Daten müssen einem Mandanten (einer Organisation) zugeordnet werden
- Anpassungen in der Anwendungslogik:
  - Alle Use-Cases müssen den aktuellen Mandantenkontext berücksichtigen
  - Neue Use-Cases für Mandantenübergreifende Operationen: z.B. Reporting, Administration

## Drei Modelle für Datenisolation

| Modell | Datenablage                      | Isolation |          Betrieb |
| ------ | -------------------------------- | --------: | ---------------: |
| Pool   | gemeinsame Tabellen, `tenant_id` |   logisch |          einfach |
| Bridge | eigenes Schema pro Mandant       |   stärker | mehr Migrationen |
| Silo   | eigene Datenbank oder Instanz    |     stark |        aufwendig |

Kein Modell gewinnt immer: Compliance, Größe, Kosten und Betrieb entscheiden.

## Silo: höchste Isolation, aufwendig im Betrieb

- Einfachste Lösung: jeder Mandant bekommt eine eigene Instanz (Datenbank oder Anwendung)
- Stärkste mögliche Isolation
- Hoher Verwaltungsaufwand für Betrieb und Updates
- Kaum gemeinsame Infrastrukturnutzung, keine Skaleneffekte
- Schlecht für Mandantenübergreifende Operationen: Administration, Observability

## Silo

![Silo](media/Silo_Dbs.pdf){height=140%}

## Pool: einfach umzusetzen, aber mit Risiken

```text
booking(id, organization_id, host_id, start_time, status)
```

- einfach umzusetzen
- jede relevante Abfrage ist mandantengebunden
- ein vergessener Filter wird zum Datenleck, Fehler haben große Auswirkungen
- gemeinsame Infrastruktur: kostengünstig, Last aber auch geteilt -> _Noisy Neighbours_

## Pool

![Pool](media/Pool.pdf){width=150%}

## Bridge: mittlere Isolation, mittlerer Aufwand

- eigenes Schema pro Mandant, aber gleiche Datenbank(instanz)
- relativ starke technische Trennung auf Datenbankebene
- Migrationen zwischen Mandanten sind aufwendiger als im Pool-Modell

Auch hybride Ansätze sind verbreitet. Häufig auch Pool für Standard-Mandanten und eigene Instanzen für regulierte oder besonders große Kunden.

## Bridge

![Bridge](media/Bridge.pdf){width=75%}

## Hybrid

![Hybrid](media/Pool_+_Silo_hybrid.pdf)

## Tenant Context

> Wie lässt sich in der Anwendung sicherstellen, dass jeder Zugriff auf die Daten eines Mandanten korrekt beschränkt ist?

- Tenant Context = Identität des aktuellen Mandanten + notwendige Daten für die Anwendung (z.B. Rollen)
- Context wird im gesamten Request Lifecycle beibehalten und weitergereicht
  - Bei verteilten Systemen z.B. über JWTs
- Datenzugriffe beziehen sich auf den Tenant Context, Datenzugriffslayer erzwingt Tenant-Filter
  - Umsetzung z.B. über Repository-Pattern oder ORM-Level-Filter

## Datenzugriff: Defense in Depth

- Datenbank-Features können den Mandantenfilter erzwingen, z.B. über Row-Level Security (RLS) in PostgreSQL
  - Jeder Datensatz einer Relation (=Row) gehört zu genau einem Tenant
  - RLS-Policy stellt Zugriff nur auf die Daten des jeweiligen Tenants sicher
- Tenant Context trotzdem wichtig: Datenbankverbindung muss auf den aktuellen Tenant konfiguriert werden
- Defense in Depth: kein Ersatz für Anwendungsautorisierung

## Noisy Neighbour

> Ein Mandant verbraucht übermäßig viele gemeinsame Ressourcen und schränkt andere dadurch ein.

- z.B. großer Export, sehr viele gleichzeitige Anfragen

Gegenmaßnahmen:

- Rate Limits und Quotas pro Mandant
- Queues und Pools begrenzen den Blast Radius
- Anspruchsvolle Kunden in ein Silo verschieben

## Aufgabe: Auswahl passendes Mandantenmodell

**Arbeitszeit: 5 Minuten, 3er Gruppen**

Fälle:

1. 50 kleine Organisationen
2. 10.000 kleine Organisationen mit stark unterschiedlicher Last
3. 20 regulierte Großkunden mit Restore-Anforderung pro Kunde

Bewertet _Isolation_, _Kosten_, _Betrieb_, _Skalierung_ für die verschiedenen Modelle für einen der Fälle 1-3.

# System-Security

## Begriffe

- Sicherheitsziele: **C**onfidentiality, **I**ntegrity, **A**vailability
- **Bedrohung:** mögliche Verletzung eines Sicherheitsziels
- **Angriff**: konkreter Versuch ein Sicherheitsziel zu verletzen
- **Risiko:** Wahrscheinlichkeit × Auswirkung
- Mechanismen: **Prevention**, **Detection**, **Recovery**

## Security ist ein Qualitätsattribut

- **Vertraulichkeit:** nur Berechtigte sehen Daten
- **Integrität:** Daten werden nicht unbemerkt verfälscht
- **Verfügbarkeit:** Berechtigte können den Dienst nutzen

**Sicherheit entsteht durch Entscheidungen im Systemdesign**

## Authentifizierung ist nicht Autorisierung

- **Authentifizierung**: Verknüpfung von Identität und Individuum (Subjekt)
  - Identität bestätigen durch: Wissen (Passwörter), Besitz (Token), Individuelle Eigenschaften (Biometrie)

- **Autorisierung**: Festlegung was ein Subjekt darf.


## Least Privilege – Prinzip der geringsten Rechte

**Prinzip:** Jede Identität erhält nur die Rechte, die sie für ihre Aufgabe (zwingend) benötigt.

Gilt für:

- Benutzer und Administratoren
- Services und Worker
- Datenbankbenutzer
- Cloud-Identitäten und externe API-Tokens

Beispiele:

- Anwendung läuft nicht als PostgreSQL-Superuser
- Webhook-Worker darf keine Benutzerrollen ändern
- OAuth-Token fordert nur notwendige Scopes an
- Supportzugriff ist zeitlich begrenzt

## Threat Modeling – Ziel

**Threat Modeling** untersucht ein System strukturiert aus Sicht möglicher Angriffe und Fehlbedienungen.

- schützenswerte Daten und Funktionen erkennen
- Angriffsflächen und Trust Boundaries sichtbar machen
- konkrete Bedrohungen identifizieren
- Risiken priorisieren
- passende Schutzmaßnahmen bereits im Design ableiten

Wichtig:

- nicht jede theoretische Bedrohung finden, sondern relevante Risiken systematisch behandeln

## Threat Modeling – grundlegendes Vorgehen

1. **Was bauen wir?**
   - Komponenten, Datenflüsse und externe Systeme modellieren
2. **Was kann schiefgehen?**
   - Assets, Einstiegspunkte, Trust Boundaries und Bedrohungen untersuchen
3. **Was tun wir dagegen?**
   - Kontrollen und Architekturmaßnahmen definieren
4. **Reicht das aus?**
   - Risiken priorisieren, Maßnahmen prüfen, Restrisiken dokumentieren

## Bedrohungskategorien: STRIDE

- **Spoofing**: Vortäuschen einer falschen Identität
- **Tampering**: Unbefugtes Verändern von Daten
- **Repudiation**: Abstreiten von Aktionen
- **Information Disclosure**: Unbefugtes Offenlegen von Informationen
- **Denial of Service**: Verhinderung der Nutzung eines Dienstes
- **Elevation of Privilege**: Erlangen höherer Rechte als erlaubt

## Threat Modeling – Systemmodell erstellen

Beispiel Terminbuchung:

- Browser des Benutzers
- Anwendung / API
- PostgreSQL-Datenbank
- externer Kalenderdienst
- eingehende Webhooks

Wichtige Assets:

- Buchungen und Kontaktdaten
- Organisations- und Berechtigungsdaten
- Sessions und OAuth-Tokens

## Trust Boundary

> Eine Trust Boundary ist die Grenze zwischen Bereichen unterschiedlichen Vertrauens innerhalb eines Systems.

- Daten/Anfragen aus andererm Vertrauensbereich müssen besonders geprüft werden
- Beispiele für Trust Boundaries:
  - Netzwerkgrenze (Internet ↔ interne Dienste)
  - Anwendungsschicht (Frontend ↔ Backend)
  - Datenbankzugriff (Service ↔ Datenbank)

## Trust Boundaries sichtbar machen

```plantuml
@startuml
left to right direction
actor Gast
rectangle "Internet" {
  [Browser]
  [Kalender-API]
}
rectangle "Terminplattform" {
  [API]
  [Buchungsservice]
  database "PostgreSQL" as DB
}

Gast --> [Browser]
[Browser] --> [API] : Request + Eingaben
[API] --> [Buchungsservice]
[Buchungsservice] --> DB
[Buchungsservice] --> [Kalender-API] : Token + API-Aufruf
@enduml
```

- An jeder Grenze: Identität, Eingaben, Rechte und Datenfluss prüfen

## Aufgabe: Threat Model

**Arbeitszeit: 8 Minuten — 2 allein, 3 zu zweit, 3 im Plenum**

Nehmt eine Grenze aus dem Diagramm und beantwortet:

1. Was kann hier schiefgehen?
2. Welche Daten oder Aktion sind betroffen?
3. Welche Maßnahmen helfen?

Checkliste bei Bedarf: STRIDE

## Threat Modeling – Beispielbedrohungen

| Bedrohung                              | Mögliche Ursache              | Mögliche Kontrolle                   |
| -------------------------------------- | ----------------------------- | ------------------------------------ |
| Benutzer liest fremde Buchung          | fehlende Tenant-Prüfung       | objektbezogene Autorisierung         |
| Angreifer übernimmt Session            | gestohlenes Session-Token     | sichere Session, kurze Laufzeit, TLS |
| OAuth-Token des Kalenders wird geleakt | Secret im Log oder Repository | Secret Store, eingeschränkte Logs    |
| Gefälschter Webhook verändert Daten    | Absender nicht geprüft        | Signatur prüfen, Replay-Schutz       |
| Ein Client überlastet die API          | unbegrenzte Requests          | Rate Limits, Queues                  |

## Risiken priorisieren

Bewertung nach:

- **Wahrscheinlichkeit:** Wie realistisch ist eine Ausnutzung?
- **Auswirkung:** Wie groß wäre der Schaden?

Vereinfachte Einordnung:

- hohe Wahrscheinlichkeit + hohe Auswirkung → zuerst behandeln
- geringe Risiken können bewusst akzeptiert werden
- Ergebnis Threat Modeling: priorisierte Risiken, geplante Kontrollen und _dokumentierte Restrisiken_

## Daten schützen – in transit

**In transit:** Verschlüsselung von Daten bei Übertragung zwischen Systemen

- TLS für alle Verbindungen nutzen
  - Schutz vor Mitlesen und Manipulation
- Zertifikate korrekt prüfen!
- sensible Daten nicht unverschlüsselt übertragen, auch nicht über interne Netzwerke

## Daten schützen – at rest

**At rest:** Verschlüsselung von gespeicherten Daten

- Verschlüsselung von Datenträgern oder Datenbank-Storage
- Backups mit demselben Schutzbedarf behandeln
- besonders sensible Felder bei Bedarf zusätzlich auf Anwendungsebene verschlüsseln
- Schlüssel getrennt von den verschlüsselten Daten verwalten!

## Audit Logging

> Nachvollziehbare Aufzeichnung sicherheitsrelevanter Aktionen

- wer (Identität), wann (Timestamp), wo (Kontext), was (Aktion/Zielobjekt) + Ergebnis (erfolgreich/abgelehnt)
- Nicht zu verwechseln mit Anwendungslog für technische Diagnose + Betrieb
- z.B. Rollenänderungen, API-Key Erstellung/Entzug, export sensibler Daten
- Logs müssen vor Manipulation und Löschung geschützt werden

## Defense in Depth

- Mehrere Schutzschichten verhindern, dass einzelne Fehler direkt problematisch werden

1. TLS schützt den Transportweg
2. Authentifizierung bestimmt die Identität
3. Autorisierung prüft Aktion und Zielobjekt
4. Least Privilege begrenzt mögliche Auswirkungen
5. Tenant Context und Datenbankregeln schützen Datenzugriffe
6. Audit Logs machen kritische Änderungen nachvollziehbar

## Network Security

- zentrales API Gateway für Authentifizierung, Rate Limits und Policies
  - Nur ein einzelner Zugangspunkt für externe Anfragen muss geschützt werden
- Segmentierung: alle anderen Systeme nicht von außen erreichbar machen, z.B. Datenbank nur intern
- Firewalls und Sicherheitsgruppen nutzen, um den Zugriff auf interne Systeme zu kontrollieren
- mTLS (mutual TLS) für die Authentifizierung zwischen internen Diensten nutzen
  - nicht jeder Service ist vertrauenswürdig, siehe Trust Boundaries

## Supply Chain Security

- Zunehmendes Risiko von Supply Chain Angriffen
  - Angriffe auf Teile der Software-Lieferkette, z.B. Libraries, Container-Images, CI/CD Pipelines
- Packages nur aus vertrauenswürdigen Quellen verwenden
- Herkunft und Integrität prüfen: Signaturen, Provenance
- Nicht sofort die allerneuste Version einsetzen, Updates bewusst steuern

# Nächste Vorlesung

- Technische Entscheidungsfindung: Trade-offs, Stakeholder & Kontext (Silas)

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

<!--
Time plan: 4 VE = 180 min content, Friday 8:30-11:45 with a 15 min break on top.
  Organisatorisches (Heute, Vorstellung)   8
  Lernziele                                2
  Datenarchitektur                        50   8 slides
  Mandantenfähigkeit                      40   worked example ~15
  System-Security                         40   6 slides
  AI Engineering                          15
  Zusammenfassung                          5
  Sum                                    160   20 min slack
-->

# Organisatorisches

## Heute

- Datenarchitektur: Datenmodell, Transaktionen, Replikation und Partitionierung
- Mandantenfähigkeit: Isolation ist eine Architekturentscheidung
- System-Security: Grenzen, Identitäten und Schutzschichten
- Durchgängiges Beispiel: Cal.diy, erweitert zum Mandanten-Teaching-Model
- AI Engineering: Daten und Sicherheit für KI-Systeme

## Vorstellung

### Dozent: Lukas Panni

<!-- TODO(Lu): bio and e-mail address -->

- Per Du
- E-Mail-Adresse: TODO

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
entity Booking

User ||--o{ EventType
User ||--o{ Availability
EventType ||--o{ Booking
User ||--o{ Booking : host
@enduml
```

## Logisches Datenmodell: relational

- Relational: Daten liegen in **Relationen** (Tabellen)
- Jede Relation ist eine Menge von **Tupeln** (Datensätzen) desselben Typs
- Beziehungen werden etwa über Fremdschlüssel abgebildet; Constraints machen Regeln prüfbar
- Für Buchungen mit Beziehungen und Transaktionen ist das ein guter Start

## Logische Datenmodelle: weitere Optionen "NoSQL"

- Dokumentenorientiert: zusammengehörige Daten als Dokument; flexibel, gut für Aggregate
- Key-Value Store: Schlüssel führt direkt zu einem Wert; gut für Cache oder Sessions
- Graphdatenbank: Knoten und Kanten, gut für viele Beziehungsabfragen
- Objektorientierte Datenbank: persistiert Objekte direkt; heute eher Nische

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

## Wiederholung Transaktionen (1)

> Folge von Operationen, die als logische Einheit betrachtet werden und entweder vollständig ausgeführt oder vollständig zurückgesetzt werden.

## Transaktionen bei Buchungen

1. Slot prüfen
2. Buchung anlegen
3. Status und Teilnehmer speichern

Nicht Teil derselben Datenbanktransaktion:

- externer Kalender
- E-Mail-Versand
- Webhooks

## Wiederholung Transaktionen (2) ACID

- **Atomicity:** ganz oder gar nicht
- **Consistency:** Datenregeln bleiben erfüllt
- **Isolation:** parallele Transaktionen sehen keinen unkontrollierten Zwischenstand
- **Durability:** bestätigte Daten bleiben gespeichert

## Think-Pair-Share: Doppelbuchung

**Arbeitszeit: 6 Minuten — 2 allein, 2 zu zweit, 2 im Plenum**

Zwei Requests prüfen "gleichzeitig" denselben freien Slot und wollen beide buchen.

1. Wo genau ist die Race Condition?
2. Welche Regel muss unabhängig vom Anwendungscode gelten?
3. Was darf bei der zweiten Anfrage passieren?

## Doppelbuchung Ergebnis (1)

![Doppelbuchung Fehler](media/doppelbuchung-fehler.png)

## Lösung mit Transaktionen? (1)

Warum lösen Transaktionen das Problem der Doppelbuchung nicht garantiert?

## Lösung mit Transaktionen? (2)

- Beide Requests müssen entweder komplett ausgeführt oder komplett zurückgesetzt werden
- Annahme: Isolation Level Read Committed o. ä.
  - Solange bei Start beider Transaktionen der Slot noch frei ist können beide ohne Verletzung von Konsistenzregeln schreiben!
- Lösungsmöglichkeit: Höheres Isolationslevel, _aber_ Performance kann leiden

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

-![Doppelbuchung Unique Index](media/doppelbuchung-unique-index.png)

## Warum Konsistenzregeln in der Datenbank?

- Invarianten im Datenmodell (z.B. nur eine bestätigte Buchung pro Raum und Zeit) sollte nah bei den Daten abgesichert werden
  - Regeln die unabhängig vom Use-Case gelten sollen
- Bei Umsetzung in Datenbank:
  - Regeln gelten unabhängig über welchen Weg die Daten geändert werden (parallele Requests, Batch-Jobs, Admin-Tools, neue Code-Pfade ...)
  - Einfach wartbar
  - Schützt vor Inkonsistenzen bei parallelen Anfragen

## Warum Daten verteilen?

Bisher: Eine Datenbank speichert alle Buchungen und erzwingt ihre Regeln.

- Leselast und Ausfälle: Replikation
- Datenmenge und Schreiblast: Partitionierung
- Externe Systeme: verteilte Transaktionen
- Jede Maßnahme schafft neue Konsistenzfragen

## Replikation: Kopien mit Verzögerung

- Single Leader: einer schreibt, Replikate lesen
- Ziele: Leselast, Ausfall, Nähe
- Asynchron: schnell, aber mit Lag
- Buchung gespeichert, Replica zeigt Slot noch frei
- Multi-Leader/Leaderless: Konflikte oder Quoren

## Partitionierung: Daten gezielt teilen

- Replikation kopiert, Partitionierung verteilt
- Range: Bereiche gut, Hot-Spot-Risiko
- Hash: gleichmäßig, Bereiche aufwendig
- Routing und Rebalancing gehören dazu
- Später: `organization_id` als Partition-Key

## Konsistenz nach der Verteilung

- **Linearizable:** eine aktuelle Reihenfolge
- **Eventual:** Kopien gleichen sich später an
- CAP: bei Partition Verfügbarkeit oder lineare Konsistenz
- BASE: verfügbarkeitsorientierte Denkweise, kein ACID-Gegenspieler
- Buchung stark; Kalenderansicht und Report dürfen nachziehen

## Verteilte Transaktionen: Buchung plus Kalender

- Datenbank-Commit und Kalender-API sind nicht atomar
- Two-Phase Commit: koordiniert, aber blockierend
- Praxis: lokale Transaktion, Outbox, Worker
- Idempotenz gegen doppelte Nebenwirkungen
- Saga: fachlich kompensieren (Vertiefung Vorlesung 5)

# Mandantenfähigkeit

## Was ist ein Mandant?

- engl. "tenant" -> "Multi-Tenancy"
- organisatorisch abgegrenzte Kundengruppe in einem gemeinsamen Produkt
- eigene Daten, Einstellungen, Benutzer und Berechtigungen

## Multi-Tenancy - Definition + Motivation

> Ein System bedient mehrere Mandanten gleichzeitig, wobei die Daten und Aktionen der Mandanten logisch getrennt bleiben.

- Ökonomischere Ressourcennutzung, spart Betriebskosten
- Zentrale Weiterentwicklung
- Skalierbarer Betrieb

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

## Pool: einfach umzusetzen, aber mit Risiken

```text
booking(id, organization_id, host_id, start_time, status)
```

- einfach umzusetzen
- jede relevante Abfrage ist mandantengebunden
- ein vergessener Filter wird zum Datenleck, Fehler haben große Auswirkungen
- gemeinsame Infrastruktur: kostengünstig, Last aber auch geteilt -> _Noisy Neighbours_

## Bridge: mittlere Isolation, mittlerer Aufwand

- eigenes Schema pro Mandant, aber gleiche Datenbank(instanz)
- relativ starke technische Trennung auf Datenbankebene
- Migrationen zwischen Mandanten sind aufwendiger als im Pool-Modell

Auch hybride Ansätze sind verbreitet. Häufig auch Pool für Standard-Mandanten und eigene Instanzen für regulierte oder besonders große Kunden.

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

## Security als Architekturthema

- Webengineering: OWASP Top 10, Injection, XSS, CSRF, Session-Sicherheit
- Jetzt: das Gesamtsystem, seine Grenzen und Abhängigkeiten
- Angriffsfläche wächst mit jeder Komponente und jeder Schnittstelle
- Security by Design, Defense in Depth, Least Privilege

## Threat Modeling

- Was bauen wir? Was kann schiefgehen? Was tun wir dagegen? Haben wir es gut gemacht?
- STRIDE als Checkliste: Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege
- Vertrauensgrenzen (Trust Boundaries) in Architekturdiagramme einzeichnen
- Zero Trust: kein implizites Vertrauen im internen Netz

## Identität und Zugriff

- Authentifizierung vs. Autorisierung
- OAuth 2.0 und OpenID Connect: Rollen, Flows, Tokens
- JWT: Aufbau, Prüfung, typische Fehler
- Service-zu-Service: mTLS, Service Accounts, Workload Identity
- Autorisierungsmodelle: RBAC, ABAC, ReBAC; zentral oder dezentral prüfen?

## Secrets und Verschlüsselung

- Secrets-Management: Vault, Cloud-KMS, keine Secrets im Code oder Image
- Rotation und kurzlebige Zugangsdaten
- Verschlüsselung in Transit (TLS überall) und at Rest
- Schlüsselverwaltung als eigenes Problem

## Netzwerk und Perimeter

- Segmentierung: öffentliche Zone, private Zone, Datenzone
- API Gateway: Authentifizierung, Rate Limiting, zentrale Policies
- WAF und DDoS-Schutz
- Egress-Kontrolle: Was darf das System nach außen?

## Supply Chain und Betrieb

- Abhängigkeiten: bekannte Schwachstellen, Lockfiles, Scans
- Container-Images: Basis-Images, Signaturen, SBOM
- Audit Logging: wer hat wann was getan
- Compliance: DSGVO, Datenlokalität, Löschkonzepte; Bezug zur Datenarchitektur

# AI Engineering: Daten und Sicherheit für KI-Systeme

## Neue Datentypen

- Embeddings und Vektordatenbanken: Ähnlichkeitssuche statt exakter Abfragen
- RAG-Datenpipeline: Chunking, Embedding, Indexierung, Aktualisierung
- Mandantenfähigkeit im Vektorindex: Namespace oder Filter pro Mandant, Isolation der Suchergebnisse
- Konsistenz zwischen Quellsystem und Index

## Datenschutz und Datenabfluss

- Inferenzdaten verlassen das System, wenn externe Modell-APIs genutzt werden
- Vertragliche und technische Absicherung: Auftragsverarbeitung, Regionen, keine Trainingsnutzung
- Personenbezogene Daten in Prompts, Logs und Traces
- Löschkonzepte für Embeddings und Caches

## Neue Angriffsklassen

- Prompt Injection: direkt und indirekt über Dokumente, Websites, Tool-Ausgaben
- Datenexfiltration über Tool-Aufrufe und Links
- Least Privilege für Tools und Agenten, Bestätigung bei kritischen Aktionen
- OWASP Top 10 for LLM Applications als Checkliste

# Zusammenfassung

## Zusammenfassung

- Replikation skaliert Lesen und schafft Verfügbarkeit, Partitionierung skaliert Schreiben und Datenmenge
- Konsistenz ist eine Entscheidung pro Operation, nicht pro System
- Mandantenfähigkeit: Silo, Bridge, Pool; Isolation muss erzwungen werden, nicht erhofft
- Security beginnt bei Vertrauensgrenzen und Identitäten, nicht bei der Firewall
- KI-Systeme bringen neue Datentypen und Angriffsklassen, die Regeln bleiben gleich

## Nächste Vorlesung

- Technische Entscheidungsfindung: Trade-offs, Stakeholder & Kontext (Silas)

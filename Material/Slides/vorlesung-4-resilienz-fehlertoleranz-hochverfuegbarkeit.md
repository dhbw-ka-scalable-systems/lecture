# Einstieg und Orientierung

## Skalierbare Systeme – Vorlesung 4

- Resilienz
- Fehlertoleranz
- Hochverfügbarkeit
- Recovery
- Durchgängiges Beispiel: Cal.com Teaching Model

[DARSTELLUNG: Buchungsfluss; ein externer Kalender ist rot als ausgefallen markiert]

## Lernziele

Nach der Vorlesung können die Studierenden …

- zentrale Zuverlässigkeitsbegriffe unterscheiden
- ein einfaches SLO formulieren
- kaskadierende Fehler erklären
- Timeout, Retry und Idempotenz sinnvoll kombinieren
- Redundanz und Backup unterscheiden
- ein kleines Ausfallszenario strukturiert bewerten

## Rückblick auf Vorlesung 2

- Organisation = Mandant
- PostgreSQL = System of Record für Buchungen
- Tenant Context schützt Datenzugriffe
- externer Kalender liegt außerhalb der lokalen Transaktion
- Webhook und E-Mail können separat verarbeitet werden

Neue Leitfrage:

- Was geschieht, wenn eine dieser Komponenten langsam oder nicht erreichbar ist?

## Das heutige Szenario

- Buchungsseite stark besucht
- PostgreSQL erreichbar
- externer Kalender antwortet langsam
- Clients wiederholen Requests
- Worker-Warteschlange wächst
- einzelne Organisation erzeugt besonders viel Last

Ziel:

- Kernfunktion erhalten
- Fehler begrenzen
- kontrolliert wiederherstellen

## Vier Einheiten

1. Zuverlässigkeit messbar machen
2. Kaskadierende Fehler verhindern
3. Hochverfügbarkeit und Recovery
4. Resilienz testen und auf Vorfälle reagieren

- Planungsumfang: etwa 38 Minuten je Einheit
- Puffer: etwa 7 Minuten je Einheit
- Eine vertiefende Arbeitsphase in der gesamten Vorlesung

# Einheit 1 – Zuverlässigkeit messbar machen

## Fehler sind normal

Mögliche Ursachen:

- Prozess stürzt ab
- Netzwerkpakete gehen verloren
- Abhängigkeit wird langsam
- Datenbankverbindung ist erschöpft
- falsche Konfiguration wird ausgerollt
- Last übersteigt die Kapazität

Architekturziel:

- nicht jeden Fehler verhindern
- Auswirkungen kontrollieren

## Definitionen im Überblick

| Begriff | Bedeutung |
|---|---|
| Reliability | System erfüllt seine Funktion über einen Zeitraum |
| Availability | System ist zum betrachteten Zeitpunkt nutzbar |
| Fault Tolerance | System arbeitet trotz bestimmter Fehler weiter |
| Resilience | System begrenzt Auswirkungen und erholt sich |
| High Availability | Architektur reduziert ungeplante Nichtverfügbarkeit |

Die Begriffe überschneiden sich, sind aber nicht identisch.

## Fehler, Ausfall und Störung

- **Fault:** Ursache oder Defekt
- **Error:** fehlerhafter interner Zustand
- **Failure:** von außen sichtbare Abweichung

Beispiel:

- Fault: Kalender-API nicht erreichbar
- Error: Synchronisationsauftrag schlägt fehl
- Failure: Buchung wird fälschlich abgelehnt

Resiliente Alternative:

- Buchung lokal bestätigen
- Synchronisation später wiederholen

## Partielle Fehler

- Ein Teil des Systems funktioniert
- Ein anderer Teil ist langsam oder ausgefallen
- Zustand ist nicht überall eindeutig sichtbar

Beispiel:

- TypeScript-App läuft
- PostgreSQL läuft
- Google Calendar ist nicht erreichbar

Schwierigkeit:

- „Alles läuft“ und „alles ist ausgefallen“ sind selten die einzigen Zustände

## Latenz ist ebenfalls ein Fehler

- Antwort nach 30 Sekunden technisch erfolgreich
- für Nutzer häufig nicht mehr brauchbar
- blockierte Requests verbrauchen Ressourcen
- langsame Abhängigkeiten können das Gesamtsystem mitreißen

Folgerung:

- Erfolg benötigt eine Zeitgrenze
- Timeout ist Teil der fachlichen Erwartung

## Definition: SLI

**Service Level Indicator**

- gemessene Größe für Nutzererfahrung

Beispiele:

- Anteil erfolgreicher Buchungen
- Antwortzeit der Slot-Suche
- Anteil innerhalb von 500 ms
- Aktualität der Kalendersynchronisation

Gutes SLI:

- aus Sicht der Nutzer relevant

## Definition: SLO

**Service Level Objective**

- Zielwert für ein SLI
- gilt für ein Zeitfenster

Beispiel:

- 99,5 % der gültigen Buchungsanfragen sind in 30 Tagen erfolgreich
- 95 % der Slot-Suchen antworten innerhalb von 500 ms

SLO ist kein Versprechen an einen Kunden.

## Definition: SLA

**Service Level Agreement**

- vertragliche Vereinbarung
- mögliche Konsequenzen bei Verletzung

Abgrenzung:

- SLI = Messgröße
- SLO = internes oder technisches Ziel
- SLA = Vertrag

[DARSTELLUNG: Drei ineinanderliegende Karten SLI → SLO → SLA]

## Definition: Error Budget

- tolerierte Abweichung vom SLO
- macht Zuverlässigkeit und Änderungsgeschwindigkeit vergleichbar

Beispiel:

- SLO: 99,5 % erfolgreiche Buchungen
- Error Budget: 0,5 % dürfen fehlschlagen

Wenn Budget aufgebraucht:

- Stabilisierung priorisieren
- riskante Releases reduzieren
- Ursachen bearbeiten

## Verfügbarkeit in Zahlen

Vereinfachte Betrachtung pro Jahr:

| Ziel | Maximale Ausfallzeit ungefähr |
|---:|---:|
| 99 % | 3,65 Tage |
| 99,9 % | 8,76 Stunden |
| 99,99 % | 52,6 Minuten |

Merksatz:

- Jede zusätzliche Neun wird deutlich teurer

## RTO und RPO

**Recovery Time Objective – RTO**

- Wie schnell muss der Dienst wiederhergestellt sein?

**Recovery Point Objective – RPO**

- Wie viel Datenverlust ist höchstens akzeptabel?

Beispiel:

- RTO: 60 Minuten
- RPO: 5 Minuten

RTO und RPO bestimmen Backup- und Recovery-Architektur.

## Beispielziele für das Buchungssystem

| Funktion | Beispielziel |
|---|---|
| Slot-Suche | 95 % unter 500 ms |
| Buchung | 99,5 % erfolgreich |
| E-Mail | innerhalb von 5 Minuten |
| Kalender-Sync | innerhalb von 15 Minuten |

Beobachtung:

- Nicht jede Funktion benötigt dasselbe Ziel
- asynchrone Verarbeitung kann den Kern schützen

## Einheit 1 – Merksätze

- Latenz kann einen Ausfall darstellen
- Verfügbarkeit muss aus Nutzersicht gemessen werden
- SLI misst, SLO setzt ein Ziel, SLA vereinbart Folgen
- Error Budgets schaffen eine bewusste Fehlertoleranz
- RTO und RPO beschreiben Wiederherstellungsziele

# Einheit 2 – Kaskadierende Fehler verhindern

## Vom kleinen Fehler zur Kaskade

1. Kalender-API wird langsam
2. Requests warten auf Antwort
3. Verbindungspool läuft voll
4. Clients senden Wiederholungen
5. Last steigt weiter
6. Buchungs-API fällt ebenfalls aus

[DARSTELLUNG: Domino-Kette vom externen Dienst bis zur Buchungsseite]

Definition:

- Kaskadierender Fehler = Fehler breitet sich über Abhängigkeiten aus

## Timeout

- begrenzt Wartezeit auf eine Abhängigkeit
- gibt Ressourcen wieder frei
- macht Fehler sichtbar

```ts
const response = await fetch(calendarUrl, {
  signal: AbortSignal.timeout(2_000)
});
```

Wichtig:

- Timeout passend zum Gesamtzeitbudget
- nicht überall derselbe Wert
- Timeout allein löst die fachliche Folge nicht

## Deadline statt isolierter Timeouts

- Gesamtes Request-Zeitbudget festlegen
- verbleibende Zeit an Abhängigkeiten weitergeben

Beispiel:

- Nutzerrequest: 2 Sekunden
- Datenbank: höchstens 500 ms
- Kalenderprüfung: höchstens 800 ms
- Reserve für Verarbeitung und Antwort

Nutzen:

- untergeordnete Aufrufe laufen nicht länger als der ursprüngliche Request

## Retry

- erneuter Versuch nach einem vorübergehenden Fehler

Sinnvoll bei:

- kurzzeitigem Netzwerkfehler
- temporärer Überlastung
- ausdrücklich wiederholbarer Operation

Gefährlich bei:

- Validierungsfehler
- dauerhafter Störung
- nicht idempotenter Operation
- bereits hoher Last

## Backoff und Jitter

**Backoff**

- Wartezeit wächst zwischen Versuchen

**Jitter**

- zufällige Abweichung der Wartezeit

Ziel:

- Clients wiederholen nicht gleichzeitig
- Abhängigkeit erhält Erholungszeit

```text
100 ms → 220 ms → 470 ms → Abbruch
```

Immer begrenzen:

- Anzahl der Versuche
- gesamte Dauer

## Definition: Idempotenz

- Wiederholung derselben Operation hat keine zusätzliche Wirkung

Beispiele:

- `GET /slots` ist typischerweise idempotent
- „Buchung erstellen“ ist es ohne Zusatzmechanismus nicht

Problem:

- Client erhält kein Ergebnis
- Buchung könnte trotzdem angelegt worden sein
- Client wiederholt Anfrage

## Idempotency Key

```http
POST /bookings
Idempotency-Key: 018f-example
```

Server:

1. Schlüssel prüfen
2. vorhandenes Ergebnis zurückgeben
3. sonst Operation einmal ausführen
4. Ergebnis zum Schlüssel speichern

Ziel:

- sicherer Retry ohne doppelte Buchung

## Circuit Breaker

Zustände:

- **Closed:** Aufrufe erlaubt
- **Open:** Aufrufe sofort abgelehnt
- **Half-open:** einzelne Testaufrufe

Nutzen:

- schützt Ressourcen
- verhindert aussichtslose Aufrufe
- beschleunigt Fehlerantwort

[DARSTELLUNG: Zustandsdiagramm Closed → Open → Half-open → Closed]

## Bulkhead

- Ressourcen für Bereiche trennen
- Fehler bleiben in einem Bereich

Beispiele:

- eigener Worker-Pool für Webhooks
- begrenzte Queue pro Integration
- separate Verbindungspools
- Limits pro Mandant

Namensursprung:

- Schotten in einem Schiff

## Backpressure und Load Shedding

**Backpressure**

- langsamer Verbraucher signalisiert begrenzte Kapazität

**Load Shedding**

- System lehnt weniger wichtige Arbeit kontrolliert ab

Beispiele:

- `429 Too Many Requests`
- Queue mit fester Größe
- Exporte pausieren
- Buchungen priorisieren

## Graceful Degradation

- Kernfunktion bleibt verfügbar
- weniger wichtige Funktion wird reduziert

Bei Ausfall der Kalender-API:

- Buchung lokal speichern
- Synchronisation als ausstehend markieren
- Nutzer transparent informieren
- später erneut versuchen

Nicht akzeptabel:

- Buchung stillschweigend verlieren

## Stabilitätsmuster zusammenführen

Für einen externen Kalenderaufruf:

1. Deadline
2. Timeout
3. wenige Retries
4. Backoff und Jitter
5. Idempotency Key
6. Circuit Breaker
7. Queue mit Limit
8. beobachtbarer Fehlerzustand

Nicht jedes Muster ist immer erforderlich.

## Einheit 2 – Merksätze

- Timeouts begrenzen Wartezeit und Ressourcenverbrauch
- Retries erhöhen Last und benötigen Grenzen
- Idempotenz macht Wiederholungen kontrollierbar
- Circuit Breaker verhindert aussichtslose Aufrufe
- Bulkheads begrenzen den Auswirkungsbereich
- Überlast muss kontrolliert abgewiesen werden können

# Einheit 3 – Hochverfügbarkeit und Recovery

## Definition: Single Point of Failure

- Einzelne Komponente kann den Gesamtdienst stoppen

Beispiele:

- eine App-Instanz
- eine Datenbank ohne Wiederherstellungsplan
- ein Load Balancer ohne Ersatz
- ein einziges Secret ohne Backup

Vorgehen:

1. kritischen Pfad bestimmen
2. einzelne Ausfallpunkte identifizieren
3. Risiko und Kosten bewerten

## Redundanz

- kritische Komponente mehrfach vorhanden
- Instanzen sollten nicht dieselbe Fehlerursache teilen

Schlechte Redundanz:

- zwei Prozesse auf demselben defekten Host

Bessere Redundanz:

- mehrere Instanzen
- getrennte Failure Domains
- automatisches oder dokumentiertes Failover

## Failure Domain

- Bereich mit gemeinsamer Ausfallursache

Beispiele:

- Prozess
- Host
- Rack
- Availability Zone
- Region
- Cloud-Anbieter

Merksatz:

- Redundanz wirkt nur über relevante Failure Domains hinweg

## Mehrere App-Instanzen

[DARSTELLUNG: Load Balancer vor drei zustandslosen TypeScript-App-Instanzen]

Voraussetzungen:

- kein wichtiger Zustand nur im Prozessspeicher
- Sessions geteilt oder signiert
- Dateien nicht nur lokal gespeichert
- Health Checks vorhanden
- Instanzen unabhängig deploybar

## Health Checks

**Liveness**

- Prozess lebt grundsätzlich
- bei Fehler eventuell neu starten

**Readiness**

- Instanz kann aktuell Traffic annehmen
- bei Fehler aus Load Balancer entfernen

Vorsicht:

- Readiness nicht von jeder optionalen Abhängigkeit abhängig machen
- sonst verstärkt eine Störung den Ausfall

## Datenbankreplikation

- Daten werden auf weitere Instanz kopiert
- verbessert Leseleistung oder Verfügbarkeit

Mögliche Rollen:

- Primary verarbeitet Writes
- Replica verarbeitet Reads oder wartet auf Failover

Neue Probleme:

- Replikationsverzögerung
- Failover-Entscheidung
- möglicher Datenverlust
- zusätzliche Betriebskomplexität

## Konsistenz bei Replikationsverzögerung

Beispiel:

1. Nutzer erstellt Buchung auf Primary
2. UI liest sofort von Replica
3. Replica hat Änderung noch nicht
4. Buchung scheint verschwunden

Mögliche Lösung:

- kritische Reads nach Write vom Primary
- UI zeigt bestätigtes Ergebnis direkt
- Verzögerung bewusst kommunizieren

## Backup ist keine Replikation

**Replikation**

- kopiert auch versehentlich gelöschte oder beschädigte Daten
- verbessert häufig Verfügbarkeit

**Backup**

- historischer Wiederherstellungspunkt
- schützt gegen Löschung und Beschädigung

Benötigt werden häufig beide.

## Restore entscheidet über den Wert des Backups

Ein Backup gilt erst als belastbar, wenn …

- Wiederherstellung getestet wurde
- Dauer bekannt ist
- Verantwortlichkeiten klar sind
- Schlüssel und Konfiguration verfügbar sind
- Anwendung mit den Daten starten kann

Bezug zu Zielen:

- Restore-Zeit ≤ RTO
- Datenstand erfüllt RPO

## Active/Passive und Active/Active

**Active/Passive**

- zweite Umgebung wartet auf Übernahme
- meist einfacher
- Umschaltzeit erforderlich

**Active/Active**

- mehrere Umgebungen bearbeiten Traffic
- schnelle Übernahme möglich
- Schreibkonflikte und Routing komplexer

Für viele Systeme:

- einfache Lösung mit gutem Recovery besser als ungetestetes Active/Active

## Hohe Verfügbarkeit ist eine Ende-zu-Ende-Eigenschaft

Kritischer Pfad einer Buchung:

- DNS
- Load Balancer
- App-Instanz
- PostgreSQL
- notwendige Secrets

Optionaler Pfad:

- E-Mail
- externer Kalender
- Analytics

Architekturentscheidung:

- optionale Pfade dürfen Kernfunktion nicht unnötig stoppen

## Einheit 3 – Merksätze

- Redundanz benötigt getrennte Failure Domains
- Health Checks steuern Traffic und Neustarts unterschiedlich
- Replikation erzeugt neue Konsistenzfragen
- Backup und Replikation lösen unterschiedliche Probleme
- Recovery muss praktisch getestet werden
- Hochverfügbarkeit betrifft den gesamten kritischen Pfad

# Einheit 4 – Resilienz testen und reagieren

## Resilienz ist keine Behauptung

Nachweis durch:

- automatisierte Tests
- Lasttests
- Fehler-Injektion
- Restore-Tests
- Failover-Übungen
- Incident-Reviews

Grundsatz:

- Verhalten unter Fehlern vor dem echten Notfall kennenlernen

## Lasttest, Stresstest und Soak Test

**Lasttest**

- erwartete Last

**Stresstest**

- Last bis zur Grenze

**Soak Test**

- Last über längeren Zeitraum

Zu beobachten:

- Antwortzeiten
- Fehlerrate
- Ressourcennutzung
- Queue-Länge
- Datenkorrektheit

## Fehler-Injektion

Kontrollierte Beispiele:

- 2 Sekunden zusätzliche Latenz
- jeder zehnte Request schlägt fehl
- Verbindung wird getrennt
- Datenbankpool stark begrenzt

Werkzeuge im Lehrsetup:

- Toxiproxy oder einfacher Calendar Stub
- k6 für Last
- Docker Compose zum Starten der Umgebung

Sicherheitsregel:

- klarer Scope und Abbruchmechanismus

## Definition: Chaos Engineering

- kontrolliertes Experiment an einem System
- überprüft eine konkrete Hypothese
- beobachtet Verhalten unter Störung

Beispielhypothese:

- „Buchungen bleiben möglich, wenn die Kalender-API fünf Minuten ausfällt.“

Nicht:

- zufällig Produktion kaputtmachen

## Minimales Experiment

1. stabilen Ausgangszustand prüfen
2. Hypothese formulieren
3. kleine Störung injizieren
4. relevante Signale beobachten
5. Abbruchbedingung definieren
6. Ergebnis auswerten

[DARSTELLUNG: Experimentkreislauf mit sechs Schritten]

## Beobachtbare Signale

- erfolgreiche Buchungen pro Minute
- p95-Antwortzeit
- Fehlerquote der Kalender-API
- Größe der Sync-Queue
- ältester wartender Auftrag
- Anzahl geöffneter Circuit Breaker

Ausblick:

- Telemetrie und Alerting werden in Vorlesung 6 vertieft

## Arbeitsphase: Incident Tabletop

**Ziel-Zeit: 12 Minuten**

Situation:

- Kalender-API benötigt plötzlich 20 Sekunden
- Buchungs-API wird langsamer
- Queue wächst
- Fehlerrate steigt

Auftrag in Dreiergruppen:

1. unmittelbare Schutzmaßnahme wählen
2. Kernfunktion und optionale Funktion trennen
3. zwei Messgrößen festlegen
4. dauerhafte Verbesserung vorschlagen

Ergebnis:

- vier Stichpunkte, keine vollständige Architektur

## Auswertung der Arbeitsphase

Plausible Sofortmaßnahmen:

- Kalenderaufruf aus Request-Pfad entfernen
- Timeout reduzieren
- Circuit Breaker öffnen
- Retry-Rate begrenzen
- Queue begrenzen

Plausible Messgrößen:

- Buchungserfolgsrate
- p95-Latenz
- Queue-Alter
- Fehlerquote der Abhängigkeit

## Incident Response – Grundablauf

1. Störung erkennen
2. Auswirkung und Priorität bestimmen
3. Verantwortliche benennen
4. Schaden begrenzen
5. Dienst wiederherstellen
6. Ursache untersuchen
7. Maßnahmen nachverfolgen

Priorität während des Incidents:

- Wiederherstellung vor perfekter Ursachenanalyse

## Blameless Postmortem

Enthält:

- Auswirkung
- Zeitlinie
- technische und organisatorische Faktoren
- hilfreiche und fehlende Signale
- konkrete Maßnahmen mit Verantwortlichen

Vermeidet:

- Suche nach einer schuldigen Person
- „menschlicher Fehler“ als Enderklärung

## Gesamtes Schutzkonzept

[DARSTELLUNG: Schichtenmodell]

1. messbare Ziele
2. Zeit- und Lastgrenzen
3. Idempotenz und Wiederholung
4. Isolation durch Bulkheads
5. Redundanz
6. Backup und Recovery
7. Tests und Incident-Lernen

## Wichtigste Begriffe

- Reliability
- Availability
- Fault Tolerance
- Resilience
- SLI, SLO, SLA
- Error Budget
- RTO, RPO
- Timeout, Retry, Backoff, Jitter
- Idempotenz
- Circuit Breaker
- Bulkhead
- Backpressure
- Failure Domain
- Graceful Degradation

## Takeaways

- Fehler und langsame Antworten müssen erwartet werden
- Ziele werden aus Nutzersicht formuliert
- Retries ohne Grenzen verschlimmern Störungen
- Idempotenz schützt vor doppelter Wirkung
- Redundanz ersetzt kein Backup
- Recovery und Resilienz müssen getestet werden

# Literatur und Quellen zum Nachschlagen

## Literaturhinweis

- Keine Pflichtlektüre
- Kapitel dienen zum Nacharbeiten und Nachschlagen
- Auswahl passend zum konkreten Problem

## Stabilitätsmuster

- Nygard: *Release It!*, 2. Auflage
  - Kapitel 3: Stabilize Your System
  - Kapitel 4: Stability Antipatterns
  - Kapitel 5: Stability Patterns
  - Kapitel 17: Chaos Engineering
- Google: *Site Reliability Engineering*
  - Kapitel 3: Embracing Risk
  - Kapitel 4: Service Level Objectives
  - Kapitel 17: Testing for Reliability
  - Kapitel 21: Handling Overload
  - Kapitel 22: Addressing Cascading Failures

## Verteilte Daten und Recovery

- Kleppmann, Riccomini: *Designing Data-Intensive Applications*, 2. Auflage
  - Kapitel 2: Defining Nonfunctional Requirements
  - Kapitel 6: Replication
  - Kapitel 9: The Trouble with Distributed Systems
  - Kapitel 10: Consistency and Consensus
- Google: *Building Secure and Reliable Systems*
  - Kapitel 8: Design for Resilience
  - Kapitel 9: Design for Recovery
  - Kapitel 10: Mitigating Denial-of-Service Attacks
  - Kapitel 16: Disaster Planning

## Architekturmerkmale

- Richards, Ford: *Fundamentals of Software Architecture*, 2. Auflage
  - Kapitel 4–6: Architecture Characteristics
  - Kapitel 22: Analyzing Architecture Risk
- Ford et al.: *Software Architecture: The Hard Parts*
  - Kapitel 3: Architectural Modularity
  - Kapitel 8: Reuse Patterns

## Frei verfügbare Quellen

- [Google SRE Book](https://sre.google/sre-book/table-of-contents/)
- [Google SRE Workbook](https://sre.google/workbook/table-of-contents/)
- [Building Secure and Reliable Systems](https://google.github.io/building-secure-and-reliable-systems/raw/toc.html)
- [Cal.com Webhooks](https://cal.com/docs/developing/guides/automation/webhooks)

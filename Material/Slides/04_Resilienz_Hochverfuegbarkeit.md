---
title: "Skalierbare Systeme - Resilienz, Fehlertoleranz & Hochverfügbarkeit"
topic: "scalable_systems_1_4"
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

## Heute

- Zuverlässigkeit aus Nutzersicht messen.
- Fehler begrenzen, bevor sie zur Kaskade werden.
- Hochverfügbarkeit und Recovery entwerfen.
- Resilienz praktisch testen und aus Vorfällen lernen.
- AI Engineering: KI-Komponenten als unzuverlässige Abhängigkeiten.
- Durchgängiges Beispiel: Cal.diy-Terminplattform aus Vorlesung 2.

## Lernziele

- Zuverlässigkeit, Verfügbarkeit, Fehlertoleranz und Resilienz unterscheiden.
- Ein SLI und SLO für einen Anwendungsfall formulieren.
- Fehlerkaskaden erkennen und passende Stabilitätsmuster kombinieren.
- Redundanz, Replikation und Backup voneinander abgrenzen.
- Ein Ausfallszenario bewerten und eine Schutzmaßnahme begründen.

## Unser Szenario: Cal.diy unter Störungen

- Die Buchungsseite erhält ungewöhnlich viele Anfragen
- PostgreSQL und die Buchungs-API sind erreichbar
- Die externe Kalender-API antwortet immer langsamer
- Clients wiederholen fehlgeschlagene Requests
- Die Synchronisationsaufträge sammeln sich; ein Mandant erzeugt besonders viel Last

Leitfrage: Wie bleibt die Buchung nutzbar, ohne Daten zu verlieren oder andere Mandanten mitzuziehen?

## Buchungsfluss und Fehlergrenzen

```plantuml
@startuml
left to right direction
actor Gast
rectangle "Buchungs-API" as API
database "PostgreSQL\nBuchungen + Outbox" as DB
rectangle Worker
cloud "Kalender / E-Mail" as External
Gast --> API : buchen
API --> DB : Buchung + Outbox atomar speichern
Worker --> DB : Aufträge lesen / quittieren
Worker --> External : nachgelagert übertragen
@enduml
```

- Buchung und Outbox-Eintrag werden gemeinsam gespeichert
- Buchungserfolg und externe Synchronisation sind getrennte Zustände
- Die Plattform verwaltet buchbare Slots verbindlich; externe Kalender werden nachgelagert synchronisiert

# Zuverlässigkeit messbar machen

## Fehler sind normal

- Wir müssen mit Fehlern rechnen
  - Prozesse stürzen ab
  - Netzwerke verlieren Pakete
  - externe Dienste sind langsam/überlastet/nicht erreichbar
  - Fehler beim Ausrollen neuer Versionen
  - Programmierfehler, Bugs
  - Komplettausfall Infrastruktur

Architekturziel: Nicht jeden Fehler verhindern, sondern seine Auswirkungen kontrollieren.

## Reliability, Availability und Resilienz

| Begriff | Definition |
|---|---|
| **Availability** | Ist der Dienst zum benötigten Zeitpunkt nutzbar? |
| **Reliability** | Erfüllt das System seine Funktion über einen Zeitraum korrekt? |
| **Fault Tolerance** | Arbeitet das System trotz bestimmter Fehler weiter? |
| **Resilience** | Begrenzt das System Auswirkungen und erholt sich? |
| **High Availability** | Welche Architektur reduziert ungeplante Nichtverfügbarkeit? |

## High Availability

- Ziel: (weitgehend) unterbrechungsfreier Betrieb des Systems auch bei Ausfällen
- Verfügbarkeitsziel "x nines" (99,9 %, 99,999 %)
- Redundanz + Fehlertoleranz/Fehlererkennung + kein Single Point of Failure
- Wiederholung CAP Theorem: AP -> Available und partition tolerant, Datenkonsistenz lässt sich nicht immer einhalten

## Fault, Error, Failure

- **Fault:** Ursache oder Defekt, zum Beispiel ein Fehler in der Prüfung freier Termine.
- **Error:** fehlerhafter interner Zustand, zum Beispiel ein belegter Termin wird als frei geführt.
- **Failure:** von außen sichtbare Abweichung, zum Beispiel eine bestätigte Doppelbuchung.
- Ein Fault führt nicht zwangsläufig zu einem Failure; das System kann den Error erkennen und abfangen.

## Partielle Fehler und Latenz

- Grundlage: verteiltes System, keine monolithische Anwendung
- Ein Teil des Systems funktioniert, während eine andere Komponente langsam oder ausgefallen ist.
- Oft ist unklar, ob eine entfernte Komponente defekt oder nur langsam ist.
- Eine Antwort nach 30 Sekunden kann technisch erfolgreich, für den Nutzer aber unbrauchbar sein.
- Wartende Requests belegen Threads, Verbindungen und Speicher.

Folge: Auch erfolgreiche Aufrufe brauchen eine Zeitgrenze.

## Verfügbarkeit messen

- Eine Prozentzahl ist nur aussagekräftig, wenn _Leistung_ und _Messzeitraum_ festgelegt sind.
- **Zeitbasiert:** Anteil der Zeit, in der der Dienst die vereinbarte Funktion erfüllt.
- **Anfragebasiert:** Anteil erfolgreicher Anfragen; ein anderes SLI als Uptime in Minuten.
- Geplante Wartung zählt nur dann nicht als Ausfall, wenn die Messdefinition sie ausdrücklich ausschließt!

## SLI, SLO und SLA

- **SLI (Service Level Indicator):** gemessene Größe aus Nutzersicht, z.B. Anteil erfolgreicher Buchungen oder Antwortzeit.
- **SLO (Service Level Objective):** Zielwert für ein SLI in einem festgelegten Messfenster, z.B. 99,9 % über 28 Tage.
- **SLA (Service Level Agreement):** vertragliche Vereinbarung mit möglichen Konsequenzen.
- Ein internes SLO sollte ausreichend Puffer zum vertraglichen SLA lassen!

## Die Neunen in Zeit übersetzt

- Annahme: 365 Tage im Jahr, 24/7-Betrieb; geplante Wartung ist nicht ausgenommen.
- **99 %:** <= 3 Tage, 15 Stunden und 36 Minuten
- **99,9 %:** <= 8 Stunden und 46 Minuten 
- **99,99 %:** <= 52 Minuten und 34 Sekunden
- **99,999 %:** <= 5 Minuten und 15 Sekunden
- Jede zusätzliche Neun erhöht den Aufwand deutlich!

## MTBF und MTTR

- **MTBF (Mean Time Between Failures):** durchschnittliche Betriebszeit zwischen zwei Ausfällen.
- Längere MTBF bedeutet, dass Ausfälle seltener auftreten; sie beschreibt nicht deren Dauer.
- **MTTR (Mean Time To Repair/Restore):** durchschnittliche Dauer bis zur Wiederherstellung.
- Messbeginn und -ende festlegen, zum Beispiel vom Beginn der Beeinträchtigung bis zur Wiederherstellung.
- Näherung für einen reparierbaren Dienst: $Verfuegbarkeit ≈ MTBF / (MTBF + MTTR)$.
- Bei MTBF 100 Stunden und MTTR 1 Stunde ergibt sich ungefähr 99,0 % Verfügbarkeit.


## Unser Buchungs-SLI: Was zählt als Erfolg?

- **Grundgesamtheit:** eindeutige, gültige _Buchungsabsichten_ für einen _zuvor als frei angezeigten_ Slot.
  - technische Retries zählen nicht erneut
  - Ungültige Eingaben, fehlende Berechtigungen und Health Checks sind ausgeschlossen
- **Gutes Ereignis:** Buchung dauerhaft gespeichert und Bestätigung innerhalb von _2 Sekunden_ zurückgegeben.
- Fehler, Timeout, verspätete oder fehlende Bestätigung zählen als schlecht.
- **Messfenster:** rollierende 28 Tage; mindestens 99,9 % der Buchungsabsichten sollen gut sein

## Fehlerbudget berechnen

- Annahme: In 28-Tagen gibt es 50.000 eindeutige Buchungsabsichten: 49.960 gut, 40 schlecht
  - **SLI:** $49.960 / 50.000 = 99,92 \%$
  - **Fehlerbudget:** höchstens $0,1 \% × 50.000 = 50$ schlechte Buchungen.
- Verbraucht sind 40 von 50 Ereignissen 80 %
- Das Budget zählt Nutzerereignisse, nicht Minuten; ein später Retry macht einen Fehler nicht rückwirkend gut.

## Verfügbarkeit verketteter Komponenten

- Sind mehrere Komponenten für eine Anfrage zwingend nötig, müssen alle verfügbar sein.
- Angenommen Ausfälle sind unabhängig voneinander: Multiplikation der Verfügbarkeiten API $99,9 \% × Datenbank 99,9 \% ≈ 99,801 \% Gesamt$.
  - In der Praxis sind Fehler nur selten wirklich unabhängig
  - Gemeinsame Fehlerursachen oder Verstärkung durch Kaskaden reduzieren Gesamtverfügbarkeit deutlich
- Redundanz kann Verfügbarkeit verbessern, beseitigt aber keine gemeinsamen Fehlerursachen.

## Warum verteilte Systeme scheitern

- Verteilte Systeme sind komplex
- Fehler in einzelnen Komponenten können immer auftreten
- Fehler in Infrastruktur und Netzwerk können immer auftreten
- Zusammenspiel sorgt für unvorhergesehene Effekte^

## Fallacies of Distributed Computing (1/2)

Peter Deutschs Annahmen, die beim Entwurf verteilter Systeme oft falsch sind:

- **The network is reliable:** Pakete können verloren gehen und Verbindungen abbrechen.
- **Latency is zero:** Antworten brauchen Zeit; Timeouts sind nur Grenzwerte.
- **Bandwidth is infinite:** Konkurrenz und Last begrenzen den Durchsatz.
- **The network is secure:** Angriffe und Fehlkonfigurationen sind möglich.

## Fallacies of Distributed Computing (2/2)

- **Topology doesn't change:** Knoten, Routen und Adressen können wechseln.
- **There is one administrator:** Systeme und Regeln liegen oft in verschiedenen Zuständigkeiten.
- **Transport cost is zero:** Übertragung braucht Zeit und Ressourcen.
- **The network is homogeneous:** Protokolle, Systeme und Geräte unterscheiden sich.

## Ausfallarten

- **Crash:** Eine Komponente stoppt.
- **Omission:** Nachrichten werden nicht gesendet oder empfangen (ausgelassen).
- **Timing:** Antworten kommen zu spät oder außerhalb vereinbarter Grenzen.
- **Korrelierte, kaskadierende und metastabile Ausfälle** können lokale Fehler verstärken.

# Fehlerkaskaden verhindern

## Vom langsamen Kalender zur Kaskade

1. Die Kalender-API wird langsam
2. Requests warten und belegen Verbindungen
3. Der Verbindungspool der Anwendung läuft voll
4. Clients und Worker wiederholen Aufrufe
5. Last und Queue-Rückstand wachsen
6. Schließlich wird auch die Buchungs-API unbenutzbar

Bleibt ein überlastetes System auch nach Ende des Auslösers instabil, spricht man von einem metastabilen Ausfall.

## Aufgabe: Wo zuerst eingreifen?

**Arbeitszeit: 5 Minuten — 2 allein, 3 zu zweit**

Im Kalender-Szenario:

1. Welche Ressource wird zuerst knapp?
2. Welche Schutzmaßnahme würdet ihr zuerst einsetzen?
3. Was sollte für den Nutzer weiterhin funktionieren?

Ergebnis: eine erste Maßnahme mit kurzer Begründung.


## Timeouts: ein Zeitbudget für den gesamten Request

- Jede Netzwerkoperation braucht Zeit, Timeouts verhindern, dass ein Request unbegrenzt blockiert.
- Beispiel: Für die interaktive Buchung stehen insgesamt 2 Sekunden zur Verfügung
  - Aufrufe, Unteraufrufe und Wiederholungen teilen dieses Budget
  - Verbindungsaufbau und Antwortzeit sind einbezogen
- Verbleibende Deadline an nachgelagerte Dienste weitergeben
- Schwierig: Wahl der Timeouts
  - zu kurz: unnötige Wiederholungen, erzeugen unnötige Last
  - zu lang: blockieren Ressourcen und erhöhen die Latenz

## Wiederholung HTTP Status Codes

- **1xx:** Informational
- **2xx:** Success
- **3xx:** Redirection
- **4xx:** Client Error
- **5xx:** Server Error

## Retries

- Retries sind nur bei transienten (vorübergehenden) Fehlern sinnvoll, zum Beispiel:
  - Netzwerkfehler
  - HTTP 503 Service Unavailable
  - HTTP 429 Too Many Requests (mit `Retry-After`)
- Nicht ohne Änderung wiederholen bei:
  - HTTP 400 Bad Request
  - HTTP 401 Unauthorized
  - HTTP 403 Forbidden
  - HTTP 404 Not Found
  - HTTP 409 Conflict (z.B. Doppelbuchung)
  - HTTP 422 Unprocessable Entity (ungültige Eingabe)
- Status Codes sind nur eine Auswahl. Wichtig ist, dass alle Komponenten sich bei der Semantik der Codes einig sind

## Backoff und Jitter

- **Backoff:** Wartezeit zwischen Wiederholungen, um Lastspitzen zu vermeiden.
  - **Fixed Backoff:** feste Wartezeit, z.B. 2s zwischen jedem Versuch
  - Bei HTTP 429 `Retry-After` berücksichtigen, sofern vorhanden
  - **Linear Backoff:** feste Wartezeit, z.B. 1s, 2s, 3s
  - **Exponential Backoff:** exponentiell steigende Wartezeit, z.B. 1s, 2s, 4s, 8s
- **Jitter:** Wartezeiten zufällig streuen, damit Clients nicht gleichzeitig erneut anfragen
  - Insbesondere bei vielen Clients sinnvoll, ansonsten können sich Lastspitzen zyklisch wiederholen
- Retry nur wenn Operation mindestens idempotent

## Idempotenz macht Wiederholung sicher

- **Idempotent:** egal ob 1..n mal ausgeführt
- **Safe:** egal ob 0..n mal ausgeführt
- Festlegung für den Anwendungsfall und Nutzung der bekannten HTTP Semantik
  - idempotent: GET, PUT, DELETE, HEAD, OPTIONS
  - safe: GET, HEAD, OPTIONS
  - Immer nur bei richtiger Implementierung!
- Beispiel: Buchung stornieren ist idempotent; neue Buchung anlegen nicht ohne weiteres.

## Cal.diy: Timeout bei Buchung

- Buchung per POST and die API
- Die Datenbank bestätigt den Commit, aber die Antwort erreicht den Gast nicht.
- Nach Ablauf des Timeouts zeigt der Browser einen unklaren Status.
- Wie können wir trotz nicht idempotenter HTTP Operation die Aktion als solches korrekt wiederholen?


## Lösung: Idempotency Key

- Der Client erzeugt einen eindeutigen Schlüssel pro Buchungsabsicht
- Die API speichert den Schlüssel zusammen mit dem Ergebnis
- Bei einem Retry kann die API erkennen, ob der Schlüssel bereits verarbeitet wurde und das ursprüngliche Ergebnis zurückgeben
- Erlaubt es auch nicht idempotente HTTP Operationen wie POST sicher zu wiederholen


## Cal.diy: Verlorene Antwort und sicherer Retry

```plantuml
@startuml
hide footbox
participant Browser
participant "Buchungs-API" as API
database PostgreSQL as DB
Browser -> API : POST /bookings, Key K
API -> DB : Transaktion: Key K + Buchung\n+ Outbox + Ergebnis speichern
DB --> API : Commit erfolgreich
API ->x Browser : Antwort geht verloren
note over Browser : Timeout: Ergebnis unbekannt
Browser -> API : gleicher Request, gleicher Key K
API -> DB : Ergebnis für K lesen
DB --> API : gespeicherte Buchungs-ID
API --> Browser : ursprüngliches Ergebnis
@enduml
```

## Idempotency Keys korrekt absichern

- Keys müssen zufällig erzeugt werden und eindeutig sein
- Übereinstimmung mit Idempotency Key in der Datenbank darf vorhandene Sicherheitsüberprüfungen nicht umgehen!
- Prüfen ob die Operation wirklich die gleiche ist, ansonsten ablehnen
- Speichern von Buchung + Schlüssel + Outbox Eintrag muss über Transaktion gesichert werden

## Cal.diy: Wenn Retries den Kalenderausfall verstärken (1)

- Der Kalenderdienst wird langsam; der Outbox Worker wiederholt die Übertragung
  - HTTP-Client im Worker selbst nutz interne Retry-Logik
  - HTTP Proxy der für die Verbindung genutzt wird, hat ebenfalls Retry-Logik
  - **worst case** $3^3 = 27$ Kalenderaufrufe für einen Auftrag
- Jeder Retry erzeugt Last und blockiert Ressourcen
  - Kann Antworten weiter verzögern
  - Queues wachsen und blockieren weitere Aufträge
  - Löst weitere Timeouts auch für andere Clients aus

## Cal.diy: Wenn Retries den Kalenderausfall verstärken (2)

- Besser:
  - Retries an einer einzigen verantwortlichen Stelle bündeln und zusätzliche Versuche insgesamt begrenzen
  - Backoff mit Jitter vermeiden Lastspitzen
  - Insbesondere exponentielles Backoff kann dem System Zeit geben, sich zu erholen


## Circuit Breaker

- Schutz vor Fehlerkaskaden und übermäßiger Last
- **Closed:** Aufrufe passieren; Fehlerquote oder Timeout-Schwellwert öffnet den Breaker
- **Open:** Aufrufe werden sofort abgewiesen oder auf einen Fallback gelenkt
  - Circuit Breaker kann zentral prüfen, ob der Dienst wieder verfügbar ist und dann die ersten Requests wieder durchlassen
  - **half-open** nur wenige Probeaufrufe zu, wenn Dienst stabil ist, wird der Breaker wieder geschlossen

## Bulkheads: Ressourcen isolieren

- Getrennte Ressourcen-Pools um Fehler zu isolieren
  - isolierte Thread-Pools / Container / VMs / Hosts 
  - z.B. Trennung von Kalender-Worker und E-Mail Versand
- Hohe Auslastung in einem Pool hat keine Auswirkungen auf andere Pools
- Ausfälle können auf einzelne Systemteile begrenzt werden
- Aufteilung der Ressourcenpools können wichtige Systemteile bevorzugen
- ABER: Ressourcen werden nicht so effizient genutzt


## Backpressure: Erzeuger bremsen

- Grundlegendes Prinzip zur Synchronisation von Sender (producer) und Empfänger (consumer)
  - Problem: producer erzeugt schneller als consumer verarbeiten kann und sort dort für hohe Last und Rückstau
- **Lösung**: consumer steuert Interaktion
  - Pull-Prinzip: consumer fordert aktiv Daten an, producer liefert nur so viel wie angefordert
  - Buffer: producer schreibt in beschränkten Buffer, consumer liest daraus, wenn Buffer voll kann producer drosseln oder pausieren
  - Drop-Prinzip: consumer kann bei voller Queue Aufträge verwerfen oder ablehnen

## Rate Limits & Quotas

- **Rate Limits:** Begrenzen von Anfrage
  - z.B. pro User, pro IP-Adresse, pro Funktion ...
  - Auch als Schutz vor übermäßigen Kosten und Missbrauch -> z.B. LLM Tokens
  - Verschiedene Rate-Limit-Strategien: Token Bucket, Fixed Window, Sliding Window
  - Strategien sind haben verschiedene Vor- und Nachteile und sind abhängig von der Art der Last (z.B. gleichmäßig vs. bursts)
- **Quotas:** Allgemeinere Ressourcen-Kontingente
  - z.B. Speicherplatz, CPU-Zeit
  - Oft über längere Zeiträume (z.B. pro Tag, pro Monat) und nicht nur kurzfristig wie Rate Limits
  - Durchsetzen von Abo-Stufen und allgemeine Fairness

## Load Shedding 

- **Load Shedding:** weniger wichtige Anfragen geziehlt ablehnen, um Kernfunktionalität zu schützen
  - Hauptfunktion bleibt so lange wie möglich verfügbar oder wichtigste Kunden können weiterhin bedient werden
  - z.B. Analyse, Reporting, Hintergrundjobs zuerst ablehnen um Ressourcen für Kernfunktion zu schützen
  - Komplex: aktuelle Last messen/einschätzen, Prioritäten bestimmen

## Graceful Degradation

- System fällt nicht komplett aus, sondern reduziert Funktionalität
  - ähnlich wie Load Shedding, oft aber vorgelagert
    - Keine aktive Ablehnung von Anfragen, sondern Reduktion der Qualität
    - Abschaltung von optionalen Features
  - z.B. UI reduziert Aktualisierungshäufigkeit, um Datenbank zu schonen

## Stabilitätsmuster in der Praxis

- Kombination dieser Muster, für ein wirklich stabiles System
- Auswahl abhängig von Zugriffsmustern, Systemdesign, externen Abhängigkeiten, Last, Fehlerverhalten, ...

# Hochverfügbarkeit und Recovery

## Single Point of Failure & Failure Domains

- Nicht jede Komponente ist gleich wichtig
  - Ausfall bestimmer Komponenten kann aber das gesamte System stoppen -> **Single Point of Failure** (SPoF)
- SPoF typischerweise: Datenbank, einzelner App-Server
- ABER: auch DNS, Internetanbindung etc. können problematisch sein.
- **Failure Domain**: physicher/logischer Bereich, der von einem einzelnen HW oder Softwarefehler betroffen ist
  - z.B. einzelner Server, komplettes Netzwerksegment, ganzes Rechenzentrum
  - Auswirkungen sind vom Systemdesign abhängig!

## Aufgabe: Failure Domains

**Arbeitszeit: 5 Minuten — 2 allein, 3 zu zweit**

Zwei App-Instanzen stehen im selben Rechenzentrum und nutzen denselben Load Balancer:

1. Gegen welche Ausfälle helfen die zwei Instanzen?
2. Welche gemeinsamen Ausfälle bleiben möglich?

## Redundanz 

- **Redundanz**: Zusätzliche Systeme/Server/Komponenten, können im Fehlerfall übernehmen
  - Verschiedene Ebenen: Prozesse, Container/Pods, Hosts, Regionen
  - Abhängig von Ausfallsicherheitsanforderungen
- Zustandslose Services lassen sich leichter replizieren
  - Keine Synchronisation von Datenzugriffen notwendig
  - Schneller Startzeiten möglich
  - Generell: je weniger Zustand, desto einfacher
    - Auch ein Grund Datenbanken pro Service aufzuteilen -> siehe Vorlesung 2

## Redundanz Active-Active

- mehrere Replikate sind gleichzeitig aktiv und teilen die Last
  - Sehr schnelle Umschaltzeit
  - Komplexität durch Synchronisierung und Datenkonsistenz
  - bestehende Replikate müssen potenziell mit mehr Last umgehen können
  - Beispiel: zwei Instanzen, eine Fällt aus -> eine Instanz muss alleine die Last bewältigen können

## Redundanz Active-Passive

- "Standby" Redundanz
- Primärsystem bearbeitet im Normalbetrieb alle Anfragen
- Sekundärsystem (für 1:1 Ersatz identisch) ist im Normalbetrieb aus bzw. im Standby
  - Springt im Fehlerfall ein
  - höhere Umschaltzeiten als Active-Active
  - Keine Datenkonsistenzprobleme, aber potenziell veraltete Daten
  - Verschiedene Umsetzungsmöglichkeiten: Hot/Warm/Cold -> wie aktiv ist das System, wie aktuell die Daten

## Failover

- Für Umschaltung auf redundante System ist Erkennung von Ausfall notwendig -> **Failover**
- Passive Überwachung durch Load-Balancer / Proxy: erkennen von hohen Timeouts, ausbleibenden Antworten
- **Heartbeat*: regelmäßiges kurzes Signal, dass System aktiv ist
  - Heißt aber nicht unbedingt, dass das System auch voll funktionsfähig ist!
  - Kann um detailliertere Healthcheck-Informationen erweitert werden
  - n verpasste Heartbeats als Schwelle zum Umschalten
- Ausbleibende Heartbeats sind nicht unbedingt ein Ausfall des Systems, Ausfall des Netzwerks oder des Heartbeat-Services sind auch möglich!

## Split Brain 

- Verteiltes System teilt sich in voneinander isolierte Teilsysteme
  - z.B. fehlerhaft ein Ausfall diagnostiziert und redundantes System aktiv ohne Kommunikation zwischen beiden
- Beide Teilsysteme (theoretisch auch mehr als 2, aber sehr unwahrscheinlich) wissen nichts voneinander und greifen schreibend ohne synchronsiation auf gleiche Daten
- Datenkonsistenz wird nicht mehr sichergestellt
- Bei Wiederherstellung der Kommunikation müssen Konflikte aufgelöst werden, das ist nicht immer ohne Datenverlust möglich
- Auswirkungen abhängig von Datenarchitetkur!

## Zusammenspiel mit Datenarchitektur

- Datenmodell, Aufteilung der Daten nach Services, Replikation + Sharding haben große Auswikrungen auf High Availability
- Einige Entscheidungen in der Datenarchitektur können HA erleichtern oder erschweren
- Teil-Ausfälle eines Systems können dafür sorgen dass temporär veraltete Daten vorhanden sind oder sogar Daten verloren gehen

## Disaster Recovery

- Strategien, Prozesse, Werkzeuge zur Wiederherstellung nach einem schwerwiegenden Ausfall
  - Ziel: Business Continuity
- **RTO (Recovery Time Objective):** maximal akzeptable Wiederherstellungsdauer.
- **RPO (Recovery Point Objective):** maximal akzeptabler Datenverlust.
- Disaser Recovery kommt zum Einsatz wenn High Availability / Redundanz nicht mehr reicht
  - Große Ausfälle z.B. ganz Rechenzentren, Cloud-Regionen
  - Naturkatastrophen
  - Cyberangriffe
- Nicht immer ohne Datenverlust möglich

## Backup

Ohne Backup kein Disaster Recovery!  

**3-2-1 Regel**

- 3 Kopien der Daten
- 2 verschiedene Medien
- 1 Offsite Backup

Replikation ersetzt kein Backup und umgekehrt. 

## Disaster Recovery Umsetzung

Je nach Anforderungen und Budget kann Infrastruktur wie bei Replikation auch, schon vorhanden sein -> eine Art der Active-Passive Redundanz, idR aber keine automatische Umschaltung sondern menschliches Eingreifen.
(Je nach Auslegung kann auch ein Active-Active System als Maßnahme zur Disaster Recovery eingestuft werden)

- Wichtig sind klare Prozesse, Zuständigkeiten und dokumentiertes Vorgehen
- DR Pläne müssen regelmäßig geübt werden

## Failover oder Restore?

| Situation | Erster möglicher Pfad | Zu prüfender Trade-off |
|---|---|---|
| Host ausgefallen, Daten intakt | Replica-Failover | Lag und mögliche Schreibverluste. |
| Daten versehentlich gelöscht | Point-in-time Restore | RTO und Umfang verlorener Änderungen. |
| Region verloren | Disaster-Recovery-Standby | Kosten, Umschaltung und Datenstand. |
| Schaden wird repliziert | Isoliertes Backup | Wiederherstellung und saubere Umgebung. |

Failover stellt den Dienst wieder bereit; Restore stellt einen früheren Datenstand wieder her.


# Resilienz testen und aus Vorfällen lernen

## Resilienz ist keine Behauptung

- Automatisierte Tests prüfen Fehlerfälle neben dem Erfolgsfall.
- Last-, Stress- und Soak-Tests zeigen unterschiedliche Grenzen.
- Fault Injection prüft Verhalten unter kontrollierten Störungen.
- Restore-Tests und Failover-Übungen prüfen Wiederherstellbarkeit.
- Game Days und Incident Reviews machen Abläufe lernbar.

## Lasttests und Fehler-Injektion

- **Lasttest:** erwartete Last; **Stresstest:** Last bis zur Grenze.
- **Soak-Test:** Last über längere Zeit.
- Erfolgsrate, p95-Latenz, Ressourcen, Queue-Länge und Datenkorrektheit beobachten.
- Fehler injizieren: zusätzliche Latenz, Paketverlust, Verbindungsabbruch oder knapper Pool.
- Scope und Abbruchbedingung vor jedem Experiment festlegen.

## Chaos Engineering als überprüfbares Experiment

- Hypothese: „Buchungen bleiben möglich, wenn die Kalender-API fünf Minuten ausfällt.“
- Stabilen Ausgangszustand und messbare Signale festlegen.
- Kleine Störung mit begrenztem Blast Radius injizieren.
- Verhalten beobachten und Experiment bei Gefahr abbrechen.
- Ergebnis auswerten und konkrete Verbesserung ableiten.

```plantuml
@startuml
start
:Hypothese formulieren;
:Stabilen Ausgangszustand prüfen;
:Kleine Störung injizieren;
:Signale beobachten;
if (Abbruchbedingung erreicht?) then (ja)
  :Experiment abbrechen;
else (nein)
  :Ergebnis auswerten;
endif
stop
@enduml
```

## Aus Ausfällen lernen

- Incident Response: Auswirkung bestimmen, Verantwortliche benennen und Schaden begrenzen.
- Dienst wiederherstellen; Ursachenanalyse nicht vor die Stabilisierung stellen.
- Zeitlinie, technische und organisatorische Faktoren dokumentieren.
- Maßnahmen mit Verantwortlichen nachverfolgen.
- Blameless Post-Mortems vermeiden Schuldzuweisung; „menschlicher Fehler“ ist keine ausreichende Ursachenanalyse.
- Incident Management und Observability werden in Vorlesung 6 vertieft.

## Fallbeispiel: GitHub-Ausfall im Oktober 2018

- Wartungsarbeiten unterbrachen für 43 Sekunden die Verbindung zum primären US-Ostküsten-Rechenzentrum.
- Automatisches Failover machte die US-Westküste zur neuen Primary für mehrere Datenbankcluster.
- Nach Wiederherstellung der Verbindung hatten beide Regionen nicht replizierte Writes.
- Cross-Region-Latenz und inkonsistente Topologien beeinträchtigten Anwendungen weiter.
- GitHub meldete insgesamt 24 Stunden und 11 Minuten beeinträchtigten Betrieb.

## GitHub: Wiederherstellung und Lehren

- GitHub priorisierte Datenintegrität und stoppte teilweise schreibende Hintergrundjobs.
- Restore großer MySQL-Cluster aus Backups dauerte Stunden; Replikationslag verzögerte die Rückkehr zur stabilen Topologie.
- Nach Wiederanlauf warteten über fünf Millionen Webhook-Ereignisse und 80.000 Pages-Builds.
- Maßnahmen: regionsübergreifendes Failover begrenzen, Multi-Datacenter-Redundanz ausbauen und Fehlerannahmen gezielt testen.
- Quelle: [October 21 post-incident analysis](https://github.blog/news-insights/company-news/oct21-post-incident-analysis/) (GitHub, 2018).

## Beispiel: Cal.diy bei Kalenderausfall

- Die Kalender-API wird langsam; Buchungsanfragen und Synchronisationsaufträge treffen gleichzeitig ein.
- Im Plenum durchspielen: Welche Funktion bleibt verfügbar, was wird begrenzt und was wird später nachgeholt?
- Antwortverlust, Retry, Circuit Breaker und Queue-Rückstand in den Ablauf einbeziehen.
- Schutzwirkung an Buchungserfolgsrate, Antwortzeit und Alter der ältesten Synchronisation messen.

# AI Engineering – Resilienz für KI-Komponenten

## Modell-APIs sind unzuverlässige Abhängigkeiten

- Erweiterung des Cal.diy-Lehrmodells: Eine optionale KI formuliert Terminbeschreibungen.
- Rate Limits (HTTP 429), Ausfälle, Latenzschwankungen und Modell-Deprecation einplanen.
- Inferenz hat ein eigenes Zeit- und Tokenbudget; die interaktive Buchung wartet nicht auf die Modellantwort.
- Fallback-Modell oder manuelle Eingabe nur innerhalb definierter Latenz- und Kostenbudgets nutzen.
- Kostenbudgets und Circuit Breaker begrenzen Retry-Schleifen und schützen vor Provider-Ausfällen.

## KI-Retries: technisch oder inhaltlich?

- Vorübergehende Transportfehler und HTTP 429/5xx begrenzt wiederholen; `Retry-After` beachten.
- Gemeinsame Deadline sowie Versuch-, Token- und Kostenlimit setzen; keine Retry-Schichten stapeln.
- Timeout ist mehrdeutig: Der Provider kann die Generierung bereits ausgeführt und berechnet haben.
- Derselbe Prompt-Retry garantiert kein besseres Ergebnis.
- Ungültige strukturierte Ausgabe gezielt reparieren und erneut validieren.
- Halluzination oder Verweigerung nicht blind wiederholen; manuelle Eingabe oder menschliche Prüfung anbieten.
- Tool-Aufrufe mit Seiteneffekten brauchen Idempotency Keys, damit ein Modell-Retry keine zweite Buchung auslöst.

## Fehler ohne Exception

- Falsches Format, Halluzination oder Verweigerung können ohne technische Exception auftreten.
- Strukturierte Ausgaben validieren und Ergebnisqualität als eigenes Signal behandeln.
- Wiederholung mit verändertem Prompt ist ein neuer Versuch, keine Garantie.
- Menschliche Eskalation oder regelbasierter Fallback kann geeigneter sein.

## Graceful Degradation mit KI

- Das Kernsystem muss ohne optionales KI-Feature weiterlaufen.
- Gecachte oder regelbasierte Antworten können als Fallback dienen.
- Feature Flags pro Modell und Funktion ermöglichen gezieltes Abschalten.
- Cal.diy: Terminbeschreibung manuell eingeben statt generieren; die Buchung bleibt möglich.
- Nutzer transparent informieren, wenn das KI-Feature nicht verfügbar ist.

# Zusammenfassung und Transfer

## Durchgängiges Schutzkonzept

1. Nutzerziele über SLI und SLO messbar machen.
2. Zeit- und Lastgrenzen setzen.
3. Idempotenz und begrenzte Wiederholungen nutzen.
4. Abhängigkeiten und Ressourcen isolieren.
5. Redundanz, Backup und Recovery passend zum Ziel wählen.
6. Fehlerverhalten testen und aus Vorfällen lernen.

## Takeaways

- Fehler und langsame Antworten müssen eingeplant werden.
- Kaskaden entstehen, wenn Warten und Retries Ressourcen aufbrauchen.
- Kernfunktion von optionalen Abhängigkeiten entkoppeln.
- Redundanz ersetzt kein Backup; Backup ohne Restore-Test ist unbewiesen.
- Resilienz zeigt sich im getesteten Verhalten unter Störungen.

## Nächste Vorlesung

- Architekturstile (Silas).

# Literatur und Quellen

## Literaturhinweise

- Nygard: *Release It!*, 2. Auflage — Stabilitätsmuster und Anti-Patterns.
- Beyer et al.: *Site Reliability Engineering* — SLOs, Fehlerbudgets und Überlast.
- Kleppmann, Riccomini: *Designing Data-Intensive Applications*, 2. Auflage — Replikation und verteilte Systeme.
- Google: *Building Secure and Reliable Systems* — Resilienz und Recovery.

## Frei verfügbare Quellen

- [Google SRE Book](https://sre.google/sre-book/table-of-contents/).
- [Google SRE Workbook](https://sre.google/workbook/table-of-contents/).
- [Building Secure and Reliable Systems](https://google.github.io/building-secure-and-reliable-systems/raw/toc.html).
- [Cal.com Webhooks](https://cal.com/docs/developing/guides/automation/webhooks).

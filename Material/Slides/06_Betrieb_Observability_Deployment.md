---
title: "Skalierbare Systeme - Betrieb, Observability & Deployment"
topic: "scalable_systems_1_6"
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
Time plan: 4 VE = 180 min content; 15 min break separately.
  Einstieg und Rückblick                   10
  Eine Änderung sicher ausliefern          30
  Laufzeit und Orchestrierung              35
  Risiko beim Rollout begrenzen            30
  Auswirkungen sichtbar machen             40
  Incident und Lernschleife                20
  Transfer: KI im Betrieb                  10
  Zusammenfassung                           5
  Sum                                     180
  Short tasks and plenum discussions included in the sections above.
-->

# Einstieg und Rückblick

## Heute: Eine Änderung geht live

- Cal.diy-Lehrmodell: Buchung, PostgreSQL, Outbox und Kalender-Worker
- Neu: interne Notiz zu einer Buchung
- Lokal funktioniert alles – was passiert beim Ausrollen?
- Weg: **prüfen → ausliefern → beobachten → eingreifen → lernen**

<!-- Dozent: Die Notiz und der folgende Vorfall sind fiktiv; keine Aussage über die reale Cal.diy-Implementierung. -->

## Was wir schon wissen (Vorlesungen 2 und 4)

- **V2:** Die Datenbank verhindert Doppelbuchungen; Mandantengrenzen gelten auch für neue Felder.
- **V4:** Buchung + Outbox atomar; Kalender-Sync läuft nachgelagert.
- **V4:** SLI = Anteil gültiger Buchungsabsichten mit Bestätigung innerhalb von 2 s.
- **V4:** SLO = mindestens 99,9 % in rollierenden 28 Tagen.

<!-- Selbststudium: Für SLI und Fehlerbudget die Folien „Unser Buchungs-SLI“ und „Fehlerbudget berechnen“ in Vorlesung 4 wiederholen. -->

## Lernziele

- Eine Änderung als geprüftes Artefakt bis zur Produktion verfolgen
- Rollout und Datenbankschema kompatibel gestalten
- Nutzerwirkung mit Metriken, Logs und Traces untersuchen
- Auf einen Incident reagieren und eine Verbesserung ableiten
- Zwischen Sicherheit, Tempo und Betriebskosten abwägen
- Laufzeitmodelle, GitOps und DORA-Kennzahlen einordnen

# Eine Änderung sicher ausliefern

## Problem: „Läuft bei mir“ genügt nicht

- Neue Notizspalte; mehrere App-Instanzen laufen parallel
- Die alte Version kennt die Spalte noch nicht
- Ein fehlerhafter Build könnte alle Buchungen betreffen
- Ziel: **reproduzierbar prüfen, genau das Geprüfte ausliefern**

## CI, Delivery, Deployment

- **Continuous Integration:** häufig integrieren, automatisch bauen und testen
- **Continuous Delivery:** geprüfte Version jederzeit auslieferbar halten
- **Continuous Deployment:** jede erfolgreiche Änderung automatisch produktiv setzen
- Automatik spart Wartezeit; riskante Freigaben dürfen bewusst manuell bleiben

## Delivery oder Deployment?

```text
Commit -> CI (Build + Tests) -> auslieferbares Image
                                 | manuelle Freigabe: Delivery
                                 ` automatisch live: Deployment
```

- Cal.diy: Image v2 testen, dann Canary bewusst freigeben
- Continuous Delivery heißt **nicht**: jeder Commit ist schon live

## Ein Artefakt, mehrere Umgebungen

```plantuml
@startuml
left to right direction
rectangle "Commit" as C
rectangle "Build + Tests" as T
artifact "Image v2\n(unveränderlich)" as I
rectangle "Staging" as S
rectangle "Produktion" as P
C --> T
T --> I
I --> S : deployen
I --> P : dasselbe Image deployen
S ..> P : Prüfung / Freigabe
@enduml
```

- Tests: fachliche Buchungsregel, Datenbankintegration, wichtiger Nutzerfluss
- Bei Fehler abbrechen; Version und Testergebnis nachvollziehbar halten
- Secrets erst zur Laufzeit bereitstellen – nie ins Image

<!-- Dozent: Verbindung zu Vorlesung 2: Der Integrationstest muss die DB-Regel gegen Doppelbuchungen wirklich ausführen. Safety-Scans und Reviews sind weitere Gates, aber kein Ersatz für fachliche Tests. -->

## Was testet die Pipeline – und was nicht?

| Prüfung | Fängt ab | Fängt nicht sicher ab |
|:--|:--|:--|
| Unit-Test | Fehler in Notizlogik | echte DB-Konflikte |
| Integrationstest | Migration + Buchungsregel | Produktionslast |
| Staging-Flow | Notiz speichern und lesen | reale Traffic-Verteilung |
| Canary | Probleme unter echter Last | seltene Fehler bei 100 % |

**Best Practice:** schnelle Checks zuerst, kritische Nutzerpfade vor dem Rollout.

## DevOps: Praktiken statt Jobtitel

- Kleine Änderungen, häufig integrieren: weniger auf einmal riskieren
- Reviews und automatisierte Tests vor dem Merge
- Entwicklung und Betrieb teilen SLO, On-Call und Rückweg
- Telemetrie und Postmortems fließen in die nächste Änderung
- Tradeoff: mehr Automatisierung braucht Pflege und klare Ownership

## Aufgabe: Wo gehört die Prüfung hin? (5 Minuten)

- Alte und neue App greifen gleichzeitig auf Buchungen zu.
- Ein doppelter Slot darf niemals bestätigt werden.
- Eine neue Notiz darf nicht bei fremden Mandanten auftauchen.
- Zu zweit: Welche Prüfungen vor Produktion? Was erst im Canary sichtbar?

<!-- Dozent: Im Plenum DB-Constraint/Integrationstest, mandantenübergreifenden Zugriffstest und Signal für reale Last sammeln. Tests und Monitoring ergänzen sich. -->

## Migration: alte und neue Version gleichzeitig

- `internal_note` zunächst **nullable** ergänzen: alte App bleibt lauffähig
- v2 deployen; Notiz nur für berechtigte Nutzer freigeben
- Für die optionale Notiz: kein Backfill und kein Entfernen alter Felder nötig
- Code-Rollback auf v1 bleibt möglich; die neue Spalte bleibt bestehen

<!-- Dozent: Bei einem späteren Spaltenersatz wären Backfill und Contract nötig. Alte Felder erst entfernen, wenn weder laufende noch als Rückweg vorgesehene Versionen sie lesen oder schreiben. -->

## GitOps: gewünschter Zustand in Git

```text
CI baut Image -> Git: v2-Digest -> Controller -> Cluster
                          ^                    |
                          +---- Abgleich ------+
```

- Review der Konfiguration; Controller erkennt Abweichungen (Drift)
- Kein CI-Zugriff auf produktive Cluster nötig; Freigabe vor Git-Änderung
- Preis: weiterer Controller, Git-Konflikte und verzögerte Synchronisation

<!-- Dozent: GitOps ist eine Variante von CD, keine Pflicht. Beispiel Argo CD; Freigaben und Canary-Gates bleiben nötig. -->

## Deployment ist nicht Release

- **Deployment:** v2 läuft technisch; **Release:** Notiz ist sichtbar
- Feature Flag: Notiz zunächst nur für ein internes Team
- Abschalten geht schnell – bereits geschriebene Daten bleiben
- Preis: zusätzliche Zustände; Flags mit Besitzer und Löschtermin

# Laufzeit und Orchestrierung

## Welche Laufzeit für Cal.diy?

| Modell | Plus | Minus |
|:--|:--|:--|
| Server (Bare Metal) | volle Kontrolle | Hardware selbst betreiben |
| VM | Isolation, eigener OS-Stack | OS und Updates pflegen |
| Container | reproduzierbares Image | Host und Orchestrierung nötig |
| Serverless (FaaS) | Betrieb/Skalierung delegiert | Laufzeit- und Plattformgrenzen |

- Mehr Abstraktion vereinfacht Bereitstellung; weniger Kontrolle über den Unterbau
- **Nicht linear:** Container auf eigenen Servern geben mehr Kontrolle als auf PaaS

## Serverless: Wer betreibt die Server?

- Server bleiben – der Anbieter verwaltet sie; wir liefern Code und Konfiguration
- Hier: **Functions as a Service (FaaS)**, ereignisgesteuert und zustandslos
- Bedarfsgerechte Skalierung und nutzungsabhängige Abrechnung
- Risiken: Kaltstart, Limits, Parallelität, Anbieterbindung
- Dauerbetrieb mit gleichmäßiger Last kann anderswo günstiger sein

<!-- Dozent: Serverless umfasst auch Managed Container/Services; die folgenden Grenzen beziehen sich speziell auf FaaS. Kaltstarts und Limits variieren je nach Anbieter und Konfiguration. -->

## Cal.diy: Wo passt eine Function?

- Kalender-Sync nach dem Outbox-Commit: Worker kann Events an FaaS übergeben
- FaaS-Verarbeitung bleibt idempotent; Wiederholungen und Duplikate erwarten (V4)
- Buchungs-API braucht niedrige p95-Latenz: Kaltstarts vorher messen
- Viele parallele Functions können PostgreSQL-Verbindungen erschöpfen
- Grenze setzen, Pooling prüfen; kein Serverless nur um „modern“ zu sein

## Serverless: Skalierung gegen die Datenbank

- Beispiel: 40 Events/s × 2 s Laufzeit: etwa 80 gleichzeitige Functions
- Je zwei DB-Verbindungen → bis zu 160; DB-Limit: 100
- Function-Skalierung drosseln oder Verbindungspool/Queue nutzen
- Warteschlange wächst dann: Alter der Aufträge beobachten (V4)
- Tradeoff: Kosten für Leerlauf sparen, aber Sättigung verlagern

<!-- Dozent: Vereinfachte Mittelwerte für stationäre Last, keine Kapazitätsgarantie. Idempotenz und erneute Zustellung auch bei Drosselung beachten. -->

## Cal.diy: Wenn Functions schneller skalieren als die Datenbank

![](media/06_serverless_db_limit.pdf){height=82%}

## Wann lohnt sich FaaS?

| Lastprofil | Tendenz | Grund |
|:--|:--|:--|
| Seltene Kalender-Events | FaaS | keine dauerhaft laufende Worker-Instanz |
| Kurze, starke Spitzen | FaaS prüfen | skaliert schnell; Limits und DB schützen |
| Dauerhaft hohe Last | Container prüfen | nutzungsabhängige Kosten summieren sich |
| Strenge API-Latenz | messen | Kaltstart kann p95 verschlechtern |

## Die Rechnung endet nicht beim Aufrufpreis

```text
FaaS: Aufrufe + Laufzeit × Speicher + Netz/Logs + ggf. Warmhaltung
Container: laufende Kapazität + Netz/Logs + Betriebsaufwand
```

- Bei Leerlauf spart FaaS oft Kapazitätskosten; bei Dauerlast vergleichen
- Warmgehaltene Functions senken Kaltstarts, kosten aber auch ohne Aufrufe
- Kosten **und** p95, DB-Verbindungen, Limits unter realer Last messen
- Cal.diy: Worker eher geeignet als die synchrone Buchungs-API

<!-- Dozent: Keine universelle Kostenschwelle; Preise und Freikontingente hängen vom Anbieter ab. Die Formeln vereinfachen u. a. Datenbank- und externe Dienste-Kosten. -->

## Serverless: Das Lastprofil entscheidet

![](media/06_serverless_lastprofil.pdf){height=82%}

## Warum Container orchestrieren?

- Eine App-Instanz fällt aus: Wer startet sie neu?
- Lastspitze: Wer verteilt Traffic und startet weitere Instanzen?
- v1 und v2 gleichzeitig: Wer steuert Update und Rückweg?
- Orchestrator löst diese Betriebsfragen, nicht die DB-Konsistenz

## Kubernetes: die Bausteine

```plantuml
@startuml
left to right direction
actor Gast
rectangle "Service\n(stabile Adresse)" as S
node "Deployment: Buchungs-API" {
  rectangle "Pod v1" as A
  rectangle "Pod v2" as B
}
database "PostgreSQL\n(extern)" as DB
Gast --> S
S --> A
S --> B
A --> DB
B --> DB
@enduml
```

- Deployment regelt gewünschte Pod-Anzahl und Rolling Updates
- Service verteilt Anfragen; Readiness bestimmt, wer Traffic erhält

## Kubernetes: Möglichkeiten und Grenzen

- Deployment: Rollouts und Selbstheilung; HPA: Pods nach Last skalieren
- Job/CronJob: einmalige oder geplante Aufgaben, z. B. Datenpflege
- ConfigMaps/Secrets: Konfiguration trennen; Zugriff absichern
- Namespace/Quota: Teams und Ressourcen begrenzen, keine harte Sicherheitsgrenze
- Mehr Pods können den DB-Pool überlasten; Datenbank separat betreiben
- Alternative: Managed Container/PaaS bei weniger Betriebsaufwand

<!-- Dozent: Pod = kleinste geplante Einheit aus einem oder mehreren Containern. Bei K8s-Service allein keine exakten Traffic-Anteile für Canarys garantiert; dafür zusätzliche Routing-Funktion nötig. -->

## Kubernetes: gewünschter Zustand und Rolling Update

```text
Deployment: 4 Pods mit Image v1 -> Image v2 eintragen
Controller: neuen Pod starten -> Readiness prüfen -> alten entfernen
```

- `maxSurge: 1`: höchstens ein zusätzlicher Pod beim Update
- `maxUnavailable: 0`: vier verfügbare Pods erhalten, wenn Kapazität reicht
- Hängt v2 in Readiness, stoppt der Fortschritt; nicht blind weiterrollen
- Service verteilt auf bereite Pods – **keine exakte 10-%-Canary-Steuerung**

<!-- Dozent: Für einen gezielten Canary getrennte Deployments plus geeignetes Routing einsetzen. Rollback des Deployments dreht DB-Änderungen nicht zurück. -->

## Kubernetes: Rolling Update im Ablauf

![](media/06_kubernetes_rollout.pdf){height=82%}

## Kubernetes: Skalierung hat eine Grenze

- HPA passt die Pod-Zahl anhand von CPU oder anderen Metriken an
- Beispiel: 2 → 6 API-Pods mit je 20 DB-Verbindungen = bis zu 120
- Hat die Datenbank nur 100 Plätze, steigt die Wartezeit trotz mehr Pods
- `maxReplicas` und Poolgrößen begrenzen; DB-Signale mitbeobachten
- HPA ersetzt weder Kapazitätsplanung noch Canary-Go/Stop-Gate

<!-- Dozent: Bei CPU-Auslastung als HPA-Metrik passende Resource Requests setzen. Rechnungen sind illustrative Obergrenzen, keine Messdaten. -->

# Risiko beim Rollout begrenzen

## Problem: v2 ist getestet – aber wie viel Risiko?

- Produktivlast und Staging unterscheiden sich
- Rollout betrifft gleichzeitig API, Datenbank und Worker
- Ein schneller Rückweg zählt mehr als ein schönes Dashboard
- Vorab festlegen: **Was stoppt den Rollout? Wer entscheidet?**

## Strategien im Vergleich

| Strategie | Vorteil | Nachteil |
|:--|:--|:--|
| Recreate | einfach | Unterbrechung |
| Rolling | wenig Zusatzkapazität | Versionen überlappen |
| Blue/Green | schneller Traffic-Wechsel | doppelte Umgebung; DB bleibt geteilt |
| Canary | kleiner Auswirkungsbereich | Traffic-Routing und Messung nötig |

**Für unsere Änderung:** Canary, falls Traffic und Versionen getrennt messbar sind; sonst Rolling mit schnellem Stopp.

## Canary: erst messen, dann erweitern

```text
10 % v2 -> messen -> Go/Stop -> 50 % -> erneut prüfen -> 100 %
                         |
                      Stop: zurück zu v1
```

- Wenige Nutzer zuerst: kleiner Auswirkungsbereich, aber wenig Messdaten
- v1 und v2 getrennt und bei vergleichbarem Traffic beobachten
- Nicht „grün“, nur weil die Gesamtfehlerquote stabil bleibt

<!-- Dozent: Kleine Canarys übersehen seltene Fehler. Bei wenig Traffic länger beobachten; v2 kann einen anderen Nutzermix erhalten als v1. -->

## Canary: Versionen getrennt betrachten

![](media/06_canary_gate.pdf){height=82%}

## Go/Stop-Gate für Cal.diy

- **Vorab:** 15 Minuten und mindestens 200 Buchungsabsichten je Version
- **Stop:** p95 von v2 > 1,5 × v1 oder Pool-Wartezeit p95 > 200 ms
- **Go:** Grenzen eingehalten, Buchungs-SLI stabil; sonst pausieren
- **Verantwortung:** Release-Team entscheidet; bei zu wenig Traffic weiter warten

<!-- Dozent: Beispielwerte, keine universellen Grenzwerte. Buchungs-SLI und DB-Fehler je Version prüfen; wenige hundert Ereignisse beweisen kein 99,9-%-SLO. Die eindeutige Buchungsabsicht aus V4 zählt nur einmal, auch bei technischen Retries. Bei akutem Buchungsausfall unabhängig vom Messfenster stoppen. -->

## Health Checks sind nicht das SLO

- **Liveness:** hängt der Prozess? Neustart nur dann sinnvoll
- **Readiness:** kann diese Instanz jetzt Traffic annehmen?
- „HTTP 200“ sagt nichts über erfolgreiche Buchungen
- Falsche Readiness kann alle Instanzen gleichzeitig aus dem Traffic nehmen

<!-- Bezug Vorlesung 4: Health Checks helfen beim Failover; das Buchungs-SLI misst Nutzerwirkung. -->

## Aufgabe: Rollout-Entscheidung (6 Minuten)

- v2 erhält 10 % Traffic; p95 ist doppelt so hoch wie bei v1.
- Gesamtfehlerquote bleibt fast unverändert.
- v2 belegt doppelt so viele DB-Verbindungen.
- In Dreiergruppen: weiterrollen, pausieren oder zurück? Welche zwei Signale fehlen?

<!-- Dozent: Erwartung: pausieren, nicht auf 50 % gehen; Buchungs-SLI nach Version und DB-Pool-Wartezeit/Queries prüfen. Rollback nur bei kompatiblem Schema und sicherem Rückweg. -->

## Rückweg: Code und Daten sind verschieden

- Bei Schaden: Canary stoppen, v2-Traffic zurück auf v1
- Nullable Spalte bleibt vorerst bestehen; nicht blind „zurückmigrieren“
- Falls v2 schon neue Daten geschrieben hat: Lesbarkeit mit v1 prüfen
- Roll-forward statt Rollback, wenn neue Daten nicht rückwärtskompatibel sind

# Auswirkungen sichtbar machen

## Problem: „Langsam“ ist noch keine Diagnose

- Nutzende melden träge Buchungen nach v2
- Datenbank erreichbar; CPU der API unauffällig
- Welche Nutzer? Welche Version? Wo wartet der Request?
- **Monitoring** meldet bekannte Symptome; **Observability** hilft bei der Ursachenfrage

## Drei Signale – drei Fragen

| Signal | Frage zum Buchungsrequest | Grenze |
|:--|:--|:--|
| Metrik | Wie viele sind langsam? | kein einzelner Ablauf |
| Log | Was geschah bei Buchung X? | kein Gesamtbild |
| Trace | Wo wartete dieser Request? | nur erfasste Requests |

**Best Practice:** Version und Trace-ID korrelieren; Signale nicht gegeneinander ausspielen.

## OpenTelemetry (OTel): Signale transportieren

- SDK instrumentiert Cal.diy: Spans, Metriken, ggf. Logs
- Trace-Kontext verbindet API, Outbox und Worker
- Collector empfängt, filtert und verteilt Telemetrie (OTLP)
- **OTel ist kein Dashboard und keine Datenbank**: Backend zusätzlich nötig
- Sampling und Datenschutz vor dem Export festlegen

## Möglicher Betriebsstack für Cal.diy

```text
Cal.diy --OTel--> Collector -- Metriken --> Prometheus --+
                         |-- Traces ----> Tempo --------+--> Grafana
                         `-- Logs ------> Loki ---------+
```

- Selbst betrieben: flexible Abfragen, aber Storage, Betrieb und Kosten
- Managed Observability: weniger Betrieb, dafür Preis und Datenabfluss
- Für die Lehre reicht die Architektur; kein Live-Stack nötig

<!-- Dozent: Prometheus kann Metriken vom Collector abholen; die Skizze abstrahiert die konkrete Export-/Scrape-Konfiguration. Tempo und Loki sind Beispiele, keine Pflicht. -->

## Betriebs- und Nutzerperspektive: zwei Wege

![](media/06_observability_wege.pdf){height=82%}

## PostHog: Nutzerperspektive als SaaS

- Produkt-Events: Wo bricht der Buchungsfluss ab?
- Session Replay + Error Tracking: Was sah der Gast beim Fehler?
- OTLP-Logs sind möglich; sensible Felder vorher minimieren
- Nicht mit vollständigem Infrastruktur-Monitoring gleichsetzen
- Ergänzung zu OTel/Grafana-Stack, kein Ersatz für DB-Pool-Metriken

<!-- Dozent: SaaS spart Betrieb, erzeugt laufende Kosten und Datenschutzfragen; bei Replays Einwilligung/Maskierung und Mandantengrenzen prüfen. PostHog-Funktionen und Preise können sich ändern. -->

## Metriken: Nutzerwirkung zuerst

- **Rate:** gültige Buchungsabsichten pro Minute
- **Errors:** schlechte Ereignisse gemäß Buchungs-SLI aus Vorlesung 4
- **Duration:** Bestätigungszeit, z. B. p95 statt nur Durchschnitt
- Ergänzend: Pool-Wartezeit und Alter der Kalender-Outbox

<!-- Dozent: Konflikt um einen wirklich belegten Slot ist nicht pauschal ein Serverfehler. Den SLI-Nenner aus Vorlesung 4 beibehalten; technische Retries nicht doppelt zählen. RED = Rate, Errors, Duration. -->

## p95 statt nur Durchschnitt

```text
90 Buchungen:  0,1 s
10 Buchungen:  5,0 s
Durchschnitt:  0,59 s
p95:           5,0 s
```

- p95: mindestens 95 % der Messwerte liegen bei oder unter diesem Wert
- Der Mittelwert versteckt die zehn stark betroffenen Anfragen
- Perzentile immer mit Zeitraum, Traffic und Version lesen

## Logs: Kontext, nicht Datenmüll

```text
level=warn event=booking.slow version=v2 trace_id=abc123 db_wait_ms=870
```

- Strukturierte Felder statt unfilterbarem Freitext
- Trace-ID durch API und Worker weiterreichen
- Keine Secrets, Kalenderinhalte oder unnötigen Personendaten loggen
- Mandantenbezug nur soweit nötig und zugriffsgeschützt

<!-- Bezug Vorlesung 2: Audit-Log für sicherheitsrelevante Aktionen ist nicht dasselbe wie Diagnose-Log. Auch Trace-Attribute können personenbezogene Daten preisgeben. -->

## Trace: Wo geht die Zeit hin?

```text
POST /bookings                 1.400 ms
|-- authorize                    12 ms
|-- wait for DB connection      870 ms
`-- DB: Buchung + Outbox         110 ms
```

- Span = ein Schritt; Trace = verbundene Schritte eines Requests
- Verdacht: Pool-Sättigung, nicht langsame Kalender-API
- Trace-Sampling spart Kosten; seltene Fehler dürfen nicht unsichtbar werden

<!-- Dozent: Die übrigen 408 ms sind hier nicht aufgeschlüsselt (z. B. Netzwerk und Anwendungscode); sichtbare Spans decken nicht zwangsläufig den ganzen Request ab. Die Outbox wird in derselben DB-Transaktion wie die Buchung gespeichert (V4), der Worker synchronisiert später. Hypothese mit Pool-Metriken prüfen, nicht aus einem Trace allein verallgemeinern. -->

## Trace: Die Wartezeit wird sichtbar

![](media/06_trace_waterfall.pdf){height=82%}

## Vom SLI zum handlungsfähigen Alert

- V4-SLO: 99,9 % gute Buchungsabsichten in 28 Tagen
- Bei 50.000 Absichten: höchstens 50 schlecht; 40 verbraucht → 10 übrig
- Alert bei schnellem Budgetverbrauch oder akutem Buchungsausfall
- Ziel: Nutzerwirkung + zuständiges Team + sicherer erster Schritt

<!-- Dozent: Kurze Canary-Gates können zusätzliche, versionsbezogene Grenzwerte nutzen; ein 28-Tage-SLO allein reagiert zu langsam auf ein neues Deployment. Kurzzeitige CPU-Spitzen ohne Nutzerwirkung gehören ins Dashboard, nicht in den Pager. -->

## Beispiel: Observability-Konzept für Cal.diy

- **Frage:** Buchen Gäste seit v2 seltener erfolgreich?
- **Dashboard:** SLI und p95 nach Version; DB-Pool-Wartezeit
- **Drill-down:** Trace-ID → langsamer DB-Span → Log zum Pool
- **Aktion:** Canary pausieren; Pool-Limit und Worker-Verbindungen prüfen

<!-- Dozent: Im Plenum erarbeiten, welches Signal zuerst den Nutzerfehler zeigt und welches die Ursache prüft. Bei Mandantenvergleich auf Datenschutz und geringe Fallzahlen achten. -->

## Aufgabe: Ein guter Alarm? (4 Minuten)

- A: „CPU für 30 Sekunden über 80 %“
- B: „Buchungs-SLI von v2 fällt; Canary stoppen, Runbook öffnen“
- Welche Meldung braucht Bereitschaft? Welche Info fehlt noch?

<!-- Dozent: B priorisieren; Schwelle, Beobachtungsdauer, Mindest-Traffic und Zuständigkeit vorab konkretisieren. A als Diagnosemetrik behalten. -->

# Incident und Lernschleife

## v2 verschlechtert Buchungen: jetzt handeln

```text
Erkennen → Koordinieren → Eingrenzen → Wiederherstellen → Lernen
```

- Eine Person koordiniert, eine untersucht; Wirkung und Zeiten notieren
- Canary pausieren, v2-Traffic zurücknehmen, Buchungs-SLI beobachten
- Stakeholder über Auswirkung und nächsten Stand informieren
- Ursachenanalyse nach der Stabilisierung vertiefen (Bezug Vorlesung 4)

## Runbook: nicht erst im Incident schreiben

- Symptom: SLI sinkt nach Rollout; v2-Pool-Wartezeit steigt
- Erste Checks: Version, Traffic-Anteil, Pool, DB-Fehler
- Sofortmaßnahme: Traffic zu v1, wenn Daten kompatibel bleiben
- Eskalation: API- und Datenbankverantwortliche; Link zum Dashboard

## Postmortem: Ursache statt Schuld

- **Befund:** v2-API öffnet zu viele Verbindungen; v1 bleibt gesund
- **Bedingung:** Testlast und Pool-Konfiguration bildeten Produktion nicht ab
- **Maßnahme:** Pool-Limit + Lasttest + Canary-Gate für Pool-Wartezeit
- **Nachweis:** Rollout erneut mit begrenztem Canary testen

<!-- Fiktiver Befund, keine Aussage über Cal.diy. In V4 erschöpft der Kalender-Worker den Pool; hier ist die neue API-Version der Auslöser. Bezug V4: blameless, Maßnahme mit Verantwortlichen und Termin festhalten. -->

## Wer trägt den Betrieb?

- DevOps: Entwicklung und Betrieb teilen Verantwortung für den Nutzerfluss
- SRE: SLOs, Fehlerbudget und weniger manuelle Routine
- On-Call braucht klare Alerts, Zugriff und getestete Runbooks
- Mehr Automatisierung spart Arbeit – falsch konfigurierte Gates blockieren Releases

## DORA: Wie gut liefern wir Änderungen aus?

- **Change Lead Time:** Commit bis produktiv; lange Wartezeit finden
- **Deployment Frequency:** wie oft geht eine Version produktiv?
- **Failed Deployment Recovery Time:** wie schnell nach einem fehlgeschlagenen Deployment wiederhergestellt?
- Cal.diy: Warten v2-Änderungen auf Freigabe oder auf Tests?

## DORA: Tempo ohne Stabilität ist kein Erfolg

- **Change Fail Rate:** Anteil Deployments mit nötigem Soforteingriff
- **Deployment Rework Rate:** Anteil ungeplanter Folge-Deployments nach Incident
- Beispiel: 20 Deployments, 2 Soforteingriffe → 10 % Change Fail Rate
- Alle fünf gemeinsam pro Dienst über Zeit betrachten – kein Personen-Ranking

<!-- Dozent: DORA 2026 zählt fünf Kennzahlen; Failed Deployment Recovery Time ist nicht die MTTR aller Incidents aus V4. Postmortem- und CI-Daten können helfen, die Ursachen hinter den Zahlen zu verstehen. -->

# Transfer: KI im Betrieb

## Optionale KI-Beschreibung: anderer Erfolgsbegriff

- Aus V4: KI schlägt Terminbeschreibung vor; Buchung bleibt unabhängig
- Technisch erfolgreich ist nicht gleich inhaltlich richtig
- Signale: Latenz, Tokens/Kosten, Verweigerungen, Nutzerkorrekturen
- Keine Prompts mit Personendaten ungeschützt in Logs oder Traces

## Modellwechsel: wie sicher ausrollen?

- Prompt und Modellversion zusammen mit Ergebnissen nachvollziehen
- Vorher: kleine, geprüfte Beispiele gegen Qualitätsregression testen
- Nachher: Canary + Nutzerfeedback; bei Schaden KI-Flag abschalten
- Shadow Mode testet ohne Nutzerwirkung, kostet aber zusätzliche Aufrufe und Datenzugriffe

<!-- Dozent: Automatische Evals sind ein Gate, kein Beweis für fachliche Wahrheit; manuelle Prüfung bei riskanten Ausgaben. Bei fremden Modell-APIs Datenschutz und Kosten vor Shadow Traffic prüfen. -->

# Zusammenfassung

## Der Kreis schließt sich

```text
Änderung -> Pipeline -> Rollout -> Nutzer-Signale
    ^                              |
    +----- Postmortem <- Incident --+
```

- Getestetes Artefakt, kompatible Daten, kleiner Auswirkungsbereich
- Erst Nutzerwirkung messen, dann mit Logs und Traces erklären
- Abstraktion spart Eigenbetrieb, ersetzt weder SLO noch Datenregeln
- Tradeoff: Tempo, Kontrolle, Kosten und Zuverlässigkeit

## Nächste Vorlesung

- Vorlesung 7: Wiederholung und Klausurvorbereitung
- Leitfrage: Welche Architekturentscheidung würden wir nach dem Incident ändern?

# Literatur und Quellen

## Zum Nachlesen

- Beyer et al.: [Site Reliability Engineering](https://sre.google/sre-book/table-of-contents/) – Release Engineering, Monitoring, Incidents
- Google: [SRE Workbook](https://sre.google/workbook/table-of-contents/) – SLOs und Alerting
- Humble, Farley: _Continuous Delivery_ – Deployment Pipeline und Datenmigrationen
- Majors et al.: _Observability Engineering_ – Logs, Metriken und Traces
- [OpenTelemetry](https://opentelemetry.io/docs/what-is-opentelemetry/) – Instrumentierung und Collector
- [PostHog Docs](https://posthog.com/docs/product-analytics) – Analytics, Replay, Error Tracking; [OTLP-Logs](https://posthog.com/docs/logs)
- [DORA Metrics](https://dora.dev/guides/dora-metrics/) – fünf Lieferkennzahlen
- [Kubernetes Docs](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) und [Argo CD](https://argo-cd.readthedocs.io/en/stable/user-guide/ci_automation/) – Orchestrierung und GitOps
- [AWS Lambda Docs](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtime-environment.html) und [Pricing](https://aws.amazon.com/lambda/pricing/) – Kaltstarts und Kosten als FaaS-Beispiel

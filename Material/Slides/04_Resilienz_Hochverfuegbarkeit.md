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

<!--
Time plan: 4 VE = 180 min content, Friday 8:30-11:45 with a 15 min break on top.
  Heute, Lernziele                         5
  Grundlagen                              25   4 slides
  Redundanz und Hochverfügbarkeit         25   4 slides
  Resilienz-Muster                        50   8 slides
  Resilienz testen                        35   2 slides + post-mortem case ~10 + example ~12
  AI Engineering                          15
  Zusammenfassung                          5
  Sum                                    160   20 min slack
-->

# Grundlagen

## Heute

- Begriffe: Fehler, Ausfall, Verfügbarkeit, Resilienz
- Warum verteilte Systeme scheitern
- Redundanz und Hochverfügbarkeit
- Resilienz-Muster
- Resilienz testen
- AI Engineering: Resilienz für KI-Komponenten

## Lernziele

- Verfügbarkeit definieren, messen und über Fehlerbudgets steuern
- Typische Ausfallarten verteilter Systeme erkennen
- Redundanz- und Failover-Strategien für zustandslose und zustandsbehaftete Komponenten entwerfen
- Resilienz-Muster gezielt einsetzen und ihre Nebenwirkungen kennen

## Begriffe

- Fault, Error, Failure: Ursache, Zustand, Wirkung
- Zuverlässigkeit: funktioniert korrekt, auch wenn etwas schiefgeht
- Verfügbarkeit: ist erreichbar, wenn gebraucht
- Resilienz: erholt sich von Störungen
- Robustheit: hält Störungen aus, ohne sich zu verändern

## Verfügbarkeit messen

- Die Neunen: 99 %, 99,9 %, 99,99 % und was sie im Jahr bedeuten
- MTBF und MTTR: Ausfälle vermeiden vs. schnell beheben
- SLI, SLO, SLA: messen, zielen, versprechen
- Fehlerbudget: Wie viel Ausfall dürfen wir uns leisten?
- Verfügbarkeit verketteter Komponenten: Multiplikation der Wahrscheinlichkeiten

## Warum verteilte Systeme scheitern

- Das Netzwerk ist unzuverlässig: Partitionen, Latenz, Paketverlust
- Uhren gehen falsch, Zeit ist nicht global
- Teilausfälle: Ein Knoten weiß nicht, ob der andere tot oder nur langsam ist
- Fallacies of Distributed Computing (Peter Deutsch)

## Ausfallarten

- Crash, Omission, Timing, Byzantine
- Korrelierte Ausfälle: gleiche Zone, gleiches Deployment, gleicher Bug
- Kaskadierende Ausfälle: Ein Fehler zieht den nächsten nach
- Metastabile Ausfälle: Das System erholt sich nicht von selbst

# Redundanz und Hochverfügbarkeit

## Redundanz auf allen Ebenen

- Instanzen, Verfügbarkeitszonen, Regionen
- Active-Active vs. Active-Passive
- Redundanz kostet: Geld, Komplexität, Konsistenzprobleme
- Wo ist der Single Point of Failure, den wir übersehen haben?

## Failover und Konsens

- Ausfall erkennen: Heartbeats, Timeouts, die Grauzone
- Leader Election, Split Brain, Quorum
- Konsensprotokolle in einem Satz: Raft, Paxos
- Wer benutzt das: etcd, ZooKeeper, Kubernetes, Datenbank-Cluster

## Zustandslos vs. zustandsbehaftet

- Zustandslose Services: replizieren und Load Balancer davor
- Datenbanken: Replikation, automatisches Failover, Backups (Bezug Vorlesung 2)
- RPO und RTO: Wie viel Datenverlust und wie lange Ausfall sind akzeptabel?
- Zustand in Queues und Caches nicht vergessen

## Disaster Recovery

- Backup-Strategien: 3-2-1, Restore regelmäßig testen
- Multi-Region: Active-Active, Warm Standby, Pilot Light, Backup-and-Restore
- Kosten gegen Nutzen: Was kostet eine Stunde Ausfall?
- Runbooks für den Ernstfall

# Resilienz-Muster

## Timeouts und Retries

- Jeder Netzwerkaufruf braucht einen Timeout
- Retries mit Exponential Backoff und Jitter
- Retry Storms und Thundering Herd: Wie Retries Ausfälle verschlimmern
- Retry-Budgets

## Idempotenz

- Voraussetzung für sichere Wiederholungen
- Idempotency Keys für Schreiboperationen
- Natürlich idempotente Operationen vs. künstlich idempotent gemachte
- Bezug zu Messaging: At-least-once erfordert Idempotenz

## Circuit Breaker und Bulkhead

- Circuit Breaker: Zustände, Schwellwerte, halb offen
- Bulkhead: Ressourcen isolieren, damit ein Fehler nicht alles mitnimmt
- Beispiel: Thread-Pools oder Connection-Pools pro Abhängigkeit
- Bibliotheken und Service Meshes, die das übernehmen

## Last begrenzen

- Rate Limiting: Token Bucket, Leaky Bucket; pro Client, pro Mandant
- Load Shedding: lieber einige ablehnen als alle verlieren
- Backpressure: Druck zurück zum Erzeuger statt Puffer bis zum Absturz
- Prioritäten: Welche Anfragen sind wichtig?

## Graceful Degradation

- Fallbacks: Cache, Default, reduzierte Funktion
- Feature Flags und Kill Switches
- Kernfunktion schützen, Komfortfunktion opfern
- Beispiel: Shop ohne Empfehlungen ist besser als kein Shop

## Health Checks und Selbstheilung

- Liveness vs. Readiness
- Orchestrierung: Neustart, Ersatz, Verschieben (Vertiefung in Vorlesung 6)
- Gefahr: Health Checks, die selbst Last erzeugen oder kaskadieren
- Startup-Verhalten: langsam anfahren, Warm-up

## Entkopplung über Queues

- Queue als Puffer zwischen Produzent und Konsument
- Dead Letter Queues für Nachrichten, die nicht verarbeitbar sind
- Outbox-Pattern: Datenbankänderung und Nachricht atomar
- Nebenwirkungen: Latenz, Reihenfolge, Duplikate

## Anti-Patterns

- Unendliche Retries ohne Backoff
- Synchrone Ketten über viele Services
- Gemeinsame Ressourcen ohne Bulkhead
- Fehlende Timeouts, Standard-Timeouts von Bibliotheken
- Tests nur für den Erfolgsfall

# Resilienz testen

## Resilienz testen

- Chaos Engineering: Hypothese, Experiment, kleiner Blast Radius
- Fault Injection: Latenz, Fehler, Ausfall einzelner Instanzen
- Game Days: Ausfälle üben, Runbooks prüfen
- Lasttests: Wo bricht das System, und wie?

## Aus Ausfällen lernen

- Blameless Post-Mortems: Zeitachse, Ursachen, Maßnahmen
- Ursachen sind selten eine einzelne: soziotechnische Sicht
- Incident Management als Prozess (Vertiefung in Vorlesung 6)
- Öffentliche Post-Mortems als Lernmaterial

## Fallbeispiel: ein öffentliches Post-Mortem

- Ein realer Großausfall Schritt für Schritt: Auslöser, Verstärker, Erkennung, Behebung
- Welche Muster aus dieser Vorlesung hätten geholfen, welche haben geschadet?
- Was hat die Organisation danach geändert?

<!-- TODO(Lu): pick one public post-mortem (e.g. AWS us-east-1, Cloudflare, GitHub 2018) -->

## Beispiel: Was passiert, wenn ...?

- Beispielsystem aus Vorlesung 1 oder 3
- Ausfallszenarien gemeinsam durchspielen: Datenbank weg, Cache weg, Abhängigkeit langsam, Region weg
- Was passiert heute, was sollte passieren, welche Muster helfen?

# AI Engineering: Resilienz für KI-Komponenten

## Die Modell-API als unzuverlässige Abhängigkeit

- Rate Limits (HTTP 429), Ausfälle, Latenzschwankungen, Modell-Deprecation
- Timeouts für Anfragen, die legitim lange dauern
- Fallback-Modelle und Multi-Provider-Routing
- Kostenbudgets als Sicherung gegen Retry-Schleifen

## Nicht-deterministische Fehler

- Falsches Format, Halluzination, Verweigerung: Fehler ohne Exception
- Ausgabevalidierung, strukturierte Ausgaben, Guardrails
- Retry mit verändertem Prompt statt identischer Wiederholung
- Menschliche Eskalation als Fallback

## Graceful Degradation mit KI

- Das System muss ohne KI-Feature weiterlaufen
- Gecachte oder regelbasierte Antworten als Fallback
- Feature Flags pro Modell und Funktion
- Was der Nutzer erfahren soll, wenn die KI nicht verfügbar ist

# Zusammenfassung

## Zusammenfassung

- Ausfälle sind normal; das Design bestimmt, ob sie Nutzer treffen
- Verfügbarkeit ist messbar und hat einen Preis, das Fehlerbudget macht ihn verhandelbar
- Redundanz braucht Failover, Failover braucht Konsens oder klare Regeln
- Timeouts, Retries mit Backoff, Idempotenz, Circuit Breaker, Bulkheads, Load Shedding
- Modell-APIs sind Abhängigkeiten wie jede andere, nur mit neuen Fehlerarten

## Nächste Vorlesung

- Architekturstile (Silas)

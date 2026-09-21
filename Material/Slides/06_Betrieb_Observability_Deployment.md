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
Time plan: 4 VE = 180 min content, Friday 8:30-11:45 with a 15 min break on top.
  Heute, Lernziele                         5
  Betrieb als Teil der Architektur        15   2 slides
  Deployment                              45   7 slides
  Observability                           60   7 slides + demo ~15 + example ~15
  AI Engineering                          20   4 slides
  Zusammenfassung                          5
  Sum                                    150   30 min slack for the demo
-->

# Betrieb als Teil der Architektur

## Heute

- Betrieb als Architekturthema: DevOps, SRE, Plattformen, Kosten
- Deployment: Orchestrierung, Infrastructure as Code, Strategien, Pipelines
- Observability: Logs, Metriken, Traces, Alerting, Demo
- Incident Management
- AI Engineering: Betrieb von KI-Systemen
- Beispiel: Observability-Konzept

## Lernziele

- Deployment-Strategien vergleichen und für einen Fall auswählen
- Autoscaling und Kostensteuerung als Architekturentscheidung verstehen
- Ein Observability-Konzept mit Metriken, Alerts und SLOs entwerfen
- Incidents strukturiert behandeln und daraus lernen

## Betrieb ist kein Nachgedanke

- Betreibbarkeit als Qualitätsattribut: Deployability, Beobachtbarkeit, Konfigurierbarkeit
- DevOps: "You build it, you run it"
- Site Reliability Engineering: Toil reduzieren, Fehlerbudget, On-Call
- Platform Engineering: interne Plattform als Produkt für Entwicklungsteams

## Kosten im Betrieb

- Cloud-Kosten verstehen: Compute, Speicher, Netzwerk (Egress!), Managed Services
- FinOps: Kosten sichtbar machen, Verantwortung zuordnen
- Skalierung kostet: Autoscaling-Grenzen, Reserved vs. On-Demand vs. Spot
- Kosten als Architekturkriterium (Bezug Vorlesung 3)

# Deployment

## Von Containern zur Orchestrierung

- Wiederholung Webengineering: Images, Container, Docker
- Kubernetes-Grundbegriffe: Pod, Deployment, Service, Ingress, ConfigMap, Secret
- Was der Orchestrator übernimmt: Scheduling, Neustart, Skalierung, Rollout
- Alternativen: Managed Container-Dienste, PaaS, Serverless

## Infrastructure as Code

- Infrastruktur deklarativ beschreiben und versionieren: Terraform, Pulumi, CloudFormation
- Reproduzierbare Umgebungen, Review von Infrastrukturänderungen
- GitOps: Git als Quelle der Wahrheit, ein Operator gleicht ab (Argo CD, Flux)
- Drift und wie man ihn erkennt

## CI/CD-Pipelines

- Stufen: Build, Test, Sicherheitsscans, Artefakt, Deployment, Verifikation
- Environments und Promotion: dev, staging, prod
- Artefakte unveränderlich, Konfiguration pro Umgebung (12-Factor)
- Pipeline als Produktionscode behandeln

## Deployment-Strategien

- Recreate, Rolling Update
- Blue/Green: zwei Umgebungen, Umschalten
- Canary: schrittweise Anteil des Traffics, Metriken beobachten
- Feature Flags und Dark Launches: Deployment von Release entkoppeln
- Rollback: Wie schnell und wie sicher?

## Datenbankmigrationen ohne Downtime

- Expand/Contract als Grundmuster: Schema erweitern, Code ausrollen, Daten nachziehen, Altes entfernen
- Abwärtskompatible Änderungen, Versionierung von Datenformaten
- Backfills bei großen Tabellen ohne Sperren
- Migrationen als Teil der Pipeline
- Rollback von Daten ist schwer: vorwärts reparieren

## Schnittstellen evolvieren

- APIs ändern, ohne Clients zu brechen: additive Änderungen, Tolerant Reader
- Versionierung: URL, Header oder gar nicht; Lebenszyklus alter Versionen
- Consumer-Driven Contract Tests statt End-to-End-Tests über alle Services
- Events und Nachrichtenformate: Schema-Registry, Kompatibilitätsregeln

## Autoscaling

- Horizontal Pod Autoscaler: CPU, Speicher, eigene Metriken
- Queue-basiertes Scaling: Rückstau als Signal
- Cluster-Autoscaling und Scale-to-Zero
- Grenzen: Aufwärmzeit, Datenbankverbindungen, Kostenobergrenzen

# Observability

## Monitoring vs. Observability

- Monitoring: bekannte Fragen beantworten
- Observability: unbekannte Fragen aus den Daten heraus beantworten können
- Drei Säulen: Logs, Metriken, Traces; dazu Events und Profiling
- Kosten der Telemetrie: Volumen, Speicherung, Sampling

## Logs

- Strukturierte Logs (JSON) statt Freitext
- Korrelation über Request-ID, Trace-ID, Mandant, Nutzer
- Log-Level sinnvoll nutzen, keine personenbezogenen Daten (Bezug Vorlesung 2)
- Zentrale Sammlung: Loki, Elasticsearch, Cloud-Dienste

## Metriken

- Typen: Counter, Gauge, Histogram
- RED-Methode für Services: Rate, Errors, Duration
- USE-Methode für Ressourcen: Utilization, Saturation, Errors
- Golden Signals: Latenz, Traffic, Fehler, Sättigung
- Prometheus und Grafana als Standardstack

## Distributed Tracing

- Einen Request über viele Services verfolgen: Trace, Span, Kontextweitergabe
- Wo geht die Zeit verloren? Wo entstehen Fehler?
- OpenTelemetry als herstellerneutraler Standard für alle drei Säulen
- Werkzeuge: Jaeger, Tempo, Cloud-Dienste

## Demo: Metriken und Traces live

- Kleine Anwendung, mit OpenTelemetry instrumentiert
- Einen Request durch mehrere Services verfolgen, Latenz und Fehler im Trace finden
- RED-Metriken im Dashboard, einen Alert auslösen

<!-- TODO(Lu): pick the stack, e.g. the OpenTelemetry demo app or a Grafana LGTM stack in Docker Compose -->

## SLOs und Alerting

- Vom SLI zum SLO zum Alert (Bezug Vorlesung 4)
- Symptombasiert alarmieren, nicht ursachenbasiert
- Burn-Rate-Alerts auf das Fehlerbudget
- Alert Fatigue: jeder Alarm muss eine Handlung auslösen
- Dashboards: für wen, welche Frage?

## Incident Management

- Rollen: Incident Commander, Kommunikation, Bearbeitung
- Runbooks und Eskalation
- On-Call gestalten: Rotation, Belastung, Vergütung
- Blameless Post-Mortem als Lernschleife (Bezug Vorlesung 4)

## Performance im Betrieb

- Lasttests vor und nach Änderungen: k6, Gatling, Locust
- Kapazitätsplanung mit realen Daten (Bezug Vorlesung 1)
- Profiling in Produktion: Continuous Profiling
- Performance-Regressionen in der Pipeline erkennen

## Beispiel: Observability-Konzept

- Beispielsystem aus einer der vorherigen Vorlesungen
- Gemeinsam entwickeln: Welche SLIs und SLOs? Welche Metriken pro Service? Welche Alerts lösen welche Handlung aus?
- Welche Fragen soll ein Dashboard beantworten?

# AI Engineering: Betrieb von KI-Systemen

## Observability für LLM-Aufrufe

- Neue Signale: Tokens, Kosten, Latenz bis zum ersten Token, Modellversion
- Qualität als Metrik: Nutzerfeedback, automatische Bewertungen, Verweigerungsrate
- Tracing von Prompt-Ketten und Tool-Aufrufen
- OpenTelemetry GenAI Semantic Conventions, Werkzeuge wie Langfuse

## Evaluation als Deployment-Gate

- Evals statt Unit-Tests: Testdatensätze, Bewertungskriterien, LLM als Richter
- Regressionen bei Modell- und Prompt-Änderungen erkennen
- Versionierung von Prompts, Modellen und Retrieval-Konfiguration
- Offline-Evaluation vor, Online-Metriken nach dem Rollout

## Deployment-Strategien für Modelle

- Canary für neue Modellversionen, A/B-Tests auf Qualitätsmetriken
- Shadow Mode: neues Modell mitlaufen lassen, Ausgaben vergleichen
- Modell-Deprecation durch Anbieter als planbares Ereignis
- Kosten-Monitoring und Budgets als Alert

## KI im Betrieb

- LLM-gestützte Incident-Analyse, Log-Zusammenfassung, Runbook-Assistenz
- Grenzen: halluzinierte Ursachen, Automatisierung kritischer Aktionen
- Der Mensch entscheidet, die KI assistiert

# Zusammenfassung

## Zusammenfassung

- Betreibbarkeit wird entworfen, nicht nachgerüstet
- Deployment: Orchestrierung, Infrastructure as Code, Pipelines; Strategien entkoppeln Deployment von Risiko
- Observability: strukturierte Logs, RED-Metriken, Tracing; SLO-basiertes Alerting
- Incidents sind Lerngelegenheiten, wenn der Prozess stimmt
- KI-Systeme brauchen Evaluation sowie Kosten- und Qualitätsmetriken als Teil des Betriebs

## Nächste Vorlesung

- Wiederholung und Klausurvorbereitung (Lukas & Silas)

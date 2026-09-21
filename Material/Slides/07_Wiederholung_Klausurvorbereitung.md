---
title: "Skalierbare Systeme - Wiederholung und Klausurvorbereitung"
topic: "scalable_systems_1_7"
author: "Lukas Panni & Silas Schnurr"
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
Time plan: 2 VE = 90 min, no break.
  Heute, Roter Faden                       5
  Wiederholung                            30   6 slides
  Musteraufgaben                          30   2-3 tasks
  Klausur, Tipps                          10
  Offene Fragen, Feedback                 10
  Sum                                     85   5 min slack
-->

# Wiederholung

## Heute

- Roter Faden der Vorlesung
- Wiederholung der Kernbegriffe pro Vorlesung
- Musteraufgaben gemeinsam lösen
- Klausur: Ablauf und Hinweise
- Offene Fragen, Feedback

## Roter Faden

- Von Anforderungen zu Architektur zu Betrieb
- Last verstehen und messen, Bausteine, Daten, Entscheidungen, Resilienz, Stile, Betrieb
- Alles ist ein Trade-off; der Kontext entscheidet
- KI-Komponenten: gleiche Regeln, neue Kostenmodelle und Fehlerarten

## Grundlagen Systemdesign

- Skalierbarkeit, Performance, Elastizität; Lastparameter, Perzentile
- Vertikal vs. horizontal, Zustand, Koordination als Grenze
- Load Balancer, Cache, Queue, Replikation, Partitionierung, CDN
- Vorgehen beim Systemdesign

## Datenarchitektur, Mandantenfähigkeit & System-Security

- Replikation, Partitionierung, Konsistenzmodelle, CAP und PACELC
- Silo, Bridge, Pool; Isolation, Noisy Neighbours
- Threat Modeling, Vertrauensgrenzen, Identität, Secrets, Supply Chain

## Technische Entscheidungsfindung

- Architekturentscheidung, Reversibilität, Trade-offs
- Qualitätsattribute messbar machen
- Stakeholder, Kontextfaktoren, Build vs. Buy
- ADRs, Optionsvergleich, Experimente

## Resilienz, Fehlertoleranz & Hochverfügbarkeit

- Fault, Error, Failure; Verfügbarkeit, SLO, Fehlerbudget
- Redundanz, Failover, Konsens, RPO und RTO
- Timeouts, Retries, Idempotenz, Circuit Breaker, Bulkhead, Load Shedding, Graceful Degradation
- Chaos Engineering, Post-Mortems

## Architekturstile

- Kopplung und Kohäsion
- Monolith, Modular Monolith, Service-basiert, Microservices, Event-Driven, Serverless
- Bewertung gegen Qualitätsattribute, Migrationspfade
- DDD und Conway's Law als Schneidewerkzeuge

## Betrieb, Observability & Deployment

- DevOps, SRE, Platform Engineering, Kosten
- Orchestrierung, Infrastructure as Code, Deployment-Strategien
- Logs, Metriken, Traces, SLO-basiertes Alerting
- Incident Management, Evaluation von KI-Systemen

# Klausurvorbereitung

## Musteraufgaben

- Systemdesign-Aufgabe gemeinsam lösen: Anforderungen, Design, Trade-offs
- Transferfrage: Ausfallszenario analysieren und Muster zuordnen
- Begriffsfragen als Schnelldurchlauf

<!-- TODO: prepare 2-3 sample tasks with solutions -->

## Klausur

<!-- TODO: format, duration, aids, weighting, date; keep in sync with session 1 -->

- Format und Dauer
- Zulässige Hilfsmittel
- Aufgabentypen: Begriffe, Transfer, Systemdesign
- Bewertungsschwerpunkt: Die Begründung zählt mehr als die "richtige" Antwort

## Tipps

- Trade-offs immer benennen: Was gewinnt man, was gibt man auf?
- Kontext nutzen: Teamgröße, Last, Budget aus der Aufgabe herauslesen
- Skizzen sind erlaubt und hilfreich
- Zeit einteilen

# Abschluss

## Offene Fragen & Feedback

- Fragen zu allen Themen
- Feedback zur Vorlesung: Was hat geholfen, was fehlt?

<!-- TODO: feedback form link -->

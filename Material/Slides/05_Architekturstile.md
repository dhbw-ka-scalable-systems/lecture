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


<!--


## Event-Driven Architecture (Pub/Sub)

- **Problem:** Ein Service muss alle anderen kennen und aufrufen, die auf sein Ereignis reagieren sollen
- **Lösung:** Ein Produzent veröffentlicht ein Ereignis ("Bestellung aufgegeben"), beliebig viele Abonnenten reagieren (Rechnung, Versand, Benachrichtigung)
- Unterschied zur Queue: Befehl an einen Empfänger vs. Ereignis an alle Interessierten
- **Nachteile:** Der Gesamtablauf ist nirgends an einer Stelle sichtbar; Reihenfolge, Duplikate, Weiterentwicklung der Event-Formate
- **Beispiele:** AWS SNS, Google Pub/Sub, Kafka
- Vertiefung in Vorlesung 5

-->
<!--
Time plan: 4 VE = 180 min content, Friday 8:30-11:45 with a 15 min break on top.
  Heute, Lernziele                         5
  Grundlagen                              15   3 slides
  Monolithische Stile                     25   5 slides
  Verteilte Stile                         50   9 slides
  Stile vergleichen und wählen            45   4 slides + example with 3 cases ~25
  AI Engineering                          15
  Zusammenfassung                          5
  Sum                                    160   20 min slack
-->

# Grundlagen

## Heute

- Kopplung, Kohäsion, Modularität
- Monolithische Stile
- Verteilte Stile: Services, Microservices, Event-Driven, Serverless
- Stile vergleichen, wählen, migrieren
- AI Engineering: Architektur von KI-Systemen
- Beispiel: Stil für drei Fallstudien wählen

## Lernziele

- Architekturstile anhand von Kopplung, Deployment und Datenhaltung unterscheiden
- Stärken und Schwächen jedes Stils gegen Qualitätsattribute bewerten
- Für einen Kontext einen Stil wählen und die Entscheidung begründen
- Migrationspfade zwischen Stilen kennen

## Architekturstil, Pattern, Taxonomie

- Stil: Grundstruktur des Gesamtsystems (Schichten, Microservices, Event-Driven)
- Pattern: wiederkehrende Lösung für ein Teilproblem (Circuit Breaker, Saga, CQRS)
- Taxonomie nach Richards/Ford: monolithisch vs. verteilt, technisch vs. fachlich partitioniert

## Kopplung und Kohäsion

- Kohäsion: Was zusammengehört, gehört zusammen
- Kopplungsarten: statisch (Code, Daten, Deployment) und dynamisch (Aufruf, Zeit, Konsistenz)
- Zeitliche Kopplung: synchron vs. asynchron
- Kontrakte als Kopplungspunkt: strikt vs. lose
- Ziel: Kopplung dort, wo sie günstig ist

## Bewertungsmaßstab

- Skalierbarkeit, Elastizität, Fehlertoleranz
- Deployability, Testbarkeit, Evolvierbarkeit
- Einfachheit, Kosten, Performance
- Sternchen-Bewertung als Diskussionsgrundlage, nicht als Wahrheit

# Monolithische Stile

## Schichtenarchitektur

- Wiederholung aus Webengineering; bei Fowler: Presentation, Domain, Data Source
- Technische Partitionierung: Änderungen an einem Feature berühren alle Schichten
- Stärken: einfach, verständlich, günstig
- Schwächen: Skalierung nur im Ganzen, Sinkhole-Antipattern

## Modular Monolith

- Fachliche Module mit klaren Schnittstellen in einem Deployment
- Regeln durchsetzen: Modulgrenzen im Code prüfen (ArchUnit, Spring Modulith)
- Eine Datenbank, aber getrennte Schemata pro Modul
- Der unterschätzte Standard: die meisten Systeme sollten hier anfangen

## Hexagonale und Clean Architecture

- Fachlichkeit innen, Technik außen: Ports und Adapter
- Testbarkeit ohne Infrastruktur
- Einordnung: Strukturierungsprinzip innerhalb eines Deployments, kein Stil des Gesamtsystems

## Microkernel

- Kern plus Plugins: IDEs, Browser, Zahlungsanbieter-Integrationen
- Erweiterbarkeit ohne Kernänderung
- Registry, Kontrakte, Versionierung der Plugin-Schnittstelle

## Den Monolithen skalieren

- Horizontal: mehrere Instanzen hinter einem Load Balancer, zustandslos halten
- Datenbank: Read Replicas, Caching, Partitionierung
- Asynchrone Verarbeitung auslagern
- Grenzen: Deployment-Geschwindigkeit, Teamgröße, unterschiedliche Lastprofile im selben Prozess

# Verteilte Stile

## Service-basierte Architektur

- Wenige, grobgranulare Services, oft mit gemeinsamer Datenbank
- Pragmatischer Mittelweg zwischen Monolith und Microservices
- Stärken: Deployability, Fehlerisolation, überschaubarer Betrieb
- Schwächen: gemeinsame Datenbank als Kopplungspunkt

## Microservices (1): Prinzipien

- Bounded Context als Schnittgrenze, Datenhoheit pro Service
- Unabhängiges Deployment, unabhängige Skalierung, Technologiefreiheit
- Teams besitzen Services (Conway's Law bewusst genutzt)
- Größe ist nicht das Kriterium, Unabhängigkeit ist es

## Microservices (2): Kommunikation

- Synchron: REST, gRPC; Vor- und Nachteile
- Asynchron: Messaging, Events
- Service Discovery, API Gateway, Backend for Frontend
- Service Mesh: Sidecars für Sicherheit, Resilienz, Telemetrie

## Microservices (3): Daten

- Keine gemeinsame Datenbank: Was das für Joins und Transaktionen heißt
- Saga-Pattern: Orchestrierung vs. Choreografie
- Eventual Consistency akzeptieren und kommunizieren
- Datenduplikation als bewusster Preis

## Microservices (4): Kosten

- Verteilte Komplexität: Netzwerk, Latenz, Teilausfälle, Debugging
- Fowlers First Law of Distributed Object Design: "Don't distribute your objects", sinngemäß auch für Services
- Betriebsaufwand: Deployment, Monitoring, Sicherheit pro Service
- Organisatorische Voraussetzungen: Teams, Plattform, Reifegrad
- Wann nicht: kleines Team, unklare Domäne, Start eines Produkts

## Event-Driven Architecture

- Ereignisse statt Aufrufe: Produzenten kennen ihre Konsumenten nicht
- Topologien: Broker (Choreografie) vs. Mediator (Orchestrierung)
- Events vs. Commands vs. Queries
- Technologie: Kafka, RabbitMQ, Cloud-Dienste; Log vs. Queue

## Event Sourcing und CQRS

- Zustand als Ereignisfolge: Nachvollziehbarkeit, Replays, Projektionen
- CQRS: getrennte Lese- und Schreibmodelle
- Kosten: Schema-Evolution der Events, Komplexität, Eventual Consistency
- Sinnvolle Einsatzbereiche: Audit, Finanzen, Kollaboration

## Serverless und FaaS

- Funktionen statt Server: Skalierung auf null und nach oben, Bezahlung pro Aufruf
- Cold Starts, Laufzeitgrenzen, Zustandslosigkeit
- Einsatzbereiche: Event-Verarbeitung, Glue-Code, sporadische Last
- Grenzen: Langläufer, hohe Dauerlast, Lock-in

## Weitere Stile kurz

- Space-Based Architecture: In-Memory-Datengrid für extreme Lastspitzen
- Cell-Based Architecture: Bulkheads im Großen, der Ausfall einer Zelle trifft nur einen Teil der Nutzer
- Microfrontends und Backend for Frontend (Bezug Webengineering)

# Stile vergleichen und wählen

## Vergleich

- Monolith, Modular Monolith, Service-basiert, Microservices, Event-Driven, Serverless
- Gegen: Skalierbarkeit, Fehlertoleranz, Deployability, Einfachheit, Kosten, Evolvierbarkeit

<!-- TODO: comparison table styles vs. quality attributes (Richards/Ford star ratings as a starting point) -->

## Domain-Driven Design als Schneidewerkzeug

- Bounded Contexts: Wo enden Begriffe, wo enden Modelle?
- Context Map: Beziehungen zwischen Kontexten
- Event Storming zur Entdeckung von Grenzen
- Schnitt nach Fachlichkeit vor Schnitt nach Technik

## Organisation und Architektur

- Conway's Law: Systeme spiegeln die Kommunikationsstruktur
- Inverse Conway Maneuver: Teams so schneiden, wie das System aussehen soll
- Team Topologies: Stream-aligned, Platform, Enabling, Complicated-Subsystem
- Kognitive Last als Grenze der Servicegröße

## Migrationspfade

- Monolith \rightarrow{} Modular Monolith \rightarrow{} Services: Grenzen erst im Code, dann im Deployment
- Strangler Fig: schrittweise ersetzen statt Big Bang
- Zurück zum Monolithen: bekannte Fälle und ihre Gründe
- Daten trennen ist der schwierigste Schritt

<!-- TODO: examples for consolidation back to a monolith (e.g. Amazon Prime Video 2023, Segment) -->

## Beispiel: Stil wählen

- Drei Fallstudien: Startup-MVP mit drei Entwicklern, Kernsystem eines Versicherers mit 200 Entwicklern, Datenpipeline für Sensordaten
- Pro Fallstudie gemeinsam: Stil wählen, Bewertung gegen Qualitätsattribute, Begründung im ADR-Format (Vorlesung 3)
- Diskussion: Welche Kontextfaktoren waren entscheidend?

# AI Engineering: Architektur von KI-Systemen

## Referenzarchitektur einer LLM-Anwendung

- Eingang und Gateway, Orchestrierung, Modellzugriff, Retrieval, Speicher (Kontext, Verlauf), Evaluation und Monitoring
- Die KI-Komponente ist ein Service unter vielen, mit besonderen Eigenschaften
- Klassische Regeln gelten: Schnittstellen, Datenhoheit, Resilienz, Observability

<!-- TODO: diagram of the reference architecture -->

## Muster für KI-Komponenten

- RAG: Retrieval vor Generierung, Datenpipeline als eigener Teil des Systems
- Tool Use und Agenten: das Modell ruft Systemfunktionen auf, der Kontrollfluss wird nicht-deterministisch
- Routing: Modellwahl pro Anfrage
- Guardrails-Schicht: Ein- und Ausgabeprüfung
- Modell-Gateway als Anti-Corruption-Layer gegenüber Anbietern

## Einbettung in den Architekturstil

- Lange Laufzeiten sprechen für asynchrone Verarbeitung: Queue, Job, Callback, Streaming
- Event-Driven: KI-Verarbeitung als Konsument von Ereignissen
- Serverless für sporadische KI-Aufgaben, dedizierte Kapazität für Dauerlast
- Standards für Integration: Model Context Protocol als Schnittstelle zu Tools und Daten

# Zusammenfassung

## Zusammenfassung

- Stile unterscheiden sich in Kopplung, Deployment und Datenhaltung, nicht in "modern oder alt"
- Modular Monolith als Standardstart, verteilte Stile bei klarem Bedarf
- Microservices kaufen Unabhängigkeit mit verteilter Komplexität
- Event-Driven und Serverless lösen spezifische Lastprofile
- KI-Komponenten sind Services mit langen Laufzeiten, hohen Kosten und nicht-deterministischem Verhalten

## Nächste Vorlesung

- Betrieb, Observability & Deployment (Lukas)

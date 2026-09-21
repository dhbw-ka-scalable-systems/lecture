---
title: "Skalierbare Systeme - Grundlagen Systemdesign"
topic: "scalable_systems_1_1"
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
Time plan: 4 VE = 180 min content, Friday 8:30-11:45 with a 15 min break on top.
  Organisatorisches                 30   student round ~15
  Inhalt, Lernziele, Literatur      10
  Was heißt skalierbar?             35   7 slides
  Bausteine skalierbarer Systeme    30   5 slides
  Systemdesign als Methode          35   worked example ~20
  AI Engineering                    15
  Zusammenfassung                    5
  Sum                              160   20 min slack
-->

# Organisatorisches

## Heute

- Organisatorisches
  - Vorstellung
  - Ablauf & Material
  - Prüfungsleistung
- Vorlesungsinhalt & Lernziele
- Was heißt skalierbar?
- Bausteine skalierbarer Systeme
- Systemdesign als Methode
- AI Engineering: LLM-Workloads als Lastprofil

## Vorstellung

### Dozent: Silas Schnurr

- Per Du
- E-Mail-Adresse: schnurr.silas@edu.dhbw-karlsruhe.de
- Seit Dezember 2025 IT-Berater/Lead Developer bei PTA IT-Beratung
- 2015 - 2025 bei PeakAvenue in Bühl
  - 2023 - 2025: Softwarearchitekt & Teamleiter Softwareentwicklung
  - 2021 - 2023: _M.Sc._ Informatik - HKA
  - 2018 - 2021: _B.Sc._ Informatik - DHBW Karlsruhe
  - 2015 - 2018: Ausbildung Fachinformatiker Anwendungsentwicklung

## Ablauf

- 7 Termine, insgesamt 26 Vorlesungseinheiten (VE)
  - Sechs Themenblöcke mit je 4 VE, im Wechsel von Silas und Lukas
  - Abschluss: Wiederholung & Klausurvorbereitung (2 VE)
- Freitags 8:30 bis 11:45 Uhr mit 15 Minuten Pause
  - Wöchentlich vom 02.10.2026 bis 06.11.2026
  - Termin 7 ein bis zwei Wochen vor der Klausur
- Vorlesung mit durchgespielten Beispielen und Diskussion von Trade-offs, keine Übungsaufgaben

## Material

- Vorlesungsfolien \rightarrow{} Slides
- Vorlesungsnotizen \rightarrow{} Notes
- Alles auf GitHub: [dhbw-ka-scalable-systems/lecture](https://github.com/dhbw-ka-scalable-systems/lecture)
  - Fertige PDFs unter Releases
  - Fehler in den Folien? \rightarrow{} Issue
  - Fragen? \rightarrow{} Discussions

<!-- TODO: enable Discussions on the repo or drop the bullet -->

## Prüfungsleistung

<!-- TODO: exam format, duration, aids, weighting, date; keep in sync with session 7 -->

- Klausur am Ende des Semesters
  - Theoriefragen zu Begriffen und Konzepten
  - Transferaufgaben: ein System entwerfen, Trade-offs begründen
- Details in der letzten Vorlesung (Klausurvorbereitung)

## Vorstellung Studis

- Name und Firma
- Was baut oder betreibt ihr im Unternehmen? Wie groß ist das System?
- Vorkenntnisse: Webengineering, Cloud, Docker/Kubernetes, verteilte Systeme
- Erwartungen & Wünsche an die Vorlesung

# Vorlesungsinhalt & Lernziele

## Ziele der Vorlesung

- Skalierbarkeit, Verfügbarkeit und Sicherheit als Qualitätsattribute verstehen und messbar machen
- Bausteine und Architekturstile verteilter Systeme kennen und einordnen
- Technische Entscheidungen im Kontext treffen und begründen: Trade-offs statt Patentrezepte
- Systeme so entwerfen, dass sie betreibbar sind: Deployment, Observability, Resilienz
- Verstehen, was KI-Komponenten an diesen Fragen ändern und was nicht

## Vorlesungsinhalte

1. 02.10. Grundlagen Systemdesign _(Silas)_
2. 09.10. Datenarchitektur, Mandantenfähigkeit & System-Security _(Lukas)_
3. 16.10. Technische Entscheidungsfindung: Trade-offs, Stakeholder & Kontext _(Silas)_
4. 23.10. Resilienz, Fehlertoleranz & Hochverfügbarkeit _(Lukas)_
5. 30.10. Architekturstile _(Silas)_
6. 06.11. Betrieb, Observability & Deployment _(Lukas)_
7. Vor der Klausur: Wiederholung und Klausurvorbereitung _(2 VE, beide)_

## Roter Faden

- Es gibt keine "richtige" Architektur, nur eine passende für einen Kontext
- Jede Vorlesung endet mit dem Blick auf KI-Systeme: Was ändert sich, wenn ein LLM Teil des Systems ist?
- Aufbau auf Webengineering: HTTP, REST, Backend, Datenbanken, Container und Web-Security werden vorausgesetzt

## Literatur: Modulhandbuch

- Mark Richards, Neal Ford: _Fundamentals of Software Architecture: An Engineering Approach_. O'Reilly, 2020
- Martin Fowler: _Patterns of Enterprise Application Architecture_. Addison-Wesley, 2013 (19. Druck)
- Neal Ford et al.: _Software Architecture: The Hard Parts: Modern Trade-off Analyses for Distributed Architectures_. O'Reilly, 2021

Auf weitere Literatur wird in der Vorlesung an den entsprechenden Stellen verwiesen.

## Weiterführende Literatur

- Martin Kleppmann, Chris Riccomini: _Designing Data-Intensive Applications_, 2. Auflage (O'Reilly)
- Michael Nygard: _Release It!_, 2. Auflage (Pragmatic Bookshelf)
- Betsy Beyer et al.: _Site Reliability Engineering_ (O'Reilly, frei unter sre.google)
- Chip Huyen: _AI Engineering_ (O'Reilly)
- Alex Xu: _System Design Interview_ (Byte Code LLC)

# Was heißt skalierbar?

## Skalierbarkeit

- Ein System ist skalierbar, wenn es mit wachsender Last umgehen kann, ohne dass Performance oder Kosten unverhältnismäßig leiden
- Keine Ja/Nein-Eigenschaft: "Wenn das System so wächst, welche Optionen haben wir?"
- Abgrenzung
  - Performance: Wie schnell ist das System bei gegebener Last?
  - Elastizität: Wie schnell passt sich das System an schwankende Last an?
  - Effizienz: Was kostet eine Einheit Arbeit?

## Last beschreiben

- Lastparameter: Requests pro Sekunde, gleichzeitige Nutzer, Datenvolumen, Schreib-/Leseverhältnis, Fan-out
- Lastprofile: Tag/Nacht, saisonale Spitzen, Batch-Fenster, virale Ereignisse
- Beispiel Twitter-Timeline: Fan-out beim Schreiben vs. beim Lesen (Kleppmann)
- Wachstum: Welcher Parameter wächst zuerst? Nutzer, Daten oder Komplexität?

## Performance messen

- Latenz vs. Antwortzeit vs. Durchsatz
- Mittelwerte lügen: Perzentile (p50, p95, p99), Tail Latency
- Warum p99 zählt: Fan-out verstärkt Ausreißer, wichtige Kunden erzeugen oft die langsamen Requests
- SLI, SLO, SLA als Vokabular (Vertiefung in Vorlesung 4 und 6)

## Skalierbarkeit ist ein Qualitätsattribut von vielen

- Zuverlässigkeit, Wartbarkeit, Sicherheit, Kosten, Time-to-Market
- Skalierbarkeit kostet: Komplexität, Betrieb, Geld
- Vorzeitige Skalierung ist ein Fehler, fehlende Skalierungsoptionen auch
- Ziel der Vorlesung: bewusst entscheiden, nicht maximal skalieren

## Vertikal vs. horizontal skalieren

- Scale-up: größere Maschine; einfach, begrenzt, teuer am oberen Ende
- Scale-out: mehr Maschinen; nahezu unbegrenzt, erfordert Verteilung
- Shared-nothing-Architektur als Voraussetzung für Scale-out
- In der Praxis: Mischformen, "so lange wie möglich vertikal"

## Grenzen der Skalierung

- Amdahlsches Gesetz: der serielle Anteil begrenzt den Speedup
- Universal Scalability Law: Contention und Coherency lassen den Durchsatz sogar sinken
- Koordination ist der Feind: Locks, verteilte Transaktionen, Konsens
- Faustregel: Skalierung funktioniert, wo Arbeit unabhängig ist

## Zustand als Herausforderung

- Zustandslose Komponenten lassen sich beliebig replizieren
- Zustand muss irgendwo liegen: Datenbank, Cache, Session Store, Queue
- Sticky Sessions als Übergangslösung und ihre Probleme (Wiederholung Webengineering)
- Session-State-Muster nach Fowler: Client, Server und Database Session State
- Muster: Zustand nach außen verlagern, Zustand partitionieren

# Bausteine skalierbarer Systeme

## Überblick

- Load Balancer
- Caches, vom Browser bis zum CDN
- Asynchrone Verarbeitung mit Queues
- Datenbank-Skalierung: Read Replicas, Sharding, Connection Pooling (Vertiefung in Vorlesung 2)

\rightarrow{} Jeder Baustein löst ein Problem und bringt neue mit

## Load Balancing

- Verteilung eingehender Anfragen auf mehrere Instanzen
- Layer 4 vs. Layer 7
- Algorithmen: Round Robin, Least Connections, gewichtet, konsistentes Hashing
- Health Checks: Ausfall erkennen und Instanzen aus der Rotation nehmen
- Der Load Balancer selbst als Single Point of Failure

## Caching (1): Ebenen

- Browser, CDN, Reverse Proxy, Anwendungscache, Datenbankcache
- CDN als Cache am Netzrand (Wiederholung Webengineering), heute auch für APIs: Edge Caching, Edge Functions
- Was cachen: teure Berechnungen, häufige Lesezugriffe, selten geänderte Daten
- Trefferquote (Hit Rate) als Kennzahl

## Caching (2): Strategien und Invalidierung

- Cache-Aside, Read-Through, Write-Through, Write-Behind
- Invalidierung: TTL, explizit, ereignisbasiert
- "There are only two hard things in Computer Science: cache invalidation and naming things"
- Probleme: Stale Reads, Cache Stampede, Hot Keys

## Asynchrone Verarbeitung

- Entkopplung von Anfrage und Verarbeitung: Queue + Worker
- Lastspitzen glätten, Fehler isolieren, unabhängig skalieren
- Backpressure: Was passiert, wenn Produzenten schneller sind als Konsumenten?
- Kosten: Eventual Consistency, Nachvollziehbarkeit, Idempotenz nötig (Vertiefung in Vorlesung 4)

# Systemdesign als Methode

## Vorgehen

1. Anforderungen klären: funktional und nicht-funktional, Lastannahmen
2. Überschlagsrechnung: Kapazität, Speicher, Bandbreite
3. High-Level Design: Komponenten und Datenfluss
4. Vertiefung: Engpässe, Datenmodell, Skalierungspfade
5. Trade-offs benennen: Was haben wir bewusst nicht gemacht?

## Zahlen, die man kennen sollte

- Latenzen: L1-Cache, RAM, SSD, Netzwerk im Rechenzentrum, Netzwerk interkontinental
- Größenordnungen: Requests pro Sekunde einer Instanz, Speicherbedarf pro Datensatz
- Faustregeln: ein Tag hat rund 86.400 Sekunden; 1 Million Nutzer mit 10 Aktionen pro Tag sind rund 115 Requests pro Sekunde

<!-- TODO: table with latency numbers (Jeff Dean / Colin Scott), updated values -->

## Beispiel: Systemdesign

- Gemeinsam am Whiteboard durchspielen: URL-Shortener oder Chat-Dienst
- Anforderungen klären, dann Überschlagsrechnung: Speicher pro Jahr, Bandbreite zur Spitzenzeit, Instanzen
- High-Level Design, Engpässe finden, Skalierungspfade benennen
- Das Ergebnis ist nie exakt, zeigt aber, wo der Engpass liegt
- Diskussion der Trade-offs

# AI Engineering: LLM-Workloads als Lastprofil

## Was ist anders?

- Requests dauern Sekunden statt Millisekunden, Antworten werden gestreamt
- Kosten pro Request in Tokens, nicht in CPU-Millisekunden
- GPU-gebundene Inferenz: teuer, knapp, schlecht teilbar
- Nicht-deterministische Antworten: gleiche Eingabe, andere Ausgabe

## Skalierungshebel für KI-Systeme

- Batching und Queueing von Inferenz-Anfragen
- Caching: exaktes Caching, Prompt Caching der Anbieter, semantisches Caching
- Modell-Routing: kleines Modell für einfache, großes für schwere Aufgaben
- Rate Limits und Kostenbudgets als Teil des Designs
- Externe Modell-API vs. eigenes Hosting (Vertiefung in Vorlesung 3)

## Ausblick

- Vorlesung 2: Vektordaten, Datenschutz, Prompt Injection
- Vorlesung 4: Modell-APIs als unzuverlässige Abhängigkeit
- Vorlesung 5: Architektur von LLM-Anwendungen
- Vorlesung 6: LLMOps, Evaluation, Observability

# Zusammenfassung

## Zusammenfassung

- Skalierbarkeit heißt Optionen haben, wenn Last wächst; messbar über Lastparameter und Perzentile
- Horizontal skaliert, was zustandslos und koordinationsfrei ist
- Bausteine: Load Balancer, Cache, Queue, Replikation, Partitionierung, CDN
- Systemdesign ist ein Vorgehen: Anforderungen, Rechnen, Entwerfen, Trade-offs benennen
- LLM-Workloads folgen denselben Regeln mit anderem Kostenmodell

## Nächste Vorlesung

- Datenarchitektur, Mandantenfähigkeit & System-Security (Lukas)

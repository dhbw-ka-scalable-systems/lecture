---
title: "Skalierbare Systeme - Technische Entscheidungsfindung"
topic: "scalable_systems_1_3"
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
  Heute, Lernziele                         5
  Architektur heißt entscheiden           25   5 slides
  Qualitätsattribute und Trade-offs       25   5 slides
  Stakeholder und Kontext                 35   4 slides + discussion ~15
  Methoden der Entscheidungsfindung       50   6 slides + ADR example ~20
  AI Engineering                          15
  Zusammenfassung                          5
  Sum                                    160   20 min slack
-->

# Architektur heißt entscheiden

## Heute

- Warum Entscheidungen der Kern von Architektur sind
- Qualitätsattribute und Trade-offs
- Stakeholder und Kontext, Diskussion eurer Praxisfälle
- Methoden: ADRs, Optionsvergleich, Experimente
- AI Engineering: Entscheidungen rund um KI-Komponenten
- Beispiel: ein ADR entwickeln

## Lernziele

- Architekturentscheidungen von Implementierungsentscheidungen unterscheiden
- Trade-offs zwischen Qualitätsattributen erkennen und explizit machen
- Stakeholder und Kontextfaktoren identifizieren und in die Entscheidung einbeziehen
- Eine Entscheidung strukturiert herleiten, dokumentieren und kommunizieren

## Was ist eine Architekturentscheidung?

- Entscheidungen, die schwer rückgängig zu machen sind (Fowler)
- Sie betreffen Struktur, Qualitätsattribute, Schnittstellen und Technologie mit Lock-in
- Beispiele: Datenbank, Kommunikationsstil, Modulschnitt, Cloud-Anbieter, Programmiersprache
- Gegenbeispiele: Bibliothek für Datumsformatierung, Namenskonventionen

## Everything is a trade-off

- Erstes Gesetz der Softwarearchitektur: Alles ist ein Trade-off (Richards/Ford)
- Zweites Gesetz: Das Warum ist wichtiger als das Wie
- "Best Practice" ohne Kontext ist eine Meinung
- Die Aufgabe: Optionen sichtbar machen und bewusst wählen

## Kosten einer Entscheidung

- Direkte Kosten: Zeit, Geld, Lizenzen
- Indirekte Kosten: Komplexität, Lernkurve, Betrieb, Lock-in
- Opportunitätskosten: Was bauen wir dafür nicht?
- Kosten des Nicht-Entscheidens: Drift, implizite Entscheidungen durch den Code

## Technische Schulden

- Schulden als Folge von Entscheidungen: bewusst oder unbewusst, umsichtig oder leichtfertig (Fowlers Technical Debt Quadrant)
- Zinsen: langsamere Änderungen, mehr Fehler, höherer Betriebsaufwand
- Schulden sichtbar machen und tilgen: Backlog, Budget, Modernisierung in Schritten
- Nicht jede Schuld ist schlecht: Time-to-Market als bewusster Kredit

## Reversibilität

- Einbahnstraßen und Drehtüren: one-way vs. two-way doors (Bezos)
- Irreversible Entscheidungen verdienen Zeit, reversible verdienen Tempo
- Last Responsible Moment: so spät wie möglich entscheiden, aber nicht später
- Reversibilität herstellen: Abstraktionen, Standards, Daten exportierbar halten

# Qualitätsattribute und Trade-offs

## Qualitätsattribute

- ISO 25010 als Landkarte: Funktionalität, Performance, Kompatibilität, Benutzbarkeit, Zuverlässigkeit, Sicherheit, Wartbarkeit, Portabilität
- Architecturally Significant Requirements: die wenigen Anforderungen, die die Architektur prägen
- Implizite Anforderungen aufdecken: Was hat der Auftraggeber nicht gesagt?

## Typische Konflikte

- Konsistenz vs. Verfügbarkeit
- Latenz vs. Durchsatz
- Flexibilität vs. Einfachheit
- Time-to-Market vs. Wartbarkeit
- Kosten vs. Resilienz
- Sicherheit vs. Benutzbarkeit

## Qualitätsattribute messbar machen

- Quality Attribute Scenarios: Auslöser, Umgebung, Reaktion, Maß (Bass et al.)
- Beispiel: "Bei zehnfachem Traffic am Black Friday bleibt p95 unter 500 ms"
- Fitness Functions: automatisierte Prüfung von Architektureigenschaften (Ford et al.)
- Nicht messbar heißt nicht entscheidbar

## Trade-offs sichtbar machen

- Trade-off-Matrix: Optionen gegen Kriterien
- Gewichtung: Wer bestimmt, was wichtig ist?
- Die Gefahr der Scheinobjektivität: Zahlen, die Meinungen verstecken
- Ehrlich sein über Unsicherheit

## Trade-off-Analyse nach Ford et al.

- Qualitativ vor quantitativ: erst die Dimensionen finden, dann messen, was messbar ist
- MECE-Listen: Kriterien überschneidungsfrei und vollständig
- Die Out-of-Context-Falle: Vergleiche gelten nur für den konkreten Fall
- Relevante Domänenfälle modellieren statt abstrakt zu vergleichen
- Bottom Line statt Evidenzflut: Stakeholder bekommen das Ergebnis, nicht die Tabelle
- Vorsicht vor Evangelisten und Wundermitteln

# Stakeholder und Kontext

## Wer entscheidet mit?

- Business und Produkt: Ziele, Budget, Zeit
- Entwicklung: Umsetzbarkeit, Skills, Motivation
- Betrieb: Betreibbarkeit, On-Call, Kosten
- Security und Recht: Compliance, Risiko
- Endnutzer: die eigentlichen Anforderungen
- Stakeholder-Map: Einfluss und Interesse

## Kontextfaktoren

- Team: Größe, Skills, Erfahrung, Fluktuation
- Organisation: Conway's Law, Team Topologies, Entscheidungswege
- Bestand: Legacy-Systeme, Verträge, vorhandene Plattformen
- Regulatorik und Branche
- Zeit und Budget
- Zukunft: erwartetes Wachstum, Lebensdauer des Systems

## Build, Buy oder Open Source

- Kernkompetenz vs. Commodity (Wardley Maps als Denkmodell)
- Total Cost of Ownership über die Lebensdauer
- Lock-in bewerten: Ausstiegskosten, Standards, Datenexport
- Managed Services: Betrieb einkaufen, Kontrolle abgeben

## Kognitive Last und langweilige Technologie

- Jedes neue Werkzeug kostet Aufmerksamkeit des Teams
- "Choose Boring Technology" (McKinley): Innovationsbudget bewusst einsetzen
- Hype-Zyklen erkennen: Resume-Driven Development
- Wann neue Technologie trotzdem richtig ist

## Diskussion: Entscheidungen aus eurer Praxis

- Welche Architekturentscheidung habt ihr im Unternehmen erlebt oder nachvollzogen?
- Wer hat entschieden, wer war betroffen, was war der Kontext?
- Was wurde gewonnen, was aufgegeben? Würde man heute anders entscheiden?
- Sammlung an der Tafel, Zuordnung zu Stakeholdern und Kontextfaktoren

# Methoden der Entscheidungsfindung

## Architecture Decision Records

- Ein kurzes Dokument pro Entscheidung: Kontext, Optionen, Entscheidung, Konsequenzen
- Status und Lebenszyklus: vorgeschlagen, akzeptiert, abgelöst
- Im Repository neben dem Code, versioniert
- Wert: Das Warum bleibt erhalten, auch wenn das Team wechselt

## ADR: Aufbau und Beispiel

- Titel und Nummer
- Kontext: Problem, Rahmenbedingungen, Anforderungen
- Betrachtete Optionen mit Vor- und Nachteilen
- Entscheidung und Begründung
- Konsequenzen: positiv, negativ, offene Punkte

<!-- TODO: full example ADR, e.g. "Postgres statt MongoDB für Bestelldaten" -->

## Optionen systematisch vergleichen

- Kriterien aus den Qualitätsattributen ableiten
- Gewichtete Bewertung, aber Gewichte offenlegen
- ATAM light: Szenarien gegen die Architektur prüfen
- Sensitivitätspunkte: Wo kippt die Entscheidung?

## Experimente statt Meinungen

- Spike, Proof of Concept, Prototyp: Unterschiede und Zweck
- Zeitbox und klare Frage: Was wollen wir lernen?
- Lasttests und Benchmarks als Entscheidungsgrundlage
- Wegwerfen dürfen

## Risiken und Unsicherheit

- Risk Storming: Architektur gemeinsam auf Risiken abklopfen
- Pre-Mortem: "Das Projekt ist gescheitert. Warum?"
- Annahmen dokumentieren und prüfbar machen
- Entscheidung mit Ablaufdatum: Wann schauen wir wieder hin?

## Entscheidungen kommunizieren

- Diagramme als Kommunikationsmittel: C4-Modell (Kontext, Container, Komponenten)
- Dokumentation mit Struktur: arc42
- Zielgruppengerecht: Management, Entwicklung, Betrieb
- Entscheidung erklären, nicht verkünden

## Beispiel: ADR entwickeln

- Fallstudie: Nachrichtendienst mit Push-Benachrichtigungen
- Frage: synchrone Aufrufe oder Messaging zwischen den Diensten?
- Gemeinsam ein ADR entwickeln: Kontext, Optionen, Entscheidung, Konsequenzen
- Diskussion: Welche Kontextfaktoren drehen die Entscheidung?

<!-- TODO: confirm the case study -->

# AI Engineering: Entscheidungen rund um KI-Komponenten

## Wann KI und wann nicht

- Deterministische Logik, wo Regeln bekannt sind; LLM, wo Sprache, Unschärfe und Varianz im Spiel sind
- Kosten pro Request und Latenz gegen Nutzen
- Evaluierbarkeit: Können wir messen, ob es funktioniert?
- Fehlerkosten: Was passiert bei einer falschen Antwort?

## Modell-API oder eigenes Hosting

- Kriterien: Qualität, Kosten, Latenz, Datenschutz, Kontrolle, Lock-in, Betriebsaufwand
- Open-Weights-Modelle vs. proprietäre APIs
- Modell-Gateway als Abstraktion für Austauschbarkeit (Vertiefung in Vorlesung 5)
- Ein ADR-Beispiel für genau diese Entscheidung

## KI als Werkzeug im Entscheidungsprozess

- LLMs für Optionsanalyse, ADR-Entwürfe, Recherche
- Grenzen: Halluzinationen, Bias zum Mainstream, kein Wissen über die eigene Organisation
- Die Entscheidung bleibt beim Menschen: Verantwortung und Begründung
- Gute Praxis: Kontext geben, Gegenargumente erfragen, Quellen prüfen

# Zusammenfassung

## Zusammenfassung

- Architektur ist die Summe der schwer umkehrbaren Entscheidungen
- Trade-offs sind unvermeidlich; die Aufgabe ist, sie sichtbar zu machen
- Kontext und Stakeholder bestimmen, was richtig ist
- ADRs halten das Warum fest, Experimente ersetzen Meinungen
- Bei KI-Komponenten gelten dieselben Fragen mit neuen Kriterien: Kosten, Datenschutz, Evaluierbarkeit

## Nächste Vorlesung

- Resilienz, Fehlertoleranz & Hochverfügbarkeit (Lukas)

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

- Vorstellung
- Datenarchitektur: Replikation, Partitionierung, Konsistenz
- Mandantenfähigkeit
- System-Security
- AI Engineering: Daten und Sicherheit für KI-Systeme

## Vorstellung

### Dozent: Lukas Panni

<!-- TODO(Lu): bio and e-mail address -->

- Per Du
- E-Mail-Adresse: TODO

# Datenarchitektur

## Lernziele

- Replikation und Partitionierung erklären und ihre Trade-offs benennen
- Konsistenzmodelle einordnen und für einen Anwendungsfall auswählen
- Mandantenmodelle vergleichen und ein mandantenfähiges Datenmodell entwerfen
- Sicherheit als Architekturthema behandeln: Bedrohungen modellieren, Vertrauensgrenzen ziehen

## Datenmodelle: Wiederholung und Einordnung

- Relational, Dokument, Key-Value, Wide-Column, Graph, Zeitreihen, Suche
- Polyglot Persistence: das passende Modell pro Zugriffsmuster
- Zugriffsmuster vor Datenmodell: Wie wird gelesen und geschrieben?
- Zugriff aus der Anwendung (Fowler): Repository, Data Mapper, Unit of Work
- OLTP vs. OLAP: Transaktionen vs. Analysen; Data Warehouse und Data Lake kurz

## Replikation (1): Warum und wie

- Ziele: Verfügbarkeit, Leselast verteilen, Latenz durch Nähe
- Leader-Follower (Single Leader): Schreiben zentral, Lesen verteilt
- Multi-Leader: mehrere Rechenzentren, Konfliktauflösung nötig
- Leaderless (Dynamo-Stil): Quorum beim Lesen und Schreiben

## Replikation (2): Probleme

- Synchron vs. asynchron: Dauerhaftigkeit gegen Latenz
- Replication Lag und seine Folgen: Read-your-writes, Monotonic Reads, Consistent Prefix
- Failover: Datenverlust, Split Brain, Timeouts
- Praxis: Postgres Streaming Replication, MySQL, MongoDB Replica Sets

## Partitionierung (Sharding) (1)

- Ziel: Datenmenge und Schreiblast über Knoten verteilen
- Strategien: nach Schlüsselbereich, nach Hash, zusammengesetzt
- Hot Spots: schiefe Verteilung, prominente Schlüssel
- Sekundärindizes: lokal (Scatter/Gather) vs. global

## Partitionierung (Sharding) (2)

- Rebalancing: Partitionen verschieben, ohne den Betrieb zu stören
- Request Routing: Wer weiß, wo welcher Schlüssel liegt?
- Partitionierung und Joins: Was nicht mehr geht; Denormalisierung und Materialized Views als Ausweg
- Praxis: Citus, Vitess, MongoDB, DynamoDB, Cassandra

## Konsistenz (1): Begriffe

- ACID vs. BASE
- Isolation Levels: Read Committed, Snapshot Isolation, Serializable
- Konsistenzmodelle verteilter Systeme: Linearisierbarkeit, sequenziell, kausal, eventual
- Nebenläufigkeit in der Anwendung: Optimistic und Pessimistic Offline Lock (Fowler)
- Was "Konsistenz" in ACID und in CAP jeweils bedeutet

## Konsistenz (2): CAP und darüber hinaus

- CAP-Theorem: bei Netzwerkpartition zwischen Konsistenz und Verfügbarkeit wählen
- Kritik: Partitionen sind selten, Latenz ist immer da \rightarrow{} PACELC
- In der Praxis: pro Datensatz und Operation entscheiden, nicht pro System
- Beispiele: Kontostand vs. Like-Zähler

## Verteilte Transaktionen

- Two-Phase Commit: Funktionsweise, Koordinator als Schwachstelle
- Saga-Pattern: lokale Transaktionen mit kompensierenden Aktionen (Vertiefung in Vorlesung 5)
- Outbox-Pattern und Idempotenz als Voraussetzung (Vertiefung in Vorlesung 4)

# Mandantenfähigkeit

## Begriff und Motivation

- Multi-Tenancy: eine Installation, viele Kunden (Mandanten)
- Motivation: SaaS, Kosten, zentraler Betrieb, schnelle Auslieferung
- Anforderungen: Isolation, Anpassbarkeit, Kosten pro Mandant, Compliance

## Mandantenmodelle

- Silo: eigene Datenbank (oder Instanz) pro Mandant
- Pool: gemeinsames Schema, `tenant_id` in jeder Tabelle
- Bridge: gemeinsame Datenbank, eigenes Schema pro Mandant
- Vergleich: Isolation, Kosten, Betrieb, Skalierung, Migration

## Datenisolation im Pool-Modell

- Jede Abfrage filtert nach Mandant: fehleranfällig
- Row-Level Security in der Datenbank
- Verschlüsselung pro Mandant, Schlüsselverwaltung
- Backup und Restore für einen einzelnen Mandanten

## Mandantenkontext durch das System tragen

- Woher kommt der Mandant: Subdomain, Header, Token-Claim
- Middleware setzt den Kontext, Datenzugriff erzwingt ihn
- Hintergrundjobs und Queues: Kontext mitgeben
- Logging und Metriken pro Mandant

## Noisy Neighbours und Fairness

- Ein Mandant frisst die Ressourcen der anderen
- Gegenmaßnahmen: Rate Limits pro Mandant, Quotas, Prioritäten, Bulkheads
- Große Mandanten in eigene Silos heben (Silo-Promotion)
- Sharding nach Mandant als natürliche Partitionierung

## Beispiel: Mandantenfähiges Datenmodell

- Fallstudie: SaaS für Zeiterfassung mit Kunden von 5 bis 50.000 Mitarbeitenden
- Gemeinsam durchspielen: Mandantenmodell wählen und begründen, Datenmodell skizzieren
- Was passiert beim Onboarding eines Großkunden?

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

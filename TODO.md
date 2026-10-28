# Plan

Lecture "Skalierbare Systeme", DHBW Karlsruhe, winter semester 2026. Seven sessions, 26 VE, alternating between Silas (Si) and Lukas (Lu). Where it fits, a session brings in AI engineering and maps its topic onto LLM-based systems; a dedicated "AI Engineering" section is optional.

## Module

- Unit in module T4INF4900, elective on Friday mornings in the winter semester. It runs only if at least 12 students choose it.
- The module lists 27 VLS, the session plan below has 26 VE.
- Official description in the module handbook, verbatim:

  > In dieser Vorlesung geht es darum, wie aus „funktionierendem Code" zuverlässige Systeme werden.
  > Moderne Softwaresysteme in der Praxis:
  > Vom Code zum Verständnis ganzheitlicher Softwaresysteme.
  > Zentrale Architekturstile großer Systeme.
  > Wie werden reale Systeme robust, sicher und skalierbar gestaltet?
  > Fehlertoleranz und Resilienz: Umgang mit Fehlern und Teil-Ausfällen.
  > Nachvollziehen technischer Entscheidungen unter Berücksichtigung verschiedener Stakeholder und Umgebungen.

## Sessions

Fridays 8:30 to 11:45 with a 15-minute break, weekly from 2026-10-02. Session 7 is one or two weeks before the exam, date open.

| # | Date | File | Topic | VE | Who | Status |
|---|------|------|-------|----|-----|--------|
| 1 | 2026-10-02 | `Material/Slides/01_Grundlagen_Systemdesign.md` | Organisatorisches, Grundlagen Systemdesign | 4 | Si | done |
| 2 | 2026-10-09 | `Material/Slides/02_Datenarchitektur_Mandantenfaehigkeit_Security.md` | Datenarchitektur, Mandantenfähigkeit & System-Security | 4 | Lu | done |
| 3 | 2026-10-16 | `Material/Slides/03_Technische_Entscheidungsfindung.md` | Trade-offs, Stakeholder & Kontext | 4 | Si | done |
| 4 | 2026-10-23 | `Material/Slides/04_Resilienz_Hochverfuegbarkeit.md` | Resilienz, Fehlertoleranz & Hochverfügbarkeit | 4 | Lu | done |
| 5 | 2026-10-30 | `Material/Slides/05_Architekturstile.md` | Architekturstile | 4 | Si | draft |
| 6 | 2026-11-06 | `Material/Slides/06_Betrieb_Observability_Deployment.md` | Betrieb, Observability & Deployment | 4 | Lu | outline |
| 7 | before the exam | `Material/Slides/07_Wiederholung_Klausurvorbereitung.md` | Wiederholung und Klausurvorbereitung | 2 | Lu + Si | outline |

Status values: outline, draft, review, done.

## AI engineering thread

| # | Section | Core message |
|---|---------|--------------|
| 1 | LLM-Workloads als Lastprofil | Same scaling rules, different cost model: seconds per request, tokens, GPUs; batching, caching, routing |
| 2 | Daten und Sicherheit für KI-Systeme | Vector data and RAG pipelines as new data types, tenant isolation in indexes, data leaving the system, prompt injection |
| 3 | Entscheidungen rund um KI-Komponenten | When to use an LLM at all, API vs. self-hosting as an ADR, LLMs as a tool in the decision process and their limits |
| 4 | Resilienz für KI-Komponenten | Model APIs as unreliable dependencies, non-deterministic failures, degrading gracefully without the AI feature |
| 5 | no own section, one slide "Was ändert KI daran?" | Code gets cheap, the cut and the boundaries do not; LLMs default to the most common style; generated code keeps module boundaries only when they are enforced |
| 6 | Betrieb von KI-Systemen | Token, cost and quality metrics, tracing prompt chains, evals as a deployment gate, canary and shadow mode for models |
| 7 | in "Roter Faden" | Recap only |

## Cross references between sessions

The outlines say where a topic is deepened later or was introduced before. Keep these in sync when moving content.

- Session 1 is the map: its building-block catalog touches most later topics briefly (what, what for, cost, alternatives) and points to the owning session: data, replication, sharding, CAP/PACELC, ACID, NoSQL, 2PC/saga, TLS (2); unreliable networks, rate limiting, idempotency, backpressure, service mesh (4); API gateway, microservices, event-driven, gRPC, Conway's Law (5); API vs. self-hosting (3)
- Session 2 previews outbox plus idempotency (4) and the data disintegrators/integrators trade-off; saga is not in session 2, session 5 introduces it. CQRS and event sourcing live in session 5 only, schema migrations in session 6 only.
- Sessions 4 and 6 share SLOs, post-mortems, health checks and orchestration
- Session 5 runs the Anmeldesystem from session 3 through all styles (modulith, microservices, event-driven, serverless) and reuses the ADR format in Aufgabe 3. It picks up what session 1 deferred to it: Monolith vs. Microservices, Kopplung/Kohäsion, API Gateway, Event-Driven, Conway's Law. It refers forward to session 6 for tracing and pipelines. It has no AI-engineering section.

## Conventions

- One file per session, `NN_Topic.md`, German content with the English technical terms the field uses.
- `#` is a section and gets a title slide, `##` is a slide, `###` is a block inside a slide. Keep a slide at seven bullets or fewer, the theme overflows beyond that.
- Every session file starts with a time plan comment: minutes per section against 180 content minutes for 4 VE (8:30 to 11:45, the 15-minute break comes on top), 90 for session 7. Update it when content moves.
- Author notes go into HTML comments (`<!-- TODO: ... -->`) after a blank line; pandoc drops them from the PDF.
- Frontmatter: copy `template_Slides.md`, set `title`, `topic` (`scalable_systems_1_N`) and `author`. The author is the one lecturer who holds the session; only session 7 names both.
- Each lecturer introduces themselves in their first session: Silas in session 1, Lukas in session 2.
- Exercises and worked examples are part of every content session. Short "Aufgabe" slides (5 minutes, alone then in pairs; sessions 2 and 4) and a large design task with the sequence Aufgabe, Ziel, Besprechung, "eine mögliche Lösung", "die Entscheidungen dahinter" (20 minutes; sessions 3 and 5). Budget them in the time plan on top of the slides, roughly 2 minutes per slide.
- Each session carries a running example (Cal.com in 2 and 4, the Anmeldesystem in 3 and 5) and shows it in diagrams. Prefer PlantUML blocks over images.
- Sessions 1 to 4 have been held and are the pattern for structure, density and exercises.
- Images and diagrams go to `Material/Slides/media/`; PlantUML code blocks are rendered by the `pandoc-plantuml` filter.
- Build: `./build.ps1 <file>` for one file, `./build.ps1` for everything. CI builds on every push and creates a release on the default branch.

## Literature

Module handbook, official. Sent to the Studiengangsleitung in June 2026, cited on the "Literatur: Modulhandbuch" slide in session 1:

- Richards, Ford: Fundamentals of Software Architecture: An Engineering Approach. O'Reilly, 2020 (sessions 3, 5)
- Fowler: Patterns of Enterprise Application Architecture. Addison-Wesley, 2013, 19th printing (sessions 1, 2, 5)
- Ford et al.: Software Architecture: The Hard Parts: Modern Trade-off Analyses for Distributed Architectures. O'Reilly, 2021 (sessions 2, 3, 5)

Further sources, referenced where they fit:

- Kleppmann, Riccomini: Designing Data-Intensive Applications, 2nd ed. (sessions 1, 2)
- Nygard: Release It!, 2nd ed. (session 4)
- Beyer et al.: Site Reliability Engineering, free at sre.google (sessions 4, 6)
- Huyen: AI Engineering (AI engineering sections)
- Xu: System Design Interview (session 1 worked example)

## Open

- [ ] Exam: format, duration, aids, weighting, date (TODO markers in sessions 1 and 7). Session 7 is scheduled one or two weeks before it.
- [ ] Session 1 has too few pictures: add diagrams or charts to the building-block slides (load balancer, CDN, caching, DNS, APIs, queues) for the next run; latency table still open
- [ ] Session 5 is a draft: review the slides, prune or extend, then set status to review
- [ ] Session 6: pick the stack for the observability demo

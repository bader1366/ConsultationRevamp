# 22 — Architecture Decisions and Open Decisions
**Platform:** Nwafeth Intelligence — منصة نوافث لذكاء الجلسات الاستشارية (Monsha'at Advisory Session Intelligence Platform)
**Status:** Draft for owner review · **Date:** 2026-08-02 · **Author:** Planning package (Fable 5)
**Depends on:** 04–20 (every document cites ADR/OD IDs owned here), 21 (needed-by phases) · **Feeds:** 00 (approval table), 01 (leadership decisions), 23 (handoff guardrails)
**Sources used:** GREENFIELD §22, §24, §7, §2.5; MASTER_PROMPT §2.4, §4.4, §5.3, E.15; CORE-BRIEF §7–§12; docs 05 §2.7, 07 §TD, 08 §15, 13 §9.6, 14 §roles, 15 §10, 16 §4/§9, 18 §format

This document is the **single registry** for architecture decision records (ADR-0001…ADR-0020) and open decisions (OD-01…OD-25). Every other document cites these IDs; where another document's local usage collided on an ID during parallel authoring, §3.1 resolves the collision authoritatively.

---

## 1. ADR register — index and status

All twenty ADRs carry status **recommended-for-acceptance**: they are planning recommendations awaiting the owner's en-bloc or itemized acceptance at the implementation start gate (00-INDEX). None is "accepted" yet — that word is reserved for the owner's signature [DECISION GREENFIELD §24 claim discipline].

| ADR | Title | Argued in | Invariants served |
|---|---|---|---|
| 0001 | Greenfield rebuild; legacy runtime independence | 02, 07 | I10 |
| 0002 | Modular monolith, two runtimes | 07 | I1 |
| 0003 | PostgreSQL + pgvector, single DB, multi-schema | 07, 08 | I12, I13 |
| 0004 | Alembic-only migrations | 07, 08 | (F3 lesson) |
| 0005 | Postgres-backed job queue | 07, 09 | I1, I16 |
| 0006 | Canonical `advisory_session` hub + explicit identity resolution | 08, 09 | I8 (coverage truth) |
| 0007 | Immutable provider-versioned transcripts; `TranscriptSource` + `rebase_transcript`; no ASR correction | 06, 08 | I6, I7 |
| 0008 | Four serving lanes; deterministic Lane 0 | 12 | I5, I18 |
| 0009 | Semantic metric layer + closed-schema compiler | 11 | I2, I3, I4 |
| 0010 | Deterministic verifier gates (R6 numeric, R7 quote) | 10, 12 | I18, I7 |
| 0011 | Versioned taxonomies; frozen packs + live view; reissue-never-edit | 04, 10 | I13 |
| 0012 | Single propose→review→approve workflow | 15, 18 | (MASTER_PROMPT §4.4) |
| 0013 | Groq primary inference; model registry; role-based selection | 14 | I17 |
| 0014 | Local embeddings; filter-first retrieval; RAG optional, non-numeric | 13, 14 | I9 |
| 0015 | Lane-3 job factory (map/verify/reduce) | 13 | I3, I16 |
| 0016 | One-time checksummed snapshot bootstrap; re-derive findings | 09, 20 | I10 |
| 0017 | OIDC/SSO + server-side RBAC; aggregate-vs-transcript split | 16 | I14, I15 |
| 0018 | React+TS RTL frontend; self-hosted assets; structured-answer rendering | 07, 18 | I11 |
| 0019 | Observability stack; reconstructable requests | 19 | I16 |
| 0020 | Correctness-over-cost doctrine with measured spend | 14, 21 | (GREENFIELD §2.5) |

---

## 2. Architecture Decision Records

### ADR-0001 — Greenfield rebuild with total legacy runtime independence
**Status:** recommended-for-acceptance.
**Context.** The legacy system's defects are structural, not cosmetic: eight unsynchronized intent registries (F5/F8), HTML welded to data with DOM-scraping export (F6), a violations table with no PK and 3,103 duplicate rows (F1), period injection defeated by the model (F16/ISS-01), silent degrade to different answers (ISS-02), and a schema not rebuildable from source (F3) [FACT arch/08]. The owner has decided the replacement is a new platform, not a refactor [DECISION GREENFIELD §2.1].
**Decision.** Build Nwafeth Intelligence as a new system. The only artifact crossing the boundary is a one-time checksummed snapshot restored read-only into `legacy_snapshot` (ADR-0016). The legacy system is never a runtime dependency: no shared DB, no shared credentials (OD-13: separate Read.ai OAuth client), no API calls in either direction (I10).
**Alternatives considered.**
- *Incremental refactor in place* (the prior MASTER_PROMPT programme) — rejected: superseded by owner decision, and the F3 non-rebuildable schema makes in-place migration discipline unenforceable.
- *Strangler-fig around the legacy app* — rejected: strangler requires trusting the legacy surface as an interim contract, but that surface returns wrong answers today (ISS-02); the interim would institutionalize errors.
**Consequences.**
- Double-run cost during the P6 parallel run (accepted, bounded).
- A full re-derivation bill for ~409k legacy knowledge rows (deliberate — ADR-0016).
- Operational: CI structurally blocks legacy connection strings outside bootstrap code; legacy containment and retirement become explicit P7 deliverables with owner-decided dates.
**Revisit trigger.** None for the independence principle (it is invariant I10). The retirement *date* is owner-owned (doc 21 P7-E7).

### ADR-0002 — Modular monolith, one repo, two runtimes
**Status:** recommended-for-acceptance.
**Context.** Scale is modest and known: 16,911 sessions / 399,501 turns over 14 months, monthly peak 1,512 [FACT CORE-BRIEF §11]. The deployment target is a restricted Saudi government environment where every extra network service is an approval and an attack surface [ASSUME OD-01]. The legacy failure was not "too monolithic" — it was god-modules without boundaries (F5).
**Decision.** One repository, one codebase, two runtimes: a web API process (serving plane, read-only on analytical schemas) and a worker process (enrichment/jobs plane, owns writes) — I1 expressed as process separation, not service separation. Module boundaries (`ingest`, `core`, `findings`, `serve`, …) enforced by import-linting in CI; a violated boundary fails the build.
**Alternatives considered.**
- *Microservices per plane* — rejected: at this scale they add network failure modes, distributed-tracing burden, and environment approvals for zero elasticity benefit ("do not create microservices merely to draw a complex diagram" [FACT GREENFIELD §22]).
- *Single runtime with background threads* — rejected: reproduces module-global state hazards (F13) and couples serving latency to enrichment load, violating I1.
**Consequences.**
- Deploys are lockstep — acceptable at pilot scale and it simplifies API versioning (doc 17).
- One CI quality bar for everything; no cross-service contract drift by construction.
- Scaling is vertical first; horizontal worker scale-out is a queue-consumer count change, not a re-architecture.
**Revisit trigger.** Sustained multi-node scale-out (>3 worker nodes) or a second consuming organization → re-evaluate splitting the worker plane into its own deployable.

### ADR-0003 — PostgreSQL 16/17 + pgvector, single database, multiple schemas
**Status:** recommended-for-acceptance.
**Context.** The platform needs relational facts, JSON payload archives, a job queue, durable agent state, audit, and vector retrieval. Every additional datastore in a government deployment is procurement + patching + backup + residency review.
**Decision.** One PostgreSQL instance carrying the eleven schemas of CORE-BRIEF §6 (`ingest`, `core`, `transcript`, `findings`, `tax`, `serve`, `jobs`, `packs`, `evidence`, `ops`, `legacy_snapshot`), pgvector for embeddings, separate roles/pools per plane (web read-only on analytical schemas; workers write). Analytical strategy: same instance at pilot scale with statement timeouts and `work_mem` discipline; pre-aggregated marts/views for anything BI-shaped (OD-22).
**Alternatives considered.**
- *Separate OLAP store (ClickHouse/DuckDB)* — rejected for now: the corpus fits Postgres comfortably; premature bifurcation splits provenance across engines.
- *Dedicated vector DB (Qdrant/Milvus)* — rejected: pgvector with filter-first exact scan matches the retrieval pattern (MASTER_PROMPT §7.1); a second store breaks the "scope filter is a SQL WHERE" property that keeps retrieval auditable.
- *MySQL* — rejected: weaker JSONB, no pgvector, no transactional DDL.
**Consequences.**
- One backup/restore story (pgBackRest), one residency review, one DR drill target.
- Queue + agent state + facts share transactions → exactly-once job transitions are a `COMMIT`, not a distributed protocol.
- The DB is the crown jewel: DR drills are non-optional (doc 19); growth path is monthly partitioning (`transcript.turn`) before any engine change.
**Revisit trigger.** `evidence` exact-scan p95 over the doc 13 latency budget at >5M vectors, or analytical interference with serving despite pooling → read replica first, engine split second.

### ADR-0004 — Alembic as the only migration mechanism from day one
**Status:** recommended-for-acceptance.
**Context.** The legacy schema could not be rebuilt from source; environments drifted; a binary dump was the de-facto schema definition (F3) [FACT arch/03].
**Decision.** Every schema object — tables, constraints, roles, grants, views, extensions — exists only as Alembic migrations in the repo. CI proves `0001→head→0001` on a fixture DB on every commit. No manual DDL in any environment; break-glass DDL requires a same-day backfill migration plus an audit entry.
**Alternatives considered.**
- *Declarative diff tools* (autogenerate-only, Atlas) — autogenerate is a draft aid, never trusted for data migrations or grants; Atlas rejected as an extra tool with no gain over disciplined Alembic.
- *"SQL files now, migrations later"* — rejected: that is exactly how F3 happened.
**Consequences.**
- Grants and masked views become code-reviewable; doc 16's G-SEC-2 diffs `information_schema` grants against the matrix in CI.
- Downgrade paths cost authoring time — accepted: they are the rollback mechanism doc 21's phases lean on.
- Fixture-DB migration tests keep the schema honest against the ERD (doc 08).
**Revisit trigger.** None realistic; if Alembic stalls upstream, migrate the discipline to an equivalent tool, never abandon the principle.

### ADR-0005 — Postgres-backed job queue (Procrastinate or equivalent)
**Status:** recommended-for-acceptance.
**Context.** Workers need retries, scheduling, kill/resume, and exactly-once-ish state transitions for Lane-3 partitions. The legacy pipeline had opt-in correctness flags, a wrapper contradicting its runbook (F10), and unbounded LLM fan-out swallowed to None.
**Decision.** A Postgres-native queue (recommend **Procrastinate**: async, LISTEN/NOTIFY, job table visible to plain SQL) in the worker runtime; periodic work (packs, retention, catalogue polling) via the same scheduler. Job state lives in the same DB as domain state, so a partition's completion and its findings commit atomically.
**Alternatives considered.**
- *Celery + Redis* — rejected: adds Redis as infrastructure and a second delivery-semantics domain; Redis is cache-only-later in this stack, never system of record.
- *Temporal* — rejected: operationally heavy for a restricted environment; its replay model is overkill for month-partitioned map/verify/reduce.
- *cron + scripts* — rejected: F10 redux, no retries, no observability.
**Consequences.**
- Queue throughput bounded by Postgres — irrelevant at thousands of jobs/day.
- Job observability is a SQL query; doc 19 dashboards read the queue table directly.
- Kill/resume (T-13) becomes a row-state design problem — exactly where it is testable.
**Revisit trigger.** Sustained >50 jobs/sec enqueue or multi-region workers; or Procrastinate maintenance stalling (fallback: an in-house table-based worker with identical semantics).

### ADR-0006 — Canonical `advisory_session` hub with explicit identity resolution
**Status:** recommended-for-acceptance.
**Context.** Three identity spaces exist: provider meetings, internal system sessions, consultant directory refs. Legacy bridged them with a TRUNCATE-and-rebuild bridge, positional Excel parsing, and the magic string `'PENDING'` [FACT arch/02 §7]; the bridge matched ~8% of report rows [FACT CORE-BRIEF §11].
**Decision.** `core.advisory_session` is the canonical hub (never "meeting"). Provider meetings map via `provider_meeting_map`; internal sessions via `internal_session_map`. Unresolved identity = nullable FK + `resolution_status` enum + provenance columns. Resolution rungs are deterministic-first (M1/M2); fuzzy M3 is measured and queued for steward review, never auto-accepted (doc 09). Ambiguity is a first-class review-portal queue.
**Alternatives considered.**
- *Provider-meeting-centric model* — rejected: sessions without transcripts (cancellations, no-shows — CAP-D4) would be second-class entities.
- *Magic-value placeholders* — rejected by name; the `'PENDING'` lesson is why `resolution_status` exists.
- *Probabilistic auto-merge above a score* — rejected at launch: a wrong merge poisons consultant-level analytics; precision-over-recall applies to identity too (E.0 spirit).
**Consequences.**
- Every aggregate declares `used ≤ matched ≤ total` honestly (I8); historical low-match months surface as declared coverage floors, not hidden joins.
- Steward workload exists by design, bounded by EXP-01's ambiguity-rate ≤ 0.5% gate.
- Crosswalk tables become the join spine of the D-family capabilities.
**Revisit trigger.** EXP-01 live-cohort results; deterministic match ≥ 0.99 sustained 3 months → the M3 fuzzy rung can be retired.

### ADR-0007 — Immutable, provider-versioned transcripts; `TranscriptSource` boundary; `rebase_transcript`; no ASR correction ever
**Status:** recommended-for-acceptance.
**Context.** Read.ai is interim [DECISION MASTER_PROMPT §2.5]; a replacement provider is expected (OD-06). The legacy system rewrote transcripts via ASR "correction", destroying quote verifiability; its removal is a settled owner decision [DECISION MASTER_PROMPT §2.4].
**Decision.** Raw transcripts are immutable and provider-versioned (I6): `transcript.source` rows per (provider, version), append-only turns, an active-source pointer per session. Providers integrate behind a `TranscriptSource` adapter contract (doc 06 §3). Switching sources is `rebase_transcript`: attach new source → re-run extraction against it → switch the pointer → retain the old source per retention policy [ASSUME OD-08]. Findings always stamp `(transcript_source, source_version)` (I13). No component ever edits transcript text — there is no correction table, no cleanup pass, nothing that "fixes" ASR output.
**Alternatives considered.**
- *Single mutable transcript with quality patches* — rejected: breaks I7's exact-substring guarantee and re-opens the ASR-correction door the owner closed.
- *Keep only the best source per session* — rejected: destroys the audit trail EXP-02/T-08 need and makes rebase irreversible.
**Consequences.**
- Storage grows with sources — bounded (≈ two sources × corpus), archived per OD-08.
- Every quote-bearing surface carries source+version, enforced by NOT NULL provenance columns.
- Rebase becomes a routine, drilled operation (T-08), not a migration crisis; the provider decision (OD-06) stops being scary.
**Revisit trigger.** None for immutability (I6). Storage retention policy revisits with OD-08.

### ADR-0008 — Four serving lanes with deterministic Lane 0 for committed capabilities
**Status:** recommended-for-acceptance.
**Context.** Committed questions (CAP-A1…CAP-D10) must be answered identically every time; the legacy let an LLM improvise routing and silently answer different questions (ISS-02, F16).
**Decision.** Lane 0: deterministic matcher — high-precision, per-capability τ/δ calibrated on DS-02 (starting τ₀ = 0.82, δ₀ = 0.06), abstains when unsure — then registered capability execution. Lane 1: bounded agent (≤6 steps, ≤8 model calls, one tool per step, closed schemas) over the 11-tool governed toolbelt. Lane 2: clarification (≤4 closed registry options) or honest boundary via the closed reason-code enum (CORE-BRIEF §4). Lane 3: async deep-analysis jobs (ADR-0015). Lane choice is deterministic and logged; no lane ever silently substitutes another question (I5).
**Alternatives considered.**
- *Agent-first for everything* — rejected: burns model calls on questions with known deterministic answers and makes committed answers non-reproducible run to run.
- *LLM intent classifier as router* — rejected: routing is a correctness gate and correctness gates are deterministic (I18); the model may propose, deterministic code accepts.
- *Two lanes only (deterministic + agent)* — rejected: without Lane 2, uncertainty degrades into guessing; without Lane 3, long-tail corpus questions are refused forever.
**Consequences.**
- A registry-driven routing table to govern (I12) and a standing τ/δ calibration duty (EXP-05 rerun as DS-02 grows).
- Clarification rate becomes a watched product metric — rising rate signals vocabulary drift, not failure.
- Budgets make worst-case Lane-1 latency and spend computable in advance.
**Revisit trigger.** EXP-05 confusion matrix showing systematic near-miss families → matcher feature work; the ≤6/≤8 budgets move only on DS-09 replay-trace evidence.

### ADR-0009 — Semantic metric layer with closed-schema compiler; the model never writes SQL
**Status:** recommended-for-acceptance.
**Context.** I2/I3 forbid model-authored SQL and model-computed numbers. The legacy injected periods that the LLM could override (F16/ISS-01), and hand-built variants of the same metric drifted apart (F5).
**Decision.** One governed metric/dimension registry (I12: unregistered ⇒ build fails). Queries are `MetricSpec` documents (metric, dimensions, filters, period, comparison) validated against closed JSON Schemas; a deterministic compiler renders SQL from reviewed templates and allow-listed fragments. The period is resolved by the harness and injected — it does not exist in any model-visible schema (I4). EXP-06 proves compiler equality against independently written reference queries before any number reaches any user. Suppression (n≥30, Wilson CI) and coverage scaffolding are emitted by the compiler, not bolted on by callers.
**Alternatives considered.**
- *Text-to-SQL with guardrails* — rejected outright: guardrails on a generative SQL author are probabilistic; I2 demands structural impossibility.
- *Off-the-shelf semantic layer (dbt metrics / Cube)* — rejected: external dependency, weaker Arabic/period semantics, and the coverage/suppression scaffolding must be inline, not adjacent.
- *Hand-written query per capability* — rejected: 26 capabilities × dimensions × periods explodes combinatorially; inter-variant drift is F5 again.
**Consequences.**
- New metrics require registry PRs — deliberate friction with a named reviewer (TL).
- The grammar stays closed: genuinely novel analytical shapes go to Lane 3, never into grammar creep.
- EXP-06's cross-product harness becomes a permanent regression suite (T-01 family).
**Revisit trigger.** A third consecutive capability needing a grammar extension → a compiler design review instead of a third ad-hoc extension.

### ADR-0010 — Deterministic verifier gates: R6 numeric provenance, R7 quote verification
**Status:** recommended-for-acceptance.
**Context.** Model self-confidence is not a gate (I18). The composer writes Arabic narrative; nothing stops a fluent model from inventing a number or "improving" a quote — except deterministic code between the model and the user.
**Decision.** Every envelope carries machine-readable allowed-literals (every number the narrative may mention, with formatting variants) and quote manifests. A deterministic verifier rejects any narrative containing numerals outside allowed-literals (R6) or quotes failing exact-substring match against the active transcript turn (R7, using the doc 10 normalization canon: digit forms, tatweel, diacritics). Rejection ⇒ `VERIFIER_REJECTED` + safe template fallback + telemetry — never silent regeneration loops (bounded retries: 1).
**Alternatives considered.**
- *LLM-judge verification* — rejected: I18 by definition.
- *Constrained decoding of the narrative itself* — rejected: cannot express "any wording, only these facts"; strict schemas govern structured outputs, narrative needs post-hoc gating.
- *No narrative (tables only)* — rejected: executive personas require Arabic narrative (doc 03); the gate makes narrative safe rather than banning it.
**Consequences.**
- Verifier false-positives surface as fallback renders — measured in EXP-09, tuned by expanding literal formatting variants, never by relaxing the gate.
- Every rejection is observable and attributable (I16 + R13 reasoning log).
- The gate doubles as the anti-hallucination proof for auditors: rejection is demonstrable by injection (EXP-09 campaign kept as a permanent suite).
**Revisit trigger.** Fallback-render rate > 5% of Lane-0/1 answers over two weeks → variant expansion work, not gate relaxation.

### ADR-0011 — Versioned taxonomies; frozen packs + live view; reissue-never-edit
**Status:** recommended-for-acceptance.
**Context.** Examples are seeds, not closed vocabularies (R-P1); categories are introduced, merged, and split continuously. Published numbers must never mutate under later taxonomy edits [FACT GREENFIELD §6.4], yet quarterly strategy questions need current-taxonomy recomputation.
**Decision.** Taxonomies (`VIOL`,`CHAL`,`SAT`,`DEC`,`IMP`,`ENT`,`QST`) are versioned with lineage edges (introduce/merge/split); every finding stamps `taxonomy_version` NOT NULL (I13). Publications freeze: a pack snapshots its finding rows and renders immutably; corrections are supersession/retraction rows producing a *reissued* pack — never an edit. Live views recompute under current taxonomy and are labelled as live. History is deliberately countable both ways (as-published vs as-current — MASTER_PROMPT §4.3).
**Alternatives considered.**
- *Single evolving taxonomy with re-labelled history* — rejected: silently changes published numbers, the exact trust-killer §6.4 forbids.
- *Frozen-only, no live view* — rejected: strategic questions need current-taxonomy recomputation across old periods.
- *Pack edits with changelog* — rejected: an edited artifact with a changelog is still a mutated artifact; sign-off (OD-10) must bind to immutable bytes.
**Consequences.**
- Two counting modes must be explained in UX — doc 18 owns the Arabic framing («كما نُشر» / «كما هو الآن»).
- Pack snapshots cost storage (small: finding refs + rendered artifacts in MinIO).
- Mechanical, scoped answer-key staleness (MASTER_PROMPT §4.4) becomes implementable because packs store category-id-bearing rows.
**Revisit trigger.** None for immutability. Lineage-query performance revisits if versions exceed ~100 (unlikely at monthly review cadence).

### ADR-0012 — One propose→review→approve workflow for everything reviewable
**Status:** recommended-for-acceptance.
**Context.** Five review streams exist — answer keys, taxonomy edges, cluster canonical labels, violation confirmations, entity aliases — plus pack publication approval. The tempting path (a spreadsheet here, an admin page there) creates parallel inconsistent audit trails; MASTER_PROMPT §4.4(3) explicitly forbids the throwaway spreadsheet-then-portal path.
**Decision.** A single review workflow engine (portal UX in doc 18; registry tables in doc 15 §1): typed payloads, generated candidates, approve/correct/reject with append-only history, reviewer RBAC [ASSUME OD-11], mechanical staleness propagation, per-item audit. All streams are payload plugins on this one engine. Built in P0, first real payloads in P2 (doc 21 §1.1 argues the early build).
**Alternatives considered.**
- *Per-stream tools* — rejected: F5-style divergence, five audit stories, five RBAC surfaces.
- *Off-the-shelf labelling platform (Label Studio etc.)* — rejected: cannot enforce our stamps (taxonomy_version, corpus_snapshot, code_version), lives outside our RBAC/audit, and PII rules (I15) would need a second enforcement surface.
- *Git-based review (YAML PRs)* — rejected on owner ergonomics: the product owner reviews Arabic candidates, not diffs.
**Consequences.**
- The portal sits on the critical path early — accepted deliberately; it is also the owner-habit-forming tool.
- One audit model answers every "who approved this" compliance question.
- Every new reviewable artifact class is a plugin, not a project.
**Revisit trigger.** A review stream whose payload genuinely cannot fit the engine (none identified) — extend the engine; fork only with a written exception recorded here.

### ADR-0013 — Groq as primary inference; model registry; role-based selection; gpt-oss-120b/20b strict-schema defaults
**Status:** recommended-for-acceptance.
**Context.** Groq is the owner's primary generative-AI infrastructure [DECISION GREENFIELD §2.4]. Verified catalogue 2026-08-02: strict JSON-Schema constrained decoding only on `openai/gpt-oss-120b` and `openai/gpt-oss-20b`; `llama-3.3-70b-versatile` deprecates 2026-08-16 for non-enterprise tiers; legacy models kimi-k2 and llama-4-scout are gone from the catalogue [FACT groq-docs 2026-08-02].
**Decision.** All generative calls route through one Groq client — a single egress choke point carrying the outbound payload gate (doc 16 §4). Role-based selection from `ops.model_registry`:
- per-session structured extraction, Lane-3 map, Lane-1 planner, composer ⇒ `openai/gpt-oss-120b` with strict json_schema;
- routing/normalization/high-volume triage ⇒ `openai/gpt-oss-20b`;
- safety/policy screening ⇒ `gpt-oss-safeguard-20b`, benchmarked with a 120b prompt-based fallback (preview caveat);
- no production role for preview or deprecating models (`qwen3.6-27b`, `llama-3.3-70b`) [REC doc 14];
- Batch API (50% discount, 24h–7d window) for backfills/re-extractions/benchmarks only — never user-accepted Lane-3 jobs.
Every call logs model id, prompt sha, tokens, cost, latency (I17); a deprecation notice creates an ops task with replay evaluation before any swap.
**Alternatives considered.**
- *Multi-provider abstraction now* — rejected: the registry IS the abstraction; a second provider is a registry row + client adapter when actually needed, and the owner decision names Groq.
- *Self-hosted large open models* — **rejected outright [DECISION owner 2026-08-03: no self-hosted generative models; the environment is CPU-only with no GPU and none planned]**. If OD-04 fails there is no standing local fallback: the escalation is a KSA-region/sovereign Groq option or a *new* GPU-procurement programme decision (doc 16 §9.1).
- *Keep llama-3.3-70b hoping for enterprise tier* — rejected: the plan must not depend on unverified tier status (OD-03).
**Consequences.**
- Strict-schema requirements shape all extraction schemas (all fields `required`, `additionalProperties: false`).
- Model swaps are registry operations with benchmark gates, not code edits; EXP-03 is re-runnable by design.
- The single client is where OD-04 policy, pseudonymization, and cost telemetry all live — one place to audit.
**Revisit trigger.** Any pinned model's deprecation notice; a Groq KSA-region/sovereign offering (re-run doc 16 §4 matrix); EXP-03 evidence that a finding family needs different routing.

### ADR-0014 — Local embeddings; filter-first retrieval; RAG optional and never numeric
**Status:** recommended-for-acceptance.
**Context.** Groq offers no embedding models [FACT groq-docs 2026-08-02], making local embedding necessary — and a data-residency win: transcript text for retrieval never leaves the environment. Legacy RAG leaked retrieval content into numeric answers.
**Decision.** Embeddings/reranking run as a local service in the worker plane; candidates benchmarked in EXP-04 (`BAAI/bge-m3` favoured vs `intfloat/multilingual-e5-large` vs legacy L12 baseline; optional reranker `bge-reranker-v2-m3`). Retrieval is filter-first: deterministic SQL scoping (period, programme, role, quality tier) then exact vector scan under the cap, ANN only above it (MASTER_PROMPT §7.1). Embedding tables carry model+dim provenance (table-per-model-generation, doc 08). RAG augments evidence and narrative colour only; every number still comes from the semantic layer or stored findings (I9) — enforced by the R6 gate and proven by the RAG-on/off numeric-parity test.
**Alternatives considered.**
- *External embedding APIs* — rejected: expands outbound transcript flow for zero demonstrated quality gain, complicating OD-04.
- *ANN-first (HNSW everywhere)* — rejected at this scale: exact scan under filters is auditable and recall-perfect; ANN is an admitted optimization above the cap, with its recall delta published.
- *RAG as an answer source* — rejected by I9, categorically.
**Consequences.**
- A worker-plane sizing question (bounded: bge-m3 is CPU-viable at our throughput — ~2.2–3.7 h full-corpus pass on 8 CPU threads, doc 19 §capacity; **CPU-only by owner decision 2026-08-03, no GPU**).
- Index rebuilds are routine derived-data operations — the index is disposable by design.
- Retrieval quality is measured per backend (QT-09); an unmeasured backend cannot be default [DECISION MASTER_PROMPT §11].
**Revisit trigger.** EXP-04/EXP-07 pin the model and backend; corpus > ~5M evidence units → ANN default with published recall delta.

### ADR-0015 — Lane-3 on-demand deep-analysis factory (map→verify→reduce)
**Status:** recommended-for-acceptance.
**Context.** Questions outside committed capabilities and stored findings must be answerable from raw transcripts + raw structured data. The legacy's retired-/v3 lesson: unbounded, unresumable, unverifiable fan-out swallowed to None [FACT MASTER_PROMPT §5.3].
**Decision.** Lane-3 jobs: question fingerprint + corpus manifest → month-partitioned **map** (per-session strict-schema extraction) → deterministic **verify** (R7 quote gate, schema/enum checks) → deterministic **reduce** (SQL aggregation — the model NEVER aggregates, I3). Jobs carry id, progress, kill/resume/cancel; a failed partition is `INCOMPLETE`, never silent (I16); artifacts cache by (fingerprint, manifest) and can be promoted into the governed registry. No cost ceiling [DECISION MASTER_PROMPT §5.3 + GREENFIELD §2.5]; concurrency bounded only for rate limits. `start_deep_analysis`/`get_analysis_job` are orchestrator-controlled; the model proposes via `declare_unanswerable(DEEP_JOB_OFFERED)` (doc 12 argues the split).
**Alternatives considered.**
- *Lane 1 looping over the corpus synchronously* — rejected: budget-bounded agents cannot honestly process a month of transcripts; timeouts would masquerade as answers.
- *Model-side aggregation of mapped findings* — rejected by I3; reduce is SQL over verified rows.
- *Cost-capped sampling* — rejected by GREENFIELD §2.5: full coverage, measured spend.
**Consequences.**
- A human approval step for generated analysis schemas — human-in-loop is the design, not a workaround.
- The artifact cache makes repeated questions nearly free; the promotion loop feeds the registry instead of spawning shadow capabilities (I12).
- Rate-limit engineering (TPM budgets, queue shaping) becomes explicit, measured work (EXP-08).
**Revisit trigger.** EXP-08 rate-limit measurements set `DEEP_JOB_CONCURRENCY`; micro-batching rules move only on per-partition quality evidence (doc 13).

### ADR-0016 — One-time checksummed snapshot bootstrap; re-derive findings, never migrate derived claims
**Status:** recommended-for-acceptance.
**Context.** ~409k legacy knowledge rows exist under an unversioned taxonomy, extracted by now-delisted models, with known defects: duplicate violations at ~1.84× inflation, circular labels, empty C2 columns [FACT CORE-BRIEF §11/§12]. Migrating them would launder untrusted claims into a trusted system.
**Decision.** One checksummed snapshot (SRC-SNAP) restored read-only into `legacy_snapshot`. Raw/authoritative layers (sessions, transcripts, ratings, operational facts) bootstrap into canonical schemas via idempotent, reconciled ETL with quote-level checksum verification (doc 09). Derived layers (findings, clusters, taxonomies) are NEVER migrated — they are re-derived by the new pipeline under the new taxonomy in P2. Legacy derived rows remain queryable in `legacy_snapshot` solely as parallel-run comparison evidence.
**Alternatives considered.**
- *Migrate findings with a quality flag* — rejected: a flag fixes neither inflation nor circular labels, and every downstream aggregate would need dual code paths forever.
- *Live sync until cutover* — rejected by I10: one snapshot, one checksum, one boundary.
- *Discard legacy derived data entirely* — rejected: it is the only baseline for the P6 six-way difference classification.
**Consequences.**
- A full re-extraction bill — measured, Batch-discounted (doc 14 sizing), and accepted as the price of trustworthy findings (ADR-0020).
- Bootstrap idempotency (delete-by-run-id, re-run) is a tested property (P1 fixture), not a hope.
- Historical answers may legitimately differ from legacy answers; the difference taxonomy (GREENFIELD §19.3) is the communication tool.
**Revisit trigger.** None; the snapshot is one-time by definition. A second snapshot would require a new OD with written justification.

### ADR-0017 — OIDC/SSO with server-side RBAC; aggregate-vs-transcript access split
**Status:** recommended-for-acceptance.
**Context.** Legacy ran unauthenticated surfaces with CORS * (F2/ISS-04). The product serves executives (aggregates) and reviewers/stewards (evidence) with fundamentally different data sensitivity; PDPL minimization applies (doc 16 §9).
**Decision.** OIDC against the authority's approved IdP (Keycloak broker interim [ASSUME OD-02]); all authorization server-side (I14): deny-by-default route policy, seven roles (executive_viewer, analyst, service_owner, reviewer, steward, admin, pipeline_service), DB-level enforcement via role-separated pools, masked views, and column grants. The load-bearing rule: **seeing an aggregate never implies seeing its evidence** — transcript windows require transcript-scoped roles, are size-limited server-side (SECURITY DEFINER function, doc 16 §2.4), and write sensitive-access audit events. PII placeholders by default everywhere (I15); steward break-glass is ticketed, time-windowed, audited [ASSUME OD-09].
**Alternatives considered.**
- *App-level checks only* — rejected: one ORM bug from a breach; defense-in-depth requires the DB to also refuse.
- *Frontend-enforced visibility* — rejected by I14 (the legacy lesson in one clause).
- *Single "user" role at pilot* — rejected: the aggregate/evidence split is a data-protection requirement, not a UX preference.
**Consequences.**
- The role-grant matrix is Alembic-managed and CI-diffed (G-SEC-2); the authorization matrix is a permanent test (T-16).
- Role changes ship as reviewed migrations — onboarding friction accepted.
- Break-glass usage is a standing owner report (P7 security review).
**Revisit trigger.** IdP mandate change (national SSO profile); any EXP-10/pen-test finding; OD-09/OD-14 widening decisions re-run the matrix.

### ADR-0018 — React + TypeScript RTL frontend; self-hosted assets; structured-answer rendering
**Status:** recommended-for-acceptance.
**Context.** Arabic-first UI (فصحى، سجل حكومي سعودي) with RTL layout is a first-class requirement; the government network blocks CDNs — legacy export libraries failed exactly this way; and legacy welded HTML to data, exporting by DOM-scraping (F6).
**Decision.** React + TypeScript + Vite SPA, RTL-first (logical CSS properties, RTL-audited components), all assets self-hosted, Arabic string catalogue (i18n infrastructure even while ar-only), self-hosted charts. The frontend renders **typed answer envelopes only** (I11): narrative, tables, coverage blocks, evidence refs are envelope fields. Exports (XLSX/JSON) are server-side renderers from the same envelope — the DOM is never a data source.
**Alternatives considered.**
- *Server-rendered templates (Jinja)* — rejected: the review portal and conversation UX are interaction-heavy; SPA + the doc 17 typed API is the better fit.
- *Next.js SSR* — rejected: adds a Node server runtime to a restricted environment for SEO/TTFB benefits an internal tool doesn't need.
- *Off-the-shelf BI frontend* — rejected: cannot render coverage/provenance semantics or review workflows; BI exposure is its own decision (OD-22).
**Consequences.**
- An API-first contract for every pixel (doc 17); envelope schema versioning discipline.
- RTL rendering is CI-tested (visual snapshots for pack renders).
- Export correctness reduces to "renderer reads envelope" — testable byte-for-byte (T-render).
**Revisit trigger.** A mandated authority design system or portal-embedding requirement → re-evaluate the shell, keep envelope rendering.

### ADR-0019 — Observability stack with end-to-end reconstructable requests
**Status:** recommended-for-acceptance.
**Context.** Legacy telemetry could not attribute failures (ISS-15/16), healthchecks lied (ISS-06), error rates lied (ISS-05), failures returned 200s. I16 demands observable failure and degradation.
**Decision.** structlog JSON logs + OpenTelemetry traces + Prometheus metrics + Grafana + Loki; request-id propagated end-to-end (web → planner → tools → workers → model calls). Every model call logs model id, prompt sha, tokens, cost, latency. Every answer is reconstructable: envelope + tool-call log + verifier verdicts + reasoning log (R13) keyed by request-id. No raw exception reaches a user; no 200-on-failure; partial results are structurally marked incomplete.
**Alternatives considered.**
- *Managed APM (Datadog etc.)* — rejected: egress + procurement in the restricted environment; the OSS stack deploys inside it.
- *Logs-only minimalism* — rejected: ISS-15/16 proved uncorrelated logs cannot answer "why did this answer say that".
**Consequences.**
- SRE owns dashboard/alert curation from P0; alert quality is a P7 exit criterion (no crying-wolf alerts).
- Trace/log storage participates in OD-08 retention decisions.
- The reasoning log doubles as parallel-run evidence and audit substrate.
**Revisit trigger.** Authority-mandated SIEM (OD-16) → add log shipping, never replace local observability; trace volume cost → sample traces only (never audit events).

### ADR-0020 — Correctness-over-cost doctrine with measured-and-reported spend
**Status:** recommended-for-acceptance.
**Context.** The owner decided correctness outranks token cost [DECISION GREENFIELD §2.5]; the legacy sampled and truncated its way to cheap wrong answers. Unmeasured spend is its own failure mode.
**Decision.** No coverage sampling for cost reasons anywhere: full-corpus extraction, full-scope Lane-3 jobs (no cost ceiling [DECISION MASTER_PROMPT §5.3]), full benchmark suites. Cost is engineered via legitimate levers only: Batch API for non-interactive backfills, role-based model selection (20b where quality-proven by EXP-03), artifact caching by fingerprint+manifest, bounded concurrency for rate limits. Every phase and every job reports spend (tokens, cost, wall-clock); the month-close rehearsal (doc 21 P6-E7) publishes the steady-state baseline.
**Alternatives considered.**
- *Per-question cost budgets with sampling fallback* — rejected by the owner decision: a sampled violation scan that misses an accusation is worse than an expensive one.
- *Unmeasured "spend whatever"* — rejected: the doctrine is paired with a measurement duty precisely so leadership sees what correctness costs.
**Consequences.**
- Predictably large one-time costs (backfills, benchmarks) are surfaced to the owner in advance with estimates.
- Rate-limit engineering (TPM budgets, queue shaping) is explicit engineering work, not an afterthought.
- Cost dashboards exist from P0 (Groq client telemetry) — the report is a query, not a project.
**Revisit trigger.** Only the owner can revisit the doctrine (it is their decision); the measurement duty never lapses.

---

## 3. Open Decision register

### 3.1 Numbering authority and collision resolution

During parallel authoring, three documents introduced new ODs under IDs that collided. **This register is authoritative.** The consistency pass must remap the references below; meanings are preserved, only IDs move [REC — alternative: renumber the majority users; rejected because it would touch five documents instead of three].

| Colliding usage | Where it appears | Authoritative ID |
|---|---|---|
| "OD-14" = consultant national-ID handling + evidence-role widening | docs 05 §2.7/§5, 08 §15, 09 §M2, 16 §2/§3 | **OD-14** (kept — majority + data-plane usage) |
| "OD-14" = BI tool mandate + mart exposure | doc 07 §BI; doc 16 §2.4 ("OD-14/BI"); doc 17 §9 | **OD-22** (new ID) |
| "OD-14" = Lane-3 artifact/collection visibility | doc 13 §9.6 and its risk register | **OD-23** (new ID) |
| "OD-19" = Groq contractual data terms (DPA) | doc 16 §4.4 + summary table | **OD-19** (kept — declared with an explicit [NEW] header) |
| "OD-19" = annotation staffing & qualification | doc 15 §10 | **OD-24** (new ID) |

### 3.2 The register

Fields per entry: question · decision owner (role at Monsha'at) · safe working assumption now in force · impact if the assumption is wrong · needed-by phase (doc 21).

**OD-01 — Deployment target.**
- *Question:* Which approved Saudi environment (cloud region / on-prem) hosts NIP?
- *Owner:* Platform owner + IT infrastructure lead.
- *Assumption in force:* container platform (Compose-capable, K8s optional) in an approved environment with S3-compatible storage [ASSUME OD-01].
- *If wrong:* deployment topology (doc 07), backup tooling, and doc 16 residency rows rework; Compose-first design minimizes blast radius.
- *Needed by:* P0 (environment provisioning).

**OD-02 — IdP + secrets platform.**
- *Question:* Which OIDC IdP and which vault/KMS are mandated?
- *Owner:* IT security directorate.
- *Assumption:* OIDC available; Keycloak broker acceptable interim; vault-style secret injection [ASSUME OD-02].
- *If wrong:* auth middleware adapters + rotation runbooks change; the RBAC model itself is IdP-agnostic by design.
- *Needed by:* P0 (login before any UI).

**OD-03 — Monsha'at Groq account tier.**
- *Question:* Enterprise committed-spend contract or not (affects llama-3.3-70b post-2026-08-16 availability and rate limits)?
- *Owner:* Procurement + platform owner.
- *Assumption:* non-enterprise; **nothing in the plan depends on llama-3.3-70b either way** [ASSUME OD-03].
- *If wrong (is enterprise):* strictly more headroom — no plan change; benchmark quotas can rise.
- *Needed by:* P2 (backfill throughput planning); informative earlier.

**OD-04 — Outbound-data approval for Groq inference.**
- *Question:* May pseudonymized transcript text be sent to Groq? Under what conditions (data classes, scrubbing, logging)?
- *Owner:* Compliance office + data steward, deciding on the doc 16 §4 data-flow matrix.
- *Assumption:* approved for pseudonymized text; hard-blocked for raw PII — the payload gate enforces this regardless of the decision [ASSUME OD-04].
- *If wrong (denied):* extraction and Lane-3 cannot run on real text as designed → escalation: sovereign/KSA-region inference or on-prem hosting (a major re-plan); staging pilots on synthetic data continue meanwhile.
- *Needed by:* P2 entry (full-corpus backfill authorization). Filed at P0.

**OD-05 — CAP-C2 route.**
- *Question:* New corpus-wide extraction fields for symptoms/causes/actual-asks, or Lane-3 pilot first? (Legacy columns empty for 0/67,082 rows [FACT MASTER_PROMPT E.9].)
- *Owner:* Product owner.
- *Assumption:* Lane-3 pilot first — recommended as EXP-08's subject — corpus-wide extraction only if the pilot proves the schema [REC docs 04/13].
- *If wrong:* either a wasted pilot (minor) or premature corpus extraction under an unproven schema needing full re-extraction (the costlier error; the assumption deliberately biases against it).
- *Needed by:* P5 (EXP-08 subject selection).

**OD-06 — Replacement transcript provider identity/timeline.**
- *Question:* Which provider replaces Read.ai, and when?
- *Owner:* Product owner + procurement, deciding on EXP-02 evidence (doc 06 §5.5 sheet).
- *Assumption:* unknown; Read.ai continues as interim; every design goes through `TranscriptSource` (ADR-0007) [ASSUME OD-06].
- *If wrong (no candidate ever passes):* the quote-fidelity ceiling measured in EXP-02 persists and is declared on quote-bearing capabilities; STT contingency only if OD-12 flips.
- *Needed by:* P6 (rebase execution slot); the benchmark needs OD-25 at P1.

**OD-07 — Internal session data access mechanism.**
- *Question:* DB replica, API, or export for SRC-INT/DIR/REF/EVAL/OUT — and what freshness SLA?
- *Owner:* Internal-systems owner + data steward.
- *Assumption:* read-only API or scheduled export; daily freshness for sessions, weekly for directory/reference [ASSUME OD-07].
- *If wrong:* adapter rework (bounded — header-mapped ingestion isolates it); worse freshness degrades CAP-C3/D-family recency, declared via I8 stamps rather than hidden.
- *Needed by:* P1 (adapters). Request filed P0.

**OD-08 — Retention windows.**
- *Question:* Retention for raw payloads, superseded transcript sources, conversation logs, exports, audit events?
- *Owner:* Data steward + compliance office.
- *Assumption:* 90d for pre-rebase transcript archives and raw payload spools; ≥18mo audit events; 30d export signed-URL TTL then archive [ASSUME OD-08].
- *If wrong:* retention jobs are parameterized — config change + storage resizing; legal exposure only if mandated windows are *shorter* than assumed (steward review at the P7 gate catches this).
- *Needed by:* P7 (jobs live); parameter draft at P1.

**OD-09 — Beneficiary PII visibility per role.**
- *Question:* Who, if anyone, routinely sees beneficiary names/identifiers?
- *Owner:* Data steward + compliance office.
- *Assumption:* nobody by default; placeholders everywhere (I15); steward break-glass with ticketed window + audit [ASSUME OD-09].
- *If wrong (a role needs routine access):* masked-view widening + new data-flow matrix row + DPIA-style review (doc 16); structurally supported, policy-gated.
- *Needed by:* P1 (column design), P4 (evidence viewer).

**OD-10 — Publication authority.**
- *Question:* Who signs monthly/quarterly packs — and retractions?
- *Owner:* Monsha'at leadership (a delegation decision).
- *Assumption:* the product owner signs packs and retractions [ASSUME OD-10].
- *If wrong (committee/executive sign-off):* publication flow gains steps; the pack SLA in doc 21 P6-E4 is renegotiated; no schema change (the signature model supports multiple signers).
- *Needed by:* P6 (first pack).

**OD-11 — Review-role powers.**
- *Question:* Can non-admin reviewers reject (not just approve/correct)? Is append-only history mandated?
- *Owner:* Product owner + data steward.
- *Assumption:* reviewers approve/correct/reject; history append-only; only admins retire items; nobody edits history [ASSUME OD-11].
- *If wrong:* portal RBAC matrix adjusts (config-level); the audit semantics must never weaken — that floor is doctrine, not preference.
- *Needed by:* P2 (portal v1 RBAC).

**OD-12 — Audio access legality.**
- *Question:* May Monsha'at lawfully access session audio (for gold transcription labelling and the STT contingency)?
- *Owner:* Legal + compliance office.
- *Assumption:* not available; EXP-02 runs Design B (comparative judgment) where gold audio labelling is impossible [ASSUME OD-12].
- *If wrong (available):* strictly better — EXP-02 Design A with true gold; the STT contingency (whisper-large-v3 on Groq, assess-only) becomes assessable (doc 06 §7).
- *Needed by:* P1 (EXP-02 design freeze).

**OD-13 — Second Read.ai OAuth client.**
- *Question:* Will Read.ai issue separate client credentials for NIP? (The refresh token rotates on use — sharing with legacy breaks both systems.)
- *Owner:* Platform owner (external request; long lead time).
- *Assumption:* granted before P1 live pulls; filed at P0 [ASSUME OD-13].
- *If wrong (delayed):* P1 exits in snapshot-mode; ongoing-ingestion start slips 1:1 with the credential; parallel-run freshness comparisons degrade.
- *Needed by:* P1 (live ingestion).

**OD-14 — Consultant national-ID handling + evidence-role widening.**
- *Question:* (a) May NIP retain era-1 consultant national IDs as restricted crosswalk attributes, or must they be hashed on ingest? (b) Do service_owners ever get evidence-viewer access?
- *Owner:* Data steward + compliance office.
- *Assumption:* retained in a column-restricted field used only by the identity resolver; hashed surrogate everywhere else; service_owner stays aggregate-only [ASSUME OD-14].
- *If wrong (must hash):* the M2 matching rung recomputes on hashes — works, with weaker manual debugging; if role widening is approved: new matrix row + audit (doc 16 §3).
- *Needed by:* P1 (ingest columns).

**OD-15 — Beneficiary cross-session linkage.**
- *Question:* May `beneficiary_key` link the same person across sessions (enabling repeat-beneficiary analytics)?
- *Owner:* Compliance office + data steward.
- *Assumption:* not linked at launch; `beneficiary_identity.national_id` stays NULL; repeat-beneficiary questions are honestly unanswerable (`DIMENSION_NOT_AVAILABLE`) [ASSUME OD-15].
- *If wrong (approved later):* a new P3-class surface → DPIA-style review, matrix row, backfill linkage job; doc 08's schema is already shaped for it.
- *Needed by:* no phase blocks on it; whenever the owner wants repeat-beneficiary analytics.

**OD-16 — Formal PDPL/NCA compliance verification.**
- *Question:* Registration of processing activity, lawful basis, cross-border transfer approval (interacts OD-04/OD-19), breach-notification chain, SIEM mandate, and NIP's classification under the authority's data-classification policy.
- *Owner:* Platform owner + compliance office (SDAIA/NCA channels as applicable).
- *Assumption:* proceed with doc 16 controls (they meet or exceed published baselines); production outbound flow blocked until OD-04 + OD-16 sign-off; pilots run on pseudonymized/synthetic staging data meanwhile [ASSUME OD-16].
- *If wrong (findings against the controls):* remediate per finding; timeline risk to P6/P7 gates — doc 21 risk R-2's larger sibling.
- *Needed by:* P6 entry (pilots with real users); hard-required at P7 (production).

**OD-17 — Rating/evaluation scale semantics.**
- *Question:* Exact scales and meanings of beneficiary rating vs consultant evaluation (CAP-D2, `eval_gap_avg`)?
- *Owner:* Service owner + data steward (via the OD-07 data dictionary).
- *Assumption:* beneficiary 1–5; consultant evaluation min-max normalized to 0–1; consultant-named gap publication blocked until confirmed [ASSUME OD-17].
- *If wrong:* metric formulas re-parameterize; any signed DS-03 items touching them are mechanically invalidated and re-signed — the staleness machinery (ADR-0011/§4.4) absorbs it.
- *Needed by:* P3 (metric registry entries for CAP-D1/D2).

**OD-18 — Hijri period resolution.**
- *Question:* Ship Umm-al-Qura Hijri→Gregorian resolution in v1, or fail loudly?
- *Owner:* Product owner.
- *Assumption:* Hijri markers fail loudly to `PERIOD_UNPARSEABLE` with a polite Arabic message («لم أتمكن من تحديد الفترة الهجرية»); rules ship in the first post-pilot iteration [ASSUME OD-18].
- *If wrong (v1 mandate):* period-resolver work lands in P3 (+M epic); golden fixtures extend; no architectural change.
- *Needed by:* P3 (resolver scope freeze).

**OD-19 — Groq contractual data terms.**
- *Question:* Signed DPA pinning: no-training-on-API-data, abuse-buffer retention window, Batch artifact deletion, zero-data-retention availability by tier (interacts OD-03)?
- *Owner:* Platform owner + legal.
- *Assumption:* Groq's published terms (transient data, no training) hold, but are treated as **unverified until the DPA is on file**; pseudonymization is mandatory regardless — defense in depth [ASSUME OD-19].
- *If wrong (terms worse than published):* OD-04 approval likely narrows; sovereign-option escalation; the single-egress client (ADR-0013) confines any change to one component.
- *Needed by:* P2 (before full-corpus outbound). Filed P0.

**OD-20 — Real transcript text in version control.**
- *Question:* May any real-transcript-derived eval item be committed to git?
- *Owner:* Data steward + security lead.
- *Assumption:* no — git holds aggregates, pointers `(meeting_ulid, turn_index, sha256)`, and synthetic text; quote-bearing items live in the restricted eval store; production-stamped assertions run only where that store is reachable [ASSUME OD-20].
- *If wrong (relaxed):* CI topology simplifies slightly; the restrictive assumption is never harmful.
- *Needed by:* P1 (eval store topology).

**OD-21 — Numerals and calendar in executive outputs.**
- *Question:* Latin (0–9) vs Arabic-Indic (٠–٩) digits; Hijri display on pack covers?
- *Owner:* Product owner (final format preference).
- *Assumption:* Latin digits with tabular figures; Gregorian primary with Arabic month names; optional secondary Hijri on pack covers [ASSUME OD-21].
- *If wrong:* renderer format packs swap (doc 18 owns the catalogue); exports re-template; zero data change.
- *Needed by:* P6 (first published pack).

**OD-22 — BI tool mandate and mart exposure policy.** *(Remapped from doc 07/16/17 "OD-14/BI".)*
- *Question:* Will an external BI tool (Power BI etc.) be mandated, and under what exposure contract?
- *Owner:* Platform owner + IT.
- *Assumption:* no external BI at pilot — governed XLSX/JSON exports suffice; if mandated post-pilot: a dedicated pre-aggregated, pre-suppressed `bi` marts schema (n≥30 and PII placeholders enforced *inside* the view definitions) via the `nip_readonly_bi` role only; direct BI access to `findings`/`core` permanently refused — an external tool cannot enforce I8/I15/suppression [ASSUME OD-22].
- *If wrong (BI required at pilot):* marts work pulls forward into P3–P4 (+L epic); the suppression-inside-views design already exists (doc 07).
- *Needed by:* P6 (pilot tooling expectations).

**OD-23 — Lane-3 artifact and custom-collection visibility.** *(Remapped from doc 13 "OD-14".)*
- *Question:* Are deep-analysis artifacts and custom RAG collections requester-private or role-shared?
- *Owner:* Product owner.
- *Assumption:* role-shared aggregates; quote access gated by the viewer's transcript permission (intersection rule, doc 13 §9.6); every served quote writes a sensitive-evidence audit event [ASSUME OD-23].
- *If wrong (private-only):* a per-artifact sharing switch (config); the promotion loop is unaffected.
- *Needed by:* P5 (collection ACL implementation).

**OD-24 — Annotation staffing and qualification.** *(Remapped from doc 15 "OD-19".)*
- *Question:* Who staffs second-annotator roles across DS-01…DS-12 — internal analysts, seconded domain experts, or a vetted vendor (vendor access to transcript text interacts with doc 16 + OD-04)?
- *Owner:* Product owner.
- *Assumption:* two internal Arabic-fluent analysts + compliance reviewer for DS-04 + product-owner adjudication, ~45 h/month steady state; no external vendor for transcript-bearing sets [ASSUME OD-24].
- *If wrong (no internal capacity):* vendor onboarding adds a compliance approval track and calendar risk to EXP-02/03/04 — doc 21 risks R-1/R-6 materialize.
- *Needed by:* P1 (DS-01 labelling start).

**OD-25 — Transcript-provider benchmark candidate shortlist and procurement.**
- *Question:* Which concrete providers enter EXP-02, with trial accounts, security pre-screening (doc 06 dim J), and contract lead times?
- *Owner:* Procurement + platform owner; technical input from the ML engineer.
- *Assumption:* 2–3 candidates procurable for trials within P1; if procurement stalls, EXP-02 runs Read.ai-baseline-only and the OD-06 decision slips without blocking any other phase [ASSUME OD-25].
- *If wrong (no candidates trialable):* OD-06 undecidable on evidence; the interim-Read.ai period extends; the quote-fidelity ceiling persists and is declared on quote-bearing capabilities.
- *Needed by:* P1 (EXP-02 launch). Request filed P0.

---

## 4. Explicit assumption log

Every load-bearing `[ASSUME]` the package builds on, in one place. Entries carrying an OD ID resolve when that OD closes. Entries marked ✻ are design-level assumptions with deliberately **no** OD — the design absorbs either outcome; they are logged for honesty, not decision-chasing.

| # | Assumption in force | OD | Principal documents leaning on it |
|---|---|---|---|
| A-01 | Approved container environment, S3-compatible storage | OD-01 | 07, 16, 19, 21-P0 |
| A-02 | OIDC + vault; Keycloak broker interim acceptable | OD-02 | 07, 16, 17 |
| A-03 | Non-enterprise Groq tier; nothing depends on llama-3.3-70b | OD-03 | 14 |
| A-04 | Pseudonymized transcript text approved for Groq outbound; raw PII never | OD-04 | 10, 13, 14, 16, 21-P2 |
| A-05 | CAP-C2 via Lane-3 pilot first | OD-05 | 04, 13, 21-P5 |
| A-06 | Read.ai interim; provider unknown; everything behind `TranscriptSource` | OD-06 | 05, 06, 09 |
| A-07 | Internal data via read-only API/export; daily/weekly freshness | OD-07 | 05, 09, 11 |
| A-08 | 90d pre-rebase archives; ≥18mo audit; 30d export TTL | OD-08 | 08, 16, 19 |
| A-09 | Beneficiary PII: placeholders everywhere; steward break-glass only | OD-09 | 08, 16, 18 |
| A-10 | Product owner signs packs/retractions | OD-10 | 04 (CAP-D10), 18, 21-P6 |
| A-11 | Reviewers append-only; admin-only retire | OD-11 | 15, 18 |
| A-12 | No lawful audio access; EXP-02 Design B | OD-12 | 06 |
| A-13 | Second OAuth client granted before live pulls | OD-13 | 05, 09, 21-P1 |
| A-14 | National IDs retained in restricted crosswalk column, hashed elsewhere; no service_owner evidence access | OD-14 | 05, 08, 09, 16 |
| A-15 | No beneficiary cross-session linkage at launch | OD-15 | 08, 16 |
| A-16 | Doc 16 controls proceed pre-verification; production gated on sign-off | OD-16 | 16, 21-P6/P7 |
| A-17 | Beneficiary scale 1–5; consultant eval normalized; gap publication blocked | OD-17 | 11 (metrics), 04 (CAP-D2) |
| A-18 | Hijri periods fail-loud at launch | OD-18 | 12, 15, 18 |
| A-19 | Groq data terms treated as unverified until DPA filed | OD-19 | 14, 16 |
| A-20 | No real transcript text in git; pointer+checksum pattern | OD-20 | 15 |
| A-21 | Latin digits, Gregorian primary in executive outputs | OD-21 | 18 |
| A-22 | No external BI at pilot; pre-suppressed marts only, if ever | OD-22 | 07, 17 |
| A-23 | Lane-3 artifacts role-shared; quotes permission-gated | OD-23 | 13 |
| A-24 | Two internal annotators, ~45 h/month; no vendor on transcript text | OD-24 | 15, 21 |
| A-25 | 2–3 provider candidates trialable in P1 | OD-25 | 06, 21-P1 |
| A-26 ✻ | Monthly volume swing (2.7×) persists → per-100-session rates are the permanent comparison unit | — | 10, 11 |
| A-27 ✻ | Corpus stays within single-Postgres headroom through pilot (+1 order of magnitude); partitioning + replica path defined | — | 07, 08 |
| A-28 ✻ | Groq strict-schema support on gpt-oss-120b/20b persists (verified 2026-08-02); registry + benchmark machinery is the hedge | — | 14 |
| A-29 ✻ | The 8 VIOL seed categories remain the owner's wording; taxonomy versioning absorbs any future edit | — | 04, 10 |

**The five highest-risk assumptions** (feeds the GREENFIELD §25 closing statement): **A-04** (outbound approval — blocks the enrichment plane), **A-13** (OAuth client — blocks live ingestion), **A-24** (annotation capacity — sets every benchmark's clock), **A-16** (compliance sign-off — gates production), **A-07** (internal access — gates D-family freshness) [INFER — ranked by blast radius × external-dependency degree].

---

## 5. Decisions requiring owner approval (feeds 00-INDEX)

Approval classes: **G1** = blocks implementation start; **G2** = blocks a named phase gate; **G3** = policy confirmation that can trail on the recorded safe assumption.

| # | Decision | Artifact to sign | Class | Blocks | Owner role |
|---|---|---|---|---|---|
| 1 | Accept ADR-0001…ADR-0020 (en bloc or itemized with exceptions) | this doc §2 | G1 | P0 start | Platform owner |
| 2 | Confirm the 8 VIOL categories' Arabic wording + the two coverage-gap additions (تهكم، تسويق شخصي) | doc 04 §B4 | G2 | P2 taxonomy seed | Product owner |
| 3 | Answer-key protocol + signing duty (~46 items at launch, weekly cadence) | doc 15 §3 | G2 | P3 exit | Product owner |
| 4 | Outbound-data approval (pseudonymized text to Groq) | doc 16 §4 matrix | G2 | P2 backfill | Compliance office (OD-04) |
| 5 | Groq DPA / contractual data terms | OD-19 file | G2 | P2 backfill | Platform owner + legal |
| 6 | Second Read.ai OAuth client request | OD-13 request | G2 | P1 live pulls | Platform owner |
| 7 | Internal-data access mechanism + freshness SLA | doc 05 contracts | G2 | P1 adapters | Internal-systems owner (OD-07) |
| 8 | Provider candidate shortlist + trial procurement | OD-25 memo | G2 | P1 EXP-02 launch | Procurement |
| 9 | Provider go/no-go after EXP-02 | doc 06 §5.5 sheet | G2 | P6 rebase slot | Product owner (OD-06) |
| 10 | Annotation staffing commitment (~45 h/month) | OD-24 plan | G2 | P1 labelling | Product owner |
| 11 | Publication authority for packs/retractions | OD-10 delegation | G2 | P6 first pack | Leadership |
| 12 | PII policy set: OD-09 / OD-14 / OD-15 confirmations | doc 16 §2–3 | G3 | P7 at latest | Data steward + compliance |
| 13 | Retention windows | OD-08 schedule | G3 | P7 retention jobs | Data steward + compliance |
| 14 | Review-role powers confirmation | OD-11 matrix | G3 | P2 (assumption suffices) | Product owner |
| 15 | PDPL/NCA formal verification file | OD-16 dossier | G2 | P6 pilots / P7 prod | Compliance office |
| 16 | CAP-C2 route confirmation after EXP-08 | OD-05 memo | G3 | post-P5 | Product owner |
| 17 | Executive output format (digits/calendar) | OD-21 sample pack | G3 | P6 first pack | Product owner |
| 18 | BI exposure policy | OD-22 memo | G3 | post-pilot | Platform owner |
| 19 | Legacy retirement date | doc 21 P7-E7 plan | G3 | post-cutover | Platform owner |
| 20 | Deployment environment + IdP/secrets specifics | OD-01/OD-02 memo | G1 | P0 provisioning | IT + security directorate |

**Implementation start gate** = items 1 + 20 signed, items 4–8 *filed* (not necessarily answered — the recorded safe assumptions carry P0–P1) [REC; restated in 00-INDEX].

---

## 6. Maintenance rules for this register

1. **New ODs**: next free number (OD-26…), never reuse; every new `[ASSUME]` in any document must reference an OD here, or be logged in §4 as a ✻ design-level assumption with its absorbing design named.
2. **Closing an OD**: record decision, date, decider, and which documents' assumptions flip; closed ODs stay in the table marked **CLOSED** — history is append-only, matching ADR-0012's doctrine.
3. **ADR changes**: an accepted ADR is amended only by a superseding ADR (`ADR-0021…`) that names what it replaces; no in-place edits after owner acceptance. Rejected ADRs are recorded with the rejection rationale, not deleted.
4. **Consistency-pass duty** (from §3.1): **executed 2026-08-03** — docs 07/16/17 "OD-14/BI" remapped to **OD-22**; doc 13 "OD-14" to **OD-23**; doc 15 "OD-19" to **OD-24**; verified by grep (the collision patterns survive only in this register's §3.1 history table and 00-INDEX's resolution log).
5. **Cadence**: the OD register is reviewed at every phase-exit gate (doc 21); any OD past its needed-by phase without a decision escalates to the platform owner with its safe assumption restated and its accumulating cost quantified.

---

*End of document 22.*

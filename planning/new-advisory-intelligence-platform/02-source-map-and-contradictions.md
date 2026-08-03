# 02 — Source Map and Contradiction Log
**Platform:** Nwafeth Intelligence — منصة نوافث لذكاء الجلسات الاستشارية (Monsha'at Advisory Session Intelligence Platform)
**Status:** Draft for owner review · **Date:** 2026-08-02 · **Author:** Planning package (Fable 5)
**Depends on:** none (this is the evidence foundation) · **Feeds:** 00, 01, 03–23 (every document cites this one for source facts, resolved contradictions, and baseline numbers)
**Sources used:** GREENFIELD §1–§2, §5–§7, §24–§25; MASTER_PROMPT §2 (live 2026-08-02 ground truth), §3, §4, §4.4, §5, §12, §13.8–13.9, Appendices A–C, E; arch/01–08 (full scan; spot-verified against originals: arch/02 §7 FK audit, arch/07 F14/§7-Q10)

---

## 0. Purpose, method, and citation discipline

This document is the **required source-analysis pass** of GREENFIELD §1.2: an evidence map for every attached source, a contradiction log with each conflict resolved, an obsolete-mechanisms register, the settled decisions carried forward, and the assumptions extracted into Open Decision IDs. Every other document in this package cites facts **through** this document rather than re-deriving them, so that a stale number or a refuted premise is corrected in exactly one place.

### 0.1 The two precedence ladders (never mixed) [FACT GREENFIELD §1.1]

**Descriptive** (what the legacy system actually is/does):

1. live measurements on the legacy database (2026-08-02);
2. MASTER_PROMPT §2 facts marked *(live)*, dated 2026-08-02;
3. arch/01–08 (snapshot of branch `feature/admin-portal`, 2026-07-16);
4. docstrings, CLAUDE.md/MEMORY.md, older memories — **lowest rank; repeatedly proven wrong** (see CON-04, CON-05, CON-19…CON-23).

**Prescriptive** (what the new platform must be):

1. GREENFIELD owner requirements (§2, §5);
2. GREENFIELD non-negotiables I1–I18 (§7);
3. settled MASTER_PROMPT decisions explicitly carried forward (SD-01…SD-16 below);
4. this package's recommendations, with alternatives and revisit-triggers;
5. current legacy behaviour — only where still justified. **Current behaviour is never an argument for keeping current behaviour** [FACT GREENFIELD §1.1].

### 0.2 Anchor notation and claim classes

- Arch anchors: `arch/NN §S` (optionally `/line`), e.g. `arch/02 §7/469`. MASTER_PROMPT anchors: `MP §S`. Greenfield: `GF §S`.
- Row classes in the source-map tables: **CE** current-state evidence · **SD** settled decision · **AS** assumption (→ OD-xx) · **CT** contradiction (→ CON-xx) · **OM** obsolete mechanism (→ OBS-xx).
- Legacy defect IDs `F1…F38` and `ISS-01…ISS-18` are the arch docs' own registers; this package cites them verbatim and maps each to a structural countermeasure (doc 07 §risk-mapping, doc 22 ADRs).

### 0.3 Verification-status discipline inherited from the sources [FACT arch/08 §Detection-method; MP §2.3]

Three hygiene rules bind every author in this package:

1. **Nothing in arch/08 was executed** — every runtime claim there is derived from control-flow reading. ISS-03's user-visible consequence is *predicted from SQLAlchemy's documented guard, not observed*; ISS-15/ISS-17 are marked "(reported, not independently verified)". These markers must never be laundered into flat fact [FACT MP §2.3].
2. **arch/06's pipeline wall-clock runtimes are "(not verified)"** throughout its runbook table; artifact sizes (~4 GB image, ~1 GB dump) are likewise unverified [FACT arch/06 §2, §7#5]. Capacity planning in docs 19/21 must not treat them as measurements.
3. **Every corpus figure in arch/05–08 is a 2026-07-16 snapshot.** MP §2 numbers (2026-08-02) supersede them; and MP §2 itself demands re-measurement via a committed script before quoting any figure to a stakeholder — a script that does not exist in the legacy repo [FACT MP §2 preamble]. §7 below imposes that discipline on this package.

---

## 1. SOURCE MAP

One table per source. Only *materially relevant* rows — facts and requirements that change what the new platform builds. Effects cite the package documents (docs 04–23), capability IDs (CAP-xx), invariants (I1–I18), and ADRs (doc 22).

### 1.1 MASTER_PROMPT.md (prior refactor prompt; settled owner decisions + live 2026-08-02 corrections)

| # | Fact / requirement | Anchor | Class | Effect on the new design |
|---|---|---|---|---|
| M1 | Two-plane split is real and enforced in practice: offline enrichment writes fact tables; online plane reads only PostgreSQL; no handler calls an LLM; no Read.ai call at query time | MP §2.1, §2.6.1 | CE | The one legacy architectural property worth keeping; hardened into I1 (docs 07, 12) with structural CI checks, not convention |
| M2 | Query-time model calls today number ≥4 (intent, rewriter, clarifier gen, clarifier resolve) plus legacy-fallback calls | MP §2.1 | CE | New serving plane budgets model touchpoints explicitly per lane (doc 12; Lane 1 ≤8 model calls) |
| M3 | Four false premises to reject on sight: zero FKs; composite-gather; ASR "5@75%"; MiniLM-L6 embedder | MP §2.2 | CT | Resolved as CON-01…CON-04 below; the corrected facts drive ADR-0006/0007/0014 |
| M4 | Live 2026-08-02 corpus corrections: PENDING=57 (0.3%); services_report=94,963; span=14 months 2025-05…2026-06; all 16,911 meetings processed & scored | MP §2.3 | CE | Baselines table §7; partition lists always discovered from data, never hardcoded (doc 13); CON-06…CON-09 |
| M5 | **ASR correction removed permanently, by owner decision (2026-08-02), data deleted not migrated; never reintroduce under any name**; only sanctioned interim mitigation = deterministic read-time whole-word glossary, no LLM, no write-back, deleted when new provider lands | MP §2.4, §12 anti-goal 1b | SD | SD-01; ADR-0007; docs 06 (glossary spec), 09; quality-bar item "revives ASR correction" = rejection |
| M6 | Deleting corrected text obligated re-extraction of 11,304 meetings (67% of corpus) whose quotes would fail the R7 gate — rebase and provider-swap are the *same operation* | MP §2.4–2.5 | CE→SD | `rebase_transcript` designed once, used for provider swaps (doc 06 §rebase; doc 09); quote verification (I7) is the load-bearing flip gate |
| M7 | A second, cleaner transcript provider is coming; timing unknown; Read.ai is interim; turn identity NOT portable across providers; no diffing/merging between providers | MP §2.5 | SD | SD-02; `TranscriptSource` contract + `(source, source_version)` on every stored pointer (docs 05, 06, 08); OD-06 |
| M8 | Preserve list: two-plane split; deterministic reviewed SQL; `check_period_coverage` structural-discovery pattern (generalise it); ASR audit-trail *shape* (never overwrite source, PK-keyed derivations, audit rows incl. skips, undo=replay); idempotent upserts; identity purity; full-ULID citation; coverage block `used ≤ matched ≤ total` | MP §2.6 | SD | Generalised structural gates (I12, doc 15 §structural-checks); audit-trail shape reused for review loops and Lane-3 artifacts (docs 09, 13); coverage invariant in the answer envelope (docs 11, 17) |
| M9 | Invariants I1–I16 (implementation-level doctrine): no LLM in data paths, no model SQL, no model arithmetic, period fail-closed, full-ULID citation, never answer a different question, one active transcript, RAG optional, no PII widening, one-time snapshot, cost-not-a-constraint, taxonomy version on every number | MP §3 | SD | Same doctrine as GF I1–I18; R1–R16 semantics carried into docs 10–13, 15 (esp. R6 numeric gate, R7 quote gate, R14 fail-closed, R15 injection isolation) |
| M10 | Product contract: 16 committed questions (A×3, B×5, C×8) with the owner's Arabic exemplars, the impact rubric, and the consequence lists | MP §4; GF §5 | SD | CAP-A1…CAP-C8 (doc 04); worked Arabic examples reused in docs 10, 15, 18 |
| M11 | **The 8-type violation taxonomy verbatim, with source examples; legacy implements only 7 of 8 (type 4 unimplemented); تهكم and تسويق شخصي detected by nothing** | MP §4-B | SD+CE | SD-10; VIOL-001…008 seed (docs 04, 10); named coverage exclusions until discovery closes the gaps (R-P1) |
| M12 | R-P1 seeds-not-closed-vocabularies (discover unseen members AND unseen measures); R-P2 semantic recurrence; R-P3 fresh-per-period analysis with real sessions and quotes surfaced | MP §4.1 | SD | SD-08, SD-09; taxonomy lifecycle (docs 10, 11); pack materialisation keyed by period (docs 11, 13) |
| M13 | Answer oracle: owner signs the answer key; machine proposes, human disposes; ~46 key items, ~400 paraphrases, ~100 evidence items; incremental signing; stale-scoped invalidation; re-sign audited against gate-weakening | MP §4.4 | SD | SD-07; ADR-0012; doc 15 owns the full protocol; review UI built once for key+taxonomy+clusters (doc 18) |
| M14 | Lane architecture: deterministic pre-processing before any model; Lane 0 τ/δ calibrated matcher (precision ≥0.99); Lane 1 bounded loop; Lane 2 honest boundary; Lane 3 async month-partitioned deep jobs | MP §5.0–5.3 | SD | Lanes 0–3 as adopted in GF §8.6; doc 12 decision tree; EXP-05 calibration |
| M15 | **Lane 3 has no cost ceiling — owner decision 2026-08-02; scope declared, never refused; bounded concurrency is a rate-limit control only; failed partition = INCOMPLETE, never silent** | MP §5.3 | SD | SD-05 (correctness over cost); ADR-0015, ADR-0020; docs 13, 14 |
| M16 | Retired `/v3` planner lesson: an LLM planner that is not period-correct loses to a deterministic router that is; deterministic period/turn resolution is a hard precondition for any agentic lane | MP §2.1, §12 anti-goal 1 | CE | Doc 12 places `resolve_period`/turn-resolution before any model call; I4 |
| M17 | Anti-goals: no `/v3` rebuild; no ASR under any name; no metric "fixed to spec" without checking the spec (error-rate trap); no docstring trust; no gate weakening | MP §12 | SD | Carried into doc 22 (constraints on ADRs) and doc 15 (I18/I13-equivalent gate rules) |
| M18 | Snapshot migration: one-time checksummed `pg_dump`, restored into `legacy_snapshot` schema inside the new DB; **raw layer only — derived data never migrated, always re-derived**; CI fixture is a ~500-meeting slice of the same snapshot | MP §13.8 | SD | SD-06; ADR-0016; docs 08 (`legacy_snapshot` schema), 20 |
| M19 | Read.ai rotates the refresh token on every refresh → sharing one credential between two systems breaks both intermittently; second OAuth client has lead time and blocks cutover | MP §13.8 | CE | OD-13 (request early); docs 05, 09, 20 |
| M20 | Parallel-run parity is valid only on a pinned common basis (closed pre-snapshot period, 1:1 taxonomy crosswalk, accepted-differences ledger with direction); never "compare everything" | MP §13.9 | SD | Doc 20 §parallel-run; difference categories reused verbatim from GF §19.3 |
| M21 | Appendix A metric starter (floor not ceiling), Appendix B capability starter mapping, Appendix C closed reason codes, Appendix E per-question method plans incl. E.0 method rules (rates not counts, DISTINCT sessions, min n=30 + Wilson CI, ≥5-consultant confound, no circular labels, precision-over-recall for accusations) | MP App A–C, E | SD | Reason codes adopted verbatim (doc 12 §reason-codes); E.0 rules become doc 10 §method-rules; metric registry seeds doc 11 |

### 1.2 arch/01-system-overview.md (2026-07-16)

| # | Fact / requirement | Anchor | Class | Effect on the new design |
|---|---|---|---|---|
| A1-1 | C4 shape: single FastAPI app (port 8011, one uvicorn process, no `--workers`), Postgres+pgvector, offline CLI pipeline; only two required env vars | arch/01 §3–§5 | CE | New topology: two runtimes (web+workers), pool isolation, container platform (doc 07; ADR-0002) |
| A1-2 | Corpus scale 2026-07-16: 16,911 meetings; 399,501 turns; 158,949 edits; 192,432 EAV sections; 109,167 chunks; 83,325 report rows | arch/01 §7/251–256 | CE(stale) | Superseded by §7 baselines where MP §2 re-measured (CON-06…CON-09); still valid for order-of-magnitude sizing |
| A1-3 | 1,629 meetings (9.6%) `consultant_id='PENDING'` | arch/01 §7/251 | CT | CON-06: live value 57 (0.3%); the *mechanism* (magic string) remains the defect → OBS-02 |
| A1-4 | Dead weight: V1 tables (13,933 meetings, 90,164 chunks), 76,111-row PII backup table with zero code references | arch/01 §7/284–287 | OM | OBS-22; snapshot scope excludes V1 + backup tables (doc 20); PII handling in doc 16 |
| A1-5 | `v2_feedback` 3 rows; `v2_violation_reviews` 0 rows — review workflow never exercised against 3,033 violation-bearing meetings | arch/01 §7/281–282 | CE | Review workflow is a build-and-adopt problem, not a migration problem; UX investment justified (doc 18); no review data worth migrating (doc 20) |
| A1-6 | Cost/latency placement already correct: enrichment offline, answers ≈1 LLM call + 1–8 SQL | arch/01 §6/239 | CE | Confirms Lane-0 economics; do not move enrichment online (I1) |
| A1-7 | Export depends on public CDNs at runtime; 3 of 4 Python export deps missing from requirements | arch/01 §3/88, §5/200 | OM | OBS-04; self-hosted assets only (doc 07 frontend; ADR-0018) |
| A1-8 | Batch pipeline not in the Docker image; `pg_dump` absent from runtime image; restore only into empty PGDATA | arch/01 §4–§5 | CE | Deploy artifact must contain everything imported (doc 19); backups via pgBackRest with restore drills (doc 19; ADR ops) |
| A1-9 | "Decide who owns the schema FIRST" — the docs' own top directive | arch/01 §6/238 | CE | ADR-0004 Alembic-only from day one; migration order is epic #1 territory (doc 23) |

### 1.3 arch/02-data-model-erd.md (2026-07-16)

| # | Fact / requirement | Anchor | Class | Effect on the new design |
|---|---|---|---|---|
| A2-1 | **28 FK constraints exist (27 CASCADE), verified against `pg_constraint`**; hub `v2_meetings.meeting_ulid` is a real enforced cascading FK for 22 of 24 meeting-scoped tables; the only gaps are the two tables created outside the ORM | arch/02 §7/465–488 (spot-verified in original) | CE | Kills the "zero FKs" premise (CON-01); new schema: FKs + PKs universal, constraints-as-pins (doc 08; ADR-0006) |
| A2-2 | `v2_violations`: **no PK, 6,794 rows / 3,691 distinct, 3,103 exact duplicates, counts inflated ≤1.84×** | arch/02 §8/531–553 | CE | F1 lesson: every violation number the legacy product ever reported is suspect; parity runs must expect the dedup delta (doc 20); natural keys + PK policy (doc 08) |
| A2-3 | Knowledge fact tables carry NO date column — period filtering only via meeting-scope subquery | arch/02 §3/177 | CE | New findings schema carries session FK + period-resolvable timestamps by construction (doc 08 `findings` schema) |
| A2-4 | EAV `v2_meeting_sections` (11.4 rows/meeting) with exact double-writes into typed tables | arch/02 §2/161–172 | OM | OBS-03 |
| A2-5 | Three disagreeing consultant tables (39 / 446 / 894 rows); `consultant_id` unenforced everywhere; Excel `session_uuid` bridge unindexed, no FK | arch/02 §7/490–496 | CE | One consultant dimension + crosswalk with resolution provenance (doc 08 `core`); consultant directory contract (doc 05); EXP-01 measures join rates |
| A2-6 | Polymorphic `v2_validation_log` (37,339 rows; free-text table discriminator; 1,382 rows unresolvable due to F1 dup ids) | arch/02 §3/298 | OM | OBS-20 |
| A2-7 | Shadow validation columns added by runner-less .sql, absent from ORM | arch/02 §3/294–296 | OM | OBS-08 (schema provenance); validation results are first-class rows in `findings.validation_results` (doc 08) |
| A2-8 | Vector store: `Vector(384)` hardcoded; char-window chunking 2000/200; **no per-row embedding-model provenance**; ivfflat lists=100 chosen before data existed; chunks name-scrubbed at read time only | arch/02 §11/613–631 | CE+OM | I13 embedding provenance mandatory; write-time PII scrubbing; index params re-derived from data (docs 08 `evidence`, 11, 16); EXP-04 |
| A2-9 | Magic string `'PENDING'` in NOT NULL column; registry refresh would mint a fake consultant | arch/02 §7/491 | OM | OBS-02; `resolution_status` enum + nullable FK (§6 CORE naming; doc 08) |
| A2-10 | Portal auth tables: plaintext session-token PK, 7-day expiry never purged; one-decision-per-meeting review PK | arch/02 §5/379–408 | OM | OBS-15, OBS-16 |
| A2-11 | Enums enforced only in Python — no CHECK constraints | arch/02 §7/502 | CE | CHECK/enum constraints in DDL as pins (doc 08) |
| A2-12 | Collapse the 4 state mechanisms (flags, magic strings, row-absence, shadow columns) into one explicit pipeline state | arch/02 §12/760–773 | CE | Pipeline state machine per session with recorded basis+fingerprint (doc 09 §state-machine) |

### 1.4 arch/03-components.md (2026-07-16)

| # | Fact / requirement | Anchor | Class | Effect on the new design |
|---|---|---|---|---|
| A3-1 | 32 handler modules (26 leaf + 6 composite), uniform contract `(session, intent) → Arabic HTML str`; zero external calls from handlers (grep-verified) | arch/03 §4.1/421–431 | CE | The committed-question inventory these handlers answer seeds the capability catalogue (doc 04); the HTML contract is OBS-14 |
| A3-2 | **Eight unsynchronized intent registries** + 236-line if/elif dispatch + per-request imports | arch/03 §7.2/592–612 | OM | OBS-05; F5/F8 lesson → I12 single governed registry, build fails on unregistered assets |
| A3-3 | `sql_agent.py` (5,691→5,776 lines) on the hot path for 16 intents via 17 registry keys — despite CLAUDE.md calling it dead | arch/03 §7.1/583 | CT | CON-21; nothing from sql_agent is migrated — its intents are re-expressed as registered metric specs (doc 11) |
| A3-4 | Period helpers both end-exclusive but **always optional** — `("", {})` when no period | arch/03 §4.2/437–460 | CE | Inverted: period is a required compiler argument, absence = compile failure (I4; doc 11) |
| A3-5 | `check_period_coverage`: structural discovery from disk, diffed vs explicit classification, fails build on unclassified modules — the one sound guardrail | arch/03 §4.5/506 | CE | Generalised to capability registration, auth coverage, escaping, curated-vs-spec boundary (docs 11, 15; MP §2.6.3) |
| A3-6 | Module-global state: `_clarification_contexts`, `_backup_tasks` dicts — blocks horizontal scaling | arch/03 §7.3/621 | OM | OBS-06; F13 → durable state in Postgres `serve` schema (doc 12) |
| A3-7 | Hand-maintained ~215-entry LLM-label→Arabic map ("LIVE, losing"); hamza normalization open-coded ×4 | arch/03 §4.2/441, §7.3/622 | OM | OBS-18; constrained enums at extraction + one normalization stage (doc 10) |
| A3-8 | Concurrency rule: concurrent `execute()` on one AsyncSession illegal; composites use `gather_seq`; real violation one level up | arch/03 §4.4/492–498 | CE | CON-02; session-per-concurrent-task by construction (doc 07) |
| A3-9 | Behavioral predicate copy-pasted ×3 with shape drift | arch/03 §7.3/623 | OM | OBS-19; one repository layer owns scopes/predicates (doc 07 layering) |
| A3-10 | Raw exception text rendered into chat bubble; HTTP 200 on failure | arch/03 §6/568 | CE | I16; error envelope with correlation id (docs 17, 19) |

### 1.5 arch/04-runtime-flows.md (2026-07-16)

| # | Fact / requirement | Anchor | Class | Effect on the new design |
|---|---|---|---|---|
| A4-1 | Live chat flow: 5-turn memory → intent LLM → ~8 Arabic keyword-override layers → `resolve_period` (end-exclusive) → `_smart_route` → one handler → raw SQL → Arabic HTML | arch/04 §1/14–77 | CE | Replaced by deterministic pre-processing + Lane routing (doc 12); end-exclusive periods retained as convention (§13.4 cross-doc rule) |
| A4-2 | Read.ai API mechanics: cursor pagination limit=10, 7 expand fields, OAuth refresh at 120s buffer, TokenBucket 80/60s, title-regex + exactly-2-participants validity filter | arch/04 §5/377–383 | CE | Read.ai adapter contract (doc 05); its quirks (newest-first assumption, lowercase-hex regex, guess-storing `_parse_title`) become explicit contract tests (doc 09) |
| A4-3 | Ingestion: DELETE+INSERT of turns/sections; upsert forces `transcript_corrected=false` while `groq_processed` survives — re-ingest destroys derived consistency | arch/04 §5/385–387 | OM | F3 lesson: immutable raw layer + content-fingerprint invalidation (docs 08, 09) |
| A4-4 | ASR ensemble mechanics: detect 7 models×2 temps ≥2-vote; fix 3 models unanimity; ≤3 propagation rounds; >10% cap discards all edits yet sets the success flag; ~168 concurrent calls/meeting via unbounded gather | arch/04 §6/464–480 | OM | OBS-01 (deleted permanently, SD-01); F10/F11/F19 lessons → bounded pools, per-step outcome states, INCOMPLETE partitions (docs 09, 13) |
| A4-5 | Extraction: exactly 1 JSON call/meeting, temp 0.1, 6144 tok, 180s; enum coercion at write time; `confirmed` violation without quote demoted to `potential` — code-enforced | arch/04 §7/504–557 | CE | Per-session extraction cost model (~1 call/session) anchors Lane-3 and backfill estimates (docs 13, 14); code-enforced demotion prefigures deterministic validation (I18) |
| A4-6 | Scoring: LLM-free, 4 criteria 0..1, mean×5 clamped; impact thresholds 2.5/1.5; violation_penalty a separate axis | arch/04 §8/604–608 | CE | Method baseline for CAP-A1 rubric alignment — must be auditable against the owner's impact rubric (doc 10; MP §4-A1) |
| A4-7 | Engagement score counts `؟` across ALL turns incl. consultant (F16-legacy scoring bug); silence ignores overlapping speech (F15) | arch/04 §8/620, §5/384 | CE | Doc 10: per-speaker-role computation and interval-merge speaking time as method rules (§13 cross-doc: silence_pct lesson) |
| A4-8 | Clarification: clarity_score <0.70 triggers clarifier; ≤4 Arabic-letter options; fallback hints outside router allowlist silently discarded; state process-local | arch/04 §3/217–220 | CE+OM | Lane 2 options generated from the registry (invalid impossible); clarification state persisted (doc 12) |
| A4-9 | Excel import: TRUNCATE + positional 49-column parsing; mandatory undocumented re-link step; `_parse_title` stores guesses | arch/04 §9/671–677 | OM | OBS-12, OBS-13; staged header-mapped import with reconciliation (docs 05, 09) |
| A4-10 | Review flow: UPSERT last-writer-wins verdicts, restore = hard DELETE, reviewer reads raw text while extractor judged corrected | arch/04 §11/780–823, §4/297 | OM | OBS-16; append-only review events + evidence fingerprints + one active-transcript accessor (docs 08, 18) |

### 1.6 arch/05-api-and-interfaces.md (2026-07-16)

| # | Fact / requirement | Anchor | Class | Effect on the new design |
|---|---|---|---|---|
| A5-1 | 36 routes / 12 routers; **21 routes no auth; CORS `*`**; anonymous `pg_dump`; default admin `admin/admin123` auto-seeded | arch/05 TL;DR, §1.5 | CE | F2/ISS-04 → I14 deny-by-default, scoped CORS, no default credentials (doc 16; ADR-0017) |
| A5-2 | Response contract: raw Arabic HTML for 26/32 handlers, 6 plain-text; client sniffs first char; `sources` hardcoded `[]` | arch/05 §2.2–2.3 | OM | OBS-14; I11 typed envelope precedes presentation (docs 11, 17) |
| A5-3 | `session_id` client-generated, no ownership column; `/v2/conversations` returns every user's sessions | arch/05 §2.1 | CE | Conversations owned server-side under OIDC identity (docs 16, 17) |
| A5-4 | Groq client reality: retry 3× (2/4/8s), 400 never retried; 3 raw-httpx bypass sites fail-open; `seed=42` best-effort; **json_object never requested for the default model** (`_JSON_MODE_UNSUPPORTED`); greedy-regex JSON parsing | arch/05 §4.1–4.6 | CE | Doc 14: strict json_schema on gpt-oss-120b/20b replaces prompt-discipline parsing [FACT groq-docs 2026-08-02]; no fail-open model calls (I16); retry taxonomy preserves 429 visibility (F27/ISS lessons) |
| A5-5 | Embedder: `paraphrase-multilingual-MiniLM-L12-v2`, 384-dim, sync `embed_text` blocking the event loop | arch/05 §5 | CE | CON-04 resolution input; embeddings move to worker-plane local service (ADR-0014; doc 14); EXP-04 includes the L12 model as baseline |
| A5-6 | Contract-hygiene defects: HTTP 200-on-failure ×3 sites, unconstrained `format` str, dropped `complexity` key, bare-token auth accepted | arch/05 §1.8 | CE | API error model + Literal enums + versioned contracts (doc 17) |
| A5-7 | Token refresh mutates `settings` in memory and rewrites `.env` from the server process | arch/05 §3.1 | OM | OBS-09; F28 → vault-injected secrets, DB-canonical token state (docs 09, 16) |
| A5-8 | Frontend: `window.fetch` monkey-patching for auth; token in localStorage; module-scope API call before login; DOM-scraping export via CDN libs | arch/05 §6 | OM | OBS-04, OBS-25; React+TS RTL frontend, self-hosted, structured-answer rendering (ADR-0018; doc 18) |
| A5-9 | Hardcoded redeploy-only config: models, buckets, ensemble params, ports, creds | arch/05 §7.8 | CE | Model registry + config surface (I17; docs 14, 19) |

### 1.7 arch/06-pipelines-and-ops.md (2026-07-16)

| # | Fact / requirement | Anchor | Class | Effect on the new design |
|---|---|---|---|---|
| A6-1 | Pipeline = 4 ordered *manual* CLI steps from a Windows shell; no orchestrator, no CI, no Alembic; wrapper contradicts runbook | arch/06 TL;DR, §7#1 | OM | OBS-08, OBS-21; F10 → orchestrated DAG with all edges encoded, fail-closed defaults (doc 09) |
| A6-2 | Step-gate flags misbehave: re-ingest does NOT reopen extraction backlog but resets correction flag; skipping scoring silently empties impact analytics | arch/06 §1 | CE | F22 → per-session pipeline state machine with staleness detection surfaced in answers (doc 09; I16) |
| A6-3 | Schema managed by 3 parallel mechanisms; 2 tables exist only in the binary dump; DR broken; drift detector WARNs instead of FAILs on DB-only tables | arch/06 §5; arch/02 §9 | OM | OBS-08; F3 → Alembic-only (ADR-0004); CI provisions fresh DB from migrations and smoke-tests every route (doc 15) |
| A6-4 | Only quality gate: 31 Tier-1 cases + period guardrail, opt-in local hook **running against the production DB**, advertises `--no-verify` | arch/06 §4 | OM | OBS-21; F11 → CI on seeded fixture DB; golden suite (doc 15) |
| A6-5 | Backups: 3 implementations, only one real (7d+4w), nothing schedules it, no restore drill; retention nonexistent elsewhere | arch/06 §5; arch/07 §7-Q4 | CE | Doc 19: pgBackRest, scheduled, drilled; retention windows per data class (OD-08; doc 16) |
| A6-6 | Cost shape: extraction ≈1 Groq call/meeting (~16.9k for full re-extract); scoring materialises all meetings in RAM + ~14k-element NOT-IN tuple | arch/06 §6 | CE | Backfill sizing for doc 14 (Batch API candidate) and doc 13; streaming/set-based processing rules (doc 09) |
| A6-7 | Scaling verdicts: ~20 concurrent users breaks the single process on CPU (sync embed + PBKDF2), not SQL; at 5× corpus the batch plane fails first | arch/06 §7; arch/07 §3.1–3.2 | CE | Capacity model inputs (doc 19); CPU-bound work off the event loop by construction (doc 07) |
| A6-8 | Observability: no metrics/tracing/structured logs; `/health` never touches DB; AVG reported where spec says P95; telemetry can't attribute (intent never persisted, no endpoint discriminator, no request IDs) | arch/06; arch/08 ISS-06/12/14/15/16/17 | CE | I16 + R13 full reasoning log; doc 19 observability stack; every answer reconstructable |
| A6-9 | Repo hygiene: 111 root scratch artifacts; 66-script heap; scripts execute on import; checkpoint state in gitignored JSON | arch/06 §7 | OM | OBS-17; repo/module conventions (doc 23) |

### 1.8 arch/07-architecture-review.md (2026-07-16)

| # | Fact / requirement | Anchor | Class | Effect on the new design |
|---|---|---|---|---|
| A7-1 | Strategy recommendation: hybrid Option C — spec-compiled long tail + curated hard composites; Phase 0 (integrity+auth) before any architecture work | arch/07 §4, §6 | CE | Direct ancestor of the semantic-layer + curated-capability split (docs 11; GF §8.4–8.5); Phase-0 instinct inherited as delivery sequencing (doc 21) |
| A7-2 | "The hard composites are deliberately never spec-compilable — product judgment, curate forever" (regex taxonomy, sector=gov-entity choice, technical-issues-not-violations, calibrated 40% silence threshold) | arch/07 §4 | CE | Curated/spec boundary verified structurally (doc 11; MP §5.5); CAP owner types in doc 04 |
| A7-3 | Wrong violation count = wrong compliance judgment about a named consultant — the team's pin-the-LLM instinct is correct | arch/07 §4 | CE | Precision-over-recall for accusations (doc 10 E.0 rule); review gates before publication (docs 15, 18) |
| A7-4 | `v2_business_profiles` = 16,911 all-NULL placeholders awaiting a nonexistent external API; every sector/stage analytic empty by construction; `v2_consultants` 39 rows vs 894-row directory | arch/07 F14, §7-Q10 (spot-verified in original) | CT | CON-09: `service_category` now 77.3% populated (live), but true `business_sector` still has NO source → «قطاع» three-way disambiguation rule; `business_sector` UNAVAILABLE until external identity source (§13.3 cross-doc rule; OD-05 context) |
| A7-5 | PII exposure is the one open question with legal exposure: real names in handler output, national IDs in report table + 76k backup | arch/07 §7-Q6 | CE | I15 + data classification and placeholder policy (doc 16); snapshot PII handling (doc 20) |
| A7-6 | Open product questions: privilege asymmetry deliberate? append-only audit a compliance requirement? re-extraction invalidates stale reviews? | arch/07 §7-Q7–Q8 | AS | → OD-11 (review roles/append-only); staleness invalidation designed in regardless (doc 18; GF §4.3) |
| A7-7 | F-register F1…F38 with structural countermeasures | arch/07 §2 | CE | §4 obsolete register + doc 07 risk mapping; the specific IDs cited across the package (§12 CORE list) |
| A7-8 | Seed-42 claim in F18 ("neither passes a seed") contradicted by arch/05 §4.3 detail verification | arch/07 F18 vs arch/05 §4.3 | CT | CON-24 — 05's later, detail-verified account wins |

### 1.9 arch/08-issues-register.md (2026-07-16)

| # | Fact / requirement | Anchor | Class | Effect on the new design |
|---|---|---|---|---|
| A8-1 | ISS-01: deterministic period injection never reaches sql_agent; LLM `target_month` beats the parser via `setdefault`; real-world silent-all-time rate **unmeasured and unmeasurable** (intent never persisted) | arch/08 ISS-01 | CE | F16 → I4 harness-injected period absent from model schema; never state a legacy failure frequency (doc 20 parity framing) |
| A8-2 | ISS-02: unmapped intent silently degrades to a session count — plausible answer to a different question, logged as success | arch/08 ISS-02 | CE | I5 hard-fail; closed-world dispatch (doc 12) |
| A8-3 | Premise corrections: "telemetry never read" FALSE; "error IS NOT NULL" spec-fix trap (would pin error rate at 100%); composite-gather false alarm; reachability-count dispute | arch/08 §PC | CT | CON-02, CON-05, CON-25; metric definitions live next to their query (doc 19) |
| A8-4 | ISS-07: four answer layers behind one endpoint; 36/52 intents served only by the degraded fallback — an error path doing routine feature delivery | arch/08 ISS-07 | CE | Fallback paths are error paths, never capability paths (doc 12); one dispatch topology |
| A8-5 | Scope note: absence from the register ≠ health; PII/data-governance explicitly excluded at owner's direction; nothing executed | arch/08 §Purpose, §Out-of-scope | CE | §0.3 hygiene rules; doc 16 must not assume the register enumerated the security surface |
| A8-6 | ISS-10: period-blind exemptions — "intentional" vs "forgot" indistinguishable | arch/08 ISS-10 | CE | Every exemption is an explicit machine-checked declaration with a prose reason (docs 11, 15) |
| A8-7 | ISS-15/16/17: intent never persisted; no endpoint discriminator; no request IDs — migration decisions unmeasurable | arch/08 ISS-15–17 | CE | Instrumentation-before-decisions rule; R13 reasoning log from day one (docs 12, 19) |

### 1.10 GREENFIELD_PROMPT.md (authoritative owner requirements, 2026-08-02)

| # | Requirement | Anchor | Class | Effect on the new design |
|---|---|---|---|---|
| G1 | New platform replaces **Read.ai's intelligence/analytics/reporting/decision-support role**, not scheduling/hosting/recording; no meeting-bot scope creep | GF §2.1 | SD | Product boundary (doc 03); out-of-scope register |
| G2 | Combine two data classes: conversation-provider data + Monsha'at internal session data (incl. ratings, evaluations, attendance, follow-up); source-by-source data dictionary required; similar names ≠ same meaning | GF §2.2 | SD | Docs 05 (contracts), 08 (model); omitting internal ratings = package rejection (GF §24) |
| G3 | Full operational independence from legacy (own DB, secrets, workers, credentials, auth, observability); one-time checksummed snapshot; parallel-run via exported replay only | GF §2.3, I10 | SD | SD-06; ADR-0001, ADR-0016; docs 07, 20 |
| G4 | Groq is primary generative-AI infrastructure; task-specific model strategy; model IDs are registry data; embeddings/reranking may run locally with residency implications stated | GF §2.4 | SD | SD-03; ADR-0013/0014; doc 14 |
| G5 | Correctness > token cost; cost measured and reported; rate limits/wall-clock engineered explicitly | GF §2.5 | SD | SD-05; ADR-0020; §13.9 cross-doc rule |
| G6 | No transcript rewriting; immutable provider text; optional deterministic read-time glossary only; quality gating is a provider-selection problem | GF §2.6 | SD | SD-01/SD-02; ADR-0007; doc 06 |
| G7 | 16 committed questions (A1–C8) + 10 additional combined-data requirements (D1–D10), each mapped to capability ID, lane, method, taxonomy, evidence, suppression, gaps, signoff | GF §5 | SD | CAP-A1…CAP-D10 (doc 04 owns); omission = package rejection |
| G8 | Product rules: seeds-not-closed-vocabularies; semantic recurrence; fresh-per-period; frozen pack + live view; human sign-off defines correctness; one shared propose→review→approve workflow | GF §6 | SD | SD-07…SD-09, SD-11; ADR-0011/0012; docs 10, 11, 15, 18 |
| G9 | Invariants I1–I18 with enforcement through schema, code boundaries, tests, operations | GF §7 | SD | Cited as I1–I18 across the package; enforcement matrix in docs 07, 15 |
| G10 | Baseline architecture: 8 layers (adapters → raw/canonical → enrichment → semantic → curated → agentic lanes 0–3 → RAG decision tree → presentation) | GF §8 | SD | Doc 07 refines; deviations must preserve invariants and argue safety |
| G11 | Transcript provider: `TranscriptSource` contract; `rebase_transcript` 10-step workflow; labelled provider benchmark with go/no-go gate; STT contingency assessed, not included | GF §9 | SD | Doc 06; EXP-02; OD-06, OD-12 |
| G12 | Model strategy: verify live catalogue (llama-3.3-70b deprecating 2026-08-16 non-enterprise; strict json_schema on gpt-oss-20b/120b); role-based benchmark; one tool per turn; Batch API for offline only; registry + deprecation ops | GF §10 | SD | Doc 14; ADR-0013; OD-03; [FACT groq-docs 2026-08-02] verified this catalogue state |
| G13 | Data architecture domains (10 groups, §11.1–11.10) | GF §11 | SD | Doc 08 ERD covers all ten; schema names per §6 of the package spine |
| G14 | Methodology: units/denominators, DISTINCT sessions, per-100 rates, min-support + suppression, CIs, multiple-comparison correction, label validation, contrastive lift, stated-vs-inferred cause, coverage as part of the answer | GF §12 | SD | Doc 10 owns; E.0 rules merged in |
| G15 | Toolbelt baseline: 11 named tools; package must decide model-callable vs orchestrator-controlled for profile/deep-job tools; no free text reaching SQL | GF §13 | SD | Doc 12 argues the split ([REC] §4 of spine: deep-job tools orchestrator-controlled, `profile_dataset` model-callable) |
| G16 | Lane-3 factory is a mandatory deepest deliverable: trigger decision, schema generation from allowed primitives, map/verify/reduce, reusable artifacts, promotion thresholds, custom-RAG lifecycle | GF §14 | SD | Doc 13 owns; ADR-0015 |
| G17 | Security: threat model (15 named threats incl. prompt injection in transcripts, provider exposure, stale reviews); RBAC roles ×7; outbound data-flow matrix; injection isolation with adversarial tests; immutable audit | GF §15 | SD | Doc 16; OD-04, OD-09; EXP-10 |
| G18 | API/UX: versioned JSON contracts; 12 UX workflows; Arabic-first formal register; narrative must not overstate causality | GF §16 | SD | Docs 17, 18 |
| G19 | Evaluation: 12 labelled sets, 17 required tests, 8 hard acceptance properties (no model numbers, no unverifiable quote, no silent fallback, no unsigned subjective capability…) | GF §18 | SD | Doc 15 owns; gates are deterministic (I18) |
| G20 | Migration: snapshot bootstrap (re-derive findings, never migrate derived claims), independent ingestion, parallel-run difference categories, staged cutover gates | GF §19 | SD | Doc 20; ADR-0016 |
| G21 | Delivery: 17 workstreams, dependency-based phasing, no calendar dates without approved staffing | GF §20 | SD | Doc 21 |
| G22 | 10 mandatory pre-implementation experiments with hypothesis/sample/method/threshold/owner/artifact/decision | GF §21 | SD | EXP-01…EXP-10 cited package-wide; doc 21 sequences them |
| G23 | Quality bar: 17 rejection conditions; claim classification mandatory; every component states the failure it prevents | GF §24 | SD | §14 of the package spine; self-check applied to every document |
| G24 | Counts are planning baselines, not permanent truths; reproducible measurement script required before use in capacity/migration/stakeholder material | GF §1.3 | SD | §7 below; the committed-script requirement is assigned to doc 09/20 deliverables |

---

## 2. CONTRADICTION LOG

Every materially inconsistent statement found across the sources, resolved by the GF §1.1 ladders. **Tier 1** items would have misdirected the redesign if inherited; **Tier 2** items are doc-vs-code/doc-vs-doc drift that calibrates how much to trust each source class.

Format: claim A vs claim B → resolution (with ladder rule) → consequence for this package.

### Tier 1 — premise-level contradictions

**CON-01 — "The database has zero foreign keys" vs 28 measured FK constraints.**
- A: "Zero FKs" — an earlier redesign brief premise, still circulating in project memory [FACT MP §2.2].
- B: **28 FK constraints exist, 27 `ON DELETE CASCADE`**, verified against `pg_constraint` and `schema.txt:659-686`; `meeting_ulid` is an enforced cascading FK on 22 of 24 meeting-scoped tables; only `v2_quality_scores` and `v2_violation_reviews` lack it (both created outside the ORM), plus all `consultant_id` relations unenforced [FACT arch/02 §7/465–488, spot-verified in the original].
- **Resolution:** descriptive ladder — direct catalog measurement beats every narrative claim. The premise is FALSE.
- Consequence: the integrity problem is *narrow and specific* (two missing FKs at zero orphan risk, one unenforced identity dimension, one PK-less table), not systemic FK absence. Remediation budget and doc 08's constraint policy are sized accordingly; repeating the false premise anywhere in this package is a §14-level defect (spine rule 13.8).

**CON-02 — "Composites `asyncio.gather` on the shared session" vs `gather_seq` reality.**
- A: Composite docstrings (esp. `consultant_360.py`) + dead `import asyncio` ×6 claim parallel gathering on the shared AsyncSession [FACT arch/03 §4.4/496; arch/08 ISS-18].
- B: All six composites execute strictly sequentially via `gather_seq`; the real violation is one level up at `orchestrator.py:158` [FACT arch/08 PC#3, ISS-03; MP §2.2].
- **Resolution:** descriptive ladder — verified control flow beats docstrings. The composite claim is a false alarm caused by a stale docstring; **ISS-03's user-visible consequence at the orchestrator is predicted, not observed** and must be cited with that marker [FACT MP §2.3].
- Consequence: the new design does not "fix composites"; it makes the whole class unreachable — every concurrent unit gets its own session from the pool by construction (doc 07 layering; doc 09). Docstring-contradicts-code is itself treated as a defect class (anti-goal 5).

**CON-03 — ASR fix ensemble: "5 models at ≥75% of ≥4 responders" vs 3 models at unanimity.**
- A: Script docstring, CLAUDE.md, MEMORY.md, and the `fix_agree` column comment ("4/5") all state the 5-model supermajority [FACT arch/07 F18; arch/01 §4/141].
- B: Code since 2026-07-06: FIX panel = **3 models, unanimity** (`FIX_AGREE_FRAC=1.0`); real `fix_agree` values are n/3. (Separately, the DETECT panel was 7 models × 2 temps, ≥2 distinct-model votes) [FACT arch/04 §6/464–468].
- **Resolution:** descriptive — code values beat docstrings. Prescriptively the dispute is **moot**: the entire mechanism is deleted permanently by owner decision, data included, docs deleted rather than corrected [DECISION MP §2.4].
- Consequence: no document in this package describes the ensemble except as a lesson (OBS-01); the residual lessons (a cap that silently sets a success flag is a false gate; unverified samples are not gates) are carried into docs 09 and 15.

**CON-04 — Embedder identity: `all-MiniLM-L6-v2` vs `paraphrase-multilingual-MiniLM-L12-v2`.**
- A: CLAUDE.md documents `all-MiniLM-L6-v2` [FACT arch/01 §4/141; MP §2.2].
- B: Code default is `paraphrase-multilingual-MiniLM-L12-v2` — same 384 dimensions, **different model**; no per-row provenance records which model produced any of the 109,167 stored vectors [FACT arch/02 §11/620–626; arch/05 §5].
- **Resolution:** descriptive — code wins. The corpus may already contain vectors from mixed models with no way to tell.
- Consequence: legacy embeddings are worthless for migration regardless (I13 + rebase invalidation); the new `evidence` schema stores `(embedding_model, dim, generation_run)` per vector [I13]; EXP-04 benchmarks `BAAI/bge-m3` vs `multilingual-e5-large` vs the L12 model as baseline [REC spine §7]; startup validates config-dim vs column-dim (F13-legacy lesson).

**CON-05 — Error-rate semantics: spec `error IS NOT NULL` vs schema `error BOOLEAN DEFAULT FALSE`.**
- A: `REPORTING_FRAMEWORK.md:78` defines error rate as `error IS NOT NULL` [FACT arch/08 PC#2].
- B: The column is `BOOLEAN DEFAULT FALSE`; NULL is unreachable; rows log FALSE even when the orchestrator failed open [FACT arch/08 PC#2, ISS-05].
- **Resolution:** neither is trustworthy — the spec measured against this schema yields a constant 100%, and the schema under-reports by construction (fail-open logs success). This is the "do not fix a metric to match a spec without checking the spec" anti-goal [DECISION MP §12.2].
- Consequence: doc 19's metric definitions live next to their queries and are tested against fixtures; degradation flags propagate into telemetry rows (I16).

### Tier 1 — corpus-number drift (2026-07-16 docs vs 2026-08-02 live)

All resolved identically: **descriptive ladder rank 1–2 (live 2026-08-02 measurement / MP §2) beats rank 3 (arch docs)**. The stale figure is retained only to date-stamp evidence; the live figure is the planning baseline (§7).

**CON-06 — PENDING scale: 1,629 meetings (9.6%) vs 57 (0.3%).**
arch/01 §7/251, arch/05 §3.7, arch/06 §2, arch/07 F4 all carry 1,629/9.6%. Live: **57 (0.3%)** — the backfill worked; the consultant bridge is no longer a headline defect, but `consultant_id` remains FK-unenforced and the magic string still exists [FACT MP §2.3]. Consequence: identity remediation in doc 08 targets the *mechanism* (OBS-02), not a mass-backfill; migration reconciliation expects ≤57 unresolved rows plus stale `PENDING` copies in `v2_business_profiles` (arch/04 §9/678).

**CON-07 — `v2_services_report` rows: 83,325 vs 94,963; bridge match ~7.3% vs ~8%.**
Doc-time: 83,325 rows, 6,023 (7.2–7.3%) matched, ~36% of meetings resolved [FACT arch/01 §7/256; arch/05 §3.7]. Live: **94,963 rows** [FACT MP §2.3]; re-measured bridge ≈8% of report rows / under half of meetings matched [FACT live measurement 2026-08-02]. Consequence: the internal-report join is structurally weak at any measured value → EXP-01 (deterministic-ID join-rate experiment) is mandatory before ingestion design freezes; doc 05's internal-data contract demands a real keyed feed, not the Excel bridge (OD-07).

**CON-08 — Corpus span and completeness: 12 implied months / mixed processing vs 14 months, fully processed.**
Doc-time: scoring partial (impact levels for ~16.9k with mixed basis; `transcript_corrected` 51.5%) [FACT arch/01 §7/290; arch/06 §6]. Live: **14 calendar months 2025-05…2026-06, 556–1,512 meetings/month; all 16,911 meetings `groq_processed` and scored** [FACT MP §2.3]. Consequence: monthly volume swings 2.7× → per-100-session rates mandatory for comparisons (doc 10); partition lists discovered from data (doc 13); the "mixed extraction basis" defect (F12) resolved by deletion + re-extraction, not by finishing correction [FACT MP §2.4].

**CON-09 — `v2_business_profiles`: "16,911 all-NULL placeholders / every sector analytic empty by construction" vs 77.3% `service_category` populated.**
arch/07 F14/§7-Q10 (2026-07-16, spot-verified): all-NULL, awaiting a nonexistent external API. Live 2026-08-02: **`service_category` populated on 77.3% of rows, 48 distinct values** [FACT live measurement 2026-08-02] — backfilled from the services-report bridge after the docs were written. **Resolution:** live wins for `service_category`; the doc-time claim remains true for the *external-identity* fields (true industry sector, business stage), which still have **no source system at all**. Consequence: the three-way «قطاع» disambiguation is a hard cross-document rule — `service_category` (available, 77.3%), `government_entity` (text-extracted mentions), `business_sector` (**UNAVAILABLE** until an external identity source exists; answers must say so per I5/I8, never infer it from transcripts [DECISION GF §5-C4, §5-D]).

**CON-10 — `v2_transcript_edits`: 158,949 vs 275,869 rows.**
Doc-time 158,949 [FACT arch/01 §7/253]; live **275,869** [FACT MP §2.4]. Resolution: live wins; moot for design (table deleted with OBS-01) but material for snapshot sizing in doc 20 (the snapshot excludes ASR artifacts entirely).

**CON-11 — Knowledge-layer state: `text_corrected` 15,908 turns (3.98%) vs 26,774 values / 11,304 meetings.**
Doc-time [FACT arch/01 §7/252] vs live [FACT MP §2.4]. Live wins. Consequence: 11,304 meetings (67%) were extracted against now-deleted text — their legacy findings can never satisfy the R7/I7 quote gate, which is the *definitive* argument for re-deriving all findings rather than migrating them [DECISION MP §13.8; GF §19.1] (ADR-0016).

**CON-12 — God-module sizes: router 1,968→2,015 lines; sql_agent 5,691→5,776.**
[FACT arch/07 F5 vs MP §2.1]. Live wins; the code moved after the docs. Consequence for this package: **anchor claims to constructs, never line numbers** except where a source line is quoted as evidence of doc-time state.

### Tier 1 — count/scope disputes

**CON-13 — Handler count: "24+6=30 total" (hardcoded health), "~30" (CLAUDE.md) vs 32 on disk.**
- A: `/v2/admin/health` hardcodes `{atomic:24, composite:6, total:30}` [FACT arch/05 §1.5]; CLAUDE.md says "~30 handlers" [FACT arch/08 ISS-07].
- B: **32 handler modules = 26 leaf + 6 composite**; the authoritative count is the final line of `regression_check.py`'s period-coverage output, which enumerates disk [FACT MP §2.1; arch/03 §5/514].
- **Resolution:** descriptive — structural discovery from disk beats a hardcoded literal and a memory doc. 32 is the number.
- Consequence: health/status facts in the new platform are *derived from live state, never hardcoded* (doc 19); the capability catalogue (doc 04) maps from the 32-handler / 44-distinct-intent surface, not from the "30" folklore. Related: ~45 intent types enumerated, **44 distinct** (`executive_summary` listed twice) [FACT MP §2.1].

**CON-14 — Reachability-count dispute: "15" vs "16" vs "all 52 reachable".**
- Claims in circulation: 15 reachable intents (wrong); 16 (right **only** for the orchestrator registry scope); "all 52 reachable" (literally true, materially misleading — **36 of 52 answer only via the degraded `answer_v2` fallback**, an error path doing feature delivery) [FACT arch/08 PC + ISS-07].
- **Resolution:** all three are answers to different questions. The register's own rule is adopted: **a reachability count is meaningless without its scope statement.** Correct statements: orchestrator registry serves 16 intents via 17 keys; the remaining 36 are served only by the fallback path; total enumerated 52.
- Consequence: the new platform has **one** dispatch topology where the question cannot arise (I12); the capability registry asserts reachability structurally at startup — unreachable registrations fail the build (F38/ISS-11 lesson; docs 11, 12).

**CON-15 — Two divergent `executive_summary` implementations; interception order decides which runs.**
The composite handler vs the orchestrator's 5 SQL sub-handlers; routed via the legacy path it degrades to a *session count* (no `_INTENT_MAP` key → `_handle_count`) [FACT arch/04 §2/152, §3/227; arch/03 §7.2/612]. **Resolution:** both are real; the system is genuinely nondeterministic at the routing level — itself the finding. Consequence: I5 (never answer a different question) + single-registry dispatch; CAP-D10 executive packs get exactly one production path (doc 04).

**CON-16 — Pipeline step count: wrapper docstring "3-step" vs main() printing 1/6…6/6 vs runbook mandating 4 steps (+2 identity steps).**
[FACT arch/06 §7#1; arch/07 F10; arch/01 §5/204 — CLAUDE.md omits the mandatory `link_services_consultants` step entirely]. **Resolution:** the runbook-vs-wrapper contradiction is unresolvable in favour of either — the wrapper also *skips* ASR correction and omits `--require-corrected`, silently extracting from raw text (F10/B9). The truthful description: 4 ordered steps + 2 manual identity steps, with a wrapper that contradicts all of it. Post-ASR-deletion the legacy pipeline is 3 steps [FACT MP §2.4]. Consequence: doc 09's orchestrated DAG encodes **every** edge including identity linking; wrappers cannot diverge from the DAG because the DAG is the only runner.

### Tier 2 — doc-vs-code / doc-vs-doc drift (trust calibration)

| ID | Contradiction | Sources | Resolution (ladder) | Package consequence |
|---|---|---|---|---|
| CON-17 | Composite docstrings claim "all stats fetched in parallel"; execution is sequential (`gather_seq`) | arch/03 §4.4/496 vs code | Code wins | Docstring claims are rank-4 evidence, always |
| CON-18 | `pattern_detector` advertises "three-tier detection"; Tier 3 is dead code (zero importers); `context_sensitive=True` flags inert | arch/04 §8/609 | Code wins — two tiers | Dead capability claims excluded from the legacy feature inventory (doc 04 gap analysis) |
| CON-19 | Router module docstring describes the retired v1 architecture, never mentions its own live entry point | arch/03 §7.1/584 | Code wins | Same |
| CON-20 | `identity_models` docstring asserts "Groq never populates identity fields" — true today but enforced by nothing | arch/03 §6/571 | Claim unverifiable as a guarantee | Identity purity becomes a lint/test (doc 15; MP §2.6.6) |
| CON-21 | CLAUDE.md frames `sql_agent.py` as legacy/dead; it serves 16 intents on the hot path | arch/03 §7.1/583 | Code wins | The 16 live sql_agent intents are inputs to doc 11's metric registry scope |
| CON-22 | CLAUDE.md claims the V1 RAG chat was removed; `/v2/chat` remains mounted exposing all 52 intents | arch/08 ISS-07/08 | Code wins | Legacy surface inventory for parity (doc 20) includes `/v2/chat` |
| CON-23 | Determinism headers ("temperature=0, seed=42") vs router comment admitting the classifier is non-deterministic — hence 8 override layers | arch/05 §4.6 vs arch/07 §1.2 | Both true: params set, determinism NOT guaranteed (Groq seed best-effort, no fingerprint check) | Doc 14: determinism is never assumed from params; stability measured across repeated runs (GF §10.3); Lane-0 matching is non-LLM |
| CON-24 | arch/07 F18: intent/clarifier calls "pass no seed" vs arch/05 §4.3: they DO send seed 42 via `call_groq_raw` | arch/05 vs arch/07 | arch/05 wins (later, detail-verified against call sites) | Do not repeat 07's seed sentence |
| CON-25 | "Telemetry is never read" vs `GET /v2/admin/metrics` reads it (unauthenticated, pull-only) | arch/08 PC#1 | Register correction wins | The defensible finding is the auth/quality one (ISS-04/12/13), not "never read" |
| CON-26 | Extractor judged corrected text; portal reviewer reads raw (`/v2/transcript` selects `text` only) — flagged quote may match no visible turn | arch/04 §4/297; arch/01 §6/243 | Both true — the contradiction is the system's, not the docs' | "One accessor for the active transcript" requirement (docs 06, 08); reviewers always see exactly the text the extractor saw (I6/I7) |
| CON-27 | Upload stats misleading by construction ("updated" counts, `imported` never reported); `--dry-run --limit N` reports `processed=0` after real (rolled-back) work | arch/04 §9/673, §5/397 | Code behaviour is as described; the *labels* lie | Doc 09: ingestion-run reports reconcile by construction (`listed = valid + skipped(by reason) + errors`; F30/B19) |
| CON-28 | `verify_v2_environment.py` WARNs (not fails) on DB-only tables — blind to exactly the F3 condition; FAILS on models-only tables | arch/02 §9/570; arch/06 §5 | Asymmetry confirmed | Drift detection in the new platform fails closed both directions (doc 15 structural checks) |
| CON-29 | main.py docstring port 8002 vs Dockerfile 8011; ADMIN_PORTAL D3 admin-scope vs actual require_admin routes; dashboard "top intents" panel actually groups post-override `plan_type` | arch/05 §7.8, §1.3; arch/08 ISS-15 | Code/measurement wins in each | Minor; folded into the docstring-distrust rule and doc 19's honest-labelling rule |

---

## 3. OBSOLETE MECHANISMS REGISTER

Mechanisms present in the legacy system that the new platform must **not** carry forward. Each names its replacement and the owning document. "Never reintroduce" items are owner decisions; the rest are architecture recommendations with the cited defect as rationale.

| ID | Obsolete mechanism (legacy evidence) | Why it is dead | Replacing mechanism in the new design | Owner doc(s) |
|---|---|---|---|---|
| OBS-01 | **Multi-model ASR correction ensemble** — 7×2 detect panel, 3-model unanimous fix, propagation rounds, `text_corrected`/`transcript_corrected` flags, `v2_transcript_edits` (275,869 rows), `--require-corrected` [arch/04 §6; MP §2.4] | Owner decision 2026-08-02: results not good enough; deleted with its data; **never reintroduce under any name** [DECISION MP §2.4, §12-1b; GF §2.6] | Provider quality gating + `TranscriptSource` + `rebase_transcript`; interim deterministic read-time glossary (whole-word, logged, no write-back, flag-disabled, deleted when provider lands) | 06, 09; ADR-0007 |
| OBS-02 | **Magic-string identity** — `consultant_id='PENDING'` in a NOT NULL column; consumers must know to exclude it; stale copies survive backfills [arch/02 §7/491; arch/04 §9/678] | F5; a registry refresh would mint a fake consultant | Nullable FK + `resolution_status` enum + provenance on `core` crosswalks (`provider_meeting_map`, `internal_session_map`); unresolved identity is a first-class state | 08; ADR-0006 |
| OBS-03 | **EAV section store + exact double-writes** (`v2_meeting_sections` vs typed tables, 192,432 rows, string-literal keys) [arch/02 §2/161–172] | Duplicate truth with no consistency enforcement | Typed relational tables only; raw provider payloads archived once in object storage (`ingest` schema + S3/MinIO) | 08, 05 |
| OBS-04 | **DOM-scraped export** + runtime CDN libraries (SheetJS, html2canvas, jsPDF, no SRI) [arch/05 §6; arch/01 §3/88] | F6; a CSS rename empties the spreadsheet; CDNs fail on gov networks | I11: typed answer envelope → server-side renderers (XLSX/PDF/JSON); self-hosted frontend assets only | 11, 17, 18; ADR-0018 |
| OBS-05 | **Eight unsynchronized intent registries** + 236-line if/elif + per-request imports [arch/03 §7.2] | F8; desyncs already produce wrong answers (CON-15) | One governed capability registry + one metric/dimension registry; unregistered ⇒ build fails; reachability asserted at startup (I12) | 04, 11, 12 |
| OBS-06 | **Module-global runtime state** (clarification contexts, backup tasks) [arch/03 §7.3/621] | F13; breaks >1 worker; loses state on restart | All durable serving state in Postgres `serve` schema (conversations, clarification, budgets, job handles); Redis later cache-only | 12, 07 |
| OBS-07 | **Opt-in period filtering** — optional period helpers, `setdefault` injection losing to LLM values, `PERIOD_EXEMPT` composites reading bespoke params, bridge dropping `filters.*` [arch/08 ISS-01; arch/03 §4.2] | F6/F7/F16-legacy; quarters/halves/years silently missed | I4: period resolved deterministically pre-model, injected by harness, **absent from model schemas**, required compiler argument (omission = compile failure); explicit `ALL_TIME` sentinel | 11, 12 |
| OBS-08 | **Binary-dump schema provenance** — 3 parallel DDL mechanisms, dump-only tables, no version table [arch/06 §5; arch/02 §9] | F3; DB not rebuildable; DR broken | Alembic as the ONLY migration mechanism from day one; CI provisions fresh DB from migrations; drift detector fails closed | 07, 09; ADR-0004 |
| OBS-09 | **`.env` write-back + per-request `SELECT FOR UPDATE` token management** [arch/05 §3.1; arch/04 §5/378] | F28; server needs write access to its own config; globally serialises ingestion | Vault-injected secrets (OD-02); provider token state DB-canonical with in-process cache + refresh mutex; app never writes config | 09, 16 |
| OBS-10 | **Unbounded LLM fan-out swallowed to None** (~168 concurrent calls/meeting; rate-limited rounds silently under-detect) [arch/04 §6/480] | F19-legacy | Bounded worker pools on all provider fan-out (rate-limit control, not cost control); failed partition = `INCOMPLETE`, surfaced (I16) | 09, 13, 14; ADR-0015 |
| OBS-11 | **Legacy fallback endpoint doing feature delivery** — `/v2/chat` + `answer_v2` serving 36/52 intents as the "error path" [arch/08 ISS-07/08] | Fallbacks must be error paths | Lane 2 honest boundary with closed reason codes; no capability served by a degraded path; one dispatch topology | 12 |
| OBS-12 | **TRUNCATE-based, positionally-parsed Excel import** (49 columns by index; wipes linkage on every upload; hidden mandatory re-link step) [arch/04 §9/671–673] | F21 | Staged transactional import (load→validate→swap), header-name mapping failing loudly, versioned ingestion runs + reconciliation reports | 05, 09 |
| OBS-13 | **Excel file as the system of record for consultant/session identity** (~8% bridge match) [arch/05 §3.7] | F14-identity; identity cannot rest on a hand-uploaded file | Internal session data contract via approved mechanism (OD-07) + consultant directory contract; crosswalk tables with resolution provenance | 05, 08 |
| OBS-14 | **Arabic-HTML-string handler contract** + client format sniffing + `innerHTML` with no sanitizer/CSP [arch/03 §4.1/430; arch/05 §2.2–2.3] | F6/F18-legacy; no JSON API, no i18n, XSS surface | I11 typed result envelope; downstream renderers; escape-by-construction; CSP | 11, 12, 17, 18 |
| OBS-15 | **Unauthenticated data/ops plane** — 21/36 routes open, CORS `*`, anonymous pg_dump, default admin auto-seed [arch/05 TL;DR] | F2/ISS-04 | I14: OIDC + server-side RBAC deny-by-default; scoped CORS; ops endpoints out of the app process; no default credentials | 16; ADR-0017 |
| OBS-16 | **Review state by row-absence + UPSERT-in-place verdicts + hard-DELETE restore** (one decision per meeting, no history, no evidence fingerprint) [arch/02 §5/406; arch/04 §11] | F4/F22-legacy; stale decisions survive re-extraction | Append-only review events; verdicts fingerprint the reviewed evidence set; mechanical staleness invalidation on transcript/model/taxonomy change | 08, 18; ADR-0012 |
| OBS-17 | **Checkpoint-JSON resumability** (592 KB gitignored file, one developer's disk) [arch/06 §1] | State outside the database | Job/partition state in Postgres `jobs` schema; resume automatic on restart; stall detection | 09, 13; ADR-0015 |
| OBS-18 | **Hand-maintained 215-entry label→Arabic translation map** + open-coded normalization ×4 [arch/03 §4.2/441] | F20-legacy; loses to extraction churn | Closed enums at extraction (strict json_schema) + one normalization stage + governed taxonomy registry with Arabic display names | 10, 11 |
| OBS-19 | **String-predicate soft delete** (`val_verdict='wrong'` copy-pasted ×3 with drift) [arch/02 §3/296] | Shape-drifting duplicated predicates | Typed `validation_status` enum, single repository predicate, CHECK constraints | 08, 11 |
| OBS-20 | **Polymorphic soft pointers** (`record_id` int + free-text `table_name`; 1,382 rows unresolvable) [arch/02 §3/298] | F26-legacy | Typed FKs per validated table (`findings.validation_results` references `findings.findings`) | 08 |
| OBS-21 | **Opt-in local pre-commit gate against the production DB**, `--no-verify` advertised; no CI [arch/06 §4] | F11-legacy | CI on a seeded fixture DB (snapshot slice); golden suite; structural checks; gate-weakening forbidden (I18 + MP I13 semantics) | 15, 21 |
| OBS-22 | **V1 data layer, dead scripts, retired `/v3` planner, `frontend/app.py`, PII backup table** [arch/07 F9; arch/06 §3] | Dead weight; `/v3` is a lesson, not code | Excluded from snapshot scope (doc 20); `/v3` lesson encoded in doc 12 (deterministic pre-processing precondition); PII table handled as a data-protection item in doc 16 |
| OBS-23 | **Silence % ignoring overlapping speech** (negative ratios clamped to 0) [arch/04 §5/384] | F15-legacy | Interval-merge speaking-time computation, property-tested on overlapping-turn fixtures | 10 |
| OBS-24 | **Health/observability theatre** — `/health` without DB, hardcoded health facts, AVG-as-P95, 200-on-failure, no request IDs [arch/08 ISS-06/12/13/17] | I16 violations | Liveness vs readiness split; derived health facts; P50/P95/P99; error envelope + correlation IDs; R13 reasoning log | 17, 19 |
| OBS-25 | **`window.fetch` monkey-patching + localStorage bearer tokens + load-order-critical scripts** [arch/05 §6] | XSS-readable tokens; fragile boot | OIDC session handling; standard authenticated API client; bundled SPA (no script-order invariants) | 16, 18 |

---

## 4. SETTLED DECISIONS CARRIED FORWARD

Owner decisions that bind this package. Each is [DECISION] class with its citation; doc 22 records the corresponding ADR. None of these may be reopened by a package document without flagging a direct evidence contradiction (GF §2 preamble).

| ID | Decision | Source | Carried into |
|---|---|---|---|
| SD-01 | **No ASR correction, ever, under any name.** The ensemble and its data are deleted; transcript quality is an upstream provider-selection and gating problem. Only sanctioned interim aid: deterministic, whole-word, read-time, logged, flag-disabled glossary with no LLM and no write-back, deleted when the new provider lands | [DECISION MP §2.4, §12-1b (2026-08-02); GF §2.6] | ADR-0007; docs 06, 09; quality bar §14 |
| SD-02 | **`TranscriptSource` boundary + `rebase_transcript`.** Read.ai is interim; a cleaner provider is coming (timing unknown); provider swap must be invisible above the boundary; turn identity not portable; re-extract, never map indices; quote re-verification blocks the flip | [DECISION MP §2.5; GF §2.1, §9.1–9.2] | ADR-0007; docs 05, 06, 08, 09; OD-06 |
| SD-03 | **Groq is the primary generative-AI infrastructure**; task-specific model strategy; model IDs are registry/config data; non-generative embeddings/reranking may run locally with residency implications stated | [DECISION GF §2.4] | ADR-0013/0014; doc 14 |
| SD-04 | **Full legacy independence + one-time checksummed snapshot.** No runtime dependency on legacy DB/app/cache/workers/credentials; snapshot restored into `legacy_snapshot` inside the new DB; parallel-run via exported replay only, never a live DB link — and the legacy system keeps running and ingesting during the build | [DECISION GF §2.3, I10; MP I14, §13.8 (2026-08-02)] | ADR-0001, ADR-0016; docs 07, 20 |
| SD-05 | **Correctness over cost.** Never trade accuracy/coverage/auditability/full-corpus processing for token cost; cost measured and reported; **Lane 3 has no cost ceiling — scope declared, never refused**; rate limits and wall-clock remain engineered constraints | [DECISION GF §2.5; MP I15, §5.3 (2026-08-02)] | ADR-0015, ADR-0020; docs 13, 14, 19 |
| SD-06 | **Re-derive, never migrate, derived claims.** The snapshot carries the raw/authoritative layer only; all findings are re-extracted under the new pipeline and versioned open taxonomy (legacy findings cannot pass the quote gate anyway — CON-11) | [DECISION MP §13.8; GF §19.1] | ADR-0016; docs 08, 20 |
| SD-07 | **The owner signs the answer key.** Machine proposes; product owner (or delegated domain owner) approves/corrects/rejects; signed fixtures ARE correctness for subjective capabilities; one shared propose→review→approve workflow for answer key, taxonomy, cluster labels, violation review, aliases, publication | [DECISION MP §4.4 (2026-08-02); GF §6.5] | ADR-0012; docs 15, 18; OD-10/OD-11 |
| SD-08 | **Recurrence is semantic, not lexical.** Repeated questions, challenge synonyms, entity aliases, repeated decisions cluster by meaning with human-approved canonical labels; synonym dictionaries are seeds only | [DECISION GF §6.2; MP §4.1 R-P2] | Docs 10, 11; CAP-B3/C1/C4/C6/C8; EXP-04 |
| SD-09 | **Each period analysed fresh from its own sessions** — no priming by prior conclusions; prior taxonomies may classify but must not suppress newly dominant categories; packs surface actual sessions with IDs and quotes | [DECISION GF §6.3; MP §4.1 R-P3] | Docs 10, 11, 13; CAP-D10 |
| SD-10 | **The 8 violation types, verbatim, are the required initial reporting taxonomy** («تصنيف موحد»): VIOL-001 أسلوب تحذيري يقلل من شعور الدعم لدى المستفيد · VIOL-002 استخدام عبارات مطلقة · VIOL-003 غياب التبرير الكافي · VIOL-004 توجيه لمسار واحد بدون مقارنة · VIOL-005 طرح رأي شخصي كحقيقة · VIOL-006 توجيه حاد عالي التأثير · VIOL-007 إنهاء غير احتوائي للحوار · VIOL-008 تواصل خارج الإطار الرسمي. A curated **seed**, not the final universe; detection vocabulary (أسلوب غير مهني، تضليل، معلومات غير مؤكدة، تهكم، تسويق شخصي) must be reconciled to it with coverage gaps stated — today **تهكم and تسويق شخصي are detected by nothing**, and legacy implemented only 7 of 8 (VIOL-004 missing) | [DECISION GF §5-B4; FACT MP §4-B] | Docs 04 (CAP-B4/B5), 10; `tax` schema seeds |
| SD-11 | **Frozen packs + live analytical view, both required.** Published numbers are never edited; corrections are superseding reissues stating the cause (taxonomy/corpus/code/behaviour) | [DECISION GF §6.4] | ADR-0011; docs 08 (`packs`), 11, 13 |
| SD-12 | **Examples are seeds, not closed vocabularies** — every taxonomy has curated seed + offline discovery + human review/naming queue + versioning/lineage + backfill/left-censoring; the platform must be able to discover unlisted signals *and unlisted measures* with the same provenance guarantees | [DECISION GF §6.1; MP §4.1 R-P1] | Docs 10, 11; ADR-0011 |
| SD-13 | **Committed questions get deterministic/curated production paths** — never dependent on a free-form planner routing correctly; Lane 0 abstains when unsure | [DECISION GF §5 preamble; MP §5.1] | ADR-0008; docs 04, 12; EXP-05 |
| SD-14 | **«قطاع» is never one dimension.** Service category, government entity mentioned, and beneficiary true industry sector are distinct; the third is unavailable and must be declared so, never inferred from transcripts | [DECISION GF §5-C4, §5-D] | Docs 04, 08, 10, 11 (dimension registry) |
| SD-15 | **CAP-C5 is literal extraction only** («مجرد استخراج من النص»): government-entity friction output is *mentions in sessions*, never an assessment of an entity's institutional performance — a wording constraint on every renderer and pack | [DECISION GF §5-C5; MP §4-C] | Docs 04, 10, 18 (Arabic copy rules) |
| SD-16 | **No transcript-provider scope creep**: the platform does not build a meeting bot/recorder without a separate approval; STT-over-audio is a contingency to assess only | [DECISION GF §2.1, §9.4] | Docs 03, 06; OD-12 |

**Consequence for document authors:** any package text that (a) proposes correcting transcript text, (b) migrates a legacy finding row, (c) prices a Lane-3 job into refusal, (d) merges the three «قطاع» meanings, or (e) renders CAP-C5 as an entity ranking — contradicts a settled decision and fails review (§14 spine).

---

## 5. ASSUMPTIONS EXTRACTED → OPEN DECISIONS

Where evidence was insufficient, GF §1.2 requires recording an assumption and continuing — never blocking on an owner question. Each assumption below is registered as an OD-xx (doc 22 owns the full register with owner, impact, and revisit trigger); the safe working assumption is what this package plans against.

| OD | Assumption extracted from sources | Evidence gap that created it | Safe working assumption used by this package |
|---|---|---|---|
| OD-01 | Deployment lands in an approved Saudi environment (cloud region or on-prem) | No source names the approved target [GF §15, §22 require the decision] | Container platform in the approved environment; Docker Compose pilot → K8s optional (doc 07) |
| OD-02 | An approved IdP (OIDC/SSO) and secrets platform exist and can be integrated | GF §15.1 mandates SSO; no source names the IdP or vault | OIDC; Keycloak as broker if direct integration unavailable; vault-injected secrets (docs 16, 07) |
| OD-03 | Monsha'at's Groq account tier is non-enterprise | llama-3.3-70b shutdown 2026-08-16 applies to free/developer tiers; enterprise committed-spend unaffected [FACT groq-docs 2026-08-02]; account status unverified [GF §10.1 demands verification] | Non-enterprise; **the plan does not depend on llama-3.3-70b either way** (doc 14) |
| OD-04 | Sending pseudonymized transcript text to Groq is approvable | GF §15.2 requires a data-flow matrix; no standing approval documented | Approved for pseudonymized text; blocked for raw PII; matrix in doc 16 |
| OD-05 | CAP-C2 (symptoms/causes/actual ask) needs new extraction — the legacy columns intended for it are empty (0 of 67,082 challenge rows populated) | [FACT GF §5-C2 "some existing columns…are empty"; live measurement 2026-08-02] | Lane-3 pilot first, then corpus-wide extraction fields if validated (docs 04, 13) |
| OD-06 | Replacement transcript provider identity/timeline unknown | [FACT MP §2.5 "timing is unknown"] | Read.ai interim; benchmark harness provider-agnostic (doc 06; EXP-02) |
| OD-07 | Internal session data reachable via a read-only API/export with a freshness SLA | Legacy evidence is an Excel bridge (OBS-13); no source documents a real feed | Read-only API or scheduled export; contract in doc 05; EXP-01 measures join quality |
| OD-08 | Retention windows for raw payloads, superseded transcript sources, conversation logs, exports | GF §15.4 requires retention design; legacy has none [arch/07 §7-Q4] | 90d pre-rebase archives; ≥18mo audit events (docs 16, 19) |
| OD-09 | Beneficiary PII visibility per role | GF §15.1–15.2, I15; legacy exposes real names in handler output [arch/07 §7-Q6] | Nobody sees raw PII by default; stewards via break-glass with audit (doc 16) |
| OD-10 | Publication sign-off authority for packs | GF §6.5 names "product owner or delegated domain owner" without an org mapping | Product owner signs (docs 15, 18) |
| OD-11 | Review-role powers (can non-admin reviewers reject?) and append-only history as compliance requirement | Open questions in [arch/07 §7-Q7–Q8]; legacy asymmetry undocumented | Append-only always; retire/restore admin-only (doc 18) |
| OD-12 | Lawful, reliable audio access for the STT contingency | GF §9.4 conditions the path on it; no evidence either way | Not available; STT assessed on paper only (doc 06) |
| OD-13 | A second Read.ai OAuth client can be issued for the new platform | Token rotates on refresh — sharing breaks both systems [FACT MP §13.8]; issuance lead time unknown | Obtainable but long-lead; **requested at project start**; blocks cutover, not the build (docs 09, 20, 21) |

Two further working assumptions used package-wide, registered under existing ODs rather than new ones: (a) the live-measurement figures in §7 that MP §2 does not itself carry (pressure-class distribution, step-clarity distribution, entity-string distinctness, speaker-role coverage, `service_category` 77.3%) were measured 2026-08-02 by the planning pass and are subject to the same re-measurement discipline [ASSUME OD-07 — re-measured through the committed script once the internal-data contract fixes source access]; (b) no additional internal outcome/follow-up source exists beyond what GF §2.2 lists [ASSUME OD-07].

---

## 6. HOW LEGACY DEFECT IDS MAP INTO THIS PACKAGE (index)

The arch docs' F- and ISS- registers are cited across the package by their original IDs. The canonical set every document may cite without re-deriving (full mapping table in doc 07 §risk-mapping; countermeasure detail in the owning docs):

- **Integrity:** F1 (no-PK violations, 1.84× inflation) → doc 08 constraints; F3 (unbuildable schema) → ADR-0004; F4/F22 (review fingerprints) → docs 08/18.
- **Correctness:** F6/F7/F16-legacy + ISS-01/ISS-02/ISS-10 (period + silent-substitute) → I4/I5, docs 11/12; F8 (registries) → I12; F36 (prompt/writer taxonomy drift) → single enum source, doc 10.
- **Security:** F2/ISS-04, F14-legacy, F20 (PII) → I14/I15, doc 16.
- **Pipelines:** F10/F11/F21/F22-legacy, F30/B19 (reconciling counters) → doc 09; F19-legacy/F27 (fan-out, 429 visibility) → docs 09/13/14.
- **State/ops:** F13 (globals), F28 (.env), ISS-06/12/14/15/16/17 (observability) → docs 12/19.
- **Method:** F15 (silence overlap), F16 (role attribution), E.0 rules → doc 10.

---

## 7. PLANNING BASELINES (measured 2026-08-02)

**Discipline** [DECISION GF §1.3]: these are planning baselines, **not permanent truths**. The legacy repo has no measurement script [FACT MP §2 preamble]; this package therefore requires a committed, re-runnable measurement script (`scripts/measure_corpus.py` in the new repo — deliverable of doc 20's snapshot workflow, mirrored in doc 09's reconciliation checks) [REC — alternative: ad-hoc SQL in a runbook; rejected because unrepeatable numbers are how the 2026-07-16 figures went stale unnoticed; revisit: never]. **No number below may appear in capacity, migration, or stakeholder material without re-measurement through that script.**

| # | Measure | Value (2026-08-02) | Source | Planning use |
|---|---|---|---|---|
| B1 | Meetings (provider hub) | 16,911 | [FACT MP §2.3] | Corpus size for backfill/re-extraction sizing (~1 Groq call/session ⇒ ~16.9k calls full pass) |
| B2 | Transcript turns | 399,501 | [FACT arch/01 §7/252; unchanged order at live check] | Evidence-index sizing; chunking strategy (doc 11) |
| B3 | Corpus span | **14 calendar months, 2025-05…2026-06** | [FACT MP §2.3] | Lane-3 partition count; **partition lists discovered from data, never hardcoded** |
| B4 | Monthly volume | 556–1,512 meetings/month (**2.7× swing**) | [FACT MP §2.3] | Rates per 100 sessions mandatory for all period comparisons (doc 10) |
| B5 | Legacy knowledge rows (11 fact tables) | ~409k total; challenges 67,082; recommendations 68,136; topics 61,909 | [FACT arch/02 §3/278–290] | Sizing only — **all re-derived, none migrated** (SD-06) |
| B6 | Violations table | 6,794 rows / 3,691 distinct / 3,103 dups (≤1.84× inflation), **no PK** | [FACT arch/02 §8; MP §2.3] | Parity expectations (counts must FALL after dedup — direction stated per MP §13.9); never quote legacy violation counts |
| B7 | `consultant_id='PENDING'` | **57 meetings (0.3%)** — was 1,629/9.6% on 2026-07-16 | [FACT MP §2.3] (CON-06) | Identity migration scope; crosswalk edge-case fixtures |
| B8 | Services report | **94,963 rows**; bridge matches ≈8% of rows / under half of meetings | [FACT MP §2.3; live measurement 2026-08-02] (CON-07) | EXP-01 join-rate baseline; internal-contract urgency (OD-07) |
| B9 | Ratings rows | ~24,171 | [FACT live measurement 2026-08-02] | CAP-D1/D2 coverage expectations |
| B10 | `service_category` on business profiles | **77.3% populated, 48 distinct values** — was all-NULL on 2026-07-16 | [FACT live measurement 2026-08-02] (CON-09) | Dimension availability for CAP-B1/C4/D6; `business_sector` remains UNAVAILABLE |
| B11 | Government entity mentions | 21,561 mentions; **4,372 distinct surface strings** | [FACT arch/02 §3/290; live measurement 2026-08-02] | CAP-C5 entity-resolution workload; `tax` ENT registry + aliases sizing |
| B12 | Pressure-signal classes | urgency 8,832 / confusion 6,987 / frustration 6,776 / **distress 544 / other 279** | [FACT live measurement 2026-08-02] | CAP-C7 class imbalance → contrastive method + small-class stability rules (doc 10) |
| B13 | Step-clarity distribution | clear 6,274 / partial 6,824 / none 2,486 / null 1,327 | [FACT live measurement 2026-08-02] | CAP-B1 denominator/null policy; re-derivation sanity ranges |
| B14 | Speaker-role coverage | **5.7% of turns lack `speaker_role`**; turn timing 100% present | [FACT live measurement 2026-08-02] | I7 quote requirements; role-resolution step in doc 09; coverage exclusions in every role-dependent metric |
| B15 | Embedded chunks (legacy) | 109,167 vectors, 384-dim, model provenance unrecorded | [FACT arch/01 §7/255; arch/02 §11] (CON-04) | NOT migrated; evidence index rebuilt with provenance (I13) |
| B16 | ASR artifacts at deletion | `text_corrected` 26,774 values / 11,304 meetings; edits table 275,869 rows; 15,740 flagged | [FACT MP §2.4] (CON-10/11) | Excluded from snapshot; explains why legacy findings fail I7 (SD-06) |
| B17 | CAP-C2 source columns | 0 of 67,082 challenge rows carry symptoms/root-cause/actual-ask values | [FACT live measurement 2026-08-02; GF §5-C2] | OD-05 route decision (Lane-3 pilot first) |
| B18 | Review workflow usage | `v2_violation_reviews` 0 rows vs 3,033 violation-bearing meetings; `v2_feedback` 3 rows | [FACT arch/01 §7/281–282] | No review data to migrate; adoption is a UX/process problem (doc 18) |

---

## 8. HOW THE REST OF THE PACKAGE USES THIS DOCUMENT

1. **Cite, don't re-derive.** A document needing a legacy fact cites `doc 02 §1/§7` (or the CON/OBS/SD ID); if it needs a number not in §7, the number must be added here (with measurement date) rather than introduced ad hoc.
2. **Contradiction hygiene.** The Tier-1 false premises (CON-01…CON-05) are never repeated as fact anywhere in the package — spine rule 13.8.
3. **Baselines expire.** Every §7 figure is stamped 2026-08-02; the committed measurement script re-stamps them before implementation gates (GF §1.3; doc 23's start gate requires a fresh run).
4. **Open decisions live in doc 22.** §5's OD mapping is the extraction record; doc 22 carries owners, impacts, and revisit triggers.

*End of document 02.*

# 00 — Package Index and Implementation Start Gate
**Platform:** Nwafeth Intelligence — منصة نوافث لذكاء الجلسات الاستشارية (Monsha'at Advisory Session Intelligence Platform)
**Status:** Draft for owner review · amended per the Owner Amendment 2026-08-03 · **Date:** 2026-08-02 (amended 2026-08-04) · **Author:** Planning package (Fable 5)
**Depends on:** 01–25 + PLAN_CHANGELOG + TRACEABILITY_OWNER_AMENDMENTS (this is the map of all of them) · **Feeds:** the owner's approval decision and the Opus 5 implementation start
**Sources used:** GREENFIELD §1.1, §23-00, §25; OWNER-AMENDMENT (2026-08-03) §0, §12, §14–§15; doc 22 §5 (approval register); doc 21 §0–§1 (phases, gates); doc 23 (start conditions); doc 25 (slice protocol); CORE-BRIEF in full

This is the front door of the 26-document planning package (plus two amendment-tracking files) for the greenfield **Nwafeth Intelligence Platform (NIP)** — the independent, Arabic-first advisory-session intelligence platform that replaces the legacy application and Read.ai's analytics role for Monsha'at. Read §1 to find your path through the package, §4 for what is settled versus recommended, §5 for the decisions leadership must sign, and §6 for the exact conditions under which implementation (Opus 5, per docs 25/23) may begin.

**This revision (2026-08-04) integrates the binding Owner Amendment of 2026-08-03** — the Sol 5.6 review accepted by the owner as a binding change request [DECISION owner 2026-08-03]. Its effect in one line: delivery becomes incremental vertical slices **VS-00…VS-12** with an owner stop-accept gate after every slice (SD-18), DataHub becomes the first-class `SRC-DATAHUB` source family, and the operational product (nightly run «التشغيل الليلي الموحد», Session 360, «اشتباه مخالفة», the monthly infographic «نبض خدمة الاستشارات والإرشاد — ملخص الشهر», Action Center) moves to the centre, with the agent as an interface above it. `PLAN_CHANGELOG.md` records what changed per document; `TRACEABILITY_OWNER_AMENDMENTS.md` maps every amendment requirement to its owning document, feature IDs, slice, and acceptance reference.

---

## 1. Recommended reading order — four audiences

| Audience | Read, in order | Skip / skim |
|---|---|---|
| **Owner / leadership** | 01 (الملخص التنفيذي) → **24 (الباك لوق — Wave A أولًا) → 25 (شرائح التسليم VS-00…VS-12 وبوابات التوقف)** → 03 (النطاق والشخصيات) → 04 §3 (capability matrix only) → 22 §5 (approval table) → this document §5–§6 → 18 (تجربة الاستخدام، للاستئناس) | All engineering documents; 01 §8 repeats every decision required of leadership in Arabic; PLAN_CHANGELOG §1–§2 summarizes this revision |
| **Architect / reviewer of the design** | 02 (evidence + contradictions, incl. §1.11 the Owner Amendment) → 07 (architecture) → 08 (data model) → 12 (serving lanes) → 13 (Lane-3 factory) → 11 (semantic layer) → 09 (pipelines, incl. التشغيل الليلي الموحد) → 06 (provider strategy) → 14 (models + detector lifecycle) → 16 (security) → 19 (ops) → 20 (migration) → 22 (ADRs) → 25 (execution overlay) | 05 as reference; 10 §3 per-capability methods on demand |
| **Implementer (Opus 5)** | **25 first and in full (slice order VS-00…VS-12, stop-accept protocol, ticket DoD) → then 23 in full (ticket inventory, conventions, MUST-NOT list)** → 07 §8 (module layout) → 08 (DDL) → 04 (registry contract) → 11 → 12 → 09 → 13 → 15 (eval discipline) → 17 (API) → 14 (models) → 16 (controls) → 21 (phase DoD) | 01/03/18 for product context when building UI-facing epics; 02 when a source fact needs verifying |
| **Reviewer / steward (Monsha'at-side reviewers, data steward, compliance)** | 15 §3 (answer-key protocol) → 18 (شاشات المراجعة، ومنها SCR-08 «اشتباه مخالفة») → 10 §1–§2 (method charter) → 04 §7–§9 (violations vocabulary, lifecycle) → 16 §2–§4 (RBAC, PII, outbound matrix) → 22 §3 (open decisions naming them as owners) | Deep engineering documents; doc 21 §2 shows where their time is on the critical path (◐OWNER items) |

The dependency spine for a cover-to-cover read is simply numeric order: 01→23 was authored so that no document depends on a later one except by explicit ID reference.

**What each audience should be able to answer after their pass:**

- *Owner/leadership:* why the rebuild is necessary and what it replaces; what the platform will and will not promise each user group; which twenty decisions carry their signature (§5) and which five assumptions carry the most risk; what "success" looks like at cutover (doc 01 §9); **which backlog items are committed to the earliest slices (PB-014 + PB-006…PB-010) and that everything else is a standing consultation list awaiting their approval (SD-21, doc 24); what they will see at each L1 preview and what a stop-accept gate asks of them (doc 25)**.
- *Architect:* why every legacy defect in CORE-BRIEF §12 is structurally unreachable in this design; how the four lanes, the semantic layer, and the Lane-3 factory divide the answer space; where every invariant I1–I18 is enforced (doc 07 §5.10, doc 23 §2); which decisions are reversible (revisit triggers) and which are doctrine; **how the VS-00…VS-12 overlay maps onto phases P0–P7 without weakening a single gate (doc 25; CON-31)**.
- *Implementer:* **the slice order VS-00…VS-12 and the stop-accept protocol — the first executable order after approval is VS-00 only (doc 25)**; the epic inventory, migration order, and per-PR evidence bar (doc 23 §5/§6/§12); where prompts, registries, schemas, and tests live in the tree; what must never be built (doc 23 §13, extended with the Amendment §10.4 prohibitions).
- *Reviewer/steward:* how a candidate becomes ground truth (propose→review→approve, doc 15 §3); what their weekly time commitment is (~45 h/month labelling pool + owner signing sessions, doc 21 §2); which queues they own — now including «اشتباه مخالفة» (SCR-08) and «مشكلات الربط والبيانات» — and what append-only means for their decisions.

### 1.1 Inter-document dependency sketch

```mermaid
graph TD
  D02[02 source map] --> D04[04 capabilities]
  D02 --> D07[07 architecture]
  D04 --> D05[05 source contracts]
  D05 --> D06[06 provider strategy]
  D05 --> D08[08 data model]
  D06 --> D08
  D07 --> D08
  D08 --> D09[09 pipelines]
  D04 --> D10[10 methodology]
  D08 --> D11[11 semantic layer]
  D10 --> D11
  D11 --> D12[12 serving]
  D12 --> D13[13 Lane-3 + RAG]
  D12 --> D14[14 models]
  D13 --> D14
  D04 --> D15[15 evaluation]
  D14 --> D15
  D08 --> D16[16 security]
  D12 --> D16
  D11 --> D17[17 API]
  D16 --> D17
  D17 --> D18[18 UX ar]
  D09 --> D19[19 observability]
  D08 --> D20[20 migration]
  D15 --> D20
  D15 --> D21[21 roadmap]
  D20 --> D21
  D21 --> D22[22 ADR/OD]
  D22 --> D23[23 handoff]
  D21 --> D23
  D04 --> D24[24 product backlog]
  D21 --> D24
  D24 --> D25[25 incremental delivery + acceptance]
  D21 --> D25
  D25 --> D23
  D01[01 exec summary ar] -.summarizes.-> D22
  D03[03 scope ar] --> D04
```

(The 2026-08-04 amendment pass adds docs 24/25: doc 25 is the execution overlay both 21 and 23 point to; `PLAN_CHANGELOG.md` and `TRACEABILITY_OWNER_AMENDMENTS.md` sit outside the dependency spine as tracking files.)

## 2. Document map

Sizes are the current draft line counts (guide to reading effort, not importance). For docs 01–23 they are the pre-amendment (2026-08-03 consistency-pass) counts; several documents grow in the 2026-08-04 amendment pass — `PLAN_CHANGELOG.md` §3 records exactly what changes per document. Language: docs whose filename ends `-ar` are formal Arabic; all others are English with Arabic product labels preserved.

| # | Document | Size | What it contains — and why it exists |
|---|---|---|---|
| 00 | This index | ~210 | Map, reading order, precedence rules, package status, owner-approval list, implementation start gate. |
| 01 | `01-executive-summary-ar.md` | 247 | Arabic executive summary for leadership: why a new platform, what it replaces (and does not), expected value per user group, the architecture in executive language, phases and critical risks, and the decisions required from leadership (§8 mirrors doc 22 §5). |
| 02 | `02-source-map-and-contradictions.md` | 456 | The evidence foundation: source map over MASTER_PROMPT + all eight legacy arch docs + the GREENFIELD prompt, the contradiction log with resolutions under the §3 precedence ladders, the obsolete-mechanisms register, settled decisions carried forward, and the measured 2026-08-02 planning baselines every other document cites. |
| 03 | `03-product-scope-and-personas-ar.md` | 382 | Arabic product definition: hard boundaries (in/out of scope), the five personas with full needs and anti-promises, target outcomes, the twelve binding user journeys (ر1–ر12), and the committed-capability table in product language. |
| 04 | `04-capability-catalogue.md` | 1,097 | **The product contract.** All 26 committed capabilities CAP-A1…CAP-D10 as registry entries: exact registry JSON Schema + DDL sketch, τ/δ routing calibration semantics, full per-capability entries, the violation detection-vocabulary ↔ VIOL-001…008 reporting-taxonomy reconciliation with named gaps (تهكم، تسويق شخصي), the «قطاع» three-way disambiguation, the capability lifecycle state machine, and the Lane-3 promotion path (CAP-E series). The 2026-08-04 pass adds the operational capabilities CAP-OPS-01…CAP-OPS-12 under the same registry discipline. |
| 05 | `05-data-source-contracts.md` | 645 | One contract per source: SRC-READAI, SRC-TSP (future provider), SRC-SNAP, and the internal contracts — **restructured in the 2026-08-04 pass as the `SRC-DATAHUB` family** (sub-contracts -SESSION / -BENEFICIARY-EVAL / -CONSULTANT-EVAL / -OUTCOME / -DIRECTORY / -REFERENCE; the former SRC-INT/SRC-EVAL/SRC-OUT/SRC-DIR/SRC-REF become views of the family; Excel upload = temporary fallback) — each with ownership, freshness SLA, keys, cursoring, idempotency, update/delete semantics, PII classification per field group, starter data dictionaries, the Field Authority Matrix, DLQ state machine, and the reconciliation/join-rate KPI model behind EXP-01 and CAP-D9. |
| 06 | `06-transcript-provider-strategy.md` | 570 | The `TranscriptSource` boundary and normalized record contract, transcript storage shape, the ten-step `rebase_transcript` state machine with the quote re-verification gate, the EXP-02 provider benchmark (stratified sample, 12 dimensions, hard gates G1–G8, go/no-go sheet), deterministic quality tiers A–D, the deletable read-time glossary, and the assess-only Groq Whisper STT contingency. |
| 07 | `07-target-architecture.md` | 809 | C4 context/container/component views, layer-by-layer specification (GREENFIELD §8.1–8.8), the invariant-enforcement map, the online/offline separation contract, all 16 technology decisions TD-01…TD-16 with alternatives and revisit triggers, the monorepo module layout + import-linter dependency contracts (feeds doc 23), and the failure-domain/degradation matrix («no cell answers differently, silently»). |
| 08 | `08-data-model-and-erd.md` | 1,453 | **The canonical data model**: one PostgreSQL database, eleven schemas, 78 tables with keys, constraints-as-pins, ERDs per domain, the identity strategy (surrogate + ULID `session_uid`), the `'PENDING'`-unrepresentable resolution model, versioning/lineage rules L1–L10, the PII boundary map with masking views, partitioning/index doctrine, volume plan, and the deliberately-not-modelled list that makes honest refusals possible. |
| 09 | `09-ingestion-reconciliation-and-pipelines.md` | 627 | The orchestrated ingestion DAG with coded edges, per-session pipeline state machine (rows, not flags; content digests for automatic invalidation), per-adapter workflow specs, the M1–M3 identity match ladder with steward queues, data-quality gates, bounded model-call semaphores and the typed-failure contract (no silent empties), scheduling/priorities, and reprocessing playbooks PB-01…PB-05 including rebase integration. The 2026-08-04 pass adds the full «التشغيل الليلي الموحد» chapter (Amendment §4: 16 steps, run states, Morning Digest, month-close trigger). |
| 10 | `10-analytical-methodology.md` | 802 | The method charter MR-01…MR-16 (rates per 100, DISTINCT sessions, n≥30 + Wilson, confound checks, no circular labels, BH correction, left-censoring, stated-vs-inferred cause…), per-capability method sheets for all 26, the Arabic claim-form rules the renderer enforces, the R-P2 semantic clustering methodology with quality metrics, and the coverage-block definition with its closed exclusion vocabulary. |
| 11 | `11-semantic-layer-and-curated-capabilities.md` | 575 | The metric/dimension/filter registries (code-as-data, hash-versioned), the deterministic spec→SQL compiler contract and its stage log, repository-only SQL discipline, period/coverage enforcement (I4/I8), suppression mechanics, the typed result envelope every lane emits, the starter metric registry, and the curated-capability contract with the SPEC/CURATED structural boundary check. |
| 12 | `12-agentic-serving-architecture.md` | 809 | The serving turn lifecycle: deterministic pre-processing (normalization, three-valued period resolution, entity resolution, closed anaphora table), the lane decision tree, Lane-0 matcher, the 11-tool toolbelt with the argued orchestrator-controlled split, tool JSON Schemas, the executor-enforced planner state machine, budgets/circuit breakers, planner + composer prompts, the deterministic verifier suite (R6/R7), prompt-injection isolation, reason-code Arabic strings, and the R13 reasoning log. |
| 13 | `13-on-demand-analysis-and-custom-rag.md` | 879 | The Lane-3 factory: the eight-path trigger decision tree, model-proposed/system-validated analysis schemas (meta-schema), frozen corpus manifests with content-derived digests, MAP (one session per call) → VERIFY (deterministic, immediate) → REDUCE (all numbers born in code), the job state machine with kill/resume/cancel, INCOMPLETE semantics, caching and promotion thresholds, the nine-step custom-RAG lifecycle (filter-first always), and sequence diagrams. |
| 14 | `14-groq-model-and-inference-strategy.md` | 623 | The verified 2026-08-02 Groq catalogue (incl. llama-3.3-70b deprecation and the strict-schema support matrix), the eight task-role model matrix (gpt-oss-120b core, 20b triage, safeguard shadow-mode), the EXP-03 benchmark plan with per-family gates, structured-output and tool-use policy, sync vs Batch decision table, rate-limit engineering, the model/prompt registries and single inference entry point, and deprecation operations (I17). |
| 15 | `15-evaluation-answer-oracle-and-golden-suite.md` | 748 | The evaluation foundation: twelve labelled datasets DS-01…DS-12 with κ bars and held-out discipline, the answer-key oracle protocol (machine proposes, human signs; mechanical scoped staleness), the T-01…T-17 test matrix, eight hard gates HG-1…HG-8 and quantitative targets QT-01…QT-12, CI tiers with honesty counters, the no-weakening ratchet, pinning discipline, the committed synthetic fixture corpus, and labelling economics (~710 h launch). The 2026-08-04 pass adds missed-case + unflagged-sample datasets, per-category recall estimation, per-slice acceptance suites, the infographic payload-identity test, and EXP-11. |
| 16 | `16-security-privacy-and-compliance.md` | 657 | Threat model (15 threats → controls → residual risk → tests), OIDC + 7-role RBAC matrix with Postgres-level mirroring, data classes P0–P3 and retention schedule, the Groq outbound data-flow matrix + pseudonymization + fail-closed payload gate (OD-04), prompt-injection architecture and the EXP-10 adversarial catalogue, secrets, encryption, the full audit-event catalogue, PDPL/NCA alignment, deployment checklist, and incident/retraction playbooks. |
| 17 | `17-api-contracts.md` | 722 | The versioned JSON API: 27-endpoint inventory with permissions and idempotency rules, RFC 7807 error model (no 200-on-failure, ever), chat/turn envelope carrier, clarification and deep-job-offer round-trips, the job API, evidence retrieval (access-audited), packs/publications, review and taxonomy APIs, exports, admin, pagination/rate-limit/versioning policy, and cross-cutting contract tests. |
| 18 | `18-ux-admin-and-review-workflows-ar.md` | 668 | Arabic screen-by-screen UX specification: shared foundations (RTL, الأرقام والتقويم، قاموس التحوط، النصوص الثابتة لكل حالة), the twelve journeys as screens SCR-01…SCR-12 plus SCR-13 «اللوحات» (per-persona dashboards) — chat, structured answer shell, evidence viewer, closed clarification, deep-job offer/progress, artifact archive, promotion queue, violation review, governance queue, publication centre, data-quality board, export dialog, dashboards — with role × screen matrix and UI-generated audit events. The 2026-08-04 pass adds SCR-14…SCR-22 (operations-first build order) and upgrades SCR-08 to «اشتباه مخالفة» per Amendment §5; chat moves later in the build order. |
| 19 | `19-observability-operations-and-dr.md` | 620 | Telemetry that cannot lie: stack, end-to-end request-id/trace propagation across async hops, the full metric catalogue per plane (every GREENFIELD §17 bullet cross-checked), model-call and R13 log records, PII-free request reconstruction procedure, dashboards, alert severities and rules, typed degraded states, runbooks RB-1…RB-9, capacity model, pgBackRest backup/restore drills, and DR targets. |
| 20 | `20-migration-parallel-run-and-cutover.md` | 575 | The one-time checksummed snapshot: exact contents, transfer PII handling, restore + schema-to-schema mapping with idempotent reruns, reconciliation balance sheet, `taxonomy_version=0` crosswalk, quote verification, the ~500-session CI fixture slice, re-derivation plan, independent ongoing ingestion (OD-13 credential, watermark handover), the external replay parity harness with six-way difference classification, pilot/cutover gates G0…G5, and rollback triggers. |
| 21 | `21-delivery-roadmap-and-work-breakdown.md` | 761 | Dependency-based delivery: phases P0–P7 each with objective, prerequisites, deliverables, epics, tests, security controls, measurable exit criteria, rollback and risks; the experiment gates EXP-01…EXP-11 with thresholds and fallbacks; the critical-path narrative (owner-paced items flagged ◐OWNER); team model, RACI, and the roadmap risk register. As of 2026-08-04 the phases remain the dependency/gate framework while execution follows the VS-00…VS-12 overlay (doc 25 owns the protocol; release waves are Wave A/B/C). |
| 22 | `22-architecture-decisions-and-open-decisions.md` | 616 | The single registry: ADR-0001…ADR-0020 in full (context, decision, alternatives, consequences, revisit triggers), the OD-01…OD-35 open-decision register with owners/safe assumptions/impact/needed-by phase (OD-26/OD-27 from the consistency pass; OD-28…OD-35 from the Owner Amendment; OD-07/OD-10 extended; next free OD-36), the explicit assumption log A-01…A-29, the ID-collision resolutions from parallel authoring, the owner-approval table, and register maintenance rules. |
| 23 | `23-opus-5-implementation-handoff.md` | 749 | **The implementation handoff** — mission, I1–I18 with enforcement mechanisms, repository structure, the 20-epic ticket inventory (re-sequenced and split ≤4–8h under the VS order), migration/API/test/prompt build orders, coding conventions, per-phase definition of done, per-PR evidence requirements, the MUST-NOT list (extended with the Amendment §10.4 prohibitions), and the topic cross-index. Re-issued 2026-08-04 as a small-sequential-ticket protocol: Opus 5 reads doc 25 first, then starts here — VS-00 only, stop-accept before VS-01. |
| 24 | `24-product-backlog.md` | new (2026-08-04) | **The product backlog** (Amendment §8 + §11 as full story cards): PB-001…PB-018 (Wave A), PB-101…PB-120 (Wave B), PB-201…PB-212 (Wave C) — each with primary user, problem/value, data + capabilities, sensitivity, dependencies, testable acceptance, release wave, feature flag; Value/Risk/Dependency ranking; the single statement of the amendment-priority→wave mapping; the SD-21 standing-consultation rule. |
| 25 | `25-incremental-delivery-and-acceptance.md` | new (2026-08-04) | **The execution protocol** (Amendment §9 + §10): vertical slices VS-00…VS-12 with demo script, acceptance checklist and stop gate each; exposure levels L0–L3; the §9.4 mandatory stop rule; ticket sizing + DoD; the agent state-file template; the first five VS-00 coding tickets as specifications only. Implementers read this before doc 23. |
| — | `PLAN_CHANGELOG.md` | 262 | Amendment-pass record (Owner Amendment §0.4): what changed per document 00–23, what stayed, the new files, resolved conflicts CON-30…CON-34, still-open decisions OD-28…OD-35 + carried §13 assumptions, SD-18…SD-22, the supersessions register, and the pass-verification checklist. |
| — | `TRACEABILITY_OWNER_AMENDMENTS.md` | 318 | Requirement-by-requirement matrix over Amendment §§2–11 and §12: every requirement → owning document/section → PB/CAP-OPS IDs → vertical slice → acceptance reference → status (integrated / integrated-with-assumption OD-xx / deferred-to-backlog-consult SD-21); Wave-A binding table; reverse index; maintenance rules. |

## 3. Source precedence (GREENFIELD §1.1, restated — binding on every document)

Two ladders, never mixed [FACT GREENFIELD §1.1]:

**Descriptive** (what the legacy system does/did):
1. running code and live database measurements (2026-08-02 committed scripts);
2. MASTER_PROMPT measured facts dated 2026-08-02;
3. the arch/01–08 documents (2026-07-16);
4. docstrings, comments, older memories, assumptions.

**Prescriptive** (what to build):
1. owner requirements in the GREENFIELD prompt **together with the Owner Amendment (2026-08-03; Sol 5.6 review accepted by the owner — registered in doc 02 §1.11) at the same rank; within this rank the amendment, being later-in-time owner instruction, prevails over pre-amendment text wherever they differ**;
2. the non-negotiable principles I1–I18 (GREENFIELD §7);
3. settled owner decisions carried forward or issued since (SD-01…SD-22, doc 02 §4);
4. this package's recommendations, with rationale and alternatives;
5. current legacy behaviour only where it remains justified.

The Owner Amendment therefore sits **above every package recommendation** and below nothing except where I1–I18 would constrain it — and it constrains nothing there: its §0.5 no-weakening list maps 1:1 onto I2, I3, I4, I5, I7, I9, I6, the review-first rule, and I15 [DECISION owner 2026-08-03, Amendment §0.5]. The five places where the amendment and pre-amendment package text genuinely diverged are resolved as CON-30…CON-34 in doc 02 §2 (Tier 3) and only there.

Corollaries the package applies throughout: current behaviour is never an argument for keeping it; live counts are planning baselines that must be re-measured by committed script before use in capacity or stakeholder material (GREENFIELD §1.3); contradictions are resolved in doc 02 and only there.

## 4. Current package status — what is settled vs recommended

**Settled owner decisions** (tagged `[DECISION]`; not up for re-argument, only for confirmation): greenfield rebuild with total legacy runtime independence (I10) · Groq as primary generative-AI infrastructure · correctness over token cost, Lane-3 jobs have **no cost ceiling** · no ASR correction, ever · transcripts immutable and provider-versioned; Read.ai is interim · the machine proposes, the human disposes (sign-off defines correctness for subjective capabilities) · the 16 committed questions + 10 D-family additions are the product contract · four serving lanes with deterministic paths for committed questions · frozen publication + live view duality · **no self-hosted generative models — all generative inference on Groq; embeddings are the only local models, CPU-only (no GPU exists or is planned)** [DECISION owner 2026-08-03; SD-17 in doc 02].

**Amendment decisions 2026-08-03** (recorded as SD-18…SD-22 in doc 02 §4; same standing as the list above): **SD-18** incremental vertical-slice delivery VS-00…VS-12 with a mandatory stop-accept after every slice (demo → acceptance tests → gaps recorded → owner approval), L1 owner previews on synthetic/staging data from the earliest slices, production gates guarding L2/L3 only, no fully-parallel multi-epic development by one agent · **SD-19** the monthly leadership infographic «نبض خدمة الاستشارات والإرشاد — ملخص الشهر» as a standalone early product (CAP-OPS-08, PB-014, VS-05; web+PDF+PNG byte-identical; structured-data-only rendering; approved-violations only; no consultant names in the general edition) · **SD-20** the violation-suspicion queue «اشتباه مخالفة» as an early product (VS-02; suspected ≠ approved everywhere; finding/case/event append-only separation; VIOL-008 first, then one type at a time; six closed decisions) · **SD-21** backlog governance — PB-014 + PB-006…PB-010 committed to the earliest releases, everything else requires explicit owner consultation before entering any implementation slice · **SD-22** governed detector learning through versioned datasets, shadow evaluation, documented promotion, canary and instant rollback — never online self-learning; consultant names/ratings are never detector features.

**Recommendations awaiting acceptance** (tagged `[REC]`; each with alternatives + revisit triggers): all twenty ADRs in doc 22 §2 — every one carries status *recommended-for-acceptance*; "accepted" is reserved for the owner's signature [DECISION GREENFIELD §24 claim discipline]. The tech stack (doc 07 TD-01…TD-16), model role matrix (doc 14), thresholds (doc 15 QT block), and every τ/δ starting value are recommendations pending their named experiment gates.

**Assumptions in force** (tagged `[ASSUME OD-xx]`): 35 open decisions, each with a safe working assumption that lets planning and early implementation proceed without blocking — doc 22 §3.2 is the register, §4 the consolidated log. **OD-28…OD-35 are new in the 2026-08-04 pass** (from Amendment §13: nightly-run time/SLA, infographic channels, review SLA + staffing, consultant-name visibility, false-negative sampling, detector release cadence, Action Center ownership, first violation type); **OD-07 is extended** with the SRC-DATAHUB sub-feed contracts + daily batch/API assumption and **OD-10 is extended** to infographic publication authority — no other OD changed; next free: **OD-36**. The five highest-risk remain: OD-04 (outbound approval), OD-13 (OAuth client), OD-24 (annotation capacity), OD-16 (compliance sign-off), OD-07 (internal access — now the DataHub umbrella) [INFER doc 22 §4].

**Editorial consistency — resolved.** Parallel authoring initially left seven mechanical inconsistencies; all seven were **normalized in the 2026-08-03 consistency pass** and doc 23 §0.2 records the binding canon (with the historical variants) for future edits. Summary of what was normalized:
1. OD-ID remaps executed: docs 07/16/17 "OD-14/BI" → **OD-22**; doc 13 "OD-14" (artifact visibility) → **OD-23**; doc 15 "OD-19" (annotation staffing) → **OD-24** [FACT doc 22 §3.1]. (Doc 16's own OD-14 — consultant national-ID handling + evidence-role widening — and OD-19 — Groq DPA terms — keep their IDs as doc 22 §3.1 mandates.)
2. Lane-0 routing precision unified: **0.99 = calibration target on held-out paraphrases (QT-01, canonical); 0.95 = EXP-05 minimum gate floor**; doc 04 §2 now cites both.
3. DB role names unified to canon `nip_migrator` and `nip_readonly_bi` across docs 07/08/22.
4. Doc 08's census now counts **78 tables**, absorbing `serve.capability_registry`/`serve.capability_paraphrase` (DDL doc 04 §1.3, migration 0009) and `evidence.custom_collection`(+`_unit`) (DDL doc 13 §9.0, migration 0010).
5. Doc 21's P2 prerequisite now cites "P0 deliverable 12 (EPIC-06, doc 23 §5)" — the phantom epic id "P0-E12" is gone.
6. Fixture tiers pinned in both docs: Tier P (per-PR) uses only the committed 120-session synthetic corpus (doc 15 §8); the ~500-session restricted slice serves Tier N/R runs where the restricted store is reachable (doc 20 §1.9).
7. `fetch_transcript_window` first argument: canon is `session_uid` (doc 08 §0.2 identity strategy); `meeting_id` survives only inside verbatim quotations of the GREENFIELD §13 baseline text.
Additionally from the review pass: doc 07 §6.2's grant sketch was subordinated to doc 08 §1 / doc 16 §2.4 (INSERT-only on append-only planes; no direct web SELECT on `transcript.turn`); strict-schema `required` arrays were completed in doc 11 §5.1 and doc 12 §5.3; doc 18 gained SCR-13 «اللوحات» (dashboards); the single bare "TBD" token was removed from doc 05.

## 5. Decisions requiring owner approval

The register of record is doc 22 §5; doc 01 §8 renders it in Arabic for leadership. Reproduced here in full so this index is self-sufficient at the approval meeting. Classes: **G1** blocks implementation start · **G2** blocks a named phase gate · **G3** policy confirmation that can trail on the recorded safe assumption. **Umbrella precondition (2026-08-04):** before any row of this table can operate, the owner must approve this amended package revision itself — that approval is gate item 0 in §6 [DECISION owner 2026-08-03, Amendment §0.2].

| # | Decision | Artifact to sign | Class | Blocks | Owner role |
|---|---|---|---|---|---|
| 1 | Accept ADR-0001…ADR-0020 (en bloc or itemized with exceptions) | doc 22 §2 | **G1** | P0 start | Platform owner |
| 2 | Confirm the 8 VIOL categories' Arabic wording + the two coverage-gap additions (تهكم، تسويق شخصي) | doc 04 §7 | G2 | P2 taxonomy seed | Product owner |
| 3 | Answer-key protocol + signing duty (~46 items at launch, weekly cadence) | doc 15 §3 | G2 | P3 exit | Product owner |
| 4 | Outbound-data approval (pseudonymized text to Groq) — OD-04 | doc 16 §4 matrix | G2 | P2 backfill | Compliance office |
| 5 | Groq DPA / contractual data terms — OD-19 | OD-19 file | G2 | P2 backfill | Platform owner + legal |
| 6 | Second Read.ai OAuth client request — OD-13 | OD-13 request | G2 | P1 live pulls | Platform owner |
| 7 | Internal-data access mechanism + freshness SLA — OD-07 | doc 05 contracts | G2 | P1 adapters | Internal-systems owner |
| 8 | Provider candidate shortlist + trial procurement — OD-25 | OD-25 memo | G2 | P1 EXP-02 launch | Procurement |
| 9 | Provider go/no-go after EXP-02 — OD-06 | doc 06 §5.5 sheet | G2 | P6 rebase slot | Product owner |
| 10 | Annotation staffing commitment (~45 h/month) — OD-24 | OD-24 plan | G2 | P1 labelling | Product owner |
| 11 | Publication authority for packs/retractions — OD-10 | OD-10 delegation | G2 | P6 first pack | Leadership |
| 12 | PII policy set: OD-09 / OD-14 / OD-15 confirmations | doc 16 §2–3 | G3 | P7 at latest | Data steward + compliance |
| 13 | Retention windows — OD-08 | OD-08 schedule | G3 | P7 retention jobs | Data steward + compliance |
| 14 | Review-role powers confirmation — OD-11 | OD-11 matrix | G3 | P2 (assumption suffices) | Product owner |
| 15 | PDPL/NCA formal verification file — OD-16 | OD-16 dossier | G2 | P6 pilots / P7 prod | Compliance office |
| 16 | CAP-C2 route confirmation after EXP-08 — OD-05 | OD-05 memo | G3 | post-P5 | Product owner |
| 17 | Executive output format (digits/calendar) — OD-21 | OD-21 sample pack | G3 | P6 first pack | Product owner |
| 18 | BI exposure policy — OD-22 | OD-22 memo | G3 | post-pilot | Platform owner |
| 19 | Legacy retirement date | doc 21 P7-E7 plan | G3 | post-cutover | Platform owner |
| 20 | Deployment environment + IdP/secrets specifics — OD-01/OD-02 | OD-01/OD-02 memo | **G1** | P0 provisioning | IT + security directorate |

Reading the table: only rows 1 and 20 gate the *start*; rows 4–8 and 10 gate *phases* and must be **filed** at start (answers may follow); every G3 row already has a safe assumption recorded in doc 22 §3.2 under which the build proceeds. Any decision not on this table is an engineering recommendation the owner may inspect via doc 22 §2 but is not asked to sign individually.

## 6. Implementation start gate — what must be true before Opus 5 begins

**The gate changed on 2026-08-04** [DECISION owner 2026-08-03, Amendment §0.2, §12-00, §15]: **no product code is written until this amended package revision (2026-08-04, integrating the 2026-08-03 Owner Amendment) is owner-approved** — approval of the pre-amendment package does not count. And what starts after approval is not the epic backlog: **the first executable order is VS-00 only** (walking skeleton, doc 25), followed by a mandatory stop — demo, acceptance-test run, gap record, owner approval — **before VS-01** (SD-18; doc 25 owns the protocol; doc 23 enforces "no automatic next task"). EPIC-01's content executes inside VS-00 as re-scoped by doc 25.

Implementation (VS-00 per doc 25; ticket detail in doc 23) may start when ALL of the following hold [REC — restating doc 22 §5's gate with doc 21's scheduling consequences]:

0. **Approved: this amended package revision** (the PLAN_CHANGELOG §9 verification checklist run and accepted by the owner). This precedes every other item.
1. **Signed:** approval item 1 (ADRs, en bloc or with itemized exceptions recorded in doc 22) and item 20 (OD-01 environment + OD-02 IdP/secrets named). Without these there is nowhere to deploy and no login to build against.
2. **Filed with named owners and tracking IDs (answers may trail on safe assumptions):** the OD-04 outbound-approval file (with the doc 16 §4 matrix attached), the OD-19 Groq DPA request, the **OD-13 second Read.ai OAuth client request** (long lead; sharing the legacy token breaks both systems — token rotates on refresh), the **OD-07 internal-data access request** with proposed freshness SLA, and the OD-25 provider-candidate shortlist + trial-account procurement. These are P0 deliverable 9 in doc 21; filing them is engineering-independent and must not wait for code.
3. **Early-closure watchlist:** OD-13 and OD-07 must close before P1 live ingestion (else P1 exits in snapshot-only mode, flagged as a P6 risk); **OD-04 must close before the P2 full-corpus backfill** (else extraction runs only on the compliance-permitted staging subset). Doc 21 tracks each with an escalation rule (doc 22 §6.5).
4. **Experiment scheduling committed:** **EXP-01** is scheduled inside P0 against the restored snapshot (it needs no external party); **EXP-02** has its calendar dependencies secured at P0 — OD-25 candidates procurable, OD-12 audio-legality question answered (Design A vs B), OD-24 annotator commitment (~45 h/month) confirmed so DS-01 labelling starts at P1. A red or slipped EXP-02 does **not** block the build (Read.ai interim is tolerated by design); an unscheduled one silently would.
5. **The consistency pass of §4's editorial-debt list is executed** (the seven §4 items: three OD remaps + six reconciliations, plus the 2026-08-03 second pass OD-26/OD-27), so Opus 5 never has to adjudicate an ID collision mid-build.
6. **Snapshot logistics agreed:** the legacy-side snapshot (SRC-SNAP) extraction window and checksum handover are booked (doc 20 §1.2) — P0-E8 is on the critical path.

What does **not** block the start: any G3 item; the OD-06 provider decision; EXP-02 results; Groq account-tier confirmation (OD-03 — nothing depends on it); BI tooling (OD-22); **every OD-28…OD-35 (each carries a §13 safe assumption)**; **Wave B/C backlog items — they are a standing consultation list (SD-21), never a start condition**. The recorded safe assumptions carry all of them [FACT doc 22 §3.2].

### 6.1 The gate as a one-screen checklist

```text
IMPLEMENTATION START GATE — Nwafeth Intelligence
[ ] AMENDED PACKAGE REVISION (2026-08-04) APPROVED ......... Platform owner   (PLAN_CHANGELOG §9)
[ ] ADR-0001…0020 accepted (en bloc / itemized) ............ Platform owner   (doc 22 §2)
[ ] OD-01 environment + OD-02 IdP/secrets named ............ IT + security    (doc 22 §3.2)
[ ] OD-04 outbound-approval file FILED (matrix attached) ... Compliance       (doc 16 §4)
[ ] OD-19 Groq DPA request FILED ........................... Owner + legal
[ ] OD-13 second Read.ai OAuth client REQUESTED ............ Platform owner   (long lead!)
[ ] OD-07 internal-data access REQUESTED + SLA proposed .... Internal systems (now incl. SRC-DATAHUB sub-feeds)
[ ] OD-25 provider shortlist + trial accounts REQUESTED .... Procurement
[ ] OD-12 audio-legality question ASKED (EXP-02 design) .... Legal
[ ] OD-24 annotator commitment (~45 h/month) CONFIRMED ..... Product owner
[ ] EXP-01 booked inside P0 (snapshot); EXP-02 deps secured for P1
[ ] Consistency pass done (7 editorial debts of §4) ........ Package steward
[ ] Legacy snapshot extraction window BOOKED ............... Owner + DE       (doc 20 §1.2)
→ ALL CHECKED ⇒ Opus 5 executes VS-00 ONLY (doc 25; EPIC-01 content re-scoped inside it),
  then STOPS: demo + acceptance report + gap record + owner approval BEFORE VS-01.
```

## 7. Package at a glance (orientation numbers)

26 documents (2 Arabic-first, 24 English with Arabic labels) + 2 amendment-tracking files (PLAN_CHANGELOG, TRACEABILITY_OWNER_AMENDMENTS) · **26 committed capabilities** (16 committed questions A1–C8 + 10 D-family additions) **+ 12 operational capabilities CAP-OPS-01…12** · **50 backlog items** PB-001…018 / PB-101…120 / PB-201…212 in release waves Wave A/B/C · **18 invariants** I1–I18, each with a named structural enforcement (doc 23 §2) · **22 settled owner decisions** SD-01…SD-22 · **20 ADRs** recommended-for-acceptance · **35 open decisions** with safe assumptions (next free: OD-36) · **12 labelled datasets** DS-01…12 (~710 h launch labelling), extended by the detector-learning datasets (versioned, doc 15) · **17 test families** T-01…17 + per-slice acceptance suites (doc 25) · **8 hard gates** + **12 quantitative targets** · **11 experiment gates** EXP-01…11 · **8 dependency phases** P0–P7 (~175–260 engineer-weeks across phases [INFER doc 21 sums]) executed through **13 vertical slices** VS-00…VS-12 with owner stop-accept gates · **11 database schemas**, 78 tables at the pre-amendment census + the Amendment §12 entity additions (doc 08 owns the census) · **1** legacy artifact ever crossing the boundary (the checksummed snapshot) · **0** model-authored numbers, ever.

**Ready-to-hand-off statement:** with §6 satisfied — beginning with owner approval of this amended revision — the package is sufficient for Opus 5 to implement from doc 25's VS-00 onward (ticket detail in doc 23) without rediscovery — every schema, contract, threshold, prompt skeleton, and gate is specified in the documents mapped above; every unresolved item has a registered owner and a safe assumption [INFER — the GREENFIELD §25(8) closing statement, maintained by the doc 22 register].

## 8. Package-wide conventions (for any new reader)

- **IDs:** capabilities `CAP-A1…CAP-D10` (+ future promoted `CAP-E…`) and operational capabilities `CAP-OPS-01…12`; invariants `I1…I18`; settled decisions `SD-01…SD-22`; experiments `EXP-01…11`; architecture decisions `ADR-0001…0020`; open decisions `OD-01…OD-35` (next free: OD-36); method rules `MR-01…16`; datasets `DS-01…12`; tests `T-01…17`; hard gates `HG-1…8`; targets `QT-01…12`; violation seeds `VIOL-001…008`; screens `SCR-01…22` (SCR-08 = «اشتباه مخالفة», upgraded 2026-08-04); backlog items `PB-001…018 / PB-101…120 / PB-201…212`; phases `P0…P7`; vertical slices `VS-00…VS-12`; exposure levels `L0…L3`; release waves `Wave A / Wave B / Wave C` (**never** "P0/P1/P2" — those are phase names; CON-33); sources `SRC-*` including the `SRC-DATAHUB` family (-SESSION/-BENEFICIARY-EVAL/-CONSULTANT-EVAL/-OUTCOME/-DIRECTORY/-REFERENCE).
- **Exact operational names (2026-08-04):** feature flags `session_360_enabled`, `violation_review_enabled`, `nightly_consolidation_enabled`, `monthly_infographic_enabled`, `action_center_enabled`, `lane1_agent_enabled`, `lane3_analysis_enabled` · nightly run states `queued → running → partial → succeeded → failed → cancelled → superseded` · ops metrics `review_backlog_age`, `data_completeness`, `source_join_rate`, `action_completion_rate`, `detector_acceptance_rate`, `suspected_cases_open`, `nightly_run_success` (registry: doc 11).
- **Claim tags:** `[FACT src]` source-derived · `[DECISION]` owner decision (amendment decisions cite `[DECISION owner 2026-08-03, Amendment §x]`) · `[REC]` recommendation with alternatives + revisit trigger · `[INFER]` inference · `[ASSUME OD-xx]` assumption bound to a registered open decision. Bare "TBD" does not appear in this package.
- **Schemas:** `ingest, core, transcript, findings, tax, serve, jobs, packs, evidence, ops, legacy_snapshot` (CORE-BRIEF §6). Reason codes: the closed 16-value enum of CORE-BRIEF §4. Lanes: 0 deterministic · 1 bounded agent · 2 clarification/honest boundary · 3 deep analysis.
- **Language:** «قطاع» is always disambiguated into `service_category` / `government_entity` / `business_sector`; periods are end-exclusive; rates per 100 sessions; n≥30 with Wilson intervals; quotes always carry the full 8-field provenance tuple; suspected violations are never rendered as confirmed — leadership numbers use approved-only, with queue size as its own separate figure (SD-19/SD-20).

---

*End of document 00. Leadership: continue to doc 01, then docs 24/25. Engineers: continue to doc 25, then doc 23. This revision: PLAN_CHANGELOG (what changed) · TRACEABILITY_OWNER_AMENDMENTS (requirement coverage).*

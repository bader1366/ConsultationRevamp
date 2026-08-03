# PLAN_CHANGELOG — Owner Amendment Integration Record (2026-08-04 pass)
**Platform:** Nwafeth Intelligence — منصة نوافث لذكاء الجلسات الاستشارية (Monsha'at Advisory Session Intelligence Platform)
**Status:** Amendment-pass record for owner review · **Date:** 2026-08-04 · **Author:** Planning package (Fable 5)
**Depends on:** OWNER-AMENDMENT (2026-08-03) in full; docs 00–25 · **Feeds:** 00 (start gate), TRACEABILITY_OWNER_AMENDMENTS.md, the owner's approval of this revision
**Sources used:** OWNER-AMENDMENT §0–§15 (esp. §0.4, §12, §13, §14); doc 02 §1.11/§2 Tier 3/§4/§5.1; doc 22 §3.2

This is the standalone file the Owner Amendment §0.4 mandates: **what changed, what stayed, which conflicts were resolved, and which decisions remain open** in the 2026-08-04 integration pass. The amendment itself (dated 2026-08-03; the Sol 5.6 review accepted by the owner as a binding change request) is registered as a prescriptive source at owner-requirements rank in doc 00 §3 and doc 02 §1.11. Requirement-level coverage lives in `TRACEABILITY_OWNER_AMENDMENTS.md`; this file is the per-document narrative.

**How this file relates to its siblings and registers:**

- `TRACEABILITY_OWNER_AMENDMENTS.md` — requirement → document/feature/slice/acceptance/status matrix (the §14.3 artifact); read it column-first when auditing coverage.
- doc 02 §4 — register of record for SD-18…SD-22; doc 02 §2 Tier 3 — register of record for CON-30…CON-34; doc 22 §3.2 — register of record for OD-28…OD-35. This file summarizes; those registers govern.
- doc 24 / doc 25 — the two new deliverables this pass introduces (backlog; delivery-and-acceptance protocol). Their content mandate is §4 below.
- doc 00 §6 — the amended start gate this file's approval feeds.

Reading order for the approval meeting: §1 (what the pass does) → §2 (decisions) → §5 (conflicts) → §6 (still open) → §3 (per-document detail, on demand) → §9 (verification steps).

---

## 1. The pass at a glance

- **What the amendment does:** re-orients delivery from horizontal architecture-first phases to an operational product built as vertical slices **VS-00…VS-12** with a mandatory stop-accept after every slice; makes **DataHub a first-class source family**; adds the **Nightly Consolidation Run «التشغيل الليلي الموحد»**, the **«اشتباه مخالفة»** review product, the **governed detector-learning loop**, the **monthly leadership infographic «نبض خدمة الاستشارات والإرشاد — ملخص الشهر»**, the **Action Center**, and **«صباحيات الخدمة»**; and installs a small-sequential-ticket execution protocol [DECISION owner 2026-08-03, Amendment §0–§10].
- **What it does not do:** it weakens **no** invariant or gate. The amendment's §0.5 no-weakening list maps 1:1 onto I2, I3, I4, I5, I7, I9, I6, the review-first rule, and I15; SD-17 (CPU-only/Groq-only) is reconfirmed; SD-01…SD-17 all stand.
- **New IDs minted in this pass (complete list — nothing else was minted):**
  - Settled decisions **SD-18…SD-22** (§2 below; register: doc 02 §4).
  - Open decisions **OD-28…OD-35** (§6 below; register: doc 22 §3.2; **next free: OD-36**). OD-07 and OD-10 are *extended*, not re-minted.
  - Experiment **EXP-11** — false-negative estimation design (stratified unflagged sampling: sizes, strata, cadence) → docs 15/21/25.
  - Operational capabilities **CAP-OPS-01…CAP-OPS-12** (doc 04, same registry discipline as CAP-A/B/C/D).
  - Backlog items **PB-001…PB-018** (Wave A), **PB-101…PB-120** (Wave B), **PB-201…PB-212** (Wave C) — release waves are **Wave A/B/C**, never "P0/P1/P2" (CON-33).
  - Vertical slices **VS-00…VS-12**; exposure levels **L0/L1/L2/L3** (Amendment §9.2/§9.3, verbatim).
  - Screens **SCR-14…SCR-22**; the existing **SCR-08** is upgraded to «اشتباه مخالفة» (no new SCR minted for it).
  - Source contracts: **SRC-DATAHUB** family with sub-contracts SRC-DATAHUB-SESSION / -BENEFICIARY-EVAL / -CONSULTANT-EVAL / -OUTCOME / -DIRECTORY / -REFERENCE (doc 05; supersedes the SRC-INT-centric framing — CON-32).
  - Ops metrics (names exact, doc 11 registry): `review_backlog_age`, `data_completeness`, `source_join_rate`, `action_completion_rate`, `detector_acceptance_rate`, `suspected_cases_open`, `nightly_run_success`.
  - Feature flags (names exact): `session_360_enabled`, `violation_review_enabled`, `nightly_consolidation_enabled`, `monthly_infographic_enabled`, `action_center_enabled`, `lane1_agent_enabled`, `lane3_analysis_enabled`.
  - Nightly run states (exact): `queued → running → partial → succeeded → failed → cancelled → superseded`.
- **Start-gate change:** no product code until THIS amended revision is owner-approved; the first executable order after approval is **VS-00 only**, with a stop-accept report before VS-01 (doc 00 §6; doc 25 owns the protocol).

## 2. Owner decisions recorded (2026-08-03) — SD-18…SD-22

Doc 02 §4 is the register of record; summary here for the approval meeting:

| ID | Decision (one line) | Citation |
|---|---|---|
| SD-18 | Incremental vertical-slice delivery VS-00…VS-12; mandatory stop after every slice (demo → acceptance tests → gaps recorded → owner approval); L1 owner previews on synthetic/staging from the earliest slices; production gates (EXP-09/10 etc.) guard L2/L3 only, never L1; no fully-parallel multi-epic development by one agent | [DECISION owner 2026-08-03, Amendment §0.6–§0.9, §9, §15] |
| SD-19 | Monthly leadership infographic «نبض خدمة الاستشارات والإرشاد — ملخص الشهر» is a standalone early product (CAP-OPS-08, PB-014, VS-05): one page; web+PDF+PNG byte-identical payloads; draft→data_review→content_review→approved→published→superseded/retracted; drawn programmatically from structured data ONLY (no LLM numbers, no image-generation models; optional LLM phrasing passes the R6 gate); approved-violations only in leadership numbers; no consultant names in the general edition | [DECISION owner 2026-08-03, Amendment §7] |
| SD-20 | Violation-suspicion queue «اشتباه مخالفة» is an early product (VS-02): suspected ≠ approved everywhere; `violation_finding`/`review_case`/`review_event` separation, append-only; first vertical type VIOL-008 then one type at a time; six closed decisions (مخالفة صحيحة / ليست مخالفة / إعادة تصنيف / تحتاج مراجعة ثانية / أدلة غير كافية / إحالة لمالك السياسة), each with closed reason code, reviewer identity, stamps, no overwrite | [DECISION owner 2026-08-03, Amendment §5] |
| SD-21 | Backlog governance: PB-014 and the violation-suspicion cluster PB-006…PB-010 are committed to the earliest releases; EVERYTHING else (all Wave B/C items and any Wave A item not yet bound to an approved slice) requires explicit owner consultation and approval before entering any implementation slice — the backlog is a standing consultation list, not a work order | [DECISION owner 2026-08-03, Amendment §8 + owner rule] |
| SD-22 | Governed detector learning, never online self-learning: review labels + missed-case reports + weekly stratified random negatives → versioned immutable datasets → candidate → offline eval per category (precision AND estimated recall) → shadow → human adjudication → documented promotion → canary → full deploy with version stamp → monitoring → instant rollback; no reviewer click ever changes production behaviour directly; consultant names/ratings are never detector features | [DECISION owner 2026-08-03, Amendment §6] |

## 3. Per-document change log (00…23)

Each entry states what this pass changes (the Amendment §12 mandate + the pre-assigned IDs) and what stays. "Unchanged" means the document's pre-amendment core content is retained verbatim or with only additive/marked-superseded edits — nothing is silently deleted, no invariant weakened.

### 3.0 Baseline and change scope

Pre-pass line counts are the 2026-08-03 consistency-pass state (the doc 00 §2 map before this revision); they date-stamp what this changelog describes. Every document 00–23 is listed in Amendment §12 and therefore carries a "changed" entry; no document is deleted or renumbered.

| Doc | Pre-pass lines | This pass | Doc | Pre-pass lines | This pass |
|---|---|---|---|---|---|
| 00 | 211 | changed (gate, sources, map) | 12 | 809 | changed (sequencing, agent bounds) |
| 01 | 247 | changed (loops, products) | 13 | 879 | changed (later-stage, board, TTL) |
| 02 | 456 | changed (SD/CON/source registers) | 14 | 623 | changed (detector lifecycle) |
| 03 | 382 | changed (personas) | 15 | 748 | changed (datasets, recall, VS suites, EXP-11) |
| 04 | 1,097 | changed (CAP-OPS, metrics) | 16 | 657 | changed (RBAC, masking, policies) |
| 05 | 645 | changed (SRC-DATAHUB family) | 17 | 722 | changed (15 endpoint groups) |
| 06 | 570 | changed (nightly/rebase coupling) | 18 | 668 | changed (SCR-14…22, SCR-08 upgrade) |
| 07 | 809 | changed (8 components, centrality) | 19 | 620 | changed (SLOs, digest, drills) |
| 08 | 1,453 | changed (run/review/learning/action entities) | 20 | 575 | changed (staging ladder, preview rule) |
| 09 | 627 | changed (nightly chapter) | 21 | 761 | changed (VS overlay, waves) |
| 10 | 802 | changed (4 method additions) | 22 | 616 | changed (OD-28…35, extensions) |
| 11 | 575 | changed (MVP five, ops metrics) | 23 | 749 | changed (§10 protocol re-issue) |

New in this pass: `24-product-backlog.md`, `25-incremental-delivery-and-acceptance.md`, `TRACEABILITY_OWNER_AMENDMENTS.md`, `PLAN_CHANGELOG.md` (§4). Post-pass line counts are recorded in doc 00 §2 as the map is refreshed.

### 00 — `00-INDEX.md`
- **changed:** Owner Amendment registered as a prescriptive source at owner-requirements rank (above every package recommendation); docs 24/25 + PLAN_CHANGELOG + TRACEABILITY added to the map, dependency sketch, and reading orders (owner/leadership read 24+25 early; implementer reads 25 before 23); **start gate changed** — no product code until this amended revision is owner-approved, first executable order = VS-00 only with stop-accept before VS-01; settled-decisions paragraph extended with SD-18…SD-22; OD-28…OD-35 noted, next free OD-36; convention/count lists refreshed (CAP-OPS, VS, PB waves, SCR-01…22, EXP-01…11, flags, run states).
- **unchanged:** core content — audiences and their outcome checklists, the two precedence ladders' descriptive side, approval-table rows 1–20, the §6 filing requirements, the editorial-consistency record.

### 01 — `01-executive-summary-ar.md`
- **changed:** the six operational loops (Amendment §2) added in executive language; first visible value restated as an interactive operational product, not an experiments report; the monthly infographic added as a standalone leadership product; «صباحيات الخدمة» and Action Center added as operational value; Owner Preview (L1) vs Pilot (L2) vs Production (L3) distinguished for leadership.
- **unchanged:** core content — the rebuild rationale, what is replaced and not, value per user group, executive architecture narrative, leadership decision list (extended, not rewritten).

### 02 — `02-source-map-and-contradictions.md`
- **changed:** OWNER-AMENDMENT added to the source map as an owner-decision source (§1.11, prescriptive rank: owner-requirements level); SD-18…SD-22 appended to the settled-decisions register; contradiction log gains Tier 3 with CON-30…CON-34 (the five resolved plan-vs-amendment conflicts, §5 below); note added that DataHub becomes a first-class source family superseding the SRC-INT-centric framing (doc 05 owns it); §5.1 records the OD-28…OD-35 extraction and the OD-07/OD-10 extensions.
- **unchanged:** core content — all legacy evidence tables, CON-01…CON-29, OBS-01…OBS-25, SD-01…SD-17, planning baselines §7.

### 03 — `03-product-scope-and-personas-ar.md`
- **changed:** personas added/clarified with daily journeys (Amendment §12): قائد/نائب يتلقى الإنفوجرافيك · مدير خدمة الاستشارات والإرشاد · مراجع اشتباه المخالفات · قائد المراجعين/Adjudicator · Data Steward · Operations Engineer · AI/Model Owner · مدير مستشارين (Coaching + Action Center) — each with top tasks, alerts, permissions, and decisions.
- **unchanged:** core content — hard scope boundaries, the original five personas, journeys ر1–ر12, the committed-capability product table.

### 04 — `04-capability-catalogue.md`
- **changed:** new operational-capability section CAP-OPS-01…CAP-OPS-12 (nightly run, data ops center, Session 360, session search, «اشتباه مخالفة» queue+case, missed-violation report, detector learning, monthly infographic, action center, morning digest, service recovery, ops dashboard) under the same registry discipline as CAP-A/B/C/D; suspected-vs-approved counts separated in every violation-reporting capability; ops-metric references added (`review_backlog_age`, `data_completeness`, `source_join_rate`, `action_completion_rate` + the doc 11 set).
- **unchanged:** core content — CAP-A1…CAP-D10 entries, registry schema, τ/δ semantics, VIOL vocabulary reconciliation, lifecycle and promotion path.

### 05 — `05-data-source-contracts.md`
- **changed:** `SRC-DATAHUB — Monsha'at Internal Advisory DataHub` added as a first-class source family with sub-contracts SRC-DATAHUB-SESSION/-BENEFICIARY-EVAL/-CONSULTANT-EVAL/-OUTCOME/-DIRECTORY/-REFERENCE; the Field Authority Matrix (Amendment §3.2) added; late-arriving updates, CDC/batch assumptions, watermarks, schema versions, matching keys and SLAs specified; **SRC-INT/SRC-EVAL/SRC-OUT/SRC-DIR/SRC-REF restructured as views of the family with the supersession stated explicitly** (CON-32); Excel upload demoted to temporary fallback only.
- **unchanged:** core content — SRC-READAI, SRC-TSP, SRC-SNAP contracts, DLQ state machine, reconciliation/join-rate KPI model (now applied per sub-contract).

### 06 — `06-transcript-provider-strategy.md`
- **changed:** provider pull tied into the Nightly Consolidation Run; rebase impact on review cases, learning datasets, and infographics specified; no silent reopening of old cases — a stale/needs-review transition is created instead.
- **unchanged:** core content — `TranscriptSource` contract, ten-step rebase, EXP-02 benchmark, quality tiers, glossary, STT contingency.

### 07 — `07-target-architecture.md`
- **changed:** explicit components added — Nightly Orchestrator, Source Watermark service, Data Reconciliation service, Review Case service, Learning Dataset & Model Promotion service, Infographic/Publication service, Notification service, Action Center service; **agentic chat explicitly demoted from the centre of the architecture — the operational product is the centre and the agent is an interface above it** (CON-34).
- **unchanged:** core content — C4 views, layer specs, TD-01…TD-16, invariant-enforcement map, module layout, failure-domain matrix.

### 08 — `08-data-model-and-erd.md`
- **changed:** entities added or verified-present per Amendment §12: `pipeline_run`, `pipeline_step_run`, `source_watermark`, `dead_letter_item`, `data_quality_issue`, `reconciliation_case`, `violation_finding`, `review_case`, `review_event`, `review_assignment`, `missed_violation_report`, `label_dataset`(+`_item`), `detector_candidate`, `detector_release`, `shadow_result`, `monthly_infographic`, `infographic_section`, `publication_event` (or pack-artifact linkage), `service_improvement_action`, `action_event`, `action_metric_baseline`, `notification_subscription`, `notification_delivery` — use-case/versioning/no-overwrite support is the requirement, exact names may map onto existing tables.
- **unchanged:** core content — the eleven schemas, identity strategy, versioning rules L1–L10, PII boundary map, partitioning doctrine.

### 09 — `09-ingestion-reconciliation-and-pipelines.md`
- **changed:** full chapter for «التشغيل الليلي الموحد» per Amendment §4 — the 16 mandatory steps, run states `queued → running → partial → succeeded → failed → cancelled → superseded`, run-record fields, failure/resume rules (idempotency, per-adapter watermarks, DLQ, `rerun failed items` / `reprocess selected sessions`, no local-file resumability); hourly incremental vs nightly authoritative distinction; Morning Digest emission and month-close trigger; user-facing progress state; partial-source-failure, late-arriving and replay tests.
- **unchanged:** core content — DAG doctrine, per-session state machine, M1–M3 identity ladder, DQ gates, playbooks (extended, not replaced).

### 10 — `10-analytical-methodology.md`
- **changed:** suspected-vs-approved separation rules for every violation figure; false-negative estimation method via weekly stratified random samples of unflagged sessions (feeds EXP-11); action-impact tracking method without unproven causality claims; infographic KPI selection-and-suppression method.
- **unchanged:** core content — MR-01…MR-16, per-capability method sheets, R-P2 clustering, coverage-block definition.

### 11 — `11-semantic-layer-and-curated-capabilities.md`
- **changed:** the VS-07 MVP set of five capabilities defined (sessions/status by period+programme; beneficiary rating distribution; clear-steps rate; suspected/approved violation counts + review state; top recurring challenges/questions from the first approved semantic artifact); completing all 26 capabilities is no longer a precondition for the first product; ops/review/DQ/action metrics registered with exact names `review_backlog_age`, `data_completeness`, `source_join_rate`, `action_completion_rate`, `detector_acceptance_rate`, `suspected_cases_open`, `nightly_run_success`.
- **unchanged:** core content — registry architecture, spec→SQL compiler, period/coverage enforcement, typed envelope, suppression mechanics.

### 12 — `12-agentic-serving-architecture.md`
- **changed:** agentic chat explicitly sequenced after the operational core is stable (VS-09); the agent can never create or approve a violation, and never publishes an infographic or an action outside the authorized workflow; safe read-only tools added for Session 360, review state, and actions, permission-scoped.
- **unchanged:** core content — lanes 0–3, deterministic pre-processing, toolbelt and schemas, budgets, verifier suite (R6/R7), injection isolation, R13 log.

### 13 — `13-on-demand-analysis-and-custom-rag.md`
- **changed:** kept as a later stage — after the nightly run, review loop, and infographic are proven (VS-10+); Promotion Board added for recurring questions; TTL/expiry/owner required on every custom-RAG artifact.
- **unchanged:** core content — trigger tree, frozen manifests, map→verify→reduce, job state machine, caching/promotion, RAG lifecycle mechanics.

### 14 — `14-groq-model-and-inference-strategy.md`
- **changed:** detector release lifecycle from Amendment §6.5 added (dataset build → candidate → offline eval → regression → shadow → adjudication → promotion decision → canary → full deployment → monitoring → rollback); extraction model, violation detector, and composer separated as roles; shadow deployment/canary/rollback operations; consultant name/rating never used as a detection feature.
- **unchanged:** core content — verified catalogue facts, task-role matrix, EXP-03, structured-output policy, registries, deprecation ops.

### 15 — `15-evaluation-answer-oracle-and-golden-suite.md`
- **changed:** missed-case reporting and unflagged random-sample datasets added to the labelled-data plan; per-category **recall estimates** (not precision only); reviewer disagreement/adjudication set; **acceptance tests per vertical slice VS-00…VS-12**; web/PDF/PNG payload-identity test for the infographic; EXP-11 (false-negative estimation design) specified.
- **unchanged:** core content — DS-01…DS-12, oracle protocol, T-01…T-17, HG-1…8, QT-01…12, no-weakening ratchet, fixture corpus.

### 16 — `16-security-privacy-and-compliance.md`
- **changed:** permissions defined for the infographic, Action Center and Consultant 360; consultant names excluded from the general infographic edition by default; the violation queue may not be used for open-ended surveillance without permission and reason; notification, export, and expiring-link policies added.
- **unchanged:** core content — threat model, RBAC matrix (extended), P0–P3 data classes, outbound matrix, EXP-10 catalogue, audit events, playbooks.

### 17 — `17-api-contracts.md`
- **changed:** endpoints added/reviewed per Amendment §12: `GET /pipeline-runs`, `GET /pipeline-runs/{id}`, `POST /pipeline-runs/{id}/retry`, `GET /data-quality/issues`, `GET /sessions/{id}/360`, `GET /violations/suspected`, `GET /violations/cases/{id}`, `POST /violations/cases/{id}/review-events`, `POST /sessions/{id}/missed-violation`, `GET /detector-releases`, `GET /infographics/monthly/{period}`, `POST /infographics/{id}/approve`, `POST /infographics/{id}/publish`, `GET/POST/PATCH /actions`, `GET/POST /subscriptions` — all with RBAC, pagination, idempotency, versioning, Problem Details.
- **unchanged:** core content — RFC 7807 error model, existing endpoint inventory, chat/turn envelope, job API, contract tests.

### 18 — `18-ux-admin-and-review-workflows-ar.md`
- **changed:** new screens in build order — SCR-14 مركز تشغيل البيانات · SCR-15 الجلسة 360 · SCR-16 بحث الجلسات · SCR-17 الإبلاغ عن اشتباه فائت · SCR-18 لوحة جودة الكاشف والتعلم · SCR-19 مركز الإجراءات · SCR-20 الإنفوجرافيك الشهري (مسودة/اعتماد/نشر) · SCR-21 صباحيات الخدمة · SCR-22 قائمة التعافي الخدمي; **SCR-08 upgraded** per Amendment §5 (display name «اشتباه مخالفة», the fixed disclaimer, finding/case/event separation, the 6 decisions, reopened-by-change filter, quote context ±5 turns); the chat surface moved later in the build order — التشغيل والمراجعة قبل المحادثة.
- **unchanged:** core content — shared foundations (RTL، الأرقام، قاموس التحوط), SCR-01…07/09…13 specifications, role×screen matrix (extended).

### 19 — `19-observability-operations-and-dr.md`
- **changed:** SLOs and alerts added for: nightly-run success by deadline (`nightly_run_success`), per-source freshness, `source_join_rate`, run partial/failure, DQ backlog, `review_backlog_age`, detector acceptance/drift (`detector_acceptance_rate`), infographic draft/publish deadlines, action overdue rate (`action_completion_rate`), notification delivery failures; Morning Digest, run manifest, and restore drills covering the new entities.
- **unchanged:** core content — stack, trace propagation, R13 records, dashboards, runbooks (extended), capacity model, DR targets.

### 20 — `20-migration-parallel-run-and-cutover.md`
- **changed:** staged cohort plan made explicit (20 sessions → 100 → one month → full corpus); **parallel run is not a precondition for Owner Preview (L1)**; suspected and approved violations compared separately in parity runs; shadow detector migration added.
- **unchanged:** core content — snapshot mechanics, restore/mapping, replay parity harness, cutover gates G0…G5 (as L2/L3 gates), rollback triggers.

### 21 — `21-delivery-roadmap-and-work-breakdown.md`
- **changed:** **the largest edit** — the execution plan becomes the vertical-slice overlay VS-00…VS-12 (doc 25 owns the protocol); phases P0–P7 and EXP gates REMAIN as the dependency/gate framework, with each gate bound to the feature it protects (not to visibility of everything); completing all 26 capabilities is no longer a precondition for the first testable product; every slice carries demo, acceptance, and stop gate; release waves recorded as Wave A/B/C; coding tickets reduced to ≤4–8h while epics remain planning units; EXP-11 added to the experiment gates; the "nothing user-facing before EXP-09/10" sequencing principle rewritten to scope those gates to L2/L3 exposure only (CON-30).
- **unchanged:** core content — phase objectives/exit criteria as gate content, EXP-01…EXP-10 definitions, team model, RACI, risk register.

### 22 — `22-architecture-decisions-and-open-decisions.md`
- **changed:** OD-28…OD-35 registered with owners, safe assumptions and impact (from Amendment §13); **OD-07 extended** as the umbrella for the DataHub access mechanism (SRC-DATAHUB sub-feed contracts + daily batch/API assumption — no new OD minted); **OD-10 extended** to cover infographic publication authority; SD-18…SD-22 cross-referenced; next free OD becomes **OD-36**.
- **unchanged:** core content — ADR-0001…ADR-0020, OD-01…OD-27, assumption log A-01…A-29, collision resolutions, maintenance rules.

### 23 — `23-opus-5-implementation-handoff.md`
- **changed:** re-issued as a **small-sequential-ticket execution protocol** carrying Amendment §10 verbatim-or-equivalent: ticket size ≤4–8h with split rules; the pre-ticket plan file (11 items); ticket Definition of Done (code, migration, unit+integration+UI/manual tests, fixtures, **authorization test**, observability, docs, `docs/STATE.md`, `PRODUCT_BACKLOG.md`, `KNOWN_GAPS.md`, run/test evidence, explicit not-done list); the ten execution prohibitions; the seven feature flags (exact names, §1); the staged data strategy (fixture → anonymized snapshot → 20–100 staging cohort → one month → full backfill); per-cycle outputs with evidence — no bare «تم الإنجاز»; **no multi-epic parallel execution; first executable order after approval = VS-00 only; acceptance report before VS-01; "do not start the next task automatically"**; EPIC-01…20 remain the ticket inventory but are re-sequenced and split under the VS order.
- **unchanged:** core content — mission, I1–I18 enforcement map, repository structure, migration/API/test/prompt build orders as reference material, coding conventions, MUST-NOT list (extended, not relaxed).

## 4. New files in this re-issue

| File | Role | Mandate |
|---|---|---|
| `24-product-backlog.md` | The product backlog: Amendment §8 + §11 as full story cards (Feature ID, primary user, problem/value, data+capabilities, sensitivity, dependencies, testable acceptance, release wave, feature flag); ranking by Value/Risk/Dependency; owner + wave + acceptance per feature; **the single statement of the amendment-priority→wave mapping (P0→Wave A, P1→Wave B, P2→Wave C)**; the SD-21 standing-consultation rule | Amendment §0.10, §8, §11, §12 |
| `25-incremental-delivery-and-acceptance.md` | The delivery/acceptance protocol: Amendment §9 + §10; VS-00…VS-12 with demo script + acceptance checklist + stop gate each; exposure levels L0–L3; the §9.4 stop rule; the agent state-file template; the first five VS-00 coding tickets as specifications only (Amendment §14.5-8) | Amendment §0.10, §9, §10, §14.5 |
| `TRACEABILITY_OWNER_AMENDMENTS.md` | Requirement-by-requirement matrix over Amendment §§2–11 and §12: each requirement → owning document/section → PB/CAP-OPS IDs → VS → acceptance reference → status | Amendment §14.3 |
| `PLAN_CHANGELOG.md` (this file) | What changed / what stayed / conflicts resolved / still-open decisions, per document | Amendment §0.4, §14.3 |

## 5. Resolved conflicts (registered as CON-30…CON-34 in doc 02 §2 Tier 3)

1. **CON-30 — "nothing user-visible before EXP-09/EXP-10" vs owner previews after every slice.** Resolved: exposure levels L0–L3 (Amendment §9.2). **L1 owner previews on synthetic or limited-staging data are allowed from the earliest slices**; EXP-09/EXP-10 and the other production gates guard **L2 (pilot) and L3 (production) exposure only, never L1**; L1 success never authorizes L2/L3, and no operational decisions are taken from L1. No gate is weakened — its scope is stated correctly [DECISION owner 2026-08-03, Amendment §0.9].
2. **CON-31 — horizontal phase delivery vs the incremental mandate.** Resolved: **phases P0–P7 + EXP gates stay as the dependency/gate framework** (what must be true); **VS-00…VS-12 become the execution overlay** (what gets built, shown, and accepted, in order). Each VS maps to a subset of phase deliverables and may pull forward the minimal slice of a later phase's machinery, provided its L2/L3 exposure still waits for the owning phase's gates. EPIC-01…20 remain the ticket inventory, re-sequenced and split ≤4–8h under the VS order; first executable order after approval = VS-00 only [DECISION owner 2026-08-03, Amendment §9, §12-21].
3. **CON-32 — SRC-INT/SRC-EVAL/SRC-OUT/SRC-DIR/SRC-REF vs SRC-DATAHUB.** Resolved: **restructured as the SRC-DATAHUB family** (SESSION / BENEFICIARY-EVAL / CONSULTANT-EVAL / OUTCOME / DIRECTORY / REFERENCE); the five prior contracts become views of the family with the supersession stated explicitly in doc 05 (no two competing framings); **Excel upload = temporary fallback only**; access mechanism stays under the extended OD-07 [DECISION owner 2026-08-03, Amendment §3].
4. **CON-33 — backlog priority names P0/P1/P2 colliding with phase names P0…P7.** Resolved: release waves are named **Wave A / Wave B / Wave C**; P0/P1/P2 remain doc 21 phase names exclusively; the mapping is stated once, in doc 24, and never re-derived.
5. **CON-34 — agentic-chat centrality vs the operational product.** Resolved: **the operational product (nightly run, Session 360, «اشتباه مخالفة», infographic, actions) is the centre of the architecture; the agent is an interface above it**, delivered in later slices (VS-09/VS-10) over capabilities that have proven out; doc 18's build order now leads with operations and review, chat later; the agent gets permission-scoped read-only tools and no create/approve/publish powers [DECISION owner 2026-08-03, Amendment §7-arch note, §12].

## 6. Still open after this pass

**New open decisions (register: doc 22 §3.2; all have safe assumptions — none blocks planning or VS-00):**

- **OD-28** nightly-run time + completion SLA — assume 02:00 Asia/Riyadh, configurable; hourly Read.ai incremental allowed, nightly remains authoritative.
- **OD-29** infographic publication channels — assume Portal + approved Email; Teams/other channels later.
- **OD-30** review SLA + reviewer staffing/assignment — assume 7d normal / 2d high-priority; assignment by the review lead.
- **OD-31** consultant-name visibility scope in infographic/coaching surfaces — assume masked in the general edition; visible only to direct-manager RBAC per OD-09/OD-14 rules.
- **OD-32** false-negative sample size/cadence — assume weekly stratified sample of unflagged sessions; size set by experiment EXP-11.
- **OD-33** detector release cadence — assume monthly or on gate-pass, whichever is LESS frequent.
- **OD-34** Action Center ownership + closure authority — assume the service owner owns actions; leadership sees aggregate status/impact.
- **OD-35** first violation vertical type — assume VIOL-008 «التواصل خارج الإطار الرسمي»; owner may swap before the VS-02 build.
- **OD-07 (extended, not re-minted)** — umbrella for the DataHub access mechanism, now covering the SRC-DATAHUB sub-feed contracts + the daily batch/API assumption and freshness SLA.
- *(OD-10 extended to infographic publication authority — the Amendment §12-22 "Publication authority" bullet maps to it; no new OD.)*

**Working assumptions carried over from Amendment §13** (planning assumptions, not final production decisions):

| الموضوع | الافتراض العامل | Bound to |
|---|---|---|
| وقت التشغيل الليلي | 02:00 Asia/Riyadh يوميًا، Configurable | [ASSUME OD-28] |
| Read.ai incremental | كل ساعة إن سمح المصدر؛ التشغيل الليلي يبقى المرجعي | [ASSUME OD-28] |
| DataHub | Batch/API مؤسسي يومي مع عقود فرعية مستقلة؛ لا اعتماد دائم على Excel | [ASSUME OD-07] |
| بوابة اكتمال الشهر | 98% من الجلسات في حالات Terminal مع إظهار الاستبعادات | §13 assumption (configurable gate; doc 10 owns) |
| Draft الإنفوجرافيك | صباح اليوم الثاني من الشهر الجديد بعد اجتياز البوابة | §13 assumption (docs 09/10) |
| صيغ الإنفوجرافيك | Web + PDF + PNG | [DECISION owner 2026-08-03, Amendment §7.2] |
| قنوات النشر | Portal + Email معتمد؛ القنوات الأخرى Open Decision | [ASSUME OD-29] |
| Review SLA | 7 أيام للحالة العادية، 2 يوم للحالة عالية الأولوية | [ASSUME OD-30] |
| أول نوع مخالفة | VIOL-008 التواصل خارج الإطار الرسمي | [ASSUME OD-35] |
| قرارات المراجعة | لا Online learning مباشر؛ تدخل Dataset versioned | [DECISION SD-22] |
| Dataset refresh | أسبوعي أو نصف شهري بحسب حجم القرارات | §13 assumption (doc 14) |
| Detector release | شهري أو عند اجتياز Gate، أيهما أقل تكرارًا | [ASSUME OD-33] |
| false-negative sampling | عينة أسبوعية طبقية من الجلسات غير المرفوعة؛ الحجم يحدد في EXP-11 | [ASSUME OD-32] |
| المخالفات في تقارير القيادة | Approved فقط؛ suspected يظهر كحجم Queue منفصل | [DECISION SD-19/SD-20] |
| أسماء المستشارين | محجوبة عن الإنفوجرافيك العام؛ تظهر فقط للصلاحيات الإدارية المحددة | [ASSUME OD-31] |
| Action Center | Service Owner يملك الإجراء، والقيادة ترى الحالة والأثر التجميعي | [ASSUME OD-34] |
| أول Owner Preview | بعد VS-01، ولا ينتظر اكتمال كل تجارب Pilot/Production | [DECISION SD-18] |

**Also open (pre-existing, unchanged by this pass):** OD-01…OD-27 as registered in doc 22 §3.2 — the amendment closes none of them; the five highest-risk (OD-04, OD-13, OD-24, OD-16, OD-07) remain on the doc 00 §6 watchlist.

## 7. What did NOT change (the no-weakening statement)

- **Invariants I1–I18** and every deterministic gate (R6 numeric provenance, R7 quote, EXP-06/09/10, HG-1…8, QT-01…12): untouched. The amendment's §0.5 list maps 1:1 onto I2, I3, I4, I5, I7, I9, I6, the review-first rule, and I15 [DECISION owner 2026-08-03, Amendment §0.5, §14.4].
- **SD-01…SD-17** all stand, including SD-17 (CPU-only environment, all generative inference on Groq — reconfirmed 2026-08-03).
- **All existing IDs are stable**: CAP-A1…CAP-D10, VIOL-001…008, EXP-01…10, ADR-0001…0020, OD-01…OD-27, DS/T/HG/QT sets, SCR-01…13, P0…P7, EPIC-01…20. New content is additive or explicitly marked as superseded with a pointer — nothing silently deleted.
- **Suspected violations never render as confirmed anywhere**; leadership numbers use approved-only; queue size may appear as its own separate figure (SD-19/SD-20).
- The evidence base (doc 02 §1 legacy tables, §7 baselines) and the descriptive precedence ladder are untouched — the amendment is a prescriptive source only.

## 8. Supersessions register (old framing → new framing, with pointers — never silent deletion)

Per the amendment-integration rule, replaced framings are struck with a pointer, not removed. The authoritative list of statements this pass marks SUPERSEDED:

1. **doc 05** — SRC-INT / SRC-EVAL / SRC-OUT / SRC-DIR / SRC-REF as five standalone general contracts → **superseded by the SRC-DATAHUB family**; the five IDs survive as named views/sub-contracts of the family (CON-32). Excel upload as an ingestion mechanism → **temporary fallback only**, never the target.
2. **doc 21 §1(4)** — "Nothing user-facing before EXP-09/EXP-10" → **superseded by the exposure-scoped rule**: nothing *L2/L3*-visible before its gates; L1 owner previews on synthetic/limited-staging data are mandatory after every slice (CON-30).
3. **doc 23** — "Opus 5 begins at EPIC-01" as the start instruction → **superseded by "the first executable order is VS-00 only (doc 25)"**; EPIC-01…20 survive as the ticket inventory, re-sequenced and split under the VS order (CON-31).
4. **doc 18** — the chat-first screen order → **superseded by the operations-first build order** (SCR-14 → SCR-15 → SCR-08 → … → chat above approved capabilities) (CON-34).
5. **Amendment §8.1's own "P0/P1/P2" release-priority names** → **superseded by Wave A / Wave B / Wave C** in every package document; P0…P7 remain phase names exclusively; the mapping is stated once in doc 24 (CON-33).
6. **doc 00 §6** — the pre-amendment start-gate checklist closing line → **superseded** by the amended gate (approval of this revision first; VS-00 only; stop-accept before VS-01).
7. **doc 02 §0.1 prescriptive ladder (rung 1)** — extended in place: GREENFIELD owner requirements now read together with the Owner Amendment at the same rank, later-in-time prevailing.

Nothing else is superseded. In particular, **no** invariant text, gate threshold, reason code, capability entry, ADR, or dataset definition changed meaning in this pass.

## 9. How to verify this pass (for the approval meeting)

1. **Coverage:** open `TRACEABILITY_OWNER_AMENDMENTS.md` — every Amendment §§2–11 subsection and every §12 mandate has a row with owner, feature IDs, slice, acceptance reference, and status; no blanks (its §0 rule).
2. **Conflicts:** doc 02 §2 Tier 3 carries CON-30…CON-34 with resolutions matching §5 above — no other document re-litigates them.
3. **Decisions:** doc 02 §4 rows SD-18…SD-22 cite [DECISION owner 2026-08-03, Amendment §x]; doc 22 §3.2 carries OD-28…OD-35 with the §13 safe assumptions; next free OD is OD-36.
4. **Gate:** doc 00 §6 requires approval of this revision before any product code, and names VS-00 as the only first executable order with a stop-accept before VS-01.
5. **No-weakening:** spot-check any of I1–I18 in doc 23 §2 and the R6/R7 gates in docs 11/12/15 — text unchanged; the amendment's §0.5 list maps onto them 1:1 (§7 above).
6. **Naming discipline:** search the package for release waves — only "Wave A/B/C" appears; "P0/P1/P2" occur solely as doc 21 phase names; ops metrics and feature flags appear only under their exact §1 names.

---

*End of PLAN_CHANGELOG. Requirement-level coverage: `TRACEABILITY_OWNER_AMENDMENTS.md`. Approval consequence: doc 00 §6 (the start gate now begins with approval of this revision).*

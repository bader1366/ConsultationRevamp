# TRACEABILITY_OWNER_AMENDMENTS — Requirement Matrix for the Owner Amendment (2026-08-03)
**Platform:** Nwafeth Intelligence — منصة نوافث لذكاء الجلسات الاستشارية (Monsha'at Advisory Session Intelligence Platform)
**Status:** Amendment-pass record for owner review · **Date:** 2026-08-04 · **Author:** Planning package (Fable 5)
**Depends on:** OWNER-AMENDMENT (2026-08-03); docs 00–25; PLAN_CHANGELOG.md · **Feeds:** the owner's approval of this revision; doc 00 §6 start gate; future audit of the build
**Sources used:** OWNER-AMENDMENT §§2–15 in full; doc 02 §1.11/§4/§5.1; doc 22 §3.2; doc 24 (waves), doc 25 (slices)

This is the Traceability Matrix the Owner Amendment §14.3 requires: **every numbered requirement or subsection of Amendment §§2–11 and every per-document mandate of §12** is mapped to (a) the plan document(s) and section owning it, (b) Feature IDs (PB-xxx backlog items and/or CAP-OPS-xx operational capabilities), (c) the Vertical Slice VS-xx that delivers it, (d) an acceptance reference, and (e) an integration status.

## 0. Column semantics and status vocabulary

- **§ref** — the Amendment section (AMD = OWNER-AMENDMENT 2026-08-03). Sub-item numbers in parentheses, e.g. AMD §3.3(5).
- **Owning doc(s)+section** — where the requirement now lives. Sections created by this 2026-08-04 pass are cited by their mandated topic (the owning document numbers its own sections); pre-existing sections are cited by number.
- **Feature ID(s)** — PB-xxx (doc 24) and CAP-OPS-xx (doc 04). "protocol" = a delivery-process requirement carried by docs 23/25 rather than a product feature; "registry" = carried by a registry entry.
- **VS** — the vertical slice (doc 25) in which the requirement is first demonstrated; "all VS" for cross-slice rules; "cross" for package-level artifacts.
- **Acceptance ref** — the check that proves it: an AMD acceptance-criteria block (§4.6, §5.7, §7.7, §9.3 per-slice bullets, §10.3 DoD, §14), a doc 25 per-slice checklist item, a doc 15 test/dataset, or an experiment (EXP-01/EXP-11).
- **Status** — exactly one of:
  - **integrated** — carried into the amended package as binding plan content;
  - **integrated-with-assumption OD-xx** — integrated, planning proceeds on the named safe assumption until the owner decides;
  - **deferred-to-backlog-consult SD-21** — integrated into doc 24 as a governed backlog item; entering any implementation slice requires explicit owner consultation and approval first (Wave B/C and unbound Wave A items).
- **No blanks:** every cell is filled; where a column does not naturally apply, the cell states why (e.g. "protocol", "cross").

### 0.1 Coverage statistics (self-check)

| Amendment part | Rows in this matrix | Section below |
|---|---|---|
| §2 operational loops | 6 | §1 |
| §3 DataHub | 8 | §2 |
| §4 nightly run | 14 (incl. §4.4 rules split 1–9) | §3 |
| §5 «اشتباه مخالفة» | 7 | §4 |
| §6 governed learning | 7 | §5 |
| §7 infographic | 7 | §6 |
| §8 backlog | 4 | §7 |
| §9 slices + exposure + stop rule | 20 (incl. L0–L3 split) | §8 |
| §10 execution protocol | 16 (incl. the 10 prohibitions split) | §9 |
| §11 additional features | 9 | §10 |
| §12 per-document mandates | 26 | §11 |
| Wave A feature→slice binding | 18 | §12 |
| §13/§14/§15 supplementary coverage | full | §13 |

## 1. AMD §2 — the six operational loops

| §ref | Requirement | Owning doc(s)+section | Feature ID(s) | VS | Acceptance ref | Status |
|---|---|---|---|---|---|---|
| AMD §2.1 | Daily data loop: sources → raw immutable intake → canonical session → reconciliation → enrichment → quality gates → serving views | doc 09 (nightly-consolidation chapter) + doc 05 (SRC-DATAHUB family) | PB-001, PB-003; CAP-OPS-01 | VS-01→VS-03 | AMD §4.6; doc 25 VS-03 checklist | integrated |
| AMD §2.2 | Violation-review loop: detection → «اشتباه مخالفة» → human review → decisions → audit trail; indicator ≠ proven violation | doc 10 (suspected/approved rules) + doc 18 SCR-08 | PB-006…PB-009; CAP-OPS-05 | VS-02 | AMD §5.7; doc 25 VS-02 checklist | integrated |
| AMD §2.3 | Detector-improvement loop: labels → versioned dataset → candidate → shadow → approval → deploy → monitor → rollback | doc 14 (release lifecycle) + doc 15 (label datasets) | PB-011; CAP-OPS-07 | VS-04 | doc 25 VS-04 checklist (AMD §9.3 VS-04 bullets) | integrated |
| AMD §2.4 | Service-improvement loop: finding → action → owner → due date → status → completion evidence → post-action metric comparison | doc 04 CAP-OPS-09 + doc 10 (impact-tracking method) + doc 18 SCR-19 | PB-015 (PB-120 follow-on); CAP-OPS-09 | VS-06 | doc 25 VS-06 demo script | integrated-with-assumption OD-34 |
| AMD §2.5 | Leadership-communication loop: month close → completeness gate → one-page infographic → review → approval → publication → drill-down | doc 04 CAP-OPS-08 + doc 11 (pack path) + doc 18 SCR-20 | PB-014; CAP-OPS-08 | VS-05 | AMD §7.7; doc 25 VS-05 checklist | integrated |
| AMD §2.6 | New-question loop: Lane 0/1 when governed → Lane 2 on ambiguity → Lane 3 for raw analysis → reviewed reusable artifact → promotion candidate | docs 12/13 (lanes, promotion) + doc 24 (PB-208 board) | PB-201…PB-208 | VS-09/VS-10 (entry via consult) | doc 25 VS-09/VS-10 checklists | deferred-to-backlog-consult SD-21 |

## 2. AMD §3 — DataHub as a first-class institutional source

| §ref | Requirement | Owning doc(s)+section | Feature ID(s) | VS | Acceptance ref | Status |
|---|---|---|---|---|---|---|
| AMD §3.1 | `SRC-DATAHUB` family + six sub-contracts (SESSION / BENEFICIARY-EVAL / CONSULTANT-EVAL / OUTCOME / DIRECTORY / REFERENCE); supersedes the SRC-INT-centric framing | doc 05 (family section + supersession note; CON-32) | PB-003; CAP-OPS-01 | VS-01 | doc 25 VS-01 checklist; EXP-01 join-rate | integrated-with-assumption OD-07 |
| AMD §3.2 | Field Authority Matrix (session status/schedule/programme: DataHub governs; transcript text/speakers/timing: active provider only; audio presence ≠ DataHub attendance; consultant identity: Directory; evaluations: DataHub direct; findings + review state: the new platform, append-only) | doc 05 (authority matrix) + doc 08 (crosswalks) | PB-003 | VS-01 | doc 25 VS-01 (per-field source shown in Session 360) | integrated |
| AMD §3.3(1–2) | Stable `advisory_session_id`; all source IDs kept in a crosswalk; no title-parsing as the primary match mechanism | doc 08 §0.2 identity strategy + doc 09 identity ladder | PB-003, PB-004 | VS-01 | doc 25 VS-01 ("Session Crosswalk واضح"; no dupes) | integrated |
| AMD §3.3(3) | Daily measures: provider↔DataHub match rate, transcript coverage, duplicates, time/status conflicts, late data | doc 09 (reconciliation metrics) + doc 11 registry (`source_join_rate`, `data_completeness`) | PB-012; CAP-OPS-02 | VS-03 | AMD §4.6 (dashboard from real data) | integrated |
| AMD §3.3(4) | «مشكلات الربط والبيانات» queue for the Data Steward | doc 09 (steward queues) + doc 18 (DQ board) | PB-012 | VS-03 | doc 25 VS-03 checklist | integrated |
| AMD §3.3(5) | Late-arriving data updates Session 360 without unnecessary full rebuilds | docs 05 (late-arriving semantics) + 09 (incremental enrichment) | PB-003, PB-004 | VS-03 | AMD §4.6 ("وصول تقييم متأخر يحدث الجلسة") | integrated |
| AMD §3.3(6) | Every internal update carries `source_record_version`, `observed_at`, `effective_at` where available | doc 05 (contract fields) + doc 08 (ingest provenance columns) | PB-003 | VS-01 | doc 25 VS-01 (field provenance visible) | integrated |
| AMD §3.3(7) | One sub-contract failing does not block the others; the run becomes `partial` and shows it | doc 09 (run states + per-adapter isolation) | PB-001 | VS-03 | AMD §4.6 ("فشل DataHub Evaluation ⇒ partial") | integrated |

## 3. AMD §4 — «التشغيل الليلي الموحد» (Nightly Consolidation Run)

| §ref | Requirement | Owning doc(s)+section | Feature ID(s) | VS | Acceptance ref | Status |
|---|---|---|---|---|---|---|
| AMD §4.1 | Named pipeline; 02:00 Asia/Riyadh working assumption, configurable; hourly incremental allowed, nightly run is authoritative (closes the previous day) | doc 09 (nightly chapter §naming/timing) | PB-001; CAP-OPS-01; flag `nightly_consolidation_enabled` | VS-03 | AMD §4.6; doc 25 VS-03 checklist | integrated-with-assumption OD-28 |
| AMD §4.2 | The 16 mandatory steps (Preflight → Pulls → Validate → Normalize → Reconcile → Activate transcript → Enrich → Score → Detect → Review-case gen → Evidence index → Refresh views → DQ/completeness → Alert/digest → Month-close trigger) | doc 09 (step table) + doc 07 (Nightly Orchestrator component) | PB-001 | VS-03 | AMD §4.6 (idempotent double-run) | integrated |
| AMD §4.3 | Run states exactly `queued → running → partial → succeeded → failed → cancelled → superseded`; run record carries run_id, watermarks, counts, per-step status, classified errors, retries, code/contract/model/prompt/taxonomy versions, DLQ link | doc 08 (`pipeline_run`, `pipeline_step_run`) + doc 09 (run manifest) | PB-001, PB-002 | VS-03 | doc 25 VS-03 (run-manifest download) | integrated |
| AMD §4.4(1) | Idempotent runs: re-running the same period never duplicates records or review cases | doc 09 (failure/resume rules) + doc 08 (natural keys as pins) | PB-001 | VS-03 | AMD §4.6 ("إعادة نفس التشغيل مرتين لا تغير الأعداد") | integrated |
| AMD §4.4(2) | Every source adapter has an independent watermark | doc 08 (`source_watermark`) + doc 09 (per-adapter cursoring) | PB-001 | VS-03 | doc 25 VS-03 (watermarks visible per source) | integrated |
| AMD §4.4(3) | One element's failure never fails the whole batch — and is never converted to silent success | doc 09 (typed-failure contract; I16) | PB-001 | VS-03 | AMD §4.6 (partial semantics) | integrated |
| AMD §4.4(4) | Every failed element enters the DLQ with a specific, retryable reason | doc 08 (`dead_letter_item`) + doc 09 (DLQ states) | PB-001, PB-012 | VS-03 | doc 25 VS-03 (DLQ inspection) | integrated |
| AMD §4.4(5) | No official infographic/report from an incomplete run without a documented override shown on the report face | doc 10 (completeness gate) + doc 04 CAP-OPS-08 | PB-014 | VS-05 | AMD §7.3 + §7.7 | integrated |
| AMD §4.4(6) | Re-running a stage does not re-execute unaffected stages when digests are unchanged | doc 09 (content-digest re-run scoping) | PB-001 | VS-03 | doc 25 VS-03 checklist | integrated |
| AMD §4.4(7) | A transcript active-source change invalidates derived results and triggers an orderly rebase — never a silent recompute | doc 06 (rebase integration) + doc 09 | PB-001 | VS-03 | doc 06 rebase quote-gate | integrated |
| AMD §4.4(8) | Operator commands exist: `rerun failed items` and `reprocess selected sessions` | doc 09 (playbooks) + doc 17 (`POST /pipeline-runs/{id}/retry`) | PB-001, PB-002 | VS-03 | doc 25 VS-03 demo (retry path) | integrated |
| AMD §4.4(9) | Resumability never depends on local JSON files or process memory | doc 09 (state in `jobs`/`ingest` schemas; OBS-17 lesson) | PB-001 | VS-03 | doc 25 VS-03 (restart mid-run test) | integrated |
| AMD §4.5 | Screen «مركز تشغيل البيانات»: last run + status, per-source health, new/changed session counts, step durations, unmatched sessions, sessions without transcript, new suspicions, overdue reviews, month completeness, actions (retry/open issue/logs/reprocess/manifest) | doc 18 SCR-14 + doc 17 (pipeline-run APIs) | PB-002; CAP-OPS-02 | VS-03 | AMD §4.6 (no hardcoded values) | integrated |
| AMD §4.6 | The six nightly acceptance criteria (20+20 sample E2E without dupes; idempotent double-run; partial on eval-feed failure; late evaluation appears next run; suspicion opens correct text; dashboard from real data) | doc 25 VS-03 acceptance checklist + doc 15 (per-slice suite) | PB-001, PB-002 | VS-03 | AMD §4.6 verbatim | integrated |

## 4. AMD §5 — «اشتباه مخالفة» and the review cycle

| §ref | Requirement | Owning doc(s)+section | Feature ID(s) | VS | Acceptance ref | Status |
|---|---|---|---|---|---|---|
| AMD §5.1 | User-visible name «اشتباه مخالفة» + the fixed disclaimer («الحالات في هذه الصفحة مؤشرات آلية تحتاج مراجعة بشرية، ولا تعد مخالفة مثبتة قبل اعتمادها») | doc 18 SCR-08 (upgraded; fixed copy) | PB-006; CAP-OPS-05; flag `violation_review_enabled` | VS-02 | AMD §5.7; doc 25 VS-02 checklist | integrated |
| AMD §5.2 | Queue rows (type, priority, detector confidence, programme, consultant per permission, date, verbatim short quote, case age, review state, second-review flag, detector/taxonomy version) + filters incl. SLA-overdue and reopened-by-change | doc 18 SCR-08 (list spec) + doc 17 (`GET /violations/suspected`) | PB-006 | VS-02 | doc 25 VS-02 checklist | integrated-with-assumption OD-30 |
| AMD §5.3 | Case detail: official definition, quote with ≥5 turns before/after, speaker+role+timing, jump-to-position in full transcript, DataHub context, detection basis (rule/model/prompt/threshold), similar approved cases without uncontrolled anchoring, append-only decision log, reviewed-evidence fingerprint, change warning | doc 18 (case screen) + doc 08 (fingerprints, `review_case`) | PB-007 | VS-02 | AMD §5.7 (verbatim quote gate) | integrated |
| AMD §5.4 | Six closed decisions — مخالفة صحيحة / ليست مخالفة / إعادة تصنيف / تحتاج مراجعة ثانية / أدلة غير كافية / إحالة لمالك السياسة — each with governed reason code, note, reviewer identity+time, text/detector/taxonomy versions, no delete/replace of prior decisions | doc 08 (`review_event` append-only) + doc 17 (`POST /violations/cases/{id}/review-events`) + doc 18 | PB-008 | VS-02 | AMD §5.7 (audit trail undeletable) | integrated |
| AMD §5.5 | Separation `violation_finding` / `review_case` / `review_event`; one session may hold two findings with opposite outcomes | doc 08 (entity trio) + doc 10 (counting rules) | PB-007, PB-008 | VS-02 | AMD §5.7 ("قبول مؤشر ورفض آخر في الجلسة نفسها") | integrated |
| AMD §5.6 | First vertical type VIOL-008 «التواصل خارج الإطار الرسمي» (deterministic rules + LLM verifier where needed), then one type at a time — never all 8 at once | doc 24 PB-009 card + doc 25 VS-02 scope | PB-009 | VS-02 | doc 25 VS-02 (labelled-sample measurement) | integrated-with-assumption OD-35 |
| AMD §5.7 | The seven suspicion-page acceptance criteria (verbatim quotes; authorized reviewer only; approved leaves «مشتبه»; fingerprint invalidation on transcript/extraction change; per-finding decisions; append-only audit; no unapproved case in leadership numbers as confirmed) | doc 25 VS-02 acceptance checklist + doc 15 (per-slice suite) | PB-006…PB-009 | VS-02 | AMD §5.7 verbatim | integrated |

## 5. AMD §6 — governed learning from review decisions

| §ref | Requirement | Owning doc(s)+section | Feature ID(s) | VS | Acceptance ref | Status |
|---|---|---|---|---|---|---|
| AMD §6.1 | Detector improves through versioned release cycles with shadow testing — never direct online self-learning from clicks | doc 14 (lifecycle doctrine) + doc 02 §4 SD-22 | PB-011; CAP-OPS-07 | VS-04 | doc 25 VS-04 (promotion requires documented approval) | integrated |
| AMD §6.2 | Eight labelled-data sources: confirmed positives, hard negatives, reclassifications, missed-case reports, unflagged random sample, disagreement set, boundary set, model-disagreement set | doc 15 (label-dataset plan) + doc 08 (`label_dataset`, `label_dataset_item`) | PB-011 | VS-04 | doc 25 VS-04 (dataset build step) | integrated |
| AMD §6.3 | «إضافة اشتباه لم يرصده النظام» button in Session 360 + full transcript: select span, pick type or «نوع جديد», reason, route to second review | doc 18 SCR-17 + doc 17 (`POST /sessions/{id}/missed-violation`) | PB-010; CAP-OPS-06 | VS-04 | doc 25 VS-04 (missed-case flow demo) | integrated |
| AMD §6.4 | Weekly stratified random sample of unflagged sessions (strata: programme, consultant, duration, month, transcript quality, under-represented classes) → light review → FN-rate estimate | doc 10 (FN-estimation method) + doc 15 EXP-11 | PB-011 | VS-04 | EXP-11 design artifact | integrated-with-assumption OD-32 |
| AMD §6.5 | Release line: immutable dataset (train/validation/holdout, no session leakage) → candidate (what changed) → offline eval per type with CIs → regression eval → shadow run → human adjudication of diffs → documented promotion → canary → full deploy with version stamp → monitoring → instant rollback without losing decisions | doc 14 (detector release lifecycle table) | PB-011; CAP-OPS-07 | VS-04 | doc 25 VS-04 (shadow compare + proven rollback) | integrated-with-assumption OD-33 |
| AMD §6.6 | Detector quality metrics: per-type precision + estimated recall, FP rate, FN estimate, review acceptance rate, reclassification rate, reviewer disagreement, mean review time, SLA-overdue share, per-type coverage, drift by month/programme/consultant/quality, invalidated-by-change share | doc 15 (metric definitions) + doc 19 (`detector_acceptance_rate` monitoring) + doc 11 registry | PB-011 (PB-113 follow-on) | VS-04 | doc 15 per-category eval spec | integrated |
| AMD §6.7 | Anti-bias rules: no single reviewer decision reaches production directly; holdout never trained on; adjudicator for sensitive disagreements; definition shown, not example floods; consultant names never features; beneficiary rating alone never implies a violation; every release stamps dataset version + prompt SHA + model ID + thresholds + taxonomy version | docs 14 (release constraints) + 15 (holdout discipline) + 16 (fairness controls) | PB-011 | VS-04 | doc 25 VS-04 checklist | integrated |

## 6. AMD §7 — the monthly leadership infographic

| §ref | Requirement | Owning doc(s)+section | Feature ID(s) | VS | Acceptance ref | Status |
|---|---|---|---|---|---|---|
| AMD §7.1 | Standalone product «نبض خدمة الاستشارات والإرشاد — ملخص الشهر» — a one-page executive infographic, not a sub-renderer of CAP-D10 | doc 04 CAP-OPS-08 + doc 24 PB-014 card | PB-014; CAP-OPS-08; flag `monthly_infographic_enabled` | VS-05 | AMD §7.7 | integrated |
| AMD §7.2 | Formats: in-platform page, one-page PDF, high-res PNG, permalink; approved internal channels (Portal + Email; Teams later); **no image-generation models; charts drawn programmatically from structured data; LLM phrasing only after the R6 numeric-provenance gate** | doc 11 (render path from pack artifact) + doc 18 SCR-20 + doc 14 (no-image-gen rule) | PB-014 | VS-05 | AMD §7.7 (byte-identical payloads) | integrated-with-assumption OD-29 |
| AMD §7.3 | Draft after month close + nightly run: morning of day 2 or completeness ≥98%, whichever later; completeness override shown on the face | doc 09 (month-close trigger) + doc 10 (completeness gate) | PB-014 | VS-05 | AMD §7.3 + §7.7 | integrated (98% gate = §13 working assumption, configurable) |
| AMD §7.4 | One-page content: نبض الخدمة، جودة وأثر الجلسات، صوت المستفيد، الجودة والالتزام (**approved-only violations; suspected as separate queue counts; no consultant names in the general edition**)، ما يحتاج قرارًا; mandatory footer (period, last update, sessions used/matched/total, exclusions, taxonomy versions, corpus snapshot, drill-down link) | doc 10 (KPI selection/suppression) + doc 18 SCR-20 (layout) | PB-014 | VS-05 | AMD §7.7 (every number traceable to a Metric Result) | integrated-with-assumption OD-31 |
| AMD §7.5 | Lifecycle `draft → data_review → content_review → approved → published → superseded/retracted`; roles: Data Owner (coverage/numbers), Service Owner (meaning), Publication Authority (release); reissue never edits a published version | doc 08 (`monthly_infographic`, `publication_event`) + doc 17 (approve/publish APIs) + doc 16 (publication RBAC) | PB-014 | VS-05 | doc 25 VS-05 (draft→approve→publish demo) | integrated-with-assumption OD-10 |
| AMD §7.6 | One fixed template first; later variants (general leadership, operations, per-programme, quarterly comparison) — no free-design system in v1 | doc 18 SCR-20 (template rules) + doc 24 (variants as backlog) | PB-014 (v1); variants via doc 24 | VS-05 | doc 25 VS-05 checklist | integrated (variants deferred-to-backlog-consult SD-21) |
| AMD §7.7 | The eight infographic acceptance criteria (traceable numbers, no LLM-computed figures, end-exclusive periods, approved-only in the violations figure, byte-identical payloads across web/PDF/PNG, KPI drill-down within permissions, regeneration from pack artifact without DOM scraping, readable on leadership screens + A4) | doc 25 VS-05 acceptance checklist + doc 15 (payload-identity test) | PB-014 | VS-05 | AMD §7.7 verbatim | integrated |

## 7. AMD §8 — the product backlog

| §ref | Requirement | Owning doc(s)+section | Feature ID(s) | VS | Acceptance ref | Status |
|---|---|---|---|---|---|---|
| AMD §8.1 | Every feature carries: Feature ID, primary user, problem/value, data+capabilities needed, sensitivity, dependencies, testable acceptance, release wave, feature flag; ranking by Value/Risk/Dependency | doc 24 (card format + ranking) | all PB-xxx | cross (planning artifact) | doc 24 card-completeness rule | integrated |
| AMD §8.2 | Wave A (amendment "P0" renamed): PB-001…PB-018 — first usable, testable product | doc 24 Wave A + doc 25 slice bindings | PB-001…PB-018 | VS-00…VS-07 | doc 25 per-slice checklists | integrated |
| AMD §8.3 | Wave B (amendment "P1" renamed): PB-101…PB-120 — service improvement + team enablement after the core is stable | doc 24 Wave B | PB-101…PB-120 | VS-08+ (entry via owner consult) | doc 24 acceptance columns | deferred-to-backlog-consult SD-21 |
| AMD §8.4 | Wave C (amendment "P2" renamed): PB-201…PB-212 — agentic intelligence + expansion after proven usage and accuracy | doc 24 Wave C | PB-201…PB-212 | VS-09…VS-12 (entry via owner consult) | doc 24 acceptance columns | deferred-to-backlog-consult SD-21 |

## 8. AMD §9 — vertical-slice delivery map

| §ref | Requirement | Owning doc(s)+section | Feature ID(s) | VS | Acceptance ref | Status |
|---|---|---|---|---|---|---|
| AMD §9.1 | Slice principle: small complete path source→screen; each slice includes only what it needs of migration, backend/domain, API, UI, fixture, automated tests, manual demo script, observability, rollback, documentation | doc 25 (slice content rule) + doc 21 (overlay statement) | protocol | all VS | doc 25 slice-content checklist | integrated |
| AMD §9.2 | Four exposure levels adopted verbatim; L3 requirements never block L1; L1 success never authorizes L2/L3 | doc 25 (exposure table) + docs 21/15/20 (gate scoping; CON-30) | protocol | all VS | doc 25 exposure table; CON-30 record | integrated |
| AMD §9.2-L0 | L0 Developer: synthetic data, developer/agent audience, unit + component tests as the gate | doc 25 (L0 row) + doc 15 (Tier P fixtures) | protocol | all VS | CI green on the committed synthetic corpus | integrated |
| AMD §9.2-L1 | L1 Owner Preview: synthetic or limited governed staging sample; owner + project team; feature-acceptance gate; **no operational decisions from L1** | doc 25 (L1 row) + doc 00 §6 (stop-accept) | protocol | every VS from VS-01 | doc 25 per-slice demo scripts | integrated |
| AMD §9.2-L2 | L2 Controlled Pilot: specified real data, authorized users only, the feature's own accuracy/security gates | doc 25 (L2 row) + doc 20 (pilot gates) + doc 16 | protocol | VS-12 (first L2) | feature gates (EXP-06/09/10 as applicable) | integrated |
| AMD §9.2-L3 | L3 Production: production data, approved internal audience, full release gates + rollback + ops readiness | doc 25 (L3 row) + docs 19/20 (readiness, G-gates) | protocol | VS-12+ | doc 20 G0…G5 + doc 19 readiness checks | integrated |
| AMD §9.3 VS-00 | Walking skeleton: app runs, login+RBAC, migration, CI, logging, home page, health/readiness, seed fixture; unified start command; green CI; no default credentials; PII-free fixture; real (not hardcoded) status page | doc 25 VS-00 + doc 23 (first five ticket specs) | PB-017 (RBAC baseline); protocol | VS-00 | AMD §9.3 VS-00 bullets; doc 25 VS-00 checklist | integrated |
| AMD §9.3 VS-01 | First unified session: 20 Read.ai + 20 DataHub records linked; Session 360 shows transcript + service data + evaluations + per-field provenance; no dupes; visible unmatched; idempotent re-ingest | doc 25 VS-01 + doc 05 (contracts) | PB-003, PB-004, PB-005 (minimal); CAP-OPS-03/04; flag `session_360_enabled` | VS-01 | doc 25 VS-01 checklist | integrated-with-assumption OD-07 |
| AMD §9.3 VS-02 | First violation source→human decision: VIOL-008 end-to-end; correct quote; finding/case/event separated; audit trail; decisions never vanish or attach to the wrong finding | doc 25 VS-02 + doc 18 SCR-08 | PB-006…PB-009; CAP-OPS-05; flag `violation_review_enabled` | VS-02 | AMD §5.7 | integrated-with-assumption OD-35 |
| AMD §9.3 VS-03 | Nightly run + ops center on a limited sample with watermarks, retries, DLQ; manual staging trigger; stages/counts/errors visible | doc 25 VS-03 + doc 09 (nightly chapter) | PB-001, PB-002, PB-012, PB-016 (digest emission); CAP-OPS-01/02/10; flag `nightly_consolidation_enabled` | VS-03 | AMD §4.6 | integrated-with-assumption OD-28 |
| AMD §9.3 VS-04 | First learning loop: VS-02 decisions → Dataset v1; candidate detector; shadow comparison on a declared dataset; no automatic promotion; versioned dataset, separate holdout, per-category results, approval-gated promotion, proven rollback | doc 25 VS-04 + doc 14 (lifecycle) | PB-010, PB-011; CAP-OPS-06/07 | VS-04 | AMD §9.3 VS-04 bullets; EXP-11 | integrated-with-assumption OD-32 |
| AMD §9.3 VS-05 | First monthly infographic: one template over a fixture/staging month; web+PDF+PNG; draft→approve→publish | doc 25 VS-05 + doc 18 SCR-20 | PB-014; CAP-OPS-08; flag `monthly_infographic_enabled` | VS-05 | AMD §7.7 | integrated |
| AMD §9.3 VS-06 | Ops dashboard + Action Center: core KPIs, drill-down, create/track an action (e.g. clear-steps drop → training action → status update) | doc 25 VS-06 + doc 18 SCR-19 + doc 04 CAP-OPS-12 | PB-013, PB-015; CAP-OPS-09/12; flag `action_center_enabled` | VS-06 | doc 25 VS-06 demo script | integrated-with-assumption OD-34 |
| AMD §9.3 VS-07 | First five deterministic capabilities (session counts/states by period+programme; beneficiary rating + distribution; clear-steps rate; suspected/approved violations + review state; top recurring challenges/questions per the first approved semantic artifact) — documented numbers, not the whole CAP catalogue | doc 25 VS-07 + doc 11 (MVP registry set) | PB-013, PB-018 (drill-down primitives); registry (CAP-B1/C1 subset) | VS-07 | doc 25 VS-07 (provenance-documented answers) | integrated |
| AMD §9.3 VS-08 | Operational-analysis expansion: Service Recovery Queue, limited Consultant 360, recurring fixed answers, inconsistent answers, weekly pulse | doc 25 VS-08 + doc 24 cards | PB-101, PB-102, PB-105, PB-106, PB-118; CAP-OPS-11 | VS-08 | doc 25 VS-08 checklist | deferred-to-backlog-consult SD-21 |
| AMD §9.3 VS-09 | Lanes 1/2: bounded agent over proven capabilities only; zero wrong route on golden questions; no model numbers; correct clarification/decline | doc 25 VS-09 + doc 12 (sequencing) | PB-203, PB-204; flag `lane1_agent_enabled` | VS-09 | doc 25 VS-09 checklist (golden-set route test) | deferred-to-backlog-consult SD-21 |
| AMD §9.3 VS-10 | Lane 3 for one real question: full job over one month, map→verify→reduce, progress + resumability | doc 25 VS-10 + doc 13 | PB-205; flag `lane3_analysis_enabled` | VS-10 | doc 25 VS-10 checklist (resumable job, verified findings) | deferred-to-backlog-consult SD-21 |
| AMD §9.3 VS-11 | Catalogue + official packs expansion: priority capabilities, quarterly report, frozen/live views, reissue/retraction | doc 25 VS-11 + doc 11/doc 04 (CAP-D10 path) | PB-201, PB-119; registry | VS-11 | doc 25 VS-11 checklist | deferred-to-backlog-consult SD-21 |
| AMD §9.3 VS-12 | Pilot, scale-out, cutover — only after prior slices prove out + security/performance/recovery/parallel-run tests | doc 25 VS-12 + doc 20 (G0…G5 gates) | PB-017/PB-018 (full); protocol | VS-12 | doc 20 gate sheet; EXP-09/EXP-10 | integrated |
| AMD §9.4 | Mandatory stop after every slice: agent stops → shows work → runs acceptance tests → records real gaps → obtains approval → never auto-starts the next slice | doc 25 (stop protocol) + doc 23 (no-auto-next rule) + doc 00 §6 (VS-00→stop→VS-01) | protocol | all VS | doc 25 stop-gate checklist | integrated |

## 9. AMD §10 — agent execution protocol

| §ref | Requirement | Owning doc(s)+section | Feature ID(s) | VS | Acceptance ref | Status |
|---|---|---|---|---|---|---|
| AMD §10.1 | Ticket size ≤4–8h agent work; split on >1 major migration or >1 independent UI journey; one domain per ticket unless a small vertical slice requires more; no unrelated cleanup bundled | doc 23 (ticket sizing rules) | protocol | all VS | doc 23 ticket template | integrated |
| AMD §10.2 | Pre-ticket plan (11 items: goal, out-of-scope, files, migration, API contract, UI flow, tests-first list, fixture data, demo steps, rollback, risks) before any edit | doc 23 (plan-file requirement) | protocol | all VS | doc 23 plan-file template | integrated |
| AMD §10.3 | Ticket DoD: complete code (no hidden stubs), testable migration, unit + integration/API tests, UI test or scripted manual acceptance, fixture/factory, authorization test, observability events/metrics, docs updated, `docs/STATE.md` + `PRODUCT_BACKLOG.md` + `KNOWN_GAPS.md` updated, run/test command + result, explicit not-done list | docs 23/25 (DoD checklist) | protocol | all VS | doc 25 DoD checklist | integrated |
| AMD §10.4 | Ten execution prohibitions adopted into the handoff MUST-NOT list (individually below) | doc 23 (MUST-NOT list, extended) | protocol | all VS | doc 23 §13 + per-PR evidence | integrated |
| AMD §10.4(1) | No starting several large epics in parallel by one agent | doc 23 (sequencing rule) + doc 25 (one slice at a time) | protocol | all VS | acceptance report per ticket | integrated |
| AMD §10.4(2) | No creating dozens of skeleton files with logic deferred | doc 23 (anti-skeleton rule) | protocol | all VS | PR review evidence (complete code, no hidden stubs) | integrated |
| AMD §10.4(3) | No TODO inside an accepted path unless behind a closed feature flag and listed in Known Gaps | doc 23 (TODO rule) + `KNOWN_GAPS.md` discipline | protocol | all VS | AMD §10.3 DoD (explicit not-done list) | integrated |
| AMD §10.4(4) | No modifying a test gate to pass a failing feature | doc 23 + doc 15 (no-weakening ratchet; I18) | protocol | all VS | doc 15 ratchet check | integrated |
| AMD §10.4(5) | No completing a backend ticket for a user-facing feature without API/UI or a suitable demo | doc 23 (user-facing completeness rule) | protocol | all VS | doc 25 demo scripts | integrated |
| AMD §10.4(6) | No hiding adapter/model-call failure by returning an empty list | doc 23 + doc 09 (typed-failure contract; I16) | protocol | all VS | failure-injection tests (doc 15) | integrated |
| AMD §10.4(7) | No hardcoded sample data or percentages in any dashboard | doc 23 + doc 19 (honest-labelling rule) | protocol | all VS | AMD §4.6 (dashboard from real data) | integrated |
| AMD §10.4(8) | No full-corpus processing before the same path succeeds on fixture then a small sample | doc 23 + doc 20 (staging ladder) | protocol | all VS | doc 20 cohort plan | integrated |
| AMD §10.4(9) | No deploying a new model/prompt directly from reviewer results without evaluation + approval | doc 23 + doc 14 (release lifecycle; SD-22) | protocol | VS-04 onward | doc 25 VS-04 (promotion gate) | integrated |
| AMD §10.4(10) | No moving to the next ticket without an acceptance report for the current one | doc 23 (no-auto-next) + doc 25 (stop protocol) | protocol | all VS | doc 25 stop-gate checklist | integrated |
| AMD §10.5 | Feature flags (exact): `session_360_enabled`, `violation_review_enabled`, `nightly_consolidation_enabled`, `monthly_infographic_enabled`, `action_center_enabled`, `lane1_agent_enabled`, `lane3_analysis_enabled` — isolation/exposure control, never a substitute for completion | docs 23/25 (flag table) + doc 19 (flag observability) | protocol (flags per feature) | VS-01…VS-10 | doc 25 flag table | integrated |
| AMD §10.6 | Dev-data strategy: small synthetic fixture covering critical cases → approved limited anonymized snapshot → staging cohort 20–100 sessions → one month → full backfill only after resumability/cost/DQ proven | doc 25 (data ladder) + doc 20 (cohort staging) | protocol | all VS | doc 20 staging plan | integrated |
| AMD §10.7 | Per-cycle outputs: one clear commit/patch, modified-files report, migration ID, test commands + results, screenshot/precise demo description for UI, seed/demo data IDs, known limits, decision needed — never a bare «تم الإنجاز» | doc 23 §12 (per-PR evidence, extended) | protocol | all VS | doc 23 §12 evidence list | integrated |

## 10. AMD §11 — additional monitoring/improvement features

| §ref | Requirement | Owning doc(s)+section | Feature ID(s) | VS | Acceptance ref | Status |
|---|---|---|---|---|---|---|
| AMD §11.1 | «صباحيات الخدمة» daily digest after nightly success: new sessions, new suspicions, overdue reviews, new 1–2★ ratings, no-clear-steps sessions, source issues, top alert — each linking to its screen | doc 04 CAP-OPS-10 + doc 09 (step 15) + doc 18 SCR-21 + doc 19 (digest) | PB-016; CAP-OPS-10 | VS-03 (emission) / VS-06 (screen) | doc 25 VS-03/VS-06 checklists | integrated |
| AMD §11.2 | Service Recovery Queue (low rating, explicit negative note, no clear steps, end-of-session pressure/confusion, repeated cancellation, follow-up recommended but absent) — fully separate from the violations queue | doc 04 CAP-OPS-11 + doc 18 SCR-22 + doc 10 (rules) | PB-101; CAP-OPS-11 | VS-08 | doc 24 PB-101 acceptance | deferred-to-backlog-consult SD-21 |
| AMD §11.3 | Rising-topics radar: rate per 100 sessions, month-over-month change, linked programmes/sectors, example sessions, real-change vs taxonomy-change flag | doc 10 (trend method) + doc 24 card | PB-107 | VS-08+ | doc 24 PB-107 acceptance | deferred-to-backlog-consult SD-21 |
| AMD §11.4 | Knowledge gaps: consistent repeated answers → FAQ/automation candidates; inconsistent → knowledge-unification/training candidates | doc 04 (CAP-B3/CAP-C6 engines) + doc 24 cards | PB-105, PB-106, PB-211 | VS-08+ | doc 24 acceptance columns | deferred-to-backlog-consult SD-21 |
| AMD §11.5 | Coaching cards per team manager — non-accusatory, no automatic punitive ranking; behaviour + approved examples + suggested training + review date + post-action metric change | doc 24 PB-103 card + doc 16 (RBAC/OD-31) | PB-103 | VS-08+ | doc 24 PB-103 acceptance | deferred-to-backlog-consult SD-21 |
| AMD §11.6 | Evaluation-contradiction detection (high rating but no clear steps; low rating with strong impact signals; consultant-positive vs beneficiary-negative) → opens a «تحتاج فهمًا» case, never declares either evaluation wrong | doc 04 (CAP-D1/CAP-D2 basis) + doc 24 card | PB-109 | VS-08+ | doc 24 PB-109 acceptance | deferred-to-backlog-consult SD-21 |
| AMD §11.7 | Governed watchlists: programme/challenge/violation-indicator/consultant-or-team-within-permissions/KPI-drop; every watchlist has reason, owner, expiry — no unbounded standing surveillance | doc 16 (anti-surveillance controls) + doc 24 card | PB-117 | VS-08+ | doc 24 PB-117 acceptance | deferred-to-backlog-consult SD-21 |
| AMD §11.8 | Best-practice library: approved sessions/spans showing diagnosis → steps → closure → impact evidence; beneficiary data masked; permission-holder approval | doc 24 PB-104 card + doc 16 (masking) | PB-104 | VS-08+ | doc 24 PB-104 acceptance | deferred-to-backlog-consult SD-21 |
| AMD §11.9 | Leadership Decision Log: links infographic/report → decision, owner, date, target metric, re-measure date, result | doc 24 PB-120 card + doc 04 (infographic part-5 hooks) | PB-120 | VS-08+ (part-5 hooks in VS-05) | doc 24 PB-120 acceptance; AMD §7.4-5 | deferred-to-backlog-consult SD-21 |

## 11. AMD §12 — per-document mandates

| §ref | Requirement (mandate summary) | Owning doc(s)+section | Feature ID(s) | VS | Acceptance ref | Status |
|---|---|---|---|---|---|---|
| AMD §12-00 | Register amendment; add docs 24/25; change start gate; add traceability | doc 00 §3/§2/§6 | protocol | VS-00 gate | PLAN_CHANGELOG §3-00; this file exists | integrated |
| AMD §12-01 | Six loops; interactive first value; infographic/digest/action-center value; preview-vs-pilot-vs-production | doc 01 (executive narrative) | PB-014/015/016 | VS-01…VS-06 | PLAN_CHANGELOG §3-01 | integrated |
| AMD §12-02 | Amendment as requirements source; DataHub family record; horizontal-vs-incremental conflict recorded | doc 02 §1.11 + §2 Tier 3 + §4 | evidence register (no PB) | cross | CON-30…CON-34 + SD-18…SD-22 present | integrated |
| AMD §12-03 | Eight personas + daily journeys, each with top tasks/alerts/permissions/decisions | doc 03 (personas section) | PB-002/004/006/014/015 users | VS-01…VS-06 | PLAN_CHANGELOG §3-03 | integrated |
| AMD §12-04 | Operational capability IDs; suspected vs approved separated; ops metrics added | doc 04 (CAP-OPS section) | CAP-OPS-01…12 | VS-01…VS-06 | I12 registry discipline check | integrated |
| AMD §12-05 | SRC-DATAHUB + sub-contracts; authority matrix; late-arriving/CDC/watermark/versions/keys/SLAs; Excel = temporary fallback | doc 05 (family + matrix) | PB-003 | VS-01 | EXP-01; doc 25 VS-01 | integrated-with-assumption OD-07 |
| AMD §12-06 | Provider pull in nightly run; rebase impact on cases/datasets/infographics; stale/needs-review transitions, no silent reopen | doc 06 (rebase §) | PB-001 | VS-03 | doc 06 quote-gate + doc 25 VS-03 | integrated |
| AMD §12-07 | Eight named components; agentic chat not the centre | doc 07 (component view) | CAP-OPS-01…09 components | VS-01…VS-06 | C4 updated; CON-34 record | integrated |
| AMD §12-08 | New/verified entities for runs, DQ, review, learning, infographic, actions, notifications | doc 08 (ERD additions) | PB-001…PB-016 storage | VS-01…VS-06 | doc 23 migration order | integrated |
| AMD §12-09 | Nightly chapter per §4; hourly vs nightly; digest + month-close; progress state; partial/late/replay tests | doc 09 (nightly chapter) | PB-001; CAP-OPS-01 | VS-03 | AMD §4.6 | integrated-with-assumption OD-28 |
| AMD §12-10 | Suspected/approved rules; FN estimation; action-impact method; infographic KPI selection/suppression | doc 10 (method additions) | PB-011/014/015 | VS-04/05/06 | doc 15 method tests | integrated |
| AMD §12-11 | VS-07 MVP set of five; catalogue not a precondition; ops/review/DQ/action metrics | doc 11 (MVP set + registry) | 7 ops metrics (exact names) | VS-07 | registry hash check | integrated |
| AMD §12-12 | Chat after operational stability; agent never creates/approves violations or publishes infographic/actions; safe read tools | doc 12 (sequencing + tools) | PB-203; flag `lane1_agent_enabled` | VS-09 | doc 25 VS-09 checklist | integrated |
| AMD §12-13 | Lane-3/RAG stays later; Promotion Board; TTL/expiry/owner per RAG artifact | doc 13 (lifecycle additions) | PB-205/206/208 | VS-10 | doc 25 VS-10 checklist | integrated |
| AMD §12-14 | §6.5 lifecycle; extractor/detector/composer separation; shadow/canary/rollback; names never features | doc 14 (detector lifecycle) | PB-011; CAP-OPS-07 | VS-04 | doc 25 VS-04 checklist | integrated-with-assumption OD-33 |
| AMD §12-15 | Missed-case + unflagged-sample datasets; per-category recall; disagreement/adjudication set; per-VS acceptance tests; web/PDF/PNG identity test | doc 15 (datasets + slice suites) | PB-010/011/014 | VS-02…VS-05 | EXP-11 artifact; payload-identity test | integrated-with-assumption OD-32 |
| AMD §12-16 | Permissions for infographic/Action Center/Consultant 360; no names in general infographic; queue anti-surveillance; notification/export/expiring-link policies | doc 16 (RBAC additions) | PB-017; CAP-OPS-08/09 | VS-00 onward | authorization tests (AMD §10.3 DoD) | integrated-with-assumption OD-31 |
| AMD §12-17 | The 15 API groups with RBAC, pagination, idempotency, versioning, Problem Details | doc 17 (endpoint additions) | PB-001…PB-016 APIs | VS-01…VS-06 | doc 17 contract tests | integrated |
| AMD §12-18 | Screens SCR-14…22 in build order; SCR-08 upgraded; chat moved later | doc 18 (screen additions + order) | CAP-OPS-02…12 screens | VS-01…VS-06 | role×screen matrix extension | integrated |
| AMD §12-19 | SLOs/alerts for nightly deadline, freshness, join rate, partial/failure, DQ backlog, review backlog age, detector acceptance/drift, infographic deadlines, action overdue, notification failures; digest + manifest + restore drill | doc 19 (SLO additions) | `nightly_run_success` + metric set | VS-03 onward | alert-rule fixtures | integrated |
| AMD §12-20 | Cohort staging 20→100→month→corpus; parallel run never blocks Owner Preview; suspected/approved compared separately; shadow detector migration | doc 20 (staging + parity additions) | PB-001/011 | VS-03/VS-04, VS-12 | parity difference categories | integrated |
| AMD §12-21 | Replace horizontal structure with VS-00…VS-12; experiments stay gates bound to affected features; no 26-cap precondition; demo/acceptance/stop per slice; release waves; smaller tickets | doc 21 (execution overlay) | protocol | all VS | PLAN_CHANGELOG §3-21; CON-31 record | integrated |
| AMD §12-22 | New open decisions registered with §13 assumptions; planning never blocked on them | doc 22 §3.2 (OD-28…OD-35; OD-07/OD-10 extended) | OD register (no PB) | cross | next-free = OD-36 check | integrated |
| AMD §12-23 | §10 protocol carried; no multi-epic order; VS-00 first + acceptance report before VS-01; ticket fields UI/API/tests/demo/rollback; no auto-next rule | doc 23 (protocol re-issue) | protocol | VS-00 | doc 23 protocol section; doc 00 §6 | integrated |
| AMD §12-24 | New `24-product-backlog.md`: §8+§11 as story cards; Value/Risk/Dependency ranking; owner/wave/acceptance per feature | doc 24 (whole document) | PB-001…PB-212 | cross | doc 24 card completeness | integrated |
| AMD §12-25 | New `25-incremental-delivery-and-acceptance.md`: §9+§10; demo script + acceptance checklist per VS; L0–L3; agent state-file template | doc 25 (whole document) | protocol | all VS | AMD §14.2 criteria | integrated |

## 12. Wave A binding — every committed backlog item to its slice

Supplementary view (the rows above are the requirement record; this is the feature-first cut for the approval meeting). Wave A = AMD §8.2. Per SD-21, PB-014 and PB-006…PB-010 are owner-committed to the earliest releases; the remaining Wave A items are bound to slices by AMD §9.3 itself and enter execution through each slice's stop-accept approval.

| PB | Feature (AMD §8.2) | CAP-OPS / flag | VS | Acceptance ref |
|---|---|---|---|---|
| PB-001 | Nightly Consolidation Run | CAP-OPS-01; `nightly_consolidation_enabled` | VS-03 | AMD §4.6 |
| PB-002 | «مركز تشغيل البيانات» | CAP-OPS-02 | VS-03 | AMD §4.5/§4.6 (real data only) |
| PB-003 | DataHub first-class connectors | (source family, doc 05) | VS-01 | doc 25 VS-01; EXP-01 |
| PB-004 | Session 360 | CAP-OPS-03; `session_360_enabled` | VS-01 | doc 25 VS-01 checklist |
| PB-005 | بحث الجلسات وتصفيتها | CAP-OPS-04 | VS-01 (minimal) / VS-08 (full filters) | doc 25 VS-01/VS-08 |
| PB-006 | Queue «اشتباه مخالفة» | CAP-OPS-05; `violation_review_enabled` | VS-02 | AMD §5.7 |
| PB-007 | Detailed violation case screen | CAP-OPS-05 | VS-02 | AMD §5.3/§5.7 |
| PB-008 | Append-only review | CAP-OPS-05 | VS-02 | AMD §5.4/§5.7 (audit trail) |
| PB-009 | VIOL-008 end-to-end | CAP-OPS-05 | VS-02 | doc 25 VS-02 (labelled sample) |
| PB-010 | Missed-case reporting | CAP-OPS-06 | VS-04 | doc 25 VS-04 (flow demo) |
| PB-011 | Dataset learning pipeline | CAP-OPS-07 | VS-04 | AMD §9.3 VS-04 bullets; EXP-11 |
| PB-012 | DQ/reconciliation queue «مشكلات الربط والبيانات» | CAP-OPS-02 (surface) | VS-03 | doc 25 VS-03 checklist |
| PB-013 | Initial ops dashboard | CAP-OPS-12 | VS-06 | doc 25 VS-06 (KPIs + drill-down) |
| PB-014 | Monthly infographic | CAP-OPS-08; `monthly_infographic_enabled` | VS-05 | AMD §7.7 |
| PB-015 | Action Center | CAP-OPS-09; `action_center_enabled` | VS-06 | doc 25 VS-06 (action lifecycle) |
| PB-016 | «صباحيات الخدمة» | CAP-OPS-10 | VS-03 (emission) / VS-06 (screen) | doc 25 VS-03/VS-06 |
| PB-017 | Role-based access and masking | (cross-cutting; doc 16) | VS-00 baseline, every slice after | authorization tests per AMD §10.3 DoD |
| PB-018 | Audit and provenance explorer | (cross-cutting; docs 11/18) | VS-07 (drill-down primitives); full surface via SD-21 consult | doc 25 VS-07 (number→run→session trace) |

## 13. Supplementary coverage — AMD §13, §14, §15

**§13 working assumptions** are carried verbatim in PLAN_CHANGELOG §6 and registered as OD-28…OD-35 (+ OD-07/OD-10 extensions) in doc 22 §3.2 — each row above that rests on one carries the matching `integrated-with-assumption OD-xx` status.

**§14 acceptance criteria for the re-planned package** — where each is satisfied:

| §ref | Criterion | Satisfied by |
|---|---|---|
| AMD §14.1-1 | Nightly run present in plan, architecture, UX, ops | rows §3 above: docs 07/09/18 SCR-14/19 |
| AMD §14.1-2 | DataHub first-class with contracts, authority, reconciliation | rows §2 above: doc 05 family + matrix |
| AMD §14.1-3 | «اشتباه مخالفة» defined end-to-end | rows §4 above: docs 08/10/17/18 |
| AMD §14.1-4 | Review→release learning loop governed and tested | rows §5 above: docs 14/15 |
| AMD §14.1-5 | FN sampling + missed-case reporting exist | AMD §6.3/§6.4 rows: PB-010/PB-011, EXP-11 |
| AMD §14.1-6 | Infographic is a standalone product with acceptance criteria | rows §6 above: PB-014, AMD §7.7 |
| AMD §14.1-7 | Action Center + Morning Digest in backlog and delivery map | PB-015/PB-016; VS-03/VS-06 rows |
| AMD §14.2-1 | Roadmap built on vertical slices | doc 21 overlay + doc 25 (CON-31) |
| AMD §14.2-2 | Every slice has demo, acceptance, stop gate | doc 25 per-slice checklists; AMD §9.4 row |
| AMD §14.2-3 | First Owner Preview does not wait for all capabilities | SD-18; L1 rules (CON-30); after VS-01 per §13 |
| AMD §14.2-4 | Every coding ticket small, specific, with DoD | AMD §10.1/§10.3 rows; doc 23 |
| AMD §14.2-5 | The agent cannot auto-advance to the next ticket | AMD §10.4(10)/§9.4 rows; doc 23 no-auto-next |
| AMD §14.3-1 | Traceability matrix requirement→doc→Feature ID→VS→acceptance test | this file (§§1–11) |
| AMD §14.3-2 | Change log over documents 00–23 | PLAN_CHANGELOG §3 |
| AMD §14.3-3 | No contradiction between roadmap and implementation handoff | docs 21/23 both point to doc 25 as the execution protocol (CON-31) |
| AMD §14.4-1 | No invariant on numbers/periods/evidence weakened | PLAN_CHANGELOG §7; doc 02 §4 note (§0.5 ↔ I-map) |
| AMD §14.4-2 | Suspected never shown as confirmed | SD-20; doc 10 rules; AMD §5.7/§7.4 rows |
| AMD §14.4-3 | No direct learning from a reviewer click | SD-22; doc 14 lifecycle rows |
| AMD §14.4-4 | LLM never publishes a number/chart not born of structured results | SD-19; AMD §7.2 row; R6 gate (I3/I18) |
| AMD §14.4-5 | No operational screen on hardcoded values | AMD §4.5/§10.4(7) rows; doc 19 honest-labelling rule |
| AMD §14.4-6 | Every user-facing feature has RBAC, audit, rollback | AMD §10.3 DoD row; doc 16 additions (§12-16 row) |
| AMD §14.5 | Required outputs: updated package; PLAN_CHANGELOG; docs 24/25; this matrix; resolved-conflicts list; open-decisions list; five VS-00 tickets (specs only, no execution) | this re-issue: all four files present; PLAN_CHANGELOG §5/§6; doc 23/25 (VS-00 ticket specs) |

**§15 mandatory start order** (VS-00 → stop → VS-01 → stop → VS-02 → stop → VS-03 → stop → VS-04 → stop → VS-05 → then expansion per approved backlog) is carried by doc 25 (slice order), doc 00 §6 (gate: VS-00 only as first executable order), and doc 23 (no-auto-next). Status: integrated.

## 14. Reverse index — plan document → amendment obligations

| Doc | Amendment sections it now carries |
|---|---|
| 00 | §0.2 start gate, §12-00, §14.3 (this matrix + changelog registered) |
| 01 | §2 loops, §12-01 |
| 02 | §12-02 (source registration, SD-18…22, CON-30…34, DataHub note) |
| 03 | §12-03 personas |
| 04 | §12-04 CAP-OPS-01…12, suspected/approved split, ops-metric refs; §2.4/§2.5 loop capabilities |
| 05 | §3 in full, §12-05 |
| 06 | §12-06 rebase/nightly coupling |
| 07 | §12-07 components + operational-core centrality (CON-34) |
| 08 | §12-08 entities; §4.3 run records; §5.5 finding/case/event |
| 09 | §4 in full, §12-09, §11.1 digest emission |
| 10 | §12-10 methods (suspected/approved, FN estimation, action impact, infographic KPIs) |
| 11 | §12-11 MVP five + ops metrics (7 exact names) |
| 12 | §12-12 agent boundaries + sequencing |
| 13 | §12-13 later-stage + promotion board + TTL |
| 14 | §6.5/§6.7, §12-14 detector lifecycle |
| 15 | §6.2/§6.4/§6.6, §12-15, EXP-11, per-VS acceptance suites |
| 16 | §12-16 permissions/masking/anti-surveillance/notifications |
| 17 | §12-17 API additions |
| 18 | §4.5, §5.1–§5.5, §12-18 screens SCR-14…22 + SCR-08 upgrade |
| 19 | §12-19 SLOs/alerts/digest/manifest/restore |
| 20 | §10.6 staging ladder, §12-20 parity/preview rules |
| 21 | §9 overlay, §12-21 |
| 22 | §12-22 OD-28…35, OD-07/OD-10 extensions |
| 23 | §10 in full, §12-23 |
| 24 | §8 + §11 in full, §12-24, wave mapping statement (CON-33) |
| 25 | §9 + §10 in full, §12-25, five VS-00 ticket specs (§14.5-8) |

## 15. Maintenance rules

1. **This matrix is updated whenever a slice closes**: the closing acceptance report cites the rows it discharges; rows never get deleted, their status may only move toward *integrated* (or gain a dated note).
2. **SD-21 discharge is recorded here**: when the owner approves a Wave B/C item (or an unbound Wave A item) into a slice, the row's status changes from `deferred-to-backlog-consult SD-21` to `integrated` with the consultation date.
3. **Assumption discharge**: when an OD-28…OD-35 decision lands, doc 22 §3.2 records it and the matching rows drop the `-with-assumption` qualifier.
4. **No row may ever be blank** — a requirement without an owner, slice, or acceptance reference is a planning defect to fix in the owning document, not here.

---

*End of TRACEABILITY_OWNER_AMENDMENTS. Per-document narrative: `PLAN_CHANGELOG.md`. Registers of record: doc 02 §4 (SD), doc 02 §2 (CON), doc 22 §3.2 (OD).*

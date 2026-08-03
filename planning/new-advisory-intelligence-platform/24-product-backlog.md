# 24 — Product Backlog (سجل ميزات المنتج — قائمة تشاور دائمة)
**Platform:** Nwafeth Intelligence — منصة نوافث لذكاء الجلسات الاستشارية (Monsha'at Advisory Session Intelligence Platform)
**Status:** Draft for owner review · **Date:** 2026-08-03 · **Author:** Planning package (Fable 5)
**Depends on:** 02 (SD-18…SD-22 register), 03 (personas), 04 (CAP-* / CAP-OPS-* registry), 05 (SRC-DATAHUB family), 16 (sensitivity + RBAC), 18 (SCR-*), 21 (phases P0…P7), 22 (OD register), 25 (slices VS-00…VS-12) · **Feeds:** 00, 21, 23, 25
**Sources used:** Owner Amendment 2026-08-03 §8 (backlog tables verbatim), §11 (monitoring features folded into cards), §5–§7 (violation / learning / infographic requirements), §13 (working assumptions), §14.3 (traceability); GREENFIELD §5, §24; CORE-BRIEF §5, §13; docs 04 §1, 11 §8, 16 §2, 21 §0–1, 23 §5

---

## 0. Backlog governance — binding rules before any item below

**SD-21 — the owner's explicit backlog rule** [DECISION owner 2026-08-03 / Amendment §8 + brief SD-21; doc 02 owns the register entry]:

> **PB-014 (الإنفوجرافيك الشهري) and the violation-suspicion cluster PB-006…PB-010 are committed to the earliest releases** — they are bound to slices VS-02 and VS-05 (doc 25) and need no further scheduling consultation, only the normal per-slice stop-gate acceptance. **EVERY other item in this backlog — all Wave B and Wave C items, and any Wave A item not yet bound to an owner-approved slice — requires explicit owner consultation and approval before it enters any implementation slice.** The backlog is a **standing consultation list, not a work order.**

Operational consequences (all binding):

1. **No item self-schedules.** Presence in this document authorizes *consultation*, never implementation. An implementation agent that starts work on a PB item whose card is not `approved-for-slice` (§7 lifecycle) violates the doc 25 §4.4 prohibitions and the slice stop rule (SD-18).
2. **The committed-early set is closed:** PB-014 → VS-05; PB-006, PB-007, PB-008, PB-009 → VS-02; PB-010 → UI in VS-02, learning-loop consumption in VS-04. Nothing is added to this set except by a new owner decision recorded in doc 02.
3. **Consultation happens at slice stop gates.** The natural consultation moment is step 5 of the mandatory stop rule (doc 25 §2): when the owner approves transition out of slice VS-n, the candidate contents of VS-n+1 are put in front of the owner as PB cards with current scores (§6). Ad-hoc consultations between gates are allowed; skipping the gate is not.
4. **Consultation is recorded, never implied.** Each approval produces a dated entry in the runtime `PRODUCT_BACKLOG.md` (seeded from this document at T-VS00-1, doc 25 §7) naming the item, the slice it enters, scope trims agreed, and the owner's identity. Silence is refusal.
5. **Deprioritization is visible.** An item the owner rejects or defers stays in the backlog with state `deferred`/`rejected` and the reason — items are never silently deleted (append-only discipline, same doctrine as review events).
6. **No invariant trade-offs via backlog pressure.** No consultation outcome may weaken I1–I18, the R-gates, or the suspected≠approved rule [DECISION Amendment §0.5; resolved-conflict register in doc 02]. A card whose acceptance criteria conflict with an invariant is a defective card and must be corrected here first.

## 1. Release waves — the amendment's P0/P1/P2 mapped once, then never re-used

The Owner Amendment §8 prioritizes backlog items as «P0 / P1 / P2». The planning package already uses **P0…P7 as doc 21 phase names**, and those names remain phase names. To avoid a live collision, the amendment's backlog priorities are renamed to **release waves** — stated here once, cited everywhere else by wave name only [DECISION owner 2026-08-03 / Amendment brief, pre-assigned IDs]:

| Amendment §8 priority | This package's name | Meaning | Items |
|---|---|---|---|
| P0 (أول منتج قابل للاستخدام) | **Wave A** | Necessary for the first usable, testable product | PB-001…PB-018 |
| P1 (تحسين الخدمة وتمكين الفريق) | **Wave B** | High operational value after the core is stable | PB-101…PB-120 |
| P2 (الذكاء الوكيلي والتوسع) | **Wave C** | Agentic intelligence and expansion after proven use and accuracy | PB-201…PB-212 |

Rules:

- **Never write "P0/P1/P2" for waves** in any package document or ticket; those tokens are reserved for doc 21 phases. Write Wave A / Wave B / Wave C.
- **Waves are value tiers; slices are execution order.** A wave says *how early the value is wanted*; a slice (VS-00…VS-12, doc 25) says *what is actually built next after owner approval*. Wave A items map onto slices VS-00…VS-07; Wave B items are candidates from VS-08; Wave C items are candidates from VS-09 onward. The mapping per item sits on its card; the binding sequence lives in doc 25.
- A wave assignment is itself consultable: the owner may promote or demote an item's wave at any stop gate; the card is updated with the dated decision.

## 2. Story-card format

Every card below carries exactly these fields [DECISION Amendment §8.1, extended by brief]:

- **Name** — Arabic product label (preserved verbatim) + English name.
- **Primary user** — persona (doc 03) and RBAC role(s) (doc 16 §2: `executive_viewer`, `analyst`, `service_owner`, `reviewer`, `steward`, `admin`, `pipeline_service`).
- **Problem + value** — why the feature exists, in operational terms.
- **Data & capabilities** — the registry IDs it needs: CAP-xx / CAP-OPS-xx (doc 04, same registry discipline as §1 of that document), SRC-* contracts (doc 05, incl. the `SRC-DATAHUB` family), metric IDs (doc 11 §8 + the new ops metrics `review_backlog_age`, `data_completeness`, `source_join_rate`, `action_completion_rate`, `detector_acceptance_rate`, `suspected_cases_open`, `nightly_run_success`).
- **Sensitivity** — doc 16 data classes: P1 aggregate · P2 personal/free-text (names, transcript, evaluation notes) · P3 hard-blocked identifiers; plus the accusation-sensitivity note where suspected violations are involved (suspected ≠ approved everywhere).
- **Dependencies** — PB / VS / EXP / OD identifiers only.
- **Wave · Slice · Flag** — release wave; proposed slice; feature flag. The seven amendment flags are used **verbatim and only these seven exist today**: `session_360_enabled`, `violation_review_enabled`, `nightly_consolidation_enabled`, `monthly_infographic_enabled`, `action_center_enabled`, `lane1_agent_enabled`, `lane3_analysis_enabled` [DECISION Amendment §10.5]. Any other flag on a card is marked **[REC — minted only at consultation]** and does not exist until the owner approves the item into a slice.
- **Owner role** — who owns the delivered feature operationally (doc 03 persona).
- **Consultation status** — `committed-early (SD-21)` or `consult-before-build (SD-21)`.
- **Acceptance criteria** — 3–6 concrete, testable checks expanding the amendment's one-line criterion. These seed the slice acceptance checklists in doc 25 §3 and the golden/structural suites in doc 15.

---

## 3. Wave A — first usable, testable product (PB-001…PB-018)

### PB-001 · «التشغيل الليلي الموحد» — Nightly Consolidation Run
- **Primary user:** Ops / Data Steward (`steward`, `pipeline_service`) · **Owner role:** Operations Engineer.
- **Problem + value:** ends manual daily stitching of Read.ai and DataHub; one authoritative nightly run closes the previous day, pulls all sources, reconciles, enriches, generates review queues, refreshes dashboards, emits the digest [DECISION Amendment §4.1–4.2].
- **Data & capabilities:** CAP-OPS-01 `nightly_consolidation_run`; SRC-READAI + SRC-DATAHUB-SESSION/-BENEFICIARY-EVAL/-CONSULTANT-EVAL/-OUTCOME/-DIRECTORY/-REFERENCE (doc 05); doc 09 DAG; metrics `nightly_run_success`, `data_completeness`, `source_join_rate`.
- **Sensitivity:** P2 in flight (transcript text, evaluation notes); run metadata P1.
- **Dependencies:** PB-003, PB-017; VS-01+VS-02 delivered; OD-28 (assume 02:00 Asia/Riyadh, configurable; hourly Read.ai incremental allowed — nightly is authoritative), OD-07 (DataHub access mechanism).
- **Wave · Slice · Flag:** Wave A · VS-03 · `nightly_consolidation_enabled`.
- **Consultation status:** consult-before-build (SD-21) — enters VS-03 only on owner approval at the VS-02 stop gate.
- **Acceptance criteria:**
  1. Fixture run over 20 Read.ai sessions + 20 DataHub records produces the correct Session-360 count with zero duplicates (Amendment §4.6).
  2. Re-running the same window twice changes no counts and creates no duplicate review cases (idempotency; watermark per source adapter).
  3. Failure of one sub-feed (e.g. SRC-DATAHUB-BENEFICIARY-EVAL) yields run state `partial` — never silent success — while transcript ingest completes; the failed items land in DLQ with typed reasons.
  4. Run lifecycle uses exactly `queued → running → partial → succeeded → failed → cancelled → superseded`; every run records watermarks, in/new/changed/rejected counts, per-step states, classified errors, retry counts, and code/contract/model/prompt/taxonomy versions (run manifest downloadable).
  5. Late-arriving beneficiary evaluation updates the affected Session 360 on the next run without rebuilding unaffected sessions.
  6. Resumability lives in Postgres — no local JSON files or process memory hold pipeline state (Amendment §4.4-9; F13 lesson).

### PB-002 · «مركز تشغيل البيانات» — Data Operations Center
- **Primary user:** Ops / Product Owner (`steward`, `admin`) · **Owner role:** Operations Engineer.
- **Problem + value:** one screen answering "what ran, what failed, what needs a human" — no log spelunking.
- **Data & capabilities:** CAP-OPS-02 `data_ops_center`; SCR-14 (doc 18); reads `ingest` run ledger, watermarks, DLQ; metrics `nightly_run_success`, `data_completeness`, `source_join_rate`, `suspected_cases_open`, `review_backlog_age`.
- **Sensitivity:** P1 (operational metadata); session identifiers visible to steward roles only.
- **Dependencies:** PB-001; PB-012; VS-03.
- **Wave · Slice · Flag:** Wave A · VS-03 · `nightly_consolidation_enabled`.
- **Consultation status:** consult-before-build (SD-21).
- **Acceptance criteria:**
  1. Shows last run + state, per-source status with last-success time and watermark, new/changed session counts, per-stage durations — all read from live tables; a CI test proves no hardcoded value renders (Amendment §4.6, §10.4-7).
  2. Lists unmatched sessions, sessions without transcript, new suspicion count, overdue review count, current-month completeness % (the PB-014 gate figure).
  3. Operator actions — retry failed items, open a DQ issue, view logs, reprocess a selected session, download run manifest — exist, are RBAC-gated, and each writes an audit event.
  4. `rerun failed items` and `reprocess selected sessions` are first-class commands (Amendment §4.4-8) and idempotent.

### PB-003 · «موصلات DataHub من الدرجة الأولى» — DataHub first-class connectors
- **Primary user:** Data Engineer (`pipeline_service`, `steward`) · **Owner role:** Data Steward.
- **Problem + value:** DataHub becomes a named institutional source family with contracts and a field-authority matrix, not scattered generic feeds [DECISION Amendment §3.1].
- **Data & capabilities:** `SRC-DATAHUB` family: SRC-DATAHUB-SESSION / -BENEFICIARY-EVAL / -CONSULTANT-EVAL / -OUTCOME / -DIRECTORY / -REFERENCE; doc 05 field-authority matrix; crosswalk on `advisory_session_id`; existing SRC-INT/SRC-EVAL/SRC-OUT/SRC-DIR/SRC-REF are restructured as views of this family (supersession stated in doc 05 — one framing only).
- **Sensitivity:** P2 (evaluation free text, directory identity); beneficiary PII never widens (I15).
- **Dependencies:** OD-07 (extended: sub-feed contracts + daily batch/API assumption); VS-01.
- **Wave · Slice · Flag:** Wave A · VS-01 · surfaced via `session_360_enabled`; pull path later governed by `nightly_consolidation_enabled`.
- **Consultation status:** consult-before-build (SD-21) — VS-01 scope confirmed at the VS-00 stop gate.
- **Acceptance criteria:**
  1. Each sub-feed has an independent contract, watermark, and DLQ; failure of one does not block the others (run goes `partial`).
  2. Field-authority matrix enforced: DataHub governs session status/date/programme; provider governs transcript; disagreement creates a DQ issue, never a silent overwrite (Amendment §3.2).
  3. Schema drift or unknown enum values fail loudly with a typed `SCHEMA_DRIFT` DLQ class — never positional guessing.
  4. Every internal update carries `source_record_version`, `observed_at`, and `effective_at` where available; late-arriving data is an update path, not a rebuild.
  5. Excel upload functions only as a flagged temporary fallback and stamps its provenance on every row it lands [DECISION Amendment §12-05].

### PB-004 · «الجلسة 360» — Session 360
- **Primary user:** فريق الاستشارات (`service_owner`, `analyst`, `reviewer` scoped) · **Owner role:** Service Owner.
- **Problem + value:** transcript, service data, and both evaluations in one place with per-field provenance — the platform's first daily-use surface.
- **Data & capabilities:** CAP-OPS-03 `session_360`; SCR-15; SRC-DATAHUB-SESSION/-BENEFICIARY-EVAL/-CONSULTANT-EVAL + active transcript (`transcript` schema); doc 17 amendment endpoint `GET /sessions/{id}/360`.
- **Sensitivity:** P2 (transcript text + evaluation notes); consultant names per role scope [ASSUME OD-31]; beneficiary pseudonyms only (م-/ج-/ف- convention).
- **Dependencies:** PB-003, PB-017; VS-01; EXP-01 discipline for identity joins.
- **Wave · Slice · Flag:** Wave A · VS-01 · `session_360_enabled`.
- **Consultation status:** consult-before-build (SD-21).
- **Acceptance criteria:**
  1. One session renders both sources with a per-field source badge that matches the doc 05 authority matrix; absent data renders as «غير متوفر» with reason — never a silent zero or empty box.
  2. Session Crosswalk visible: provider meeting ID ↔ `advisory_session_id` ↔ internal session ID, with resolution status; unmatched sessions appear in a named list with reasons.
  3. Re-ingesting the fixture changes no counts (idempotency proven at the UI-visible level).
  4. A late-arriving evaluation appears after the next run without manual repair.
  5. Evaluations and notes render only for permitted roles (authorization test per doc 16 §2.5 matrix); every transcript open is access-audited.

### PB-005 · «بحث الجلسات وتصفيتها» — Session search and filtering
- **Primary user:** فريق الاستشارات (`analyst`, `service_owner`) · **Owner role:** Service Owner.
- **Problem + value:** fast retrieval of cases without asking an agent; the navigation backbone for every other surface.
- **Data & capabilities:** CAP-OPS-04 `session_search`; SCR-16; `core` dimensions + rating facts.
- **Sensitivity:** P1–P2 (metadata search; consultant-scoped filters restricted).
- **Dependencies:** PB-004; VS-01.
- **Wave · Slice · Flag:** Wave A · VS-01 · `session_360_enabled`.
- **Consultation status:** consult-before-build (SD-21).
- **Acceptance criteria:**
  1. Filters: الفترة، البرنامج/النافذة، المستشار (per RBAC), حالة الجلسة، التقييم — combinable, end-exclusive periods (I4 discipline).
  2. Results are RBAC-scoped server-side; a role without consultant-name scope gets masked names in both list and export.
  3. Stable sort + pagination; every result opens its Session 360.
  4. Consultant-scoped searches write an audit event (anti-surveillance control, doc 16).

### PB-006 · «اشتباه مخالفة» — Violation-suspicion queue
- **Primary user:** Reviewer (`reviewer`) · **Owner role:** Review Lead.
- **Problem + value:** all automatic indicators in one governed queue instead of scattered files; the fixed disclaimer keeps the legal meaning honest: «الحالات في هذه الصفحة مؤشرات آلية تحتاج مراجعة بشرية، ولا تعد مخالفة مثبتة قبل اعتمادها.» [DECISION owner 2026-08-03 / Amendment §5.1; SD-20].
- **Data & capabilities:** CAP-OPS-05 `violation_review_queue`; **SCR-08 upgraded** (display name «اشتباه مخالفة»; no new SCR minted); `findings.violation_finding` / `review_case` / `review_event`.
- **Sensitivity:** highest accusation sensitivity — suspected ≠ approved everywhere; reviewer-only visibility; consultant names in-queue per role.
- **Dependencies:** PB-004, PB-008, PB-017; VS-02; OD-30 (SLA assume 7d normal / 2d high-priority).
- **Wave · Slice · Flag:** Wave A · VS-02 · `violation_review_enabled`.
- **Consultation status:** **committed-early (SD-21)**.
- **Acceptance criteria:**
  1. Each row shows: نوع الاشتباه، درجة الأولوية، ثقة الكاشف، البرنامج/النافذة، المستشار (per RBAC)، تاريخ الجلسة، اقتباس قصير مطابق حرفياً، عمر الحالة، حالة المراجعة، هل تحتاج مراجعة ثانية، إصدار الكاشف والتصنيف (Amendment §5.2).
  2. Filters work for: الحالة، النوع، الشدة، البرنامج، المستشار، الفترة، الثقة، المتأخرة عن SLA [ASSUME OD-30]، والحالات التي أعاد النظام فتحها بسبب تغير النص أو الكاشف.
  3. The fixed disclaimer renders on the queue and every case, verbatim.
  4. An approved case never appears under «مشتبه» after refresh; leadership numbers count approved-only, with queue size available as a separate figure (Amendment §5.7, §13).
  5. Unauthorized roles cannot list, read, or infer queue contents (deny-by-default probe test).

### PB-007 · «حالة مخالفة تفصيلية» — Violation case detail
- **Primary user:** Reviewer (`reviewer`) · **Owner role:** Review Lead.
- **Problem + value:** decisions grounded in context and evidence, not a bare quote.
- **Data & capabilities:** CAP-OPS-05 (detail view of SCR-08); `fetch_transcript_window` mechanics; DataHub context block via PB-004.
- **Sensitivity:** as PB-006; evaluation notes shown only where permitted.
- **Dependencies:** PB-004, PB-006; VS-02.
- **Wave · Slice · Flag:** Wave A · VS-02 · `violation_review_enabled`.
- **Consultation status:** **committed-early (SD-21)**.
- **Acceptance criteria (expanding Amendment §5.3):**
  1. Shows the indicator type with its official definition, the verbatim quote with **at least ±5 لفات** context, speaker + role + timing, and a jump-to-position link into the full transcript.
  2. Shows the detection basis: rule/model/prompt/threshold + detector and taxonomy versions.
  3. Similar **approved** cases are reachable behind an explicit click *after* the definition — anti-anchoring ordering (Amendment §5.3-7, §6.7-4).
  4. Append-only decision log and the fingerprint of exactly what the reviewer saw are displayed; a change of transcript or extraction after a decision renders a clear stale warning and re-queues per fingerprint rules.
  5. DataHub context (البرنامج، الحالة، التقييمات، الملاحظات المسموح عرضها) renders per RBAC.

### PB-008 · «مراجعة Append-only» — Append-only review decisions
- **Primary user:** Reviewer / Compliance (`reviewer`, `admin` read) · **Owner role:** Review Lead.
- **Problem + value:** decision history that cannot be lost or rewritten — the audit substance of the whole review product.
- **Data & capabilities:** `review_event` append-only (doc 08 amendment entities); six decisions with closed reason codes.
- **Sensitivity:** accusation-grade; append-only at DB level (REVOKE UPDATE/DELETE).
- **Dependencies:** PB-006/PB-007; VS-02; OD-11 doctrine (append-only, admin-only retire).
- **Wave · Slice · Flag:** Wave A · VS-02 · `violation_review_enabled`.
- **Consultation status:** **committed-early (SD-21)**.
- **Acceptance criteria:**
  1. Exactly six decisions exist: **مخالفة صحيحة / ليست مخالفة / إعادة تصنيف / تحتاج مراجعة ثانية / أدلة غير كافية / إحالة لمالك السياسة** — each with a closed reason code, optional-or-mandatory note per decision type, reviewer identity, timestamp, and text/detector/taxonomy version stamps (Amendment §5.4).
  2. UPDATE and DELETE on decision events fail at the database level (structural test); a second decision appends and supersedes visibly, never replaces.
  3. إعادة تصنيف preserves the original type in history and re-stamps the case with the new category + taxonomy version.
  4. A decision on finding A in a session never mutates finding B in the same session (`violation_finding` / `review_case` / `review_event` separation; one accepted + one rejected in the same session is a fixture test).

### PB-009 · VIOL-008 end-to-end — أول نوع اشتباه كامل الرحلة
- **Primary user:** Reviewer (`reviewer`) · **Owner role:** AI/Model Owner + Review Lead jointly.
- **Problem + value:** proves the whole journey ingest → detect → queue → review → label → metrics on one clear, verifiable type: **VIOL-008 «التواصل خارج الإطار الرسمي»** [ASSUME OD-35 — owner may swap the first type before the VS-02 build].
- **Data & capabilities:** VIOL taxonomy seed (doc 04 §7); deterministic rules + LLM verifier (doc 14 detector split); CAP-B4/B5 alignment.
- **Sensitivity:** as PB-006.
- **Dependencies:** PB-006/007/008; VS-02; DS-04 labelling discipline (doc 15).
- **Wave · Slice · Flag:** Wave A · VS-02 · `violation_review_enabled`.
- **Consultation status:** **committed-early (SD-21)**.
- **Acceptance criteria:**
  1. Detector emits findings carrying quote + لفة index + speaker + confidence + detector version + taxonomy version (I13 septet discipline).
  2. Quote passes the R7 verbatim gate against the active transcript on every rendered surface.
  3. Measured precision on the labelled fixture set meets the QT-05 bar (≥0.90 per category) before any L2 exposure; L1 preview allowed earlier on synthetic data (SD-18).
  4. Only VIOL-008 is enabled — one type at a time; enabling a second type is a new consultation (SD-20).
  5. Consultant names and consultant ratings are structurally absent from detector features (SD-22; CI test on the feature builder).

### PB-010 · «إضافة اشتباه لم يرصده النظام» — Missed-violation report
- **Primary user:** Reviewer (`reviewer`) · **Owner role:** Review Lead.
- **Problem + value:** without it the system measures only what it caught — false negatives stay invisible (Amendment §6.3).
- **Data & capabilities:** CAP-OPS-06 `missed_violation_report`; SCR-17; button inside Session 360 and the full-transcript view; feeds `label_dataset` (PB-011) and EXP-11.
- **Sensitivity:** as PB-006; manual origin flagged.
- **Dependencies:** PB-004, PB-006; VS-02 (UI) + VS-04 (learning consumption).
- **Wave · Slice · Flag:** Wave A · VS-02/VS-04 · `violation_review_enabled`.
- **Consultation status:** **committed-early (SD-21)**.
- **Acceptance criteria:**
  1. Reviewer selects a transcript span, chooses a type or «نوع جديد», adds a reason, and submits — creating a finding + case flagged `manual_origin` and routed to **مراجعة ثانية** automatically.
  2. The created case is visibly distinct from detector cases in the queue and carries no detector confidence.
  3. The label origin is recorded so learning datasets separate reviewer-reported positives from detector positives (SD-22).
  4. Reviewer-only + audited; «نوع جديد» routes to the taxonomy proposal flow (ADR-0012), never creates an ad-hoc category.

### PB-011 · «حلقة تحسين الكاشف» — Detector learning pipeline
- **Primary user:** ML/AI Owner (`admin`/model-owner role) · **Owner role:** AI/Model Owner.
- **Problem + value:** the detector improves through governed releases, never online self-learning [DECISION SD-22 / Amendment §6].
- **Data & capabilities:** CAP-OPS-07 `detector_learning_pipeline`; SCR-18; entities `label_dataset`, `detector_candidate`, `detector_release`, `shadow_result` (doc 08 amendment); EXP-11 recall estimation; metric `detector_acceptance_rate`.
- **Sensitivity:** P2 (labelled quotes); datasets in the restricted eval store.
- **Dependencies:** PB-008 (labels), PB-010 (missed cases), EXP-11, OD-32 (sample size), OD-33 (release cadence: monthly or gate-pass, whichever less frequent); VS-04.
- **Wave · Slice · Flag:** Wave A · VS-04 · `violation_review_enabled` (learning surfaces live under the review product; release promotion is workflow-gated, never flag-gated).
- **Consultation status:** consult-before-build (SD-21) — enters VS-04 on approval at the VS-03 stop gate.
- **Acceptance criteria:**
  1. Datasets are versioned and immutable (hash-addressed) with train/validation/holdout splits and **no session leakage across splits** (structural check).
  2. A candidate declares exactly what changed (prompt / model / rules / thresholds / taxonomy) and records Dataset version + Prompt SHA + Model ID + thresholds + taxonomy version (Amendment §6.7-7).
  3. Offline evaluation reports **per-category precision AND estimated recall** (via EXP-11 sampling), plus regression status on previously stable categories — never a single aggregate number.
  4. Shadow runs never touch the production queue; promotion requires a documented human approval; canary precedes full deploy; rollback restores the prior release without losing any review decision.
  5. No reviewer click changes production behaviour directly — the only path is dataset → candidate → eval → shadow → adjudication → promotion (structural test on the config write path).

### PB-012 · «مشكلات الربط والبيانات» — Data-quality / reconciliation queue
- **Primary user:** Data Steward (`steward`) · **Owner role:** Data Steward.
- **Problem + value:** unmatched, conflicting, duplicate, and late records get a workflow, not a spreadsheet.
- **Data & capabilities:** `reconciliation_case` + `data_quality_issue` entities; SCR-14 adjunct queue; metric `source_join_rate`.
- **Sensitivity:** P1–P2 (identifiers for stewards only).
- **Dependencies:** PB-003; VS-03 (queue live), seeds from VS-01.
- **Wave · Slice · Flag:** Wave A · VS-03 · `nightly_consolidation_enabled`.
- **Consultation status:** consult-before-build (SD-21).
- **Acceptance criteria:**
  1. Every unmatched/conflicting/duplicate/late item appears with a typed reason and suggested action.
  2. Steward actions (link, exclude-with-reason, retry) update the crosswalk with append-only history and audit events.
  3. Join-rate and completeness figures on SCR-14 derive from these tables live (no hardcoded values).
  4. Resolving a case triggers re-processing of only the affected sessions.

### PB-013 · «اللوحة التشغيلية الأولية» — Initial operations dashboard
- **Primary user:** Service Owner (`service_owner`) · **Owner role:** Service Owner.
- **Problem + value:** daily volume/quality monitoring without asking questions; the first governed-numbers surface.
- **Data & capabilities:** CAP-OPS-12 `ops_dashboard`; SCR-13 board; metrics `sessions_count`, `attendance_rate`, `cancellation_rate`, `beneficiary_rating_avg`/`_distribution`, `clear_steps_rate` (when enrichment lands), `suspected_cases_open` (separate figure), `review_backlog_age`, `data_completeness` (doc 11 §8 + ops metrics).
- **Sensitivity:** P1 aggregates; suppression n≥30 + Wilson CI (doc 10).
- **Dependencies:** PB-001 (serving views refreshed nightly); VS-06; deeper drill-down matures with VS-07 (EPIC-19 metric layer).
- **Wave · Slice · Flag:** Wave A · VS-06 · [REC — minted only at consultation] `ops_dashboard_enabled`; data production governed by `nightly_consolidation_enabled`.
- **Consultation status:** consult-before-build (SD-21).
- **Acceptance criteria:**
  1. Every KPI card declares period (end-exclusive), coverage, and source freshness; suppression rules render as «حجب» marks, never blank cells.
  2. Suspected-violation volume renders as its own queue-size figure, visually distinct from approved violations.
  3. Drill-down from any KPI opens the underlying session list within the viewer's permissions (I8/I11).
  4. A CI test proves no hardcoded numbers or demo ratios exist in dashboard code (Amendment §10.4-7).

### PB-014 · «نبض خدمة الاستشارات والإرشاد — ملخص الشهر» — Monthly leadership infographic
- **Primary user:** Leadership (`executive_viewer`) · **Owner role:** Publication Authority (OD-10) with Data Owner + Service Owner reviews.
- **Problem + value:** the month and its decisions on one page; a standalone early product, not a CAP-D10 sub-renderer [DECISION SD-19 / Amendment §7.1].
- **Data & capabilities:** CAP-OPS-08 `monthly_infographic`; SCR-20; entities `monthly_infographic`/`infographic_section`/`publication_event` (or pack-artifact binding, doc 08); five sections + mandatory footer per Amendment §7.4; drawn programmatically from structured data ONLY — no LLM numbers, no image-generation models; optional LLM phrasing passes the R6 gate.
- **Sensitivity:** P1 published aggregates; **approved-violations only**; **no consultant names in the general edition** [ASSUME OD-31].
- **Dependencies:** PB-001 (month close + completeness gate ≥98% [ASSUME Amendment §13]), PB-013 metrics; VS-05; OD-29 (channels: assume Portal + approved Email).
- **Wave · Slice · Flag:** Wave A · VS-05 · `monthly_infographic_enabled`.
- **Consultation status:** **committed-early (SD-21)**.
- **Acceptance criteria (expanding Amendment §7.7):**
  1. Every number traces to a Metric Result and a session set (provenance click-through); zero numbers generated or computed by an LLM — narrative numerals must be ⊆ the envelope's allowed-literals (R6).
  2. Comparison sets and periods are end-exclusive correct; the previous-month deltas reproduce from frozen artifacts.
  3. Unapproved (suspected) violations never enter the published «المخالفات» figure; queue size may appear as its own labelled figure.
  4. Web page, PDF, and PNG carry byte-identical payload numbers, regenerated from the pack artifact without DOM scraping (I11); a repeat render is byte-identical.
  5. Lifecycle enforced: `draft → data_review → content_review → approved → published → superseded/retracted`; a reissue creates a new version and never edits the published one; completeness override prints on the face of the infographic.
  6. Template stays single-page, readable on a leadership screen and A4 print; KPI click opens drill-down within the viewer's permissions.

### PB-015 · «مركز الإجراءات» — Action Center
- **Primary user:** Service Owner (`service_owner`) · **Owner role:** Service Owner [ASSUME OD-34 — service owner owns actions; leadership sees aggregate status/impact].
- **Problem + value:** analysis becomes an action with an owner, due date, status, and later measured effect — reports stop being an archive (Amendment §2.4).
- **Data & capabilities:** CAP-OPS-09 `action_center`; SCR-19; entities `service_improvement_action`/`action_event`/`action_metric_baseline`; metric `action_completion_rate`.
- **Sensitivity:** P1–P2 (actions may reference consultant-scoped findings; RBAC-scoped).
- **Dependencies:** PB-013 (KPIs to act on), PB-014 (recommended-actions section), PB-017; VS-06.
- **Wave · Slice · Flag:** Wave A · VS-06 · `action_center_enabled`.
- **Consultation status:** consult-before-build (SD-21).
- **Acceptance criteria:**
  1. An action is created from a finding/KPI/infographic item with mandatory owner + due date, and keeps the source link.
  2. Status transitions are append-only events with audit; overdue actions surface in SCR-19 and the morning digest.
  3. Post-action metric comparison renders baseline → follow-up **without causal claims** (doc 10 discipline; wording test).
  4. Leadership view shows aggregate `action_completion_rate` and impact summaries only — no bypass of consultant-name masking.

### PB-016 · «صباحيات الخدمة» — Morning digest (folds Amendment §11.1)
- **Primary user:** Service Owner (`service_owner`) · **Owner role:** Service Owner.
- **Problem + value:** "ماذا حدث أمس؟" answered in one message after the nightly run — no dashboard tour needed.
- **Data & capabilities:** CAP-OPS-10 `morning_digest`; SCR-21; nightly run step 15 (doc 09 amendment); `notification_subscription`/`notification_delivery`.
- **Sensitivity:** P1 counts + links; content respects each recipient's RBAC (no leaked names).
- **Dependencies:** PB-001; PB-006 (suspicion counts); VS-03; OD-29 (delivery channels).
- **Wave · Slice · Flag:** Wave A · VS-03 · [REC — minted only at consultation] `morning_digest_enabled`; produced under `nightly_consolidation_enabled`.
- **Consultation status:** consult-before-build (SD-21).
- **Acceptance criteria:**
  1. Digest contains exactly the §11.1 items: الجلسات الجديدة، حالات الاشتباه الجديدة، الحالات المتأخرة في المراجعة، تقييمات 1–2 نجمة الجديدة، جلسات بلا خطوات واضحة، مشكلات DataHub/Read.ai، أهم تنبيه أو انحراف.
  2. Every item deep-links to the appropriate screen (queue, Session 360, SCR-14…).
  3. Digest is emitted only from a `succeeded` or `partial` run and labels `partial` prominently; a `failed` run emits an ops alert instead.
  4. Two recipients with different roles receive correctly scoped variants (masking test).

### PB-017 · Role-based access and masking — الصلاحيات والإخفاء
- **Primary user:** all users · **Owner role:** Security Engineer + Data Steward.
- **Problem + value:** every surface above exposes exactly the permitted minimum; accusation-grade and personal data stay contained (I14, I15).
- **Data & capabilities:** doc 16 §2 role model (7 roles), permission vocabulary, masking rules; OD-09 break-glass; OD-31 consultant-name scope.
- **Sensitivity:** the control itself; P3 identifiers remain structurally blocked.
- **Dependencies:** none (precondition for everything); VS-00 onward, every slice adds its matrix rows.
- **Wave · Slice · Flag:** Wave A · VS-00 (baseline) then every slice · **no flag — security is never behind a feature flag**.
- **Consultation status:** consult-before-build (SD-21) formally, but structurally mandatory — descoping it is not a consultable option, only its surface-by-surface matrix rows are.
- **Acceptance criteria:**
  1. G-SEC-1 route walk: every route authenticated except the two probes; deny-by-default proven by planted-route test.
  2. Authorization-matrix tests exist per persona × surface (queue, Session 360, dashboards, infographic, actions, exports) and run in CI from VS-00.
  3. Consultant names masked outside the authorized management scope on lists, details, exports, and digests alike [ASSUME OD-31].
  4. Suspected-violation surfaces are invisible to roles without review permissions — including via search, export, and audit-view side channels.

### PB-018 · Audit and provenance explorer — مستكشف التدقيق والأصل
- **Primary user:** Owner / Auditor (`admin`, `executive_viewer` scoped) · **Owner role:** Product Owner.
- **Problem + value:** every number and decision explains itself: from figure → metric/run/sessions/model/taxonomy; from decision → full event chain (I13, R13).
- **Data & capabilities:** `ops.audit_event`, run manifests, `detector_release` stamps, metric result provenance; UI over doc 17 endpoint 25.
- **Sensitivity:** P1 metadata; underlying payloads gated by the viewer's own permissions.
- **Dependencies:** PB-001, PB-008, PB-011, PB-014; audit spine from VS-00 (T-VS00-2), explorer UI proposed at VS-06+.
- **Wave · Slice · Flag:** Wave A · spine VS-00, UI VS-06+ · [REC — minted only at consultation] `provenance_explorer_enabled`.
- **Consultation status:** consult-before-build (SD-21).
- **Acceptance criteria:**
  1. From any published KPI: metric spec + run id + corpus snapshot + model/prompt/taxonomy versions reachable in ≤3 clicks.
  2. From any review decision: the complete append-only chain with fingerprints and version stamps.
  3. From any infographic figure: the same trace as (1) plus the publication event chain.
  4. Audit-view access is itself audited; the explorer is read-only by construction.

---

## 4. Wave B — service improvement and team enablement (PB-101…PB-120)

All Wave B items are **consult-before-build (SD-21)**; proposed entry window is VS-08 onward, each subject to owner approval at a stop gate. Cards are full but compact; acceptance criteria are the seed of the eventual slice checklist.

### PB-101 · «قائمة التعافي الخدمي» — Service Recovery Queue (folds Amendment §11.2)
- **Primary user:** Service Owner (`service_owner`) · **Owner role:** Service Owner · **Sensitivity:** P2 (session-level, non-accusatory).
- **Problem + value:** fast operational intervention on low ratings, explicit negative feedback, no-clear-steps sessions, high end-of-session pressure/confusion, repeated cancellations, or promised-but-missing follow-up — **strictly separate from the violations queue** (these are not violations).
- **Data & capabilities:** CAP-OPS-11 `service_recovery_queue`; SCR-22; rating/eval facts + findings signals; governed trigger rules.
- **Dependencies:** PB-004, PB-013; proposed VS-08.
- **Wave · Slice · Flag:** Wave B · VS-08 (proposed) · [REC] `service_recovery_enabled`.
- **Acceptance criteria:** (1) trigger rules are registered and versioned, not hardcoded; (2) each case carries follow-up state and documented closure; (3) zero visual or data overlap with «اشتباه مخالفة» (separate queue, separate wording, structural test); (4) closure requires an outcome note; re-opened cases keep history.

### PB-102 · Consultant 360 — «الملف الشامل للمستشار»
- **Primary user:** Manager (`service_owner` manager scope) · **Owner role:** Consultants' Manager · **Sensitivity:** P2 consultant-named — the most fairness-sensitive surface.
- **Problem + value:** coaching and improvement support, never automatic punishment (Amendment §8.3; doc 10 fairness rules).
- **Data & capabilities:** CAP-D5 `consultant_360`; approved findings only; coverage + n≥30 + ≥5-consultant confound rules.
- **Dependencies:** PB-017 (OD-31 scope), CAP-D5 method (doc 10); proposed VS-08 (limited version).
- **Wave · Slice · Flag:** Wave B · VS-08 (proposed) · [REC] `consultant_360_enabled`.
- **Acceptance criteria:** (1) trends render only above support thresholds with Wilson CIs; (2) examples are review-approved findings with verbatim quotes; (3) comparison views are normalized and coverage-labelled — no league tables by raw counts; (4) access limited to the direct-management RBAC scope and audited [ASSUME OD-31/OD-14].

### PB-103 · Coaching Cards — «بطاقات التدريب والمتابعة» (folds Amendment §11.5)
- **Primary user:** Manager / Consultant · **Owner role:** Consultants' Manager · **Sensitivity:** P2 consultant-named, non-punitive by design.
- **Problem + value:** a concrete behaviour, approved examples, a proposed action, a follow-up date, and the metric change after action — instead of accusatory rankings.
- **Data & capabilities:** approved findings + CAP-D5 inputs + `service_improvement_action` links.
- **Dependencies:** PB-102, PB-015; proposed VS-08+.
- **Wave · Slice · Flag:** Wave B · VS-08+ (proposed) · [REC] `coaching_cards_enabled`.
- **Acceptance criteria:** (1) every card names one behaviour + ≥1 approved example; (2) card carries follow-up date and later metric delta without causal claims; (3) no automatic punitive ordering of consultants (structural: no rank field); (4) card visibility follows OD-31 scope.

### PB-104 · Best-practice sessions — «مكتبة الممارسات المثلى» (folds Amendment §11.8)
- **Primary user:** Quality Team (`analyst`, `service_owner`) · **Owner role:** Quality Lead · **Sensitivity:** P2 quotes; beneficiary data masked.
- **Problem + value:** spread high-impact practice with evidence: how the problem was diagnosed, how it became steps, how the session closed, what the impact evidence is.
- **Data & capabilities:** rubric-based selection over approved findings (CAP-A1 inputs); library entity with approval workflow (ADR-0012).
- **Dependencies:** PB-004; review workflow; proposed VS-08+.
- **Wave · Slice · Flag:** Wave B · VS-08+ (proposed) · [REC] `best_practice_library_enabled`.
- **Acceptance criteria:** (1) entry requires rubric scores + authority approval; (2) beneficiary identity masked in every excerpt; (3) each entry links diagnosis→steps→closure→impact evidence quotes (R7-verified); (4) consultant consent/visibility follows OD-31 policy.

### PB-105 · «الأسئلة المتكررة المرشحة للأتمتة» — FAQ / automation candidates (folds §11.4 first kind)
- **Primary user:** Knowledge Team · **Owner role:** Knowledge Lead · **Sensitivity:** P1 clusters + P2 example quotes.
- **Problem + value:** recurrent questions with consistent answers become FAQ/fixed content; reduces load.
- **Data & capabilities:** CAP-B3 `repeated_questions_top` + CAP-D8 `kb_automation_candidates`; QST clusters (R-P2, approved labels only).
- **Dependencies:** EPIC-18 clustering; proposed VS-08.
- **Wave · Slice · Flag:** Wave B · VS-08 (proposed) · [REC] `kb_candidates_enabled`.
- **Acceptance criteria:** (1) candidates ranked by semantic recurrence + answer-consistency measure, both defined in doc 10; (2) every candidate carries example sessions + quotes; (3) promotion to FAQ is an owner workflow, not automatic; (4) declining answer-dispersion after publication is tracked as the value metric.

### PB-106 · «مواضيع الإجابات غير المتسقة» — Inconsistent-answer topics (folds §11.4 second kind)
- **Primary user:** Service Owner · **Owner role:** Service Owner · **Sensitivity:** P2 (consultant-adjacent; framed as knowledge gap, not fault).
- **Problem + value:** recurrent questions with divergent answers signal knowledge-unification or specialist-training needs; separates person-variance from policy gaps.
- **Data & capabilities:** CAP-C6 `inconsistent_answers`; contrasting quote pairs; QST/CHAL clusters.
- **Dependencies:** EPIC-18; proposed VS-08.
- **Wave · Slice · Flag:** Wave B · VS-08 (proposed) · [REC] `inconsistent_answers_enabled`.
- **Acceptance criteria:** (1) each topic shows contradictory quotes with full identifiers; (2) the person-difference vs policy-gap split is explicit per topic; (3) topics link to a recommended action type (توحيد معرفة / تدريب متخصص); (4) no consultant blame framing in copy (wording review).

### PB-107 · «تحديات واتجاهات صاعدة» — Rising challenges radar (folds Amendment §11.3)
- **Primary user:** Leadership / Product · **Owner role:** Service Owner · **Sensitivity:** P1 aggregates.
- **Problem + value:** see what is changing before it becomes a crisis.
- **Data & capabilities:** CAP-C3 + CAP-C1 over CHAL clusters; radar fields: الموضوع، معدل الظهور لكل 100 جلسة، التغير عن الشهر السابق، البرامج/القطاعات المرتبطة، أمثلة جلسات، هل التغير حقيقي أم أثر تغيير في التصنيف.
- **Dependencies:** EPIC-18 clustering + taxonomy lineage; proposed VS-08+.
- **Wave · Slice · Flag:** Wave B · VS-08+ (proposed) · [REC] `topic_radar_enabled`.
- **Acceptance criteria:** (1) rates per-100-sessions with support + CI, never raw counts; (2) MoM change computed on end-exclusive windows; (3) taxonomy-change effects flagged via lineage (as-published vs as-current); (4) each rising topic links ≥2 example sessions.

### PB-108 · Voice of Beneficiary — «صوت المستفيد»
- **Primary user:** Service Owner · **Owner role:** Service Owner · **Sensitivity:** P2 (rating comments).
- **Problem + value:** merge the direct rating with what the beneficiary actually said, without conflating the two.
- **Data & capabilities:** CAP-A3 + CAP-D1; SRC-DATAHUB-BENEFICIARY-EVAL + satisfaction findings.
- **Dependencies:** enrichment (VS-07+ data); proposed VS-08.
- **Wave · Slice · Flag:** Wave B · VS-08 (proposed) · [REC] `voice_of_beneficiary_enabled`.
- **Acceptance criteria:** (1) direct rating and text-inferred signals render as separate labelled tracks; (2) coverage (rated share, matched share) on the face; (3) rating is never inferred from text where the rating is absent (structural rule from doc 05 authority matrix); (4) drill-down to quotes passes R7.

### PB-109 · «تقييم المستشار مقابل المستفيد» — Evaluation-contradiction analysis (folds Amendment §11.6)
- **Primary user:** Manager · **Owner role:** Service Owner · **Sensitivity:** P2 consultant-named — needs-understanding framing.
- **Problem + value:** detect perception gaps (high rating + no clear steps; low rating + strong impact signals; very positive consultant self-eval + negative beneficiary note). The result opens a **«تحتاج فهمًا»** case and never declares either rating wrong.
- **Data & capabilities:** CAP-D2 + `eval_gap_avg`; contradiction rules registered in doc 10.
- **Dependencies:** OD-17 scale semantics; proposed VS-08+.
- **Wave · Slice · Flag:** Wave B · VS-08+ (proposed) · [REC] `eval_contradiction_enabled`.
- **Acceptance criteria:** (1) each contradiction type is a registered rule with version; (2) output cases are labelled «تحتاج فهمًا» and carry both sides' evidence; (3) no automatic escalation to violation or recovery queues; (4) scale normalization caveat prints until OD-17 closes.

### PB-110 · Follow-up and outcome tracking — «متابعة الأثر التشغيلي»
- **Primary user:** Service Owner · **Owner role:** Service Owner · **Sensitivity:** P1–P2.
- **Problem + value:** did the session turn into an action/service? closes the loop with DataHub outcome data.
- **Data & capabilities:** CAP-D7 `followup_completion`; SRC-DATAHUB-OUTCOME; `followup_required_rate`, `followup_completion_rate`.
- **Dependencies:** OD-07 outcome feed availability; proposed VS-08+.
- **Wave · Slice · Flag:** Wave B · VS-08+ (proposed) · [REC] `outcome_tracking_enabled`.
- **Acceptance criteria:** (1) linkage of subsequent sessions/actions with coverage stated; (2) absent outcome feed renders `DATA_NOT_ENRICHED`, never zeros; (3) follow-up promised vs delivered split by programme; (4) no causal claims in copy.

### PB-111 · Program comparison — «مقارنة البرامج والنوافذ»
- **Primary user:** Leadership · **Owner role:** Service Owner · **Sensitivity:** P1.
- **Problem + value:** compare programmes/windows on identical metrics and periods with statistical support.
- **Data & capabilities:** CAP-D6 `program_service_comparison`; semantic layer (EPIC-19).
- **Dependencies:** VS-07 metric layer; proposed VS-08+.
- **Wave · Slice · Flag:** Wave B · VS-08+ (proposed) · [REC] `program_comparison_enabled`.
- **Acceptance criteria:** (1) unified metric definitions + same end-exclusive windows; (2) n≥30 + Wilson CI per cell with «حجب» rendering; (3) «قطاع» always disambiguated three ways (CORE-BRIEF §13.3); (4) export reproduces the on-screen numbers byte-identically (I11).

### PB-112 · Workload and queue balance — «توازن الحمل والطوابير»
- **Primary user:** Operations Manager · **Owner role:** Operations Engineer · **Sensitivity:** P1–P2 (team-level).
- **Problem + value:** monitor load, waiting, and review congestion by team before backlogs become crises.
- **Data & capabilities:** ops metrics `review_backlog_age`, queue depths, session volumes by team; doc 19 SLOs.
- **Dependencies:** PB-006 (review queue data); proposed VS-08+.
- **Wave · Slice · Flag:** Wave B · VS-08+ (proposed) · [REC] `workload_balance_enabled`.
- **Acceptance criteria:** (1) waiting/backlog ages computed from event timestamps, not snapshots; (2) SLA-late items countable per OD-30 assumption; (3) team views respect RBAC; (4) alert thresholds registered, not hardcoded.

### PB-113 · Reviewer quality dashboard — «لوحة جودة المراجعة»
- **Primary user:** Review Lead · **Owner role:** Review Lead · **Sensitivity:** P2 (reviewer-named internally).
- **Problem + value:** consistency of reviewers and definitions: inter-reviewer agreement, decision time, reclassification rate — improves definitions, never punishes silently (Amendment §6.6–6.7).
- **Data & capabilities:** review events + `detector_acceptance_rate`, disagreement/adjudication sets (DS discipline, doc 15); SCR-18 adjunct.
- **Dependencies:** PB-008, PB-011; proposed VS-08+.
- **Wave · Slice · Flag:** Wave B · VS-08+ (proposed) · [REC] `reviewer_quality_enabled`.
- **Acceptance criteria:** (1) agreement metrics computed on the disagreement set with adjudicator outcomes; (2) sensitive cases route to adjudication, not majority vote (§6.7-3); (3) definition shown before decision — anchoring controls verified in UI test; (4) reviewer-level views restricted to the review lead.

### PB-114 · Taxonomy Studio — «استوديو التصنيفات»
- **Primary user:** Domain Owner (`steward`/owner) · **Owner role:** Product Owner · **Sensitivity:** P1 governance data.
- **Problem + value:** approve/merge/split categories with lineage and version bumps in one governed UI (extends SCR-09).
- **Data & capabilities:** `tax` schema mechanics; ADR-0011/0012; proposal→review→approve.
- **Dependencies:** EPIC-16/18; proposed VS-08+.
- **Wave · Slice · Flag:** Wave B · VS-08+ (proposed) · [REC] `taxonomy_studio_enabled`.
- **Acceptance criteria:** (1) every change is a proposal with review trail; (2) merges/splits write lineage edges and bump versions; (3) both-countings (as-published vs as-current) reproducible after a merge; (4) findings re-stamp policy (recompute_required) honoured with visible invalidation.

### PB-115 · KPI drill-down — «التتبع من الرقم إلى الحالات»
- **Primary user:** all analysts · **Owner role:** Product Owner · **Sensitivity:** inherits the viewer's scope.
- **Problem + value:** every KPI opens its session set and evidence — numbers stop being unexplainable.
- **Data & capabilities:** metric-result → session-set contract (I8/I11); evidence refs.
- **Dependencies:** VS-07 envelope; proposed VS-08.
- **Wave · Slice · Flag:** Wave B · VS-08 (proposed) · [REC] `kpi_drilldown_enabled`.
- **Acceptance criteria:** (1) every registered KPI resolves to a DISTINCT-session list matching its denominator; (2) drill-down respects RBAC and audits transcript opens; (3) suppressed cells drill to counts-only views; (4) drill-down numbers equal the KPI (reconciliation test).

### PB-116 · Saved views and subscriptions — «العروض المحفوظة والاشتراكات»
- **Primary user:** Managers · **Owner role:** Product Owner · **Sensitivity:** P1 + delivery policy (doc 16 notification rules).
- **Problem + value:** recurring monitoring saved once, delivered on schedule or event.
- **Data & capabilities:** `notification_subscription`/`notification_delivery`; saved MetricSpec views (closed schemas).
- **Dependencies:** PB-013; OD-29 channels; proposed VS-08+.
- **Wave · Slice · Flag:** Wave B · VS-08+ (proposed) · [REC] `saved_views_enabled`.
- **Acceptance criteria:** (1) a saved view stores a validated spec, never free SQL; (2) delivery honours the viewer's permissions at send time (revocation test); (3) schedule + event triggers deduplicated; (4) unsubscribes and failures observable (`notification_delivery` states).

### PB-117 · Alerts and governed watchlists — «التنبيهات وقوائم المتابعة المحكومة» (folds Amendment §11.7)
- **Primary user:** Service Owner · **Owner role:** Service Owner · **Sensitivity:** P2 when consultant-scoped — governance is the point.
- **Problem + value:** early deviation detection with controls that prevent unbounded permanent surveillance: **every watchlist has a reason, an owner, and an expiry date**.
- **Data & capabilities:** threshold + anomaly rules over registered metrics; dedupe + acknowledgement; watchlist entity with mandatory `reason`, `owner`, `expires_at`.
- **Dependencies:** PB-013 metrics; PB-017 scope rules; proposed VS-08+.
- **Wave · Slice · Flag:** Wave B · VS-08+ (proposed) · [REC] `watchlists_enabled`.
- **Acceptance criteria:** (1) creating a watchlist without reason/owner/expiry is impossible (schema-level); (2) expiry auto-disables and notifies the owner; renewal is a fresh justified act; (3) consultant/team watchlists require the management scope and are audited; (4) alerts deduplicate and require acknowledgement; noisy rules are measurable (alert precision report).

### PB-118 · Weekly operational pulse — «النبض الأسبوعي»
- **Primary user:** Managers · **Owner role:** Service Owner · **Sensitivity:** P1.
- **Problem + value:** lighter weekly rhythm between monthly infographics.
- **Data & capabilities:** subset of PB-013 metrics; auto-issued artifact with drill-down; renderer shared with PB-014 (structured artifact only).
- **Dependencies:** PB-013, PB-014 render path; proposed VS-08.
- **Wave · Slice · Flag:** Wave B · VS-08 (proposed) · [REC] `weekly_pulse_enabled`.
- **Acceptance criteria:** (1) fixed metric set, auto-generated after the week's last nightly run; (2) numbers trace to metric results (R6 discipline); (3) drill-down links live; (4) no publication workflow needed (internal artifact) but provenance footer mandatory.

### PB-119 · Export/share center — «مركز التصدير والمشاركة»
- **Primary user:** authorized users · **Owner role:** Product Owner · **Sensitivity:** inherits payload class; exports audited + expiring links (doc 16).
- **Problem + value:** uniform JSON/XLSX/PDF outputs from structured artifacts — never DOM scraping (I11; extends SCR-12).
- **Data & capabilities:** doc 17 endpoint 24; envelope renderers.
- **Dependencies:** VS-07 envelope; proposed VS-08.
- **Wave · Slice · Flag:** Wave B · VS-08 (proposed) · [REC] `export_center_enabled`.
- **Acceptance criteria:** (1) every export derives from the typed envelope/artifact (byte-equality test with on-screen numbers); (2) exports carry provenance footer + masking per role; (3) links expire per policy and access is audited; (4) failed renders fail loudly (no empty file with 200).

### PB-120 · Improvement impact tracking + leadership decision log — «تتبع أثر التحسين وسجل القرارات» (folds Amendment §11.9)
- **Primary user:** Leadership · **Owner role:** Publication Authority + Service Owner · **Sensitivity:** P1.
- **Problem + value:** link infographic/report → decision taken → owner → date → target metric → re-measurement date → result, and action → later metric movement: reports stop being an archive without effect.
- **Data & capabilities:** `action_metric_baseline` + decision-log entity bound to publication events; `action_completion_rate`.
- **Dependencies:** PB-014, PB-015; proposed VS-08+.
- **Wave · Slice · Flag:** Wave B · VS-08+ (proposed) · [REC] `decision_log_enabled`.
- **Acceptance criteria:** (1) each logged decision links a published artifact + owner + target metric + re-measure date; (2) baseline → intervention → follow-up comparisons render **without unproven causal claims** (wording test); (3) re-measure reminders fire on schedule; (4) decision log is append-only and visible on the next month's infographic «حالة إجراءات الشهر السابق» section.

---

## 5. Wave C — agentic intelligence and expansion (PB-201…PB-212)

All Wave C items are **consult-before-build (SD-21)**; none enters a slice before VS-09, and each requires its gate experiments green for L2+ exposure (doc 25 §1 rules).

### PB-201 · Deterministic governed Q&A — «الإجابات الحتمية المحكومة»
- **Primary user:** Analyst · **Owner role:** Product Owner · **Sensitivity:** P1 aggregates + evidence per role.
- **Problem + value:** fast answers for committed metrics; the VS-07 five-capability seed grows toward the 26-capability catalogue.
- **Data & capabilities:** Lane 0 over CAP-A/B/C/D entries (doc 04); EPIC-19/20 substrate.
- **Dependencies:** VS-07 delivered; EXP-05/EXP-06 for expansion tranches; proposed VS-09/VS-11.
- **Wave · Slice · Flag:** Wave C · VS-09/VS-11 (proposed) · [REC] `governed_qa_enabled` (Lane-0 surface; the seven amendment flags untouched).
- **Acceptance criteria:** (1) 5–7 capabilities first, each route-correct on its paraphrase set (EXP-05 floor ≥0.95, target ≥0.99); (2) numbers verified by R6 with allowed-literals; (3) abstention → «الاستيضاح المغلق», never a neighbour answer (I5); (4) every answer carries period + coverage + provenance.

### PB-202 · Evidence Explorer — «مستكشف الأدلة»
- **Primary user:** Analyst / Reviewer · **Owner role:** Product Owner · **Sensitivity:** P2 quotes; access audited.
- **Problem + value:** search quotes and contexts without touching any number.
- **Data & capabilities:** `evidence` schema + retrieval backends (EPIC-21/22); R7 gate; RAG stays optional and non-numeric (I9).
- **Dependencies:** EXP-07; proposed VS-09+.
- **Wave · Slice · Flag:** Wave C · VS-09+ (proposed) · [REC] `evidence_explorer_enabled`.
- **Acceptance criteria:** (1) every result passes verbatim R7 verification with full identifiers; (2) filter-first retrieval respects RBAC before ranking; (3) recall measured per backend before default status (QT-09); (4) zero influence on metric values (structural separation test).

### PB-203 · Bounded Lane-1 Agent — «الوكيل المقيد»
- **Primary user:** Analyst · **Owner role:** AI/Model Owner · **Sensitivity:** P1–P2 via tools; prompt-injection isolation (R15).
- **Problem + value:** composite questions over governed tools — never over a raw spreadsheet.
- **Data & capabilities:** toolbelt (CORE-BRIEF §4); budgets in Postgres; EPIC-23.
- **Dependencies:** VS-07/VS-09; EXP-05/09/10 for exposure; proposed VS-09.
- **Wave · Slice · Flag:** Wave C · VS-09 · `lane1_agent_enabled`.
- **Acceptance criteria:** (1) tool-only: no SQL, no model arithmetic, no schema names in model space (I2/I3); (2) zero wrong route on the golden set; (3) ≤6 steps / ≤8 calls budget enforced server-side; (4) injection fixtures (DS-11) alter nothing; (5) L1 demo allowed on synthetic; L2 waits for EXP-09/EXP-10 (SD-18).

### PB-204 · Lane-2 Clarification — «الاستيضاح المغلق»
- **Primary user:** Analyst · **Owner role:** Product Owner · **Sensitivity:** P1.
- **Problem + value:** prevents guessing under ambiguity; honest refusal with closed reason codes.
- **Data & capabilities:** ≤4 closed options from the registry; reason-code enum (CORE-BRIEF §4); resolved-facts store.
- **Dependencies:** PB-201/PB-203; proposed VS-09.
- **Wave · Slice · Flag:** Wave C · VS-09 · `lane1_agent_enabled` (same serving surface).
- **Acceptance criteria:** (1) options come only from registry content; (2) context preserved across the clarification round-trip; (3) refusal uses the closed enum with Arabic fixed strings; (4) clarification rate monitored (excess triggers matcher re-calibration, not looser routing).

### PB-205 · Lane-3 Deep Analysis — «مهمة تحليل معمّق»
- **Primary user:** Analyst · **Owner role:** AI/Model Owner · **Sensitivity:** P2 (raw transcripts in the job plane, pseudonymized outbound).
- **Problem + value:** answer a new question from raw data with verified findings and code-computed reduction.
- **Data & capabilities:** `jobs` machinery (EPIC-28/29/30); map→verify→reduce; no cost ceiling, bounded concurrency (SD-05).
- **Dependencies:** EXP-08; proposed VS-10 (one real question first).
- **Wave · Slice · Flag:** Wave C · VS-10 · `lane3_analysis_enabled`.
- **Acceptance criteria:** (1) resumable month-partitioned job with progress + job id; (2) failed partition = INCOMPLETE, never silent; (3) findings pass deterministic verification before reduce; reduce is code, never model aggregation (I3); (4) result cacheable + promotable via PB-208.

### PB-206 · Custom RAG lifecycle — «دورة حياة الفهارس المخصصة»
- **Primary user:** Analyst / AI Owner · **Owner role:** AI/Model Owner · **Sensitivity:** P2 corpus-bound; residency: embeddings local.
- **Problem + value:** recurring question → reusable governed artifact — build/validate/version/expire/promote, not an ungoverned temp index.
- **Data & capabilities:** doc 13 custom-collection contract; TTL/expiry/owner mandatory.
- **Dependencies:** PB-205; proposed VS-10+.
- **Wave · Slice · Flag:** Wave C · VS-10+ (proposed) · `lane3_analysis_enabled` (rides the Lane-3 plane).
- **Acceptance criteria:** (1) every collection has owner + TTL + validation report; (2) expired collections stop serving automatically; (3) versioned rebuilds with embedding-model provenance; (4) never a number source (I9 structural test).

### PB-207 · Proactive Insight Agent — «وكيل الإشارات الاستباقية»
- **Primary user:** Leadership / Owner · **Owner role:** Product Owner · **Sensitivity:** P1 signals over approved artifacts.
- **Problem + value:** surfaces an important unasked signal with an importance rule and evidence, without noise spam.
- **Data & capabilities:** registered importance rules over metric movements + radar (PB-107) + watchlist events; dedupe.
- **Dependencies:** PB-107/PB-117; proposed VS-11+.
- **Wave · Slice · Flag:** Wave C · VS-11+ (proposed) · [REC] `proactive_insights_enabled`.
- **Acceptance criteria:** (1) every signal cites its rule id + evidence set; (2) dedupe window prevents repeats; (3) signal precision reviewed monthly (owner feedback loop); (4) no signal implies an unapproved violation.

### PB-208 · Capability promotion board — «مجلس ترقية القدرات»
- **Primary user:** Product / AI Owner · **Owner role:** Product Owner · **Sensitivity:** P1 governance.
- **Problem + value:** most-repeated Lane-3 questions become Lane 0/1 capabilities on evidence (frequency + value + cost + owner approval); extends SCR-07.
- **Data & capabilities:** question fingerprints, job stats; doc 04 §9 lifecycle.
- **Dependencies:** PB-205; proposed VS-11.
- **Wave · Slice · Flag:** Wave C · VS-11 (proposed) · [REC] `promotion_board_enabled`.
- **Acceptance criteria:** (1) candidates ranked by fingerprint frequency + measured cost + declared value; (2) promotion creates a registry entry through the normal doc 04 discipline (never a shortcut handler); (3) each promotion has an owner sign-off record; (4) demoted/retired capabilities keep history.

### PB-209 · Natural-language dashboard filters — «مرشحات اللوحات باللغة الطبيعية»
- **Primary user:** Managers · **Owner role:** Product Owner · **Sensitivity:** P1.
- **Problem + value:** easier dashboard use without widening permissions or metric definitions.
- **Data & capabilities:** NL → **governed enums only** (registered dimensions/filters, doc 11 §3–4); no free predicates.
- **Dependencies:** VS-07 registries; proposed VS-11+.
- **Wave · Slice · Flag:** Wave C · VS-11+ (proposed) · [REC] `nl_filters_enabled`.
- **Acceptance criteria:** (1) output is a validated closed spec — unknown values hard-fail with suggestions; (2) period text resolves through the deterministic period resolver (I4); (3) no new data exposure vs manual filters (authorization equivalence test); (4) misparse never silently changes the question (I5).

### PB-210 · Internal display mode — «وضع العرض الداخلي»
- **Primary user:** Leadership spaces · **Owner role:** Publication Authority · **Sensitivity:** P1 — approved artifacts only.
- **Problem + value:** rotate the infographic and pulse on an internal screen.
- **Data & capabilities:** approved publication artifacts (PB-014/PB-118); auto-rotation; kiosk token policy (doc 16).
- **Dependencies:** PB-014; proposed VS-11+.
- **Wave · Slice · Flag:** Wave C · VS-11+ (proposed) · [REC] `display_mode_enabled`.
- **Acceptance criteria:** (1) renders only `approved`/`published` artifacts; drafts structurally excluded; (2) kiosk credential is display-scoped read-only; (3) rotation config owner-set; (4) a retracted artifact disappears on next cycle.

### PB-211 · Knowledge-gap recommendations — «توصيات فجوات المعرفة»
- **Primary user:** Knowledge Team · **Owner role:** Knowledge Lead · **Sensitivity:** P1–P2.
- **Problem + value:** propose content/training from recurrence + dispersion, each tied to clusters and approved evidence (completes the §11.4 pair with PB-105/PB-106).
- **Data & capabilities:** CAP-D8 + QST/CHAL clusters + dispersion measures.
- **Dependencies:** PB-105/PB-106; proposed VS-11+.
- **Wave · Slice · Flag:** Wave C · VS-11+ (proposed) · [REC] `knowledge_recs_enabled`.
- **Acceptance criteria:** (1) every recommendation cites cluster ids + approved evidence; (2) recommendation type matches the gap type (FAQ vs توحيد معرفة vs تدريب); (3) owner accept/reject recorded; accepted ones become PB-015 actions; (4) impact re-measured after publication (dispersion delta).

### PB-212 · Cohort comparison — «مقارنة الشرائح»
- **Primary user:** Analyst · **Owner role:** Product Owner · **Sensitivity:** P1.
- **Problem + value:** fair comparison of programmes/sectors/periods with explicit cohort definitions — no unfair rankings.
- **Data & capabilities:** cohort definition objects over registered dimensions; support + coverage rules (doc 10 E.0 discipline).
- **Dependencies:** VS-07 metric layer; proposed VS-11+.
- **Wave · Slice · Flag:** Wave C · VS-11+ (proposed) · [REC] `cohort_comparison_enabled`.
- **Acceptance criteria:** (1) cohort definitions are saved, versioned, and printed with every result; (2) minimum support per cohort cell enforced with «حجب»; (3) «قطاع» three-way disambiguation enforced; (4) no ranked league table without CI overlap indication.

---

## 6. Ranking model — how the consultation list is ordered [REC]

**Formula:**

> **score = (Value × Confidence) ÷ (Risk × Dependency-depth)**

with owner decisions overriding scores (see rule 3). Alternatives considered: value-only ranking (rejected — the amendment §12-24 explicitly demands Value/Risk/Dependency, and value-only ordering is what produced the legacy horizontal plan), WSJF cost-of-delay (rejected for now — no credible job-duration estimates pre-VS-03; revisit-trigger: two slices of actuals).

**Scales (anchored, 1 line each):**

- **Value 1–5** — 5: enables a committed early product or a daily operational loop (Amendment §2 loops); 3: strong recurring value for one persona; 1: convenience.
- **Confidence 0.25–1.0** — 1.0: data exists + method proven in this package; 0.5: method defined but unvalidated (needs an EXP); 0.25: data availability itself open (OD-gated).
- **Risk 1–5** — 5: accusation-grade or consultant-named surface (fairness/PII exposure) or production-detector change; 3: published leadership numbers; 1: internal ops tooling.
- **Dependency-depth 1–5** — count of undelivered upstream slices/artifacts on its critical path (cap at 5).

**Worked Wave A scores (computed at 2026-08-03; recomputed at every stop gate):**

| Item | V | C | R | D | Score | Reading |
|---|---|---|---|---|---|---|
| PB-004 Session 360 | 5 | 1.0 | 2 | 1 | 2.50 | First product value; low risk; only VS-00 ahead of it |
| PB-003 DataHub connectors | 5 | 0.75 | 1 | 1 | 3.75 | Highest score — everything joins through it; OD-07 trims confidence |
| PB-006/007/008 review cluster | 5 | 0.75 | 5 | 2 | 0.75 | Committed by SD-21 despite high accusation risk — decision overrides score |
| PB-009 VIOL-008 E2E | 5 | 0.75 | 5 | 2 | 0.75 | Same cluster; one-type-at-a-time contains the risk |
| PB-001 Nightly run | 5 | 0.75 | 2 | 2 | 0.94 | Needs VS-01+VS-02 shapes to consolidate meaningfully |
| PB-011 Learning pipeline | 4 | 0.5 | 5 | 3 | 0.13 | Deliberately after first labels exist (VS-04) |
| PB-014 Infographic | 5 | 0.75 | 3 | 3 | 0.42 | Committed by SD-19/SD-21; needs trustworthy month data (VS-03) |
| PB-015 Action Center | 4 | 0.75 | 2 | 3 | 0.50 | After KPIs exist to act on |
| PB-013 Ops dashboard | 4 | 0.75 | 1 | 3 | 1.00 | Rides VS-06 with PB-015 |
| PB-017 RBAC/masking | 5 | 1.0 | 1 | 0→1 | 5.00 | Not optional; structurally first (VS-00) |
| PB-018 Provenance explorer | 3 | 1.0 | 1 | 3 | 1.00 | Spine early, UI later |

**Why the Wave A order stands (three sentences):** the skeleton and security baseline (PB-017) precede everything because every later acceptance test presumes login, deny-by-default, and migrations; source unification (PB-003/004/005) precedes violations because a review decision without session context and verbatim quotes is untestable; the committed-early cluster (PB-006…PB-010 → VS-02, PB-014 → VS-05) is scheduled by owner decision [DECISION SD-19/SD-20/SD-21], with the nightly run (VS-03) and learning loop (VS-04) between them because the infographic needs a trustworthy month close and the learning loop needs the labels VS-02 produces. Amendment §15's mandatory start order (VS-00 → VS-01 → VS-02 → VS-03 → VS-04 → VS-05) is the binding sequence; this ranking model exists to order **consultations beyond it**, not to reorder it.

**Standing rules:** (1) scores are recomputed at every stop gate with the owner present; (2) a score never schedules work — only owner approval does (SD-21); (3) any item touching consultant-named or accusation-grade surfaces carries Risk = 5 and cannot be averaged down by bundling it inside a bigger card.

---

## 7. Backlog lifecycle and traceability

### 7.1 Lifecycle states (every card carries exactly one)

`proposed → consulted → approved-for-slice → in-slice → delivered → measured` (+ terminal side-states `deferred` / `rejected`, both reversible only by a new consultation)

| State | Entry condition | Artifact | Who moves it |
|---|---|---|---|
| `proposed` | Card exists in this document with full fields | The card | Planning/author |
| `consulted` | Presented to the owner at a stop gate or ad-hoc session, with current score | Dated consultation note in runtime `PRODUCT_BACKLOG.md` | Product Owner |
| `approved-for-slice` | Owner names the slice and scope trims | Approval entry naming PB → VS binding | Owner only |
| `in-slice` | The slice's ticket plan (doc 25 §4.2) references the PB id | Ticket plan files | Implementation agent |
| `delivered` | Slice acceptance checklist green + stop-gate approval (doc 25 §2) | Acceptance report `docs/acceptance/VS-xx.md` | Owner sign-off |
| `measured` | Post-delivery value check at the next monthly close (usage + the card's value metric) | Measurement note; feeds PB-120 discipline | Service Owner |

The committed-early set (PB-006…PB-010, PB-014) enters at `approved-for-slice` from day one [DECISION SD-21]; every other card starts at `proposed`.

### 7.2 Traceability pointers

- **Decisions:** SD-18…SD-22 live in doc 02's settled-decision register; this document cites, never redefines.
- **Open decisions:** OD-28…OD-35 (+ extended OD-07) live in doc 22 §3.2; every `[ASSUME]` above names its OD.
- **Capabilities/sources/screens:** CAP-OPS-01…12 in doc 04 (same registry discipline as §1 of that document); SRC-DATAHUB family in doc 05; SCR-14…SCR-22 (and the SCR-08 upgrade) in doc 18.
- **Execution:** slice bindings and acceptance checklists in doc 25; epic/ticket alignment in doc 23 §5; phase gates in doc 21.
- **Runtime mirrors:** `PRODUCT_BACKLOG.md` (seeded from this document at T-VS00-1 and updated per ticket DoD, doc 25 §4.3) carries live states; `docs/STATE.md` names the current slice/ticket; `KNOWN_GAPS.md` records accepted gaps per card.
- **Amendment coverage:** the package-level traceability file (`TRACEABILITY_OWNER_AMENDMENTS.md`) maps Amendment §8/§11 rows → PB ids → VS slices → acceptance tests; any PB row missing there is a package defect (Amendment §14.3).

# 10 — Analytical Methodology (Method Charter)
**Platform:** Nwafeth Intelligence — منصة نوافث لذكاء الجلسات الاستشارية (Monsha'at Advisory Session Intelligence Platform)
**Status:** Draft for owner review · **Date:** 2026-08-02 · **Author:** Planning package (Fable 5)
**Depends on:** 04 (capability catalogue), 05 (source contracts), 08 (data model) · **Feeds:** 11 (semantic layer), 12 (serving), 13 (Lane 3), 15 (evaluation), 18 (review UX), 20 (parallel run), 21 (roadmap), 22 (ADR/OD register), 24, 25
**Sources used:** GREENFIELD §2.5, §5 (full question texts), §6.1–6.3, §12 (full), §21, §23; MASTER_PROMPT §4.4, Appendix A, Appendix C, Appendix E (E.0–E.15, full); CORE-BRIEF §§3–13; arch docs as current-state evidence only; OWNER AMENDMENT 2026-08-03 §5–§7, §11, §12 doc 10 item, §13
**Amended:** 2026-08-03 — Owner Amendment integrated: method rules MR-17 (suspected/approved separation), MR-18 (false-negative estimation), MR-19 (action impact tracking), MR-20 (infographic KPI selection & suppression) added; constants and enforcement map extended [DECISION owner 2026-08-03]

---

This is the **method charter** of the platform: the normative statistical and analytical rules every capability obeys, the per-capability method for all 26 committed capabilities (CAP-A1…CAP-D10), the causality-wording doctrine, the semantic-clustering methodology, the coverage-block contract, and the statistical appendix. Doc 04 says *what* each capability promises; this document says *how the numbers are made and when they may be shown*. A capability whose implementation contradicts this charter is wrong even if its tests pass.

Normative keywords: **MUST / MUST NOT / SHOULD** in the RFC-2119 sense. Rules carry IDs **MR-01…MR-20** (MR-17…MR-20 added by the 2026-08-03 Owner Amendment) and are cited by ID from docs 11, 12, 13, 15 and from the capability registry itself.

**Contents**
- §1 Method charter — MR-01…MR-20 with enforcement points
- §2 Question-class taxonomy and error-cost assignment
- §3 Per-capability methods — CAP-A1…CAP-C8 (deep), CAP-D1…CAP-D10
- §4 Stated-cause vs inferred-cause doctrine and Arabic labelling rules
- §5 Semantic clustering methodology (R-P2)
- §6 Coverage block: definition and named-exclusion vocabulary
- §7 Statistical appendix (tests, corrections, formulas, worked examples)
- §8 Build order across the method set
- §9 Open decisions and revisit triggers

---

## 1. Method charter — MR-01…MR-20

These rules restate MASTER_PROMPT E.0's ten rules as platform-wide normative rules [DECISION MASTER_PROMPT E.0, carried forward per GREENFIELD §12], extended with four rules GREENFIELD §12.2 requires that E.0 did not spell out (multiple-comparison correction, composition checks, left-censoring, coverage-as-answer) and two engineering rules that make the charter enforceable rather than aspirational. The 2026-08-03 Owner Amendment adds four operational-product rules — MR-17 suspected/approved separation, MR-18 false-negative estimation, MR-19 action impact tracking, MR-20 infographic KPI selection & suppression [DECISION owner 2026-08-03 / Amendment §12 doc 10 item]. Violating these produces answers that are **confidently wrong**, which in a government compliance context is worse than no answer [FACT MASTER_PROMPT E.0].

Each rule states: the rule, why, the **enforcement point** (the structural mechanism that makes violation unbuildable or unpublishable — a lesson from the legacy system, where method rules lived in review comments and died there), and a worked example where useful.

### MR-01 — Rates per 100 sessions, never bare counts, for any comparison

Monthly session volume swings **2.7×** across the measured corpus (556 sessions in 2025-08 → 1,512 in 2025-12) [FACT CORE-BRIEF §11]. Any month-over-month comparison of raw counts measures the calendar, not behaviour. Every comparative statement MUST be a rate per 100 sessions **with its denominator printed**: «12.4 لكل 100 جلسة (من أصل 1٬204 جلسات)». Raw counts remain reportable as *levels* («كم مخالفة سُجّلت؟») but MUST NOT appear in a trend, ranking-across-periods, or programme-comparison claim without the rate beside them.

*Enforcement:* the metric registry (doc 11) marks every metric `comparison_safe: rate_only | count_ok`; the compiler refuses to place a `count`-granularity metric on a period-comparison or cross-programme axis. The deterministic verifier (I3/I18) checks that any rendered percentage-change claim references a rate-typed result.

### MR-02 — The session is the unit; row counts are separate declared measures

One session can carry 12 violations, 9 challenges, 30 satisfaction signals [FACT MASTER_PROMPT E.0-2]. Rankings and prevalence claims MUST use `COUNT(DISTINCT advisory_session_id)`. Row-grain measures (findings, mentions, signals) are legitimate **only as separately registered metrics with `grain: finding`** whose renderings say so on their face: «عدد الإشارات (قد تتكرر داخل الجلسة الواحدة)». Where both are informative — CAP-B5 explicitly requires finding count *and* distinct sessions *and* distinct consultants [FACT GREENFIELD §5-B5] — both are shown, labelled.

*Enforcement:* every registry metric declares `grain ∈ {session, consultant, finding, question, decision, rating_row, programme_period_cell}` [DECISION GREENFIELD §12.1]; the compiler derives `COUNT(DISTINCT …)` from grain, never from handler code. A finding-grain metric rendered without its grain label fails the render-contract test in doc 15.

### MR-03 — Every top-N carries its base

"Top 10 challenges" without "out of N sessions in the window, M of which had ≥1 challenge extracted" is a leaderboard, not an analysis [FACT MASTER_PROMPT E.0-3]. Every ranked answer MUST render the base line: «أعلى 10 تحديات — من أصل 1٬204 جلسات في الفترة، 862 منها استُخرج فيها تحدٍ واحد على الأقل».

*Enforcement:* the `RankedList` result type in the answer envelope (doc 11) has non-optional fields `population_n`, `population_with_signal_n`; a renderer receiving a ranked list without them is a type error, not a style bug.

### MR-04 — Minimum cell size n ≥ 30, with visible suppression

Below `MIN_CELL_N = 30` sessions per cell, report the count but **suppress the rate/percentage and mark the cell** («أقل من الحد الأدنى للعرض؛ العدد: 17») [DECISION MASTER_PROMPT E.0-4; precedent `_MIN_MEETINGS = 30` in the legacy `clear_steps_by_sector`]. Suppression is visible, never silent removal — a dropped row reads as "does not exist", which is a different (false) claim. Suppressed cells still count toward totals and coverage.

*Enforcement:* `MIN_CELL_N` is one shared constant in the analytical-constants registry (MR-16), applied by the semantic-layer post-processor — **not** re-implemented per capability (the legacy per-handler-literal failure). Golden tests include at least one suppressed-cell fixture per segmented capability (doc 15).

### MR-05 — Wilson 95% intervals on every published proportion

A 40% clear-steps rate on n=31 and on n=1,400 are different claims; stakeholders treat them identically unless the interval is shown [FACT MASTER_PROMPT E.0-5]. Every proportion MUST carry a Wilson score interval at `CI_LEVEL = 0.95` (formula §7.1). Rendering: «40% (فاصل ثقة 95%: 34–46%)». Intervals are computed deterministically by the semantic layer, never by a model (I3).

*Enforcement:* the `Proportion` result type carries `{p, n, ci_low, ci_high, method: "wilson95"}`; the composer's numeric provenance gate (R6) admits `ci_low/ci_high` into allowed literals, so a narrative cannot state a proportion without also being able to state its interval.

### MR-06 — The four confounds, checked before any pattern is published

Before any language/behaviour pattern is published, it MUST be stratified by: (1) **consultant identity** — a pattern MUST appear across `MIN_CONSULTANT_SPREAD = 5` distinct consultants or be reported only as «لوحظ لدى عدد محدود من المستشارين» and never as a programme-level pattern (one verbose consultant can manufacture a "pattern" single-handed); (2) **programme** (`window_name`: irshad/istisharat/sharakat); (3) **service category**; (4) **session length** (quartile bins). A pattern that vanishes under any stratification is an artifact, not a finding [DECISION MASTER_PROMPT E.0-6].

*Enforcement:* the contrastive-lift utility (§7.4) computes the four stratifications as part of one call and emits `consultant_spread`, `survives_strata: bool[4]` in its result; the publication gate for pattern-class capabilities (CAP-A1, CAP-C7) refuses `survives_strata` containing `false` without the qualifying rendering.

### MR-07 — Never explain a label with the features that define it

The single most dangerous trap in this question set [FACT MASTER_PROMPT E.0-7, E.1]. If a label (impact, quality, clarity) is computed from transcript features, then "which patterns characterise high-label sessions?" answered against that label returns the features that computed it — circular, tautological, and completely convincing. Every pattern analysis MUST document label lineage and either (a) use a label from an **independent pipeline or human source**, or (b) disclose the shared features and exclude them from the feature space (holdout methodology) [DECISION GREENFIELD §5-A1]. See CAP-A1 (§3.1) for the worked separation.

*Enforcement:* the capability registry records `label_source` and `feature_families` per pattern capability; a build check fails when `label_source.derived_from` intersects `feature_families` without an explicit `circularity_disclosure` entry. This is a structural check, not a review habit.

### MR-08 — Asymmetric error costs, assigned deliberately per question class

Error costs are never symmetric and the asymmetry MUST be chosen per question class, not left to a default [DECISION MASTER_PROMPT E.0-8]:

- **Accusation class** (violations, per-consultant negative claims): **precision over recall.** A false accusation against a named consultant is far more damaging than a miss. Requires verified verbatim quote (R7) + human review before publication. A stated miss is a data-quality note; a fabricated or misattributed accusation is an incident.
- **Discovery class** (R-P1 mining, new-category candidates): **recall over precision.** Surface candidates cheaply; the review queue filters. Nothing from this class reaches a published surface without passing review.
- Full class table with all 26 capabilities: §2.

*Enforcement:* the registry field `error_posture ∈ {precision_first, recall_first, balanced}` plus `publication_gate ∈ {none, review_required, review_required_named_individual}` drive the serving plane's rendering rules mechanically (doc 12).

### MR-09 — Every claim ships session identifiers and verifiable quotes

Every categorical claim MUST be backed by real sessions: identifiers (session ULID, turn index, speaker role, transcript source + version, extraction run) and verbatim quotes verified as exact substrings of the active transcript turn (I7, R7) [DECISION MASTER_PROMPT E.0-9 / R-P3(i)]. A category label with no session behind it is not reportable. This is a contractual requirement from the owner («يتم عرض الجلسات الفعلية او المشاكل الحقيقية») [FACT MASTER_PROMPT R-P3].

*Enforcement:* `findings.quote_ref` is FK-anchored to `transcript.turn` (doc 08 §6.1); the R7 quote gate runs at extraction time *and* at render time; `dropped_unverifiable` is a named coverage exclusion (§6).

### MR-10 — Validate the label before building analytics on it

Every model-derived label family (step_clarity, satisfaction polarity, violation class, friction flag, speaker role, cluster assignment, transcript quality) has an unmeasured error rate until measured. Before analytics on a label are published: sample `LABEL_VALIDATION_N = 100` sessions per label family (stratified across its values), owner/reviewer adjudication through the single review loop (ADR-0012), agreement computed (§7.7) and **published as a stated limitation inside the answers that use the label** [DECISION MASTER_PROMPT E.0-10; GREENFIELD §12.3 lists the required families]. Publication thresholds [REC]: Cohen's κ ≥ 0.70 AND raw agreement ≥ 85% → label unlocked; κ 0.50–0.69 → usable with a prominent caveat and **no cross-period trend claims**; κ < 0.50 → label blocked, capabilities depending on it answer `DATA_NOT_ENRICHED`. Alternatives considered: fixed accuracy threshold only (rejected — inflated by class imbalance); Krippendorff's α (kept as secondary when >2 adjudicators). Revisit when any family accumulates 3 validation rounds ≥ 0.85 κ (then re-validation cadence may drop to annually or on model/prompt change).

*Enforcement:* the capability registry marks each capability's `required_label_validations[]`; the golden-suite runner (doc 15) refuses to sign a capability whose required validations lack a current passing record in `ops` (validation records carry `model_id + prompt_sha + taxonomy_version`; any of these changing voids the record).

### MR-11 — Multiple-comparison correction on every pattern search

Any analysis that tests many hypotheses (phrase families in contrastive lift, dozens of categories scanned for trend movement) MUST apply Benjamini–Hochberg FDR correction across the tested set before any item is called significant [DECISION GREENFIELD §12.2]. Constants: `FDR_Q_PUBLISH = 0.05` for anything reaching a published surface; `FDR_Q_DISCOVERY = 0.10` for internal review-queue candidates [REC — asymmetry mirrors MR-08; alternative: Bonferroni, rejected as needlessly conservative for correlated Arabic phrase families; revisit if a published pattern is later falsified]. The correction scope is **one search family within one period** (e.g., all phrase features tested in one A1 monthly run; all category cells scanned in one C3 comparison). Procedure and worked example: §7.5.

*Enforcement:* the shared contrastive-lift and trend-scan utilities take the full hypothesis set and return BH-adjusted q-values; there is no code path that returns raw per-feature p-values to a renderer.

### MR-12 — Composition-change checks on every period comparison

A rate can move because behaviour changed or because the mix of sessions changed (new programme cohort, consultant turnover, ingestion backlog landing mid-window) [FACT MASTER_PROMPT E.10]. Every period-over-period claim MUST run a composition check: compare the distribution of sessions over `window_name`, `service_category`, and consultant cohort between the two windows; if total variation distance > `COMPOSITION_TVD_MAX = 0.10` on any axis (or consultant-cohort turnover > 20%), the answer MUST carry the composition caveat and report whether the trend survives stratification on the shifted axis: «تنبيه: تغيّر مزيج البرامج بين الفترتين (32%→45% لبرنامج إرشاد)؛ الاتجاه بعد التثبيت على البرنامج: …» [REC — TVD chosen for interpretability as "share of sessions that would need to move"; alternative Jensen–Shannon divergence, rejected as unexplainable to a stakeholder; revisit-trigger: TVD flagging >50% of comparisons, then tune threshold].

*Enforcement:* composition check is computed inside the trend engine (CAP-C3 shared code), not per capability; its result is a required field of the `PeriodComparison` envelope.

### MR-13 — Left-censoring rules for taxonomy introductions

Categories are discovered over time (R-P1), so a category's first period is the period detection began, not the period the phenomenon began. For any category with `introduced_at_version > version active at period start`: (a) live-view trend series MUST begin at the introduction period and render earlier periods as «غير مقيس قبل v6 (2026-03)» — **left-censored, never zero**; (b) frozen packs are never retro-patched (ADR-0011); (c) a backfill is only valid via re-extraction that stamps the new `taxonomy_version` on re-derived findings, after which the censoring boundary moves and the change is announced in the pack changelog; (d) growth claims spanning an introduction boundary are forbidden by the trend engine — «ارتفاع» computed from a censored zero is the archetypal false trend. The same applies to detection-gap closures: when تهكم (VIOL detection) first ships, its "increase from zero" is a censoring artifact and MUST be rendered as coverage change, not behaviour change.

*Enforcement:* `tax.taxonomy_category.effective_from_version` + the trend engine's censoring check; a golden fixture pins the "new category does not create a fake trend" behaviour (doc 15).

### MR-14 — Coverage is part of the answer, not a footnote

Every analytical answer MUST compute and render its coverage block (`used ≤ matched ≤ total`, `used + Σexclusions = matched`) with **named** exclusions — unmatched sessions, unresolved consultants, missing ratings, unavailable transcripts, unclassified taxonomy rows, incomplete Lane-3 partitions, missing speaker roles, review-rejected findings, provider-source distribution [DECISION GREENFIELD §12.7; I8]. Full contract and closed exclusion vocabulary: §6.

*Enforcement:* the coverage block is computed by the harness from query metadata, never written by a model (a `check_scope_declared` failure is a bug and raises, it is not a "bad draft" [FACT MASTER_PROMPT §"verifier"]). The identity `used + Σexclusions = matched` is asserted in the regression suite on every golden answer.

### MR-15 — Stated cause is reportable; inferred cause is not (operationally)

The platform reports causes **as stated by beneficiaries**, clustered and counted with quotes, labelled «الأسباب كما ذكرها المستفيد». Model-inferred causal claims MUST NOT appear in operational reporting; they require a separately approved research methodology [DECISION GREENFIELD §12.5]. Full doctrine, claim-form table, and renderer-enforced Arabic wording: §4.

### MR-16 — One analytical-constants registry; no per-capability numeric literals

Every threshold in this charter is a **named constant in one registry** (`ops`-owned, versioned, changes via ADR-level review): the legacy system's `_MIN_MEETINGS = 30` lived in one handler while others had nothing — the rule existed only where someone remembered it [FACT MASTER_PROMPT E.0-4]. Calibratable constants (dispersion thresholds, near-duplicate cosine, dead-air floor) additionally record `calibration_method`, `calibrated_on (corpus_snapshot)`, and `recalibrate_when`. Initial registry:

| Constant | Value | Kind | Used by |
|---|---|---|---|
| `MIN_CELL_N` | 30 | fixed [DECISION E.0-4] | all segmented capabilities |
| `MIN_CONSULTANT_SPREAD` | 5 | fixed [DECISION E.0-6] | A1, C7, all pattern claims |
| `LABEL_VALIDATION_N` | 100 per label | fixed [DECISION E.0-10] | MR-10 families |
| `CI_LEVEL` | 0.95 (z=1.96) | fixed | all proportions |
| `FDR_Q_PUBLISH` / `FDR_Q_DISCOVERY` | 0.05 / 0.10 | fixed [REC] | MR-11 |
| `MIN_SUPPORT_PATTERN` | 30 sessions in smaller class | fixed [DECISION E.1] | contrastive lift |
| `COMPOSITION_TVD_MAX` | 0.10 | tunable [REC] | MR-12 |
| `KAPPA_UNLOCK` / `KAPPA_CAVEAT_FLOOR` | 0.70 / 0.50 | tunable [REC] | MR-10 |
| `CLOSING_WINDOW_SHARE` | last 15% of turns | tunable [FACT E.1] | A1 closing features |
| `CONFUSION_END_WINDOW` | max(last 20% of turns, last 5 turns) | tunable [FACT E.2] | A2 |
| `NEAR_DUP_COSINE` | 0.92 starting point → calibrated in EXP-04 | calibrated | B2 repetition, B3 clustering guard |
| `DEAD_AIR_FLOOR_S` | 20 s starting point → sensitivity analysis §7.10 | calibrated | B2 |
| `DISPERSION_FIXED_MAX` / `DISPERSION_INCONSISTENT_MIN` | corpus quartiles at calibration (§3.6) | calibrated | B3/C6 |
| `MIN_CLUSTER_OCCURRENCES` | 10 sessions | tunable [REC] | B3/C6/C8 cluster reporting |
| `STALE_MIX_TURNOVER_MAX` | 20% consultant-cohort turnover | tunable [REC] | MR-12 |
| `MONTH_COMPLETENESS_GATE` | 0.98 (sessions in terminal states) | assumed [ASSUME OD-28 — Amendment §13 working assumption] | MR-20, CAP-OPS-08 draft trigger |
| `REVIEW_SLA_NORMAL_D` / `REVIEW_SLA_HIGH_D` | 7 d / 2 d | assumed [ASSUME OD-30] | MR-17 queue reporting; QT-14 (doc 15) |
| `FN_WEEKLY_SAMPLE_N` | set by EXP-11 (weekly stratified sample of unflagged sessions) | calibrated [ASSUME OD-32] | MR-18 / DS-14 (doc 15) |
| `ACTION_BASELINE_MONTHS` / `ACTION_FOLLOWUP_MONTHS` | 1 closed month each, matched lengths | tunable [REC] | MR-19 |

*Enforcement:* an import-lint rule bans numeric literals in analytical modules for any quantity whose name exists in the registry; golden fixtures reference constants by name so a re-calibration re-runs the affected suite.

### MR-17 — Suspected and approved violations are separate populations; never mixed

[DECISION owner 2026-08-03 / Amendment §5, §7.4, §13; SD-20] **Definitions.** A **suspected** violation is a `violation_finding` attached to a `review_case` that has not reached the approved state (every state before and other than an approving decision — new, in review, second review, أدلة غير كافية, إحالة لمالك السياسة). An **approved** violation is one whose case carries an append-only `review_event` with the decision «مخالفة صحيحة» by an authorized reviewer, with closed reason code, reviewer identity, and text/detector/taxonomy stamps. «ليست مخالفة» removes the finding from both populations; «إعادة تصنيف» re-enters it as *suspected* under the new category — reclassification never transfers approval.

**Denominators.** Published violation rates — packs, dashboards, the monthly infographic, leadership surfaces, CAP-B5's ranked series, CAP-D5's violation axis — MUST be computed from **approved findings only**, per 100 sessions with the session denominator printed (MR-01/MR-02 unchanged). Suspected volume is a **workload measure over cases, not a violation measure over sessions**: it is reported only through the queue metrics `suspected_cases_open` (open-case count) and `review_backlog_age` (age distribution of open cases), each at case grain with its own stated base (doc 11 §8.4).

**Never mixing.** No series, table, chart, or delta may combine suspected and approved counts. A leadership surface MAY show the queue size as its own separate figure beside the approved figure, labelled «حجم قائمة الاشتباه» — never summed, never trended together. Trend claims on approved counts that cross a review-throughput change MUST carry a composition caveat (a draining backlog looks like a violation spike — MR-12 logic applied to review state).

**Queue-size reporting.** Queue figures render in review-state vocabulary only («حالات قيد المراجعة», «متأخرة عن اتفاقية مستوى الخدمة» [ASSUME OD-30: 7d normal / 2d high-priority]) together with the fixed disclaimer of the «اشتباه مخالفة» surface; the word «مخالفة» unqualified is banned on suspected rows (§4.3 lint list extended).

*Enforcement:* every violation/review metric in the registry carries `review_state_scope ∈ {approved_only, suspected_queue_only}` (doc 11 §2/§8.4, gate G-REG-8); the compiler refuses a spec mixing scopes or placing a `suspected_queue_only` metric on a published/leadership surface; renderer lint bans confirmed wording on non-approved cases; doc 15 T-19 pins the rule end-to-end.

### MR-18 — Estimated recall via weekly stratified sampling of unflagged sessions

[DECISION owner 2026-08-03 / Amendment §6.4] Reviewing only detector-flagged cases measures precision and is structurally blind to what detection missed; a published "accuracy" derived from flagged-only review is forbidden — it is the false-high-precision failure the amendment names. Every violation category with a live detector MUST carry an **estimated recall**, produced as follows:

1. **Weekly stratified random sample of unflagged sessions** (no detector fired) sent to light human review [ASSUME OD-32: weekly cadence; size set by EXP-11]. Strata — Amendment §6.4 verbatim: **programme; consultant; session duration; session month; transcript quality; under-represented categories** (boosted allocation). Each weekly manifest is immutable and versioned (SD-22 dataset discipline); found cases enter the normal review queue as suspected cases (they are production findings too — MR-17 states apply).
2. **Design-weighted estimation, per category, with CIs.** Estimated recall per category = approved detections ÷ (approved detections + design-weighted missed cases found in sampling), reported **per category with a confidence interval** (Wilson on the effective sample size; bootstrap when stratum weights are unequal) — never one aggregate recall number (doc 15 §6.6 makes the blended-total row structurally impossible in release reports).
3. **Missed-case reports are a complement, never the instrument.** Reviewer-initiated reports («إضافة اشتباه لم يرصده النظام», DS-13) feed the estimate only as an attention-biased **sensitivity lower bound**; they never replace random sampling.
4. **EXP-11 owns the sizing** (sample sizes, stratum allocation, cadence; doc 15 §2). QT-13 (doc 15) sets the per-category floor; a floor breach blocks detector-release promotion, but never unpublishes precision-gated approved cases.

*Enforcement:* one shared design-weighted estimator reads DS-14 manifests; no code path computes "recall" from flagged-case review alone; release-evaluation templates carry per-category rows only.

### MR-19 — Action impact tracking is association-only

[DECISION owner 2026-08-03 / Amendment §2.4, §8.3 PB-120; extends MR-15] Every service-improvement action (CAP-OPS-09) that claims measurable intent declares at creation: the **target metric** (registered id), the **scope** (programme/consultant cohort/category), a **baseline window** (≥ `ACTION_BASELINE_MONTHS` closed month(s) ending before the intervention), the **intervention marker** (action completion date with its completion evidence), and a **follow-up window** (matched length, starting after completion) [REC — matched closed-month windows; alternative: cumulative post-period, rejected as trend-confounded; revisit-trigger: owner needs faster reads, then add an interim indicator clearly marked غير محسوم].

The post-action reading renders baseline vs follow-up with a CI on the difference and MUST use association wording: «تغيّر المؤشر بعد الإجراء» — never «بسبب الإجراء»; the MR-15/§4 causal-claim ban applies verbatim («دون ادعاء سببية غير مثبتة»). **Confound notes are mandatory on every reading:** composition change across the boundary (MR-12), overlapping actions on the same scope (listed by id), detector/taxonomy/label version changes inside either window (MR-13 censoring), and seasonal volume swing (MR-01 rates only). `action_completion_rate` measures execution; metric movement measures association; the platform states both and claims neither as proof. A genuinely causal evaluation remains the §4.2-3 research escape hatch — separately approved methodology, never merged into operational reporting.

*Enforcement:* the action-impact envelope has required fields `{target_metric, scope, baseline, followup, confounds[]}`; an action without a registered target metric renders status only (no impact panel); §4.3's banned-phrase lint gains «نتيجة للإجراء», «أثبت الإجراء» as system claims.

### MR-20 — Infographic KPI selection and suppression

[DECISION owner 2026-08-03 / Amendment §7; SD-19] The monthly leadership infographic (CAP-OPS-08, «نبض خدمة الاستشارات والإرشاد — ملخص الشهر») is one page built around what the leader needs, not everything the platform owns. **Selection criteria — all required for a KPI to appear:**

1. **Leader-relevant / decision-linked:** it serves one of Amendment §7.4's five fixed parts (نبض الخدمة، جودة وأثر الجلسات، صوت المستفيد، الجودة والالتزام، ما يحتاج قرارًا) — the parts are the frame; KPIs compete for slots inside them.
2. **Registered and governed:** every number is a Metric Result from the registry with drill-down to its session set (I3/I12; no bespoke infographic arithmetic; no LLM-computed numbers).
3. **Stable definition:** the metric definition and taxonomy version are unchanged across the compared windows, or the change is stated on the face (MR-13).
4. **Coverage-fit:** the month passes the completeness gate — ≥ `MONTH_COMPLETENESS_GATE` (98%) of sessions in terminal states [ASSUME OD-28 — Amendment §13 working assumption]; any owner-documented override renders **on the face of the infographic**, never in a footnote elsewhere.
5. **Validated labels:** MR-10 unlocks apply — a KPI on an unvalidated label family may not appear.

**Suppression and review-state rules carried through, not relaxed for layout:** MR-04's n ≥ 30 suppression renders as suppression inside the infographic (editorial un-suppression forbidden); the violations part shows **approved-only** figures with the queue size as its separate figure (MR-17); no consultant names in the general edition [ASSUME OD-31 — masked; direct-manager visibility rides OD-09/OD-14 RBAC].

**Month-over-month comparison rules:** closed months only, end-exclusive; rates per 100 sessions (MR-01) with denominators in the footer; comparison against the immediately preceding closed month (plus an owner-fixed reference window if approved); MR-12 composition caveat and MR-13 censoring apply; a KPI whose comparison month is censored, suppressed, or below support renders «غير قابل للمقارنة» — never a delta computed anyway.

*Enforcement:* the infographic composes exclusively from pack-grade Metric Results (I11; R6 allowed-literals; optional LLM phrasing passes the R6 gate); the draft job refuses to run before the completeness gate or a recorded override (Amendment §4.4-5: no official artifact from an incomplete run without a face-visible override); template KPI slots bind to registry ids so an unregistered number cannot be typeset (I12); tri-format payload parity is pinned by doc 15 T-18.

### 1.1 Enforcement map (summary)

| Rule | Structural enforcement | Failure behaviour |
|---|---|---|
| MR-01/02/03 | typed metric registry fields (`grain`, `comparison_safe`); typed `RankedList` envelope | compiler refuses; type error |
| MR-04/05 | shared post-processor; `Proportion` type | render-contract test fails |
| MR-06/11 | shared contrastive-lift utility, single implementation | no raw-p code path exists |
| MR-07 | registry `label_source` × `feature_families` build check | build fails |
| MR-08 | registry `error_posture` + `publication_gate` | serving refuses unreviewed publication |
| MR-09 | FK-anchored `quote_ref` + R7 gate at extract & render | finding not countable; `dropped_unverifiable` |
| MR-10 | validation records keyed by model/prompt/taxonomy; suite refuses unsigned | capability stays `DATA_NOT_ENRICHED` |
| MR-12/13 | trend-engine required fields; censoring check | comparison refuses to render |
| MR-14 | harness-computed coverage; asserted identity | raises (bug), never ships silently |
| MR-15 | renderer wording contract + banned-phrase lint (§4) | render-contract test fails |
| MR-16 | constants registry + literal lint | build fails |
| MR-17 | registry `review_state_scope` (G-REG-8, doc 11) + compiler scope check + renderer lint | mixed-state spec refuses to compile; doc 15 T-19 red |
| MR-18 | single design-weighted estimator over DS-14 manifests; per-category-only report template | no flagged-only recall path exists; release evaluation fails to file |
| MR-19 | action-impact envelope required fields + association-wording lint | impact panel not rendered; render-contract test fails |
| MR-20 | completeness-gate check before draft; KPI slots bound to registry ids; T-18 parity | draft job refuses; build fails on unregistered slot |

---

## 2. Question-class taxonomy and error-cost assignment

Every capability is assigned exactly one **question class**, which fixes its error posture (MR-08), its publication gate, and its statistical machinery. This table is normative; doc 04's registry entries carry the class as a required field.

| Class | Error posture | Publication gate | Statistical core | Capabilities |
|---|---|---|---|---|
| **Measurement** (rates & levels of governed metrics) | balanced | none (deterministic) | Wilson CI, two-proportion tests, suppression | B1, B5, C3, C4, D4, D6, D7, D9 |
| **Pattern / contrastive** (what distinguishes X from Y) | balanced, BH-corrected | reviewer sign-off on new pattern families | contrastive lift + Fisher + BH + confound strata | A1, C7 |
| **Detection + topic attribution** (find sessions with signal S, attribute topics) | recall-tilted detection, precision-tilted attribution | review for new categories | position-scoped lexicons + extractor cross-check, rate ranking | A2, A3, B2, C1, C2, C8 |
| **Accusation** (names individuals with negative claims) | **precision-first** | **human review mandatory per case; named-individual gate** | union-of-detectors, quote-verified, per-case review | B4, B5 (case lists), D2 (per-consultant gaps), D5 (violation axis) |
| **Consistency / alignment** (two instruments compared) | balanced; disagreement is the product | review before any per-consultant disagreement is shown | agreement matrices, weighted κ, dispersion | B3, C6, D1, D2, D3 |
| **Entity-scoped extraction** (names government bodies) | precision-first on entity resolution; framing-locked | steward review of registry edges | entity resolution + rate ranking; no evaluative language | C5 |
| **Composition / serving** (assembles other capabilities) | inherits strictest constituent | sign-off (OD-10) for packs | n/a — composition rules | D5, D8, D10 |

Two consequences worth stating plainly: (1) **B5 is measurement at the aggregate level but accusation at the case level** — the ranked taxonomy table may render from reviewed findings only; unreviewed findings appear solely as «قيد المراجعة: N» counts. (2) **D2's programme-level agreement matrix is measurement; the same data grouped by named consultant is accusation-class** and takes the review gate.

---

## 3. Per-capability methods

Format per entry: unit/denominator → method → parameters → failure modes & confounds → validation → output & coverage. Schema names are doc 08's (`core.advisory_session`, `findings.finding`, `findings.quote_ref`, `findings.cluster`, `transcript.turn`, `tax.entity_registry`, `core.beneficiary_rating`, `core.consultant_evaluation`, `core.attendance_fact`, `core.session_status_fact`, `core.followup_fact`). All extraction is fresh on the new platform (findings re-derived, never migrated — ADR-0016); legacy counts below are planning baselines only [FACT CORE-BRIEF §11].

### 3.1 CAP-A1 · `impact_pattern_analysis` — أنماط الجلسات عالية/منخفضة الأثر

The hardest question in the set and the one most likely to be answered vacuously [FACT MASTER_PROMPT E.1]. Class: pattern/contrastive.

**The circularity trap (MR-07), restated for the new platform.** Any composite "session quality/impact score" the new platform computes will be computed from transcript features. If A1 is answered against that score, the analysis returns the scoring rubric to its author and calls it a discovery. The legacy system had exactly this live (`impact_level` from pattern-based scorers, then `high_impact_patterns` filtered on it) [FACT MASTER_PROMPT E.1]. The new platform therefore separates **label families** from **feature families** structurally:

| Candidate label | Independence | Planning coverage baseline | Role |
|---|---|---|---|
| `core.beneficiary_rating` (1–5, human) | fully independent (a human in the session) | rating rows ~24k; bridge resolves under half of sessions [FACT CORE-BRIEF §11] | **validation label** on the bridged subset |
| `step_clarity` finding (clear/partial/none) | independent extraction pipeline; matches the owner's own definition «تنتهي بخطوات واضحة» | legacy: clear 6,274 / partial 6,824 / none 2,486 / null 1,327 | **primary label** (validate per MR-10 first) |
| any composite quality score | circular with linguistic features | n/a | secondary cross-check only, never primary |
| follow-up session booked (`core.followup_fact` + internal data) | independent behavioural outcome | availability per OD-07 | tie-breaker where present |

**Method.**
1. **Label:** `step_clarity ∈ {clear}` vs `{none}`; `partial` dropped from the contrast and reported as the named exclusion `label_partial_excluded` (contrast the extremes; avoid arguing about the middle) [FACT E.1].
2. **Features — three families, consultant turns only, all through the single Arabic normaliser:**
   - *Lexical:* phrase families seeded from the owner's exemplars — imperatives «ابدأ»، «سوّي»، «ارفع طلب»; ordinals «أول شيء»، «ثاني شيء»; temporal sequencing «بعد شهر»، «خلال أسبوع»; diagnostic reframes «مشكلتك … وليس …»; closing summaries «إذن الخطوة الأولى…» [FACT GREENFIELD §5-A1 seeds].
   - *Structural:* `consultant_talk_share`, `consultant_question_rate` (the owner's «يشرح أكثر مما يسأل» as a ratio), turn count, mean consultant turn length, and **closing-window variants** of the lexical families restricted to the last `CLOSING_WINDOW_SHARE` of turns — the rubric is about how the session *ends*.
   - *Sequential:* ordered step sequence present (≥2 ordinal markers at increasing positions); action item anchored to a named actor (`action_item.assignee_role` ≠ null).
3. **Scoring:** contrastive lift `P(f|clear)/P(f|none)` with `MIN_SUPPORT_PATTERN`, Fisher exact, BH at `FDR_Q_PUBLISH`, effect size + CI + raw counts both sides (§7.4). A lift of 3.0 on 31-vs-10 sessions is a hypothesis; on 900-vs-300 it is a finding.
4. **Confound stratification** (MR-06). A family surviving in ≤4 consultants renders as observed-in-few, never programme-level.
5. **Validation arm:** on the bridged subset, confirm that top-ranked families also separate rating 5 from rating 1–2. **If they do not, the answer is not ready — say so** rather than publish [FACT E.1-5].
6. **Discovery arm (R-P1):** cluster closing windows of `clear` sessions (§5); high-lift clusters absent from the seed list go to the naming queue. This is where «مقاييس أخرى أيضاً» is honoured.

**Output:** ranked pattern families, each with lift + CI + q-value, session counts both sides, consultant spread, 2–3 verified quotes with ULIDs; the owner's low-impact consequence list attached as *stated* consequences, never measured outcomes. **Failure modes:** circularity (structurally blocked); publishing pre-BH p-values (no code path); treating `partial` silently (named exclusion). **Cadence:** monthly.

### 3.2 CAP-A2 · `confusion_end_topics` — جلسات تنتهي بارتباك وموضوعاتها

Class: detection + topic attribution. Two canonical errors [FACT MASTER_PROMPT E.2]: counting confusion anywhere («ما فهمت» at minute 3 is a normal part of consultation — the owner is explicit: «في نهاية الجلسة»), and ranking topics by raw confused-count (re-ranks popularity).

**Method.**
1. **Position-scoped detection:** markers matched in **beneficiary turns within `CONFUSION_END_WINDOW`** (max of last 20% of turns, last 5 turns). Turns lacking `speaker_role` (legacy baseline 5.7%) are excluded and declared (`speaker_role_missing`).
2. **Negation/inversion pairs — the highest-yield correctness detail in this question.** The lexicon stores explicit polarity pairs; a confusion marker preceded in-turn by a positive-transition verb inverts. Seed table (golden fixture, extendable via review queue):

| Confusion (counts) | Resolution (inverts — MUST NOT count) |
|---|---|
| «ما فهمت» | «الحين فهمت»، «توني فهمت» |
| «مو واضح» / «ما هو واضح» | «صار واضح»، «وضحت الصورة» |
| «ما استوعبت» | «استوعبت الآن» |
| «يعني كيف؟» | «تمام، عرفت كيف» |
| «طيب وبعدين؟» *(end-position only; mid-session it is a normal prompt)* | «عرفت وش الخطوة الجاية» |

3. **Extractor cross-check:** the per-session extractor's `confusion_marker` findings are an independent detection of the same construct. Publish the agreement rate between lexicon and extractor; large disagreement (>20 p.p.) means one is wrong and the capability answers `DATA_NOT_ENRICHED` until adjudicated [FACT E.2-3].
4. **Topic attribution & ranking by rate:** attribute each confused session to dominant topic (canonical CHAL category, `service_category`, government entity — labelled per «قطاع» rule); rank by **confusion rate = confused / sessions-on-topic** with Wilson CI and MR-04 suppression; raw counts beside rates so a small-but-severe topic stays visible.
5. **Owner's seven exemplar themes** (توقعات التمويل، اختيار نموذج العمل، المستثمرون والشركاء، المشاريع متعددة المسارات، الجلسات المعلوماتية، التسويق، الإجراءات الحكومية) are reported explicitly even when ranked low — their absence from the top is itself a finding [FACT E.2-5].

**Validation:** MR-10 sample on the end-window classifier (label family: confused-end yes/no). **Cadence:** monthly.

### 3.3 CAP-A3 · `satisfaction_signals` — مؤشرات الرضا (إيجابي/سلبي/محايد)

Class: detection. The neutral class is mandatory (the legacy system never built its handler) [FACT MASTER_PROMPT E.3].

**Method.**
1. **Categories from clustering, not a hardcoded list (R-P1/R-P2):** embed satisfaction-signal quotes, cluster **within polarity** (§5), owner names clusters through the review loop. The owner's five-per-side exemplars are seed + sanity check — if the corpus produces five wildly different categories, raise it as a finding, don't suppress it.
2. **Rank by distinct sessions** (MR-02) — one effusive beneficiary can emit eight positive signals.
3. **Explicit vs implicit, separately:**
   - *Explicit:* lexical («شكراً»، «استفدت»، «وضحت الصورة»، «ما استفدت»، «ضيعت وقتي»).
   - *Implicit — the state transition* «من تردد/تشويش إلى وضوح/قرار», the owner's strongest implicit indicator and the one no phrase list can find: measure hesitation-marker density in the **first third** of beneficiary turns vs decision-language density («خلاص بسوي»، «قررت»، «بمشي على الخطة») in the **last third**; flag sessions where the swing exceeds the calibrated threshold (calibrated as the 75th percentile of the within-session swing distribution on the corpus snapshot, then frozen in the constants registry). Budget for it explicitly — it is an extraction feature, not a query [FACT E.3-3].
4. **Commitment-to-next-step** («حجز جلسة أخرى»): prefer behavioural evidence from internal follow-up facts over text; each figure states its source system.
5. **Boundary with CAP-D1:** transcript signals and internal ratings are **never interchangeable measurements** [FACT GREENFIELD §5-A3]; their agreement analysis is CAP-D1, cross-referenced, not duplicated here.

**Validation:** MR-10 on polarity assignment (positive/negative/neutral — 100 sessions each). **Cadence:** monthly.

### 3.4 CAP-B1 · `clear_steps_rate` — نسبة الجلسات المنتهية بخطوات واضحة

Class: measurement. Headline metric of the whole product [FACT MASTER_PROMPT E.4].

**Method.** Numerator: sessions whose current `step_clarity` finding = `clear`. Denominator: sessions in window with non-null clarity; null-clarity sessions are the named exclusion `label_missing` (legacy baseline 1,327 — declared, never silently dropped). Splits: `service_category`, canonical CHAL category, `government_entity`, programme, and other approved dimensions — **every split labelled with which meaning of «قطاع» it uses** (the legacy `clear_steps_by_sector` said "sector" and grouped by government entity; that class of quiet mismatch is banned by the three-dimension rule [FACT CORE-BRIEF §13.3]). Wilson CI; MR-04 suppression; two-proportion test on month-over-month deltas; MR-12 composition check.

**Non-negotiable:** strongest validation in the platform — this is the number most likely to be quoted in a slide. MR-10 with 100 sessions **per category value** (clear/partial/none), agreement published beside the rate: «نسبة الخطوات الواضحة 48% (ثقة 95%: 45–51%) — دقة التصنيف المُتحقّق منها: κ=0.78». A 62% rate means nothing if the label is 60% accurate. **Cadence:** monthly.

**Worked rendered answer (the golden-fixture shape for every measurement capability):**

> **نسبة الجلسات المنتهية بخطوات واضحة — يوليو 2026**
> **48.2%** (فاصل ثقة 95%: 45.3–51.1%) — 554 من أصل 1٬150 جلسة مُقيَّمة
> مقارنة بيونيو 2026: 45.1% → **+3.1 نقطة** (الفرق غير ذي دلالة إحصائية عند 0.05؛ p=0.14)
> **التغطية:** استُخدمت 1٬150 من 1٬240 جلسة مطابقة (من إجمالي 16٬911) — مستبعد: 41 بلا نص، 22 أدوار متحدثين غير محسومة، 27 تصنيف غير متوفر
> **حدود القياس:** دقة تصنيف «خطوات واضحة» المتحقق منها بشرياً: κ=0.78 (اتفاق 89% على عينة 300 جلسة، 2026-06)

Every element above is typed envelope output (I11): the number, interval, denominator, comparison verdict, coverage line, and validation caveat are separate fields; the renderer composes them; no model wrote any of them.

### 3.5 CAP-B2 · `session_time_loss` — مواضع فقدان وقت الجلسة

Class: detection (temporal). Feasible — turn timing was 100% present in the legacy corpus [FACT MASTER_PROMPT E.5] — but the legacy `silence_pct` was computed wrong and the new platform MUST NOT inherit the bug.

**Speaking-time computation — interval-merge, normative.** The legacy computation summed per-speaker seconds without merging overlapping intervals; cross-talk inflated speaking time and drove the ratio negative, then clamped to 0 with a warning [FACT MASTER_PROMPT E.5 / CORE-BRIEF §12]. The new computation (pseudocode §7.8): merge each speaker's turn intervals, merge the union for total-speech time, silence = session span − union; **assert non-negative by construction**. Any threshold derived from the old values (the legacy 40% waste threshold, p75=34%) is invalid; thresholds are re-derived on the new computation over a corpus snapshot before first publication, with sensitivity analysis (§7.10) — no single arbitrary threshold [DECISION GREENFIELD §5-B2].

**Four loss modes, measured separately, in seconds:**

| Mode | Detection | Unit reported |
|---|---|---|
| **Repetition** «شرح مكرر» | near-duplicate consultant turns within one session: cosine ≥ `NEAR_DUP_COSINE` on turn embeddings (fallback: normalised token overlap); duration of later occurrences summed | minutes + share of session |
| **Argument/churn «جدال دائري»** | runs of rapid short alternating turns (both speakers, turn length < session median, inter-turn gap < 3 s) sustained ≥ 8 turns [REC starting values → sensitivity §7.10] | minutes |
| **Side questions / drift «استطراد»** | turn-window topic distance from session's dominant topic above threshold, sustained ≥ 2 min | minutes |
| **Dead air «صمت زائد»** | inter-turn gaps > `DEAD_AIR_FLOOR_S`, summed; pure timing, no model | minutes |

Plus the derived span **«طويل بلا قرار»**: longest contiguous span with no `decision_point` finding and no `action_item` anchor; report duration *and position* — 12 decision-free minutes at the start is orientation; the same span at the end is failure [FACT E.5].

**Output:** per-mode minutes and share of session duration, ranked by total minutes lost across the window, per-mode examples with ULIDs and timestamps. **Validation:** MR-10 sample where adjudicators mark loss spans on 100 sessions; per-mode precision/recall published. **Cadence:** monthly.

### 3.6 CAP-B3 · `repeated_questions_top` + CAP-C6 · `inconsistent_answers` — one computation, two thresholds

**Build once, read at both ends** [DECISION MASTER_PROMPT E.6; GREENFIELD §5-C6 "shares core machinery"]. B3 asks for recurring questions with *stable* answers (automation candidates); C6 for recurring topics with *unstable* answers (human-specialist candidates). Same clustering, same dispersion measure, opposite tails.

**Method.**
1. **Extract beneficiary questions:** interrogative beneficiary turns → `beneficiary_question` findings (extractor); recurrence is **semantic** (R-P2, §5): «كيف أحصل على تمويل؟» ≡ «وش الطريقة عشان أمول مشروعي؟» ≡ «أبغى أموّل مشروعي، وش أسوي؟» → one QST cluster. String matching fails the question entirely.
2. **Cluster** into QST clusters with owner-approved canonical labels (e.g. `QST-0042` «كيفية الحصول على تمويل»).
3. **Collect answering turns:** consultant turns immediately following each question occurrence within a window (first consultant turn + continuation turns up to the next beneficiary turn).
4. **Answer dispersion** per cluster with ≥ `MIN_CLUSTER_OCCURRENCES` occurrences: mean pairwise cosine distance among answer embeddings + spread over structured advice extracted from them (§7.9), **length-normalised**.
5. **Two thresholds, calibrated then frozen:** on a corpus snapshot, compute the dispersion distribution over all qualifying clusters; `DISPERSION_FIXED_MAX` = 25th percentile, `DISPERSION_INCONSISTENT_MIN` = 75th percentile [REC — quartile calibration makes the thresholds corpus-anchored and explainable; alternative: fixed cosine values, rejected as embedding-model-dependent; recalibrate on embedding-model change (EXP-04) or annually].
   - **Low dispersion + high frequency ⇒ B3:** report cluster, frequency (distinct sessions), the modal answer, and its variance. The owner supplies expected modal answers (سجل تجاري، قوائم مالية، إثبات إيرادات…) — used as a **detection floor**: if the pipeline does not surface the owner's listed ten, it is under-detecting [FACT E.6-5].
   - **High dispersion + high frequency ⇒ C6:** report cluster with **2–3 contrasting verified quotes side-by-side** — the contrast *is* the finding and is more persuasive than any dispersion score.
6. **The dispersion guards (mandatory):**
   - *Length confound:* normalise before comparing (§7.9).
   - *Consultant-mixture guard:* decompose dispersion into within-consultant vs between-consultant components. A "high dispersion" cluster that is two consultants each internally consistent but different from each other is a **training/alignment problem**; genuinely scattered answers are a **knowledge-gap problem**. The two route to different owners, so the answer states which it is [FACT E.6-6].
   - *Legitimate contextual variation (GREENFIELD §5-C6):* before a C6 cluster is published, stratify answers by beneficiary context (programme, service category, business stage where stated); dispersion explained by context renders as «تباين مشروع حسب حالة المستفيد», and only residual dispersion is a candidate inconsistency. **Human specialist review is required before any C6 item becomes a compliance/quality claim** [DECISION GREENFIELD §12.6].

**Context-controlled comparison protocol (GREENFIELD §12.6, normative for C6).** "Same question, different answers" is only an inconsistency if the *contexts* were comparable. Before a cluster may enter the C6 candidate list:
1. **Context vector per occurrence:** `(programme, service_category, business_stage_where_stated, period_quarter, channel)` attached to each question occurrence from governed dimensions (never inferred from transcript beyond what the extractor already captured as structured fields).
2. **Stratified dispersion:** recompute dispersion within context strata (cells with ≥5 occurrences). If pooled dispersion ≥ `DISPERSION_INCONSISTENT_MIN` but every within-stratum dispersion < `DISPERSION_FIXED_MAX`, the variation is contextual → render «تباين مشروع حسب حالة المستفيد» with the strata shown, and the cluster does NOT enter C6.
3. **Temporal legitimacy check:** answers may legitimately change when regulations change. If dispersion is low within each quarter but high across quarters, render «تغيّر في الإجابة عبر الزمن (يرجّح تغيّراً تنظيمياً)» and route to review as a *KB update candidate* (D8), not an inconsistency.
4. **Residual inconsistency** (high dispersion within comparable context and period) is the only thing published as C6, and only after specialist review [DECISION GREENFIELD §12.6] — the reviewer sees the contrasting quotes, the context vectors, and the consultant decomposition, and records a typed verdict: `inconsistent_confirmed / contextual / regulatory_change / insufficient_evidence`.

**Worked example.** QST-0107 «هل أحتاج سجل تجاري للبيع أونلاين؟» — 84 occurrences, pooled dispersion 0.61 (> τ_inconsistent). Stratified: within قطاع التجزئة 0.22, within الخدمات اللوجستية 0.19; across strata 0.58 → contextual variation (different activity classes have different registration rules); NOT published as inconsistency. Counter-case: QST-0031 «كم رأس المال المطلوب لتأسيس شركة؟» — dispersion 0.66 within the same programme, same quarter, same category, between-consultant share 78% → C6 candidate, review verdict `inconsistent_confirmed`, published with two contrasting verified quotes.

**Validation:** MR-10 on cluster assignment (§12.3 "repeated-question clusters"); cluster purity metrics §5.3. **Cadence:** B3 monthly; C6 quarterly.

### 3.7 CAP-B4 · `violations_with_quotes` — مؤشرات المخالفات مع اقتباسات

Class: **accusation — highest stakes in the set; it names individuals** [FACT MASTER_PROMPT E.7]. Precision over recall (MR-08); publication strictly review-first.

**Preconditions (all mandatory):** deduplication by natural key at the schema level (the legacy table had no PK and 3,103 duplicate rows inflating counts ~1.84× [FACT CORE-BRIEF §11] — doc 08's constraints make this unrepresentable); R7 quote verification live; every published case passed human review (ADR-0012).

**Method.**
1. **Union of three detectors, each with its own confidence and provenance:**
   - (a) the curated rule/pattern taxonomy over VIOL-001…VIOL-008 (the owner's 8 required types, Arabic wording verbatim from GREENFIELD §5-B4);
   - (b) the per-session LLM extractor's `violation` findings (strict schema, quote-first);
   - (c) Lane-3 / R-P1 discovery for what neither covers — **تهكم and تسويق شخصي are covered by no legacy detector** [FACT CORE-BRIEF §13.2]; until their detectors ship, both render as the named exclusion `detection_gap` — never silently omitted.
   Findings carry `detector ∈ {rule, extractor, discovery}`; agreement between (a) and (b) on the same turn raises case confidence; disagreement routes to review with both provenances shown.
2. **Speaker attribution is mandatory:** a violation is a *consultant* behaviour; beneficiary turns never enter; turns without `speaker_role` are excluded and declared.
3. **Quote first, label second:** every case carries a verbatim quote verified as an exact substring of the active transcript turn, with session ULID, turn index, source + version, and surrounding context window. A case that cannot produce a verifiable quote is **not a case** — it is a review-queue candidate at most.
4. **VIOL-004 «توجيه لمسار واحد بدون مقارنة» needs a structurally different detector** — it is the *absence* of comparison, not the presence of a phrase [FACT E.7-4]. Detector: a directive recommendation turn (imperative + path noun: «لازم تفتح فرع»، «الحل الوحيد إنك…») with **no alternative-bearing marker** (أو، بدل، خيار ثاني، مقارنة، من جهة أخرى، تقدر كذلك) in the surrounding ±10-turn window. Worked positive example: «الحل الوحيد قدامك إنك تسجل شركة، لا تفكر في شي غيره» with zero alternative markers in the window → candidate. Expected low precision; **100% of VIOL-004 candidates route through review before ever being counted.**
5. **Review-first publication:** the published surface shows *reviewed-confirmed* cases; candidate volume appears only as «قيد المراجعة: N». Reviewed-rejected cases are the named exclusion `review_rejected` (append-only history, OD-11).

**Validation:** MR-10 per violation type where volume allows; types below 100 lifetime candidates validate on all cases (they are individually reviewed anyway). **Cadence:** monthly. **Output:** case list (reviewed), each with quote + context + detector provenance + review trail.

### 3.8 CAP-B5 · `violation_types_ranked` — أكثر أنواع المخالفات تكراراً

Class: measurement over reviewed accusation data. For each of the 8 types + discovered types: **finding count, distinct affected sessions, distinct affected consultants, rate per 100 sessions, reviewed-vs-unreviewed split, examples with verified quotes, taxonomy version, coverage exclusions** [DECISION GREENFIELD §5-B5 — all eight fields required]. Rank by distinct sessions (MR-02); a single pathological session with many findings MUST be visible, not distortive — render the max-findings-per-session diagnostic when the finding:session ratio for a type exceeds 3 [REC]. **Report all 8 owner types every period, including zeros** — a type with zero detections renders as either true zero (detector live, nothing found) or `detection_gap` (detector not yet shipped); the two are different statements and the answer says which [FACT E.7-6]. Poisson rate test for month-over-month movement on rare types (§7.3); MR-13 censoring when new detectors ship. **Cadence:** monthly.

### 3.9 CAP-C1 · `challenges_top` — أبرز التحديات المتكررة

Class: detection + topic attribution. Map `challenge` findings to canonical CHAL groups via the curated synonym seed first (legacy `v2_synonyms`, 91 rows, is *seed evidence* — re-reviewed, not migrated as truth), then embedding-cluster the unmapped remainder (§5) and route new groups to owner naming (R-P1). **Publish mapping coverage** — «87% من صفوف التحديات أُسندت لمجموعة معيارية» — because an unstated long tail is how a top-10 quietly becomes a top-10-of-40% [FACT MASTER_PROMPT E.8]. Distinct sessions; per-100 rate; Wilson CI; MR-04; base line per MR-03. Fresh-per-period discovery (R-P3): each period's clusters mined from that period's sessions; prior taxonomy assists classification but may not suppress a newly dominant category [DECISION GREENFIELD §6.3]. **Cadence:** monthly or quarterly («آخر شهر/ربع» — both registered).

### 3.10 CAP-C2 · `challenge_symptoms_causes_asks` — الأعراض والأسباب المعلنة وطلب المستفيد الفعلي

Class: detection; **blocked on new extraction** — the legacy columns for symptoms and beneficiary intent are empty for 0/67,082 rows [FACT MASTER_PROMPT E.9]; no query over existing data can answer it, and faking it from challenge text is forbidden.

**Route [ASSUME OD-05, recommendation carried forward]:** run the **first** cycle as a Lane-3 deep-analysis job per major challenge category (learns what the extraction schema should ask), then commit the corpus-wide extraction with `challenge` payload fields `symptoms[]`, `stated_cause`, `actual_ask` populated — because C2 is a recurring quarterly obligation, not a one-off [FACT E.9]. Per-schema field definitions live in doc 04/08; extraction model per doc 14.

**Method (once populated).** For each major CHAL group: recurring symptoms (clustered, counted by distinct sessions), **stated causes** (clustered quotes of what the beneficiary said the obstacle was), and the **actual ask** — what the beneficiary requested, which is frequently not the challenge's textbook remedy. Every element ships quotes + ULIDs. **Framing is locked by §4:** section header «الأسباب كما ذكرها المستفيد», never «الأسباب الجذرية» unqualified; no model-inferred causality [DECISION GREENFIELD §5-C2, §12.5]. **Cadence:** quarterly.

### 3.11 CAP-C3 · `trends_vs_previous_month` — الاتجاهات صعوداً وهبوطاً

Class: measurement. Quarterly refresh cadence, monthly comparison unit — both declared [FACT MASTER_PROMPT E.10]. For each canonical category (CHAL, VIOL, SAT, QST clusters): rate per 100 sessions this month vs last, absolute + relative change, two-proportion z-test (or Poisson e-test for rare categories, §7.3), CI on the difference. **BH correction across all category cells scanned in the run** (MR-11) — a scan of 60 categories at α=0.05 expects 3 false "trends" uncorrected. **Suppress any cell below MIN_CELL_N rather than reporting a dramatic swing on 4 sessions — that swing is the single most likely thing to end up on a slide** [FACT E.10]. MR-12 composition check mandatory (programme mix, consultant cohort, ingestion backlog); MR-13 censoring at taxonomy introductions. Rendering: rising/falling/flat with q-value tiers, never raw sorted percentage deltas.

### 3.12 CAP-C4 · `top_inquiries_and_categories_3m` — أبرز الاستفسارات والفئات خلال 3 أشهر

Class: measurement over B3's clustering artifact (no separate pipeline). Rolling 3-month window; top QST clusters by distinct sessions; top-5 categories with «قطاع» disambiguated into its three registered dimensions and the answer labelled with which is shown — `service_category` (planning baseline 77.3% populated, 48 values; remainder = named exclusion `dimension_unresolved`), `government_entity`, and `business_sector` = **UNAVAILABLE until an external identity source exists** → `declare_unanswerable(DIMENSION_NOT_AVAILABLE)` for that reading [FACT CORE-BRIEF §13.3]. The discipline is entirely denominators + suppression; nothing methodologically novel [FACT E.11].

### 3.13 CAP-C5 · `government_friction_mentions` — الجهات الحكومية كنقاط احتكاك

Class: entity-scoped extraction. **An entity-resolution problem before an analytics problem:** planning baseline 4,372 distinct entity strings across 21,561 mentions — one agency spelled many ways plus ASR noise plus departments conflated with ministries. Ranking without normalising produces a meaningless list that looks plausible [FACT MASTER_PROMPT E.12].

**Method.**
1. **Canonical entity registry before analytics** (`tax.entity_registry` + `tax.entity_alias`): reviewed list of actual bodies with alias sets covering spelling variants, abbreviations, ASR mishearings, English forms. Worked alias set: `ENT-0007` هيئة الزكاة والضريبة والجمارك ← {«الزكاة»، «هيئة الزكاة»، «الزكاة والدخل»، «مصلحة الزكاة والدخل»، «زاتكا»، «ZATCA»، «هيئة الزكاه والضريبه» *(ASR/spelling)*}. Seeding: frequency-sort the distinct strings — the head covers most mentions; the tail routes to steward review. New aliases only via the review loop (ADR-0012); alias edges are versioned.
2. **Resolution at extraction time:** `government_mention` findings carry `entity_id` (resolved) or `raw_mention` + unresolved status. **Publish mapping coverage** («91% من الإشارات أُسندت لجهة معيارية معتمدة») — the unresolved tail is the named exclusion `entity_unresolved`, a stated limitation, never a silent one.
3. **Problem-type attachment:** classify the co-occurring difficulty from surrounding turns into the reviewed problem taxonomy (تراخيص، قرارات تمويل، متطلبات مستندية، مدد زمنية، تداخل اختصاصات…), stored as `problem_category_id`.
4. **Rank by distinct sessions**, per 100 sessions, Wilson CI, MR-04.
5. **Framing — hard constraint from the owner: «(مجرد استخراج من النص)».** Output is *mentions in sessions*, never an assessment of an entity's performance. Renderer contract: no evaluative adjectives, no «الأسوأ»-style ranking language, standing caveat that mention frequency reflects what beneficiaries discussed, not agency quality [DECISION GREENFIELD §5-C5]. This is a government product naming government bodies — wrong framing makes correct numbers unusable. Banned-phrase lint list in §4.3.

**Validation:** MR-10 on `is_friction` and on entity resolution (100 mentions sampled; resolution precision target ≥ 0.95 — precision-first per §2). **Cadence:** quarterly.

### 3.14 CAP-C7 · `pressure_language_patterns` — الأنماط اللغوية في جلسات الضغط والضيق

Class: pattern/contrastive — same machinery as CAP-A1, different label and different speaker [FACT MASTER_PROMPT E.13]. Label: sessions carrying `pressure_signal` findings (planning baselines: urgency 8,832 / confusion 6,987 / frustration 6,776 / distress 544 / other 279). Features: phrases in **beneficiary** turns. Contrast: unflagged sessions **matched on session length and topic**. Report lift + CI + counts both sides; BH; MR-06 strata. Cross-tabulate surfaced phrase families with the owner's named themes (سيولة، شريك، إيرادات) so the customer's vocabulary is visibly covered.

**Class-imbalance rule (normative, §7.6):** `distress` (544) and `other` (279) are too small for stable pattern mining. Either merge into a coarser «ضغط حاد» class **with the merge stated on the face of the answer**, or report them as counts-with-quotes only, no pattern claims. **Never run lift on 279 rows and present the output as a pattern** [DECISION MASTER_PROMPT E.13]. Distress sessions individually carry quotes and are eligible for the review queue regardless (welfare-relevant), but review-queue visibility is not a published "pattern". **Cadence:** quarterly.

### 3.15 CAP-C8 · `decision_hesitation` — نقاط القرار الأكثر تردداً

Class: detection over the R-P2 clustering artifact — **cannot ship before the DEC clustering artifact exists and is owner-approved** [FACT MASTER_PROMPT E.14-5]. The owner supplies the operational definition verbatim: hesitation «(يظهر من كثرة الأسئلة حول نفس القرار)» — a **counting rule over a semantic grouping**, not a lexicon.

**Method.**
1. Cluster `decision_point` findings into canonical decisions (§5): «شركة أم مؤسسة»، «أُدخل شريكاً أم لا»، «أغلق الفرع أم أستمر»، «أستمر بالتمويل الذاتي أم أطلب تمويلاً».
2. Attribute beneficiary questions (from B3's `beneficiary_question` findings) to a decision cluster **within the same session** (embedding similarity to the cluster centroid above threshold; unattributed questions stay unattributed — no forced assignment).
3. **Hesitation index** = questions attributed to the same decision cluster per session, averaged over sessions where that decision appears. **Rank by index, not frequency:** a decision appearing in 2,000 sessions at 1.1 questions each is *settled*; one in 200 sessions at 4.8 questions each is where beneficiaries are stuck — and the second is what the owner asked about [FACT E.14-3].
4. Report **resolution rate** beside the index: share of appearances followed by an `action_item` anchored to that decision — converts the finding into something actionable («قرار الكيان القانوني: مؤشر التردد 4.2، يُحسم داخل الجلسة في 31% فقط من الحالات»).
5. MR-04 suppression on decisions appearing in <30 sessions; index reported with a bootstrap CI (§7.11).

**Validation:** MR-10 on decision-cluster assignment. **Cadence:** quarterly.

### 3.16 CAP-D1 · `rating_vs_transcript_alignment` — اتساق تقييم المستفيد مع المؤشرات النصية

Class: consistency/alignment. **The two instruments are never interchangeable measurements** [DECISION GREENFIELD §5-A3/§5-D1]: a rating is a post-session human judgement on a 1–5 scale; a transcript satisfaction class is a model-derived reading of in-session language. The deliverable is the **agreement/disagreement structure itself**, not a merged "true satisfaction".

**Method.**
1. **Scope:** bridged sessions only (both a resolved rating and an analysed transcript). The bridged share is the **first line** of the coverage block (planning baseline: bridge resolves under half of sessions [FACT CORE-BRIEF §11]); unbridged sessions are the named exclusions `rating_unbridged` / `no_transcript`.
2. **Banding:** rating 1–2 → سلبي, 3 → محايد, 4–5 → إيجابي [REC — matches SAT polarity classes; alternative: keep 5 bands, rejected for cell sparsity; revisit at >5k bridged sessions/quarter]. Session-level transcript class = dominant polarity by distinct signals with deterministic tie rules (ties → محايد).
3. **Agreement matrix** (3×3), linear-weighted κ with CI (§7.7) [REC — weighted, because سلبي-vs-إيجابي disagreement is worse than سلبي-vs-محايد; simple percent agreement reported alongside but never alone (inflated by class imbalance)].
4. **Disagreement queues, typed:** (a) **rating high / text negative** — courtesy-rating hypothesis «مجاملة في التقييم», routed to review as a measurement-quality signal; (b) **rating low / text positive** — instrument/timing problem candidate (e.g., rating reflects scheduling frustration, not session content). Individual disagreement cases carry quotes + rating row identifiers and are review-queue items, never auto-published. Worked example: rating = 5 with final-third beneficiary turn «بصراحة ما استفدت شي، بس شكراً لوقتك» → queue (a).
5. **Aggregate reporting:** agreement rate by programme/service category (MR-04, Wilson CI); trend in agreement over time (MR-12 checks apply).

**Wording rule (§4):** «اتساق/عدم اتساق بين مصدرين» — never «التقييم خاطئ» or «المستفيد يجامل» as a system claim. **Cadence:** monthly.

### 3.17 CAP-D2 · `consultant_vs_beneficiary_eval` — اتساق تقييم المستشار مع تقييم المستفيد

Class: consistency (aggregate) / **accusation-class when grouped by named consultant**. Scope: sessions with both `core.consultant_evaluation` and `core.beneficiary_rating`; dual-rated share leads the coverage block.

**Method.** (1) Paired band cross-tab (consultant self/session evaluation bands × beneficiary bands), weighted κ, programme-level. (2) **Per-consultant systematic gap:** mean signed difference (consultant − beneficiary) for consultants with ≥ `MIN_CELL_N` dual-rated sessions, with bootstrap CI; a consultant systematically rating sessions higher than beneficiaries do is a calibration signal for coaching. (3) **Publication gate:** the per-consultant gap list is review-gated (named individuals, MR-08); the published aggregate shows the *distribution* of gaps, not names; names appear only inside CAP-D5 for authorized roles after review. (4) Disagreement extremes (gap ≥ 2 bands) route to review with both evaluations + transcript evidence attached. **Wording:** «فجوة معايرة بين التقييمين», never a competence claim. **Cadence:** monthly.

### 3.18 CAP-D3 · `status_attendance_quality_link` — الحالة التشغيلية والحضور مقابل جودة الجلسة

Class: consistency/association. Vocabularies for status/attendance come from the internal source contract [ASSUME OD-07 — access mechanism and enum semantics pending; safe assumption: read-only export with documented enums; unmapped legacy status strings surface through CAP-D9, never silently bucketed].

**Method.** Condition quality outcomes (clear-steps rate, satisfaction polarity mix, beneficiary rating mean) on operational categories (status, attendance class, reschedule count, lead time between booking and session). Two-proportion tests between categories; MR-04 per cell; MR-12 composition (programme mix differs across channels). **Association wording only (§4):** «الجلسات المعاد جدولتها مرتين تُظهر نسبة خطوات واضحة أقل (38% مقابل 49%)» — never «إعادة الجدولة تسبب انخفاض الجودة». Confound note rendered: attendance patterns correlate with beneficiary segment and channel; stratify by both before publishing any difference. **Cadence:** monthly.

### 3.19 CAP-D4 · `cancellation_noshow_analysis` — تحليل الإلغاء وعدم الحضور

Class: measurement, entirely on internal data — **no transcript exists for a no-show**, so this capability's denominators are booking-based, not session-based (the one place MR-01's "per 100 sessions" is *wrong*):

- **Denominator:** booked appointments in window (scheduled sessions from internal data).
- **Metrics:** cancellation rate = cancelled / booked; no-show rate = no-show / booked; late-cancellation rate (< 24h) where timestamps allow [ASSUME OD-07]. All per-100-**booked**, denominator printed.
- **Splits:** programme, channel, service category, consultant (per-consultant rates only at ≥30 booked — MR-04), weekday/time-of-day (descriptive), period.
- **Tests:** two-proportion between programmes/channels; Poisson e-test on rare cells; MR-11 across scanned cells; MR-12 composition (a channel's no-show "improvement" that is really a booking-mix change must be caught).
- **Linkage analysis:** no-show/cancellation propensity vs prior-session experience (previous session's rating, clear-steps outcome) — association wording only; this is the D-cap most tempting for causal overreach.
- **Coverage:** unmatched bookings (`internal_unmatched`), bookings with unknown status (`status_unmapped`).

**Cadence:** monthly.

### 3.20 CAP-D5 · `consultant_360` — الملف الشامل للمستشار

Class: composition — inherits the strictest constituent gate. Axes: volume; beneficiary rating (with bridge coverage per axis); clear-steps rate vs same-programme cohort percentile (Wilson CI); **reviewed** violation cases; D2 calibration gap (post-review); follow-up completion (CAP-D7 feed, where outcome data exists); reviewed evidence highlights (verified quotes).

**Composition rules (normative):**
1. **Never a single composite score** [REC — a single score invites ranking abuse and hides coverage asymmetries; alternative: owner-weighted index — rejected for launch; revisit-trigger: owner explicitly requests a governed rubric with published weights]. Each axis renders with its own denominator, coverage, and period.
2. **What may NOT be shown without review:** unreviewed violation candidates (show only «قيد المراجعة: N» with no case detail); unreviewed D2 per-consultant gaps; any quote not R7-verified; any comparative claim on <30 sessions in window; beneficiary comments containing PII (placeholder policy, I15).
3. **Cohort context is mandatory:** every rate renders beside the same-programme cohort distribution (percentile band, not rank — «ضمن الشريحة 60–80» not «المرتبة 12 من 45») [REC — exact ranks on noisy rates imply false precision; revisit if the owner requires ranks, then only with overlapping-CI grouping].
4. **Access:** quality/compliance roles per RBAC (I14); the 360 is a coaching instrument, not a league table — the renderer carries this framing note.

**Cadence:** monthly refresh; ad-hoc by authorized query.

### 3.21 CAP-D6 · `program_service_comparison` — مقارنة البرامج وفئات الخدمة

Class: measurement. All programme/service-category comparisons use the **same registered metric definitions** (I12 — no per-programme metric variants), normalized rates per 100 sessions, per-cell coverage declared (programmes differ in transcript coverage and rating bridge rate — a comparison ignoring that is a comparison of pipelines, not programmes).

**Normalization [REC]:** report **crude rates and directly-standardized rates** side by side — standardized over the pooled service-category mix (both windows/all compared programmes), so a programme serving harder categories is not penalized by mix. Alternative: regression adjustment — rejected for launch (opaque to stakeholders); revisit-trigger: >3 confounding axes demonstrated. Ranking language only where CIs separate; otherwise «لا فرق ذا دلالة». MR-04 per cell; MR-11 across the comparison grid; MR-12 across periods. **Cadence:** monthly + quarterly pack section.

**Worked standardization example.** Programme إرشاد: crude clear-steps rate 42% (n=800); programme استشارات: crude 52% (n=600). Category mix: إرشاد serves 60% legal/licensing sessions (hard: pooled clear rate 35%) and 40% marketing (easy: pooled 58%); استشارات serves the reverse. Standardized to the pooled mix (50/50): إرشاد = 0.5·(its legal rate 38%) + 0.5·(its marketing rate 48%) = 43.0%; استشارات = 0.5·(41%) + 0.5·(59%) = 50.0%. The crude 10-point gap is really ≈7 points after mix; both rates render with the sentence «بعد التثبيت على مزيج فئات الخدمة». If the ordering *flips* under standardization (Simpson's case), the answer leads with the standardized reading and shows the crude one as the confounded view — never the reverse.

### 3.22 CAP-D7 · `followup_completion` — متطلبات المتابعة وإتمام الإجراءات

Class: measurement, **only where outcome data exists** [DECISION GREENFIELD §5-D7]. Denominator: sessions with `followup_fact.requirement = true` AND outcome/closure data present in the internal source; sessions with a requirement but no outcome tracking are the named exclusion `outcome_unavailable` — the platform states «بيانات الإتمام غير متوفرة لهذه الفئة» rather than inferring completion from transcript language (inferring completion from talk is exactly the invented-outcome failure GREENFIELD §5-D bans). Metrics: completion rate, median time-to-closure, aging buckets (0–30/31–60/61–90/>90 days), by programme/service/consultant (MR-04). Cross-link: A3's «حجز جلسة أخرى» prefers this behavioural source over text. **Cadence:** monthly.

### 3.23 CAP-D8 · `kb_automation_candidates` — مرشحات التوحيد المعرفي والأتمتة والتصعيد

Class: composition over B3/C6 artifacts. Scoring per QST cluster [REC — weights are a starting point, recalibrated after the first owner review cycle]:

```
kb_score(cluster) =
    0.35 * frequency_norm          # distinct sessions, min-max over qualifying clusters
  + 0.25 * (1 - dispersion_norm)   # answer stability (B3 side)
  + 0.20 * consultant_consensus    # within/between decomposition: high = consultants agree
  + 0.10 * period_stability        # answer stable across ≥2 periods (§5.4)
  + 0.10 * answerable_from_docs    # reviewer flag: answer exists in official sources
```

Routing classes (each cluster gets exactly one, review-approved before appearing in any published list): **standardize** (KB article candidate — high score), **automate** (high score + fully deterministic answer), **escalate/specialist** (C6 side: high dispersion not explained by context), **keep-human** (contextual by nature — dispersion legitimately explained by beneficiary context). The owner's expected modal answers (سجل تجاري، قوائم مالية، إثبات إيرادات…) serve as the detection floor here as in B3. Output rows carry the cluster's evidence pack (quotes, modal answer, dispersion decomposition). **Cadence:** quarterly.

### 3.24 CAP-D9 · `data_quality_disagreement` — جودة البيانات وتعارض المصادر

Class: measurement — **data quality as a first-class user-visible capability** [DECISION GREENFIELD §5-D9], not an ops dashboard. Registered DQ metrics (all governed, all drill-down): unmatched provider↔internal sessions (both directions); unresolved consultant identities by `resolution_status`; rating rows bridging to no session + bridge rate trend; transcript-less sessions; speaker-role-missing turn share; extraction validation rejection rate; `not_yet_reclassified` counts by taxonomy version; provider-source distribution of active transcripts; duration disagreement beyond tolerance between provider metadata and internal records; DLQ depth and age. Each metric declares a warning threshold in the constants registry; threshold breaches emit `ops` observations and appear in packs' coverage pages. This capability is why the answer to «ليش التغطية 48%؟» is a report, not a shrug — and it is the public gauge of doc 09's reconciliation work. **Cadence:** continuous computation; monthly summary.

### 3.25 CAP-D10 · `executive_packs` — الحزم التنفيذية الشهرية والربعية

Class: composition + publication. A pack is a **frozen, reproducible artifact** (ADR-0011): stamped with `(period, corpus_snapshot, taxonomy versions, model ids, prompt shas, code version)`; recomputed per period from that period's sessions (R-P3), never patched.

**Composition rules:**
1. Only capabilities whose gates pass may contribute; a capability failing MR-10 validation contributes a «غير جاهز — التصنيف قيد التحقق» placeholder, never a number.
2. **Coverage page is section 1 of every pack** (MR-14), including provider-source distribution and every named exclusion aggregated.
3. Suppressed cells render as suppressed (MR-04) — a pack with hidden rows is a different document than a pack with marked rows.
4. Accusation-class content: reviewed cases only; the pack records the review state at freeze time.
5. Persisted form: one row per finding + explicit session set (denominator), so every rate is re-derivable after taxonomy merges [FACT MASTER_PROMPT §4.3-3b]; non-additive statistics (BH-corrected lift, dispersion reads, hesitation index) are marked `RECOMPUTE_REQUIRED` — a taxonomy edge touching their inputs invalidates the whole pack section, recomputed before display [FACT MASTER_PROMPT §4.3].
6. Publication: sign-off by publication authority [ASSUME OD-10 — product owner]; supersession-never-edit; retraction obligations for withdrawn accusations rendered as open obligations until acknowledged.
7. Every pack states its period **and** its comparison unit (quarterly pack, monthly comparison — CAP-C3's dual cadence).

**Cadence:** monthly + quarterly (scheduled Lane-3/worker jobs keyed by period).

---

## 4. Stated-cause vs inferred-cause doctrine — and the Arabic labelling rules

The distinction between what people said and what a model concluded is the difference between evidence and opinion, and only one of them belongs in a government report [FACT MASTER_PROMPT E.9]. GREENFIELD §12.5 makes it binding: stated causes are reportable; model-inferred causal claims require a separately approved research methodology and MUST NOT mix into operational reporting.

### 4.1 Claim forms — what may be said, in which words

| Claim form | Allowed? | Required Arabic framing | Example |
|---|---|---|---|
| **Stated cause** (beneficiary said X is why) | Yes — clustered, counted, quoted | «الأسباب كما ذكرها المستفيد» — mandatory header; each cause with quote + ULID | «قال المستفيد: ما قدرت أفتح الحساب البنكي لأن السجل معلّق» |
| **Association** (A co-occurs with B) | Yes — with tests + confound checks | «يرتبط بـ» / «يظهر مع» / «مصاحب لـ», never «بسبب» | «الجلسات المُلغاة مرتبطة بفئة خدمة التمويل (فرق ذو دلالة)» |
| **Contrastive pattern** (feature distinguishes class) | Yes — lift + CI + BH + strata (MR-06/11) | «يميّز» / «أكثر شيوعاً في» with counts both sides | «صيغ الأمر أكثر شيوعاً 3.2× في الجلسات الواضحة الخطوات» |
| **Model-inferred cause** | **No** (operational surfaces) | — | banned: «السبب الجذري لانخفاض الرضا هو…» as a model conclusion |
| **Agreement/disagreement between instruments** | Yes — as structure, not verdict | «اتساق/عدم اتساق بين المصدرين» — neither instrument declared "right" | D1/D2 wording |
| **Extraction about a government entity** | Yes — mention-report only | «كما ورد في الجلسات (مجرد استخراج من النص)»; no evaluative adjectives | C5 framing |

### 4.2 The doctrine, operationally

1. **«الأسباب كما ذكرها المستفيد» is a fixed string** in the renderer contract for C2 and any cause-bearing section; «الأسباب الجذرية» may appear only in the compound «الأسباب الجذرية كما صرّح بها المستفيد» and never unqualified.
2. **A stated cause is an extraction with a quote.** No quote passing R7 ⇒ no stated cause. The extractor's `stated_cause` field is filled only from beneficiary speech, never synthesized.
3. **Research escape hatch:** a genuinely causal question (e.g., "does consultant training reduce violations?") is answerable only under a separately approved methodology (design, identification strategy, owner sign-off) delivered as a Lane-3 research artifact clearly marked «دراسة بحثية — ليست تقريراً تشغيلياً», never merged into packs or capabilities [DECISION GREENFIELD §12.5].
4. **Trend language is movement language:** «ارتفع/انخفض» describes the measured rate; «بسبب» never follows it from the system. Composition caveats (MR-12) are the only explanatory device the system volunteers.

### 4.3 Renderer enforcement (structural, per I11)

The Arabic renderer holds a **banned-phrase lint** applied to composed narrative before display (composer output is already gated by R6/R7; this adds wording): banned unqualified — «السبب الجذري», «يسبب», «يؤدي إلى» *(as system inference)*, «الأسوأ», «تقصير الجهة», «فشل الجهة», «أداء الجهة ضعيف»; required substitutions — «كما ذكر المستفيد», «يرتبط بـ», «الأكثر ذكراً». A lint hit is `VERIFIER_REJECTED` → deterministic template render (fail-closed, R14). Golden fixtures include narratives engineered to trip each banned phrase (doc 15).

---

## 5. Semantic clustering methodology (R-P2)

One clustering machine serves five capability families — QST question clusters (B3/C6/C4/D8), CHAL challenge groups (C1/C2), DEC decision clusters (C8), SAT signal categories (A3), ENT entity resolution (C5, a constrained special case). It is the critical path for a third of the product [FACT MASTER_PROMPT E.15]; build it once.

### 5.1 Pipeline

```
finding text → Arabic normalisation (single shared normaliser: alef/ya/ta-marbuta unification,
               diacritics strip, tatweel removal, digit unification — NO stemming before
               embedding [REC: modern multilingual embedders handle inflection; stemming
               destroys dialect signal])
→ embedding   (model per EXP-04 — deferred; candidates below)
→ clustering  (per family, §5.2)
→ candidate clusters + representative members (medoids + highest-degree members)
→ REVIEW LOOP (ADR-0012): owner/steward names cluster → canonical_label_ar approved
               / merges / splits / rejects (append-only)
→ versioned artifact: findings.cluster_run + findings.cluster + cluster_member
               (embedding model id + dim + params_sha + corpus_snapshot recorded — I13)
```

**Embedding choice is deferred to EXP-04** [DECISION CORE-BRIEF §7 — Groq offers no embedding models; embeddings run locally, which keeps transcript text for retrieval inside the environment]. Candidates: `BAAI/bge-m3` (1024-dim, strong Arabic) vs `intfloat/multilingual-e5-large` vs legacy `paraphrase-multilingual-MiniLM-L12-v2` as baseline; decision metric = clustering quality on the labelled sets below, not retrieval benchmarks alone.

### 5.2 Clustering algorithms — candidates and recommendation

| Candidate | Fit | Verdict |
|---|---|---|
| **HDBSCAN on cosine distance** | finds variable-density clusters; leaves noise unassigned (honest — an unassigned question is *not* forced into a wrong cluster) | **Recommended for discovery runs** [REC] |
| Agglomerative (average linkage, distance threshold) | deterministic, explainable dendrogram for review UI | **Recommended for ENT alias grouping and small families (DEC, SAT)** where reviewability dominates |
| k-means | requires k; spherical assumption poor for text | rejected |
| LLM-only grouping | unbounded cost, unstable, violates determinism of committed paths | rejected as *mechanism*; allowed as a **labelling assistant** that proposes cluster names for the review queue |

Revisit-trigger: if EXP-04 shows HDBSCAN noise share > 40% on questions, fall back to agglomerative with tuned threshold + a second-pass noise assignment step. **Incremental assignment between discovery runs:** new findings are assigned to existing approved clusters by centroid similarity ≥ assignment threshold (calibrated in EXP-04); below it they pool for the next discovery run — R-P3's fresh-discovery guarantee is the *scheduled re-run per period*, where new clusters can emerge and are never suppressed by the existing vocabulary [DECISION GREENFIELD §6.3].

### 5.3 Quality metrics (EXP-04 exit criteria; measured per family)

- **Purity against labelled samples:** for each family, a 200-pair labelled set (same-meaning / different-meaning pairs, drawn stratified across programmes) → pairwise precision/recall of cluster co-membership. Targets [REC]: QST pairwise precision ≥ 0.85 at recall ≥ 0.70; ENT resolution precision ≥ 0.95 (precision-first — a wrong entity merge misattributes friction to the wrong government body).
- **Review workload:** clusters-per-1000-findings sent to the naming queue, and median reviewer minutes per cluster (measured in the pilot) — the mechanism is only viable if the queue is sustainable; if workload exceeds ~2 reviewer-hours/week per family, raise assignment thresholds and batch discovery runs quarterly [REC].
- **Coverage:** share of findings assigned to an approved cluster (published per MR-14 — e.g. C1's mapping coverage).
- **Stability across periods (§5.4).**

### 5.4 Cluster stability across periods

Comparability requires that «QST-0042» means the same thing in March and June. Mechanisms:
1. **Canonical clusters are versioned taxonomy objects** (`findings.cluster` with approved labels; edges through the review loop; lineage on merge/split — ADR-0011/0012).
2. **Membership churn metric per period:** Adjusted Rand Index between consecutive discovery runs restricted to the shared finding set, plus per-cluster membership turnover. ARI < 0.75 on a family flags an unstable vocabulary → freeze the previous version for reporting and route the diff to review before adoption [REC; revisit after 3 stable periods].
3. **Left-censoring (MR-13)** applies to newly approved clusters exactly as to taxonomy categories.
4. **Embedding-model change = new `cluster_run` lineage:** clusters re-derived, crosswalk proposed cluster-to-cluster by member overlap (≥ 0.8 Jaccard auto-proposes; below that human review), and packs' `RECOMPUTE_REQUIRED` statistics invalidate (§3.25).

### 5.5 ENT special case — entity resolution is matching, not clustering

Government entities have a **finite true registry**; the task is alias resolution, not discovery of unknown concepts. Pipeline: exact/normalised match against `tax.entity_alias` → fuzzy candidates (embedding + edit distance) scored against registry entries → auto-accept above high threshold, queue 0.75–0.9 band for steward review, leave below unresolved [REC thresholds calibrated in EXP-04 on a 200-mention labelled sample]. New *entities* (not just aliases) enter the registry only through review. Precision-first per §3.13.

---

## 6. Coverage block — definition and named-exclusion vocabulary

Every analytical answer, pack section, and Lane-3 result carries one coverage block [DECISION GREENFIELD §12.7; I8; carried forward from the legacy Phase-5 contract which is the one legacy behaviour worth keeping [FACT MASTER_PROMPT §"Phase 5"]].

### 6.1 Definition and identities

```json
{
  "total":   16911,        // population in scope class (e.g., all sessions all-time, or all in period)
  "matched": 1240,         // sessions matching the question's filters/period
  "used":    1150,         // sessions actually contributing to the computation
  "exclusions": [
    {"code": "no_transcript",        "n": 41, "label_ar": "جلسات بلا نص متاح"},
    {"code": "speaker_role_missing", "n": 22, "label_ar": "أدوار متحدثين غير محسومة"},
    {"code": "label_missing",        "n": 27, "label_ar": "تصنيف الوضوح غير متوفر"}
  ],
  "provider_distribution": {"read_ai": 1108, "provider_x": 42},   // active-transcript sources
  "as_of": "2026-08-02T10:00:00Z",
  "period": {"start": "2026-07-01", "end": "2026-08-01", "end_exclusive": true}
}
```

**Invariants (asserted in the regression suite on every golden answer):**
- `used ≤ matched ≤ total`
- `used + Σ exclusions[].n = matched` — every excluded session is excluded *for a named reason*; there is no anonymous gap.
- The block is computed by the harness from query metadata (I3/I18); a model never writes it; `check_scope_declared` failure raises.
- Exclusion codes come from the closed vocabulary below (adding a code = registry change, I12). `n` counts **sessions** (MR-02); finding-grain exclusions state their grain explicitly.

### 6.2 Named-exclusion vocabulary (closed enum, v1)

| Code | Arabic label | Meaning / typical capabilities |
|---|---|---|
| `no_transcript` | جلسات بلا نص | session matched but no transcript ingested — all transcript capabilities |
| `transcript_quality_below_gate` | جودة النص دون الحد | source-quality score below the doc-06 gate; blocked or qualified use |
| `speaker_role_missing` | أدوار المتحدثين غير محسومة | turns lacking speaker attribution (legacy baseline 5.7%) — A2, B4, C7 |
| `provider_unmatched` | جلسات المزود غير المطابقة | provider meeting with no internal session — reconciliation (D9) |
| `internal_unmatched` | جلسات داخلية غير مطابقة | internal session with no provider meeting — D3, D4 |
| `consultant_unresolved` | مستشار غير محسوم الهوية | `resolution_status` ≠ resolved (never a magic string) — consultant-grouped answers |
| `rating_unbridged` | تقييم غير مرتبط بجلسة | rating row bridges to no session — D1, D5 |
| `rating_missing` | لا تقييم للمستفيد | bridged session without a rating — D1, D2 |
| `outcome_unavailable` | بيانات الإتمام غير متوفرة | follow-up requirement without outcome tracking — D7 |
| `label_missing` | التصنيف غير متوفر | required label null for the session (e.g. clarity null) — B1, A1 |
| `label_partial_excluded` | فئة «جزئي» خارج المقارنة | deliberate contrast-design exclusion — A1 |
| `label_not_validated` | التصنيف قيد التحقق | MR-10 gate not yet passed; capability-level `DATA_NOT_ENRICHED` |
| `not_yet_reclassified` | بانتظار إعادة التصنيف | findings stamped at older taxonomy version, excluded from live view [FACT MASTER_PROMPT §4.3] |
| `detection_gap` | فئة بلا كاشف بعد | category named by owner, detector not shipped (تهكم، تسويق شخصي) — B4/B5 |
| `entity_unresolved` | جهة غير مُسندة | mention not resolved to canonical entity — C5 |
| `cluster_unassigned` | غير مُسند لمجموعة | finding not in any approved cluster — B3/C6/C8/C1 mapping coverage |
| `review_pending` | قيد المراجعة | candidate awaiting review — accusation-class surfaces |
| `review_rejected` | مرفوض بعد المراجعة | reviewed and rejected — B4/B5, D5 |
| `dropped_unverifiable` | اقتباس غير قابل للتحقق | R7 gate failed — Lane-3 and extraction pipelines |
| `partition_incomplete` | قسم تحليل غير مكتمل | Lane-3 partition with failed calls — `DEEP_JOB_INCOMPLETE` answers |
| `below_min_support` | دون حد العرض | cells suppressed by MR-04 (count shown, rate suppressed) |
| `left_censored` | قبل بدء القياس | periods before a category/detector existed (MR-13) |
| `status_unmapped` | حالة تشغيلية غير مُفسّرة | internal status string not in the mapping table — D3, D4 [ASSUME OD-07] |
| `pending_ingestion` | بانتظار الاستيعاب | source rows behind the freshness watermark — all, via doc 09 |

Rendering rule: the top-3 exclusions by `n` always render in the answer body; the full list is one click away (doc 18). Every code has a fixed Arabic string — coverage language is product surface, not logs.

### 6.3 Worked Arabic rendering

The B1 example of §3.4 shows the inline form. The expanded form (answer footer / pack coverage page):

> **نطاق التحليل والتغطية**
> الفترة: 2026-07-01 إلى 2026-08-01 (نهاية غير مشمولة) · محسوبة بتاريخ 2026-08-02
> إجمالي الجلسات: 16٬911 · المطابقة لنطاق السؤال: 1٬240 · المستخدمة في الحساب: 1٬150
> **المستبعدات (مسمّاة):** 41 جلسة بلا نص · 22 أدوار متحدثين غير محسومة · 27 تصنيف الوضوح غير متوفر
> **توزيع مصادر النصوص:** Read.ai: 1٬108 · مزود بديل: 42
> التحقق: 1٬150 + (41+22+27) = 1٬240 ✓

The identity line renders in debug/reviewer views and is asserted always; user-facing views render the counts without the arithmetic. A Lane-3 answer adds «أقسام غير مكتملة: 1 من 14 (يوليو 2025) — النتيجة معلَّمة كغير مكتملة» (never silently complete — I16, `DEEP_JOB_INCOMPLETE`).

---

## 7. Statistical appendix

Deterministic implementations live in the shared analytical library (one implementation per §1.1); models never compute any of these (I3).

### 7.1 Wilson score interval (every proportion, MR-05)

For successes `k` of `n`, `p̂ = k/n`, `z = 1.96`:

```
centre = (p̂ + z²/2n) / (1 + z²/n)
half   = (z / (1 + z²/n)) · sqrt( p̂(1−p̂)/n + z²/4n² )
CI     = centre ± half
```

Worked: k=40, n=100 → p̂=0.40, CI ≈ (0.309, 0.498) — rendered «40% (31–50%)». Preferred over normal approximation because it behaves at small n and near 0/1 (violation rates) [REC — standard choice; alternative Jeffreys interval acceptable; do not mix methods across the product].

### 7.2 Two-proportion comparison (B1, C3, D4, D6)

Primary: two-proportion z-test on rates `p̂1 = k1/n1`, `p̂2 = k2/n2` with pooled SE; report the difference with its CI (unpooled SE for the interval). **Switch to Fisher's exact test** when any expected cell < 5 or either class count < 10 [REC]. Never test suppressed cells (MR-04 first). All scan-style comparisons feed MR-11 before rendering.

### 7.3 Poisson / rare-event rate test (B5, D4 rare cells)

For event counts `X1, X2` over exposures `n1, n2` sessions (violation-type occurrences): conditional binomial (e-test): under H₀ equal rates, `X1 | X1+X2 ~ Binomial(X1+X2, n1/(n1+n2))`; exact p-value from the binomial tail. Used when counts < 30 per side, where the z-test misbehaves. Rate CIs for rare events: exact Poisson (Garwood) intervals on the count, scaled per 100 sessions.

### 7.4 Contrastive lift (A1, C7)

For feature `f`, classes `A` (n₁ sessions, `a` with `f`) and `B` (n₂ sessions, `b` with `f`):

```
lift = (a/n₁) / (b/n₂)
SE(log lift) = sqrt(1/a − 1/n₁ + 1/b − 1/n₂)
CI = exp( ln(lift) ± 1.96·SE )
```

Gates, in order: (1) **support** — `f` present in ≥ `MIN_SUPPORT_PATTERN` sessions of the smaller class; (2) Fisher exact p per feature; (3) **BH across the full feature set** (§7.5); (4) **confound strata** (MR-06) — recompute lift within each stratum of consultant/programme/service-category/length; report `survives_strata`; (5) render lift + CI + q + raw counts both sides. Zero-cell handling: Haldane–Anscombe +0.5 only for CI display, never to pass the support gate.

### 7.5 Benjamini–Hochberg procedure (MR-11) — worked example

Sort m p-values ascending; find largest k with `p(k) ≤ (k/m)·q`; reject hypotheses 1…k; report q-values `q(i) = min over j≥i of (m/j)·p(j)`.

Example, m=10 features, q=0.05: p = [0.001, 0.004, 0.008, 0.012, 0.030, 0.041, 0.060, 0.210, 0.550, 0.900]. Thresholds (k/m)·q = [0.005, 0.010, 0.015, 0.020, 0.025, 0.030, …]. Largest k with p(k) ≤ threshold: k=4 (0.012 ≤ 0.020; 0.030 > 0.025). Features 1–4 publishable; features 5–6, "significant" uncorrected, are exactly the false discoveries the rule exists to stop.

### 7.6 Class-imbalance rules (C7 and any pattern label)

A pattern claim requires the **smaller class ≥ 300 sessions** in the analysis window [REC — with MIN_SUPPORT_PATTERN=30 and typical feature prevalences, smaller classes make lift estimates degenerate; revisit with power analysis §7.11]. Below that: (a) merge into a coarser class **with the merge stated on the answer's face** («دُمجت فئتا الضيق الشديد وأخرى لقلة العدد»), or (b) counts-with-quotes only, explicitly marked «عدد محدود — لا يُستخلص منه نمط». Applied to the planning baselines: distress 544 qualifies marginally in a full-year window only; `other` 279 never qualifies alone [FACT MASTER_PROMPT E.13].

### 7.7 Agreement statistics for label validation (MR-10) and D1/D2

- **Binary/nominal labels:** raw agreement + Cohen's κ with bootstrap CI (1,000 resamples).
- **Ordinal labels** (rating bands, clarity clear/partial/none): **linear-weighted κ**.
- **>2 adjudicators:** Krippendorff's α as secondary.
- Publication thresholds per MR-10 (κ ≥ 0.70 unlock; 0.50–0.69 caveat, no trend claims; < 0.50 blocked). The validation record stores the confusion matrix, not just κ — the *direction* of label errors feeds capability caveats (e.g., if `clear` over-triggers on long sessions, B1 carries that specific caveat).

### 7.8 Interval-merge speaking time (B2 — the legacy `silence_pct` fix, normative)

```
def speech_stats(turns):                     # turns: (speaker, start_s, end_s)
    per_speaker = {s: merge(sorted(intervals)) for s, intervals in by_speaker(turns)}
    union_all   = merge(sorted(all_intervals(turns)))       # overlaps collapse once
    session_span = max(end) - min(start)
    speech_time  = total_len(union_all)
    silence_time = session_span - speech_time                # ≥ 0 BY CONSTRUCTION
    talk_share   = {s: total_len(iv) / speech_time for s, iv in per_speaker}
    overlap_time = sum(total_len(iv) for iv in per_speaker.values()) - speech_time
    assert silence_time >= 0                                  # invariant, not a clamp
    return speech_time, silence_time, talk_share, overlap_time

def merge(sorted_intervals):                 # standard sweep
    out = []
    for s, e in sorted_intervals:
        if out and s <= out[-1][1]: out[-1][1] = max(out[-1][1], e)
        else: out.append([s, e])
    return out
```

`overlap_time` is reported as its own diagnostic (cross-talk is itself a churn signal). The legacy code summed per-speaker durations, went negative on cross-talk, and clamped to zero with a warning — the clamp is the tell that the model was wrong [FACT MASTER_PROMPT E.5].

### 7.9 Answer-dispersion measure (B3/C6)

For cluster `c` with answer set `A = {a₁…a_k}` (embedded, length-normalised):
- `dispersion(c) = mean pairwise cosine distance over A`, computed on embeddings of answers **truncated/pooled to comparable length** (length is a nuisance variable: long answers drift apart lexically while agreeing substantively).
- **Decomposition:** `dispersion = within_consultant + between_consultant` (ANOVA-style split on the pairwise matrix by whether the pair shares a consultant). `between/within ≥ 2` ⇒ «اختلاف بين المستشارين» (training/alignment); otherwise «تشتت عام» (knowledge gap) [FACT MASTER_PROMPT E.6-6].
- Secondary spread measure over structured advice extracted from answers (categorical mode share) reported beside the geometric measure.

### 7.10 Sensitivity analysis (B2 thresholds; GREENFIELD §5-B2 "no arbitrary threshold")

For each calibrated constant (`DEAD_AIR_FLOOR_S`, churn run length, drift threshold): compute the headline result at threshold × {0.5, 0.75, 1.0, 1.25, 1.5}; publish the elasticity («ترتيب مواضع الفقد ثابت عبر النطاق 15–30 ثانية») in the method note. A ranking that reorders materially within the band is not publishable as a ranking — render per-mode totals without ordinal claims and flag for re-calibration.

### 7.11 Support planning and bootstrap CIs

Quick reference (two-proportion, 80% power, α=0.05 two-sided): detecting 10 p.p. difference around 40% needs ≈ 385/side; 5 p.p. needs ≈ 1,565/side. Consequence: **monthly** windows (556–1,512 sessions) support only large-effect claims on whole-corpus splits; fine segment trends belong in **quarterly** windows — the trend engine picks the window per requested granularity and says so. Non-analytic statistics (hesitation index, dispersion): bootstrap CIs, 1,000 resamples at the session level (resample sessions, not findings — findings within a session are correlated).

### 7.12 Label-validation sampling design (MR-10, per GREENFIELD §12.3's eight families)

One protocol, eight families. Sampling is **stratified random** (never convenience, never model-confidence-ordered — confidence-ordered samples flatter the model). Adjudication runs through the single review loop (ADR-0012) with a written rubric per family, signed by the owner before the round starts; rubric changes void prior rounds for that family. Two adjudicators per item where feasible; disagreements resolved by the owner and **counted in the published agreement** (adjudicator-vs-adjudicator κ is reported beside model-vs-human κ — if humans cannot agree, the label definition, not the model, is the problem).

| # | Label family | Unit sampled | n (initial round) | Strata | Notes |
|---|---|---|---|---|---|
| 1 | impact / clear-step labels (`step_clarity`) | session (full transcript end) | 100 per value = 300 | programme × length quartile | strongest gate — B1 headline (§3.4) |
| 2 | satisfaction classes (polarity) | signal-in-context | 100 per polarity = 300 | explicit vs implicit; programme | implicit state-transition flags validated separately |
| 3 | violation classes | candidate case | 100 per detector source (rule/extractor) + all VIOL-004 candidates | violation type | precision emphasis: adjudicate published-eligible cases first |
| 4 | government-friction mentions (`is_friction` + entity resolution) | mention | 200 mentions | resolved/unresolved; head/tail entities | entity-resolution precision target ≥ 0.95 |
| 5 | speaker roles | turn | 400 turns | provider source; session position | gates A2/B4/C7 exclusion accounting |
| 6 | repeated-question clusters | question pair | 200 pairs (same/different meaning) | head/tail clusters; programme | drives §5.3 pairwise purity |
| 7 | decision clusters | decision-point pair + question attribution | 200 pairs + 100 attributions | decision frequency band | gates C8's index |
| 8 | provider transcript quality | session sample (doc 06 rubric) | 100 per provider | programme; audio condition where known | feeds the doc-06 quality gate, not an analytics label |

Cadence: initial round before the dependent capability publishes; re-run on model/prompt/taxonomy change (records are keyed to `model_id + prompt_sha + taxonomy_version` and void automatically); steady-state annual re-validation for stable families (MR-10 revisit rule).

---

## 8. Build order across the method set

Dependencies, not preferences (adapted from MASTER_PROMPT E.15 to the new platform; feeds doc 21):

| Order | Build | Unlocks |
|---|---|---|
| 1 | Shared analytical library: constants registry (MR-16), Wilson/tests/BH utilities, coverage-block assembly, interval-merge speech stats (§7.8) | everything; B2 correctness from day one |
| 2 | **R-P2 clustering artifact + review/naming loop** (§5, ADR-0012 UI) | A3, B3/C6, C1, C4, C5(ENT), C8, D8 — **seven of 26 capabilities**; the critical path |
| 3 | Contrastive-lift utility with support/Fisher/BH/strata as one call (§7.4) | A1, C7 |
| 4 | Canonical government-entity registry + alias resolution (§5.5) | C5 |
| 5 | MR-10 label-validation rounds: step_clarity, satisfaction polarity, violation classes, friction flag, speaker roles, cluster assignments, transcript quality (GREENFIELD §12.3 list) | B1 headline, A1 label, A2/A3, B4/B5; unblocks trend claims |
| 6 | OD-05 execution: C2 Lane-3 pilot → corpus extraction with populated fields | C2 |
| 7 | Violation review queue end-to-end (candidates → reviewed → published) | B4/B5 publication; D5 violation axis |
| 8 | Internal-data contracts landed (OD-07): status/attendance/follow-up enums mapped | D3, D4, D7; D1/D2 at full coverage |

Item 2 starts first; it needs owner time (naming queues), not just engineering time [FACT MASTER_PROMPT E.15].

**Experiment gating (GREENFIELD §21 → this charter):**

| Experiment | Gates which method decisions here |
|---|---|
| EXP-01 source linkage | D1/D2 bridged-share baselines; `provider_unmatched`/`internal_unmatched` expected magnitudes |
| EXP-02 provider benchmark | family 8 of §7.12 (transcript-quality labels); `transcript_quality_below_gate` threshold |
| EXP-03 extraction model benchmark | which model produces each finding family; MR-10 rounds re-run per winning model |
| EXP-04 semantic clustering | embedding model, clustering algorithm per family, assignment thresholds, `NEAR_DUP_COSINE`, dispersion calibration (§3.6), ENT thresholds (§5.5) |
| EXP-05 committed-question routing | not method — but confusable pairs (A3↔D1, B3↔C6) come from §2's class table |
| EXP-06 metric compiler correctness | MR-01/02 enforcement (grain-derived DISTINCT); suppression post-processor |
| EXP-07 evidence retrieval | quote-search quality; does not gate any number (I9) |
| EXP-08 deep-analysis job | Lane-3 coverage identity incl. `dropped_unverifiable`, `partition_incomplete` |
| EXP-09 provenance gates | R6/R7 + §4.3 banned-phrase lint proven to reject |
| EXP-10 adversarial | prompt-injection isolation of extraction inputs — protects every label this charter builds on |

## 9. Open decisions and revisit triggers touching this charter

| Item | Where | Status |
|---|---|---|
| C2 route (Lane-3 pilot then corpus extraction) | §3.10 | [ASSUME OD-05] — recommendation stated; owner confirms after pilot |
| Internal enum semantics + freshness (status/attendance/outcome) | §3.18–3.19, §3.22 | [ASSUME OD-07] |
| Publication authority for packs | §3.25 | [ASSUME OD-10] |
| Reviewer powers / append-only history | §3.7, §4 | [ASSUME OD-11] |
| Embedding model + clustering thresholds | §5 | deferred to EXP-04 by design |
| κ thresholds (0.70/0.50), FDR q (0.05/0.10), TVD 0.10, smaller-class ≥ 300 | §1, §7.6 | [REC] with stated revisit triggers — revisit after two full validation cycles |
| Dispersion quartile calibration | §3.6 | calibrated on first corpus snapshot; frozen per version; recalibrate on embedding change |
| Review SLA values (7d/2d) + reviewer staffing/assignment | MR-17 queue reporting; doc 15 QT-14 | [ASSUME OD-30] |
| False-negative sample size/cadence (weekly stratified) | MR-18; doc 15 DS-14 | [ASSUME OD-32] — EXP-11 owns the design |
| Month completeness gate ≥98% + nightly deadline | MR-20; CAP-OPS-08 trigger | [ASSUME OD-28 — Amendment §13 working assumption] |
| Action Center ownership + closure authority | MR-19 readings | [ASSUME OD-34] |
| Detector release cadence | MR-18 floor application per release | [ASSUME OD-33] — doc 15 §6.6 evidence standard |

Every [REC] threshold above is a starting point with a named recalibration trigger — none is a bare TBD, and none may be changed outside the constants registry's review path (MR-16).

---
*End of document 10. The per-capability registry entries (doc 04) cite this document's sections as `doc10#CAP-xx`; the semantic layer (doc 11) implements MR-01…MR-05 and §6 as typed contracts; the evaluation plan (doc 15) turns every golden fixture named here into a suite item.*

# 25 — Incremental Delivery and Acceptance (التسليم المرحلي والقبول — الشرائح الرأسية)
**Platform:** Nwafeth Intelligence — منصة نوافث لذكاء الجلسات الاستشارية (Monsha'at Advisory Session Intelligence Platform)
**Status:** Draft for owner review · **Date:** 2026-08-03 · **Author:** Planning package (Fable 5)
**Depends on:** 04 (CAP/CAP-OPS registry), 08 (schemas), 09 (pipelines), 15 (DS/QT/HG gates), 16 (security), 17 (API), 18 (SCR-*), 21 (phases P0…P7 + EXP gates), 22 (ODs), 23 (EPIC-01…36, migrations 0001…0012), 24 (PB cards + SD-21 governance) · **Feeds:** 00, 21, 23
**Sources used:** Owner Amendment 2026-08-03 §9 (slices, exposure, stop rule), §10 (ticket protocol carried in full meaning), §4.6/§5.7/§7.7 (acceptance sources), §6.4 (EXP-11), §13 (working assumptions), §15 (start order); CORE-BRIEF §3, §13; docs 21 §0–§4, 23 §5–§6

---

## 0. The delivery model — how phases and slices coexist (binding)

Doc 21's phases **P0…P7 and the experiment gates EXP-01…EXP-10 REMAIN the dependency and gate framework — they say what must be true.** The **execution overlay becomes vertical slices VS-00…VS-12 — they say what gets built, shown, and accepted, in order** [DECISION owner 2026-08-03 / Amendment §9; SD-18, doc 02 register]. Each VS maps to a subset of phase deliverables/epics; a VS may pull forward the *minimal* slice of a later phase's machinery (e.g. VS-05 pulls a minimal pack-render path forward from P6) **provided its L2/L3 exposure still waits for the owning phase's gates**. EPIC-01…20(+20b) in doc 23 remain the ticket inventory, but they are re-sequenced and **split into ≤4–8h tickets under the VS order** (§4.1). The first executable order after plan approval is **VS-00 only — then stop, demo, accept; only then VS-01** [DECISION Amendment §15]. This document owns the protocol; docs 21 and 23 point here.

**Resolved conflict #1** [DECISION Amendment §0.9; recorded in doc 02]: the earlier package sentence "nothing user-facing before EXP-09/EXP-10" (doc 21 §1-4) is **superseded in scope**: production-grade results for Pilot/Production users (L2/L3) still wait for their gates, **but internal owner previews (L1) happen after every slice** on synthetic or governed staging samples, precisely so gaps surface early. No invariant is weakened by this: I1–I18 and the R-gates apply at every level; L1 merely sees them applied to synthetic data.

Working directives that shape every slice below [DECISION Amendment §0.6–0.7, §1.2]:

- Build a small complete path from source to screen; test it; accept it; then add **one complete feature at a time**, keeping the system runnable after every step.
- Never build all 26 analytical capabilities before an initial product works end-to-end and the owner has tested it.
- No fully-parallel multi-epic development by one agent (SD-18; §4.4-1).

---

## 1. Exposure levels L0–L3 [DECISION Amendment §9.2]

| Level | البيانات | الجمهور | البوابات |
|---|---|---|---|
| **L0 Developer** | Synthetic only | المطور/الوكيل البرمجي | Unit + component tests green |
| **L1 Owner Preview** | Synthetic أو عينة Staging محدودة ومحكومة | المالك وفريق المشروع | Feature acceptance للشريحة؛ **لا تُتخذ قرارات تشغيلية من L1** |
| **L2 Controlled Pilot** | بيانات حقيقية محددة | مستخدمون مصرح لهم بأسمائهم | بوابات الدقة والأمن الخاصة بالميزة (EXP المرتبط + فحوص doc 16) |
| **L3 Production** | البيانات الإنتاجية | الجمهور الداخلي المعتمد | Full release gates + rollback + ops readiness (doc 21 phase exit) |

**The two rules (binding, cited by every slice):**

1. **L3 gate requirements must never be used to block an L1 preview.** EXP-09/EXP-10 and phase-exit criteria guard L2/L3 exposure only [DECISION Amendment §9.2; SD-18].
2. **An L1 pass never authorizes L2 or L3.** Moving a feature up a level is a separate, explicit owner decision against that feature's own gates; "the owner saw it and liked it" is not a gate.

Enforcement mechanics:

- Exposure level is a per-flag, per-environment server-side property (flag → allowed audience + data class), audited on change (doc 16 audit events).
- A flag turned on in staging (L1) stays off in production until an L2/L3 decision record exists in the acceptance artifact.
- L1 previews carry a visible banner naming the data tier («بيانات اصطناعية/عينة تجريبية — ليست أرقامًا تشغيلية»).

---

## 2. The mandatory stop rule — after EVERY slice [DECISION Amendment §9.4; SD-18]

After each vertical slice, in order, no exceptions:

1. **The implementation agent stops.** No new slice work, no "preparatory" tickets for the next slice.
2. **It presents what was built** — the demo per the slice's script, on the slice's data tier.
3. **It runs the slice's acceptance tests** and attaches the results (commands + output, not claims).
4. **It records the actual gaps** in `KNOWN_GAPS.md` — real state, not aspiration; every TODO behind a closed flag is listed.
5. **It obtains the owner's transition approval** — a dated, named decision.
6. **It never starts the next slice automatically.** Absence of a response is a stop, not a yes.

**Approval artifact:** `docs/acceptance/VS-xx-acceptance.md`, containing:

- slice id + the demo record (date, attendees, data tier used);
- the acceptance checklist with per-item pass/fail and evidence links (test output, screenshots);
- gaps accepted, each with its `KNOWN_GAPS.md` id and flag state;
- the owner's decision — `approve` / `approve-with-conditions` / `rework` — with identity and date;
- the consultation outcome for the next slice's candidate content (PB ids + scope trims, per doc 24 §0 rule 3);
- the exposure-level decision, if any surface moves beyond L1 (rule 2 above).

`docs/STATE.md` (§5) is updated in the same change set. A slice without this artifact is, by definition, not delivered.

---

## 3. The slice catalogue VS-00…VS-12 [DECISION Amendment §9.3]

Every slice includes only what it needs of the Amendment §9.1 list: **Migration · Backend/domain · API · UI · Fixture · Automated tests · Manual demo script · Observability · Rollback · Documentation.** "Mapped EPICs" cite doc 23 §5 ids; phase deliverables cite doc 21 §3; PB bindings cite doc 24; SCR ids cite doc 18.

### VS-00 — Walking Skeleton — الهيكل السائر

**Goal:** a running, secured, observable application shell: login + roles, baseline migrations, CI with boundary enforcement, logging, Home page, health/readiness, PII-free seed fixture.

**In scope:** repo scaffold; migrations 0001 + 0002a subset; OIDC + 7 roles + deny-by-default; honest health; version/build page; minimal RTL home; synthetic seed.
**Out of scope:** any domain data (sessions, transcripts); any detector; any dashboard number; the full MinIO/Grafana/Loki/pgBackRest stack (declared gap → VS-03); production IdP wiring beyond the broker assumption [ASSUME OD-02].

**What it needs (§9.1):**
- *Migration:* 0001 + `0002a_ops_audit_minimal` (§7 T-VS00-2).
- *Backend/domain:* app factory, auth middleware, status aggregation.
- *API:* probes; `GET /api/v1/meta/version`; `GET /api/v1/me`; `GET /api/v1/meta/status`.
- *UI:* login → RTL home shell with live status cards.
- *Fixture:* `seed_manifest` + demo audit rows — PII-free by construction.
- *Automated tests:* G-SEC-1 route walk; migration cycle; healthcheck-lies regression; seed idempotency; PII scan.
- *Manual demo script:* below; authored as `docs/demos/VS-00-demo.md` (T-VS00-5).
- *Observability:* structlog JSON + request-id on every line.
- *Rollback:* `alembic downgrade base` + revert deploy.
- *Documentation:* `docs/STATE.md`, `PRODUCT_BACKLOG.md`, `KNOWN_GAPS.md` seeded.

**Demo script (يؤديه المالك):**
1. افتح رابط بيئة Staging وسجّل الدخول عبر الهوية الموحدة.
2. لاحظ الصفحة الرئيسية: اسمك ودورك، نسخة البناء، حالة قاعدة البيانات، حالة بيانات العينة — كلها قيم حية.
3. افتح مسارًا إداريًا بدور «محلل» — تظهر رسالة منع صلاحيات بالعربية (403).
4. يطفئ المهندس قاعدة البيانات مؤقتًا — تتحول صفحة الحالة إلى «غير جاهز» خلال ثوانٍ، ثم تعود خضراء بعد التشغيل.
5. اطلب من الوكيل عرض نتيجة CI الأخضر وإثبات فشل بناءٍ زُرع فيه استيراد محظور.

**Acceptance checklist:**
- [ ] Local/staging run via a single documented command; CI green (lint → type → import-contracts → unit → structural).
- [ ] No default credentials anywhere; secret scan green.
- [ ] The two probes are the only anonymous routes (G-SEC-1 walk passes).
- [ ] Fixture contains zero PII (P3-regex + roster deny-list scan in CI).
- [ ] Status page values come from live checks — stopping Postgres flips `/readyz` red (no static 200s, I16).
- [ ] `0001 → head → base` migration cycle green in CI; `ops.audit_event` rejects UPDATE at the DB level.

**Stop gate:** owner approval in `docs/acceptance/VS-00-acceptance.md` + consultation on VS-01 content (PB-003/004/005).
**Mapped EPICs / phase deliverables:** thin slices of EPIC-01, EPIC-02, EPIC-03, EPIC-04 (+0002a of EPIC-05) = doc 21 P0-E1/E2/E4/E5 subset.
**Flags touched:** none exposed — all seven created, default OFF. · **Exposure:** L0 → L1 (synthetic).

### VS-01 — أول جلسة موحدة من المصدرين — first unified session from both sources

**Goal:** ingest 20 fixture sessions from Read.ai and 20 records from the DataHub family, link them, and show Session 360 with per-field provenance.

**In scope:** `ingest`/`core`/`transcript` DDL subsets; SRC-READAI fixture-mode adapter; SRC-DATAHUB-SESSION/-BENEFICIARY-EVAL/-CONSULTANT-EVAL/-DIRECTORY/-REFERENCE fixture adapters; deterministic identity rungs M1/M2 + a visible unmatched list; SCR-15 «الجلسة 360»; SCR-16 minimal search.
**Out of scope:** scheduled/nightly orchestration (VS-03); scored M3 matching + steward queue UI (VS-03); enrichment/detection (VS-02); live OD-07/OD-13 credentials (fixtures only); SRC-DATAHUB-OUTCOME (no consumer until Wave B).

**What it needs (§9.1):**
- *Migration:* 0004/0005/0006 subsets (doc 23 §6 numbering preserved).
- *Backend/domain:* adapters with run ledger + DLQ; crosswalk builder; M1/M2 resolver.
- *API:* `GET /sessions/{id}/360`; session search endpoint (doc 17 amendment rows).
- *UI:* SCR-15 with per-field source badges; SCR-16 minimal filters.
- *Fixture:* 20+20 authored records including edge cases — unmatched ×2, duplicate ×1, late-evaluation ×1.
- *Automated tests:* idempotent re-ingest; crosswalk correctness; RBAC on evaluations; authority-matrix badge test.
- *Manual demo script:* below.
- *Observability:* per-adapter run ledger rows + counts.
- *Rollback:* `session_360_enabled` off + downgrade of the three migrations.
- *Documentation:* crosswalk explainer inside Session 360 help.

**Demo script (يؤديه المالك):**
1. افتح «بحث الجلسات» وصفِّ بالفترة والبرنامج.
2. افتح جلسة واحدة: شاهد النص، بيانات الخدمة، تقييم المستفيد وتقييم المستشار، وشارة مصدر كل حقل.
3. افتح قائمة «غير المطابق» وشاهد جلستين بلا نظير مع السبب المصنف.
4. اطلب إعادة الاستيراد أمامك — الأعداد لا تتغير.
5. بدّل إلى دور بلا صلاحية التقييمات — تختفي الملاحظات النصية من الشاشة نفسها.

**Acceptance checklist:**
- [ ] Zero duplicates (natural-key + payload-sha idempotency test green).
- [ ] Session Crosswalk explicit: provider meeting id ↔ `advisory_session_id` ↔ internal session id + resolution status.
- [ ] Unmatched cases visible with typed reasons — no third, silent state (ADR-0006 doctrine).
- [ ] Re-ingest changes no counts at the UI-visible level.
- [ ] Per-field source badges match the doc 05 authority matrix (DataHub governs session facts; provider governs transcript).
- [ ] Absent data renders «غير متوفر» with reason — never a silent zero or empty box.
- [ ] Evaluation visibility per role matrix; every transcript open is access-audited.

**Stop gate:** VS-01 acceptance artifact; **first Owner Preview of record** [DECISION Amendment §13 — first L1 after VS-01]; consultation for VS-02.
**Mapped EPICs / phase deliverables:** EPIC-10 / EPIC-11 / EPIC-12 (restructured to the SRC-DATAHUB family) / EPIC-13 (M1–M2 only) / EPIC-15 subsets = P1-E1…E6 slice.
**Flags touched:** `session_360_enabled` (staging on). · **Exposure:** L1 (synthetic).

### VS-02 — أول مخالفة من المصدر إلى قرار بشري — VIOL-008 end-to-end

**Goal:** one violation type — VIOL-008 «التواصل خارج الإطار الرسمي» [ASSUME OD-35] — flows ingest → detect → «اشتباه مخالفة» → human decision → append-only audit; the committed-early cluster PB-006…PB-010 lands [DECISION SD-20/SD-21].

**In scope:** `findings`/`tax` DDL subsets with the `violation_finding` / `review_case` / `review_event` separation + VIOL-001…008 seed; deterministic VIOL-008 rules + LLM verifier; SCR-08 **upgraded** («اشتباه مخالفة», the fixed disclaimer, ±5 لفات, the six decisions, reopened-by-change filter); SCR-17 missed-case report (PB-010 UI).
**Out of scope:** any second violation type (one at a time — SD-20); detector learning (VS-04); nightly scheduling (VS-03); any leadership-facing number.

**What it needs (§9.1):**
- *Migration:* 0007/0008 subsets + `0002b_ops_registries` remainder.
- *Backend/domain:* VIOL-008 detector (rules + verifier) with version stamps; case/finding services; fingerprint mechanics.
- *API:* `GET /violations/suspected`; `GET /violations/cases/{id}`; `POST /violations/cases/{id}/review-events`; `POST /sessions/{id}/missed-violation` (doc 17 amendment rows).
- *UI:* SCR-08 upgrade (queue + case detail) + SCR-17.
- *Fixture:* sessions with planted VIOL-008 phenomena + clean negatives + one session carrying two findings.
- *Automated tests:* R7 quote gate on render; append-only structural; RBAC deny probes; fingerprint invalidation; two-findings independence.
- *Manual demo script:* below.
- *Observability:* detector run metrics; queue counts; `suspected_cases_open`.
- *Rollback:* `violation_review_enabled` off — findings retained, invisible.
- *Documentation:* reviewer guide covering the six decisions and reason codes.

**Demo script (يؤديه المراجع أمام المالك):**
1. افتح «اشتباه مخالفة» — لاحظ العبارة الثابتة أعلى الصفحة: «الحالات في هذه الصفحة مؤشرات آلية تحتاج مراجعة بشرية، ولا تعد مخالفة مثبتة قبل اعتمادها.»
2. افتح حالة: اقرأ التعريف الرسمي، الاقتباس مع ±5 لفات، وانتقل إلى موضعه في النص الكامل.
3. سجّل قرار «مخالفة صحيحة» مع رمز السبب؛ ثم على مؤشرٍ آخر في الجلسة نفسها سجّل «ليست مخالفة».
4. أعد فتح الحالة — القراران ظاهران في سجل غير قابل للحذف، والحالة المعتمدة خرجت من قائمة «مشتبه».
5. من «الجلسة 360» أضف «اشتباهًا لم يرصده النظام» على مقطع محدد وشاهده يدخل قائمة المراجعة الثانية.
6. جرّب الوصول للقائمة بدور بلا صلاحية مراجعة — منع كامل.

**Acceptance checklist (expanding Amendment §5.7):**
- [ ] Every quote is an exact substring of the active transcript (R7 verified at render).
- [ ] No case can be approved without an authorized reviewer (RBAC test).
- [ ] An approved case never shows under «مشتبه» after refresh.
- [ ] Transcript or re-extraction change invalidates the old decision or re-queues the case per fingerprint — with a visible stale warning.
- [ ] Accept-one-reject-other in the same session works (finding/case/event separation proven by fixture).
- [ ] Every transition appears in the append-only audit trail; UPDATE/DELETE rejected at DB level.
- [ ] No unapproved case enters any leadership figure (structural: leadership views read approved-only; queue size is a separate figure).
- [ ] Exactly six decisions exist — مخالفة صحيحة / ليست مخالفة / إعادة تصنيف / تحتاج مراجعة ثانية / أدلة غير كافية / إحالة لمالك السياسة — each with closed reason code + reviewer identity + stamps.
- [ ] Detector precision on the labelled fixture reported per category (QT-05 discipline; ≥0.90 required for any L2 exposure; L1 preview exempt per §1 rule 1).

**Stop gate:** VS-02 acceptance artifact; consultation for VS-03.
**Mapped EPICs / phase deliverables:** EPIC-16/EPIC-17 subsets + EPIC-08 (review portal v0 as the decision engine) + EPIC-05 completion = P2-E1/E2 slice + P0-E6.
**Flags touched:** `violation_review_enabled` (staging). · **Exposure:** L1.

### VS-03 — التشغيل الليلي ومركز التشغيل — nightly consolidation + Data Operations Center

**Goal:** one authoritative scheduled run over a limited sample with watermarks, retries, and DLQ, observable in SCR-14; morning digest v1 [ASSUME OD-28 — 02:00 Asia/Riyadh, configurable; hourly Read.ai incremental allowed, nightly stays authoritative].

**In scope:** the 16-step nightly pipeline (doc 09 amendment §4.2) on fixtures/staging sample; run states exactly `queued → running → partial → succeeded → failed → cancelled → superseded`; scored M3 + reconciliation queue (PB-012); SCR-14 «مركز تشغيل البيانات»; SCR-21 «صباحيات الخدمة» v1 (PB-016).
**Out of scope:** month-close/infographic content (VS-05); observability stack beyond run metrics + the first Grafana board; live source credentials unless OD-07/OD-13 have closed; email delivery channels [ASSUME OD-29 — in-app first].

**What it needs (§9.1):**
- *Migration:* `pipeline_run` / `pipeline_step_run` / `source_watermark` / `dead_letter_item` / `reconciliation_case` (doc 08 amendment entities).
- *Backend/domain:* nightly orchestrator; watermark service; DLQ + retry ladder; digest composer (deterministic).
- *API:* `GET /pipeline-runs`; `GET /pipeline-runs/{id}`; `POST /pipeline-runs/{id}/retry`; `GET /data-quality/issues`.
- *UI:* SCR-14 (stages, counts, durations, actions) + SCR-21 (digest with deep links).
- *Fixture:* multi-night scenario including one forced sub-feed failure + one late evaluation.
- *Automated tests:* idempotent re-run; `partial` semantics; DLQ replay; kill/resume mid-run.
- *Manual demo script:* below.
- *Observability:* `nightly_run_success`, `data_completeness`, `source_join_rate` exported; run manifest downloadable.
- *Rollback:* `nightly_consolidation_enabled` off — manual per-adapter runs remain possible.
- *Documentation:* run manifest format + operator runbook (`rerun failed items`, `reprocess selected sessions`).

**Demo script (يؤديه المالك مع المهندس):**
1. شغّل التشغيل الليلي يدويًا في Staging من «مركز تشغيل البيانات».
2. راقب المراحل الست عشرة: الأعداد والمدد وحالة كل مرحلة حتى الاكتمال.
3. شاهد سيناريو فشل عقد التقييم: حالة التشغيل `partial` والعناصر الفاشلة في DLQ بسبب مصنف — لا نجاح صامت.
4. نفّذ «إعادة تشغيل العناصر الفاشلة» بأمر واحد وشاهد التشغيل يكتمل.
5. تحقق من وصول التقييم المتأخر إلى «الجلسة 360» في التشغيل التالي.
6. افتح «صباحيات الخدمة» وشاهد ملخص الليلة، وكل عنصر يفتح شاشته الصحيحة.

**Acceptance checklist (= Amendment §4.6, expanded):**
- [ ] The 20+20 sample produces the correct Session-360 count with no duplication.
- [ ] Running the same window twice changes no counts and creates no duplicate review cases.
- [ ] DataHub-evaluation feed failure ⇒ run `partial` while transcript ingest still succeeds.
- [ ] A late evaluation updates the session and shows in Session 360 on the next run.
- [ ] A new suspicion appears in the queue and its quote opens the correct text at the correct لفة.
- [ ] SCR-14 shows watermarks/coverage/errors from live data — a CI test proves no hardcoded values.
- [ ] Resumability survives a worker kill — pipeline state lives in Postgres only (no local JSON/process memory).
- [ ] Digest emitted only from `succeeded`/`partial` runs, with `partial` labelled prominently.

**Stop gate:** VS-03 acceptance artifact; consultation for VS-04.
**Mapped EPICs / phase deliverables:** EPIC-07 + orchestration completion of EPIC-11/12/13 + doc 09 amendment §4 = P1-E2…E5 completion + the new Nightly Orchestrator component (doc 07 amendment).
**Flags touched:** `nightly_consolidation_enabled`. · **Exposure:** L1 (synthetic/staging sample).

### VS-04 — حلقة التعلم الأولى — first governed learning loop

**Goal:** VS-02 decisions become Dataset v1; a candidate detector runs in shadow against the incumbent; promotion requires documented human approval; rollback proven — **no online self-learning, ever** [DECISION SD-22].

**In scope:** `label_dataset`(+items) / `detector_candidate` / `detector_release` / `shadow_result`; missed-case reports (PB-010) consumed as labels; SCR-18 «لوحة جودة الكاشف والتعلم»; **EXP-11 designed and first sample drawn (§6)**; per-category offline evaluation (precision AND estimated recall).
**Out of scope:** automatic promotion (never exists); a second violation type; canary on the real corpus (needs an L2 decision + OD-04); dataset refresh cadence tuning [ASSUME §13 — weekly/bi-weekly by decision volume].

**What it needs (§9.1):**
- *Migration:* 0003 (eval registry) + the learning tables above.
- *Backend/domain:* dataset builder (immutable, hashed, split-frozen); shadow runner; release pointer with rollback.
- *API:* `GET /detector-releases` (+ candidate/eval read endpoints).
- *UI:* SCR-18 — dataset versions, candidate vs incumbent per-category table, shadow diffs, release history.
- *Fixture:* labelled VS-02 outcomes + synthetic negatives + a planted holdout.
- *Automated tests:* split-leakage structural test; shadow-isolation test; rollback restore; feature-scan (no consultant names/ratings).
- *Manual demo script:* below.
- *Observability:* `detector_acceptance_rate`; shadow-diff counts; dataset build events.
- *Rollback:* release-pointer revert — review decisions untouched.
- *Documentation:* release procedure per the Amendment §6.5 stage table.

**Demo script (يؤديه مالك النموذج أمام المالك):**
1. افتح «لوحة جودة الكاشف»: شاهد Dataset v1 بإصدارها وتقسيماتها المجمدة (تدريب/تحقق/Holdout).
2. شاهد مقارنة الكاشف الحالي والمرشح على نفس البيانات: دقة لكل نوع، وتقدير الاسترجاع من عينة EXP-11، وفروق الحالات.
3. حاول «ترقية» المرشح — النظام يطلب اعتمادًا بشريًا موثقًا؛ لا يوجد زر نشر مباشر.
4. نفّذ التراجع الفوري إلى الإصدار السابق وشاهد أن قرارات المراجعة لم تُمس.
5. اطلب أثر نقرة مراجع واحدة على سلوك الإنتاج — لا شيء: القرار يدخل Dataset القادمة فقط.

**Acceptance checklist:**
- [ ] Dataset versioned + immutable (hash-addressed) with train/validation/holdout and no session leakage across splits (structural test).
- [ ] Candidate declares exactly what changed and records Dataset version + Prompt SHA + Model ID + thresholds + taxonomy version.
- [ ] Evaluation reports per-category precision AND estimated recall — never a single aggregate number.
- [ ] Shadow run provably writes nothing to the production queue.
- [ ] Promotion requires a recorded human approval; no write path exists from reviewer UI to detector config (structural test).
- [ ] Rollback restores prior behaviour with zero decision loss.
- [ ] Consultant names/ratings structurally absent from detector features (CI scan).

**Stop gate:** VS-04 acceptance artifact; consultation for VS-05.
**Mapped EPICs / phase deliverables:** EPIC-05 (0003) + EPIC-17 harness reuse + the new Learning-Dataset & Model-Promotion service (doc 07 amendment; P2-E2 adjunct); EXP-11 registered in docs 15/21.
**Flags touched:** `violation_review_enabled` (learning surfaces live under the review product; promotion is workflow-gated, never flag-gated). · **Exposure:** L0/L1.

### VS-05 — الإنفوجرافيك الشهري الأول — first monthly infographic

**Goal:** «نبض خدمة الاستشارات والإرشاد — ملخص الشهر» on one fixed template over a fixture/staging month: web + PDF + PNG, draft → approve → publish (PB-014, committed-early — SD-19/SD-21).

**In scope:** minimal pack-artifact + render path pulled forward from P6 (§0 reconciliation — its L2/L3 exposure still waits for P6 gates); SCR-20; the five sections + mandatory footer (Amendment §7.4); approval chain `draft → data_review → content_review → approved → published → superseded/retracted`; approved-violations-only rule; completeness gate ≥98% [ASSUME §13] with face-of-report override.
**Out of scope:** publication channels beyond the in-platform page [ASSUME OD-29 — Portal + approved Email next]; additional templates (§7.6 — later); LLM phrasing (v1 ships fully deterministic text; any later phrasing passes the R6 gate first).

**What it needs (§9.1):**
- *Migration:* `monthly_infographic` / `infographic_section` / `publication_event` (or pack-artifact binding per doc 08 amendment).
- *Backend/domain:* month-close trigger; section composer from metric results; three-format renderer from ONE structured payload.
- *API:* `GET /infographics/monthly/{period}`; `POST /infographics/{id}/approve`; `POST /infographics/{id}/publish`.
- *UI:* SCR-20 with state chips (مسودة/مراجعة بيانات/مراجعة محتوى/معتمد/منشور).
- *Fixture:* one closed synthetic month ≥98% complete + one month deliberately failing the gate.
- *Automated tests:* byte-identity across web/PDF/PNG payloads; approved-only violations; end-exclusive comparisons; re-render determinism.
- *Manual demo script:* below.
- *Observability:* generation + publication events; draft-deadline alert (doc 19 amendment list).
- *Rollback:* `monthly_infographic_enabled` off; retraction path proven on fixtures.
- *Documentation:* reviewer/approver guide — Data Owner → Service Owner → Publication Authority [ASSUME OD-10].

**Demo script (يؤديه المالك):**
1. بعد إغلاق شهر العينة، افتح المسودة في «الإنفوجرافيك الشهري».
2. راجع الأقسام الخمسة: نبض الخدمة، جودة وأثر الجلسات، صوت المستفيد، الجودة والالتزام، ما يحتاج قرارًا — ثم التذييل الإلزامي.
3. اعتمد بصفة مالك البيانات، ثم مالك الخدمة، ثم انشر بصفة سلطة النشر.
4. نزّل PDF وPNG وقارن الأرقام مع الصفحة — متطابقة تمامًا.
5. اضغط على مؤشر «المخالفات» — رقم المعتمد فقط، وحجم قائمة الاشتباه رقم منفصل، وDrill-down ضمن صلاحياتك.
6. جرّب شهر العينة الناقص — لا مسودة تُنشأ دون Override موثق يظهر على وجه الصفحة.

**Acceptance checklist (= Amendment §7.7, expanded):**
- [ ] Every number traces to a Metric Result + session set (provenance click-through works).
- [ ] Zero numbers generated or computed by an LLM — the renderer consumes the structured artifact only.
- [ ] Comparison sets and periods are end-exclusive correct; previous-month deltas reproduce from frozen artifacts.
- [ ] Unapproved violations never enter the published «المخالفات» figure; queue size appears only as its own labelled figure.
- [ ] Web/PDF/PNG carry byte-identical payload numbers, regenerated from the pack artifact without DOM scraping (I11).
- [ ] KPI click opens drill-down to the underlying sessions within the viewer's permissions.
- [ ] Template stays single-page and readable on a leadership screen and A4 print.
- [ ] No consultant names in the general edition [ASSUME OD-31].
- [ ] Reissue creates a new version and never edits the published one; retraction leaves an auditable trace.

**Stop gate:** VS-05 acceptance artifact; consultation for VS-06 (after VS-05, expansion follows the approved backlog — Amendment §15).
**Mapped EPICs / phase deliverables:** minimal pull-forward of EPIC-31 (packs DDL/factory subset) + render substrate; CAP-OPS-08 registry entry lands in doc 04.
**Flags touched:** `monthly_infographic_enabled`. · **Exposure:** L1 (fixture/staging month); L2/L3 await P6 gates (+ EXP-09 for any future LLM phrasing).

### VS-06 — لوحة التشغيل وAction Center — ops dashboard + Action Center

**Goal:** basic KPIs with drill-down (PB-013) and the finding → action → owner → due date → status → measured-impact loop (PB-015); the digest links into both.

**In scope:** SCR-13 board (CAP-OPS-12) over registered serving views with methodology versions; SCR-19 «مركز الإجراءات»; action entities + `action_completion_rate`.
**Out of scope:** free-form metric queries (VS-07's semantic compiler); coaching/recovery queues (VS-08 candidates); leadership rollups beyond aggregates [ASSUME OD-34 — service owner owns actions].

**What it needs (§9.1):**
- *Migration:* `service_improvement_action` / `action_event` / `action_metric_baseline`.
- *Backend/domain:* KPI card service over registered views; action lifecycle service.
- *API:* `GET/POST/PATCH /actions`.
- *UI:* SCR-19 + dashboard cards with coverage chips.
- *Fixture:* a month with a planted clear-steps decline in one programme.
- *Automated tests:* no-hardcoded-values; RBAC per card; action lifecycle append-only; baseline-comparison wording test.
- *Manual demo script:* below.
- *Observability:* action-overdue alert; dashboard freshness stamp.
- *Rollback:* `action_center_enabled` off.
- *Documentation:* action ownership + closure rules.

**Demo script (يؤديه مالك الخدمة أمام المالك):**
1. افتح اللوحة ولاحظ انخفاض «نسبة الخطوات الواضحة» في برنامج معين.
2. افتح جلسات الخلية المنخفضة (Drill-down) وتصفح عينة منها.
3. أنشئ إجراء تدريب بمالك وموعد استحقاق من الشاشة نفسها، مرتبطًا بالمؤشر.
4. حدّث حالة الإجراء وشاهد خط الأساس → ما بعد الإجراء دون أي ادعاء سببية.
5. في صباح اليوم التالي شاهد الإجراء المتأخر يظهر في «صباحيات الخدمة» برابط مباشر.

**Acceptance checklist:**
- [ ] Every KPI card declares period (end-exclusive), coverage, and freshness; suppression renders as «حجب» + Wilson CI where rates apply.
- [ ] Drill-down session lists reconcile exactly with the card's numerator/denominator.
- [ ] An action requires owner + due date; transitions are append-only events with audit.
- [ ] Post-action comparison renders baseline → follow-up without causal claims (copy test).
- [ ] Leadership sees aggregate `action_completion_rate` only — no consultant-name bypass.
- [ ] Zero hardcoded dashboard values (CI test).

**Stop gate:** VS-06 acceptance artifact; consultation for VS-07.
**Mapped EPICs / phase deliverables:** new Action Center service + dashboard v0 (doc 07 amendment components); precursor to EPIC-19/20 (P3).
**Flags touched:** `action_center_enabled` (+ [REC] `ops_dashboard_enabled` if minted — doc 24). · **Exposure:** L1.

### VS-07 — أول خمس قدرات حتمية — first five deterministic capabilities

**Goal:** the governed-numbers path proves itself on five capabilities of direct value — not the whole catalogue [DECISION Amendment §9.3-VS-07].

**The five (registered per doc 04 §1 discipline before build):**
1. عدد الجلسات وحالاتها حسب الفترة والبرنامج — `sessions_count` + status splits.
2. تقييم المستفيد وتوزيعه — `beneficiary_rating_avg` / `_distribution` (+ `rating_response_rate` caveat).
3. نسبة الخطوات الواضحة — CAP-B1 `clear_steps_rate`.
4. حالات المخالفات المشتبهة/المعتمدة وحالة المراجعة — CAP-B5 alignment + `suspected_cases_open` as separate figures, approved-only in any leadership view.
5. أبرز التحديات أو الأسئلة المتكررة وفق أول Artifact دلالي معتمد — CAP-C1 or CAP-B3 minimal over owner-approved cluster labels ◐OWNER.

**In scope:** metric/dimension registries + deterministic spec→SQL compiler + period resolver (subset); typed answer envelope + R6/R7 verifiers; Lane-0 matcher for the five (τ/δ per doc 04 §2); minimal R-P2 loop for capability 5.
**Out of scope:** the other 21 capabilities (VS-11); free chat (VS-09); export center (Wave B); Hijri parsing beyond fail-loud [ASSUME OD-18].

**What it needs (§9.1):**
- *Migration:* 0009 subset (`serve` projections).
- *Backend/domain:* registries + compiler + envelope + verifiers; clustering minimal for capability 5.
- *API:* endpoints 5 (capability list) + 6 (metric-queries) per doc 17.
- *UI:* dashboard cards + fixed-question answer views (SCR-02 rendering subset).
- *Fixture:* the 120-session synthetic corpus (doc 15 §8) as substrate.
- *Automated tests:* EXP-06-style cross-product for the five vs independent reference queries (100% numeric equality); all six period grains; zero-row honesty; suppression fixtures.
- *Manual demo script:* below.
- *Observability:* verifier-rejection telemetry; router abstention counts.
- *Rollback:* per-capability `suspended` status.
- *Documentation:* registry YAML per entry with owner sign-off fields.

**Demo script (يؤديه المالك):**
1. اطرح الأسئلة الخمسة المحددة (أو افتح بطاقاتها) على بيانات العينة.
2. لكل إجابة: شاهد الرقم، الفترة المحسومة، التغطية والاستبعادات المسماة، ومصدر الرقم القابل للتتبع.
3. جرّب سؤالًا خارج الخمسة — رفض صريح أو «الاستيضاح المغلق» بخيارات من السجل، لا إجابة مجاورة.
4. جرّب فترة فارغة — إجابة «لا بيانات» صادقة، لا أصفار مزيفة.
5. جرّب صيغة سؤال مختلفة لنفس القدرة — نفس الرقم بالضبط (حتمية التوجيه والتجميع).

**Acceptance checklist:**
- [ ] Five capabilities registered with all doc 04 §1.2 fields; owner sign-off fields present.
- [ ] Reference-query equality 100% on the fixture corpus (a mismatch stops the slice — fix the compiler, never the reference).
- [ ] R6: narrative numerals ⊆ allowed-literals, including formatting variants.
- [ ] R7 on any quoted evidence shown in drill-downs.
- [ ] Route correctness on the five's paraphrase sets ≥0.95 floor (EXP-05 discipline at mini-scale; full EXP-05 remains the P3 gate).
- [ ] Abstention → «الاستيضاح المغلق» with ≤4 registry options — never a guess (I5).
- [ ] Suspected vs approved violations never merged in any figure.

**Stop gate:** VS-07 acceptance artifact; consultation for VS-08 content (Wave B candidates, doc 24 §4).
**Mapped EPICs / phase deliverables:** EPIC-19 + EPIC-20 subsets (P3-E1…E6 slice) + EPIC-18 minimal (clustering for capability 5).
**Flags touched:** [REC — minted at consultation] `governed_qa_enabled`; the seven amendment flags untouched. · **Exposure:** L1; L2 for named pilot analysts only after EXP-06 + the affected EXP-05 tranche pass.

### VS-08 — توسيع التحليل التشغيلي — operational analysis expansion

**Goal:** the first Wave B tranche the owner approves. The amendment's candidates: Service Recovery Queue (PB-101, SCR-22), limited Consultant 360 (PB-102), recurring fixed answers (PB-105), inconsistent answers (PB-106), weekly pulse (PB-118).

**In scope / out of scope:** decided item-by-item at the VS-07 stop-gate consultation — **this slice has no pre-authorized content** (SD-21); the list above is a proposal only. Non-negotiable regardless of scope trims: fairness rules for PB-102 (coverage, n≥30 + Wilson CI, no punitive auto-ranking) and the strict separation of recovery from violations.

**What it needs (§9.1):** per entering item, from its doc 24 card — cluster machinery completion (EPIC-18) for PB-105/106; recovery entities + SCR-22 for PB-101; weekly artifact on the PB-014 render path for PB-118; fixtures + tests per card.

**Demo script:** assembled at planning time from the entering items' cards; each item demos separately, e.g.:
1. افتح «قائمة التعافي الخدمي» وشاهد حالة تقييم منخفض مع سببها المسجل.
2. عالجها وأغلقها بتوثيق النتيجة.
3. افتح موضوعًا بإجابات غير متسقة وشاهد الاقتباسين المتعارضين بهويتهما الكاملة.

**Acceptance checklist:**
- [ ] The entering items' card criteria (doc 24), promoted verbatim into this slice's checklist.
- [ ] Strict-separation test: the recovery queue shares no UI surface, wording, or data path with «اشتباه مخالفة».
- [ ] Every new list/queue has RBAC matrix rows + audit events from day one.

**Stop gate:** VS-08 acceptance artifact; consultation for VS-09.
**Mapped EPICs / phase deliverables:** EPIC-18 completion + first EPIC-20b curated engines (P2-E5…E8 / P3 slice).
**Flags touched:** [REC] per-item flags minted at consultation. · **Exposure:** L1 → L2 per item gate.

### VS-09 — Lane 1 وLane 2 — bounded agent + closed clarification

**Goal:** a constrained agent over the **proven** capabilities — never over an open data table (PB-203/PB-204).

**In scope:** Lane-1 planner + toolbelt adapters + budgets in Postgres; Lane-2 «الاستيضاح المغلق» + closed reason codes + resolved-facts store; composer + verifier v2; conversation UI (SCR-01/02/04).
**Out of scope:** Lane 3 (VS-10); capability expansion (VS-11); any pilot exposure before the named gates; `search_evidence` tool unless EXP-07 machinery enters by consultation.

**What it needs (§9.1):**
- *Migration:* 0009 completion (conversations/turns/tool calls).
- *Backend/domain:* planner loop (≤6 steps, ≤8 calls, one tool per step); tool adapters; budget store.
- *API:* endpoints 1–4 (doc 17).
- *UI:* SCR-01/SCR-02/SCR-04.
- *Fixture:* DS-09 subset (golden long-tail) + DS-11 injection set + DS-12 follow-up forms.
- *Automated tests:* golden routing; injection suite; budget enforcement; no-model-numbers (R6) on composed answers.
- *Manual demo script:* below.
- *Observability:* full reasoning log (R13) + admin trace view.
- *Rollback:* `lane1_agent_enabled` off — Lane 0 unaffected.
- *Documentation:* toolbelt contract + reason-code Arabic strings.

**Demo script (يؤديه محلل أمام المالك):**
1. اسأل سؤالًا مركبًا يتطلب أداتين متتابعتين.
2. شاهد (بصفة مشرف) سجل التفكير والأدوات المستدعاة خطوة بخطوة.
3. اسأل سؤالًا غامضًا — «الاستيضاح المغلق» بخيارات ≤4 من السجل.
4. اسأل سؤالًا خارج النطاق — رفض صريح برمز سبب مغلق.
5. جرّب حقن تعليمات داخل نص جلسة مزروع — لا أثر على السلوك أو الأرقام.

**Acceptance checklist:**
- [ ] Zero wrong route on the golden question set.
- [ ] No model-computed numbers (R6) and no SQL/schema names in model space (I2/I3 structural tests).
- [ ] Clarification/decline behaviour correct on DS-12 fixtures; context preserved across the round-trip.
- [ ] Budgets enforced server-side (step/call caps produce governed degradation, never silent overrun).
- [ ] DS-11 injection fixtures alter nothing (quotes, numbers, routes).
- **Gates for L2:** EXP-05 full + EXP-09 + EXP-10 first pass (+ EXP-07 if the evidence tool entered). L1 demo does not wait for them (§1 rule 1).

**Stop gate:** VS-09 acceptance artifact; consultation for VS-10.
**Mapped EPICs / phase deliverables:** EPIC-23/24/25/26/27 (P4-E3…E9).
**Flags touched:** `lane1_agent_enabled`. · **Exposure:** L1; L2 after the named gates.

### VS-10 — Lane 3 لسؤال واحد حقيقي — deep analysis for one real question

**Goal:** one full month-scoped job — Map → Verify → Reduce — with progress and resumability (PB-205 first proof; «مهمة تحليل معمّق»).

**In scope:** `jobs` DDL + orchestration (kill/resume/cancel); analysis-schema approval flow; job progress UI; one owner-chosen question end-to-end on the fixture month.
**Out of scope:** custom RAG lifecycle (PB-206 — later consultation); promotion board (VS-11); unrestricted job submission; full-corpus scope (§4.6 ladder applies).

**What it needs (§9.1):**
- *Migration:* 0011 (`jobs` schema).
- *Backend/domain:* job factory; month partitions; map/verify/reduce stages; resumable checkpoints.
- *API:* endpoints 8–13 (doc 17).
- *UI:* SCR-05 (progress/accept) + SCR-06 (archive).
- *Fixture:* one synthetic month with planted phenomena for the chosen question.
- *Automated tests:* kill/resume; INCOMPLETE partition honesty; reduce-is-code; cancel-draining idempotency.
- *Manual demo script:* below.
- *Observability:* job progress, spend telemetry per model call.
- *Rollback:* `lane3_analysis_enabled` off; draining cancel for in-flight jobs.
- *Documentation:* job runbook + promotion-candidate note format.

**Demo script (يؤديه المالك):**
1. اقبل عرض «مهمة تحليل معمّق» لسؤال حقيقي على شهر واحد.
2. راقب التقدم بالأقسام الشهرية ومعرف المهمة.
3. اقتل العامل عمدًا (بواسطة المهندس) وشاهد الاستئناف من نقطة التوقف دون فقد.
4. افتح النتيجة: نتائج موثقة باقتباسات متحققة، والتجميع محسوب برمجيًا.
5. شاهد قسمًا فاشلًا معلمًا INCOMPLETE بلا إخفاء ولا «قائمة فارغة».

**Acceptance checklist:**
- [ ] Job resumable with job id + progress; checkpoint survives worker death.
- [ ] Failed partition = INCOMPLETE, never silent (typed failure end-to-end).
- [ ] All findings pass deterministic verification before reduce; reduce computed by code, never by the model (I3).
- [ ] Quotes in results pass R7 with full identifiers.
- [ ] Result cacheable + fingerprinted for later promotion (PB-208 input).
- **Gate for L2:** EXP-08.

**Stop gate:** VS-10 acceptance artifact; consultation for VS-11.
**Mapped EPICs / phase deliverables:** EPIC-28/29/30 (P5).
**Flags touched:** `lane3_analysis_enabled`. · **Exposure:** L1; L2 after EXP-08.

### VS-11 — توسيع الكتالوج والحزم الرسمية — catalogue expansion + official packs

**Goal:** complete the priority capability tranches (toward all 26 built-or-governed-blocked), the quarterly report, frozen/live views, and reissue/retraction (PB-201 growth + CAP-D10).

**In scope:** EPIC-20b remaining tranche in owner-approved order; EPIC-31 full packs machinery (the VS-05 pull-forward grows into its owning phase); promotion-board candidates (PB-208) if approved.
**Out of scope:** cutover activities (VS-12); Wave C items without consultation.

**What it needs (§9.1):**
- *Migration:* 0012 (`packs`).
- *Backend/domain:* pack factory + freeze semantics; capability engines per tranche.
- *API:* endpoints 14–15 (publications/packs).
- *UI:* SCR-10 (نشر/إعادة إصدار).
- *Fixture:* two-period corpus for freeze/reissue tests.
- *Automated tests:* doc 21 P3 exit list; pack freeze/live both-countings; reissue/retraction append-only.
- *Manual demo script:* below.
- *Observability:* pack-generation SLOs + deadline alerts.
- *Rollback:* capability suspension; pack retraction path.
- *Documentation:* publication authority matrix [ASSUME OD-10].

**Demo script (يؤديه المالك):**
1. اطلب قدرات جديدة من الكتالوج — إجابات موثقة أو رفض محكوم بالرمز المسجل.
2. أنشئ حزمة ربع سنوية مجمدة، واعتمدها، وانشرها.
3. أعد إصدارًا مصححًا وشاهد النسخة القديمة باقية بعلامة superseded مع سبب التصحيح.

**Acceptance checklist:**
- [ ] Every committed capability either built (structurally valid envelope on fixtures) or returning its registered blocked reason code — never a neighbour answer (I5).
- [ ] Frozen pack + live recount coexist and reconcile under both countings after a taxonomy change.
- [ ] Reissue/retraction are append-only publication events with stated cause.
- [ ] EXP-09 green before any LLM narrative appears on packs or the infographic.

**Stop gate:** VS-11 acceptance artifact; consultation for VS-12 readiness.
**Mapped EPICs / phase deliverables:** EPIC-20b + EPIC-31 (P3 completion + P6-E1/E2).
**Flags touched:** per-capability statuses; `monthly_infographic_enabled` (quarterly template addition). · **Exposure:** L1/L2 per item gates.

### VS-12 — Pilot والتوسع والقطع — pilot, expansion, cutover

**Goal:** controlled real-data exposure and the path to production default — starts only after the previous slices are proven [DECISION Amendment §9.3-VS-12].

**In scope:** security/performance/restore drills; parallel-run harness with six-way difference classification — suspected and approved violations compared **separately** (doc 20 amendment); pilots with named users (L2); cohort staging 20 → 100 → month → full corpus (§4.6); cutover + legacy containment.
**Out of scope:** new features — this slice hardens and exposes only.

**What it needs (§9.1):**
- *Migration:* none new.
- *Backend/domain:* parallel-run harness; shadow detector migration checks.
- *API / UI:* none new.
- *Fixture:* production-shaped staging cohorts per the ladder.
- *Automated tests:* EXP-10 full; restore drill within RTO/RPO; load tests.
- *Manual demo script:* below.
- *Observability:* full SLO/alert list live (doc 19 amendment: nightly-by-deadline, freshness, join rate, DQ backlog, review backlog age, detector acceptance/drift, infographic deadlines, action overdue, notification failures).
- *Rollback:* documented cutover rollback; flags-off playbook.
- *Documentation:* runbooks + on-call rota.

**Demo script (تؤديه القيادة مع الفريق):**
1. راجع تقرير التشغيل المتوازي وفروقه المصنفة الستة، والمخالفات المشتبهة والمعتمدة كل على حدة.
2. راجع نتائج اختبار الأمن الهجومي (EXP-10 الكامل).
3. راجع تمرين الاستعادة وزمنه.
4. اعتمد قائمة القطع أو أوقفها ببند مسمى ومالك وإجراء.

**Acceptance checklist:**
- [ ] EXP-10 full green; findings remediated or accepted by name.
- [ ] Parallel-run report with zero unexplained differences (suspected vs approved compared separately).
- [ ] Restore drill within RTO/RPO targets.
- [ ] L2 pilot sign-offs recorded per feature gate; L3 enablement decisions per flag.
- [ ] Cutover checklist (doc 20 G-gates) signed.

**Stop gate:** the cutover decision itself — the final owner approval of the programme.
**Mapped EPICs / phase deliverables:** EPIC-32/33/34/35/36 (P6/P7).
**Flags touched:** production enablement decisions for all seven flags. · **Exposure:** L2 → L3.

---

## 4. The Opus-5 ticket protocol [DECISION Amendment §10 — carried in full meaning]

### 4.1 Ticket size (§10.1)

- One Coding Ticket = one unit completable and reviewable as a whole: **4–8 hours of actual agent work**; anything larger is split before starting.
- More than one major migration, or more than one independent UI journey ⇒ split.
- At most one domain per ticket — unless the ticket IS a small vertical slice that inherently crosses domains, and the plan file says so.
- Never bundle unrequested "hygiene fixes" into a feature ticket — file them as their own tickets.

### 4.2 The pre-ticket plan file (§10.2) — written BEFORE any edit

Location: `docs/tickets/T-VSxx-n-plan.md`. All 11 sections mandatory; "none" must be stated explicitly, never omitted:

```markdown
# T-VSxx-n — <title>
1. Goal — one paragraph; ties to PB-xxx + VS-xx + EPIC-nn.
2. Out of scope — explicit exclusions.
3. Files expected to change — paths, each marked new/modified.
4. Proposed migration — id, tables, up/down summary (or "none").
5. API contract — endpoints + request/response sketch (or "none").
6. UI flow — screens/states touched (or "none").
7. Tests to write BEFORE/WITH the code — named list.
8. Fixture data — what, where, PII statement.
9. Demo steps — numbered; Arabic where user-facing.
10. Rollback — exact procedure.
11. Risks — named, each with its mitigation.
```

Execution starts only after the plan file exists in the branch. The reviewer (or the owner at gates) reads the plan before the diff.

### 4.3 Ticket Definition of Done (§10.3)

A ticket is complete only when ALL of the following exist:

- [ ] Complete code with no hidden stubs.
- [ ] Migration that upgrades AND downgrades under test.
- [ ] Unit tests.
- [ ] Integration/API tests.
- [ ] UI test or scripted manual acceptance, per stage.
- [ ] Fixture or factory.
- [ ] Authorization test.
- [ ] Required observability events/metrics emitted.
- [ ] Documentation updated.
- [ ] **`docs/STATE.md` updated** (§5).
- [ ] **`PRODUCT_BACKLOG.md` updated** — PB state changes per doc 24 §7.
- [ ] **`KNOWN_GAPS.md` updated** — including flag-hidden TODOs.
- [ ] The run/test command and its actual result attached.
- [ ] An explicit list of what was NOT done.

### 4.4 Execution prohibitions (§10.4 — full meaning, binding)

1. No starting multiple large Epics in parallel by one agent.
2. No creating dozens of skeleton files with their logic deferred.
3. No TODO inside an accepted path unless it sits behind a **closed** feature flag AND is listed in `KNOWN_GAPS.md`.
4. No modifying a test gate to get past a failing feature.
5. No completing a backend ticket for a user-facing feature without its API/UI or an adequate demo.
6. No hiding an adapter or model-call failure by converting it into an empty list (typed failure only — doc 09 §6.4).
7. No hardcoded demo data or ratios in any dashboard.
8. No full-corpus processing before the same path succeeds on a fixture and then a small sample (§4.6 ladder).
9. No deploying a new model/prompt directly from reviewer results without evaluation and documented approval (SD-22).
10. No moving to the next ticket without an acceptance report for the current one.

### 4.5 Feature flags (§10.5)

Every large feature runs behind its clear flag — exactly these seven, verbatim:

`session_360_enabled` · `violation_review_enabled` · `nightly_consolidation_enabled` · `monthly_infographic_enabled` · `action_center_enabled` · `lane1_agent_enabled` · `lane3_analysis_enabled`

A flag isolates a feature and controls its exposure level (§1); **it is never a substitute for finishing the feature.** New flags are minted only through the doc 24 consultation path.

### 4.6 Data strategy during development (§10.6 — the ladder, strictly in order)

1. **Small synthetic fixture** covering the critical cases (PII-free, committed to git).
2. **Approved anonymized snapshot subset** for integration tests — only after the approval exists [ASSUME OD-04/OD-08 scope].
3. **Staging cohort of 20–100 sessions.**
4. **One full month** before any full corpus.
5. **Full backfill** only after resumability, cost, and DQ are proven on the smaller rungs.

Skipping a rung requires an owner decision recorded in the slice acceptance artifact.

### 4.7 Required outputs of every execution cycle (§10.7)

- One clear commit/patch.
- Modified-files report.
- Migration ID.
- Test commands **and their actual results**.
- Screenshot or precise demo description for any UI change.
- Seed/demo data IDs.
- Known limitations and issues.
- The decision needed before continuing.

**A bare «تم الإنجاز» / "done" without the evidence above is invalid and is treated as not done.**

---

## 5. The state file — `docs/STATE.md` template

Machine-readable YAML header + human log; updated by every ticket (§4.3) and every stop gate (§2):

```yaml
# --- nwafeth-intelligence delivery state (docs/STATE.md header) ---
plan_version: "planning-package@2026-08-03 + owner-amendment"
current_vs: VS-00                      # VS-00 … VS-12
vs_status: in_progress                 # planned | in_progress | demo_ready | accepted | blocked
current_ticket: T-VS00-3
ticket_status: tests_written           # planned | tests_written | implementing | demo_ready
                                       # | acceptance_reported | closed
acceptance_report_ref: docs/acceptance/VS-00/T-VS00-2-acceptance.md
last_green_ci: "<CI run id/url>"
flags_state:                           # the seven flags, per environment
  staging:
    session_360_enabled: false
    violation_review_enabled: false
    nightly_consolidation_enabled: false
    monthly_infographic_enabled: false
    action_center_enabled: false
    lane1_agent_enabled: false
    lane3_analysis_enabled: false
  production: {}                       # empty until VS-12 decisions
known_gaps:
  - id: KG-0001
    ticket: T-VS00-4
    description: "MinIO/Grafana/Loki full stack deferred to VS-03"
    flag: "n/a (ops-internal)"
next_decision_needed: >-
  Owner approval of VS-00 acceptance + consultation on VS-01 content
  (PB-003/004/005 scope).
owner_approvals:
  - vs: VS-00
    decision: pending                  # pending | approved | approved-with-conditions | rework
    approved_by: null
    approved_at: null
# --- end header; human-readable narrative log follows below ---
```

---

## 6. EXP-11 — false-negative estimation design (runs in VS-04)

**Placement:** VS-04, alongside Dataset v1 — because estimated recall per category is a mandatory column of the first offline evaluation (§3-VS-04) [DECISION Amendment §6.4; EXP-11 pre-assigned; registered in docs 15/21 experiment inventories].

- **Question:** what sample size, strata, and cadence give a usable per-category false-negative (missed-violation) estimate without overloading reviewers?
- **Design:** weekly stratified random sample of sessions **not flagged** by the violation detector, sent for light review («مراجعة خفيفة» — one reviewer, escalation on find). Strata [DECISION Amendment §6.4]: البرنامج، المستشار، مدة الجلسة، شهر الجلسة، جودة الترانسكربت، الفئات قليلة التمثيل.
- **Outputs:** estimated FN rate per category with confidence interval; discovery of uncovered types (feeds taxonomy proposals via ADR-0012); the standing weekly sample size + cadence recommendation → closes **OD-32** [ASSUME OD-32 until then: weekly stratified sample; size set by this experiment's first two cycles].
- **Gate use:** VS-04 acceptance requires the design executed on fixture data + the first real-cycle plan approved; every detector release (cadence per OD-33) must cite the latest EXP-11 estimate in its evaluation report.
- **Guards:** sampled sessions enter the same review UI flagged `fn_sample` — never mixed with detector cases in metrics; reviewer time budgeted and monitored via `review_backlog_age`; found cases become labels through the PB-010 path with origin recorded.

---

## 7. The first five coding tickets — VS-00 ONLY

> **SPECIFICATION ONLY — NOT EXECUTED.** These five tickets are written to the §4.2 template so the implementation agent can start immediately after plan approval. No application code exists or is authorized yet [DECISION Amendment §0.3, §14.5-8]. Order is strict; each ticket ends with its own acceptance report before the next starts (§4.4-10). Each fits the 4–8h size (§4.1).

### T-VS00-1 — Repo scaffold + CI + boundary enforcement (≈ EPIC-01 slice; doc 21 P0-E1)

1. **Goal:** the doc 23 §3.1 monorepo tree exists, empty but enforced: toolchain pins, import-linter contracts, CI (lint → type → import-contracts → unit → structural), state files seeded. Ties: PB-017 spine · VS-00 · EPIC-01.
2. **Out of scope:** database, auth, frontend build beyond a placeholder workspace, Docker stack, any endpoint.
3. **Files expected:** `pyproject.toml` · `.github/workflows/ci.yml` · `.importlinter` · `.pre-commit-config.yaml` · package dirs per doc 23 §3.1 with `__init__` stubs · `Makefile` (`make ci`, `make run` placeholder) · `docs/STATE.md` (§5 template) · `PRODUCT_BACKLOG.md` (seeded from doc 24 §7 states) · `KNOWN_GAPS.md` · `docs/BUGS.md`.
4. **Migration:** none.
5. **API contract:** none.
6. **UI flow:** none.
7. **Tests first:** import-contract test with an intentionally planted cross-plane import (must fail, then reverted); marker-scan test (a planted `skip` under `tests/golden/*` fails); secret-scan CI job; pytest collection smoke.
8. **Fixture:** none.
9. **Demo steps:** (1) `make ci` محليًا → أخضر. (2) ازرع استيرادًا محظورًا → CI أحمر باسم العقد المخالف، ثم أرجِعه. (3) أظهر ملفات الحالة الثلاثة المهيأة (`STATE`/`PRODUCT_BACKLOG`/`KNOWN_GAPS`).
10. **Rollback:** revert the branch — no persistent state exists.
11. **Risks:** over-scaffolding (mitigation: only doc 23 §3.1 directories, zero logic files — §4.4-2); CI runner limits (mitigation: dependency caching pinned).

### T-VS00-2 — Baseline migration 0001 + 0002 subset with up/down proof (≈ EPIC-02 + EPIC-05 slice; P0-E2)

1. **Goal:** the database exists only as migrations (F3 unrepresentable): 0001 — 11 schemas, extensions `pgvector` + `pg_trgm`, roles `nip_web`/`nip_worker`/`nip_migrator`/`nip_readonly_bi`/`nip_steward_breakglass`, base grants; plus the 0002 **subset** `0002a_ops_audit_minimal` — `ops.audit_event` (monthly RANGE, append-only REVOKEs) + `ops.model_registry` + `ops.prompt_registry` skeletons. Doc 23 §6 logical numbering is preserved as label prefixes: the 0002 remainder ships as `0002b_ops_registries` in VS-02 prep — no renumbering ever.
2. **Out of scope:** 0003 eval tables; 0004+ domain schemas; model-registry seed rows (arrive with the Groq-client epic); partition automation beyond current+next month.
3. **Files expected:** `alembic.ini` · `migrations/env.py` · `migrations/versions/0001_baseline.py` · `migrations/versions/0002a_ops_audit_minimal.py` · CI job `migration-cycle` · `deploy/dev/docker-compose.db.yml` (Postgres 17 only) · grant-snapshot baseline file.
4. **Migration:** 0001 + 0002a as above; both with authored `downgrade()` — never stubbed.
5. **API contract:** none.
6. **UI flow:** none.
7. **Tests first:** empty PG17 container → `upgrade head` → schemas/roles asserted → `downgrade base` → empty again; audit append-only test (UPDATE/DELETE fail for non-superuser roles); role-existence + grant-snapshot diff test.
8. **Fixture:** none — the empty database is the fixture.
9. **Demo steps:** (1) شغّل `make db-up && make migrate` وشاهد المخططات الأحد عشر بالاستعلام. (2) نفّذ `make migrate-down` وشاهد قاعدة فارغة. (3) جرّب UPDATE على سجل تدقيق → مرفوض من قاعدة البيانات نفسها.
10. **Rollback:** `alembic downgrade base`.
11. **Risks:** grant drift vs the doc 08 §1 matrix (mitigation: grant-snapshot test committed now, diffed forever — future G-SEC-2); partition tooling creep (mitigation: two partitions only; tooling arrives with EPIC-16).

### T-VS00-3 — OIDC login + roles + deny-by-default + probe routes (≈ EPIC-04 slice; P0-E4)

1. **Goal:** from the first route onward, no unauthenticated surface ever exists (F2/ISS-04 unrepeatable): OIDC via broker [ASSUME OD-02], JWT validation (cached JWKS, access-token TTL ≤15 min), the seven roles wired, per-route permission declarations, scoped CORS, `/healthz` + `/readyz` as the only anonymous probes. Ties: PB-017 · VS-00 · EPIC-04.
2. **Out of scope:** consultant-name masking rules (VS-01 surfaces); break-glass flows (OD-09); production IdP wiring; any session UI beyond the login/logout shell.
3. **Files expected:** `apps/web/main.py` (app factory) · `apps/web/auth/oidc.py` · `apps/web/auth/jwt.py` · `apps/web/auth/permissions.py` · `apps/web/routes/meta.py` (`GET /api/v1/me`) · middleware wiring · `tests/security/test_route_walk.py` · Keycloak dev-realm config under `deploy/dev/`.
4. **Migration:** none — roles are IdP-side; DB pool-per-role wiring is configuration only at this stage.
5. **API contract:** `GET /api/v1/me` → `{user_id, display_name, roles[]}`; 401/403 as RFC 7807 problem+json with Arabic `detail` strings (doc 17 §0/§3).
6. **UI flow:** unauthenticated visit → IdP redirect → home shell showing name + role → logout. One protected demo route per role tier for the matrix test.
7. **Tests first:** G-SEC-1 route walk (every route except the two probes → 401 unauthenticated); role-matrix test (analyst → 403 on the admin route); CORS assertion (explicit origins only — wildcard structurally absent); token-expiry test.
8. **Fixture:** dev-realm users, one per role, documented test credentials — dev only, never defaults in code (VS-00 checklist item).
9. **Demo steps:** (1) افتح الرابط دون دخول → تحويل لصفحة الهوية. (2) سجّل الدخول وشاهد اسمك ودورك. (3) افتح مسارًا إداريًا بدور «محلل» → 403 بالعربية. (4) سجّل الخروج وتأكد من انتهاء الجلسة.
10. **Rollback:** revert deploy — security is never flag-gated, so rollback = previous build; IdP sessions invalidated.
11. **Risks:** real IdP unavailable in dev (mitigation: containerized broker realm committed per OD-02 assumption); JWKS cache staleness (mitigation: TTL + kid-miss refetch test).

### T-VS00-4 — Health/readiness + version/build page from real data (≈ EPIC-03 slice; P0-E3/E5)

1. **Goal:** telemetry that cannot lie (ISS-05/06 unrepeatable): `/healthz` = process liveness; `/readyz` = real dependency checks (DB `SELECT 1`, alembic-head match); `GET /api/v1/meta/version` returns git SHA, build time, migration revision **read from the database**, environment name — all live values; structlog JSON + request-id propagation.
2. **Out of scope:** MinIO/queue/Grafana/Loki/pgBackRest (declared in `KNOWN_GAPS.md` → VS-03); tracing across worker hops (no worker exists yet).
3. **Files expected:** `apps/web/routes/probes.py` · `apps/web/routes/meta.py` (extended) · `common/observability/logging.py` · `common/observability/request_id.py` · `deploy/dev/docker-compose.yml` (web + postgres) · CI build-arg injection · minimal `/metrics` exposition.
4. **Migration:** none — reads `alembic_version`.
5. **API contract:** `GET /healthz` → 200 alive; `GET /readyz` → 200/503 with per-dependency booleans; `GET /api/v1/meta/version` → `{git_sha, built_at, migration_revision, environment}` (doc 17 endpoints 26/27 shapes).
6. **UI flow:** none yet — the home screen consumes these in T-VS00-5.
7. **Tests first:** healthcheck-lies regression (DB stopped → `/readyz` 503 within threshold while `/healthz` stays 200); version-matches-alembic test; request-id present on every log line; no-static-200 structural check.
8. **Fixture:** none.
9. **Demo steps:** (1) افتح `/api/v1/meta/version` وقارن SHA مع آخر Commit. (2) أوقف قاعدة البيانات → `/readyz` أحمر خلال ثوانٍ. (3) أعد تشغيلها → أخضر، والرحلة كلها ظاهرة في السجلات بمعرف طلب واحد.
10. **Rollback:** revert deploy.
11. **Risks:** the static-200 temptation during dependency flaps (forbidden — I16 pin test); build metadata absent in local runs (mitigation: dev fallback values explicitly labelled `dev`, asserted in tests).

### T-VS00-5 — Synthetic seed fixture (PII-free) + home screen with DB status + VS-00 demo script (≈ EPIC-01/EPIC-14-precursor slice)

1. **Goal:** `make seed` writes an idempotent, checksummed, PII-free synthetic seed — a `seed_manifest` row + demo `ops.audit_event` rows (no domain schemas exist yet); the RTL home screen renders build version, DB status, migration revision, seed status, and the seven flags' states — all from live APIs; the VS-00 demo script document is authored. Ties: VS-00 demo · EPIC-01 · precursor of the doc 15 §8 corpus discipline (EPIC-14).
2. **Out of scope:** the 120-session synthetic corpus (doc 15 §8 — arrives with VS-01 ingestion needs); any session/transcript data; a flag admin UI (env-config read-only display v0 only).
3. **Files expected:** `fixtures/seed_vs00.py` · `fixtures/manifest.json` (checksums) · `frontend/src/pages/Home.tsx` + RTL layout shell (self-hosted assets only) · `apps/web/routes/status.py` (aggregated home payload) · `docs/demos/VS-00-demo.md` · PII-scan CI hook for `fixtures/`.
4. **Migration:** none — seeds 0002a tables; if a dedicated `seed_manifest` table is preferred over reusing ops rows, it is added to 0002a **during plan review of this ticket**, not as a new migration.
5. **API contract:** `GET /api/v1/meta/status` → `{version:{…}, db:{ready, revision}, seed:{name, checksum, row_counts}, flags:{…the seven…}}` — every field live-sourced.
6. **UI flow:** login (T-VS00-3) → home: بطاقة نسخة البناء، بطاقة قاعدة البيانات، بطاقة بيانات العينة، وقائمة الأعلام السبعة وحالتها (كلها OFF).
7. **Tests first:** seed idempotency (run twice → identical counts + checksum); PII scan on fixture files (P3 regex + roster deny-list) green; home component test with an injected fake API proving zero hardcoded values; RTL snapshot test.
8. **Fixture:** the seed itself; checksum manifest committed; documented `make seed-clean` (delete by `seed_run_id`).
9. **Demo steps:** the full VS-00 demo of §3-VS-00 — this ticket authors `docs/demos/VS-00-demo.md` and the acceptance-report skeleton for the stop gate.
10. **Rollback:** `make seed-clean` + revert.
11. **Risks:** hardcoded status values sneaking into the UI (mitigation: component test asserts API-sourced rendering — §4.4-7); fixture drift (mitigation: checksum in CI); CDN asset temptation (forbidden — self-hosted only; the CDN-grep check from T-VS00-1 covers the build output).

**After T-VS00-5:** run the VS-00 stop rule (§2). The next planning artifact is the VS-01 ticket set — written only after `docs/acceptance/VS-00-acceptance.md` records owner approval and the VS-01 consultation outcome (PB-003/004/005 scope).

---

## 8. Traceability

- **Slices → backlog:** every VS section names its PB ids; backlog states move per doc 24 §7 at each gate; SD-21 consultation is step 5's standing agenda item.
- **Slices → epics/phases:** "Mapped EPICs" cite doc 23 §5 (EPIC-01…20b, reserved 21…36) and doc 21 §3 phase deliverables; doc 21 keeps the gate framework, this document keeps the execution order (§0).
- **Slices → experiments:** EXP gates named per slice; EXP-11 owned here at VS-04 (§6) and registered in docs 15/21.
- **Runtime artifacts:** `docs/STATE.md` (§5) · `docs/acceptance/VS-xx-acceptance.md` (§2) · `docs/tickets/T-VSxx-n-plan.md` (§4.2) · `docs/demos/VS-xx-demo.md` · `PRODUCT_BACKLOG.md` · `KNOWN_GAPS.md`.
- **Amendment coverage:** §9 → §§1–3 here; §10 → §4; §6.4/EXP-11 → §6; §14.5-8 (five tickets, specification only) → §7; §15 start order → §0 + the slice sequence. The package traceability file maps each Amendment requirement → document → PB → VS → acceptance test (Amendment §14.3).

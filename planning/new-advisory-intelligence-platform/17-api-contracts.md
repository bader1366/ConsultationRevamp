# 17 — API Contracts
**Platform:** Nwafeth Intelligence — منصة نوافث لذكاء الجلسات الاستشارية (Monsha'at Advisory Session Intelligence Platform)
**Status:** Draft for owner review · **Date:** 2026-08-02 · **Amended:** 2026-08-03 (Owner Amendment integrated) · **Author:** Planning package (Fable 5)
**Depends on:** 08 (entities), 11 (typed envelope §7), 12 (lanes, reason codes, tool semantics), 13 (job state machine §7, API shapes §7.6), 16 (authz, permissions, audit) · **Feeds:** 18 (UX consumes these contracts), 19 (SLOs per endpoint class), 21, 23
**Sources used:** GREENFIELD §16.1, §8.8, §23-17; MASTER_PROMPT §5.3 (job endpoints), Appendix C (reason codes); CORE-BRIEF §4, §6, §13; docs 11 §7, 12 §12, 13 §7.6; arch/05 (legacy contract anti-patterns), arch/08 ISS-05/ISS-13

---

## 0. Conventions (binding for every endpoint)

1. **Base path `/api/v1`**, JSON only (`application/json; charset=utf-8`), Arabic text NFC-normalized. **No endpoint ever returns HTML as the normative result** [FACT GREENFIELD §16.1] — the legacy contract was "Arabic HTML string in a JSON field" injected via `innerHTML` [FACT arch/05 §TL;DR]; here `narrative_ar` is plain text and all structure is typed (I11).
2. **Auth**: OIDC Bearer JWT on every route (doc 16 §2.2). The only anonymous paths are `GET /healthz` and `GET /readyz` (static/dependency probes, no data). CI gate G-SEC-1 enforces this.
3. **Authz**: each endpoint declares required **permission strings** (doc 16 §2.5 closed vocabulary); scoped roles get resource-level checks. 401 = no/invalid token; 403 = valid token, missing permission; 404 = resource exists but principal must not learn that (ownership hiding).
4. **Status codes — no 200-on-failure, ever** (legacy ISS-13: HTTP 200 carrying a raw exception string; ISS-05: failures logged as success [FACT arch/08]):

| Code | Use |
|---|---|
| 200 | Successful read / synchronous result |
| 201 | Resource created (`Location` header) |
| 202 | Accepted for async processing (jobs, exports) — body carries the tracking resource |
| 204 | Successful delete/cancel with no body |
| 400 | Malformed request (JSON, types) |
| 401 / 403 / 404 | Auth / permission / not-found-or-hidden |
| 409 | State conflict (job not terminal, stale review basis, idempotency conflict, quota) |
| 410 | Sunset version or expired artifact/offer |
| 422 | Well-formed but semantically invalid (unknown metric id, bad period, schema violation) — carries field errors |
| 429 | Rate limit — `Retry-After` mandatory |
| 500 | Unexpected server error — problem+json, **no stack trace, no exception string** (I16) |
| 503 | Dependency down / degraded mode — body carries `system_state` (e.g. `pinned_lane0`) |

5. **Request correlation**: client MAY send `X-Request-Id`; server always echoes it (generated if absent) and joins it to the R13 reasoning log and audit rows (doc 16 §8).
6. **Timestamps** ISO-8601 UTC (`2026-08-02T10:41:07Z`); **periods end-exclusive** `{start, end}` dates (CORE-BRIEF §13.4); identifiers are ULIDs/prefixed uids (`JOB-…`, `PACK-…`) — opaque to clients.
7. **Unknown-field tolerance**: clients MUST ignore unknown response fields; servers MUST reject unknown *request* fields (422) — additive evolution without silent client drift.
8. **Pagination** (§15), **idempotency keys** (§15.2), **rate limits** (§15.3) apply as stated there.

---

## 1. Endpoint inventory

| # | Method + path | Purpose | Permission (doc 16 §2.5) | Idempotency |
|---|---|---|---|---|
| 1 | `POST /api/v1/conversations` | open a chat thread | `chat.ask` | key optional |
| 2 | `GET /api/v1/conversations` · `GET …/{conv_uid}` | list/read own threads | `chat.ask` | — |
| 3 | `POST /api/v1/conversations/{conv_uid}/turns` | ask a question / answer a clarification / accept an offer | `chat.ask` | **key required** |
| 4 | `GET /api/v1/conversations/{conv_uid}/turns/{seq}` | re-fetch a turn (answer envelope) | `chat.ask` (owner) | — |
| 5 | `GET /api/v1/capabilities` | capability catalogue (registry view) | `aggregate.read` | — |
| 6 | `POST /api/v1/metric-queries` | governed dashboard/BI query (closed spec, registry-validated — the model is NOT the caller) | `aggregate.read` | key optional |
| 7 | `GET /api/v1/sessions/{session_uid}/transcript-window` | evidence at exact turn (access-audited) | `transcript.read` | — |
| 8 | `POST /api/v1/jobs` | submit deep-analysis job (from offer or analyst-direct) | `jobs.submit` | **key required** |
| 9 | `GET /api/v1/jobs` · `GET …/{job_uid}` | list/status+progress | `jobs.read` | — |
| 10 | `GET /api/v1/jobs/{job_uid}/result` | structured answer envelope (409 until terminal) | `jobs.read` | — |
| 11 | `GET /api/v1/jobs/{job_uid}/findings` | verified findings, paginated | `findings.read_quotes` | — |
| 12 | `POST /api/v1/jobs/{job_uid}/rerun` | re-run failed tasks or full supersede | `jobs.submit` | key required |
| 13 | `DELETE /api/v1/jobs/{job_uid}` | cancel (draining, idempotent) | `jobs.submit` (owner) / `sources.admin` | inherent |
| 14 | `GET /api/v1/publications` · `GET /api/v1/packs/{pack_uid}` | current publications + frozen pack fetch | `aggregate.read` | — |
| 15 | `POST /api/v1/publications` | publish / reissue a pack | `packs.publish` | key required |
| 16 | `GET /api/v1/reviews` · `GET …/{review_item_id}` | review queues | `review.decide` | — |
| 17 | `POST /api/v1/reviews/{review_item_id}/decisions` | append-only decision event | `review.decide` (`review.retire` for retire) | key required |
| 18 | `GET /api/v1/taxonomies` · `…/{tax_id}/versions` · `…/{tax_id}/lineage` | taxonomy read + lineage | `aggregate.read` | — |
| 19 | `POST /api/v1/taxonomy-proposals` | propose category/merge/split/alias | `taxonomy.propose` | key required |
| 20 | `POST /api/v1/taxonomies/{tax_id}/versions` | publish approved version | `taxonomy.approve` | key required |
| 21 | `GET /api/v1/health/data-quality` | DQ health board (CAP-D9 surface) | `dq.read` | — |
| 22 | `GET /api/v1/ingest/runs` · `GET /api/v1/ingest/dlq` · `POST …/dlq/{id}/replay` | ingestion ops | `sources.admin` | replay: key required |
| 23 | `GET /api/v1/sources` · `POST /api/v1/transcript-sources/{id}/activate` | provider/source admin (activation audited) | `sources.admin` | key required |
| 24 | `POST /api/v1/exports` · `GET /api/v1/exports/{export_uid}` | server-rendered XLSX/PDF/JSON export | `exports.create` | key required |
| 25 | `GET /api/v1/audit-events` | audit log read | `audit.read` | — |
| 26 | `GET /api/v1/meta/version` · `GET /api/v1/meta/system-state` | build + degradation state | any authenticated | — |
| 27 | `GET /healthz` · `GET /readyz` | probes (anonymous, static/dependency) | — | — |
| 28 | `GET /api/v1/pipeline-runs` · `GET …/{run_id}` | nightly/incremental run list + detail + manifest (Amendment — §13A.1) | `ops.runs_read` | — |
| 29 | `POST /api/v1/pipeline-runs/{run_id}/retry` | re-run failed items only (Amendment — §13A.1) | `sources.admin` | key required |
| 30 | `POST /api/v1/pipeline-runs/{run_id}/reprocess-sessions` | reprocess selected sessions (Amendment — §13A.1) | `sources.admin` | key required |
| 31 | `GET /api/v1/data-quality/issues` | DQ + reconciliation work items (Amendment — §13A.2) | `dq.read` | — |
| 32 | `GET /api/v1/sessions/{session_uid}/360` | Session 360 composite (Amendment — §13A.3) | `session360.read` | — |
| 33 | `GET /api/v1/violations/suspected` | «اشتباه مخالفة» queue (Amendment — §13A.4) | `violations.queue_read` | — |
| 34 | `GET /api/v1/violations/cases/{case_uid}` | violation case detail (Amendment — §13A.4) | `violations.queue_read` | — |
| 35 | `POST /api/v1/violations/cases/{case_uid}/review-events` | append six-decision review event (Amendment — §13A.4) | `review.decide` | key required |
| 36 | `POST /api/v1/sessions/{session_uid}/missed-violation` | report a missed suspicion (Amendment — §13A.5) | `violations.report_missed` | key required |
| 37 | `GET /api/v1/detector-releases` · `GET …/{release_id}/shadow-results` | detector releases + shadow comparisons, read-only (Amendment — §13A.6) | `detector.learning_read` | — |
| 38 | `GET /api/v1/infographics/monthly/{period}` | monthly infographic, published/draft per role (Amendment — §13A.7) | `infographic.read_published` / `infographic.read_draft` | — |
| 39 | `POST /api/v1/infographics/{infographic_uid}/approve` | data/content approval event (Amendment — §13A.7) | `infographic.approve` | key required |
| 40 | `POST /api/v1/infographics/{infographic_uid}/publish` | publish — authority-gated, step-up (Amendment — §13A.7) | `infographic.publish` | key required |
| 41 | `GET`/`POST /api/v1/actions` · `PATCH …/{action_uid}` | Action Center (Amendment — §13A.8) | `actions.read` / `actions.manage` (+`actions.close`) | POST: key required |
| 42 | `GET`/`POST /api/v1/subscriptions` | digest/notification subscriptions (Amendment — §13A.9) | `digest.subscribe` | POST: key required |

Everything the model-facing toolbelt does (doc 12 §5) happens **inside** the serving plane; the toolbelt is not HTTP and is not exposed here. `start_deep_analysis`/`get_analysis_job` being orchestrator-controlled [REC CORE-BRIEF §4] is visible in this API: jobs are created only by endpoint 8 under a human's token. Rows 28–42 are the Owner Amendment operational surfaces [DECISION owner 2026-08-03 / Amendment §12-17]; their contracts are §13A, and none of them is reachable by the agent toolbelt (doc 12 §0.1 hard rules — reads go through registered capabilities, mutations are workflow-only).

---

## 2. Error model — RFC 7807 `application/problem+json`

Every non-2xx response is a Problem document. `type` URIs live under `https://nip.monshaat.gov.sa/problems/…` [ASSUME OD-01 final host]; they are stable identifiers, not required to dereference.

```json
{
  "type": "https://nip.monshaat.gov.sa/problems/validation",
  "title": "طلب غير صالح",
  "status": 422,
  "detail": "قيمة metric غير مسجلة في السجل.",
  "instance": "/api/v1/metric-queries",
  "request_id": "req_01K1TWJ8Q2",
  "errors": [
    { "pointer": "/spec/metric", "code": "UNREGISTERED_METRIC",
      "message_ar": "المقياس «معدل_الرضا_الفوري» غير مسجل. المقاييس المتاحة عبر GET /api/v1/capabilities." }
  ]
}
```

**Closed problem `type` catalogue** (slug after `/problems/`):

| Slug | Status | Notes |
|---|---|---|
| `validation` | 400/422 | `errors[]` with JSON-pointers; unknown request fields land here |
| `unauthenticated` | 401 | `WWW-Authenticate: Bearer` |
| `forbidden` | 403 | includes `permission` extension naming the missing permission string |
| `not-found` | 404 | also used for ownership hiding (doc 16 §2.5) |
| `conflict` | 409 | `conflict_kind` extension: `job_not_terminal` · `stale_basis` · `idempotency_mismatch` · `quota_exceeded` · `already_decided` · `version_conflict` |
| `gone` | 410 | expired offer/export, sunset API version (`sunset` extension: date) |
| `rate-limited` | 429 | `Retry-After` header; `limit_class` extension |
| `boundary` | 200 — **not an error** | honest boundaries are successful turn responses (§4.4), never HTTP errors: the request worked; the answer is a governed refusal |
| `degraded` | 503 | `system_state` extension (`degraded_inference`, `pinned_lane0`, …) |
| `internal` | 500 | `request_id` only — no internals (I16) |

**Reason codes vs HTTP errors — the load-bearing distinction.** The 16 closed reason codes (CORE-BRIEF §4: `DIMENSION_NOT_AVAILABLE` … `DEEP_JOB_INCOMPLETE`) describe *analytical* outcomes and travel inside successful (200) turn/job payloads as `boundary.reason_code`. HTTP problems describe *transport/authz/validation* failures. Conflating them was the legacy failure mode (ISS-05: analytical failure logged as HTTP success with no marker; ISS-13: HTTP success wrapping an exception). Here: an unanswerable question is `200 + kind:"boundary" + reason_code`; a broken request is `4xx problem+json`; a broken server is `5xx problem+json`. Nothing else.

---

## 3. Auth contract

- `Authorization: Bearer <JWT>` — validated per doc 16 §2.2 (signature, `aud: nip-api`, `exp`, ≤15 min TTL).
- Roles/permissions resolved server-side per request; responses never echo the permission map (clients call `GET /api/v1/meta/session` — sub, role, display fields — for UI affordances [REC]; UI hiding is convenience, never the gate — I14).
- Step-up: endpoints marked destructive (15, 20, 23-activate) reject tokens with `auth_time` older than 5 min → `403 forbidden` with `reauth_required: true` extension.
- Service accounts (client-credentials) may call only ingestion/ops endpoints their client is scoped to; `pipeline_service` cannot call chat/export/review endpoints at all (doc 16 §2.3).

---

## 4. Chat / analysis contract

### 4.1 Create conversation, ask a question

```text
POST /api/v1/conversations              → 201 {"conv_uid":"01K1TWH4…","started_at":"…"}
POST /api/v1/conversations/{conv_uid}/turns     (Idempotency-Key required)
```

```json
// TurnRequest (JSON Schema sketch — closed, additionalProperties: false)
{
  "$id": "nip:v1:TurnRequest",
  "type": "object",
  "oneOf": [
    { "required": ["question_ar"],
      "properties": { "question_ar": { "type": "string", "minLength": 2, "maxLength": 2000 } } },
    { "required": ["clarification_response"],
      "properties": { "clarification_response": {
        "type": "object", "required": ["turn_seq", "selected_option_id"],
        "properties": { "turn_seq": {"type":"integer"},
                        "selected_option_id": {"type":"string"} },
        "additionalProperties": false } } },
    { "required": ["accept_offer"],
      "properties": { "accept_offer": {
        "type": "object", "required": ["turn_seq", "offer_id"],
        "properties": { "turn_seq": {"type":"integer"}, "offer_id": {"type":"string"} },
        "additionalProperties": false } } }
  ]
}
```

Synchronous response (Lane 0 ≤3 s, Lane 1 ≤8 s hard budgets [DECISION MASTER_PROMPT R16] make a blocking POST correct; no streaming of numeric answers [REC — partial numeric output cannot pass gates; alternative SSE token streaming rejected as an I18/R6 hazard; revisit trigger: a pure-narrative surface with no numbers]).

### 4.2 TurnResponse — the typed envelope carrier

```json
// TurnResponse (schema sketch)
{
  "$id": "nip:v1:TurnResponse",
  "type": "object",
  "required": ["conv_uid", "turn_seq", "kind", "generated_at", "request_id"],
  "properties": {
    "conv_uid": {"type":"string"}, "turn_seq": {"type":"integer"},
    "kind": {"enum": ["answer", "clarification", "boundary", "deep_job_offer"]},
    "answer": { "$ref": "nip:v1:AnswerEnvelope" },          // kind=answer
    "clarification": { "$ref": "nip:v1:Clarification" },     // kind=clarification
    "boundary": { "$ref": "nip:v1:Boundary" },               // kind=boundary
    "deep_job_offer": { "$ref": "nip:v1:DeepJobOffer" },     // kind=deep_job_offer
    "generated_at": {"type":"string","format":"date-time"},
    "request_id": {"type":"string"}
  }
}
```

`AnswerEnvelope` **is** doc 11 §7's typed result envelope plus presentation blocks — one schema, every renderer downstream (I11):

```json
{
  "$id": "nip:v1:AnswerEnvelope",
  "type": "object",
  "required": ["title_ar", "summary_ar", "results", "methodology_ar", "period",
               "coverage", "versions", "followups", "export_refs", "lane"],
  "properties": {
    "title_ar":   {"type":"string"},
    "summary_ar": {"type":"string"},              // plain text; NEVER HTML
    "results": { "type":"array", "items": { "oneOf": [
        {"$ref":"nip:v1:MetricBlock"},            // {metric:{id,arabic_label,unit,denominator_ar}, value, ci?, comparisons[]}
        {"$ref":"nip:v1:TableBlock"},             // {columns:[{key,label_ar,unit?}], rows:[[…]], suppressed_cells[]}
        {"$ref":"nip:v1:FindingsBlock"},          // {taxonomy_id, version, items:[{category_id,label_ar,sessions,rate_per_100,support}]}
        {"$ref":"nip:v1:EvidenceBlock"}           // {quotes:[{session_uid,turn_index,speaker_role,quote_text,transcript_source_id,source_version,pseudonymized:true}]}
    ]}},
    "methodology_ar": {"type":"string"},          // registry-derived method text incl. denominators, suppression, CI method
    "period":   { "$ref":"nip:v1:Period" },       // {start,end,label,all_time,decision_source,comparisons[]}
    "coverage": { "$ref":"nip:v1:Coverage" },     // {used,matched,total,exclusions:[{code,count,label_ar}]}
    "versions": { "type":"object",                // I13/I17 provenance on the face of the answer
      "required":["registry_version","taxonomy_stamps","model_ids","code_version"],
      "properties": { "registry_version":{"type":"string"},
        "taxonomy_stamps":{"type":"array","items":{"type":"object",
          "properties":{"taxonomy_id":{"type":"string"},"version":{"type":"integer"},"view":{"enum":["frozen","live"]}}}},
        "model_ids":{"type":"array","items":{"type":"string"}},
        "code_version":{"type":"string"} } },
    "followups": { "type":"array", "maxItems":4,  // registry-derived suggested next questions
      "items":{"type":"object","properties":{"question_ar":{"type":"string"},"capability_id":{"type":"string"}}} },
    "export_refs": { "type":"array",              // pre-authorized export descriptors (client POSTs /exports with one)
      "items":{"type":"object","properties":{"source":{"type":"string"},"formats":{"type":"array","items":{"enum":["xlsx","pdf","json"]}}}} },
    "job_ids": {"type":"array","items":{"type":"string"}},   // when the answer was served from a Lane-3 artifact
    "narrative_ar": {"type":"string"},            // optional composer text, plain, gate-passed; deterministic render if absent
    "lane": {"enum":["0","1","2","3"]},
    "system_state": {"enum":["normal","degraded_inference","degraded_retrieval","pinned_lane0","partial"]}
  }
}
```

### 4.3 Worked example — metric answer (Lane 0)

`POST /api/v1/conversations/01K1TWH4…/turns` · body `{"question_ar": "ما نسبة الحضور في يونيو 2026؟"}` → `200`:

```json
{ "conv_uid": "01K1TWH4…", "turn_seq": 3, "kind": "answer",
  "answer": {
    "title_ar": "نسبة الحضور — يونيو 2026",
    "summary_ar": "بلغت نسبة الحضور 84.2% من الجلسات المجدولة المعروفة الحالة في يونيو 2026، بانخفاض 1.8 نقطة مئوية عن مايو 2026.",
    "results": [
      { "block": "metric",
        "metric": { "id": "attendance_rate", "arabic_label": "نسبة الحضور",
                    "unit": "pct", "denominator_ar": "الجلسات المجدولة المعروفة الحالة في الفترة" },
        "value": 84.2, "ci": { "method": "wilson_95", "low": 82.1, "high": 86.1 },
        "comparisons": [ { "period_label": "مايو 2026", "value": 86.0, "delta": -1.8 } ] },
      { "block": "table",
        "columns": [ {"key":"channel","label_ar":"القناة"}, {"key":"rate","label_ar":"نسبة الحضور","unit":"pct"},
                     {"key":"n","label_ar":"عدد الجلسات"} ],
        "rows": [ ["عن بُعد", 85.1, 1093], ["حضوري", 81.7, 165] ],
        "suppressed_cells": [ {"dimension_value": "قناة الفروع", "n": 14, "label_ar": "أقل من حد العرض (30 جلسة)"} ] }
    ],
    "methodology_ar": "الوحدة: جلسة. المقام: الجلسات المجدولة المعروفة الحالة. جلسات الحالة غير المعروفة مستبعدة ومعلنة ضمن التغطية. فترة نهاية مفتوحة (2026-06-01 إلى 2026-07-01). حد العرض n≥30 مع فترات ثقة Wilson 95%.",
    "period": { "start": "2026-06-01", "end": "2026-07-01", "label": "يونيو 2026",
                "all_time": false, "decision_source": "resolved",
                "comparisons": [ {"start":"2026-05-01","end":"2026-06-01","label":"مايو 2026","kind":"previous_month"} ] },
    "coverage": { "used": 1258, "matched": 1401, "total": 1460,
                  "exclusions": [ {"code":"STATUS_UNKNOWN","count":59,"label_ar":"حالة الحضور غير معروفة"},
                                  {"code":"CONSULTANT_UNRESOLVED","count":143,"label_ar":"جلسات لم يُحسم مستشارها"} ] },
    "versions": { "registry_version": "sha256:9c4f…", "taxonomy_stamps": [],
                  "model_ids": [], "code_version": "git:ab12cd3" },
    "followups": [ {"question_ar":"ما نسبة الإلغاء في يونيو 2026؟","capability_id":"CAP-D4"},
                   {"question_ar":"ما علاقة الحضور بجودة الجلسة؟","capability_id":"CAP-D3"} ],
    "export_refs": [ {"source":"answer_artifact:AA-01K1TWJ9…","formats":["xlsx","pdf","json"]} ],
    "job_ids": [], "lane": "0", "system_state": "normal"
  },
  "generated_at": "2026-08-02T10:41:07Z", "request_id": "req_01K1TWJ8Q2" }
```

Note `model_ids: []` — a Lane-0 metric answer with the composer disabled involves **zero** model calls, and the contract shows it.

### 4.4 Clarification round-trip (Lane 2) and honest boundary

Turn 1 — `{"question_ar": "كيف كان أداء قطاع التجزئة الشهر الماضي؟"}` → `200 kind:"clarification"` («قطاع» is three-way ambiguous, CORE-BRIEF §13.3):

```json
{ "kind": "clarification",
  "clarification": {
    "prompt_ar": "«قطاع» تحتمل أكثر من معنى في بيانات المنصة. أيها تقصد؟",
    "options": [
      { "option_id": "OPT-sector-service_category", "label_ar": "فئة الخدمة الاستشارية (مثل: التجزئة ضمن فئات الخدمة)" },
      { "option_id": "OPT-sector-government_entity", "label_ar": "جهة حكومية مذكورة في الجلسات" },
      { "option_id": "OPT-sector-business_sector", "label_ar": "قطاع نشاط منشأة المستفيد (غير متوفر حالياً — لا يوجد مصدر هوية خارجي)" }
    ],
    "expires_at": "2026-08-02T11:11:07Z"
  }, "turn_seq": 4 }
```

Turn 2 — `{"clarification_response": {"turn_seq": 4, "selected_option_id": "OPT-sector-service_category"}}` → `200 kind:"answer"` scoped to `service_category='التجزئة'`. Options come only from the registry (≤4, closed — Lane 2 contract); a free-text reply instead of a selection is treated as a **new question**. Expired clarification: `410 gone`.

Boundary example — «ما القطاع التجاري الأكثر نمواً؟» → `200 kind:"boundary"`:

```json
{ "kind": "boundary",
  "boundary": {
    "reason_code": "DIMENSION_NOT_AVAILABLE",
    "message_ar": "هذا التقسيم (القطاع التجاري للمنشأة) غير متوفر في بيانات المنصة حالياً. التقسيمات المتاحة: فئة الخدمة، البرنامج، القناة، الجهة الحكومية المذكورة.",
    "alternatives": [ {"question_ar":"ما فئات الخدمة الأكثر طلباً خلال 3 أشهر؟","capability_id":"CAP-C4"} ]
  }, "turn_seq": 5 }
```

`message_ar` strings are the fixed renderer templates of doc 12 §12 — the model never authors failure prose.

### 4.5 Deep-job offer (Lane 3 hand-off)

`200 kind:"deep_job_offer"` when the deterministic trigger tree (doc 13 §1) routes P6:

```json
{ "kind": "deep_job_offer",
  "deep_job_offer": {
    "offer_id": "OFF-01J9Z2…",
    "message_ar": "المؤشرات المسجلة لا تجيب على هذا السؤال، لكن يمكن تشغيل تحليل معمّق يقرأ نصوص الجلسات ضمن الربع الأول 2026 (3304 جلسات، تقدير المدة: 165 دقيقة). هل تريد بدء التحليل؟",
    "declared_scope": { "period": {"start":"2026-01-01","end":"2026-04-01","label":"الربع الأول 2026"},
                        "partitions": 3, "sessions": 3304,
                        "estimated_calls": 3304, "estimated_wallclock_min": 165,
                        "estimated_spend_usd": 15.6 },
    "analysis_schema_sha": "b1f4…",
    "expires_at": "2026-08-05T10:41:07Z"
  }, "turn_seq": 6 }
```

Accepting via `{"accept_offer": {...}}` on the conversation **or** `POST /api/v1/jobs` with the `offer_id` (§5.1) — both create the same job; the chat path is sugar over the job endpoint. Estimates are declared, never gates (ADR-0020).

---

## 5. Deep-analysis job API (shapes fixed in doc 13 §7.6; normative contract here)

### 5.1 Submit

```text
POST /api/v1/jobs        (Idempotency-Key required; permission jobs.submit)
```

```json
{ "offer_id": "OFF-01J9Z2…",
  "question_ar": "كم مرة نصح المستشارون بالتقدم لبرنامج تمويل محدد خلال الربع الأول 2026؟",
  "period": { "start": "2026-01-01", "end": "2026-04-01" },
  "scope": { "programme_id": null, "consultant_id": null },
  "analysis_schema_sha": "b1f4…" }
```

`202 Accepted` + `Location: /api/v1/jobs/JOB-2026-08-02-000117`:

```json
{ "job_uid": "JOB-2026-08-02-000117", "status": "accepted",
  "declared_scope": { "partitions": 3, "sessions": 3304, "estimated_calls": 3304,
                      "estimated_wallclock_min": 165, "estimated_spend_usd": 15.6 } }
```

Validation: `analysis_schema_sha` must equal the offer's reviewed schema (else `409 conflict_kind:version_conflict`); analyst-direct submissions (no `offer_id`) carry a full `analysis_schema` object validated against the meta-schema (doc 13 §2.3) → `422` on violation. Per-user concurrent quota exceeded → `409 conflict_kind:quota_exceeded` (doc 16 T08).

### 5.2 Status + partition progress

`GET /api/v1/jobs/{job_uid}` → `200` (supports `ETag`/`If-None-Match`; clients poll every 15–30 s [REC]; optional SSE `GET …/events` deferred to post-pilot [REC — polling is sufficient at tens of jobs/day; revisit trigger: operator dashboard needs sub-5s updates]):

```json
{ "job_uid": "JOB-2026-08-02-000117",
  "status": "mapping",            // offered|accepted|planning|mapping|verifying|reducing|complete|incomplete|stalled|cancelled|failed|superseded (doc 13 §7.1)
  "question_ar": "كم مرة نصح المستشارون …؟",
  "period": {"start":"2026-01-01","end":"2026-04-01","label":"الربع الأول 2026"},
  "partitions": [
    { "partition": "2026-01", "state": "complete",  "sessions_total": 998,  "sessions_done": 998,  "tasks_failed": 0 },
    { "partition": "2026-02", "state": "running",   "sessions_total": 981,  "sessions_done": 512,  "tasks_failed": 3 },
    { "partition": "2026-03", "state": "pending",   "sessions_total": 1325, "sessions_done": 0,    "tasks_failed": 0 } ],
  "progress": { "sessions": {"total": 3304, "done": 1510, "failed": 3, "skipped": 63},
                "findings": {"verified": 214, "dropped_unverifiable": 4, "dropped_schema": 1},
                "spend": {"model_calls": 1513, "tokens_in": 5211840, "tokens_out": 310112, "usd_so_far": 6.91},
                "eta_seconds": 4980 },
  "created_by": "u:7f3a…", "created_at": "2026-08-02T08:00:11Z" }
```

### 5.3 Result — and INCOMPLETE semantics

`GET /api/v1/jobs/{job_uid}/result`:

- non-terminal → `409 conflict_kind:job_not_terminal` (with current `status`) — **never a partial answer on this path** (I16);
- `complete` → `200` with a full `AnswerEnvelope` (`lane:"3"`, `job_ids:[job_uid]`);
- `incomplete` → `200` with the envelope **plus** a mandatory boundary block — the declared-gap contract [FACT doc 13 §7.4]:

```json
{ "answer": { "…": "…", "system_state": "partial",
    "coverage": { "used": 3221, "matched": 3241, "total": 3304,
      "exclusions": [ {"code":"TASKS_FAILED","count":17,"label_ar":"مهام فشلت بعد 3 محاولات"},
                      {"code":"NO_TRANSCRIPT","count":63,"label_ar":"جلسات بلا محضر متاح"},
                      {"code":"PARTITION_INCOMPLETE","count":1,"label_ar":"قسم شهري غير مكتمل: فبراير 2026"} ] } },
  "boundary": { "reason_code": "DEEP_JOB_INCOMPLETE",
    "message_ar": "اكتمل التحليل المعمّق مع فجوة معلنة: شهر فبراير: قُرئت 981 من 998 جلسة؛ 17 مهمة فشلت بعد 3 محاولات (أقسام غير مكتملة: 2026-02). النتائج المعروضة تصف الجزء المكتمل فقط.",
    "rerun": { "endpoint": "/api/v1/jobs/JOB-2026-08-02-000117/rerun", "mode": "failed_only" } } }
```

- `failed` → `200` with `kind:"boundary"` payload (e.g. `PERIOD_EMPTY` — the *job resource* is fine; the analysis has an honest outcome);
- `cancelled` (terminal, user-initiated) → `200 {"status":"cancelled"}` with no envelope and no reason code; clients render «أُلغي التحليل بواسطة المستخدم»;
- `superseded` → `200` with a `superseded_by` pointer to the replacing job and the original frozen envelope still served.

The OpenAPI spec carries the closed terminal-state → response-shape mapping table; every terminal state has exactly one response shape, so no client ever has to guess whether an absent envelope means failure (I16).

### 5.4 Rerun / cancel / findings

```text
POST   /api/v1/jobs/{job_uid}/rerun    {"mode": "failed_only" | "full"}   → 202 (failed_only reuses job_uid; full mints a new job that supersedes)
DELETE /api/v1/jobs/{job_uid}          → 204 (draining; repeat DELETE → 204, idempotent)
GET    /api/v1/jobs/{job_uid}/findings?cursor=…&limit=100                 → 200 page of verified findings
```

Finding row (quote-bearing ⇒ `findings.read_quotes`, audited via the evidence accessor when quote text is expanded):

```json
{ "finding_id": 88412, "session_uid": "01JGN0V9…", "partition": "2026-02",
  "fields": { "advice_kind": "funding_programme_application", "programme_ref": "PRG-تمويل-004",
              "occurred": true },
  "quote": { "turn_index": 87, "speaker_role": "consultant",
             "quote_text": "أنصحك بالتقدم لبرنامج التمويل قبل نهاية الشهر",
             "transcript_source_id": "SRC-READAI", "source_version": 2 },
  "verification": { "status": "verified", "extraction_run_id": 511,
                    "model_id": "openai/gpt-oss-120b", "prompt_sha": "e7a9…" },
  "taxonomy": { "taxonomy_id": null, "version": null } }
```

---

## 6. Transcript evidence retrieval (access-audited)

```text
GET /api/v1/sessions/{session_uid}/transcript-window?turn_index=142&before=2&after=2
    permission: transcript.read      (executive_viewer: 403 — ADR-0017 split)
```

`200`:

```json
{ "session_uid": "01JGN0V9WQZ2Y4X8C6B3A1MKRT",
  "transcript_source_id": "SRC-READAI", "source_version": 2, "active": true,
  "window": { "center": 142, "before": 2, "after": 2 },
  "turns": [
    { "turn_index": 140, "speaker_role": "beneficiary", "started_at_s": 1201.4,
      "text": "جربت المنصة ثلاث مرات ورُفض الطلب في كل مرة" },
    { "turn_index": 141, "speaker_role": "consultant", "started_at_s": 1214.0,
      "text": "هل ظهرت لك رسالة سبب الرفض؟" },
    { "turn_index": 142, "speaker_role": "consultant", "started_at_s": 1225.7,
      "text": "أنصحك بالتقدم لبرنامج التمويل قبل نهاية الشهر", "highlight": true },
    { "turn_index": 143, "speaker_role": "beneficiary", "started_at_s": 1233.1,
      "text": "طيب، ما هي المستندات المطلوبة؟" },
    { "turn_index": 144, "speaker_role": "consultant", "started_at_s": 1240.9,
      "text": "السجل التجاري والقوائم المالية لآخر سنة" } ],
  "audit_notice_ar": "هذا الاطلاع مسجل في سجل التدقيق." }
```

Rules: window caps `before,after ≤ 10` (`422` above — evidence viewing, not bulk export); text is served from the **active** transcript version and stamps `(source, source_version)` (I6/I7); every call writes `evidence_accessed` in the same transaction (doc 16 §2.4 — no audit row, no data); names appear per the caller's role (re-expansion rules, doc 16 §4.2).

---

## 7. Capability catalogue

```text
GET /api/v1/capabilities?lane=0&cursor=…       permission: aggregate.read
```

```json
{ "items": [
    { "capability_id": "CAP-B1", "slug": "clear_steps_rate",
      "name_ar": "نسبة الجلسات المنتهية بخطوات واضحة", "lane": "0",
      "status": "active", "default_period": "last_closed_month",
      "dimensions": ["consultant","programme","service_category","channel"],
      "unit": "pct", "denominator_ar": "الجلسات ذات استخراج وضوح الخطوات",
      "taxonomy_refs": [], "min_support": 30,
      "example_questions_ar": ["ما نسبة الجلسات المنتهية بخطوات واضحة الشهر الماضي؟"],
      "registry_version": "sha256:9c4f…" } ],
  "next_cursor": null }
```

This is the registry's read-only projection (I12) — the same registry the Lane-0 matcher and `list_capabilities` tool consume; there is exactly one source of truth and this endpoint is a view of it.

---

## 8. Dashboards, metric queries, packs and publications

### 8.1 Governed metric query (dashboards/BI; human-initiated)

```text
POST /api/v1/metric-queries        permission: aggregate.read
```

Request is the **closed spec** — identical schema to the toolbelt's `metric_query` spec (doc 12 §5.3), registry-validated; free text cannot reach SQL from here either:

```json
{ "spec": { "metric": "attendance_rate",
            "dimensions": ["channel"],
            "filters": [ {"key":"programme_id","op":"eq","value":"PRG-004"} ],
            "period": {"start":"2026-06-01","end":"2026-07-01"},
            "comparison": "previous_month" } }
```

`200` → the doc 11 §7 tool envelope verbatim (`data`, `metric`, `provenance` incl. `sql_fingerprint`, `coverage`, `period`, `quality`). Unknown metric/dimension → `422 UNREGISTERED_METRIC` — the API refuses exactly like the compiler (I12); scoped roles get their mandatory filters injected server-side (doc 16 §2.5).

### 8.2 Publications and frozen packs

```text
GET /api/v1/publications?capability_id=CAP-B5&period_start=2026-04-01&period_end=2026-05-01
GET /api/v1/publications?current=true&period_start=…&period_end=…        # dashboard "published view"
GET /api/v1/packs/{pack_uid}                                             # frozen artifact by id
POST /api/v1/publications                                                # publish/reissue (packs.publish, step-up)
```

Publication list — the reissue chain is explicit, append-only (ADR-0011):

```json
{ "items": [
    { "publication_id": 3, "pack_uid": "PACK-2026-04-CAPB5-b", "capability_id": "CAP-B5",
      "period": {"start":"2026-04-01","end":"2026-05-01","label":"أبريل 2026"},
      "published_at": "2026-08-02T11:02:05Z", "published_by": "u:9c11…",
      "supersedes_publication_id": 2, "cause_code": "TAXONOMY",
      "reason_ar": "قُسِّم VIOL-017 إلى VIOL-017a/b؛ أعادت إعادة التصنيف المجدولة عدّ أبريل على الإصدار ن6",
      "current": true },
    { "publication_id": 2, "pack_uid": "PACK-2026-04-CAPB5-a", "current": false, "…": "…" } ],
  "next_cursor": null }
```

`GET /api/v1/packs/{pack_uid}` → `200` with the pack's frozen `result_envelope` (an `AnswerEnvelope`), its stamps (`corpus_snapshot_id`, taxonomy stamps, `code_version`), `constituents`, and `reproducibility_status`. **Immutability contract:** response carries `Cache-Control: immutable`, strong `ETag = sha256(result_envelope)`; a superseded pack keeps returning `200` at its own `pack_uid` forever with a `superseded_by` pointer — published numbers are never edited and never disappear (§6.4 GREENFIELD).

`POST /api/v1/publications` body `{"pack_uid": "...", "supersedes_publication_id": 2?}`; the server computes `cause_code` mechanically from stamp diffs (doc 08 §10) and rejects a reissue that cannot name one (`409 conflict_kind:version_conflict`). Retraction obligations, when triggered, are returned in the response so the UI can route the notification workflow (doc 16 §11.3).

---

## 9. Review queues and decision events (append-only)

```text
GET  /api/v1/reviews?kind=finding_review&state=pending&cursor=…      permission: review.decide
GET  /api/v1/reviews/{review_item_id}                                 (violation items: quote via evidence rules)
POST /api/v1/reviews/{review_item_id}/decisions                       (Idempotency-Key required)
```

Queue item (violation review example):

```json
{ "review_item_id": 5512, "item_kind": "finding_review", "state": "pending",
  "opened_at": "2026-07-30T09:00:00Z", "basis_sha": "a3c9…",
  "target": { "finding_id": 90211, "session_uid": "01JGN0V9…",
    "taxonomy": {"taxonomy_id":"violations","category_id":"VIOL-003","version":6},
    "quote": { "turn_index": 87, "speaker_role": "consultant",
               "quote_text": "…", "transcript_source_id": "SRC-READAI", "source_version": 2 } },
  "history": [ {"seq":1,"decision":"request_changes","decided_by":"u:44…","decided_at":"…","note_ar":"الاقتباس لا يكفي وحده؛ أضف السياق"} ] }
```

Decision event:

```json
// POST …/decisions
{ "decision": "approve",              // approve|reject|request_changes|mark_stale|withdraw (closed; retire is admin-only via review.retire)
  "note_ar": "مخالفة مؤكدة — أسلوب غير مهني",
  "basis_sha": "a3c9…" }              // MUST match the item's current basis
```

- `201` with the appended event (`seq` incremented). History is **append-only**: there is no PUT/PATCH/DELETE on decisions anywhere in this API [DECISION OD-11/ADR-0012].
- `basis_sha` mismatch (content re-extracted/rebased since the reviewer loaded it) → `409 conflict_kind:stale_basis` — the stale-review defense (doc 16 T13) enforced at the contract level; the client re-fetches and re-presents.
- Deciding an already-terminal item → `409 conflict_kind:already_decided` unless the new event is `mark_stale` (system) — reversal happens by a *new* opposite event, never mutation.

---

## 10. Taxonomy proposals, approvals, lineage

```text
GET  /api/v1/taxonomies                                  → list (id, prefix, current_version, display_ar)
GET  /api/v1/taxonomies/{tax_id}/versions                → dense version history + approved_by
GET  /api/v1/taxonomies/{tax_id}/lineage?from=5&to=6     → typed edges
POST /api/v1/taxonomy-proposals                          → propose (taxonomy.propose; discovery pipeline uses its service token)
POST /api/v1/taxonomies/{tax_id}/versions                → publish version from approved proposals (taxonomy.approve, step-up)
```

Proposal:

```json
{ "taxonomy_id": "challenges", "kind": "INTRODUCE",       // INTRODUCE|MERGE|SPLIT|RENAME|RETIRE_ABSORBED|RETIRE_INVALID|ENTITY_ALIAS
  "payload": { "display_ar": "تعثر التمويل الجسري", "definition_ar": "…",
               "evidence_cluster_id": "CLU-CHAL-0412", "exemplar_session_uids": ["01JGN…","01JH2…"] } }
```

`201` → creates `tax.proposal` + its `ops.review_item` (single review loop, ADR-0012); approval happens in §9's endpoint, never here. Lineage response rows: `{edge_type, from_category_id, to_category_id, version, requires_reclassification}` — the contract exposes exactly doc 08 §7's model so the live-vs-frozen views (§6.4) are client-renderable.

---

## 11. Data-quality health

```text
GET /api/v1/health/data-quality?period_start=…&period_end=…       permission: dq.read
```

```json
{ "as_of": "2026-08-02T10:00:00Z",
  "sources": [ { "source_id": "SRC-READAI", "last_run_at": "2026-08-02T06:00:04Z",
                 "lag_hours": 4.0, "dlq_depth": 2, "watermark": "2026-08-02T05:41:00Z" } ],
  "identity": { "sessions_total": 16911, "consultant_resolved": 15282,
                "consultant_unresolved": 1629, "resolution_rate": 0.904 },
  "transcripts": { "with_active_transcript": 16702, "missing": 209,
                   "by_source": [ {"source_id":"SRC-READAI","version":2,"count":16702} ] },
  "extraction": { "backlog_sessions": 41, "failed_7d": 3 },
  "observations": [ { "dq_id": "DQ-001", "severity": "warning",
                      "label_ar": "جلسات بلا مستشار محسوم", "count": 1629,
                      "trend_7d": -12 } ] }
```

This endpoint is CAP-D9's serving surface (coverage numbers here must equal the exclusions the envelopes declare — same queries, one source of truth).

Ingestion ops (steward): `GET /api/v1/ingest/runs?source_id=…` returns doc 05 §1.6 reconciliation envelopes verbatim; `GET /api/v1/ingest/dlq?state=DEAD`; `POST /api/v1/ingest/dlq/{id}/replay` → `202` (audited; `DISCARDED` requires `{"reason_ar": "…"}`).

---

## 12. Exports (server-rendered, provenance-preserving)

```text
POST /api/v1/exports        (Idempotency-Key required; permission exports.create)
GET  /api/v1/exports/{export_uid}
```

```json
// POST body — source is a reference, never inline data
{ "source": "answer_artifact:AA-01K1TWJ9…",     // or "pack:PACK-2026-04-CAPB5-b" or "job:JOB-…"
  "format": "xlsx" }                             // xlsx | pdf | json
```

`202` → `{"export_uid":"EXP-01K1V2…","status":"rendering"}`; then `GET`:

```json
{ "export_uid": "EXP-01K1V2…", "status": "ready",          // rendering|ready|failed|expired
  "format": "xlsx", "sha256": "f00d…", "size_bytes": 48211,
  "download_url": "https://…/exports/EXP-01K1V2….xlsx?sig=…",   // signed, ≤15 min TTL (doc 16 §7)
  "download_expires_at": "2026-08-02T10:56:07Z",
  "provenance": { "source": "answer_artifact:AA-01K1TWJ9…",
                  "period_label": "يونيو 2026", "registry_version": "sha256:9c4f…",
                  "taxonomy_stamps": [], "code_version": "git:ab12cd3" } }
```

Contract rules: exports render **server-side from the stored envelope/pack** — never from the DOM (F6/I11; the legacy XLSX export scraped rendered HTML [FACT arch/05]); every sheet/page carries the provenance footer (period, coverage, versions, generation time) so a file forwarded by email keeps its provenance (GREENFIELD §16.2-12); JSON format returns the envelope itself; quote-bearing sources require the caller to hold `findings.read_quotes` or the export renders pseudonymized/aggregate-only per role (doc 16 §2.3); every creation writes `export_created` audit. `410` after `download_expires_at` (re-request mints a fresh URL from the stored artifact within its 180-day retention, doc 16 §3.2b).

---

## 13. Provider/source administration

```text
GET  /api/v1/sources                                   → registered source systems + adapter state
POST /api/v1/sources/{source_id}/runs                  → trigger incremental/backfill run → 202
GET  /api/v1/transcript-sources?session_uid=…          → versions available for a session
POST /api/v1/transcript-sources/{source_id}/activate   → activation/rebase (steward; step-up; audited)
```

Activation body + response:

```json
// POST …/activate
{ "scope": { "session_uids": null, "provider": "provider_x", "from_version": 1 },
  "reason_ar": "اعتماد المزود الجديد بعد اجتياز معايير المقارنة (وثيقة 06)",
  "rebase_plan": { "reextract": true, "reverify_quotes": true } }
// 202 → { "activation_id": "ACT-…", "rebase_job_id": "JOB-…", "sessions_affected": 11304 }
```

The response's `rebase_job_id` is a normal Lane-3-style job (§5) whose completion report includes the R7 re-verification residual count — the provider-swap safety contract (doc 06; MASTER_PROMPT §2's 11,304-meeting lesson). Activation is refused (`409`) while a previous rebase for the same scope is non-terminal.

---

## 13A. Operational product APIs (Owner Amendment 2026-08-03)

Contracts for the operational surfaces the Owner Amendment adds [DECISION owner 2026-08-03 / Amendment §12-17]. Every §0 convention applies unchanged: `/api/v1`, OIDC on every route, RFC 7807 problem+json errors, the §0.4 status-code table, cursor pagination (§15.1), `Idempotency-Key` on every state-creating POST (§15.2), end-exclusive periods, ULID/prefixed uids. Permissions are the doc 16 §2.6 strings; scoped roles get server-side scope checks exactly as in §3. Backing entities are doc 08's Owner Amendment additions: `pipeline_run`, `pipeline_step_run`, `source_watermark`, `dead_letter_item`, `data_quality_issue`, `reconciliation_case`, `violation_finding`, `review_case`, `review_event`, `review_assignment`, `missed_violation_report`, `label_dataset`, `detector_candidate`, `detector_release`, `shadow_result`, `monthly_infographic`, `infographic_section`, `publication_event`, `service_improvement_action`, `action_event`, `action_metric_baseline`, `notification_subscription`, `notification_delivery`. Each surface ships behind its feature flag (`nightly_consolidation_enabled`, `session_360_enabled`, `violation_review_enabled`, `monthly_infographic_enabled`, `action_center_enabled`); a disabled flag yields `404 not-found` — the surface is hidden, not half-exposed [REC]. Rate-limit classes (§15.3): reads → `read`; mutations → `admin`; quote-bearing expansions → `evidence`.

### 13A.1 Pipeline runs — «التشغيل الليلي الموحد» and the ops centre (SCR-14)

```text
GET /api/v1/pipeline-runs?run_kind=nightly&state=partial&period_start=…&cursor=…   permission: ops.runs_read
GET /api/v1/pipeline-runs/{run_id}
```

```json
{ "run_id": "RUN-20260803-N01", "run_kind": "nightly",   // nightly | incremental | backfill | reprocess
  "state": "partial",   // exact closed enum: queued → running → partial → succeeded → failed → cancelled → superseded
  "started_at": "2026-08-02T23:00:04Z", "finished_at": "2026-08-02T23:41:19Z",   // 02:00 Asia/Riyadh [ASSUME OD-28]
  "watermarks": [ {"source_id": "SRC-DATAHUB-SESSION", "before": "…", "after": "…"},
                  {"source_id": "SRC-READAI", "before": "…", "after": "…"} ],
  "counters": { "pulled": 412, "new": 371, "updated": 38, "rejected": 3,
                "sessions_completed": 58, "sessions_excluded": 2, "late_arrivals": 4 },
  "steps": [ { "step": "pull_datahub_subfeeds", "status": "partial", "duration_s": 211,
               "detail_ar": "فشل عقد SRC-DATAHUB-BENEFICIARY-EVAL؛ سُحبت بقية العقود" } ],
  "versions": { "code": "git:ab12cd3", "contracts": {"SRC-DATAHUB-SESSION": "1.2"},
                "model_ids": ["openai/gpt-oss-120b"], "prompt_shas": ["e7a9…"], "taxonomy_versions": {"VIOL": 6} },
  "dlq_ref": "/api/v1/ingest/dlq?run_id=RUN-20260803-N01",
  "manifest_uri": "s3://nip-artifacts/runs/RUN-20260803-N01/manifest.json",
  "suspected_cases_new": 6, "review_overdue": 3, "month_completeness": 0.976 }
```

`200`; `404`. A `partial` run always names the failing source and is never presented as success (I16; Amendment §4.4-3); retry counts and classified errors live in the run manifest (doc 19 §4). Entities: `pipeline_run` + `pipeline_step_run` + `source_watermark`.

```text
POST /api/v1/pipeline-runs/{run_id}/retry                permission: sources.admin · step-up · Idempotency-Key required
```
The amendment's `rerun failed items` command: re-runs **failed items only**, never the whole window. `202` + `Location` of the linked run (`run_kind: "reprocess"`, `parent_run_id` set); `409 conflict_kind:run_not_terminal` while the parent is `queued/running`; re-running never duplicates rows or review cases (idempotent natural keys + case fingerprints, doc 09).

```text
POST /api/v1/pipeline-runs/{run_id}/reprocess-sessions   permission: sources.admin · step-up · Idempotency-Key required
```
Body `{"session_uids": ["01JGN…"], "reason_ar": "…"}` (1–100 uids). The amendment's `reprocess selected sessions` command. `202`; `422` unknown session uid; `409 conflict_kind:conflict` while an overlapping reprocess is non-terminal. Reprocessing re-derives enrichment for exactly those sessions and flips affected review cases to stale via basis fingerprint (doc 16 T13) — never silently.

### 13A.2 Data-quality issues — queue «مشكلات الربط والبيانات»

```text
GET /api/v1/data-quality/issues?state=open&kind=unmatched_provider&cursor=…       permission: dq.read
```

Work items from `data_quality_issue` + `reconciliation_case` (the steward queue; §11's health board stays the aggregate view — same governed queries, one source of truth):

```json
{ "items": [ { "issue_uid": "DQI-01K2…", "kind": "unmatched_provider",
    // closed kinds: unmatched_provider | unmatched_datahub | duplicate | time_conflict |
    //               status_conflict | late_data | contract_drift
    "severity": "warning", "source_id": "SRC-DATAHUB-SESSION", "session_uid": "01JGN…",
    "opened_at": "…", "state": "open",            // open | investigating | resolved | dismissed (reason mandatory)
    "assigned_to": "u:44…", "resolution_ar": null } ],
  "next_cursor": null }
```

`200`. Resolution transitions append events (no overwrite); conflicts resolve per the doc 05 field-authority matrix — DataHub governs session facts, and a conflicting provider value is recorded, never silently merged.

### 13A.3 Session 360 (SCR-15)

```text
GET /api/v1/sessions/{session_uid}/360        permission: session360.read (blocks filtered per caller role/scope)
```

```json
{ "session_uid": "01JGN0V9…",
  "session": { "status": "completed", "scheduled_at": "…", "programme_id": "PRG-004", "channel": "عن بُعد",
               "consultant_ref": "C-004217",             // display per caller's RBAC (doc 16 §2.6)
               "field_provenance": { "status": "SRC-DATAHUB-SESSION", "scheduled_at": "SRC-DATAHUB-SESSION" } },
  "crosswalk": { "provider_meeting_ref": "…", "datahub_refs": {"session": "…"},
                 "match_method": "crosswalk", "resolution_status": "resolved" },
  "transcript": { "available": true, "source_id": "SRC-READAI", "source_version": 2 },
  "evaluations": { "beneficiary": {"rating": 2, "evaluated_at": "…"}, "consultant": {"submitted": true} },
  "findings_summary": { "suspected_violations": 1, "challenges": 3, "step_clarity": "partial",
                        "disclaimer_ar": "الحالات المعروضة مؤشرات آلية تحتاج مراجعة بشرية، ولا تعد مخالفة مثبتة قبل اعتمادها." },
  "review_cases": [ {"case_uid": "RC-01K2…", "state": "pending", "suspected": true} ],
  "actions": [ {"action_uid": "ACT-01K2…", "status": "open"} ],
  "coverage": { "sources_present": ["SRC-DATAHUB-SESSION", "SRC-READAI", "SRC-DATAHUB-BENEFICIARY-EVAL"],
                "missing": ["SRC-DATAHUB-OUTCOME"], "dq_issues_open": 0 },
  "can_report_missed_violation": true }
```

`200`; `404`. Rules: per-field source provenance is mandatory (the amendment's «مصادر كل حقل» acceptance for VS-01); suspected findings always carry the disclaimer and never render as confirmed [DECISION SD-20]; transcript **text** is absent from this payload — the transcript pane calls §6's window endpoint under `transcript.read` with its unbypassable audit (ADR-0017 split inside the composite screen). Late-arriving evaluations appear on the next consolidation without full rebuild (Amendment §3.3-5).

### 13A.4 «اشتباه مخالفة» — suspected queue, case detail, review events (SCR-08)

Violation findings previously flowed through the generic `finding_review` queue of §9; with the Owner Amendment they flow through this richer case surface (finding/case/event separation [DECISION SD-20]) — **§9 remains authoritative for the other review kinds** (taxonomy, answer keys, clusters, aliases, publication); this supersession is deliberate and one-way.

```text
GET /api/v1/violations/suspected?state=pending&violation_category=VIOL-008&severity=…&programme_id=…
    &consultant_id=…&period_start=…&period_end=…&confidence_min=…&overdue=true&reopened_by_change=true
    &needs_second_review=true&cursor=…                              permission: violations.queue_read (✔ᴬ)
```

Every response carries the fixed disclaimer header field `disclaimer_ar: "الحالات في هذه الصفحة مؤشرات آلية تحتاج مراجعة بشرية، ولا تعد مخالفة مثبتة قبل اعتمادها."` Queue rows (filters per Amendment §5.2; consultant filter scope-checked):

```json
{ "case_uid": "RC-01K2…", "violation_category": "VIOL-008", "severity": "high",
  "detector_confidence": 0.91, "programme_id": "PRG-004", "consultant_ref": "C-004217",
  "session_date": "2026-07-29", "quote_excerpt": "…اقتباس حرفي قصير…",
  "case_age_days": 4, "state": "pending",        // pending | in_review | second_review | decided | stale | reopened
  "needs_second_review": false, "reopened_by_change": false,
  "detector_release_id": "DR-2026-07-02", "taxonomy_version": 6, "sla_class": "high_priority" }
```

Approved cases leave this queue — suspected and approved are never one list or one number [DECISION SD-19/SD-20].

```text
GET /api/v1/violations/cases/{case_uid}                             permission: violations.queue_read (✔ᴬ)
```
Case detail per Amendment §5.3: `findings[]` (each `violation_finding`: verbatim quote + `turn_index` + speaker + a ±5-turn context reference resolved through §6 under `transcript.read`, detection basis — rule/model/prompt_sha/threshold, detector release id), the session block (DataHub facts per RBAC), similar **approved** cases as ids only (anchoring guard, Amendment §6.7-4), append-only `history[]` of review events, `basis_fingerprint`, and a stale warning whenever transcript/extraction changed after a decision. `200`/`404`.

```text
POST /api/v1/violations/cases/{case_uid}/review-events              permission: review.decide · Idempotency-Key required
```

**Append-only. No PUT/PATCH/DELETE exists on review events anywhere in this API** [DECISION SD-20 / OD-11].

```json
{ "decision": "reclassify",
  // closed enum — exactly six [DECISION SD-20]:
  //   true_violation        «مخالفة صحيحة»
  //   not_violation         «ليست مخالفة»
  //   reclassify            «إعادة تصنيف»       (target_category required)
  //   second_review         «تحتاج مراجعة ثانية»
  //   insufficient_evidence «أدلة غير كافية»
  //   refer_policy_owner    «إحالة لمالك السياسة»
  "target_category": "VIOL-004",
  "reason_code": "RC-V-07",            // closed per-decision reason lists (docs 08/10 own the code tables)
  "note_ar": "…",                       // mandatory for not_violation / insufficient_evidence [REC]
  "finding_ids": [90211],               // a decision may target a subset — one session can have one finding
                                        // approved and another rejected (Amendment §5.5)
  "basis_fingerprint": "a3c9…" }
```

`201` + the appended event (`seq` incremented; reviewer identity, timestamp, and transcript/detector/taxonomy stamps recorded). `409 conflict_kind:stale_basis` on fingerprint mismatch (transcript or extraction changed — client re-fetches and re-presents); `409 conflict_kind:already_decided` on terminal cases unless the event is `second_review` or a system stale/reopen transition; `422` unknown decision/category/reason. Review events never mutate detection [DECISION SD-22]: they feed `label_dataset` builds only through the doc 14 §6.5 lifecycle.

### 13A.5 Missed violation — «إضافة اشتباه لم يرصده النظام» (SCR-17)

```text
POST /api/v1/sessions/{session_uid}/missed-violation      permission: violations.report_missed · Idempotency-Key required
```

```json
{ "turn_index": 87, "turn_span": 3,
  "category": "VIOL-008",                    // or {"new_type_proposal": {"label_ar": "…", "definition_ar": "…"}}
  "reason_ar": "طلب تواصل عبر واتساب خارج القناة الرسمية" }
```

`201` → creates a `missed_violation_report` plus a `review_case` routed to second review (Amendment §6.3); after review, the label enters the false-negative datasets feeding recall estimation (EXP-11) — through the §6.5 lifecycle only, never directly into production detection [DECISION SD-22]. `422` turn outside the active transcript; `404` unknown session.

### 13A.6 Detector releases — read-only learning surface (SCR-18)

```text
GET /api/v1/detector-releases?detector=violations&status=shadow&cursor=…     permission: detector.learning_read
GET /api/v1/detector-releases/{release_id}/shadow-results?agreement=candidate_only&cursor=…
```

```json
{ "release_id": "DR-2026-08-01", "detector": "violations", "scope": ["VIOL-008"],
  "status": "shadow",       // candidate | shadow | canary | active | rolled_back | retired
  "label_dataset_version": "LDS-VIOL-008-v3", "model_id": "openai/gpt-oss-120b",
  "prompt_sha": "f21c…", "thresholds": {"VIOL-008": 0.85}, "taxonomy_version": 6,
  "offline_metrics": [ {"category": "VIOL-008", "precision": 0.94, "recall_estimate": 0.71,
                        "ci": {"low": 0.62, "high": 0.79}} ],       // per-category, never one total (Amendment §6.5)
  "promoted_by": null, "promoted_at": null, "rollback_of": null }
```

Shadow rows (`shadow_result`): `{session_uid, current: finding|null, candidate: finding|null, agreement: both|current_only|candidate_only|category_differs, adjudication: pending|resolved}` — quote expansion follows the evidence rules (`findings.read_quotes`, audited). **No mutating endpoint exists on this surface**: promotion, canary, and rollback are doc 14 §6.5 workflow actions under `detector.promote` (step-up), recorded as audited release transitions — never an API side effect, never a reviewer click [DECISION SD-22]. `200`; `404`.

### 13A.7 Monthly infographic — «نبض خدمة الاستشارات والإرشاد — ملخص الشهر» (SCR-20)

```text
GET /api/v1/infographics/monthly/{period}         period = YYYY-MM
    permission: infographic.read_published (published edition) · infographic.read_draft (?state=draft…)
```

```json
{ "infographic_uid": "INF-2026-07", "period": {"start": "2026-07-01", "end": "2026-08-01", "label": "يوليو 2026"},
  "state": "approved",     // draft → data_review → content_review → approved → published → superseded | retracted
  "sections": [ { "section_id": "service_pulse", "payload": { "…": "structured numbers only, each traceable to a Metric Result with allowed_literals refs" } } ],
  "completeness": { "data_completeness": 0.983, "gate": 0.98, "override": null },   // override text renders on the face
  "violations_note_ar": "أرقام المخالفات معتمدة بشريًا فقط؛ حجم قائمة الاشتباه يظهر كرقم مستقل",
  "renders": { "web": "/infographics/INF-2026-07", "pdf_uri": "s3://…", "png_uri": "s3://…",
               "payload_sha256": "9d1a…" },        // web + PDF + PNG payloads byte-identical [DECISION SD-19]
  "approvals": [ {"stage": "data_review", "by": "u:9c…", "at": "…"} ],
  "published_at": null, "superseded_by": null,
  "footer": { "sessions_used": 1258, "sessions_matched": 1401, "sessions_total": 1460,
              "exclusions": [ {"code": "NO_TRANSCRIPT", "count": 59} ],
              "taxonomy_versions": {"VIOL": 6, "CHAL": 4}, "corpus_snapshot_id": "CS-…", "drilldown": "/api/v1/…" } }
```

`200`; `404` (drafts are existence-hidden without `infographic.read_draft`). Every number is programmatically drawn from structured Metric Results — no LLM-computed figure, no image-generation model; optional LLM phrasing passes the R6 gate [DECISION SD-19]; the general edition carries no consultant names [ASSUME OD-31]; leadership violation figures are approved-only with queue size (`suspected_cases_open`) as its own figure.

```text
POST /api/v1/infographics/{infographic_uid}/approve      permission: infographic.approve · Idempotency-Key required
```
Body `{"stage": "data_review" | "content_review", "note_ar": "…"}` — Data Owner reviews coverage/numbers, Service Owner reviews meaning (Amendment §7.5). `201` approval event (`publication_event` chain, append-only); `409 conflict_kind:version_conflict` out of state order.

```text
POST /api/v1/infographics/{infographic_uid}/publish      permission: infographic.publish (Publication Authority
                                                         per OD-10/OD-29) · step-up (§3) · Idempotency-Key required
```
`201` publication (permanent link minted; channels per OD-29); `409` if not `approved`, or if the completeness gate is unmet without a documented override — an override always renders on the face of the infographic (Amendment §7.3/§4.4-5). Reissue creates a new version and never edits the published one; retraction follows the §8.2 supersession doctrine.

### 13A.8 Action Center — «مركز الإجراءات» (SCR-19)

```text
GET  /api/v1/actions?owner=…&status=open&overdue=true&period_start=…&cursor=…    permission: actions.read
     (executive_viewer receives the aggregate roll-up view only [ASSUME OD-34])
POST /api/v1/actions                                     permission: actions.manage · Idempotency-Key required
```

```json
{ "title_ar": "خطة تدريب على إغلاق الجلسات بخطوات واضحة",
  "description_ar": "…", "owner_sub": "u:7f3a…", "due_date": "2026-08-20",
  "source_ref": "infographic:INF-2026-07",     // or finding:… | digest:… | dashboard:…
  "metric_ref": "clear_steps_rate" }
```

`201` `{action_uid, baseline_captured: true}` — `action_metric_baseline` is captured at creation so post-action comparison is honest (baseline → intervention → follow-up; association only, never causal claims — doc 10).

```text
PATCH /api/v1/actions/{action_uid}                        permission: actions.manage (closing: actions.close)
```
Body `{"status": "in_progress" | "blocked" | "done_pending_verification" | "closed", "note_ar": "…", "evidence_ref": "…"}` with `If-Match` ETag. Every transition appends an `action_event` — no overwrite, full history. `200`; `409 conflict_kind:version_conflict` on stale ETag; `403` closing without closure authority [ASSUME OD-34: service owner owns actions; leadership sees aggregate status/impact].

### 13A.9 Subscriptions — digest and notices (SCR-21)

```text
GET  /api/v1/subscriptions                               permission: digest.subscribe (own subscriptions)
POST /api/v1/subscriptions                               permission: digest.subscribe · Idempotency-Key required
```
Body `{"kind": "morning_digest" | "infographic_publication" | "alert_class", "channel": "portal" | "email", "scope": {…role-checked…}}` [ASSUME OD-29 channels: Portal + approved email; others later]. `201`; `422` unknown kind/channel; `DELETE /api/v1/subscriptions/{subscription_id}` → `204` (idempotent). Deliveries land in `notification_delivery` with outcome and are audited (`notification_sent`); payload policy per doc 16 §2.6-5 — counts and login-gated links only, approved/published artifacts only, no suspected-case detail. Digest content list: doc 19 §4 (Amendment §11.1).

---

## 14. Meta, audit, probes

```text
GET /api/v1/meta/version       → { "code_version":"git:ab12cd3", "registry_version":"sha256:9c4f…", "api_version":"1.4.0" }
GET /api/v1/meta/system-state  → { "system_state":"normal", "since":"…", "detail_ar": null }   // degraded states named honestly (I16)
GET /api/v1/meta/session       → { "sub":"u:7f3a…", "role":"analyst", "display_name":"…" }
GET /api/v1/audit-events?actor=…&action=…&object_ref=…&from=…&to=…&cursor=…    permission: audit.read
GET /healthz                   → 200 static liveness (no body secrets)
GET /readyz                    → 200/503 with per-dependency booleans (db, minio, vault, idp, groq) — the legacy /health never touched the DB (ISS-06); this one fails when the DB fails
```

---

## 15. Pagination, filtering, idempotency, rate limits

### 15.1 Pagination + filtering

- **Cursor-based**: `?limit=` (default 25, max 100) `&cursor=<opaque>`; response `{"items":[…], "next_cursor": "…"|null}`. `total` included only where it is one cheap aggregate (review queues, publications); large sets (findings, audit) omit it — an honest contract beats an estimated count.
- Cursors encode `(sort_key, id)` — stable under concurrent inserts; expired/foreign cursor → `422 INVALID_CURSOR`.
- Filtering: exact-match query params named after schema fields; period filters always the pair `period_start`/`period_end` (end-exclusive); no free-text query params on data endpoints (search happens through governed capabilities, not ad-hoc API grep).
- Sorting: fixed per endpoint (documented in OpenAPI); no client-supplied `ORDER BY` — the sort column set is closed (the `ORDER BY RANDOM()` and injection classes both die here).

### 15.2 Idempotency keys

- Header `Idempotency-Key: <uuid>` — **required** on every state-creating POST (§1 table); server stores `(key, principal, endpoint, request_sha, response)` for 24 h in Postgres (`serve.idempotency_record` [NEW table — doc 08 addendum]).
- Same key + same body → replay the stored response with `Idempotency-Replayed: true` (no double job, no double export, no double decision — network retries become safe).
- Same key + **different** body → `409 conflict_kind:idempotency_mismatch` (never guess which request the client meant).
- `DELETE` endpoints are inherently idempotent (repeat → same terminal state, `204`).

### 15.3 Rate limits

Per-principal token buckets, enforced at the API layer (proxy layer additionally per-IP — doc 16 D-12). Headers on every response: `RateLimit-Limit`, `RateLimit-Remaining`, `RateLimit-Reset`; `429` adds `Retry-After`.

| Class | Endpoints | Default limit [REC — tune in pilot] |
|---|---|---|
| interactive | turns (3), metric-queries (6) | 30/min/user; burst 10 |
| read | catalogues, packs, reviews, DQ, jobs GET | 240/min/user |
| evidence | transcript-window (7), findings (11) | 60/min/user (audited anyway; the limit slows scraping — T01/T09) |
| heavy | exports (24), job submit (8), rerun (12) | exports 10/h; jobs: 2 concurrent + 10/day/user [REC — owner tunes in pilot (§18); admission quota, never a cost ceiling] |
| admin | sources, activation, publications | 30/min/user + step-up |

Rate limits protect availability (T08); they are never a correctness lever — a Lane-3 job inside the platform has no cost ceiling (ADR-0020); only its *creation* rate is governed.

---

## 16. API versioning and deprecation policy

1. **Major version in the path** (`/api/v1`). Breaking changes (removing/renaming a field, changing semantics, narrowing an enum used in requests) mint `/api/v2`; v1 and v2 run in parallel **≥6 months** [REC].
2. **Additive changes are non-breaking** and flow into v1 continuously: new optional request fields, new response fields (clients ignore unknowns — §0.7), new endpoints, new enum members in **response-only** enums (documented list; clients must render unknown reason-code members with the generic Arabic fallback string).
3. **Closed request enums never widen silently**: adding a request-enum value is minor but announced in the changelog; adding a *reason code* is a doctrine change requiring the CORE-BRIEF/doc 22 registry update first (I12 discipline applies to the API surface too).
4. **Deprecation signalling**: deprecated endpoints/fields return `Deprecation: @<unix-ts>` and `Sunset: <http-date>` headers plus a `Link: <changelog>; rel="deprecation"`; after sunset → `410 gone` with the successor named in the problem body. `GET /api/v1/meta/version` exposes `api_version` (semver: minor = additive).
5. **OpenAPI 3.1 document is the normative artifact**, generated from the FastAPI models, versioned in the repo, and diffed in CI: a PR that changes the spec without a changelog entry fails; a PR that breaks compatibility without a major-version plan fails (structural check, same spirit as I12's registry gate).
6. **Consumers**: the NIP frontend, the export renderer, and (post-pilot, per OD-22/BI decision) BI tools — all pinned to major versions; `pipeline_service` clients pinned per deploy (lockstep monolith deployment makes internal skew a non-issue at pilot scale).

---

## 17. Cross-cutting contract tests (doc 15 owns the harness)

| Test | Assertion |
|---|---|
| API-1 | OpenAPI walk: every route has auth + permission metadata (feeds G-SEC-1) |
| API-2 | Every 2xx response validates against its published JSON Schema; every error validates against problem+json |
| API-3 | **No-200-on-failure sweep**: fault-injected dependencies (DB down, Groq down, verifier reject) produce 503/500/`kind:boundary` respectively — never 200 with an exception string (ISS-13 pin) |
| API-4 | Idempotency: duplicate POST with same key → single side-effect + replayed response; different body → 409 |
| API-5 | Pack immutability: fetch, reissue, re-fetch old `pack_uid` → byte-identical envelope, `superseded_by` set |
| API-6 | Review append-only: no mutating verb succeeds on a decision event; `stale_basis` round-trip |
| API-7 | Evidence audit: N transcript-window calls ⇒ N `evidence_accessed` rows (joins doc 16 G-SEC-9) |
| API-8 | INCOMPLETE contract: kill a partition in staging job → result carries `DEEP_JOB_INCOMPLETE` + exclusions; `complete` shape absent |
| API-9 | Arabic integrity: all `*_ar` fields NFC, no HTML tags (renderer-side escaping is not the API's excuse to ship markup) |
| API-10 | Version discipline: CI diff of OpenAPI vs previous release classifies changes additive/breaking; breaking without major bump fails |
| API-11 | Violation cases (§13A.4): only the six-decision enum is accepted; no mutating verb succeeds on a review event; `stale_basis` round-trip on fingerprint change; `GET /violations/suspected` never returns an approved-state case (suspected ≠ approved — SD-19/SD-20) |
| API-12 | Infographic (§13A.7): publish refused below `approved` or on a failed completeness gate without documented override; published web/PDF/PNG payload sha256 identical; general edition contains no consultant name pattern [ASSUME OD-31] |
| API-13 | Pipeline runs (§13A.1): `retry` re-runs failed items only (no duplicate rows/cases on replay); a `partial` run is never rendered as `succeeded` anywhere in the payload |

---

## 18. Open decisions and revisit triggers

- **Job/export quota values** (§15.3) — [ASSUME] defaults stated; owner tunes in pilot; quota ≠ cost ceiling (ADR-0020 boundary is documented at the endpoint).
- **SSE for job progress** (§5.2) — polling ships first; revisit when the ops dashboard needs sub-5 s updates.
- **BI exposure** (OD-22/BI): if approved, `POST /api/v1/metric-queries` gets a read-only service-account class and per-client rate limits; masked-view enforcement already in place (doc 16 §2.4) — no new surface needed.
- **Public problem-type host** — final domain per OD-01; slugs stable regardless.
- **`serve.idempotency_record`** table — flagged to doc 08 as an addendum (small, self-pruning at 24 h).
- Revisit triggers: v2 trigger = first genuinely breaking need (e.g., multi-tenant scoping); reason-code enum change (doctrine-level, via doc 22); provider-swap rebase API load (§13) if replacement provider requires bulk re-activation semantics beyond the current scope shape.

# 11 — Semantic Layer and Curated Capabilities
**Platform:** Nwafeth Intelligence — منصة نوافث لذكاء الجلسات الاستشارية (Monsha'at Advisory Session Intelligence Platform)
**Status:** Draft for owner review · **Date:** 2026-08-02 · **Author:** Planning package (Fable 5)
**Depends on:** 04 (capability catalogue), 05 (source contracts), 07 (target architecture), 08 (data model), 10 (methodology) · **Feeds:** 12, 13, 15, 17, 18, 23, 24, 25
**Sources used:** GREENFIELD §8.4–8.5, §11, §13, §23-11; MASTER_PROMPT §5.2, §5.5, §6 (R1–R2, R6, R10), Appendix A; CORE-BRIEF §4–6, §11–13; doc 08 schema names; OWNER AMENDMENT 2026-08-03 §9.3 (VS-07), §12 doc 11 item, §13
**Amended:** 2026-08-03 — Owner Amendment integrated: VS-07 MVP-five serving set (§8.3, SD-18); operational/review metric registry §8.4 (names exact); `review_state_scope` field + gate G-REG-8 (doc 10 MR-17) [DECISION owner 2026-08-03]

---

## 0. Purpose and scope

This document specifies **Layer 4 (semantic metric layer)** and **Layer 5 (curated analytical capabilities)** of the target architecture (doc 07): the governed registries, the deterministic spec→SQL compiler, the typed result envelope every fact-producing surface returns, the starter metric registry, the curated-capability contract, and the structural `SPEC_ANSWERED`/`CURATED` boundary check.

Binding doctrine restated once: the model **never writes SQL or names schema** (I2), **never computes numbers** (I3), every aggregate carries an **explicit period decision** (I4), every answer **declares scope + coverage** (I8), structured results **precede presentation** (I11), and there is **exactly one governed registry** for capabilities and one for metrics/dimensions — an unregistered id fails the build (I12). The legacy system's eight unsynchronized intent registries (F5/F8) are the defect this layer makes structurally unreachable.

---

## 1. Registry architecture — one registry, code-as-data, hash-versioned

**[REC] Registries are declarative YAML files in the repository**, under `registry/` (`metrics/*.yaml`, `dimensions.yaml`, `filters.yaml`, `capabilities/*.yaml`, `clarification_options.yaml`), validated in CI against the meta-schemas in this document, loaded once at process start, and snapshotted to `ops.registry_snapshot` at deploy.

- **Alternatives considered:** (a) database-first registry editable via admin UI — rejected: registry changes alter what the platform is allowed to claim and must pass code review, golden-suite re-run, and τ/δ recalibration (doc 12 §4), which a live DB edit bypasses; (b) Python constants — rejected: harder for stewards/reviewers to diff, and mixes executable code with reviewable data.
- **Revisit trigger:** stewards demonstrably need same-day metric additions without a deploy → introduce a `draft` status editable in DB that is *serving-invisible* until promoted through the file-based flow.

**`registry_version`** is a SHA-256 over the canonical serialization of *all* registries **plus every taxonomy's current category-id list** [DECISION MASTER_PROMPT §6 R16, carried forward]. It appears in every envelope (`provenance.registry_version`), keys the Lane-1 spec cache (doc 12 §7), and a taxonomy version bump therefore necessarily invalidates cached specs.

```sql
CREATE TABLE ops.registry_snapshot (
  registry_version char(64) PRIMARY KEY,          -- sha256 hex
  git_sha          char(40) NOT NULL,
  content          jsonb    NOT NULL,             -- full canonical serialization
  deployed_at      timestamptz NOT NULL DEFAULT now(),
  deployed_by      text NOT NULL                  -- CI identity
);
```

**Build gates (CI, deterministic — I18):**

| Gate | Fails the build when |
|---|---|
| G-REG-1 | any YAML fails its meta-schema |
| G-REG-2 | a capability, metric, dimension, filter, or clarification option is referenced anywhere in code/prompts but absent from the registry (I12) — grep-based structural discovery, no hand-list (F5/F8) |
| G-REG-3 | a capability lacks `classification` (`SPEC_ANSWERED`/`CURATED`) — §10 |
| G-REG-4 | a metric enum destined for a tool schema exceeds 40 members without a declared family split (doc 12 §5.4) |
| G-REG-5 | a non-zero `tolerance` lacks a `tolerance_reason` naming the rounding step (R6.3) |
| G-REG-6 | Lane-0 τ/δ fields changed without a committed recalibration artifact (doc 12 §4) |
| G-REG-7 | a renderer module contains a bare formatted-number f-string outside `num()/pct()/count()` (R6.2) |
| G-REG-8 | a violation/review-state metric lacks `review_state_scope`, or a compiled spec mixes `approved_only` and `suspected_queue_only` scopes in one result, or a `suspected_queue_only` metric is placed on a published/leadership surface (doc 10 MR-17) [DECISION owner 2026-08-03 / Amendment §13] |

---

## 2. Metric registry — entry schema

Every metric is one YAML document. Full field list (all fields required unless marked optional; `additionalProperties: false` in the meta-schema):

| Field | Type | Semantics |
|---|---|---|
| `id` | slug, unique | e.g. `attendance_rate`. The only name the planner may emit (I2). |
| `arabic_label` | string | Renderer + `list_capabilities` display, e.g. «نسبة الحضور». |
| `description_ar` | string | One sentence, shown in the catalogue UI (doc 18). |
| `status` | enum `draft·active·retired` | Only `active` is planner-visible. `retired` ids stay resolvable for audit replay, never for new answers. |
| `base` | schema-qualified name | Base table/view from doc 08, e.g. `core.attendance_fact`, `findings.finding` (+ `kind` pin), `transcript.turn`. |
| `finding_kind` | optional slug | Required when `base = findings.finding`; pins one of the 17 seeded kinds (doc 08 §6.2). |
| `join_spine` | const `core.advisory_session` | Every metric joins through the canonical hub; the compiler adds `session_extraction_state` / `active_transcript` pointer joins per `basis` (doc 08 L6). |
| `grain` | enum `session·finding·turn·rating·evaluation·mention·cluster` | The unit the aggregation counts. E.0-2: session-grain (`COUNT(DISTINCT advisory_session_id)`) is the default for ranking; row-grain metrics must say so on the face of the answer. |
| `aggregation` | enum `count·distinct_sessions·rate·avg·median·p90·distribution·share` | Closed; the compiler owns each implementation. |
| `numerator_predicate` | predicate ref | **A named reference into repository builder functions** (e.g. `pred.attendance_attended`), never a SQL string in YAML [REC — SQL fragments in data files were rejected: they re-open the F10 "correctness by convention" hole; predicate refs are compiled code under test. Revisit trigger: >30 predicates and churn makes the indirection the bottleneck]. |
| `denominator` | object | `{predicate_ref, description_ar}` — e.g. `{pred.scheduled_known_status, "الجلسات المجدولة المعروفة الحالة في الفترة"}`. Mandatory for `rate`/`share` (CORE-BRIEF §13.4: every metric declares unit + denominator). |
| `unit` | enum `count·pct·per_100_sessions·minutes·score·ms` | Comparisons across months always use `per_100_sessions` (E.0-1: monthly volume swings 2.7×). |
| `allowed_dimensions` | array of dimension ids | Closed; anything else hard-fails (§5). |
| `allowed_filters` | array of filter keys | Closed; subset of §4. |
| `period_column` | schema-qualified column | e.g. `core.advisory_session.scheduled_date`. |
| `period_scope` | enum `scheduled·actual·extraction` | Which date semantics the period clause binds to. Cancellation/no-show metrics use `scheduled` (an unattended session has no actual date); transcript-derived metrics use `actual`. |
| `default_period` | enum `all_time_declared·last_closed_month·last_closed_quarter·rolling_3_months` | Consumed when `resolve_period` returns `NO_PERIOD_MENTIONED` (doc 12 §2.2). This is what makes I4's "no silent all-time" enforceable: all-time is always a *declared* decision. |
| `comparison_windows` | array enum `previous_month·previous_quarter·same_month_previous_year` | Declared, harness-selected; feeds `period.comparisons` and the verifier's `declared_periods` set (doc 12 §10). |
| `support_threshold` | int, default 30 | Below n, the cell reports the count, suppresses the percentage, and is marked (E.0-4; shared constant, never per-handler literals). |
| `ci` | enum `wilson_95·none` | `wilson_95` mandatory for every proportion (E.0-5). |
| `tolerance` | number, default 0 | R6: zero numeric tolerance by default. |
| `tolerance_reason` | optional string | Required when `tolerance ≠ 0`; names the rounding step (gate G-REG-5). |
| `rounding` | object | `{step: 0.1, authority: pct}` — the *only* formatter allowed to render this metric's values (R6.2 emitted allowed-literals). |
| `caveats_ar` | array of strings | Rendered verbatim under every answer using the metric (e.g. overlap caveat on talk-share). |
| `coverage_formula` | object | `{used, matched, total}` as predicate refs — e.g. total = sessions in period scope, matched = passing filters, used = with the required enrichment present. Drives the envelope `coverage` block (I8). |
| `exclusion_codes` | array | Closed exclusion reasons this metric can emit, each `{code, label_ar}` — e.g. `{CONSULTANT_UNRESOLVED, "جلسات لم يُحسم مستشارها"}` (doc 08 §4 worked example). |
| `derivable_from_transcripts` | bool | Feeds doc 12 §3's Lane-3-offer classifier: could a Lane-3 job compute this signal from transcript text if the enriched form is missing? |
| `review_state_scope` | conditionally required enum `approved_only·suspected_queue_only` | Mandatory for violation/review-state metrics, absent elsewhere (doc 10 MR-17) [DECISION owner 2026-08-03 / Amendment §13]. `approved_only` = computed from review-approved findings; the only scope allowed on published/leadership surfaces. `suspected_queue_only` = workload/queue figures at case grain; never mixed with an approved series in one output (gate G-REG-8). |
| `tests` | array of golden-suite ids | ≥1 required for `active` status (doc 15). |

### 2.1 Worked entry — `attendance_rate`

```yaml
id: attendance_rate
arabic_label: نسبة الحضور
description_ar: نسبة الجلسات المجدولة التي حضرها المستفيد فعلياً خلال الفترة
status: active
base: core.attendance_fact
join_spine: core.advisory_session
grain: session
aggregation: rate
numerator_predicate: pred.attendance_attended        # attendance_status = 'attended'
denominator:
  predicate_ref: pred.scheduled_known_status
  description_ar: الجلسات المجدولة المعروفة الحالة في الفترة
unit: pct
allowed_dimensions: [consultant, programme, service_category, channel, month, quarter]
allowed_filters: [programme, channel, service_category, consultant_id]
period_column: core.advisory_session.scheduled_date
period_scope: scheduled
default_period: last_closed_month
comparison_windows: [previous_month]
support_threshold: 30
ci: wilson_95
tolerance: 0
rounding: {step: 0.1, authority: pct}
caveats_ar:
  - «تُستبعد الجلسات ذات حالة الحضور غير المعروفة من المقام وتُعلن ضمن التغطية.»
coverage_formula:
  total:   pred.sessions_scheduled_in_period
  matched: pred.after_filters
  used:    pred.known_attendance_status
exclusion_codes:
  - {code: STATUS_UNKNOWN, label_ar: حالة الحضور غير معروفة من المصدر الداخلي}
derivable_from_transcripts: false      # attendance is an operational fact, not transcript content
tests: [GT-MET-ATT-01, GT-MET-ATT-02, GT-MET-ATT-SUPP-01]
```

---

## 3. Dimension registry

Each dimension declares: `id`, `arabic_label`, `value_source` (where legal values come from), `value_kind`, `cardinality_class`, `availability`, and `derivable_from_transcripts`. Values are **closed**: either a small in-registry enum, a governed dimension table, or a versioned taxonomy — never free text (I2).

| Dimension id | Arabic | Value source | Kind | Cardinality | Availability |
|---|---|---|---|---|---|
| `consultant` | المستشار | `core.consultant` (`consultant_uid`) | table | ~10² | available |
| `programme` | البرنامج | `core.programme` | table | ~10¹ | available |
| `service` | الخدمة | `core.service` | table | ~10¹–10² | available |
| `channel` | القناة | `core.channel` | table | <10 | available |
| `service_category` | فئة الخدمة | `core.service` category attr (48 distinct legacy values; 77.3% populated [FACT CORE-BRIEF §11]) | table | ~50 | available — remainder is a **named coverage exclusion**, never silent |
| `government_entity` | الجهة الحكومية | `tax.entity` (+ `tax.entity_alias` resolution) | entity registry | ~10² canonical | available (extraction-based; mentions, not assessments) |
| `business_sector` | القطاع الاقتصادي للمنشأة | — (external identity source absent) | — | — | **UNAVAILABLE** → `declare_unanswerable(DIMENSION_NOT_AVAILABLE)`; `derivable_from_transcripts: false` |
| `violation_category` | فئة المخالفة | `tax` taxonomy `VIOL` (versioned; seeds VIOL-001…008) | taxonomy | 8+ | available |
| `challenge_category` | فئة التحدي | taxonomy `CHAL` | taxonomy | ~10¹–10² | available |
| `satisfaction_polarity` | قطبية مؤشر الرضا | in-registry enum `positive·negative·neutral` | enum | 3 | available |
| `pressure_class` | فئة الضغط | enum `urgency·confusion·frustration·distress·other` | enum | 5 | available — class imbalance caveat (distress n=544 [FACT CORE-BRIEF §11]) |
| `impact_level` | مستوى الأثر | enum `high·medium·low` (IMP taxonomy view) | enum | 3 | available when enrichment ran — else `DATA_NOT_ENRICHED`, never zeros |
| `step_clarity` | وضوح الخطوات | enum `clear·partial·none` | enum | 3 | available |
| `severity` | الشدة | enum `potential·confirmed` (violations) | enum | 2 | available |
| `session_status` | حالة الجلسة | `core.session_status_fact` closed vocabulary (per OD-07 dictionary) | enum | <10 | available |
| `attendance_status` | حالة الحضور | enum `attended·no_show·cancelled_beneficiary·cancelled_consultant·cancelled_system·unknown` [ASSUME OD-07: exact source vocabulary pending the internal data dictionary; mapping table owned by doc 05] | enum | 6 | available |
| `rating_band` | نطاق التقييم | enum `1·2·3·4·5` [ASSUME OD-07: 1–5 scale] | enum | 5 | available |
| `speaker_role` | دور المتحدث | enum `consultant·beneficiary·other·unresolved` | enum | 4 | available (5.7% unresolved declared as exclusion) |
| `month` | الشهر | derived `date_trunc('month', period_column)` — a GROUP BY bucket *inside* the harness-injected period, never a period decision | derived | ≤ corpus months | available |
| `quarter` | الربع | derived, as above | derived | ≤ corpus quarters | available |
| `question_cluster` | عنقود الأسئلة | `findings.cluster` (`QST-…`, approved labels only) | cluster registry | ~10²–10³ | available after R-P2 clustering |
| `decision_cluster` | عنقود القرارات | `findings.cluster` (`DEC-…`) | cluster registry | ~10² | available after clustering |

**The three-way «قطاع» rule (binding, CORE-BRIEF §13.3).** A question saying «قطاع» is ambiguous across **three registered dimensions** and is never resolved by guessing:

1. `service_category` — service/business domain of the consultation (الابتكار، القانونية، دراسة الجدوى، الإقراض والتمويل…). Populated and usable; the unresolved ~22.7% is a named coverage exclusion [FACT MASTER_PROMPT App-A].
2. `government_entity` — the government body mentioned in the session (what the legacy "clear steps by sector" actually grouped by).
3. `business_sector` — the beneficiary firm's industry. **Genuinely unavailable** until an external identity source exists; the only one that returns `DIMENSION_NOT_AVAILABLE`.

The registry records which of the three every capability means; Lane 2 clarification asks via registered options `OPT-sector-service_category` / `OPT-sector-government_entity` / `OPT-sector-business_sector` (doc 12 §5.3, tool 9).

---

## 4. Filter registry

Filter keys are a closed set; each declares its value domain and the **only** operators the compiler implements for it. No free-text value ever reaches SQL (I2); values outside the small in-schema enums are validated **registry-side** with a hard fail listing up to 10 nearest valid values.

| Filter key | Value domain | Operators | Notes |
|---|---|---|---|
| `consultant_id` | `core.consultant.consultant_uid` | eq, in(≤10) | The planner uses ids already resolved by `resolve_entity` verbatim (doc 12 §2.3); it never spells a name. |
| `programme` | `core.programme` slug | eq, in | |
| `channel` | `core.channel` slug | eq | |
| `service_category` | registry values (~48) | eq, in(≤5) | Large vocabulary ⇒ registry-side validation, not JSON-Schema enum (gate G-REG-4 interplay). |
| `government_entity` | `tax.entity.entity_id` | eq, in(≤5) | Alias resolution happens in `resolve_entity`, never in the filter. |
| `violation_category` | `VIOL-001…` current taxonomy version | eq, in | Envelope must stamp taxonomy (I13). |
| `challenge_category` | `CHAL-…` | eq, in | |
| `severity` | `potential·confirmed` | eq | |
| `satisfaction_polarity` | `positive·negative·neutral` | eq | |
| `pressure_class` | 5-member enum | eq | |
| `impact_level` | `high·medium·low` | eq, in | |
| `step_clarity` | `clear·partial·none` | eq | |
| `session_status` | closed vocabulary | eq, in | |
| `attendance_status` | 6-member enum | eq, in | |
| `speaker_role` | 4-member enum | eq | `search_evidence` shares this registry (doc 12 §5.3). |
| `rating_band` | 1…5 | eq, in | |
| `has_transcript` | bool | eq | Compiles to an `active_transcript` existence join. |
| `validated_only` | bool — **fixed `true` for findings-based metrics; not planner-settable** | — | Unvalidated findings are triage material, never answer material (I18). Appears in the registry so the fix is data, not convention. |

**There is deliberately no `period`, `date_from`, `date_to`, or `month_of` filter key.** Period is harness-injected (I4, §6.1); a spec containing any date-shaped key fails schema validation before the registry is even consulted.

---

## 5. The spec→SQL compiler contract

### 5.1 Input: the validated `MetricSpec`

The **only** thing the planner may emit for a metric question. JSON Schema (strict — `additionalProperties: false` everywhere; strict structured outputs on `openai/gpt-oss-120b` require all fields required, so optional semantics use explicit `null` [FACT groq-docs 2026-08-02]):

```json
{
  "$id": "nip:schemas/metric-spec/v1",
  "type": "object",
  "additionalProperties": false,
  "required": ["metric", "dimensions", "filters", "sort", "limit"],
  "properties": {
    "metric":     { "enum": ["sessions_count", "attendance_rate", "cancellation_rate", "…all active registry ids, generated at build; ≤40 or family-split (G-REG-4)"] },
    "dimensions": { "type": "array", "maxItems": 2,
                    "items": { "enum": ["consultant", "programme", "service", "channel",
                               "service_category", "government_entity", "violation_category",
                               "challenge_category", "satisfaction_polarity", "pressure_class",
                               "impact_level", "step_clarity", "severity", "session_status",
                               "attendance_status", "rating_band", "speaker_role",
                               "month", "quarter", "question_cluster", "decision_cluster"] } },
    "filters":    { "type": "object", "additionalProperties": false,
                    "properties": {
                      "consultant_id":       { "type": ["string","null"] },
                      "programme":           { "type": ["string","null"] },
                      "channel":             { "type": ["string","null"] },
                      "service_category":    { "type": ["string","null"] },
                      "government_entity":   { "type": ["string","null"] },
                      "violation_category":  { "type": ["string","null"] },
                      "challenge_category":  { "type": ["string","null"] },
                      "severity":            { "enum": ["potential","confirmed",null] },
                      "satisfaction_polarity": { "enum": ["positive","negative","neutral",null] },
                      "pressure_class":      { "enum": ["urgency","confusion","frustration","distress","other",null] },
                      "impact_level":        { "enum": ["high","medium","low",null] },
                      "step_clarity":        { "enum": ["clear","partial","none",null] },
                      "session_status":      { "type": ["string","null"] },
                      "attendance_status":   { "type": ["string","null"] },
                      "speaker_role":        { "enum": ["consultant","beneficiary","other",null] },
                      "rating_band":         { "enum": [1,2,3,4,5,null] },
                      "has_transcript":      { "type": ["boolean","null"] } },
                    "required": ["consultant_id","programme","channel","service_category",
                                 "government_entity","violation_category","challenge_category",
                                 "severity","satisfaction_polarity","pressure_class","impact_level",
                                 "step_clarity","session_status","attendance_status","speaker_role",
                                 "rating_band","has_transcript"] },
    "sort":  { "enum": ["value_desc", "value_asc", "label_asc", null] },
    "limit": { "type": ["integer","null"], "minimum": 1, "maximum": 50 }
  }
}
```

**Note what is absent:** no period, no date, no SQL, no table, no column, no aggregation override, no denominator override. Period is injected by the harness and the aggregation/denominator are registry data — the model selects, it never defines (I2/I3/I4; the F16/ISS-01 defect made structurally unreachable).

### 5.2 Compiler stages (all deterministic, all logged per R13)

```
validate_schema(spec)                     # JSON Schema, additionalProperties:false
  └─ fail ⇒ machine-readable error: {field, given, valid_values[≤10], hint_ar}; one repair allowed (R5)
validate_registry(spec)                   # semantic layer
  ├─ metric active? dimension ∈ allowed_dimensions? filter ∈ allowed_filters?
  ├─ filter value ∈ value domain? (registry-side for large vocabularies)
  ├─ dimension availability: business_sector ⇒ DIMENSION_NOT_AVAILABLE (never compiled)
  └─ fail ⇒ same error shape; unknown anything is a HARD FAIL listing valid values — never a nearest-match substitution (I5, ISS-02)
inject_period(resolved_facts, registry)   # I4 — §6.1
build_sql(spec, period)                   # repository builders ONLY (§5.3)
execute(read_only_role, statement_timeout=5s, row_cap)
shape_envelope(result, registry, period)  # §7: coverage, suppression, CI, taxonomy stamp
fingerprint = sha256(normalized_sql + ordered_params)
```

**Worked hard-fail.** Spec `{"metric": "cancellations_rate", …}`:

```json
{ "ok": false, "error": {
    "code": "VALIDATION_FAILED_FIELD",
    "field": "metric",
    "given": "cancellations_rate",
    "valid_values": ["cancellation_rate", "no_show_rate", "attendance_rate", "sessions_count"],
    "hint_ar": "المقياس غير مسجل؛ الأقرب: cancellation_rate" } }
```

One repair attempt, then `VALIDATION_FAILED` ends the turn (R5). The executor — not the prompt — enforces this (doc 12 §6).

### 5.3 Repository builders — the only place SQL lives

`repository/` owns raw SQL; nothing above it may author a fragment (doc 07 module contract). The closed builder set:

| Builder | Contract |
|---|---|
| `period_clause(period, period_column, period_scope)` | End-exclusive `>= start AND < end` always; the *sole* author of date predicates. |
| `session_scope(filters)` | Joins `core.advisory_session` to dimension tables; parameterized values only; identifiers exclusively from registry constants. |
| `active_findings_join(kind)` | Joins `findings.finding` **through `findings.session_extraction_state`** (current-pointer, doc 08 L6) and pins `validation_status='verified'` + the finding `kind`. |
| `active_transcript_join()` | Joins `transcript.turn` through `transcript.active_transcript` — serving can never read an inactive source (I6). |
| `predicate(ref)` | Resolves a registry predicate ref (`pred.*`) to a tested SQL fragment with bound parameters. |
| `taxonomy_pin(taxonomy_id, version, view)` | Category joins under an explicit version; `view=live` resolves via lineage to current, `frozen` pins the stamped version (ADR-0011). |
| `aggregate(aggregation, grain)` | Implements the 8 closed aggregations; `distinct_sessions` ⇒ `COUNT(DISTINCT advisory_session_id)` (E.0-2). |
| `suppression_wrapper(support_threshold)` | Emits per-cell `n`; cells with `n < threshold` return `value = NULL, suppressed = true` **in SQL output shape**, so no downstream layer can "forget" suppression. |

**Compiled sketch** for `attendance_rate` × `programme`, period 2026-06 (illustrative shape, not production code):

```sql
SELECT p.programme_slug,
       COUNT(DISTINCT s.advisory_session_id) FILTER (WHERE af.attendance_status = 'attended') AS num,
       COUNT(DISTINCT s.advisory_session_id) FILTER (WHERE af.attendance_status <> 'unknown') AS den
FROM core.advisory_session s
JOIN core.attendance_fact af USING (advisory_session_id)
JOIN core.programme p ON p.programme_id = s.programme_id
WHERE s.scheduled_date >= $1 AND s.scheduled_date < $2      -- period_clause: end-exclusive, params only
GROUP BY p.programme_slug;
-- rate, Wilson CI, suppression, rounding: computed in code by shape_envelope, never by the model
```

### 5.4 Execution discipline

- Runs under the serving plane's **read-only role** on governed schemas (`core`, `findings`, `tax`, `transcript`, `evidence` views) — workers own writes (CORE-BRIEF §6).
- `statement_timeout = 5s` [REC — inside Lane-1's 8s p95; alternative: per-metric timeouts; revisit when a legitimate registered metric exceeds it, which is a modelling smell first].
- Row cap: the envelope carries full aggregates up to 10,000 rows to the renderer side-channel; the *model-facing digest* is capped at K = 50 rows × ≤8 columns (R10, doc 12 §7). `truncated` is set on the envelope, never silent.

---

## 6. Period and coverage enforcement

### 6.1 Period (I4 — the load-bearing invariant)

1. `resolve_period` (doc 12 §2.2) is **three-valued**: `RESOLVED` / `NO_PERIOD_MENTIONED` / `UNPARSEABLE`. `NO_PERIOD_MENTIONED` resolves to the metric's or capability's mandatory `default_period` — so "all-time" exists only as `all_time_declared`, printed on the answer face. `UNPARSEABLE` is a hard stop (`PERIOD_UNPARSEABLE`), never a degrade to all-time (F16/ISS-01 pin).
2. The compiler receives the period from the harness; **no model output can override it** — there is no field to write it into (§5.1).
3. All period predicates are end-exclusive (`>= start AND < end`), everywhere, including comparisons (CORE-BRIEF §13.4).
4. `period.decision_source ∈ resolved · default_period` is stamped in the envelope, and comparisons are declared in `period.comparisons` — the verifier's `declared_periods` set is built from exactly these (doc 12 §10).

### 6.2 Coverage (I8)

Every envelope computes `coverage = {used, matched, total, exclusions[]}` with `used ≤ matched ≤ total`:

- `total` — sessions in the resolved period scope (the denominator the user thinks in);
- `matched` — after filters;
- `used` — after enrichment/validation requirements (active transcript exists, extraction ran, finding validated, consultant resolved…);
- `exclusions[]` — one row per named reason with count and Arabic label, e.g. `{code: CONSULTANT_UNRESOLVED, count: 143, label_ar: "جلسات لم يُحسم مستشارها"}` (doc 08 §4). An empty enrichment (quality scoring not yet run) raises a `DATA_NOT_ENRICHED` warning state — **never returns zeros as if measured** [FACT MASTER_PROMPT App-A `impact_level_distribution` caveat].

### 6.3 Suppression and uncertainty

- `n < support_threshold` (default 30): percentage suppressed, count shown, cell flagged `suppressed: true` with Arabic marker «أقل من حد العرض (ن<30)» (E.0-4).
- Every proportion carries `ci95: [lo, hi]` (Wilson) computed in code (E.0-5).
- Rates for cross-period comparison are per-100-sessions with the denominator printed (E.0-1/E.0-3).

---

## 7. The typed result envelope

Adapted from MASTER_PROMPT §5.2 [DECISION — carried forward] with three additions for the new platform: `metric` block, `quality` block, and `system_state` (doc 07 §9). **Every tool and every curated capability returns exactly this envelope; nothing else may enter an answer (I11).**

```jsonc
{
  "ok": true,
  "data": [ /* rows/aggregates, already shaped, suppression applied, capped for the model digest */ ],
  "metric": {                          // present for metric_query; capability block for curated
    "id": "attendance_rate",
    "arabic_label": "نسبة الحضور",
    "unit": "pct",
    "denominator_ar": "الجلسات المجدولة المعروفة الحالة في الفترة"
  },
  "provenance": {
    "tool": "metric_query",
    "capability_id": null,             // set for curated_analysis
    "session_uids": ["01JGN0V9WQZ2Y4X8C6B3A1MKRT", "…"],   // ≤50 sample + total via row_count
    "sql_fingerprint": "sha256:…",
    "spec": { /* the validated spec that produced this */ },
    "row_count": 1234,
    "truncated": false,
    "registry_version": "sha256:…",
    "extraction_runs": [418, 422]      // findings-based only: lineage anchors (I13)
  },
  "coverage": { "used": 1258, "matched": 1401, "total": 1460,
                "exclusions": [ {"code":"STATUS_UNKNOWN","count":59,"label_ar":"حالة الحضور غير معروفة"},
                                {"code":"CONSULTANT_UNRESOLVED","count":143,"label_ar":"جلسات لم يُحسم مستشارها"} ] },
  "period":   { "start": "2026-06-01", "end": "2026-07-01", "label": "يونيو 2026",
                "all_time": false, "decision_source": "resolved",
                "comparisons": [ {"start":"2026-05-01","end":"2026-06-01","label":"مايو 2026","kind":"previous_month"} ] },
  "taxonomy": { "taxonomy_id": null, "version": null, "view": null, "stale": false },
                                       // MANDATORY non-null when data is category-classified (I13);
                                       // a renderer reporting a category count without it raises (I16)
  "quality":  { "support_threshold": 30,
                "suppressed_cells": [ {"dimension_value":"قناة الفروع","n":14} ],
                "ci_method": "wilson_95" },
  "system_state": "normal",            // normal · degraded_inference · degraded_retrieval · pinned_lane0 · partial
  "generated_at": "2026-08-02T10:41:07Z"
}
```

Envelope rules (testable, doc 15 owns fixtures):

| # | Rule |
|---|---|
| ENV-1 | `taxonomy` non-null whenever any `data` row is category-classified; violation raises, never renders (I13/I16). |
| ENV-2 | `coverage` present and internally consistent (`used ≤ matched ≤ total`); the verifier's `check_scope_declared` rejects **and raises** on absence — coverage is computed, not written (doc 12 §10). |
| ENV-3 | `period` present on every aggregate, end-exclusive, with `decision_source`. |
| ENV-4 | Numeric values travel as numbers; **formatting happens only in the rendering layer's `num()/pct()/count()`**, which emit into `allowed_literals` for the R6 gate (doc 12 §10). |
| ENV-5 | `ok:false` envelopes carry a closed `error.code`; partial results carry `system_state: partial` plus explicit incompleteness markers — no 200-on-failure (I16). |
| ENV-6 | Quotes never appear in `data` for metric envelopes; evidence envelopes (from `search_evidence` / `fetch_transcript_window`) carry quote objects `{session_uid, turn_index, speaker_role, quote_text, transcript_source_id, source_version}` (I7) and contribute **no numbers** (I9). |

---

## 8. Starter metric registry

Floor, not ceiling (R-P1b). ~27 metrics at launch, ported from MASTER_PROMPT Appendix A to the new schema names **plus** the internal-data metrics the legacy platform never had (ratings, evaluations, attendance, follow-up — the GREENFIELD §2.2 class-2 data), **plus the seven operational/review metrics of §8.4 added by the 2026-08-03 Owner Amendment**. All findings-based metrics pin `validated_only=true` and join through `session_extraction_state`; all rates declare denominators per §2. `DP` = default_period, `LCM` = last_closed_month, `ATD` = all_time_declared, `R3M` = rolling_3_months.

### 8.1 Operational and evaluation metrics (internal data — new in NIP)

| Metric id | Arabic label | Base (doc 08) | Grain | Agg | Denominator | DP | Key caveat |
|---|---|---|---|---|---|---|---|
| `sessions_count` | عدد الجلسات | `core.advisory_session` | session | count | — | LCM | Consultant-grouped views exclude unresolved-consultant sessions as a declared exclusion, never silently (magic-string lesson). |
| `session_duration_avg` / `_median` / `_p90` | متوسط/وسيط مدة الجلسة | `core.advisory_session` | session | avg/median/p90 | — | LCM | Actual (transcript-anchored) duration; scheduled duration is a separate field, never conflated (doc 05 dictionary rule). |
| `attendance_rate` | نسبة الحضور | `core.attendance_fact` | session | rate | scheduled, known status | LCM | §2.1 worked entry. |
| `cancellation_rate` | نسبة الإلغاء | `core.session_status_fact` | session | rate | scheduled | LCM | Dimension `attendance_status` splits canceller (مستفيد/مستشار/نظام) [ASSUME OD-07 vocabulary]. Period binds to `scheduled_date`. |
| `no_show_rate` | نسبة عدم الحضور | `core.attendance_fact` | session | rate | scheduled, known status | LCM | Distinct from cancellation — a no-show was never cancelled. CAP-D4 primary input. |
| `beneficiary_rating_avg` | متوسط تقييم المستفيد | `core.beneficiary_rating` | rating | avg | rated sessions | LCM | Coverage must state rated/attended AND source-match coverage — legacy bridge matched ~8% of report rows [FACT CORE-BRIEF §11]; the new reconciliation (doc 09) makes match-rate a first-class coverage figure. |
| `beneficiary_rating_distribution` | توزيع تقييم المستفيد | `core.beneficiary_rating` | rating | distribution | rated sessions | LCM | Bands 1–5; suppression per band. |
| `rating_response_rate` | نسبة الاستجابة للتقييم | `core.beneficiary_rating` ÷ `core.attendance_fact` | session | rate | attended sessions | LCM | Low response ⇒ selection-bias caveat rendered verbatim on every rating answer (doc 10). |
| `consultant_eval_avg` | متوسط تقييم المستشار للجلسة | `core.consultant_evaluation` | evaluation | avg | evaluated sessions | LCM | CAP-D2 input. |
| `consultant_eval_submission_rate` | نسبة تسليم تقييم المستشار | `core.consultant_evaluation` ÷ attended | session | rate | attended sessions | LCM | An operational compliance measure in its own right. |
| `eval_gap_avg` | متوسط فجوة التقييم (مستشار−مستفيد) | paired `core.consultant_evaluation` × `core.beneficiary_rating` | session | avg | sessions with both | LCM | Scales normalized before differencing [ASSUME OD-17: scale semantics pending OD-07 dictionary; working assumption min-max to 0–1 with the caveat printed; publication of consultant-named gaps blocked until scales confirmed]. |
| `followup_required_rate` | نسبة الجلسات المتطلبة متابعة | `core.followup_fact` | session | rate | sessions with followup field known | LCM | CAP-D7 input. |
| `followup_completion_rate` | نسبة إتمام إجراءات المتابعة | `core.outcome_fact` | session | rate | followup-required sessions with outcome data | LCM | Availability depends on OD-07 outcome feed; absence ⇒ `DATA_NOT_ENRICHED`, never zeros. |

### 8.2 Transcript- and findings-based metrics

| Metric id | Arabic label | Base | Grain | Agg | Denominator | DP | Key caveat |
|---|---|---|---|---|---|---|---|
| `clear_steps_rate` | نسبة الجلسات المنتهية بخطوات واضحة | `findings.finding` kind=`step_clarity` | session | rate | sessions with step-clarity extraction | LCM | CAP-B1 headline. `clarity ∈ clear·partial·none`; the null cohort (legacy 1,327) is an exclusion, never a bucket. |
| `violations_count` | عدد المخالفات | `findings.finding` kind=`violation` | finding | count | — | LCM | Verified + post-dedup only (`dedup_hash`, F1 pin); taxonomy stamp mandatory (I13). |
| `violations_session_rate` | الجلسات ذات مخالفة لكل 100 جلسة | same | session | rate | sessions with transcript + extraction | LCM | `COUNT(DISTINCT advisory_session_id)`; per-100-sessions (E.0-1/2). |
| `challenges_count` | عدد التحديات | `findings.finding` kind=`challenge` | finding | count | — | LCM | Counts by approved CHAL cluster (R-P2), never raw string. |
| `challenges_session_rate` | الجلسات ذات تحدٍّ لكل 100 جلسة | same | session | rate | sessions with extraction | LCM | CAP-C1 spec half. |
| `satisfaction_signals_count` | مؤشرات الرضا | `findings.finding` kind=`satisfaction_signal` | finding | count | — | LCM | All three polarities required (CAP-A3: إيجابي/سلبي/محايد). |
| `impact_level_distribution` | توزيع مستوى الأثر | session-level IMP classification | session | distribution | scored sessions | LCM | Empty enrichment ⇒ `DATA_NOT_ENRICHED` warning, never zeros. |
| `government_mentions_count` | ذكر الجهات الحكومية | `findings.finding` kind=`government_mention` | mention | count | — | R3M | Reported as *mentions extracted*, never as an assessment of the entity (CAP-C5 wording rule); child-table count only (legacy denormalized-counter lesson). |
| `government_friction_session_rate` | جلسات احتكاك حكومي لكل 100 جلسة | same, `is_friction=true` | session | rate | sessions with extraction | R3M | Extraction-only framing; entity resolution via `tax.entity_alias` (4,372 raw strings → canonical [FACT CORE-BRIEF §11]). |
| `pressure_signal_rate` | مؤشرات الضغط لكل 100 جلسة | `findings.finding` kind=`pressure_signal` | session | rate | sessions with extraction | LCM | Severe class imbalance (urgency 8,832 vs distress 544) ⇒ per-class suppression bites; never rank classes by raw count without the base (E.0-3). |
| `decision_points_count` | نقاط القرار | `findings.finding` kind=`decision_point` | finding | count | — | LCM | Hesitation = DEC-cluster recurrence (CAP-C8), not raw counts. |
| `repeated_question_cluster_count` | عناقيد الأسئلة المتكررة | `findings.cluster` (`QST-…`, approved) | cluster | count | — | R3M | CAP-B3/C6 shared engine; approved labels only (ADR-0012). |
| `consultant_talk_share` | نسبة كلام المستشار | `transcript.turn` via `active_transcript` | session | avg | sessions with active transcript | LCM | **Interval-merged** speaking time — the legacy `silence_pct` overlap bug is structurally fixed by doc 08's `total_speech_ms` interval-merge; cross-talk caveat still printed. |
| `consultant_question_rate` | معدل أسئلة المستشار | `transcript.turn` | session | avg | consultant turns | LCM | Deterministic interrogative detector (regex + particle list, doc 10); «يتكلم أكثر من يسأل» = ratio of this and talk-share. |
| `non_speech_time_pct` | نسبة الوقت غير الحواري | `transcript.turn` | session | avg | sessions with timing | LCM | Replaces legacy `silence_pct`; interval-merge math; timing 100% present [FACT CORE-BRIEF §11]. |
| `time_loss_minutes_avg` | متوسط الدقائق المفقودة | `findings.finding` kind=`time_loss_span` | session | avg | sessions with extraction | LCM | CAP-B2 spec companion; loss classes are curated judgement (§9). |

**Dimension applicability** is per-metric (`allowed_dimensions`); the compiler rejects e.g. `speaker_role` on `attendance_rate` with the standard hard-fail. The three-way «قطاع» note of §3 applies to every metric here: `service_category` and `government_entity` are registered and available, `business_sector` is registered and UNAVAILABLE.

### 8.3 VS-07 MVP serving set — the first five governed capabilities

[DECISION owner 2026-08-03 / Amendment §9.3 VS-07; SD-18] **The first usable product does NOT require all 26 committed capabilities.** SD-18 re-plans delivery as vertical slices with a mandatory stop-and-accept after each; the semantic layer's first governed, owner-testable serving set is exactly **five**, chosen for direct value and realistic data coverage. The remaining catalogue follows slice-by-slice under SD-21's consultation rule (docs 04 §11.5, 21, 24, 25) — building the full catalogue before a working product is the horizontal-delivery failure the amendment corrects.

| # | VS-07 capability (Amendment §9.3 wording) | Served through | Registry objects |
|---|---|---|---|
| 1 | عدد الجلسات وحالاتها حسب الفترة والبرنامج | `metric_query` | `sessions_count` × dimensions `session_status`, `programme`, `month` |
| 2 | تقييم المستفيد: المتوسط والتوزيع | `metric_query` | `beneficiary_rating_avg`, `beneficiary_rating_distribution` (+ the `rating_response_rate` selection-bias caveat rendered verbatim) |
| 3 | نسبة الخطوات الواضحة | CAP-B1 (spec half) | `clear_steps_rate` — the MR-10 label-validation gate applies before publication |
| 4 | حالات المخالفات المشتبه/المعتمدة وحالة المراجعة | approved-only series + separate queue figures | `violations_count` / `violations_session_rate` (scope `approved_only`, MR-17) beside `suspected_cases_open` + `review_backlog_age` (scope `suspected_queue_only`, §8.4) — never one merged series (T-19, doc 15) |
| 5 | أبرز التحديات أو الأسئلة المتكررة | CAP-C1 / CAP-B3 minimal read | `challenges_session_rate` + approved QST/CHAL clusters — gated on the **first owner-approved semantic artifact** (an R-P2 cluster run whose canonical labels the owner has named/approved through the review loop); until that approval exists the capability answers `DATA_NOT_ENRICHED`, never a read over unapproved clusters |

Rules for the set: the five ship with signed answer keys (doc 15 DS-03) before any L2 exposure; **L1 owner previews run earlier on fixture/staging data — EXP-09/EXP-10 and the production gates guard L2/L3 only, never L1 previews (SD-18)**; item 4's rendering obeys MR-17 end-to-end. This set is the acceptance surface of slice VS-07 (doc 25 owns the checklist; doc 15 §4 maps the suite tiers).

### 8.4 Operational and review metrics (Owner Amendment — names exact)

[DECISION owner 2026-08-03 / Amendment §12 doc 11 item] Seven operational/review metrics are registered with the same YAML discipline as §2 — same meta-schema, same CI gates, same envelope. They are **ops-class**: workload/health measures, not per-100-session analytics; their `unit`/`comparison_safe` marks keep them off analytical comparison axes, and MR-17 scope marks keep suspected volume out of published violation series (G-REG-8). Two registry consequences elsewhere: the existing `violations_count` / `violations_session_rate` entries gain `review_state_scope: approved_only`; and the amendment's shorthand `join_rate` is registered under its exact package name `source_join_rate`. Base tables are the doc 08 amendment entities ([REC] schema placement below follows CORE-BRIEF §6 — `ops` for run/audit/action/detector state, `findings` for review cases, `ingest` for reconciliation; doc 08 owns final DDL and may move a table, in which case the registry entry updates in the same change set).

| Metric id | Arabic label | Base (doc 08) | Grain | Agg | Denominator | DP | Key caveat |
|---|---|---|---|---|---|---|---|
| `nightly_run_success` | نجاح التشغيل الليلي في موعده | `ops.pipeline_run` | run | rate | scheduled nightly runs in window | rolling_3_months | Success = terminal `succeeded` before the completion deadline [ASSUME OD-28: start 02:00 Asia/Riyadh, configurable; deadline SLA part of OD-28]. A `partial` run is NOT success for this metric even though serving continues on unaffected feeds. Run states exact: `queued → running → partial → succeeded → failed → cancelled → superseded`. Feeds QT-15 (doc 15) and doc 19 SLOs. |
| `data_completeness` | اكتمال بيانات الفترة | `ops.data_quality_observation` + `core.advisory_session` terminal-state facts | session | share | expected sessions in period (DataHub-authoritative count, SRC-DATAHUB-SESSION) | last_closed_month | Month-close gate: ≥98% required before the infographic draft (MR-20) [ASSUME OD-28 — Amendment §13 working assumption]; always rendered with the per-sub-feed breakdown (SRC-DATAHUB family, doc 05) — never only a blended figure. |
| `source_join_rate` | نسبة ربط المصادر | `ingest.reconciliation_result` | session | rate | sessions eligible for the join direction | last_closed_month | **Two directions, registered separately and never merged:** provider→DataHub matched share and DataHub-session→transcript share (Amendment §3.3). Unmatched flow to the reconciliation queue (CAP-D9 / SCR-14). |
| `review_backlog_age` | عمر قائمة مراجعة الاشتباه | `findings.review_case` (open states) | case | distribution (p50/p90 + overdue share) | open review cases | last_closed_month | Workload metric at case grain — never a violation rate (MR-17). SLA thresholds 7d normal / 2d high-priority [ASSUME OD-30]; overdue share feeds QT-14 (doc 15) + doc 19 alerts. |
| `suspected_cases_open` | حالات الاشتباه المفتوحة | `findings.review_case` | case | count | — (point-in-time queue size) | last_closed_month | `review_state_scope: suspected_queue_only` — queue-size figure only; refused on published/leadership violation axes (MR-17, G-REG-8, T-19); renders with review-state vocabulary and the fixed «اشتباه مخالفة» disclaimer, never as «مخالفات». |
| `detector_acceptance_rate` | نسبة قبول قرارات الكاشف | `findings.review_event` × `ops.detector_release` stamps | decided case | rate | decided cases per category × detector version | last_closed_month | Always per category × release version — a blended acceptance figure is banned in release evaluation (doc 15 §6.6). A monitored drop triggers SD-22 review/rollback consideration, **never** an automatic model change. |
| `action_completion_rate` | نسبة إتمام الإجراءات | `ops.service_improvement_action` | action | rate | actions due in window | last_closed_month | Execution measure only; impact readings are MR-19 association-only (baseline → intervention → follow-up, no causal claims). Ownership/closure authority [ASSUME OD-34: service owner owns; leadership sees aggregates]. |

Queue metrics (`review_backlog_age`, `suspected_cases_open`) are additionally readable as an **as-of snapshot** (the queue as it stands now): the snapshot read is stamped `as_of`, carries no period comparison, and is the form CAP-OPS-02 (مركز تشغيل البيانات) and the morning digest use; historical series bind `period_column` to the case-opened date. All seven surface on the OPS screens (doc 04 §11.4), alert in doc 19, and are governed by I12 — an ops number on any screen that is not one of these registered ids fails the build.

---

## 9. Curated capability contract

### 9.1 What "curated" means here

A curated capability encodes **product judgement that must not be spec-compilable**: seeded pattern lexicons, calibrated thresholds, deliberate scope decisions, review-gated outputs (GREENFIELD §8.5). Curated ≠ unstructured: every curated capability is registered, declared, versioned, tested, and returns **the same envelope as §7** with one additional block.

### 9.2 Registration schema (`registry/capabilities/*.yaml`)

| Field | Semantics |
|---|---|
| `capability_id` | `CAP-A1…CAP-D10` (doc 04 owns the catalogue; ids are the cross-document currency). |
| `slug`, `arabic_name` | e.g. `session_time_loss`, «مواضع فقدان وقت الجلسة». |
| `classification` | `SPEC_ANSWERED` or `CURATED` — mandatory (gate G-REG-3, §10). |
| `lane` | Serving lane (Lane 0 for all committed capabilities). |
| `tau`, `delta` | Lane-0 matcher acceptance thresholds, calibration-artifact-backed (doc 12 §4). |
| `data_requirements` | Array of `{source, minimum_state}` — e.g. `{findings.finding kind=time_loss_span, validated}`, `{transcript.turn via active_transcript, present}`, `{tax VIOL, version ≥ 1}`. The executor checks these **before** running and emits `DATA_NOT_ENRICHED` with the missing item named, never a silently thinner answer (I16). |
| `period_behaviour` | `{default_period, comparison_windows, cadence}` — cadence (monthly/quarterly refresh for packs) and comparison granularity are independent axes [FACT MASTER_PROMPT App-B]. |
| `taxonomy_deps` | Taxonomies read + pinning behaviour (`live` for interactive answers, `frozen` inside packs — ADR-0011; doc 08 L7). |
| `methodology_ref` | Section of doc 10 that defines the method (e.g. `doc10 §E.5` for CAP-B2). No curated capability ships without a written method. |
| `judgement_params` | **Every product-judgement constant, named and versioned** — see §9.3. Hardcoded literals in capability code fail review; the legacy 40%-silence threshold buried in a handler (p75=34% at calibration) is the anti-pattern. |
| `args_schema` | Closed JSON Schema for `curated_analysis(capability_id, args)` — flat, `additionalProperties:false`, enums only (doc 12 §5.3). |
| `output_sections` | Declared envelope `data` shape (named sections, e.g. `ranked_loss_classes`, `example_quotes`). |
| `coverage_formula` | Same structure as metrics (§2). |
| `review_gate` | `none · pre_publication · per_finding` — CAP-B4 (violations with quotes) is `per_finding`: nothing user-visible before reviewer approval (precision-over-recall for accusations, E.0-8). |
| `tests` | Golden ids: ≥1 routing test, ≥1 numeric fixture test, ≥1 coverage test (doc 15). |
| `derivable_from_transcripts` | Feeds the Lane-3-offer classifier (doc 12 §3). |

### 9.3 Worked declaration — CAP-B2 `session_time_loss`

```yaml
capability_id: CAP-B2
slug: session_time_loss
arabic_name: مواضع فقدان وقت الجلسة
classification: CURATED
lane: 0
tau: 0.87        # from calibration artifact cal/2026-08/CAP-B2.json (placeholder until doc 15's run)
delta: 0.11
data_requirements:
  - {source: "findings.finding kind=time_loss_span", minimum_state: validated}
  - {source: "transcript.turn via active_transcript", minimum_state: present}
period_behaviour: {default_period: last_closed_month, comparison_windows: [previous_month], cadence: monthly}
taxonomy_deps: []
methodology_ref: doc10 §CAP-B2
judgement_params:
  loss_classes: [repeated_explanation, long_non_speech, admin_digression, tool_fumbling, unresolved_tangent]
  non_speech_span_threshold_pct: {value: 0.40, calibration: "p75 of corpus non_speech spans = 0.34 at 2026-08; recalibrate per corpus snapshot, EXP-06", authority: judgement}
  min_span_seconds: 45
args_schema:
  type: object
  additionalProperties: false
  required: [loss_class, top_n]
  properties:
    loss_class: {enum: [repeated_explanation, long_non_speech, admin_digression, tool_fumbling, unresolved_tangent, null]}
    top_n: {type: [integer, "null"], minimum: 1, maximum: 20}
output_sections: [ranked_loss_classes, per_class_minutes, example_quotes, coverage]
coverage_formula: {total: pred.sessions_in_period, matched: pred.with_active_transcript, used: pred.with_timeloss_extraction}
review_gate: none
tests: [GT-CAP-B2-ROUTE-01, GT-CAP-B2-NUM-01, GT-CAP-B2-COV-01]
derivable_from_transcripts: true
```

### 9.4 Envelope extension for curated capabilities

Curated envelopes add one block; everything else is identical to §7:

```jsonc
"curated": {
  "capability_id": "CAP-B2",
  "methodology_ref": "doc10 §CAP-B2",
  "judgement_params_sha": "sha256:…",     // params are versioned; a change reissues, never edits (ADR-0011 spirit)
  "review_gate": "none",
  "sections": ["ranked_loss_classes", "per_class_minutes", "example_quotes"]
}
```

Quotes inside curated envelopes obey ENV-6 and CORE-BRIEF §13.5 fully: `session_uid + turn_index + speaker_role + transcript_source + source_version + extraction_run` on every one (I7).

---

## 10. The SPEC_ANSWERED / CURATED structural boundary check

**Why:** the boundary between "compiled from the registry" and "encodes judgement" decays by convention. The legacy lesson is `check_period_coverage`: a boundary survives only when an unclassified item **fails the build** [FACT MASTER_PROMPT §5.5].

**Mechanism (CI, deterministic):**

```python
# planning pseudocode — structural boundary check (gate G-REG-3 + G-BOUND)
caps = discover_capabilities()                    # filesystem walk over registry/capabilities/ +
                                                  # executor modules; NO hand-maintained list (F5/F8)
for cap in caps:
    assert cap.classification in ("SPEC_ANSWERED", "CURATED"), f"{cap.id}: unclassified ⇒ build fails"

    if cap.classification == "SPEC_ANSWERED":
        # a spec capability is NOTHING BUT a stored, compilable spec + declared period behaviour
        assert cap.stored_spec is not None
        assert compiler.validate(cap.stored_spec).ok
        assert not has_custom_executor(cap), \
            f"{cap.id}: custom code in a SPEC capability ⇒ reclassify CURATED or move logic into the registry"
        assert cap.judgement_params in (None, {}), f"{cap.id}: judgement params imply CURATED"

    else:  # CURATED
        assert cap.methodology_ref, f"{cap.id}: curated without a written method"
        assert cap.judgement_params is not None   # may be empty only with an explicit waiver comment
        assert cap.tests and len(cap.tests) >= 3
        assert no_bare_numeric_literals(cap.executor_module), \
            f"{cap.id}: thresholds must be named judgement_params (40%-silence lesson)"
```

`has_custom_executor` is import-graph-based: a capability whose executor imports anything outside `{semantic_layer.compiler, envelope, rendering-agnostic shaping}` is structurally CURATED. Misclassification is therefore *detected*, not merely reviewed.

**Starter classification** (doc 04 owns; restated for the check's fixtures): SPEC_ANSWERED — CAP-B5, CAP-C3, CAP-C4, and the spec halves of CAP-B1/CAP-C1; CURATED — CAP-A1, CAP-A2, CAP-A3, CAP-B2, CAP-B3, CAP-B4, CAP-C2, CAP-C5, CAP-C6, CAP-C7, CAP-C8, CAP-D1…CAP-D10 (composites and judgement-bearing analyses) [FACT CORE-BRIEF §5 owner-type column].

---

## 11. Governance, change workflow, and revisit triggers

**Change workflow (single path — ADR-0012):** propose (YAML PR + methodology/doc-10 delta if curated) → CI gates G-REG-1…7 + golden suite → for Lane-0-routed items, τ/δ recalibration artifact committed → review (steward for taxonomy-touching, product owner for capability semantics) → merge → deploy stamps `ops.registry_snapshot` → `registry_version` change invalidates the spec cache by construction (R16).

**Retirement:** `status: retired` keeps the id resolvable for answer-artifact replay (doc 19) but removes it from planner enums and `list_capabilities`. Ids are never reused.

**Open decisions introduced here:**

- **[ASSUME OD-17]** Rating/evaluation scale semantics for `eval_gap_avg` (and CAP-D2): working assumption 1–5 beneficiary scale, consultant evaluation min-max normalized to 0–1; consultant-named gap publication blocked until OD-07's data dictionary confirms both scales. Owner: service owner + data steward. Impact: CAP-D2, two metrics, doc 10 methods.

**Revisit triggers:**

| Trigger | Action |
|---|---|
| Metric enum approaches 40 active ids | Family-split the tool schema per G-REG-4 (doc 12 §5.4) — decided *before* the 41st metric merges. |
| A SPEC capability accretes a second judgement param | It was CURATED all along — reclassify; the boundary check makes this visible at the first param. |
| Stewards need same-day registry changes | §1 alternative (serving-invisible `draft` DB status). |
| A curated capability's judgement_params churn monthly | Promote the parameter to a calibrated artifact with its own EXP (like τ/δ), or split the capability. |
| `statement_timeout` breaches on a legitimate metric | Examine grain/partition pruning first (doc 08 §16); only then raise per-metric timeout. |

**Cross-references:** tool schemas that carry these registries to the model — doc 12 §5; Lane-3 promotion of a recurring question *into* this registry — doc 13; golden fixtures for every gate here — doc 15; API projection of the envelope — doc 17.

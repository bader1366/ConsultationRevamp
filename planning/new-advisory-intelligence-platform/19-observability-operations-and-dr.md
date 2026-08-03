# 19 — Observability, Operations, and Disaster Recovery
**Platform:** Nwafeth Intelligence — منصة نوافث لذكاء الجلسات الاستشارية (Monsha'at Advisory Session Intelligence Platform)
**Status:** Draft for owner review · **Date:** 2026-08-02 · **Author:** Planning package (Fable 5)
**Depends on:** 07 (runtimes, roles, pools), 08 (schemas, `ops.*`), 09 (pipeline states, reconciliation), 12 (lanes, budgets, circuit breaker), 13 (Lane-3 job model), 14 (model registry), 16 (audit, retention, PII), 17 (`/healthz`, `/readyz`, `system_state`) · **Feeds:** 20 (cutover gates read these dashboards), 21 (ops work packages), 22 (ADR-0019/0020, OD-22), 23 (implementation order)
**Sources used:** GREENFIELD §17 (fully), §2.5, §15.4, §16.1; MASTER_PROMPT §6 R13/R14/R16, §5.3; arch/06 §5–§7 (backup/migration/ops reality); arch/08 ISS-05/06/12/14/17; CORE-BRIEF §8 (stack), §11 (baselines), §12 (defect pins)

---

## 0. Doctrine: observability that cannot lie

The legacy system did not merely lack observability — the observability it had **actively misled operators**. Each defect below is pinned to a structural countermeasure in this design; a reviewer should be able to point at any legacy lie and name the mechanism that makes it unreachable here.

| Legacy lie | Evidence | Structural countermeasure in NIP |
|---|---|---|
| Error rate read ~0% while users saw failures — orchestrator failed open, row logged `error=FALSE` | [FACT arch/08 ISS-05] | Degradation is a **first-class outcome enum** on every request record (`ok / degraded / failed / rejected_by_gate`), written by the layer that *decided* the outcome, not inferred by the caller. `nip_api_requests_total` carries `outcome` as a label; I16 forbids 200-on-failure; R14 forbids fail-open defaults |
| `GET /health` never touched the DB — Docker green through a total outage | [FACT arch/08 ISS-06] | Liveness and readiness are **split** (§6). `/readyz` executes real dependency probes (DB `SELECT 1` on both roles, queue heartbeat, MinIO, vault, IdP metadata) and returns 503 with a per-dependency map. Deploy gate: a readiness endpoint that does not touch the DB fails review |
| Dashboard showed AVG latency where the spec mandated P95 | [FACT arch/08 ISS-12] | All latency metrics are Prometheus **histograms**; dashboards render p50/p95/p99 only. There is no AVG latency panel anywhere; the SLOs in §5 are quantile-defined |
| Telemetry writes swallowed by `except: pass` — the log could not report its own gaps | [FACT arch/08 ISS-14] | **Meta-telemetry**: `nip_telemetry_write_failures_total` counter incremented on any failed audit/R13/metric write, with its own SEV-2 alert. A telemetry failure is an operational event, never a silent `pass` |
| No request IDs, no correlation router→handler→Groq; a user complaint was unreconstructable | [FACT arch/08 ISS-17; MASTER_PROMPT R13/ISS-15] | `request_id` (ULID) + W3C `traceparent` propagated end-to-end (§1.2); R13 reasoning log persisted per turn (§3.3); worked reconstruction procedure in §3.4 proves the property |
| Healthcheck endpoint reported **hardcoded** handler counts and rate-limiter numbers | [FACT arch/06 §7.11] | Every health/status number is computed from live state (registry query, TokenBucket introspection). CI structural check: no literal constants in health payload builders |
| Pipeline health answered by grepping logs; no timing instrumentation existed at all | [FACT arch/06 §6] | Every pipeline stage transition is a DB row (doc 09 §2) **and** a metric; wall-clock per stage is a histogram from day one |

Two standing principles govern everything below:

1. **[DECISION GREENFIELD §2.5 / ADR-0020]** Cost is measured and reported per model call, per job, per capability — but no alert, dashboard, or runbook may recommend sampling below full coverage as a remedy. Cost signals inform budgeting; rate limits and wall-clock are the engineered constraints.
2. **[DECISION I16]** Every failure and degradation is observable: no raw exception reaches a user, no 200-on-failure, partial results are marked incomplete. The observability plane is the *proof mechanism* for I16, not decoration.

---

## 1. Telemetry architecture

### 1.1 Stack

[REC — per CORE-BRIEF §8; alternatives and revisit triggers at end of subsection]

| Component | Choice | Runs where | Notes |
|---|---|---|---|
| Structured logs | **structlog** (JSON lines to stdout) | web + worker runtimes | One schema (§3.1); no free-text `print` allowed (lint rule) |
| Log aggregation | **Loki** + promtail/agent | ops stack | Labels: `runtime`, `env`, `level`, `event_class` only (low cardinality); everything else is JSON fields queried by LogQL |
| Metrics | **Prometheus** (pull) | ops stack | Web + worker expose `/metrics`; `postgres_exporter` for DB; one custom **ops-SQL exporter** job for DB-derived gauges (queue ages, backlog counts) |
| Traces | **OpenTelemetry SDK → Grafana Tempo** | web + worker | Tempo keeps the whole pane of glass in Grafana; sampling: 100% of Lane-1/Lane-3 and all errors, 10% of healthy Lane-0 [REC] |
| Dashboards/alerts | **Grafana + Alertmanager** | ops stack | Dashboards §4, alert rules §5; provisioned from the repo (dashboards-as-code), never hand-edited in the UI |
| Uptime probe | blackbox-exporter against `/readyz` + one synthetic Lane-0 golden question hourly | ops stack | The synthetic answers "up but answering wrongly?" — the case no internal probe catches [FACT arch/08 ISS-06 option 3] |

All components self-hosted inside the approved environment [ASSUME OD-01]; no SaaS telemetry (transcript-adjacent metadata never leaves the environment — consistent with doc 16 data-flow matrix). Alternatives considered: (a) ELK stack — rejected: heavier operational footprint for a small team, and Grafana-native alerting keeps one alert pipeline; (b) OpenTelemetry-collector-only pipeline with vendor backend — rejected: OD-01 environment cannot assume egress. **Revisit trigger:** if the authority mandates a central SIEM (OD-16), promtail/OTel exporters ship a copy there; the local stack remains the operator surface.

### 1.2 Request-id and trace propagation — end-to-end, including async hops

```text
Browser/API client
  │  X-Request-Id: 01J… (minted at ingress if absent; ULID)
  ▼
FastAPI middleware ── binds {request_id, trace_id, user_id, role} into structlog contextvars
  │                    starts OTel root span  api.request
  ▼
Lane router / executor spans: lane.route → capability.exec → tool.metric_query …
  │
  ├─ SQL: every statement carries a leading comment  /* rid=01J… cap=CAP-B1 */
  │       (pg_stat_activity + slow-query log join back to the request)
  ├─ Model call (Groq): span model.call; request_id in Groq client metadata field;
  │       log record per §3.2 — the span and the log share trace_id
  └─ Job enqueue: jobs.analysis_job.request_id + traceparent persisted on the job row
        ▼  (async boundary — trace continues as a LINKED trace, not the same trace)
     Worker claims job → new root span job.execute, link=stored traceparent,
        binds {job_id, request_id_origin} into every log line and every
        partition/session-task span beneath it
```

Rules [REC]:
- `request_id` is a ULID minted at ingress and **immutable** through the request, all tool calls, all model calls, the R13 row, the audit rows, and any job the request spawns (`request_id_origin` on the job). It is returned to the client in `X-Request-Id` and rendered in the UI error footer («رمز الطلب») so a user complaint arrives with the key.
- Async hops (queue, scheduler, rebase, pack builder) persist `traceparent` on the queue row and continue as **span links** — one Lane-3 job is one trace with per-partition child spans, linked back to the originating request trace.
- Scheduled work with no originating request mints its own `request_id` with prefix semantics recorded in the log (`origin=scheduler`).
- Every `ops.audit_event` row (doc 08 §12.1) carries `request_id` — audit and telemetry reconcile by construction, never by timestamp proximity.

### 1.3 Metric naming and cardinality rules

- Names: `nip_<plane>_<noun>_<unit>` (`nip_serve_api_request_seconds`, `nip_enrich_model_tokens_total`). Planes: `ingest`, `enrich`, `serve`, `jobs`, `gov` (governance), `plat` (platform).
- **Forbidden labels** (cardinality + PII): `consultant_id`, `session_uid`, `meeting_ulid`, `user_id`, free-text question, taxonomy category beyond top-level prefix. Per-entity analysis belongs in the DB (R13 rows), not in Prometheus.
- Allowed label vocabularies are closed enums from the registries: `capability_id` (CAP-xx, ≤30 values), `lane`, `model_id` (registry), `source_id` (SRC-xx), `reason_code` (§4 CORE-BRIEF enum), `outcome`, `job_class`, `queue`, `stage`, `gate`.
- Histogram buckets fixed at declaration and recorded in the metric registry appendix of doc 11's registry file [REC]: API seconds `(.1,.25,.5,1,2,3,5,8,13,21)` — aligned with the 3s/8s R16 budgets; model call seconds `(.25,.5,1,2,4,8,16,32,60,120)`; tokens `(256,1k,2k,4k,8k,16k,32k,64k,131k)`.

---

## 2. Metric catalogue — every GREENFIELD §17 bullet, by plane

Type key: C=counter, G=gauge, H=histogram. Source key: WEB=web runtime, WRK=worker runtime, SQLX=ops-SQL exporter (gauges computed by governed SQL against `ops`/`jobs`/`ingest` read-only), PGX=postgres_exporter. Thresholds are alert triggers (severity + routing in §5); all are [REC] initial values with the §11 monthly review as the tuning loop.

### 2.1 Ingestion plane (GREENFIELD §17: ingestion lag/errors · reconciliation/unmatched identities · provider distribution/quality)

| Metric | Type | Labels | Source | Threshold |
|---|---|---|---|---|
| `nip_ingest_lag_seconds` (now − newest successfully ingested `session_end` per source) | G | `source_id` | SQLX | > 93,600s (26h; SLA T+24h + 2h grace [ASSUME OD-06]) → SEV-2 |
| `nip_ingest_runs_total` | C | `source_id`, `outcome` (`ok/failed/partial`) | WRK | `failed` ≥ 2 consecutive per source → SEV-2 |
| `nip_ingest_items_total` | C | `source_id`, `disposition` (`ingested/skipped/dlq`) | WRK | `dlq` rate > 2% of items over 1h → SEV-2 |
| `nip_ingest_dlq_depth` / `nip_ingest_dlq_oldest_seconds` | G | `source_id` | SQLX | depth > 50 or oldest > 86,400s → SEV-2 |
| `nip_ingest_reconciliation_identity_failures_total` (listed ≠ skipped+fetched+dlq, doc 09 §3) | C | `source_id` | WRK | any ≥ 1 → SEV-1 (a counter identity break means silent loss is possible) |
| `nip_ingest_unmatched_queue_depth` / `_oldest_seconds` (Q1–Q5, doc 05 §10.1) | G | `queue` (`q1_provider…q5_conflict`) | SQLX | Q1/Q2 oldest > 7d, Q4 oldest > 3d → SEV-3 steward ticket |
| `nip_ingest_provider_active_share` (share of sessions whose active transcript is source X) | G | `provider`, `month` (last 3 only) | SQLX | Informational; drives doc 06 rebase planning |
| `nip_ingest_transcript_quality_tier_total` (tier A–D at ingest scoring, doc 06 §6) | C | `provider`, `tier` | WRK | tier-D share > 10% weekly → SEV-3 to provider owner |
| `nip_ingest_speaker_role_missing_ratio` (per ingest run) | G | `provider` | WRK | > 8% (legacy baseline 5.7% [FACT CORE-BRIEF §11]) → SEV-3 |

### 2.2 Enrichment plane (extraction backlog/failures · model calls/latency/tokens/cost/rate-limits/retries · structured-output validation failures · findings kept/dropped · quote-verification failures)

| Metric | Type | Labels | Source | Threshold |
|---|---|---|---|---|
| `nip_enrich_backlog_sessions` (sessions not yet fully enriched, by stage) | G | `stage` (doc 09 §2 states) | SQLX | > 300 for > 6h → SEV-2 (≈ a stalled worker at baseline arrival rates) |
| `nip_enrich_stage_seconds` | H | `stage` | WRK | p95 regression ×2 vs 7d baseline → SEV-3 |
| `nip_enrich_freshness_slo_ratio` (sessions enriched ≤ 24h after transcript arrival, daily) | G | — | SQLX | < 0.95 → SEV-2 (doc 09 §6.6 SLO) |
| `nip_model_calls_total` | C | `model_id`, `job_class` (`extract/lane1_plan/compose/lane3_map/verify_aux/route`), `outcome` (`ok/schema_invalid/timeout/http_4xx/http_5xx/rate_limited`) | WEB+WRK | see rate-limit + validation rows below |
| `nip_model_call_seconds` | H | `model_id`, `job_class` | WEB+WRK | p95 > 2× model-registry baseline for 30 min → SEV-2 (feeds circuit breaker §6.3) |
| `nip_model_tokens_total` | C | `model_id`, `job_class`, `direction` (`in/out`) | WEB+WRK | — (spend dashboard) |
| `nip_model_cost_usd_total` (computed from registry price × tokens) | C | `model_id`, `job_class` | WEB+WRK | Daily spend > 3× 14-day median → SEV-3 **informational — never a throttle trigger** [DECISION ADR-0020] |
| `nip_model_ratelimit_events_total` (429s) + `nip_model_retries_total` | C | `model_id`, `job_class` | WEB+WRK | 429 share > 5% of calls over 15 min → SEV-2 (RB-3) |
| `nip_model_ratelimit_headroom_ratio` (remaining/limit from Groq response headers, min over window) | G | `model_id`, `limit_kind` (`rpm/tpm/rpd/tpd`) | WRK | < 0.10 sustained 15 min → SEV-3 pre-warning |
| `nip_enrich_schema_validation_failures_total` (strict-JSON output rejected; R5) | C | `model_id`, `schema_id`, `attempt` (`first/repair`) | WRK | repair-failure rate > 1% daily → SEV-2 (model drift; doc 14 replay) |
| `nip_enrich_findings_total` | C | `capability_family`, `disposition` (`kept/dropped_quote_gate/dropped_schema/dropped_validation/dropped_dedup`) | WRK | `dropped_quote_gate` share > 3% daily → SEV-2 (extraction fabricating quotes — model or prompt regression) |
| `nip_enrich_quote_verification_failures_total` (R7 batch verifier) | C | `context` (`extract/rebase/pack/serve`) | WEB+WRK | any in `pack` context → SEV-1 (publication gate breached) |
| `nip_enrich_extraction_run_progress_ratio` | G | `extraction_run_id` (active runs only, ≤5) | SQLX | no progress 60 min → SEV-2 (RB-2 pattern) |
| `nip_enrich_embedding_queue_depth` / `nip_enrich_embedding_seconds` | G/H | `model_id` (embedding registry id) | WRK | queue > 50k units or p95 > 1s/unit → SEV-3 |

### 2.3 Serving plane (tool selection/lane distribution · period/entity resolution outcomes · coverage/exclusions · verifier failures · API latency/errors)

| Metric | Type | Labels | Source | Threshold |
|---|---|---|---|---|
| `nip_serve_api_request_seconds` | H | `route_group`, `method`, `status_class` | WEB | p95 > 3s Lane-0 routes / > 8s Lane-1 for 10 min → SEV-2; p99 tracked on dashboard (never AVG — ISS-12 pin) |
| `nip_serve_api_requests_total` | C | `route_group`, `status_class`, `outcome` (`ok/degraded/failed/rejected`) | WEB | 5xx > 1% over 10 min → SEV-1; `degraded` > 5% over 30 min → SEV-2 (the ISS-05 counter-metric: degraded is counted, loudly) |
| `nip_serve_lane_turns_total` | C | `lane` (`0/1/2/3_offer`), `capability_id` | WEB | Lane-0 share < 60% weekly (routing decay) → SEV-3; sudden Lane-2 spike > 20% daily → SEV-3 |
| `nip_serve_lane0_abstain_total` + `nip_serve_lane0_margin` | C/H | `capability_id` | WEB | abstain rate ×2 vs calibration baseline → SEV-3 (recalibrate τ/δ, doc 12 §3) |
| `nip_serve_tool_calls_total` | C | `tool` (11-tool enum), `outcome` | WEB | `validation_failed` > 2% daily → SEV-3 |
| `nip_serve_period_resolution_total` | C | `outcome` (`explicit/default_declared/unparseable/empty`) | WEB | `unparseable` > 10% daily → SEV-3 (resolver gap — e.g. Hijri, OD-18); **any** silent-fallback occurrence is impossible by construction (I4) — a unit test, not an alert |
| `nip_serve_entity_resolution_total` | C | `entity_kind` (`consultant/gov_entity/programme/service`), `outcome` (`resolved/ambiguous/not_found`) | WEB | `ambiguous` > 15% for `gov_entity` weekly → SEV-3 (alias registry gap, feeds tax governance) |
| `nip_serve_verifier_rejections_total` (R6 numeric / R7 quote / schema / scope) | C | `gate` (`r6/r7/schema/scope`), `lane` | WEB | > 1% of composed answers daily → SEV-2; single R7 rejection logged ERROR with `meeting_ulid` claimed (MASTER_PROMPT R7) |
| `nip_serve_coverage_declared_ratio` (answers carrying scope+coverage block; must be 1.0 — I8) | G | — | SQLX | < 1.0 → SEV-1 (structural regression) |
| `nip_serve_answers_with_exclusions_total` | C | `exclusion_reason` (closed enum: `low_n/missing_dimension/unenriched/partition_incomplete`) | WEB | informational — feeds data-quality dashboard |
| `nip_serve_budget_exhausted_total` (R16: steps/calls/deadline) | C | `budget_kind`, `lane` | WEB | > 3% of Lane-1 turns daily → SEV-3 |
| `nip_serve_spec_cache_hits_ratio` | G | — | WEB | informational; drop to ~0 after registry bump is *expected* (cache keyed on `registry_version`) |
| `nip_serve_clarification_total` | C | `capability_id` | WEB | — (UX review input) |
| `nip_serve_reason_codes_total` (honest-boundary outcomes) | C | `reason_code` (§4 CORE-BRIEF closed enum) | WEB | `VERIFIER_REJECTED` or `VALIDATION_FAILED` trending ×3 WoW → SEV-2 |

### 2.4 Jobs plane (deep-job progress by partition · stall · lifecycle)

| Metric | Type | Labels | Source | Threshold |
|---|---|---|---|---|
| `nip_jobs_active` | G | `status` (doc 13 §7.1 enum: `planning/mapping/verifying/reducing/stalled…`) | SQLX | `stalled` ≥ 1 → SEV-2 (RB-1) |
| `nip_jobs_partition_progress_ratio` (done tasks / tasks, per active job) | G | `job_id` (active only), `partition` (month) | SQLX | no delta 30 min while `mapping` → stall detector flips job to `stalled` (worker-side), which alerts — the *metric* is the human view, the *detector* is code |
| `nip_jobs_session_tasks_total` | C | `outcome` (`done/failed/skipped_no_transcript`) | WRK | `failed` > 2% of a job's tasks → job will close `incomplete`; SEV-3 notice to requester |
| `nip_jobs_wallclock_seconds` (accepted→terminal) | H | `terminal` (`complete/incomplete/cancelled/failed`) | WRK | p95 > 4h at baseline corpus → SEV-3 capacity review (§8) |
| `nip_jobs_queue_wait_seconds` (accepted→claimed) | H | — | WRK | p95 > 15 min → SEV-3 (pool starving; §8.5) |
| `nip_jobs_cache_hits_total` / `nip_jobs_promotion_candidates` | C/G | — | WRK/SQLX | promotion candidates ≥ 3 for same fingerprint → product review (doc 13 §9) |

### 2.5 Governance plane (review queue age · taxonomy proposal age · packs)

| Metric | Type | Labels | Source | Threshold |
|---|---|---|---|---|
| `nip_gov_review_queue_depth` / `_oldest_seconds` | G | `review_type` (doc 08 `ops.review_item` enum: `answer_key/violation/taxonomy/cluster/alias/publication`) | SQLX | `violation` oldest > 72h → SEV-2 (B4 SLA); others > 14d → SEV-3 |
| `nip_gov_taxonomy_proposal_oldest_seconds` / `_depth` | G | `taxonomy` (`VIOL/CHAL/SAT/DEC/IMP/ENT/QST`) | SQLX | oldest > 30d → SEV-3 steward+owner (stale taxonomy stalls reclassification, RB-6) |
| `nip_gov_reclassification_backlog_sessions` (sessions with findings pinned to superseded taxonomy version) | G | `taxonomy` | SQLX | > 20% of corpus for > 14d → SEV-2 (RB-6) |
| `nip_gov_pack_pipeline_state` (monthly/quarterly pack build progress) | G | `pack_kind`, `stage` | SQLX | monthly pack not `published` by day 5 of following month → SEV-2 to product owner (OD-10) |
| `nip_gov_unsigned_capability_count` (subjective capabilities serving without approved answer key — must be 0 in production; GREENFIELD §18.3) | G | — | SQLX | > 0 → SEV-1 |

### 2.6 Platform plane (DB health/pool pressure · export and sensitive-evidence access · telemetry health)

| Metric | Type | Labels | Source | Threshold |
|---|---|---|---|---|
| `pg_up`, replication/WAL metrics, `pg_stat_activity` counts | G | standard | PGX | `pg_up=0` → SEV-1; WAL archive failure → SEV-1 (RPO breach risk, §10) |
| `nip_plat_db_pool_in_use` / `_waiters` | G | `role` (`nip_web/nip_worker`), `runtime` | WEB+WRK | waiters > 0 for 5 min → SEV-2 (RB-8) |
| `nip_plat_queue_depth` / `_oldest_seconds` (Procrastinate tables) | G | `queue` (`ingest/enrich/jobs/packs/maintenance`) | SQLX | oldest > 2× queue SLA → SEV-2 |
| `nip_plat_worker_heartbeat_age_seconds` | G | `pool` | SQLX | > 120s → SEV-1 (worker plane down) |
| `nip_plat_export_events_total` | C | `export_kind` (`xlsx/pack_pdf/api`), `contains_quotes` (bool) | WEB | quote-bearing exports reviewed weekly (dashboard, doc 16 §5) |
| `nip_plat_sensitive_evidence_access_total` (transcript-window fetches; break-glass PII reads) | C | `access_class` (`transcript_window/breakglass_pii`) | WEB | `breakglass_pii` **any** → same-day steward review (SEV-2 ticket auto-filed) [ASSUME OD-09] |
| `nip_telemetry_write_failures_total` (meta-metric; ISS-14 pin) | C | `sink` (`audit/r13/metrics/loki`) | WEB+WRK | any > 0 over 15 min → SEV-2 |
| `nip_plat_backup_last_success_age_seconds` / `nip_plat_restore_drill_age_days` | G | `backup_kind` (`pgbackrest_full/diff/wal`, `minio_sync`) | SQLX | full > 8d, diff > 26h, WAL > 15 min → SEV-1; drill age > 100d → SEV-3 (§9.4) |
| `nip_plat_model_deprecation_days_remaining` (from `ops.model_registry.deprecation_date`) | G | `model_id` | SQLX | < 45d with model still in an active role → SEV-2 ops task (I17; the llama-3.3-70b 2026-08-16 shutdown is the live example [FACT groq-docs 2026-08-02]) |

### 2.7 Coverage cross-check — GREENFIELD §17 bullets → metrics

| §17 bullet | Covered by |
|---|---|
| source ingestion lag and errors | 2.1 rows 1–4 |
| source reconciliation and unmatched identities | 2.1 rows 5–6 |
| transcript-provider distribution and quality | 2.1 rows 7–9 |
| extraction backlog and failure rate | 2.2 rows 1–3, 12 |
| model calls, latency, tokens, cost, rate limits, retries | 2.2 rows 4–10 |
| structured-output validation failures | 2.2 row 11 |
| tool selection and lane distribution | 2.3 rows 3–5 |
| period/entity resolution outcomes | 2.3 rows 6–7 |
| deep-job progress by partition | 2.4 rows 1–2 |
| findings kept/dropped and reasons | 2.2 row 12 |
| quote-verification failures | 2.2 row 13; 2.3 row 8 |
| coverage and exclusions | 2.3 rows 9–10 |
| answer verifier failures | 2.3 row 8 |
| review queue age | 2.5 row 1 |
| taxonomy proposal age | 2.5 rows 2–3 |
| API p50/p95/p99 latency and errors | 2.3 rows 1–2 |
| database health and pool pressure | 2.6 rows 1–2 |
| export and sensitive-evidence access | 2.6 rows 5–6 |

---

## 3. Structured logging and full reconstructability

### 3.1 Log event schema

Every log line is one JSON object. Mandatory envelope fields (structlog processors add them; a lint test asserts no logger call can omit them):

```json
{
  "ts": "2026-08-02T14:03:22.113Z",
  "level": "info",
  "event": "tool.exec.ok",                 // dot-namespaced closed catalogue, registered in repo
  "runtime": "web",                        // web | worker
  "env": "prod",
  "request_id": "01J4XW…",
  "trace_id": "4bf92f35…", "span_id": "00f067aa…",
  "actor": {"user_id": "u_1042", "role": "analyst"},   // staff identity only; never beneficiary identity
  "ctx": { /* event-class-specific fields, schemas below */ }
}
```

Event classes with fixed `ctx` schemas (registered in `ops.log_event_catalogue` doc-side; enforced by a typed logging facade, not by convention): `api.request`, `lane.route`, `tool.exec`, `model.call`, `verifier.verdict`, `job.transition`, `partition.transition`, `ingest.run`, `stage.transition`, `rebase.step`, `pack.step`, `audit.emit`, `telemetry.failure`.

### 3.2 Model-call log record — mandatory on every model call, no exceptions

[DECISION CORE-BRIEF §8: "every model call logged with tokens/cost/latency"]

```json
"ctx": {
  "model_id": "openai/gpt-oss-120b",
  "provider": "groq",
  "job_class": "lane3_map",
  "prompt_id": "extract_findings_v3", "prompt_sha": "sha256:ab12…",   // from ops.prompt_registry
  "schema_id": "finding_v2", "structured_mode": "json_schema_strict",
  "tokens_in": 3812, "tokens_out": 942,
  "cost_usd": 0.00341,                       // registry price × tokens, computed at log time
  "latency_ms": 1840,
  "attempt": 1, "retry_cause": null,          // rate_limited | timeout | http_5xx
  "finish_reason": "stop",
  "outcome": "ok",                            // ok | schema_invalid | timeout | rate_limited | http_error
  "rate_limit_headroom": {"rpm": 0.44, "tpm": 0.31},
  "job_id": "job_01J4…", "partition": "2026-05", "session_task_id": 88123,  // when applicable
  "capability_id": "CAP-C1"                   // when attributable
}
```

**Never present**: prompt text, completion text, transcript text, quote text. Payloads live in governed storage (`findings.*`, `serve.*`, MinIO artifacts) referenced by ids; the log is the index, not the vault. `prompt_sha` + `ops.prompt_registry` reproduce the exact prompt; `extraction_run_id`/`turn_id` reproduce the exact input.

### 3.3 R13 reasoning log — the per-turn record

[DECISION MASTER_PROMPT §6 R13, carried forward] Persisted as rows (`serve.turn` + `serve.tool_call` + `serve.verifier_result`, doc 08 §8), not as log lines; log lines carry the ids. Full field list the serving plane must persist per turn:

| Field group | Fields |
|---|---|
| Question | `raw_question` (serve schema only — restricted, never Loki), `normalised_question`, `question_sha256`, `language` |
| Resolution | `resolved_period {start,end,kind,source}` (I4), `resolved_entities[]` (kind, id, method, score), `turn_resolution_outcome` (follow-up context decision: `{resolved_period, resolved_entities, capability_id, result_scope_fingerprint, result_ulids_ref, page}` [FACT MASTER_PROMPT §5.0]) |
| Routing | `lane`, `lane0_score`, `lane0_margin`, `capability_id`, `route_reason` |
| Execution | per tool call: `tool`, `validated_spec` (JSON), `sql_fingerprint` (sha of compiled SQL — never model-authored, I2), `row_count`, `duration_ms`, `allowed_literals_ref` (R6 emit side) |
| Gates | verifier verdicts per gate (`r6/r7/schema/scope`) with machine-readable failure detail |
| Budget | `steps_used`, `model_calls_used`, `tokens_used`, `deadline_remaining_ms` |
| Output | `outcome`, `reason_code` (if boundary), `answer_artifact_id`, `coverage_block` (I8 copy), `prompt_shas[]`, `plan_label` |
| Environment | `registry_version`, `taxonomy_versions{}`, `model_ids{}`, `system_state` at serve time |

### 3.4 Reconstructing a request without PII — the worked procedure

Scenario: an analyst reports «الرقم غير صحيح في سؤال الأمس عن نسبة الخطوات الواضحة» with the UI footer code `01J4XW9K…`.

1. `serve.turn WHERE request_id = '01J4XW9K…'` → the full R13 row: normalised question, resolved period `[2026-07-01, 2026-08-01)`, lane 0, CAP-B1, spec, SQL fingerprint, verifier verdicts, answer artifact id. *(No transcript text, no beneficiary identity touched.)*
2. Grafana → Tempo trace `trace_id` from the same row → per-stage latency; Loki `{env="prod"} | json | request_id="01J4XW9K…"` → every log event including the compiled-spec validation.
3. The number itself: `serve.answer_artifact` → `metric_results[]` with `allowed_literals` and the `sql_fingerprint`; re-execute the *registered* spec via the replay harness (doc 15) against the same period → byte-identical number or a real data drift (then `ingest.reconciliation_result` explains what changed and when).
4. If a quote is disputed: artifact carries `(meeting_ulid, turn_index, source, source_version)`; the reviewer role fetches the turn via the governed transcript window endpoint — that access itself lands in `nip_plat_sensitive_evidence_access_total` and `ops.audit_event`.

Acceptance test (doc 15 T-batch): for any golden-suite request, steps 1–3 must complete using only `request_id` — this is the operational definition of "reconstructable from logs without exposing unnecessary PII" [FACT GREENFIELD §17].

### 3.5 Telemetry data retention

| Sink | Retention | Rationale |
|---|---|---|
| Prometheus | 30d raw, 13 months downsampled (5m) | capacity trends across the 2.7× seasonal swing |
| Loki | 90d [ASSUME OD-08] | operational forensics window |
| Tempo traces | 14d | debugging window; R13 rows are the durable record |
| `serve.*` R13 rows | ≥ 18 months [ASSUME OD-08] | audit + eval replay |
| `ops.audit_event` | ≥ 18 months, append-only, tamper-evident digests (doc 16 §5) | compliance |

---

## 4. Dashboards

Five provisioned dashboards (JSON in repo under `ops/grafana/`; UI edits forbidden — drift-checked in CI). Panel lists are the build spec:

**D1 — Ops overview (on-call home).** `system_state` banner (normal/degraded_inference/pinned_lane0 — doc 17 §meta) · API RED per route_group (rate, error %, p50/p95/p99 — no AVG anywhere) · outcome split incl. `degraded` share (ISS-05 pin) · lane distribution now/7d · circuit-breaker state timeline · DB pool in-use/waiters per role · queue depths · worker heartbeats · active SEV alerts.

**D2 — Pipeline (steward + ops).** Ingestion lag per source vs SLA line · runs and DLQ (depth, oldest, top error classes) · reconciliation identity status per run (green/red) · Q1–Q5 queue depth+age · enrichment backlog by stage with 24h-SLO attainment gauge · stage p95 durations · extraction-run progress bars · rebase operation progress (when active, doc 06 §4) · provider share + quality-tier trend.

**D3 — Model spend & inference (eng lead + owner report).** Calls/tokens/cost by model × job_class (day/week/month) · cost per session enriched; cost per Lane-3 job (job table) · latency p95 per model vs registry baseline · 429 rate + headroom minima · schema-validation failure rate per model (drift canary) · Batch API job states · deprecation countdown per registry model with role assignments (I17) · spend is **reported, never gated** [DECISION ADR-0020] — the panel carries this caption verbatim so nobody "optimizes" it into a sampling decision.

**D4 — Review queues & governance (steward + product owner).** Review queue depth+oldest by type with SLA lines · reviewer throughput/week · taxonomy proposal age by taxonomy · reclassification backlog % per taxonomy version · pack pipeline state with day-5 deadline marker · unsigned-capability count (must be 0) · append-only review-event stream (last 50).

**D5 — Data quality (CAP-D9 operator mirror).** Join-rate KPIs per monthly cohort vs targets (doc 05 §10.2) · `resolution_status` distribution (the anti-`'PENDING'` view) · speaker_role coverage · attribute-conflict (Q5) trend · coverage/exclusion reasons emitted in answers · findings kept/dropped reasons · DQ observations feed (doc 05 §10.3) · quote-verification failure trend by context.

The user-facing CAP-D9 capability reads the same governed queries as D5 [DECISION doc 05 §10.3 — one source of truth; the operator dashboard is not a second computation].

---

## 5. Alerting — severities, rules, routing

### 5.1 Severity ladder

| Sev | Meaning | Response | Examples |
|---|---|---|---|
| SEV-1 | Users get wrong/no service, or a correctness/compliance invariant is breached, or recovery capability is at risk | Page on-call now; incident channel; product owner informed same day | `pg_up=0`; worker plane heartbeat lost; reconciliation identity failure; quote-verification failure in a pack; unsigned capability serving; WAL archiving broken; coverage-declaration ratio < 1.0 |
| SEV-2 | Degraded service or an SLO breach with a bounded blast radius; correctness gates holding | Business-hours response < 4h | 429 storm; stalled Lane-3 job; freshness SLO miss; degraded-answer share high; pool waiters; violation review > 72h; break-glass PII access review |
| SEV-3 | Trend/`hygiene`; no user impact yet | Ticket, weekly ops review | abstain-rate drift; tier-D transcript share; taxonomy proposal age; cost anomaly; drill overdue |

### 5.2 Routing

[REC] Grafana Alertmanager routes by label `team`: `ops` (SEV-1/2 infra+serving) → on-call channel + email; `steward` (queues, DQ, PII review) → steward group; `ml` (model drift, schema failures, deprecations) → eng lead; `owner` (pack deadline, unsigned capability, spend report) → product owner. Every SEV-1/2 alert names its runbook id (`runbook: RB-3`) in the annotation — an alert without a runbook link fails the dashboards-as-code CI check.

**[ASSUME OD-22 — new]** On-call staffing and notification channels inside the government network (SMS/Teams-equivalent/email; whether a 24/7 rotation exists at pilot). Safe working assumption: **business-hours on-call by a 2-person ops rotation, notifications via approved email + the authority's messaging platform; out-of-hours SEV-1 auto-degrades the platform to safe states (circuit to Lane 0, queue pause) rather than assuming a human is awake.** Impact: §6 degraded-state design is sized for unattended nights; doc 21 staffing. Owner: platform owner.

### 5.3 Alert rule sketches (PromQL-shaped, illustrative not production code)

```text
ALERT NipApiErrorBudget    (SEV-1, team=ops, runbook=RB-8)
  expr: sum(rate(nip_serve_api_requests_total{status_class="5xx"}[10m]))
        / sum(rate(nip_serve_api_requests_total[10m])) > 0.01

ALERT NipDegradedShare     (SEV-2, team=ops, runbook=RB-4)
  expr: sum(rate(nip_serve_api_requests_total{outcome="degraded"}[30m]))
        / sum(rate(nip_serve_api_requests_total[30m])) > 0.05

ALERT NipRateLimitStorm    (SEV-2, team=ops, runbook=RB-3)
  expr: sum(rate(nip_model_ratelimit_events_total[15m]))
        / sum(rate(nip_model_calls_total[15m])) > 0.05

ALERT NipJobStalled        (SEV-2, team=ops, runbook=RB-1)
  expr: nip_jobs_active{status="stalled"} >= 1

ALERT NipWalArchiveBroken  (SEV-1, team=ops, runbook=RB-10/§10)
  expr: nip_plat_backup_last_success_age_seconds{backup_kind="pgbackrest_wal"} > 900
```

---

## 6. Health model — liveness, readiness, degraded states

### 6.1 Probes (aligned with doc 17 §endpoints)

| Probe | Path | Semantics | Checks |
|---|---|---|---|
| Liveness | `GET /healthz` | "process should not be restarted" — static, no dependencies, no DB | event loop responsive; returns build SHA + uptime only. Deliberately dependency-free so a DB outage does not restart-loop the web tier [FACT arch/08 ISS-06 option-2 trade-off] |
| Readiness (web) | `GET /readyz` | "may receive traffic" | `SELECT 1` as `nip_web` **and** a 50ms read on `serve.conversation`; MinIO HEAD; vault token TTL > 10 min; IdP metadata cached-fresh; migration head == code head (Alembic) |
| Readiness (worker) | heartbeat row per pool, 30s | "pool is draining queues" | queue claim round-trip as `nip_worker`; embedding service ping; Groq reachability is **not** a readiness dependency (its failure degrades, not kills — §6.3) |

Container/orchestrator wiring: liveness → restart; readiness → remove from load balancing. The legacy conflation (one `/health`, DB-blind, wired to restart) is structurally impossible: CI check asserts `/readyz` handler references the DB session factory.

### 6.2 Degraded states are typed, surfaced, and end-to-end

`system_state` (doc 17 §2: `normal / degraded_inference / degraded_retrieval / pinned_lane0 / partial`) is computed server-side from live signals, returned on `GET /api/v1/meta/system-state`, stamped into every R13 row, and rendered in the UI banner («وضع النظام: خدمة محدودة — الإجابات المحفوظة فقط»). Rules:

- `/readyz` stays **200 in degraded states** — traffic is still served honestly; the state map inside the body names what is impaired (I16: degradation observable, service not pretending).
- Any answer produced under a degraded state carries the state in its coverage block; Lane availability changes are honest boundaries (`BUDGET_EXHAUSTED`/`DEEP_JOB_STALLED` reason codes), never silent lane substitution (I5).

### 6.3 Circuit breaker (provider degradation → pin Lane 0)

[DECISION MASTER_PROMPT R16 carried forward] Deterministic breaker in the serving plane, keyed per provider:

```text
CLOSED  → OPEN   when over 5 min: 429_share > 20%  OR  timeout_share > 10%
                  OR  5 consecutive model-call failures
OPEN             Lane 0 continues (no model on path); Lane 1 turns answer from
                 spec-cache when valid, else Lane 2 honest boundary
                 (reason_code=BUDGET_EXHAUSTED, detail=provider_degraded);
                 Lane-3 jobs auto-pause between session tasks (state preserved)
OPEN → HALF_OPEN after 60s: 3 canary calls (registry-designated cheap model)
HALF_OPEN → CLOSED on 3/3 ok; → OPEN on any failure (backoff ×2, cap 15 min)
```

State transitions are audit events and a metric (`nip_serve_circuit_state`); the expired-Groq-key production outage in the legacy system is the pinned motivation [FACT MASTER_PROMPT R16].

---

## 7. Job operations runbooks

Format per runbook: **Trigger → Diagnose → Remediate → Verify → Escalate/Never**. All SQL is operator-read-only against governed views; remediation actions go through `opsctl` commands (worker-plane CLI, every invocation audited), never ad-hoc UPDATEs. [REC: `opsctl` is the single ops entrypoint — the legacy 66-script heap with no lifecycle convention is the anti-pattern being killed, arch/06 §7.8.]

### RB-1 — Stalled Lane-3 job

- **Trigger:** `NipJobStalled`; user-visible state `stalled`, reason code `DEEP_JOB_STALLED` if the requester asks.
- **Diagnose:** `opsctl job show <job_id>` → last task transition, per-partition progress, last model-call outcome for the job (`model.call` events filtered by `job_id` in Loki). Three common causes: (a) worker pool died (heartbeat alert will co-fire) — restart pool; (b) provider degradation (breaker OPEN) — job auto-paused, see RB-4; (c) poison task: same `session_task_id` cycling `running→failed`.
- **Remediate:** (a/b) restart/wait — resume is automatic from `jobs.session_task` state (doc 13 §7.2; the 592KB-JSON-checkpoint failure mode is structurally gone). (c) `opsctl job skip-task <task_id> --reason <text>` → task `failed(operator_skip)`, partition will close `incomplete` honestly.
- **Verify:** progress ratio rising; job reaches terminal state; if `incomplete`, the answer artifact declares the gap (I8/I16).
- **Never:** never mark a failed task `done`; never let a stall silently expire the user's request — the job either completes, closes incomplete, or is cancelled with the requester notified (doc 18 notification).

### RB-2 — Failed partition / job closes `incomplete`

- **Trigger:** `nip_jobs_session_tasks_total{outcome="failed"}` breach, or job terminal `incomplete`.
- **Diagnose:** `opsctl job failures <job_id>` groups failed tasks by `error_class` (`schema_invalid` after repair / timeout / rate_limited / transcript_missing). One class dominating = systemic (model drift → doc 14 replay; missing transcripts → check rebase/retention state).
- **Remediate:** systemic cause fixed → `opsctl job retry-failed <job_id>` (re-runs only failed tasks — idempotent, task-keyed). Data-cause (`skipped_no_transcript`) is not an error: the coverage block already declares it.
- **Verify:** re-reduce runs automatically after retries; aggregate recomputed in code (I3); artifact superseded-by chain intact (doc 13 §7.1).
- **Never:** never hand-patch `jobs.job_aggregate`; never re-run reduce over a partially retried map without the verifier accounting close.

### RB-3 — Rate-limit storm (429s)

- **Trigger:** `NipRateLimitStorm`; headroom gauges < 0.10.
- **Diagnose:** D3 dashboard → which `job_class` is consuming the window. Typical: a corpus re-extraction launched during business hours competing with serving-plane calls. The legacy failure mode — 429s swallowed to `None`, indistinguishable from clean output [FACT arch/06 §6] — is unreachable: every 429 is counted and retried with cause recorded.
- **Remediate:** (1) `opsctl pool throttle enrich --concurrency <n>` (bounded pools are runtime-tunable); serving-plane calls have priority by design (separate token budget per plane [REC — doc 14 §rate-budget]). (2) Move backfills to the Batch API (50% discount, 24h–7d window) — allowed for corpus re-extraction, **never** for user-accepted Lane-3 jobs [DECISION CORE-BRIEF §7]. (3) If storm persists with no internal cause: provider-side limit change → verify tier (OD-03), open ticket with Groq.
- **Verify:** 429 share < 1%; freshness SLO recovering; no `INCOMPLETE` jobs created by the storm (they auto-resumed).
- **Never:** never raise concurrency to "push through" a storm; never sample the corpus to cut calls [DECISION GREENFIELD §2.5].

### RB-4 — Provider outage → circuit to Lane 0

- **Trigger:** breaker OPEN (`nip_serve_circuit_state`), `system_state=pinned_lane0`, degraded-share alert.
- **Diagnose:** confirm scope: Groq status page vs local egress (proxy logs) vs credential expiry (vault TTL, 401s vs timeouts — the legacy expired-key outage looked like a model failure [FACT MASTER_PROMPT R16]).
- **Remediate:** nothing to force — the breaker manages entry/exit (§6.3). Operator actions: post the status banner text (doc 18 templated «انقطاع مزود الاستدلال…»), pause non-urgent enrichment (`opsctl pool pause enrich`) to shorten the recovery queue, verify Lane-0 serving is green (synthetic golden question).
- **Verify:** HALF_OPEN canaries pass; queued Lane-3 jobs resume; degraded share returns < 1%; post-incident: count of turns answered under `pinned_lane0` reported in the weekly ops review.
- **Never:** never disable the breaker to "test"; never switch to an unverified model id mid-incident (I17 — registry + benchmark first; doc 14 fallback table is pre-approved).

### RB-5 — Rebase gone wrong

- **Trigger:** rebase operation dashboard red: validation-exclusion spike, quote re-verification failures above gate, or a flip that must be reverted.
- **Diagnose:** `transcript.rebase_operation` + `rebase_item` states (doc 06 §4): which step, which sessions excluded, verifier failure classes.
- **Remediate — before flip:** the design is fail-closed — the flip is blocked automatically if any required evidence fails (GREENFIELD §9.2 step 6); fix cause (provider re-delivery, validation thresholds per doc 06 §4) and resume the operation (it is resumable per `rebase_item`). **After flip:** the old source is retained 90 days read-only [DECISION doc 06 §4]; `opsctl rebase revert <op_id>` re-points `active_transcript` to the retained source atomically and re-invalidates caches/embeddings/live views exactly as a forward flip does — same code path, opposite direction.
- **Verify:** post-revert: full R7 sweep over served artifacts touched during the bad window; affected published packs assessed for reissue (doc 06/pack supersession — never in-place edits).
- **Never:** never map old turn indices onto new ones to "save" findings [FACT GREENFIELD §9.2]; never extend the flip past a failed quote gate "temporarily".

### RB-6 — Taxonomy reclassification backlog

- **Trigger:** `nip_gov_reclassification_backlog_sessions` > 20% of corpus for 14d; live-view exclusions rising (findings pinned to superseded taxonomy_version excluded from live analytics until reclassified [DECISION ADR-0011; doc 08 §13]).
- **Diagnose:** D4: which taxonomy version bump; reclassification run progress (`extraction_run` filtered `run_kind=reclassify`); whether stalled on review (human) or compute (pool).
- **Remediate:** compute-bound → schedule Batch API reclassification (backfill class, allowed); review-bound → steward triage session; the live view remains honest meanwhile — answers declare reduced coverage (I8) rather than mixing taxonomy versions (I13).
- **Verify:** backlog trending to < 5%; parity between pre/post-bump counts explained in the taxonomy changelog (counted-twice-on-purpose doctrine, MASTER_PROMPT §4.3).
- **Never:** never serve mixed-version aggregates; never bulk-approve proposals to clear the queue.

### RB-7 — Ingestion DLQ growth / reconciliation deficit

Summary here; full procedure in doc 09 §7. Trigger: DLQ depth/age alert or roll-up deficit (`provider_in ≠ matched + queued`). Diagnose from `ingest.reconciliation_result` + DLQ error classes. Remediate via `opsctl repair plan --period <month>` → replay manifest (doc 09 §7.2). SEV-1 if the counter identity itself fails (silent-loss risk).

### RB-8 — DB pool saturation / readiness flapping

Trigger: pool-waiter alert, `/readyz` 503s. Diagnose: `pg_stat_activity` by role — the web/worker split (separate roles + pools, doc 07 §7) localizes the offender; long-running analytical query from `nip_web` should be impossible (statement_timeout=10s on the web role [REC]); look for worker bulk jobs missing `SET LOCAL work_mem` discipline. Remediate: kill offending backend (`opsctl db kill <pid>` — audited), throttle the source pool, raise pool size only with §8.5 math. Never: raise `max_connections` as a reflex.

### RB-9 — Per-session data purge (DSR erasure request)

Referenced by doc 16 §9 (data-subject rights). Trigger: steward-verified DSR via the authority's channel [ASSUME OD-16]. Procedure: `opsctl purge session <session_uid> --dsr <ticket>` deletes/anonymizes: `transcript.turn` text (row kept, text nulled, `purged=true`), findings + quote refs for the session, evidence units + embeddings, staged raw payloads in MinIO (versioned delete), `legacy_snapshot` rows if present (exception to read-only, executed by admin role, audited). Published frozen packs quoting the session → reissue path with cause `CORPUS` (doc 16 §9 — never in-place edits). Verify: `opsctl purge verify <session_uid>` proves zero text remnants across schemas + object store; audit event chain complete.

---

## 8. Capacity model

### 8.1 Baselines and the re-measure rule

Planning baselines [FACT CORE-BRIEF §11, measured 2026-08-02]: 16,911 sessions · 399,501 turns (≈ 23.6 turns/session) · 14 months · monthly volume 556–1,512 (2.7× swing) · ~409k legacy knowledge rows (re-derived, not migrated). **[DECISION GREENFIELD §1.3]** every number below is re-measured by the committed measurement script before use in procurement or stakeholder material; this section fixes the *model*, not the figures.

Derived planning quantities [INFER]: average Arabic session transcript ≈ 3.5–5k tokens (23.6 turns × ~150–200 tokens); extraction ≈ 1 strict-schema call/session ≈ 4–6k in + 1–2k out tokens; corpus full pass ≈ **~100–135M tokens total**.

### 8.2 Serving plane

| Quantity | Estimate | Basis |
|---|---|---|
| Interactive users at launch | ≤ 50 named, ≤ 10 concurrent | doc 03 personas [INFER] |
| Peak request rate | ≤ 1 req/s sustained, 5 req/s burst | small expert population |
| Web replicas | 2 × (2 vCPU / 4 GB) | headroom + zero-downtime deploys |
| `nip_web` pool | 10 conns/replica (20 total), `statement_timeout=10s` | Lane-0 queries are index-backed aggregates < 100ms p95 (doc 11 compiler output) |

Serving is **not** the sizing constraint at any plausible growth; the enrichment plane is.

### 8.3 Enrichment throughput

**Steady state** (peak month 1,512 sessions ≈ 70/business-day): extraction 70 calls/day is trivial; the 24h freshness SLO is dominated by queue scheduling, not compute. Pool: 4 concurrent extraction workers suffice with > 10× headroom.

**Full-corpus pass** (bootstrap re-derivation, taxonomy bump, rebase re-extraction — the design-driving case):

| Constraint | Math | Result |
|---|---|---|
| Pure compute at `openai/gpt-oss-120b` ~500 t/s [FACT groq-docs 2026-08-02] | 16,911 calls × ~2s gen + net ≈ 3.5s/call ÷ 8 concurrent | ≈ 2.1 h |
| Tokens-per-minute cap (assume 300k TPM developer tier [ASSUME OD-03]) | ~120M tokens ÷ 300k TPM | ≈ 6.7 h — **TPM-bound, not compute-bound** |
| Batch API alternative (backfills only) | 24h–7d window, 50% discount, JSONL ≤ 50k lines / ≤ 200 MB → corpus fits in 1 batch by lines; split to ~3 files by size | overnight, half cost [DECISION CORE-BRIEF §7: backfills only] |

Planning consequence: online full-pass ≈ one working day at developer tier; enterprise tier (OD-03) would compress to ~2h. Lane-3 job wall-clock at defaults (`DEEP_JOB_CONCURRENCY=8`): one month partition ≈ 1,208 tasks × 3.5s ÷ 8 ≈ **9 min**; a 14-month all-corpus job ≈ **2.1 h** — consistent with the §2.4 p95 < 4h threshold.

### 8.4 Embedding and storage growth

| Item | Baseline | Growth ×2 (36 mo) | Notes |
|---|---|---|---|
| Evidence units (≈ turns, merged windows) | ~400k | ~1.4M | doc 13 §RAG lifecycle |
| Embedding pass, bge-m3 1024-dim [REC doc 14/EXP-04] | GPU (L4-class): ~250–400 units/s → 17–27 min full pass; CPU-only: 30–50/s → 2.2–3.7 h | ×3.5 | full re-embed is a scheduled maintenance job; incremental is negligible |
| pgvector storage | 400k × 1024 × 4B ≈ 1.6 GB + index ≈ 3–4 GB total | ~12 GB | filter-first exact scan under `RETRIEVAL_EXACT_CAP`; ANN (HNSW) only above cap [DECISION MASTER_PROMPT §7.1]; re-tune index after bulk loads — the legacy ivfflat `lists=100`-forever drift is the pinned lesson [FACT arch/06 §5] |
| Postgres total | < 25 GB at launch [INFER] | < 80 GB | single instance comfortable; no partitioning needed except audit/serve monthly partitions (doc 08) |
| MinIO (raw payloads, artifacts, snapshots, backups) | ~10 GB + backup sets | ~40 GB | versioned buckets |

### 8.5 Pool sizing summary (single source for RB-8 decisions)

| Pool | Size [REC] | Scale trigger |
|---|---|---|
| `nip_web` DB pool | 20 total | waiters > 0 at p95 load |
| `nip_worker` DB pool | 20 (ingest 4 · enrich 8 · jobs 4 · packs/maintenance 4) | queue oldest-age SLA breaches with idle CPU |
| Groq concurrency: serving | effective ≤ 8 in-flight (R16 per-turn caps do the real bounding) | — |
| Groq concurrency: enrichment | 8 (tunable at runtime) | 429 share > 5% → down; headroom > 0.5 sustained + backlog → up |
| `DEEP_JOB_CONCURRENCY` | 8 | same rule; never raised during a 429 storm (RB-3) |
| Embedding service | 1 GPU worker (or 8 CPU threads) | queue depth alert §2.2 |
| Postgres `max_connections` | 120 (60 app + exporters + superuser reserve) | never raised as a reflex (RB-8) |

### 8.6 Growth scenarios and revisit triggers

- **×2 volume** (national rollout of advisory programmes): no architectural change; enrichment pool 8→12; Prometheus retention unchanged. Trigger to act: freshness SLO < 0.95 two consecutive weeks with pools maxed.
- **×5 volume or second corpus** (new session types): move Lane-3/backfill traffic permanently to Batch API windows; consider read replica for `nip_web` (revisit ADR-0003 single-instance); embedding service to dedicated GPU node. Trigger: full-corpus pass > 24h online, or DB > 300 GB, or sustained TPM ceiling at enterprise tier.
- Model-side capacity is re-evaluated at every registry change (I17) — a deprecation replacement with lower TPS reruns this section's math as part of the doc 14 switch checklist.

---

## 9. Backup and restore

### 9.1 What is backed up (and what deliberately is not)

| Asset | Mechanism | Schedule | Retention [ASSUME OD-08] |
|---|---|---|---|
| PostgreSQL (all schemas incl. `legacy_snapshot`) | **pgBackRest**: full + differential + continuous WAL archiving to MinIO bucket `nip-backup-pg` (versioned, separate credentials) | full weekly Sun 02:00 AST; diff daily 02:00; WAL `archive_timeout=60s` | 4 full sets (≈ 5 weeks PITR window); monthly full retained 18 months |
| MinIO data buckets (raw payloads, pack renders, exports, eval store) | bucket versioning ON + `mc mirror` nightly to secondary storage target [ASSUME OD-01 for target locality] | nightly 03:00 | versions 90d; pack renders and snapshot artifacts exempt from expiry (see §9.3) |
| Grafana/Prometheus/Loki config | in-repo (dashboards-as-code); TSDB data is *not* backed up (rebuildable, §3.5 retention only) | — | — |
| Secrets | **never in backups** — vault is the system of record [DECISION CORE-BRIEF §8]; recovery = re-issue per §10.3 | — | — |
| Legacy bootstrap snapshot + manifest | immutable object, `nip-snapshots` bucket, object-lock/WORM | written once per snapshot (doc 20 §1) | as long as any published number derives from it [DECISION MASTER_PROMPT §13.8] |

Contrast pinned: the legacy system had three duplicated backup scripts, two with hardcoded passwords, none scheduled, no restore script, no drill, and pruning that could delete the only good backup [FACT arch/06 §5]. Every property is inverted here: one mechanism, scheduled by the platform (not a docstring), restore-tested on a calendar, prune only after verify.

### 9.2 pgBackRest configuration sketch

```ini
[global]
repo1-type=s3            ; MinIO endpoint, dedicated backup credentials (not nip app creds)
repo1-retention-full=4
repo1-cipher-type=aes-256-cbc          ; passphrase from vault at process start
process-max=4
start-fast=y

[nip]
pg1-path=/var/lib/postgresql/17/main
archive-async=y
```

`archive_command` failure ⇒ `NipWalArchiveBroken` SEV-1 (§5.3) — WAL backlog on the primary is the RPO clock ticking. `pgbackrest verify` runs weekly after the full; **prune never runs unless the latest full verifies** (inverting the legacy prune-regardless bug).

### 9.3 Object-store versioning rules

- Buckets `nip-raw-payloads`, `nip-artifacts`, `nip-eval-store`: versioning ON; deletes are soft for 90d.
- Buckets `nip-snapshots` (legacy bootstrap + Lane-3 corpus snapshots referenced by packs) and `nip-pack-renders`: **object-lock (WORM), no expiry**; a quarterly job cross-checks `packs.publication` and `jobs.corpus_snapshot` references and reports (never auto-deletes) unreferenced candidates to the steward.
- DSR purge (RB-9) is the only sanctioned versioned-delete on locked content, executed with the compliance ticket recorded.

### 9.4 Restore drills — scheduled and evidenced

[DECISION GREENFIELD §17 "restore testing"; CORE-BRIEF §8 "scheduled restore drills"]

- **Cadence:** quarterly, plus once before each cutover gate G2/G4 (doc 20), plus after any major schema migration batch.
- **Procedure (scripted as `opsctl drill restore`):** provision scratch instance → `pgbackrest restore --type=time` to a random point inside the PITR window → run verification suite: Alembic head matches expectation for that time; per-table row counts vs nightly `ops` digests; golden-suite smoke (10 Lane-0 questions) against the restored DB; R7 spot-verification of 100 random stored quotes against restored transcript turns.
- **Evidence:** drill writes `ops.audit_event(action='restore_drill')` with duration, RPO/RTO achieved, checks passed/failed, and a signed report to `nip-artifacts`. `nip_plat_restore_drill_age_days` > 100 → SEV-3. **A drill that was not evidenced did not happen** — the metric reads the audit row, not a human claim.
- **Pass criteria:** achieved RTO ≤ target (§10.1), zero verification failures; any failure opens a SEV-2 with the backup chain frozen (no prune) until resolved.

---

## 10. Disaster recovery

### 10.1 Targets [REC — alternatives and triggers below]

| Scenario | RPO target | RTO target | Mechanism |
|---|---|---|---|
| Postgres corruption / bad migration | ≤ 5 min (WAL) | ≤ 2 h | PITR to pre-event timestamp on same host |
| DB host loss | ≤ 5 min | ≤ 4 h | rebuild from IaC + pgBackRest restore from MinIO |
| MinIO/object-store loss | ≤ 24 h (nightly mirror) for data buckets; 0 for WORM buckets (mirrored synchronously on write [REC]) | ≤ 8 h | secondary storage target [ASSUME OD-01] |
| Full environment loss | ≤ 24 h | ≤ 72 h | off-environment copy of backup repo + IaC rebuild in alternate approved zone [ASSUME OD-01 — existence of a second approved zone is an owner decision; without it, this row's commitment is best-effort and stated so to stakeholders] |
| Vault/secrets loss | n/a (re-issue) | ≤ 8 h | §10.3 |
| Groq extended outage | n/a — continuity, not DR | Lane 0 unaffected | §6.3 circuit; Lane-3 queue paused; no data at risk |

Alternatives considered: warm standby Postgres (streaming replica) would cut DB RTO to minutes — deferred: single-team pilot, one more stateful component; **revisit trigger:** executive packs become an operational dependency (post-G4, doc 20) or RTO breach in any drill. RPO tighter than 5 min (sync replication) rejected: no requirement supports the operational cost; ingestion sources are replayable (re-ingest closes any tail gap — doc 09 idempotent runs), so *effective* data loss for ingested facts is near zero even at RPO 5 min; the truly unreplayable data are serve-plane conversations and review decisions, for which 5 min is accepted [REC].

### 10.2 DR runbook skeleton (full text lives with RB-set in the ops repo)

1. Declare incident (SEV-1), freeze deploys, snapshot current state of what survives.
2. Restore order: **Postgres → MinIO data buckets → workers → web** (workers before web so `/readyz` dependencies come up green; queues drain backlog before users return).
3. Post-restore verification = §9.4 suite + reconciliation roll-up for the gap window (`opsctl repair plan --period <gap>`) — re-ingest replays anything the RPO window lost (idempotent by construction, doc 09).
4. Honest-state obligations: any answer served in the gap-repair window carries `system_state=partial`; if a published pack's underlying data was restored, the pack integrity check (checksum vs `packs.publication` stamp) runs before the pack is servable again.
5. Post-incident: RPO/RTO achieved vs target recorded in `ops.audit_event`; §10.1 table re-argued if missed.

### 10.3 Secrets and credential recovery

Re-issue order after vault loss: DB roles → MinIO backup credentials (restores gated on this) → OIDC client (users can wait) → Groq API key → Read.ai OAuth. **Read.ai is the special case [FACT MASTER_PROMPT §13.8; OD-13]:** the refresh token rotates on every refresh, so there is no "restore the old token" — recovery is a fresh OAuth grant on NIP's own client. If the platform's client registration itself is lost, re-registration has provider lead time; this is one more reason OD-13's early, NIP-owned client request precedes any cutover dependency (doc 20 §2).

### 10.4 Snapshot-file retention (the DR ↔ governance joint)

Three snapshot classes, one rule each:
- **Legacy bootstrap snapshot** (doc 20): WORM, retained as long as any published number derives from it — practically, life of the platform or explicit owner disposal decision after full re-derivation and G5 [ASSUME OD-08].
- **Lane-3 corpus snapshots/manifests** referenced by served artifacts: retained while the artifact is servable; unreferenced after artifact expiry → steward-reviewed deletion (never automatic).
- **Pack corpus stamps**: packs are immutable and eternal (ADR-0011); their manifests and checksums are part of the pack render bucket (WORM).

---

## 11. Operations calendar and evidence ledger

| Cadence | Activity | Evidence artifact |
|---|---|---|
| Continuous | WAL archiving; alert evaluation; synthetic golden probe hourly | metrics + `ops.audit_event` |
| Daily | pgBackRest diff; MinIO mirror; pipeline reconciliation roll-up review (D2) | backup verify log; roll-up report row |
| Weekly | pgBackRest full + verify; ops review (SEV-3 queue, threshold tuning §2, spend report D3 to owner) | review minutes in repo |
| Monthly | capacity snapshot vs §8 model; model-registry deprecation sweep (I17); restore-drill age check; retention jobs (OD-08) execution report | `ops.audit_event(action='ops_review')` |
| Quarterly | **restore drill (§9.4)** · access review (doc 16 §2: role grants, break-glass log) · alert-threshold recalibration against measured baselines · DR tabletop on one §10.1 scenario | drill report; access-review sign-off |
| Per release | golden suite + structural checks in CI (doc 15); dashboard-as-code drift check; runbook links validated | CI record |
| Per model/prompt change | replay evaluation before switch (GREENFIELD §18.3) | eval report id in `ops.model_registry` |

---

## 12. Open decisions and revisit triggers

| ID | Decision needed | Safe working assumption | Impact if changed |
|---|---|---|---|
| **OD-22 (new, §5.2)** | On-call staffing + notification channels in the gov network; 24/7 vs business hours | business-hours 2-person rotation; nights covered by automatic safe degradation (circuit, queue pause) | alert routing config; §6 degraded-state depth; doc 21 staffing |
| OD-01 | Deployment target; existence of a secondary approved zone | container platform, approved environment; full-site DR best-effort until second zone confirmed | §10.1 site-loss row becomes a commitment; MinIO mirror target |
| OD-03 | Groq account tier | developer tier (300k TPM class) | §8.3 full-pass math compresses ~3×; §2.2 headroom thresholds |
| OD-08 | Retention windows | Loki 90d; R13/audit ≥ 18 mo; monthly fulls 18 mo | §3.5, §9.1 tables |
| OD-16 | SIEM mandate | local stack only; ship-copy on mandate | §1.1 export pipelines |

**Revisit triggers for this whole document:** first SEV-1 (post-incident review re-opens the relevant section) · any restore drill miss · freshness SLO missed two weeks running · ×2 volume trigger (§8.6) · Groq tier confirmation (OD-03) · second approved zone confirmed (OD-01) → upgrade §10.1 from best-effort to committed.

*End of document 19. Doc 20 consumes the dashboards and drills as cutover-gate evidence; doc 21 turns §11 into work packages; doc 22 records ADR-0019/ADR-0020 with this document as the argument.*

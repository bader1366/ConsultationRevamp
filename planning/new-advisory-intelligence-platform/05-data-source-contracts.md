# 05 — Data Source Contracts
**Platform:** Nwafeth Intelligence — منصة نوافث لذكاء الجلسات الاستشارية (Monsha'at Advisory Session Intelligence Platform)
**Status:** Draft for owner review · **Date:** 2026-08-02 · **Author:** Planning package (Fable 5)
**Depends on:** 02, 04 · **Feeds:** 06 (provider strategy), 08 (data model), 09 (ingestion pipelines), 16 (security/PII), 20 (migration), 22 (ADR/OD register)
**Sources used:** GREENFIELD §2.2, §8.1, §9.1, §11.1–11.4, §15.2, §19.1–19.2, §23-05; MASTER_PROMPT §13.8, Appendix A dimension notes; arch/05 §3 (Read.ai interface reality, full), arch/04 §9 (services-report mechanics), arch/02 §4 (bridge + consultant tables); CORE-BRIEF §6, §11, §12

---

## 0. Purpose and reading guide

This document is the **contract book for every data source** the platform ingests. One contract per source; each contract declares owner, transport, auth, cursor/watermark, idempotency, update/delete semantics, freshness SLA, PII classification, quality rules, dead-letter behaviour, and reconciliation report content, plus a field-by-field data dictionary [DECISION GREENFIELD §8.1 — the adapter contract list is a requirement, not a suggestion].

Two governing principles from GREENFIELD §2.2:

1. The platform combines **two classes of data** — conversation-provider data and Monsha'at internal session data — and must *reconcile* them, not merely store them side by side.
2. **Fields with similar names are never assumed identical.** Every dictionary row below carries a `similar-name warning` column precisely because the legacy corpus contains at least three distinct meanings of «قطاع», two unrelated "ratings", three disagreeing consultant tables, and two unrelated "durations" [FACT MASTER_PROMPT App. A; arch/02 §4].

Source IDs used throughout the package:

| Source ID | Name | Class | Contract § |
|---|---|---|---|
| `SRC-READAI` | Read.ai meetings + transcripts (interim) | provider | §2 |
| `SRC-TSP` | Future transcript provider via `TranscriptSource` | provider | §3 |
| `SRC-INT` | Monsha'at internal session data | internal | §4 |
| `SRC-DIR` | Consultant directory | internal | §5 |
| `SRC-REF` | Programme/service/window/channel reference data | internal | §6 |
| `SRC-EVAL` | Beneficiary rating + consultant evaluation sources | internal | §7 |
| `SRC-OUT` | Optional outcome / follow-up sources | internal | §8 |
| `SRC-SNAP` | One-time legacy snapshot import | bootstrap | §9 |

Identity resolution across sources (the crosswalk into `core.advisory_session`) is owned by doc 09; this document defines what each adapter **stages** and what the reconciliation layer must report. Adapters never resolve identity and never write magic strings — unresolved identity is a nullable FK + `resolution_status` + provenance [DECISION CORE-BRIEF §6; the legacy `'PENDING'` lesson, arch/04 §9].

---

## 1. Shared contract framework

### 1.1 Contract checklist (every source declares all twelve)

`owner` · `transport` · `auth & credential reference` · `cursor/watermark strategy` · `idempotency key` · `update/delete semantics` · `freshness SLA` · `PII classification per field group` · `quality rules (typed, with severities)` · `dead-letter behaviour` · `reconciliation report content` · `data dictionary`. A source whose contract omits any of these does not pass review [DECISION GREENFIELD §8.1].

### 1.2 Adapter interface (planning pseudocode)

```python
class SourceAdapter(Protocol):
    source_id: str            # "SRC-READAI" … registered in ingest.source_system
    contract_version: str     # semver; bumped when this document's section changes

    def plan_run(self, cursor: CursorState, mode: RunMode) -> RunPlan: ...
    # mode ∈ {INCREMENTAL, SWEEP, BACKFILL} — sweeps are scheduled, not ad hoc

    def list_changes(self, plan: RunPlan) -> Iterator[ChangeRef]: ...
    def fetch(self, ref: ChangeRef) -> RawRecord: ...
    # RawRecord = raw bytes + fetch metadata; stored BEFORE any parsing (I6-adjacent)

    def classify(self, raw: RawRecord) -> QualityVerdict: ...
    # verdict ∈ {OK, WARN(rule_ids), BLOCK(rule_ids)} — BLOCK ⇒ DLQ, never silent skip

    def normalize(self, raw: RawRecord) -> list[StagedRow]: ...
    # typed staging rows; NO identity resolution, NO joins, NO derived analytics

    def reconcile(self, run: IngestionRun) -> ReconciliationReport: ...
```

[REC] Adapters are pure I/O + typing; all cross-source logic lives in the resolver (doc 09). Alternative: adapters that resolve identity inline (legacy `_resolve_consultant_id` inside ingestion) — rejected because it welded a fragile Excel bridge into the ingest hot path and minted `'PENDING'` rows [FACT arch/04 §9]. Revisit trigger: none foreseen; this boundary is structural.

### 1.3 `ingest` schema objects backing every contract (DDL sketch; doc 08 owns final DDL)

```sql
CREATE TABLE ingest.source_system (
  source_id        text PRIMARY KEY,          -- 'SRC-READAI' …
  display_name_ar  text NOT NULL,
  source_class     text NOT NULL CHECK (source_class IN ('provider','internal','bootstrap')),
  contract_version text NOT NULL,
  credential_ref   text,                      -- vault path REFERENCE, never a value (§11.1)
  freshness_sla    interval NOT NULL,
  enabled          boolean NOT NULL DEFAULT true
);

CREATE TABLE ingest.ingestion_run (
  run_id       bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  source_id    text NOT NULL REFERENCES ingest.source_system,
  mode         text NOT NULL CHECK (mode IN ('incremental','sweep','backfill','snapshot')),
  started_at   timestamptz NOT NULL, finished_at timestamptz,
  status       text NOT NULL CHECK (status IN ('running','succeeded','failed','partial')),
  stats        jsonb NOT NULL DEFAULT '{}'    -- the reconciliation report counters (§1.6)
);

CREATE TABLE ingest.source_cursor (
  source_id    text NOT NULL REFERENCES ingest.source_system,
  cursor_name  text NOT NULL,                 -- a source may keep several watermarks
  cursor_value jsonb NOT NULL,                -- {"high_watermark":"2026-08-01T00:00:00Z",...}
  updated_at   timestamptz NOT NULL,
  updated_by_run bigint REFERENCES ingest.ingestion_run,
  PRIMARY KEY (source_id, cursor_name)
);

CREATE TABLE ingest.raw_payload (
  payload_id     bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  source_id      text NOT NULL REFERENCES ingest.source_system,
  run_id         bigint NOT NULL REFERENCES ingest.ingestion_run,
  natural_key    text NOT NULL,               -- source-scoped idempotency key (per contract)
  payload_sha256 char(64) NOT NULL,
  object_uri     text NOT NULL,               -- MinIO URI; payloads >1MB live in object storage
  fetched_at     timestamptz NOT NULL,
  UNIQUE (source_id, natural_key, payload_sha256)   -- same bytes twice = no-op
);

CREATE TABLE ingest.dlq_item (
  dlq_id       bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  source_id    text NOT NULL, run_id bigint NOT NULL,
  natural_key  text, payload_ref bigint REFERENCES ingest.raw_payload,
  error_class  text NOT NULL CHECK (error_class IN
    ('AUTH','RATE_LIMIT','TRANSPORT','SCHEMA_DRIFT','QUALITY_BLOCK','PARSE','UNKNOWN')),
  detail       jsonb NOT NULL,
  state        text NOT NULL DEFAULT 'new' CHECK (state IN
    ('new','retrying','dead','replayed','discarded')),
  attempts     int NOT NULL DEFAULT 0,
  first_seen   timestamptz NOT NULL, last_attempt timestamptz,
  resolved_by  text, resolved_at timestamptz    -- steward identity for dead→replayed/discarded
);

CREATE TABLE ingest.reconciliation_result (
  run_id    bigint NOT NULL REFERENCES ingest.ingestion_run,
  report    jsonb NOT NULL,                   -- §1.6 envelope, validated against JSON Schema
  PRIMARY KEY (run_id)
);
```

Quality observations land in `ops.data_quality_observation` (CORE-BRIEF §6 places them in `ops`) — schema in §10.3. Credentials are **references only**; values live in the vault [ASSUME OD-02].

### 1.4 PII classification scheme (used by every field group below; doc 16 owns the outbound matrix)

| Class | Definition | Handling (I15, GREENFIELD §15.2) |
|---|---|---|
| **P0** | No personal data (counts, codes, timestamps, UUIDs) | Unrestricted within RBAC |
| **P1** | Pseudonymous / internal identifiers (surrogate keys, session UUIDs, consultant surrogate key) | Unrestricted within RBAC; safe in prompts as stable placeholders |
| **P2** | Personal data — names, free text that may incidentally contain personal details (transcripts, comments, notes) | Restricted roles; enters model prompts only pseudonymized inside `<<<DATA…>>>` delimiters [ASSUME OD-04]; never in logs |
| **P3** | High-sensitivity direct identifiers — national IDs, phone, email, birth date | Never leaves `ingest`/restricted `core` columns; **never in any prompt, log, export, or answer**; column-level grants; break-glass audit only [ASSUME OD-09] |

### 1.5 Dead-letter state machine (shared)

```
NEW ──auto──▶ RETRYING (≤5 attempts, exponential 1m/5m/30m/2h/12h, jittered)
RETRYING ──success──▶ REPLAYED (closed)
RETRYING ──exhausted──▶ DEAD  ── steward action ──▶ REPLAYED | DISCARDED (reason mandatory)
```

Rules: `AUTH` errors do **not** auto-retry beyond attempt 1 — they page ops (a rotated-token conflict looks exactly like this; see §2.3) [FACT MASTER_PROMPT §13.8]. `QUALITY_BLOCK` items skip retry (retrying identical bytes cannot pass) and go straight to `DEAD` for steward triage. A DLQ item is **never** dropped silently; `DISCARDED` requires a reason string and appears in the run's reconciliation report (I16). DLQ depth per source is a paged alert at >50 or age >72h [REC].

### 1.6 Reconciliation report envelope (every run of every adapter emits one)

```json
{
  "run_id": 4211, "source_id": "SRC-READAI", "mode": "incremental",
  "window": {"from": "2026-08-01T00:00:00Z", "to": "2026-08-02T00:00:00Z"},
  "counters": {
    "listed": 41, "window_skipped": 3, "fetched": 38, "new": 30,
    "updated": 6, "unchanged": 2, "quality_blocked": 0, "dead_lettered": 0
  },
  "invariant": "listed == window_skipped + fetched + dead_lettered",
  "watermark": {"before": "…", "after": "…"},
  "volume_band_check": {"expected_monthly": [556, 1512], "observed_rate_ok": true},
  "join_rates": {"provider_to_internal": 0.97, "cohort": "2026-08"},
  "top_quality_rules": [{"rule_id": "DQ-READAI-002", "hits": 4}],
  "dlq_depth_after": 2,
  "payload_set_sha256": "…"
}
```

The counter identity is enforced in code and asserted in tests — the legacy counters where `listed != valid + skipped_invalid + errors` (window skips counted nowhere) are the anti-pattern [FACT arch/05 §3.6]. `volume_band_check` compares run rate against the measured 556–1,512 sessions/month band [FACT CORE-BRIEF §11] and raises a `WARN` observation when outside it.

### 1.7 Data dictionary TEMPLATE (used for every starter dictionary below)

| Column | Meaning |
|---|---|
| `field` | Canonical staging field name (snake_case) |
| `source path` | JSON path / API field / file column (with Arabic header where applicable) |
| `type` | Logical type + nullability |
| `PII` | P0–P3 per §1.4 |
| `semantics` | One-sentence meaning, Arabic label where the business knows it by name |
| `target` | Canonical destination `schema.table.column` (doc 08) |
| `quality rules` | Rule IDs that reference this field |
| `similar-name warning` | Fields elsewhere with similar names that mean something ELSE |

---

## 2. `SRC-READAI` — Read.ai adapter (interim provider)

Read.ai is **interim** — the contract exists to be replaced invisibly behind the `TranscriptSource` boundary (§3, doc 06) [DECISION GREENFIELD §2.1/§2.2]. Everything in this section is written against the live interface reality documented in arch/05 §3 [FACT arch/05, verified 2026-07-16, token mechanics re-verified 2026-08-02 in MASTER_PROMPT §13.8].

### 2.1 Contract header

| Contract field | Value |
|---|---|
| Owner | Platform data engineering (adapter); Read.ai (source semantics); Monsha'at IT (OAuth client relationship) |
| Transport | HTTPS REST, `https://api.read.ai/v1`; token endpoint `https://authn.read.ai/oauth2/token` (different host) [FACT arch/05 §3.1] |
| Auth | OAuth2 refresh-token flow, HTTP Basic `(client_id, client_secret)` on refresh; **NIP-dedicated OAuth client** [ASSUME OD-13] — see §2.3 |
| Cursor/watermark | Cursor pagination on list + time watermark with overlap re-read; **ordering treated as unverified** — see §2.4 |
| Idempotency key | `(source_id='SRC-READAI', meeting_ulid, payload_sha256)` on raw; `(meeting_ulid, fetched_at)` versions the detail payload |
| Update/delete | No update/delete signal from provider assumed; re-fetch produces a **new immutable payload version**; disappearance detected by sweep → flagged, never deleted (I6) — see §2.6 |
| Freshness SLA | Meeting available for ingest T+24h after meeting end [ASSUME OD-06 — provider processing time unverified]; adapter runs hourly incremental + monthly sweep [REC] |
| Rate limit | Client-side token bucket **80 req / 60 s** shared across list + detail + token refresh [FACT arch/05 §3.5 — provider tolerance at this rate is operationally proven] |

### 2.2 Endpoints and shapes (current reality)

```
GET  /v1/meetings?limit=10[&cursor=<last_id>]           # list; MAX limit=10 (hard API limit)
GET  /v1/meetings/{ulid}?expand[]=transcript&expand[]=summary
     &expand[]=chapter_summaries&expand[]=action_items
     &expand[]=key_questions&expand[]=topics&expand[]=metrics   # detail; 7 expand fields
POST https://authn.read.ai/oauth2/token                 # refresh (Basic auth)
```

- The **list call carries no `expand[]`** — `title` and `participants` arrive in the list payload, so shape classification (§2.7) runs before any detail fetch is spent [FACT arch/05 §3.2].
- Cost per meeting: 2 requests (1/10th of a list page + 1 detail). Backfill of the full 16,911-meeting corpus ≈ 16,911 detail + ~1,700 list requests ≈ **3.9 hours at 80/60s** [INFER from FACT rate limit]; steady state (≤1,512/month) is negligible.
- 429 → sleep `max(Retry-After, 2)` and retry; 5xx retried; 8 attempts then DLQ `TRANSPORT` [REC — carries forward the proven legacy retry envelope, arch/05 §3.5, but exhaustion dead-letters instead of aborting the run].

### 2.3 Auth: independent credential, rotate-on-refresh (OD-13)

**Verified constraint:** Read.ai **rotates the refresh token on every refresh** — the response carries a replacement refresh token and the old one dies. Two systems sharing one credential invalidate each other intermittently and untraceably [FACT MASTER_PROMPT §13.8, `token_manager.py:154,163`]. Therefore:

1. **NIP gets its own OAuth client** (client_id/secret/refresh token) — never the legacy credential, not even briefly during testing [DECISION MASTER_PROMPT §13.8; ASSUME OD-13 — the second client must be requested from Read.ai early; lead time blocks cutover].
2. **Single-writer token service** in the worker plane: one Postgres row (`ingest.provider_token_state`, unique per provider) under `SELECT … FOR UPDATE` with a short lease; the new refresh token is **committed before first use**. Crash between receive-and-commit is the classic loss window — commit order eliminates it.
3. Refresh fires within 120 s of expiry; retry 3× exponential on network/5xx; honour `Retry-After` on 429; **hard-fail 400/401/403 → ops task, no auto-retry** (human re-auth required) [REC — proven policy carried from legacy, arch/05 §3.1].
4. **No secret write-back to config files, ever** — the legacy `.env`-rewrite by the running server is the named anti-pattern [FACT arch/05 §3.1; CORE-BRIEF §8]. The vault holds the bootstrap secret; Postgres holds the live rotating state; alert at `consecutive_failures ≥ 2`.
5. During the build phase **only the legacy system ingests**; NIP works from re-snapshots (§9). NIP's own credential activates at cutover [DECISION MASTER_PROMPT §13.8 phase table].

### 2.4 Cursor/watermark strategy

Legacy relied on newest-first list ordering for `--since` skipping — an **unverified assumption**; if ordering changes, ingestion silently under-ingests and reports success [FACT arch/05 §3.4]. The new contract does not trust ordering:

- **Incremental:** walk cursor pages from the head; stop only after `K=3` consecutive pages contain **only** meetings that are already stored with an identical `payload_sha256` at list level. Store `{last_cursor, high_watermark_start_time, stop_reason}` in `ingest.source_cursor`.
- **Guarded cursor advance:** a page with `has_more=true` and empty `data[]` is a `SCHEMA_DRIFT` DLQ item + run `partial`, never an uncaught `IndexError` aborting the run (legacy hazard) [FACT arch/05 §3.4].
- **Monthly sweep (`mode=SWEEP`):** full cursor walk of the trailing 3 months comparing provider meeting set vs stored set — this is the delete/miss detector (§2.6) and the ordering-assumption insurance.
- **Overlap re-read:** incremental windows overlap 48h; idempotency makes re-reads free.

### 2.5 Idempotency

Raw detail payloads are content-addressed: `UNIQUE (source_id, natural_key=meeting_ulid, payload_sha256)`. Same bytes → no-op (counted `unchanged`); different bytes → **new payload version** appended, prior versions retained (I6). Transcript rows derive only from a specific payload version, so a provider-side transcript change becomes a new `transcript.transcript_version` — never an in-place mutation [DECISION ADR-0007].

### 2.6 Update/delete semantics

- **Updates:** Read.ai gives no change feed; the sweep + overlap re-read detect changed payloads by hash. A changed transcript payload for an already-analysed meeting raises observation `DQ-READAI-010 provider_payload_changed_post_analysis` (WARN) and queues re-extraction per doc 09 — findings are never silently stale (legacy re-ingest hazard: knowledge pointing at a transcript that no longer exists) [FACT arch/04 §10].
- **Deletes:** a stored meeting absent from a sweep window is marked `provider_status='missing_at_source'` with the sweep run id. Raw payloads and derived turns are **retained** (I6); the meeting stays in analytics with its coverage flag. Physical deletion only via the retention policy [ASSUME OD-08].

### 2.7 Validity, title eras, and the regex trap

**Title-format history** [FACT arch/05 §3.6–3.7]:

| Era | Window | Format | What it carries |
|---|---|---|---|
| Era 1 | 2025-05 → ~2026-06 | `^(\d{10})-(UUID)$` | **10-digit national ID** (consultant) + a UUID (internal session id, presumed) — the title itself is **P3 PII** |
| Era 2 | ~2026-06 → now | `^(UUID)-(UUID)$` | Two UUIDs; **which half is the internal session id is a guess** — legacy guessed "trailing" and stored the guess even when nothing matched |

New-contract rules:

1. **Case-insensitive UUID matching.** The legacy regex was lowercase-hex only (`[a-f0-9]`); an uppercase-hex title was silently counted `skipped_invalid` [FACT arch/05 §3.6 — the regex trap]. New pattern: `(?i)[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}`, normalised to lowercase on staging.
2. **No silent drops.** Titles matching neither era → staged with `title_shape='unrecognized'` + observation `DQ-READAI-001` (WARN). They appear in the reconciliation report; a human decides whether a third title era has begun (revisit trigger for this contract).
3. **Both UUID halves staged as claims** (`title_claims jsonb`: `{era, national_id?, uuid_leading?, uuid_trailing?}`). The resolver (doc 09) matches **both** halves against `SRC-INT` session UUIDs; both-match-different-rows → `resolution_status='ambiguous'` → unmatched-identity queue (§10.1). Never a stored guess.
4. **Era-1 national IDs are P3**: staged into a column-restricted field, used only by the identity resolver's crosswalk, pseudonymized immediately to the consultant surrogate key [ASSUME OD-14 — whether NIP may retain consultant national IDs as crosswalk attributes at all, or must hash them on ingest; safe assumption: retain in restricted column, hash exposed everywhere else].

**The two-participants-attended rule** — legacy validity was `exactly 2 participants with attended=true` (using `attended`, never `invited`), and everything else was invisibly discarded [FACT arch/05 §3.6]. New contract: **classify, don't drop**:

```
participant_shape ∈ { two_party_attended,     -- default advisory analytics scope
                      single_party, multi_party, zero_attended }
```

All shapes are ingested and retained; only `two_party_attended` enters the default advisory-session scope, and every aggregate's coverage block can state how many meetings were out-of-shape (I8). Rationale: the filter encodes a business rule ("an advisory session is a 2-person meeting") that belongs in governed scope definitions, not in an ingest-time discard.

### 2.8 Quality rules

| Rule ID | Condition | Severity |
|---|---|---|
| `DQ-READAI-001` | Title matches neither era regex (case-insensitive) | WARN + `title_shape='unrecognized'` |
| `DQ-READAI-002` | `participant_shape != two_party_attended` | INFO (classification, reported) |
| `DQ-READAI-003` | Detail payload has no transcript turns or empty text | WARN; meeting staged, flagged `transcript_empty` — analytics exclude with named coverage exclusion |
| `DQ-READAI-004` | Turn timestamps non-monotonic or missing (baseline: timing 100% present [FACT CORE-BRIEF §11]) | WARN |
| `DQ-READAI-005` | Turn missing speaker label (baseline 5.7% of turns lack role [FACT CORE-BRIEF §11]) | INFO; role resolution downstream (doc 06) |
| `DQ-READAI-006` | Payload fails JSON Schema for the pinned provider schema version | BLOCK → DLQ `SCHEMA_DRIFT` |
| `DQ-READAI-007` | List page empty with `has_more=true` | BLOCK → DLQ `SCHEMA_DRIFT` |
| `DQ-READAI-008` | Meeting `start_time` outside plausible window (before 2025-01 or in the future) | WARN |
| `DQ-READAI-010` | Payload hash changed after findings exist for the meeting | WARN + re-extraction queue |

### 2.9 PII classification per field group

| Field group | PII | Note |
|---|---|---|
| Meeting ULID, timestamps, cursor values | P0 | |
| Era-2 title UUIDs, session UUID claims | P1 | |
| **Era-1 title national ID** | **P3** | Restricted column; crosswalk only (§2.7.4) |
| Participant names, emails | P2 (names) / **P3 (emails)** | Provider sends both; emails never propagate past staging [REC] |
| Transcript text, summaries, chapters, topics, key questions, action items | P2 | Untrusted text — prompt-injection isolation applies (R15/I15; doc 16) |
| Provider metrics (`read_score`, `sentiment`, `engagement`) | P0 | **Source features only, never satisfaction truth** (see dictionary warning) |

### 2.10 Dead-letter & reconciliation specifics

DLQ per §1.5. Reconciliation report (per §1.6) additionally carries: era distribution of new meetings (`era1/era2/unrecognized` counts — a rise in `unrecognized` is the third-era alarm), participant-shape histogram, transcript-empty count, provider-token refresh count and failures, and sweep-mode `missing_at_source` list.

### 2.11 Starter data dictionary — `SRC-READAI` (template §1.7)

| field | source path | type | PII | semantics | target | quality rules | similar-name warning |
|---|---|---|---|---|---|---|---|
| `meeting_ulid` | list/detail `id` | ulid NOT NULL | P0 | Provider meeting identity | `core.provider_meeting_map.provider_ref` | — | NOT the internal session id; NOT `advisory_session.id` |
| `title_raw` | list `title` | text NOT NULL | **P3 (era 1)** | Booking-generated title carrying identity claims | staging only; claims extracted | DQ-001 | — |
| `title_claims` | derived | jsonb | P1/P3 | `{era, national_id?, uuid_leading?, uuid_trailing?}` | `core.provider_meeting_map.title_claims` | DQ-001 | national_id is the **consultant's**, not the beneficiary's |
| `start_time` / `end_time` | detail | timestamptz | P0 | Provider wall-clock meeting window | `core.advisory_session.provider_start_at` (via map) | DQ-008 | ≠ `SRC-INT.scheduled_at` (booked) and ≠ `SRC-INT.actual_date` (operational record) |
| `duration_provider_s` | derived end−start | int | P0 | Provider-measured elapsed time | provider feature column | — | ≠ `SRC-INT.duration_minutes` (booked/recorded duration); never mix in one metric |
| `participants[].name` | list | text | P2 | Display name as captured by provider | `core.participant.display_name` | — | Not authoritative — `SRC-DIR` name wins for consultants |
| `participants[].email` | list | text NULL | **P3** | Provider account email | staging only, dropped after crosswalk attempt | — | — |
| `participants[].attended` | list | bool | P0 | Provider attendance flag | shape classifier input | DQ-002 | ≠ `SRC-INT.attendance_status` (operational truth); disagreement feeds CAP-D3/CAP-D9 |
| `transcript.turns[]` | detail expand | array | P2 | Speaker-labelled turns: `speaker_name`, `text`, start/end ms | `transcript.turn` (immutable, versioned) | DQ-003/004/005 | text is UNTRUSTED (R15) |
| `summary`, `chapter_summaries`, `topics`, `key_questions`, `action_items` | detail expands | text/array | P2 | Provider-generated derivatives | provider-feature tables; **never treated as findings** (I13 — no provenance) | — | provider "action items" ≠ NIP extracted recommendations (CAP pipeline) |
| `metrics.read_score`, `.sentiment`, `.engagement` | detail expand | numeric | P0 | Provider scoring — opaque methodology | source-feature columns (GREENFIELD §11.4) | — | **NEVER a satisfaction or quality metric** — CAP-A3/CAP-D1 compute their own signals |
| `payload` | full JSON | jsonb→object store | mixed | 1:1 raw archive | `ingest.raw_payload` | DQ-006 | — |

---

## 3. `SRC-TSP` — future transcript provider via `TranscriptSource`

The replacement provider's identity and timeline are unknown [ASSUME OD-06]. This contract therefore pins the **normalized interface** every provider adapter (including the Read.ai adapter, retrofitted) must emit, so replacement is invisible above the boundary [DECISION GREENFIELD §2.1, §9.1; ADR-0007]. Doc 06 owns the benchmark, go/no-go gate, and `rebase_transcript` workflow; doc 09 owns the rebase pipeline mechanics.

| Contract field | Value |
|---|---|
| Owner | Platform data engineering; provider identity unknown — Read.ai remains the interim source [ASSUME OD-06] |
| Transport | Provider-specific behind the adapter; contract is the normalized record below. Webhook/push support is a benchmark criterion (GREENFIELD §9.3), not an assumption |
| Auth | Independent NIP credential from day one (the OD-13 lesson generalised: **never share rotating credentials across systems**) |
| Cursor/watermark | Provider-native change feed if it exists; else the §2.4 cursor+sweep pattern is the default template |
| Idempotency key | `(provider_id, provider_meeting_ref, source_version, payload_sha256)` |
| Update/delete | New provider payload = new `transcript_version`; **exactly one version active per meeting**; flips only via `rebase_transcript` with quote re-verification (doc 06) — never in-place |
| Freshness SLA | Contractual target for provider selection: T+6h transcript availability [REC — benchmark criterion; alternatives: T+24h acceptable at pilot; revisit at provider contract signature] |
| PII | Same classes as §2.9 — transcript P2, any embedded identifiers P3, diarization labels P2 |
| DLQ / reconciliation | §1.5/§1.6 unchanged; report adds per-version turn counts and language distribution |

**Normalized `TranscriptSource` record** (GREENFIELD §9.1 — this is the frozen field list; doc 06 elaborates):

| field | type | semantics |
|---|---|---|
| `provider_meeting_ref` | text NOT NULL | Provider's meeting/source identifier |
| `source` + `source_version` | text + text NOT NULL | Provider id and payload/version stamp — provenance on every turn (I6) |
| `acquired_at` | timestamptz NOT NULL | Acquisition time |
| `language` | text | BCP-47; expected `ar-SA` dominant |
| `turn_index` | int NOT NULL | 0-based, unique per version; **never mapped across providers** (re-extract, don't map — GREENFIELD §9.2) |
| `speaker_label` | text | Provider's raw label |
| `resolved_role` | enum `consultant/beneficiary/unknown` | Resolution downstream (doc 06 §role-resolution); `unknown` is honest, not imputed |
| `started_ms` / `ended_ms` | int | Offsets within meeting |
| `text` | text NOT NULL | **Immutable** — no LLM rewriting ever (GREENFIELD §2.6) |
| `confidence` | numeric NULL | Provider ASR confidence where available — quality-gate input |
| `raw_payload_ref` | bigint | FK to `ingest.raw_payload` |

Quality rules `DQ-TSP-001…005` mirror `DQ-READAI-003…007` (empty transcript, non-monotonic timing, missing labels, schema drift, empty page). One addition: `DQ-TSP-006 quote_reverification_failure` (BLOCK on rebase — any stored quote not found as exact substring in the candidate version blocks the source flip; GREENFIELD §9.2 step 6).

---

## 4. `SRC-INT` — Monsha'at internal session data (the authoritative operational record)

This is the second data class of GREENFIELD §2.2 and the **authoritative** side of every reconciliation: the internal session id anchors `core.internal_session_map`. Everything the legacy system obtained through the hand-uploaded "services report" Excel (94,963 rows; positional 49-column parsing; TRUNCATE-on-import destroying the consultant bridge on every upload [FACT arch/04 §9]) must arrive here through a governed feed instead.

### 4.1 Contract header

| Contract field | Value |
|---|---|
| Owner | Monsha'at operations systems team (source); platform data engineering (adapter) [ASSUME OD-07 — the owning system and team must be named at OD-07 resolution] |
| Transport | **Recommended:** daily read-only extract API or managed file drop (CSV/Parquet + manifest + per-file SHA-256), pulled by the adapter [ASSUME OD-07 — safe assumption: read-only API/export]. Alternatives: (a) read replica — rejected as default: couples NIP to source schema churn and violates least-privilege posture; (b) continued manual Excel upload — **interim-only**, hardened per §4.6. Revisit trigger: OD-07 decision |
| Auth | Service account / mTLS or signed URLs per Monsha'at IT standard [ASSUME OD-02]; credential reference in vault |
| Cursor/watermark | `max(updated_at)` per entity with a **48h lookback re-read** window; weekly full-snapshot compare as sweep (row counts + per-column checksums) |
| Idempotency key | `(internal_session_id, updated_at, row_sha256)` — a re-delivered identical row is a no-op; changed hash at same id = new fact version |
| Update/delete | Source rows are mutable (status transitions مجدولة → مكتملة/ملغاة). Adapter stages **versioned facts** (valid-from/valid-to); cancellation/no-show are status values, never row deletions. A physically vanished id → `DQ-INT-009` + reconciliation flag; never propagate deletion into `core` |
| Freshness SLA | T+24h from operational-system commit [ASSUME OD-07 — SLA is part of the OD-07 decision]; staleness > 48h raises an ops alert and stamps affected capabilities' coverage blocks (I8) |

### 4.2 Semantics rules — similar names are NOT the same field

These are contract-level rules, enforced as dictionary annotations + registry dimension definitions (I12):

1. **«قطاع» is three dimensions, never one** [DECISION MASTER_PROMPT App. A; CORE-BRIEF §13.3]: `service_category` (service/business domain of the session, 48 observed values, e.g. «الابتكار، القانونية، دراسة الجدوى، الإقراض والتمويل», 77.3% populated), `government_entity` (transcript-extracted mention — NOT a `SRC-INT` field at all), `business_sector` (beneficiary firm's industry — **UNAVAILABLE** until an external identity source exists). `SRC-INT` supplies only `service_category`.
2. **Two unrelated "ratings"**: the legacy services-report `beneficiary_rating` (1–5, present on 24,171 of 94,963 rows) vs any rating fields in other internal modules. They are separate facts with separate lineage (§7); an adapter must never coalesce them.
3. **Three "dates"**: `scheduled_at` (booked), `actual_at` (held), `created_at` (record creation, Arabic header «تاريخ الإنشاء» in the legacy export). Legacy analytics used them interchangeably; the dictionary pins each.
4. **Two "durations"**: internal booked/recorded duration vs provider elapsed time (§2.11). CAP-D3/D4 compare them deliberately; no metric mixes them.
5. **`attendance` vs `status`**: status is the workflow outcome (مكتملة/ملغاة/…); attendance is who showed up. «لم يحضر المستفيد» is attendance, not status; the source may encode it either way [ASSUME OD-07 — source enum inventory is a mandatory OD-07 deliverable; adapter maps through a versioned enum-mapping table, unknown values → `DQ-INT-003` BLOCK].

### 4.3 Quality rules

| Rule ID | Condition | Severity |
|---|---|---|
| `DQ-INT-001` | `internal_session_id` missing or malformed | BLOCK |
| `DQ-INT-002` | Duplicate `internal_session_id` within one extract with differing content | BLOCK |
| `DQ-INT-003` | `status`/`attendance` value outside the versioned enum mapping | BLOCK (new value ⇒ steward adds mapping, replay) |
| `DQ-INT-004` | `actual_at` present while status ∈ cancelled-family (e.g. «ملغاة») | WARN → CAP-D9 |
| `DQ-INT-005` | `consultant_ref` absent on a completed session | WARN |
| `DQ-INT-006` | `service_category` null (baseline 22.7% unresolved [FACT CORE-BRIEF §11]) | INFO — named coverage exclusion, not an error |
| `DQ-INT-007` | `scheduled_at > actual_at + 90d` or negative durations | WARN |
| `DQ-INT-008` | Extract volume outside ±40% of trailing 3-month mean | WARN (run `partial` until steward acks) |
| `DQ-INT-009` | Previously delivered id absent from full sweep | WARN → reconciliation queue |

### 4.4 PII classification per field group

| Field group | PII |
|---|---|
| Session ids, dates, status/attendance codes, programme/service/window/channel codes, duration | P0/P1 |
| Consultant name, beneficiary name | P2 |
| Beneficiary national ID, phone, email, birth date (present in the legacy export [FACT arch/02 §9]) | **P3 — the adapter's default extract SHOULD NOT include them at all** [REC: negotiate their exclusion at OD-07; if delivered, they stop at staging with column grants] |
| Beneficiary comments, consultant notes/recommendations | P2 (free text; may contain incidental P3 — outbound policy per doc 16) |

### 4.5 Starter data dictionary — `SRC-INT`

| field | source path (Arabic header where legacy-known) | type | PII | semantics | target | quality rules | similar-name warning |
|---|---|---|---|---|---|---|---|
| `internal_session_id` | «رقم الجلسة» / session UUID | uuid/text NOT NULL | P1 | **Authoritative session identity**; join anchor for provider title claims | `core.internal_session_map.internal_ref` | DQ-001/002 | ≠ provider `meeting_ulid` |
| `consultant_ref` | consultant id | text | P1→FK | Source's consultant identifier | `core.advisory_session.consultant_id` via crosswalk | DQ-005 | Legacy used the **national ID** as this key — new system uses a surrogate; see OD-14 |
| `consultant_name` | «اسم المستشار» | text | P2 | Display name in source | crosswalk evidence only | — | `SRC-DIR` is authoritative for names, not this field |
| `programme` | programme/initiative code | text | P0 | Programme the session belongs to | `core.programme` dim | DQ-003 | — |
| `service` / `service_category` | service + «فئة الخدمة» | text | P0 | Service and its category (48 observed values) | `core.service` dim | DQ-006 | This is `service_category` — ONE of the three «قطاع» meanings; never labelled «قطاع» bare |
| `window` | «النافذة» — `irshad/istisharat/sharakat` | enum | P0 | Delivery window | `core.advisory_session.window` | DQ-003 | Called `program` in legacy Appendix A metric registry — the registry dimension name is `window` here; doc 11 reconciles |
| `channel` | delivery channel (remote/in-person…) | enum | P0 | Delivery channel | `core.advisory_session.channel` | DQ-003 | — |
| `scheduled_at` | booked date/time | timestamptz | P0 | When the session was booked to happen | `core.advisory_session.scheduled_at` | DQ-007 | ≠ `created_at` («تاريخ الإنشاء»), ≠ `actual_at` |
| `actual_at` | actual/held date | timestamptz NULL | P0 | When it actually happened | `core.advisory_session.actual_at` | DQ-004/007 | ≠ provider `start_time` (compare, don't conflate — drift feeds CAP-D9) |
| `status` | «الحالة» (مكتملة/ملغاة/…) | enum via mapping table | P0 | Operational workflow outcome | `core.session_status_fact` | DQ-003/004 | ≠ provider meeting lifecycle state; ≠ attendance |
| `attendance` | attendance / no-show indicator | enum | P0 | Who attended («لم يحضر المستفيد» family) | `core.attendance_fact` | DQ-003 | ≠ Read.ai `participants[].attended` (provider observation; disagreement → CAP-D3) |
| `cancellation_reason` | reason code/text | text NULL | P2 | Why cancelled | `core.session_status_fact.reason` | — | — |
| `duration_minutes` | recorded duration | int NULL | P0 | Source-recorded session length | `core.advisory_session.duration_recorded_min` | DQ-007 | ≠ provider elapsed (§2.11) |
| `followup_required` / `closure_status` | follow-up + closure | bool/enum | P0 | Follow-up requirement and closure tracking | `core.followup_fact` (feeds CAP-D7) | DQ-003 | closure ≠ session `status` |
| `beneficiary_ref` | beneficiary identifier | text | **P3** if national-id-based | Beneficiary identity boundary — pseudonymized at staging | `core.beneficiary_pseudonym` | — | never joins outward (GREENFIELD §11.2 beneficiary identity boundary) |
| `beneficiary_rating` | «تقييم المستفيد» 1–5 | int NULL | P0 | Beneficiary's post-session rating (logical source `SRC-EVAL`, §7) | `core.beneficiary_rating_fact` | see §7 | ≠ consultant evaluation; ≠ provider `sentiment` |
| `beneficiary_comments` | «ملاحظات المستفيد» | text NULL | P2 | Free-text comments | `core.beneficiary_rating_fact.comments` | — | — |
| `consultant_evaluation` / `consultant_notes` | consultant's evaluation + recommendations | int/text NULL | P2 | Consultant's own session evaluation and notes (logical `SRC-EVAL`) | `core.consultant_evaluation_fact` | see §7 | ≠ beneficiary rating (CAP-D2 compares them — they must remain distinct facts) |
| `created_at` / `updated_at` | «تاريخ الإنشاء» / modified | timestamptz | P0 | Record lifecycle; `updated_at` is the watermark | staging + cursor | — | `created_at` is NOT the session date |

### 4.6 Interim hardening if manual Excel persists [REC]

Until OD-07 lands, uploads may continue as an interim transport — but under this contract's rules, none of the legacy mechanics survive: **header-name mapping** (never positional — the legacy `raw_row[:49]` zip silently corrupted all fields on any column reorder [FACT arch/04 §9]), **append-with-supersede** keyed on `(internal_session_id, row_sha256)` (never `TRUNCATE` — the legacy TRUNCATE wiped the consultant linkage on every upload), authenticated endpoint, per-file manifest + checksum, and full reconciliation report per upload. Errors return errors, not HTTP 200 (I16).

---

## 5. `SRC-DIR` — consultant directory

Legacy reality: **three disagreeing consultant tables** (`v2_consultants` 39 rows, `v2_consultant_scores` 446, `v2_consultant_directory` 894 — the last with no DDL in the repo), plus 19 consultant ids appearing in meetings with no directory row [FACT arch/02 §4]. The new platform has exactly one consultant dimension, fed by exactly one contract.

| Contract field | Value |
|---|---|
| Owner | Monsha'at consultant-management / HR-adjacent system [ASSUME OD-07 — named at resolution]; steward: platform data steward |
| Transport | Weekly full snapshot via the same OD-07 mechanism (small table — snapshot-compare beats deltas) |
| Auth | Same service account family as `SRC-INT` |
| Cursor/watermark | None (full snapshot); `snapshot_sha256` comparison short-circuits unchanged weeks |
| Idempotency key | `(consultant_source_ref, snapshot_id)` |
| Update/delete | **SCD-2**: attribute changes close the current row (`valid_to`) and open a new one; disappearance → `directory_status='inactive'`, never deletion (historical sessions keep their FK) |
| Freshness SLA | Weekly; staleness >14d → WARN observation |
| PII | Name P2; **national ID P3** (crosswalk only, per OD-14); specialty/qualification P0/P1 |
| Quality rules | `DQ-DIR-001` duplicate source ref in one snapshot (BLOCK) · `DQ-DIR-002` consultant referenced by sessions but absent from directory (WARN → unmatched-identity queue Q3, §10.1) · `DQ-DIR-003` name normalisation collision — two refs, same normalised Arabic name (WARN; blocks fuzzy matching for that name) |
| DLQ / reconciliation | §1.5/§1.6; report adds: active/inactive counts, new/closed SCD rows, count of session-referenced-but-missing consultants (KPI input) |

Starter dictionary:

| field | type | PII | semantics | target | similar-name warning |
|---|---|---|---|---|---|
| `consultant_source_ref` | text NOT NULL | P1/**P3** if national id | Source identity — crosswalked to surrogate `consultant_key` | `core.consultant.source_ref` (restricted if P3) | legacy PK **was** the 10-digit national ID; new PK is surrogate [ASSUME OD-14] |
| `full_name_ar` | text NOT NULL | P2 | Arabic full name | `core.consultant.full_name_ar` | fuzzy matching against `SRC-INT.consultant_name` uses the legacy-proven normaliser (stop-words بن/ابن/ال/آل/بنت/عبد) but only as **resolver evidence**, never silent writes [FACT arch/04 §9] |
| `specialty`, `qualifications` | text NULL | P0 | Domain expertise (feeds CAP-D5 consultant 360) | `core.consultant.specialty` | — |
| `engagement_status` | enum | P0 | active/inactive/suspended | `core.consultant.status` (SCD-2) | ≠ session attendance |
| `joined_at` / `left_at` | date NULL | P0 | Tenure window — CAP-D5 rate denominators need it | `core.consultant.valid_from/to` | — |

---

## 6. `SRC-REF` — programme / service / window / channel reference data

| Contract field | Value |
|---|---|
| Owner | Monsha'at programme office [ASSUME OD-07]; steward approves mapping changes (ADR-0012 workflow) |
| Transport | Monthly full snapshot (tiny tables) via OD-07 mechanism; manual steward-entered bootstrap allowed at phase 0 with the same versioning |
| Cursor/watermark | None; `snapshot_sha256` compare |
| Idempotency key | `(ref_domain, code, snapshot_id)` — domains: `programme`, `service`, `service_category`, `window`, `channel` |
| Update/delete | Versioned reference rows (`valid_from/valid_to`); **codes are never re-used or re-labelled in place** — a renamed service is a new row linked by lineage edge (mirrors taxonomy governance, `tax` schema pattern) |
| Freshness SLA | Monthly; unknown code arriving via `SRC-INT` (`DQ-INT-003`) is the real-time drift detector |
| PII | P0 throughout |
| Quality rules | `DQ-REF-001` duplicate code per domain (BLOCK) · `DQ-REF-002` code referenced by sessions missing from snapshot (WARN) · `DQ-REF-003` Arabic label changed for an existing code (WARN → steward review; labels feed user-facing answers) |
| Reconciliation | Report lists added/closed/re-labelled codes per domain |

Baseline content [FACT CORE-BRIEF §11; MASTER_PROMPT App. A]: `window ∈ {irshad — إرشاد, istisharat — استشارات, sharakat — شراكات}`; `service_category` — 48 observed distinct values (77.3% session coverage). Dictionary is four columns (`code`, `label_ar`, `parent_code?`, `valid_from/to`) per domain — omitted as rows here; the domains table above is the contract.

---

## 7. `SRC-EVAL` — rating and evaluation sources

Physically, ratings/evaluations may arrive **inside** the `SRC-INT` extract (they did in the legacy services report). Contractually they are a separate source because their semantics, lineage, and consumers differ — CAP-D1 (rating vs transcript alignment) and CAP-D2 (consultant vs beneficiary evaluation) require the two evaluation streams to be independently versioned facts, and I18/E.0 method rules forbid quietly averaging across sources.

| Contract field | Value |
|---|---|
| Owner | Same operational system as `SRC-INT` [ASSUME OD-07]; product owner owns the *interpretation* (what a 1–5 means per instrument version) |
| Transport / auth / cursor / DLQ | Inherited from `SRC-INT` (§4.1) when co-delivered; contract split is logical |
| Idempotency key | `(internal_session_id, instrument ∈ {beneficiary_rating, consultant_evaluation}, submitted_at, row_sha256)` |
| Update/delete | A revised rating is a **new fact version** (prior retained with `superseded_at`); deletion never propagates — a withdrawn rating becomes `withdrawn=true` with provenance |
| Freshness SLA | With the `SRC-INT` feed (T+24h); ratings characteristically **lag** sessions — the dictionary marks `submitted_at` as the lag measure, and CAP-D1 coverage blocks must state the lag distribution (I8) |
| PII | Scores P0; comments/notes P2; rater identity P1 (beneficiary side pseudonymized) |

Quality rules: `DQ-EVAL-001` rating outside 1–5 (BLOCK) · `DQ-EVAL-002` rating for a session id unknown to `SRC-INT` (WARN → queue Q2) · `DQ-EVAL-003` multiple non-superseding ratings, same instrument+session (WARN) · `DQ-EVAL-004` instrument version missing (WARN — scale changes must be explicit; a 1–5 → 1–10 change without a version bump would silently poison trends).

**Baseline coverage reality the contract must carry** [FACT CORE-BRIEF §11; MASTER_PROMPT App. A]: legacy report rows with a rating ≈ 24,171 of 94,963; the report↔meeting bridge matched ~8% of report rows and under half of meetings. Every capability consuming ratings declares this as a named coverage exclusion until the join-rate KPIs (§10.2) improve it. **Best label, worst coverage** — the rating is the validation label for impact analysis on the subset where it exists, never silently extrapolated [DECISION MASTER_PROMPT App. E].

Starter dictionary (delta over §4.5 rows):

| field | type | PII | semantics | target | similar-name warning |
|---|---|---|---|---|---|
| `instrument` | enum NOT NULL | P0 | `beneficiary_rating` \| `consultant_evaluation` | fact table discriminator | the two instruments are NEVER averaged together |
| `instrument_version` | text NOT NULL | P0 | Questionnaire/scale version | fact provenance | DQ-EVAL-004 |
| `score` | int 1–5 | P0 | The rating value | `core.beneficiary_rating_fact.score` / `core.consultant_evaluation_fact.score` | ≠ provider `read_score`/`sentiment` (P0 source features, §2.11); ≠ NIP satisfaction signals (CAP-A3 transcript-derived) |
| `comments_text` | text NULL | P2 | «ملاحظات المستفيد» / consultant notes | fact `.comments` | untrusted text under R15 when it enters any prompt |
| `submitted_at` | timestamptz | P0 | When the evaluation was submitted (lag measure) | fact `.submitted_at` | ≠ session `actual_at` |

---

## 8. `SRC-OUT` — optional outcome / follow-up sources

GREENFIELD §2.2 lists "any available outcome/action tracking" and §8.1 requires the adapter even though availability is unconfirmed. The contract is defined now; **activation is gated on OD-07 scope** [ASSUME OD-07 — safe assumption: not available at phase 0; CAP-D7 launches on `SRC-INT.followup_required/closure_status` alone and upgrades when this source activates].

| Contract field | Value |
|---|---|
| Owner | Whichever Monsha'at system tracks post-session actions/bookings [ASSUME OD-07] |
| Transport / auth | OD-07 mechanism family |
| Cursor/watermark | `updated_at` + 48h lookback (as §4.1) |
| Idempotency key | `(outcome_ref, updated_at, row_sha256)` |
| Update/delete | Versioned facts; completion-state transitions append |
| Freshness SLA | T+72h [REC — outcomes are slow-moving; alternatives: daily; revisit when the source is named] |
| PII | Outcome codes P0; linked beneficiary refs pseudonymized P1; free-text P2 |
| Quality rules | `DQ-OUT-001` outcome referencing unknown session (WARN → Q2) · `DQ-OUT-002` completion before session `actual_at` (WARN) |
| Reconciliation | Adds: outcome→session join rate; follow-up bookings matched («حجز جلسة أخرى» is checkable behavioural evidence — prefer it over text inference where present [DECISION MASTER_PROMPT App. E]) |

Starter dictionary: `outcome_ref` (P1, key), `internal_session_id` (P1, join), `outcome_type` (enum: `followup_booked`, `action_completed`, `referral`, `escalation` — seed, not closed vocabulary per R-P1), `outcome_status` (enum), `occurred_at` (P0), `details_text` (P2). Target: `core.outcome_fact` feeding CAP-D7/CAP-D8.

---

## 9. `SRC-SNAP` — one-time legacy snapshot import

Full mechanics (cutover sequencing, reconciliation totals, fixture extraction) are owned by doc 20; this section is the *source contract* view [DECISION GREENFIELD §19.1; MASTER_PROMPT §13.8; ADR-0016].

| Contract field | Value |
|---|---|
| Owner | Platform team executes; legacy system owner approves the dump scope; product owner approves the cutover snapshot |
| Transport | `pg_dump --format=custom` of named tables from the legacy DB, produced **outside** NIP; file + SHA-256 manifest delivered to object storage; `pg_restore` into schema `legacy_snapshot` inside NIP's own database. **No network path between the systems, ever** (I10) |
| Auth | None from NIP's side — NIP never holds legacy credentials; the dump is produced by legacy ops |
| Cursor/watermark | None. Each snapshot is a fixed, versioned artifact `snap_YYYYMMDD_nn`. During the build, periodic re-snapshots refresh the corpus [FACT MASTER_PROMPT §13.8]; exactly **one** becomes the canonical bootstrap at cutover, recorded in `jobs.corpus_snapshot` |
| Idempotency key | `(snapshot_id, table_name, source_pk)`; restore is idempotent — re-running a restore of the same artifact yields byte-identical `legacy_snapshot.*` (verified by table checksums) |
| Update/delete | Immutable. A corrected snapshot is a **new** artifact; `legacy_snapshot` is read-only after restore (role-enforced) |
| Freshness SLA | Not applicable — frozen input is the point (four properties: frozen input, re-runnable migration, free CI fixture, reshaping as SELECT [FACT MASTER_PROMPT §13.8]) |
| Retention | Snapshot file + checksum retained as long as any published number derives from it [DECISION MASTER_PROMPT §13.8; window per OD-08] |

**Scope — raw layer only** [DECISION MASTER_PROMPT §13.8]:

| Included (raw/authoritative) | Excluded (and why) |
|---|---|
| `v2_meetings` (identity/date/gate columns; ASR-correction flags ignored downstream) | All derived knowledge tables (~409k rows: violations, challenges, satisfaction/pressure signals, decision points, government mentions, quality scores…) — **re-derived** under the new open taxonomy, never migrated (fixed-vocabulary classifications are not evidence) |
| `v2_transcript_turns` — **raw `text` only; `text_corrected` is dropped in migration** (ASR correction is cancelled; GREENFIELD §2.6) | `v2_transcript_edits` (158,949 rows — audit trail of a cancelled experiment) |
| `v2_meeting_raw` (raw Read.ai JSON payloads — the true raw layer; where present, turns re-derive from these) | Conversation history, query logs, feedback (legacy serving artifacts) |
| participants, `v2_metrics` (provider features) | `v2_services_report_bak_20260610` — **explicitly excluded**: 76,111-row PII-bearing orphan backup; its deletion is recommended to legacy owners as a compliance action [FACT arch/02 §9] |
| `v2_services_report` (94,963 rows, incl. rating columns) | Portal users/sessions (new IdP, new accounts) |
| The three consultant tables (`v2_consultants`, `v2_consultant_scores`, `v2_consultant_directory`) — collapsed into ONE dimension in migration SQL | `v2_oauth_state` — **credentials never migrate** (OD-13: shared tokens break both systems) |

PII: the snapshot carries P3 (national IDs in era-1 titles, services-report beneficiary columns, directory ids). Handling: `legacy_snapshot` gets the most restrictive grants in the database; the migration pseudonymizes into `core` per §1.4, and P3 columns that have no crosswalk purpose are **not selected** into `core` at all [REC].

Quality rules: `DQ-SNAP-001` table checksum mismatch vs manifest (BLOCK restore) · `DQ-SNAP-002` row-count drift vs manifest (BLOCK) · `DQ-SNAP-003` migration reconciliation totals off (per-table `legacy_snapshot` count vs `core` count + named-exclusion counts must balance exactly — doc 20 owns the totals sheet) · `DQ-SNAP-004` sampled quote-verification failure — findings re-derived from snapshot transcripts must quote-verify against those transcripts (GREENFIELD §19.1).

Reconciliation report: manifest vs restored checksums per table; migration balance sheet (in = out + named exclusions, zero unexplained); duplicate-collapse counts (e.g. the 3,103 duplicate violation rows are *not* migrated but their collapse is documented); fixture-slice extraction record (~500 meetings → CI fixture).

---

## 10. Source-reconciliation model

Reconciliation is a first-class product surface, not plumbing: its outputs feed CAP-D9 (`data_quality_disagreement` — جودة البيانات وتعارض المصادر), which is a **user-facing capability** [DECISION GREENFIELD §5-D].

### 10.1 Unmatched-identity queues

All queues are views over `core` resolution state (nullable FK + `resolution_status` + provenance — never magic strings):

| Queue | Population | Resolution paths |
|---|---|---|
| **Q1 `unmatched_provider`** | Provider meeting whose title claims match no `SRC-INT` session (legacy equivalent: the 1,629 `'PENDING'` rows, 9.6% of corpus [FACT arch/04 §9]) | Automatic retry on each new `SRC-INT` delivery; steward manual link; steward `not_an_advisory_session` disposition |
| **Q2 `unmatched_internal`** | `SRC-INT` completed session with no provider meeting after SLA+72h (transcript never arrived) | Sweep re-check; provider-side search; disposition `no_recording_expected` (e.g. channel without transcription) |
| **Q3 `unmatched_reference`** | Session referencing a consultant/service/programme code absent from `SRC-DIR`/`SRC-REF` (legacy: 19 orphan consultant ids) | Directory/reference refresh; steward mapping |
| **Q4 `ambiguous`** | Multiple candidate matches (e.g. both era-2 title UUID halves matching different internal sessions — the case legacy could not even detect [FACT arch/04 §9]) | Steward adjudication only; auto-match forbidden |
| **Q5 `attribute_conflict`** | Matched pair with disagreeing facts (attendance provider-vs-internal; date drift > 24h; duration drift > 50%) | Not blocking — becomes `ops.data_quality_observation` rows; CAP-D3/CAP-D9 analyse them; steward can flag source bugs upstream |

Queue mechanics: every queue row carries `first_seen`, `evidence jsonb` (the claims and candidates), `age`, `disposition`, `disposed_by`. Queues are steward UI surfaces (doc 18) with SLAs: Q1/Q2 triage within 7 days; Q4 within 3 days [REC]. Dispositions are append-only (ADR-0012 review doctrine).

### 10.2 Join-rate KPIs (EXP-01)

EXP-01 (first experiment in the GREENFIELD §21 order) measures identity-join quality and sets the improvement loop. Definitions (all per monthly cohort of session `actual_at`, end-exclusive periods):

```
provider_match_rate   = matched provider meetings / provider meetings in cohort (two_party_attended scope)
internal_match_rate   = internal completed sessions with matched transcript / internal completed sessions
rating_link_rate      = beneficiary ratings linked to a matched session / ratings in cohort
consultant_link_rate  = sessions with resolved consultant_key / sessions in cohort
ambiguity_rate        = Q4 entries / provider meetings in cohort
```

Baselines to beat [FACT CORE-BRIEF §11; arch/02 §4]: legacy bridge matched ~8% of report rows, under half of meetings overall, ~36% of meetings consultant-resolved via the bridge, 9.6% stuck `PENDING`. Targets [REC]: for cohorts **after** go-live watermark — `provider_match_rate ≥ 0.95`, `consultant_link_rate ≥ 0.98`, `ambiguity_rate ≤ 0.5%` within 2 monthly cycles; historical cohorts — best-effort with published per-cohort coverage (no silent extrapolation; I8). Alternatives considered: a single global join-rate KPI — rejected: monthly volume swings 2.7× and the title-era boundary makes cohort-level the only honest grain. Revisit trigger: if OD-07 delivers a feed carrying the provider meeting id directly, the whole title-claims crosswalk collapses to a trivial join and targets move to ≥0.99.

Every reconciliation report (§1.6) publishes the cohort KPIs it moved; EXP-01's experiment write-up (doc 21) owns the acceptance evaluation.

### 10.3 Data-quality observation feed → CAP-D9

```sql
CREATE TABLE ops.data_quality_observation (
  observation_id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  source_id    text NOT NULL,
  rule_id      text NOT NULL,              -- 'DQ-READAI-002', 'DQ-INT-004', …
  severity     text NOT NULL CHECK (severity IN ('INFO','WARN','BLOCK')),
  entity_kind  text NOT NULL,              -- 'provider_meeting','internal_session','consultant',…
  entity_ref   text NOT NULL,
  observed_at  timestamptz NOT NULL,
  run_id       bigint,
  details      jsonb NOT NULL,             -- rule-specific evidence (values, drifts, candidates)
  status       text NOT NULL DEFAULT 'open'
      CHECK (status IN ('open','acknowledged','resolved','suppressed')),
  suppressed_reason text                   -- mandatory when suppressed; append-only history
);
```

Contract: **every** WARN/BLOCK from every quality rule in §§2–9, every Q5 conflict, and every SLA breach writes exactly one observation row (idempotent on `(source_id, rule_id, entity_ref, observed_at::date)`). CAP-D9 is a Lane-0 capability over this table plus the queues: rates per 100 sessions by source/rule/month, disagreement drill-downs with evidence, and trend vs previous month — same envelope, coverage, and n≥30/Wilson-CI discipline as every other capability (CORE-BRIEF §13.4). This closes the loop the legacy system never had: data-quality failures were invisible (silent skips, 200-on-failure, uncounted window skips [FACT arch/05, CORE-BRIEF §12]); here they are queryable product data.

### 10.4 Cross-source freshness & PII summary

| Source | Cadence | Freshness SLA | Highest PII in feed | P3 stops at |
|---|---|---|---|---|
| SRC-READAI | hourly + monthly sweep | T+24h [ASSUME OD-06] | P3 (era-1 titles, emails) | staging/crosswalk |
| SRC-TSP | provider-native | T+6h target [REC] | P2/P3 | staging/crosswalk |
| SRC-INT | daily | T+24h [ASSUME OD-07] | P3 (negotiate exclusion) | staging |
| SRC-DIR | weekly | 14d | P3 (source ref) | restricted column [OD-14] |
| SRC-REF | monthly | drift-detected | P0 | — |
| SRC-EVAL | with SRC-INT | T+24h | P2 | — |
| SRC-OUT | daily when active | T+72h [REC] | P2 | — |
| SRC-SNAP | one-time (+ build re-snapshots) | n/a | P3 | `legacy_snapshot` grants |

### 10.5 Revisit triggers for this contract book

| Trigger | Action |
|---|---|
| OD-07 resolved (mechanism + SLA + field inventory) | Finalise §4–§8 transports, enum mappings, and the P3-exclusion negotiation; re-issue dictionaries at contract_version 1.0 |
| OD-13 second Read.ai OAuth client granted | Activate §2.3; record credential_ref; rehearse token rotation in staging |
| Third Read.ai title era detected (`DQ-READAI-001` rise) | Amend §2.7 era table; new claims parser version |
| Replacement provider shortlisted (OD-06) | Instantiate §3 against the real API; run doc 06 benchmark + rebase rehearsal |
| OD-14 decided | Set national-ID handling (retain-restricted vs hash-on-ingest) across §2.7, §5 |
| Join-rate targets met 3 consecutive cohorts | Downgrade Q1/Q2 steward SLA; consider auto-disposition rules |

---

*End of document 05. The ingestion pipelines that execute these contracts, including the identity resolver and rebase mechanics, are specified in doc 09; the canonical target model in doc 08; provider benchmarking and the `TranscriptSource` lifecycle in doc 06; snapshot migration execution in doc 20.*

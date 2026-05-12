# Technical Implementation Plans — Part B

Each spec below lists: components affected, new data requirements, API changes (Method → Endpoint → Purpose → Request → Response), frontend state changes, and third-party needs.

---

## Spec 1 — Tatkal Virtual Queue & Booking State Tracker

Components affected:
- API Gateway (queue gateway)
- Booking Service
- Queue Service (Redis Streams / Kafka)
- Notification/Event Stream (SSE/WS)
- Payment Service (no change, but handshake)

New data requirements:
- Table/collection `booking_attempts`:
  - `booking_attempt_id` (UUID)
  - `client_request_id` (string)
  - `user_id` (nullable)
  - `train_id`, `class`, `pax_count`
  - `queue_token` (string)
  - `position` (int)
  - `status` (enum: queued, processing, seat_locked, payment_pending, confirmed, failed)
  - `hold_id` (nullable)
  - `created_at`, `updated_at`, `expires_at`

API changes (examples):
1) POST → `/api/v1/tatkal/enter-queue`
- Purpose: create booking_attempt and assign queue_token
- Request: { client_request_id, user_id?, train_id, class, pax, preferences }
- Response: { booking_attempt_id, queue_token, position_estimate, eta_seconds }

2) GET → `/api/v1/tatkal/status?booking_attempt_id=...`
- Purpose: poll booking status
- Response: { booking_attempt_id, position, status, eta_seconds, last_updated }

3) POST → `/api/v1/tatkal/confirm` (when position=1)
- Purpose: lock seats and move to payment
- Request: { booking_attempt_id, hold_confirmation }
- Response: { hold_id, hold_expires_at, payment_url }

Frontend state changes:
- Add `TatkalQueueContext` (or Redux slice) with fields: `booking_attempt_id`, `queue_token`, `position`, `status`, `eta`, `hold_id`.
- Subscribe to SSE/WS endpoint `/stream/tatkal?booking_attempt_id=...` for live updates; fallback to polling every 2s.

Third-party services / infra:
- Redis Streams or Kafka for ordered queueing and scalable consumer groups.
- Optional: Global rate-limiter service and circuit-breaker (Envoy/Hystrix-like).

Edge behaviours:
- Deduplication by `client_request_id` to avoid duplicate booking_attempts.
- If gateway unavailable, fall back to best-effort POST to booking endpoint with client guidance and clear failure states.

Estimated effort: 3–5 engineer-weeks (PoC + load test)

Success metrics:
- Tatkal booking completion rate during peak increases X → Y (baseline to be measured in spike tests)
- Reduction in duplicate booking attempts

---

## Spec 2 — Deterministic Search Filters with State Persistence

Components affected:
- Search Service / API
- Frontend (filter UI + router)
- API Gateway (query param handling)
- Cache layer (Redis CDN)

New data requirements:
- No new user tables; add `filter_schema_version` and per-request `availability_timestamp` in search responses.

API changes:
1) GET → `/api/v1/search?src=&dst=&date=&filters=...&v=2`
- Purpose: return results with `availability_timestamp` and `filter_metadata`
- Response (excerpt):
  - results: [{ train_id, depart, arrive, classes: [{class, availability, price, availability_timestamp}], result_generated_at }]
  - filter_metadata: {applied_filters, server_filter_schema_version}

2) POST → `/api/v1/search/persist` (optional)
- Purpose: create a `share_token` for current filter set
- Request: { filters }
- Response: { share_token, expires_at }

Frontend changes:
- Encode filter state to URL query params on change using `history.pushState`.
- Mirror URL state to `sessionStorage` for session restore across reloads.
- Show a "stale" badge when `availability_timestamp` older than configured threshold (e.g., 15s).

Migration notes:
- Version filter schema and support `v=1` for legacy clients; gateway rewrites to new schema where possible.

Estimated effort: 1–2 engineer-weeks

---

## Spec 3 — Seat Lock Persistence Across Booking Steps

Components affected:
- Seat Hold Service (new small service or booking service feature)
- Booking DB updates
- Frontend hold management

New data requirements:
- `seat_holds` table
  - `hold_id` (UUID)
  - `booking_attempt_id`
  - `seat_id` (string)
  - `expires_at`
  - `status` (active, released, expired)

API changes:
1) POST → `/api/v1/holds` (create hold)
- Request: { booking_attempt_id, seat_id, client_request_id }
- Response: { hold_id, expires_at }

2) GET → `/api/v1/holds/{hold_id}`
- Response: { hold_id, seat_id, expires_at, status }

3) POST → `/api/v1/holds/{hold_id}/renew`
- Request: { hold_id }
- Response: { new_expires_at }

Frontend changes:
- Persist `hold_id` in booking state and include it in subsequent booking steps.
- Show countdown and an explicit "hold expired" modal with options to reselect.

Consistency model:
- Use optimistic UI but verify with server on every transition; rollback if mismatch.

Estimated effort: 1–3 engineer-weeks

---

## Spec 4 — Resilient Search API with Timeout Recovery

Components affected:
- Search backend + cache layer
- Proxy/Gateway (circuit breaker)
- Frontend partial-render and retry UI

New data requirements:
- Cache keys for route-date-popularity combos with TTLs and `stale` markers.

API changes:
- Search endpoint adds headers/fields: `X-Source: cache|live`, `result_generated_at`, `stale:true|false`.
- Add `GET /api/v1/search/retry` to trigger a forced live refresh (rate-limited).

Frontend changes:
- If backend times out, render cached results with a visible banner and `Retry` CTA.
- Implement exponential backoff retry logic for client-initiated refreshes.

Infra:
- Redis as read-through cache (hot keys), fallback to origin with circuit-breaker via API Gateway.

Estimated effort: 2–4 engineer-weeks

---

## Spec 5 — Progressive Authentication for Discovery Flow

Components affected:
- Auth service (intent token store)
- Frontend routing and session handling

New data requirements:
- `intent_tokens` table: { intent_token, user_context_blob, expires_at }

API changes:
1) POST → `/api/v1/auth/intent` (create intent token)
- Request: { context: { route, date, selected_train, filters } }
- Response: { intent_token, expires_at }

2) POST → `/api/v1/auth/complete?intent_token=...`
- Purpose: link auth result to intent and resume flow

Frontend changes:
- On auth-required action, create `intent_token` and open auth modal/redirect.
- On successful auth, call resume API and navigate to deep-linked state.

Estimated effort: 1–2 engineer-weeks

---

## Spec 6 — Mobile Information Architecture Refresh

Components affected:
- Frontend components, CSS tokens, accessibility checks

Implementation notes:
- Rewrite result card components with prioritized blocks and larger tap targets.
- Add sticky summary and minimize DOM for performance.
- Run accessibility and performance audits on low-end Android (3G/2G emulation).

Estimated effort: 1–2 engineer-weeks

---

## Cross-cutting concerns
- Monitoring and observability: add metrics (queue length, hold expiry rate, search latency) and dashboards.
- Rollout strategy: feature flags to enable gradual rollout (percentage-based).
- Testing: integration tests for idempotency, load tests for queue gateway, user-acceptance tests for UX flows.

End of technical plans.

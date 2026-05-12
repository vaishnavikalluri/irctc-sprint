# Peer Review Notes — Part B (Top 2 specs presented)

Date: (add session date)
Attendees: PM, Backend Eng, Frontend Eng, Designer, Reviewer

## Presented Specs
1) Tatkal Virtual Queue and Booking State Tracker (Spec 1)
2) Deterministic Search Filters with State Persistence (Spec 2)

---

## Questions from Reviewers (Tatkal Queue)
- What happens if the queue gateway becomes a single point of failure?
- How do you handle idempotency when users reconnect with different network conditions?
- What are the SLOs for queue position update latency and hold duration?

## Actionable Updates (Tatkal Queue) — 3 meaningful updates
1. Add autoscaling and multi-zone queue gateway with Redis Streams partitioning to avoid single point of failure.
2. Define idempotency contract: client stores `client_request_id` and server returns canonical `booking_attempt_id` to dedupe retries.
3. Specify SLOs: queue update P95 <= 2s, hold TTL 90s with server-side renew option for active payment flows.

---

## Questions from Reviewers (Search Filters)
- How will legacy endpoints be migrated to the new filter schema without breaking clients?
- What happens when cached results become stale and conflict with live availability?
- Can we support deep links for shared filter views?

## Actionable Updates (Search Filters) — 3 meaningful updates
1. Introduce a versioned filter schema and compatibility layer in API gateway to support old clients during rollout.
2. Add per-result `availability_timestamp` and surface "stale" badge when cached results are older than 15s; provide "Refresh" CTA.
3. Persist filters in URL and support a short `share_token` to allow deep linking/sharing of current filter set.

---

## Review Outcome
- Team approved prototype scope for Tatkal queue (proof-of-concept) and requested a spike (2 weeks) for load testing.
- Search filters work prioritized as immediate quick-win for next sprint.
- Reviewer requested updated spec docs to include concrete API shapes and DB fields (see technical-plans.md).

End of peer-review notes.

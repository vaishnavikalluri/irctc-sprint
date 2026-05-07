# IRCTC Part B - Prioritization Matrix (Impact vs Effort)

## 2x2 Matrix Definition
- High Impact: affects large user base, booking success, or revenue-critical flow.
- Low Impact: localized UX or narrower segment impact.
- High Effort: major backend/platform change or multi-team dependency.
- Low Effort: mostly product/front-end and scoped backend changes.

---

## High Impact, Low Effort (Do First)

1. Forced Login Interrupts Search Flow
- Impact: high conversion recovery in discovery-to-booking funnel.
- Effort: moderate; mostly auth gating and context restore.

2. Search Filter Reliability and State Persistence
- Impact: improves trust and decision speed for all search users.
- Effort: moderate; schema alignment + state persistence.

---

## High Impact, High Effort (Strategic Bets)

1. Tatkal Virtual Queue + Booking State Tracker
- Impact: protects highest stress and highest visibility use case.
- Effort: high; requires queue infra, status orchestration, idempotency.

2. Resilient Search API for 504 Timeout Recovery
- Impact: reduces hard failures during heavy demand windows.
- Effort: high; backend caching, circuit breaker, timeout architecture.

3. Seat Lock Persistence Across Booking Steps
- Impact: directly affects booking confidence and completion.
- Effort: high; transactional seat hold, cross-step consistency.

---

## Low Impact, Low Effort (Quick Wins)

1. Mobile Information Hierarchy Refresh (Phase 1)
- Impact: moderate usability improvement, especially on small screens.
- Effort: low to moderate for initial redesign and component polish.

---

## Low Impact, High Effort (Avoid in Early Sprints)

No item currently placed here for the initial roadmap.

---

## Suggested Rollout Sequence

1. Search filters + progressive authentication (quick conversion wins)
2. Mobile hierarchy phase 1
3. Search timeout resilience
4. Seat lock persistence
5. Tatkal virtual queue and live booking state


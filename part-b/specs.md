# IRCTC Part B - Feature Specs

This document converts the 6 Part A pain points into implementable feature specs.

---

## Spec 1 - Tatkal Virtual Queue and Booking State Tracker

### Problem linked
Tatkal booking crash at 10:00 AM.

### User story
As a Tatkal user, I want to see my real-time queue and booking state so I know whether to wait, retry, or switch options.

### Solution
- Introduce a virtual waiting room before Tatkal submit endpoint.
- Assign short-lived queue token with position and ETA.
- Provide booking state timeline: Queued -> Processing -> Seat Locked -> Payment Pending -> Confirmed/Failed.

### Technical notes
- Add queue gateway in front of booking endpoint.
- Use idempotency key per booking attempt.
- Event stream for status updates (polling fallback).

### Acceptance criteria
1. User sees queue position within 2 seconds of entering Tatkal flow.
2. Repeat clicks do not create duplicate booking attempts.
3. Failed attempts show explicit failure state and retry guidance.

---

## Spec 2 - Deterministic Search Filters with State Persistence

### Problem linked
Search filters not reliable.

### User story
As a train search user, I want filters to stay applied and accurate across refresh/navigation.

### Solution
- Server-backed filtering contract (not only client-side).
- Persist filter state in URL/query params and session store.
- Display filter chips with active criteria and result count.

### Technical notes
- Shared filter schema for API + frontend validation.
- Cache invalidation with timestamped availability refresh.
- Back navigation restores exact filter state.

### Acceptance criteria
1. Filtered results match selected criteria 100% in functional tests.
2. Back navigation restores previous filter state.
3. Availability refresh does not silently drop active filters.

---

## Spec 3 - Seat Lock Persistence Across Booking Steps

### Problem linked
Seat selection resets randomly.

### User story
As a user selecting berth preference, I want my selected seat to remain locked through passenger details and payment steps.

### Solution
- Introduce temporary seat hold with countdown.
- Persist selected berth in server session tied to booking ID.
- Visual confirmation of selected berth at each subsequent step.

### Technical notes
- Optimistic UI with server confirmation rollback.
- Cross-device-safe state via booking token.
- Mobile-safe rendering to avoid local state loss on re-render.

### Acceptance criteria
1. Selected seat remains consistent across step transitions.
2. If hold expires, user gets explicit message and reselection path.
3. Mobile and desktop parity for seat persistence.

---

## Spec 4 - Resilient Search API with Timeout Recovery

### Problem linked
Search failure / 504 timeout.

### User story
As a search user, I want the system to recover from slow backend responses without hard failing my session.

### Solution
- Introduce read-through cache for popular route-date pairs.
- Add graceful timeout fallback (partial results + retry action).
- Circuit breaker for unstable downstream dependency.

### Technical notes
- SLA target for search P95 latency.
- Retry policy with exponential backoff for non-fatal failures.
- User-visible status messages for retry in progress.

### Acceptance criteria
1. Search P95 under target in peak simulation.
2. Timeout responses return actionable UI state, not dead-end error.
3. Repeat retries preserve user query and do not clear form.

---

## Spec 5 - Progressive Authentication for Discovery Flow

### Problem linked
Forced login interrupts search flow.

### User story
As a user exploring options, I want to compare trains before login and authenticate only when required to book/pay.

### Solution
- Allow guest search, comparison, and shortlist.
- Trigger login only at booking commitment step.
- Preserve full context after login redirect/modal completion.

### Technical notes
- Pre-auth intent token stores route/date/selected train.
- Post-auth resume routing with deep-link restore.
- Session-safe modal flow for mobile and desktop.

### Acceptance criteria
1. User can complete discovery without mandatory early login.
2. Post-login resumes at exact prior step.
3. Drop-off rate at auth wall reduces in A/B test.

---

## Spec 6 - Mobile Information Architecture Refresh

### Problem linked
Poor mobile responsiveness and information hierarchy.

### User story
As a mobile user, I want train result cards and booking forms to be readable, tappable, and scannable.

### Solution
- Rebuild mobile card hierarchy (time, availability, fare, quota priority blocks).
- Increase tap targets and spacing.
- Add sticky summary bar for selected train and next action.

### Technical notes
- Mobile-first breakpoints and tokenized spacing.
- Accessibility pass: contrast, font scaling, focus states.
- Performance budget for low-end Android devices.

### Acceptance criteria
1. Critical booking info visible without excessive scroll.
2. Tap error rate drops in usability tests.
3. Accessibility checks pass for text size and contrast baselines.


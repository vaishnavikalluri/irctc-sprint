# Part B Wireframes (Mid-fidelity)

Notes: grey boxes indicate components; annotations show interactions. No colors. Mobile first (375px width). Each wireframe ties to a Part A problem.

---

## 1. Tatkal Queue Screen (Mobile)

HEADER: [IRCTC Logo] [Profile]

SECTION: "TATKAL BOOKING OPENS IN"
┌────────────────────────────┐
│  00 : 05 : 12              │  ← Live countdown
└────────────────────────────┘

SECTION: "YOUR QUEUE POSITION"
┌──────────────────────────────────────────────┐
│  #4,281                                      │
│  Est. wait: ~9 min                           │
│  [███████░░░░░░░░] Progress bar              │
│  Status: Queued → Processing → Seat Locked   │
└──────────────────────────────────────────────┘

DETAILS BLOCK:
- Train: 12622 Tamil Nadu Exp
- Class: SL | Pax: 2
- Passengers: Saved

ACTIONS: [Change Train]  [Edit Passengers]

ANNOTATIONS:
- Queue token created on entering flow (server assigns `queue_token`).
- Live updates via SSE/WS; polling fallback every 2s.
- When position <= 1: show "Proceed to confirm" with 90s timer.

---

## 2. Search Results with Persistent Filters (Desktop + Mobile)

TOP BAR: [Source] [Destination] [Date] [Search]

FILTER BAR (sticky): [Quota chip] [Class chip] [Time range chip] [Clear]

RESULT CARD (repeat):
┌────────────────────────────────────┐
│  Time  | Train # | Fare  | Avail    │
│  [Select]  [Details]                │
└────────────────────────────────────┘

ANNOTATIONS:
- Filters encoded in URL query params: `?src=...&dst=...&date=...&class=SL&quota=TATKAL`
- State synced to session storage for back/forward and modal flows.
- Active filter chips show result count; updates cause shallow route push (`history.pushState`).

---

## 3. Seat Selection + Booking Flow (Mobile)

SEAT MAP SCREEN:
┌──────────────────────────┐
│ Seat map grid (tappable) │
│ Selected: L1 (highlight) │
└──────────────────────────┘
[Confirm Seat]

PASSENGER DETAILS SCREEN:
- Summary sticky bar: Selected seat: L1  •  Hold expires in 04:12
- Form fields for passenger names

ANNOTATIONS:
- On Confirm Seat: POST `/booking/hold` returns `hold_id` and `expires_at`.
- Hold persisted server-side; frontend stores `hold_id` in booking state.
- If hold expires, show explicit modal with reselection option.

---

## 4. Progressive Auth Flow (Modal)

SEARCH → SELECT TRAIN → [Create Shortlist]

If action requires auth: show modal overlay:
┌─────────────────────────┐
│ Login to continue       │
│ [Continue as Guest?]    │
└─────────────────────────┘

ANNOTATIONS:
- Pre-auth intent stored as `intent_token` (route/date/train/selected options).
- After successful login, `intent_token` resumes flow and deep-links user to last screen.

---

## 5. Mobile Information Hierarchy (Result Card)

CARD LAYOUT (top-down):
- Departure time (large)
- Arrival time & duration
- Availability (green/red) and quota tags
- Fare (prominent) and class
- CTA: [Book] sticky

ANNOTATIONS:
- Prioritize tap targets and reduce clutter.
- Sticky summary for selected train during scroll.

---

End of wireframes.

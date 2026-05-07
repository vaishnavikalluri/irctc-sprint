# IRCTC Part B - AI Feature Proposals

These AI features are grounded in the 6 documented Part A pain points.

---

## AI Feature 1 - Smart Tatkal Assistant (Queue + Decision Coach)

### Problems addressed
- Tatkal crash uncertainty
- Session anxiety and repeated retries

### What it does
- Interprets live queue/booking states and explains next best action.
- Provides dynamic alternatives when success probability drops (nearby trains/classes/dates).
- Warns users against duplicate clicks that can worsen failures.

### Inputs
- Queue position, ETA, booking state events
- Route inventory snapshots
- User preference profile

### Output examples
- "Your request is queued at position 12,450. Estimated processing in 2 min 40 sec."
- "Success probability is low for this train. Try 2A in Train X departing 20 min later."

### KPI
- Reduced duplicate retries
- Higher completion during Tatkal peak window

---

## AI Feature 2 - Search Reliability Co-Pilot

### Problems addressed
- 504 timeout during search
- Filter distrust and inconsistent results

### What it does
- Detects unstable search responses and returns confidence-scored result sets.
- Suggests robust fallback queries (date +/- 1 day, nearby stations, alternate classes).
- Explains why a result changed after refresh (availability drift transparency).

### Inputs
- API latency/failure telemetry
- Filter state and query history
- Availability snapshots over short time windows

### Output examples
- "Live data is delayed right now. Showing cached results from 35 sec ago."
- "Try Chennai Egmore -> Lokmanya Tilak with Sleeper + GN for higher success."

### KPI
- Lower search abandonment
- Lower repeated failed query rate

---

## AI Feature 3 - Mobile Booking Clarity Layer

### Problems addressed
- Poor mobile hierarchy
- Forced-login confusion and step loss

### What it does
- Reorders mobile screen blocks based on predicted user intent.
- Highlights top 3 decision signals (departure time, availability, total fare).
- Gives contextual micro-guidance in plain language at each step.

### Inputs
- Device viewport and interaction pattern
- Current booking step and error state
- User profile (new vs returning)

### Output examples
- "Next step: verify quota before proceeding to passenger details."
- "You are about to leave this step. We saved your selected train and date."

### KPI
- Reduced time-to-decision on mobile
- Lower mobile drop-off before payment

---

## Safety and Governance Notes

1. AI suggestions must not fabricate seat availability.
2. Every AI recommendation must include source state timestamp.
3. AI should be advisory; final booking decisions remain deterministic and system-validated.


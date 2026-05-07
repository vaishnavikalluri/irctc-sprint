# IRCTC Part A - Problem Discovery Documentation

This document covers 6 unique IRCTC pain points:
1. 3 given problems (Tatkal crash, filter reliability, seat reset)
2. 3 self-discovered problems from field exploration

Each problem follows the required framework:
- What is broken
- Who is affected
- How often
- Current flow (step-by-step)
- Where exactly it breaks

---

## Problem 1 - Tatkal Booking Crashes at 10:00 AM

### 1) What is broken
At exactly 10:00 AM when Tatkal opens, booking requests spike and the platform becomes unresponsive. Users frequently hit HTTP 502 errors, CAPTCHA/session resets, and dropped sessions between seat confirmation and payment.

### 2) Who is affected
- Primary: Tatkal users in the 9:58 to 10:05 AM window
- Estimated affected base: 20 to 40 lakh high-intent users/day in this window
- Higher impact groups: users with urgent travel, Tier 2/3 users with slower networks

### 3) How often
Daily recurring incident during Tatkal opening. Peak-time failure pattern is highly consistent.

### 4) Current flow (step-by-step)
1. User opens IRCTC before 10:00 AM and logs in.
2. User searches route and chooses Tatkal quota.
3. User enters passenger details and prepares to book.
4. At 10:00 AM user clicks Book Now.
5. UI shows long spinner with no queue/progress state.
6. User receives 502/session timeout/CAPTCHA reset.
7. On retry, quota is gone or waitlist has moved up.

### 5) Where exactly it breaks
Breaks mainly at steps 5 to 7. The system fails under concurrent load and gives no transactional state feedback, causing retries and amplifying load.

---

## Problem 2 - Search Filters Do Not Work Reliably

### 1) What is broken
Filters (quota, class, availability, time) apply inconsistently, show mismatched results, or reset after refresh/navigation.

### 2) Who is affected
- Primary: all users searching trains
- High impact groups: first-time users, senior citizens, users relying on class/availability filters

### 3) How often
Intermittent. Observed and reported as high during traffic peaks; roughly 30 to 40% unreliable behavior in heavy traffic windows.

### 4) Current flow (step-by-step)
1. User enters source, destination, date and runs search.
2. Results list loads with many train options.
3. User applies Sleeper + Available filter.
4. Result list refreshes.
5. Waitlisted or non-matching entries still appear.
6. User opens train detail and finds mismatch vs filter.
7. On back navigation, filters reset and must be reapplied.

### 5) Where exactly it breaks
Breaks at steps 4 to 7. Filter state and result state are not consistently synchronized across refreshes and navigation.

---

## Problem 3 - Seat Selection Resets Randomly

### 1) What is broken
Selected seat/berth does not persist when moving from seat map to passenger details. Users get auto-assigned or different seat than selected.

### 2) Who is affected
- Family travelers
- Elderly/disabled passengers needing lower berth
- Users with strict berth preference

### 3) How often
Intermittent. Approximate failure rate:
- Desktop: around 10 to 15%
- Mobile: around 25 to 35%

### 4) Current flow (step-by-step)
1. User selects train/class/quota.
2. Seat map displays available berths.
3. User selects preferred berth (for example lower berth).
4. User proceeds to passenger details.
5. Preference changes to Auto or different berth.
6. User goes back to reselect.
7. Previously selected berth may now appear unavailable.

### 5) Where exactly it breaks
Breaks at steps 4 to 6 due to state persistence issues between seat map and downstream booking form, especially on mobile re-renders.

---

## Problem 4 (Self-Discovered) - Search Failure / 504 Timeout

### Category
Technical / Backend / Scalability

### Evidence noted during exploration
Browser DevTools network/console showed request timeouts and failed search API calls during route/date search under load.

### 1) What is broken
Train search occasionally fails with gateway timeout (504) or long hangs before fallback failure. Instead of a controlled retry/queue experience, users are left with hard errors.

### 2) Who is affected
- Users searching during high traffic periods
- Mobile data users with unstable connectivity
- Users trying alternate dates quickly (multiple requests in short time)

### 3) How often
Peak-hour dominant issue. Estimated intermittent failure in heavy windows (for example morning/evening spikes), with repeated retries needed in a meaningful share of attempts.

### 4) Current flow (step-by-step)
1. User enters source, destination, and date.
2. User clicks Search Trains.
3. Spinner continues for extended duration.
4. Network call exceeds timeout threshold.
5. UI shows generic failure or no actionable message.
6. User retries with same parameters.
7. Multiple retries either eventually load or continue to fail.

### 5) Where exactly it breaks
Breaks at steps 3 to 5. Search endpoint latency and timeout handling are weak under load, and the UX does not provide graceful degradation.

---

## Problem 5 (Self-Discovered) - Forced Login Interrupts Search Flow

### Category
Product Flow / UX / Conversion Friction

### 1) What is broken
Users are forced into authentication mid-flow before they can complete basic discovery (comparing trains/fares across options), breaking momentum and causing drop-off.

### 2) Who is affected
- New users evaluating options before deciding to book
- Returning users not immediately ready with credentials/OTP
- Casual planners checking routes and dates

### 3) How often
Consistent in guarded flows where deep interaction triggers auth wall. High impact on first-session conversion.

### 4) Current flow (step-by-step)
1. User lands on search page and starts route/date exploration.
2. User opens details on a preferred train.
3. User attempts next intent action (fare/seat/book continuation).
4. Login modal/page appears and blocks progress.
5. Context is partially lost after login (state/reset/back confusion).
6. User repeats earlier steps.
7. Some users abandon due to repeated interruption.

### 5) Where exactly it breaks
Breaks at steps 4 to 6. Authentication gating appears at a high-friction moment and does not preserve exploratory session context reliably.

---

## Problem 6 (Self-Discovered) - Poor Mobile Responsiveness and Information Hierarchy

### Category
UI / Accessibility / Mobile UX

### 1) What is broken
Mobile layout appears dense and outdated: cramped controls, weak visual hierarchy, inconsistent spacing, and low scannability for critical booking information.

### 2) Who is affected
- Mobile-first users (majority in India internet usage context)
- Users on small-screen Android devices
- Users with low digital literacy who need clear visual guidance

### 3) How often
Persistent on mobile screens; worsens under stress moments (search, booking, payment). Usability friction is continuous, not occasional.

### 4) Current flow (step-by-step)
1. User opens IRCTC on mobile browser/app.
2. User enters search details.
3. Results render with dense cards and crowded metadata.
4. User tries to compare class/availability quickly.
5. Important info competes visually (price, status, quota, timing).
6. User scrolls repeatedly and mis-taps controls.
7. Decision time increases; confidence decreases.

### 5) Where exactly it breaks
Breaks at steps 3 to 6. Responsive design and hierarchy are insufficient for fast decision-making on small screens.

---

## Frequency Analysis Summary (Part A)

1. Tatkal crash: daily peak-window critical failure.
2. Filter reliability: intermittent, higher during load.
3. Seat reset: intermittent; notably worse on mobile.
4. Search timeout 504: peak-window backend instability.
5. Forced login interruption: consistent conversion friction in guarded actions.
6. Mobile responsiveness: persistent usability debt across flow.

---

## Notes on Evidence

- Screenshots to be added under assets/screenshots by the owner.
- Problem 4 includes direct runtime evidence from DevTools (timeouts/failed requests), strengthening technical diagnosis.


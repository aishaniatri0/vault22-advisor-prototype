# BOOKING REDESIGN PROMPT — Vault22 Advisor Prototype (Calendly-grade scheduling)

> Paste everything below the line into a fresh Claude Code session on this repo.
> Self-contained. Rebuild the "Book a meeting" surface into a best-in-class scheduling
> experience. Verify by DRIVING THE REAL UI and LOOKING (clicking through a full booking),
> never by reading source.

---

You are a senior product designer + frontend engineer redesigning the meeting-booking surface
in the Vault22 Advisor prototype (`/Users/aishaniatri/Vault22-Advisor-Prototype/index.html`,
single-file vanilla JS, no build). **Read `bookDrawer()` and the meeting flow first.**

## Goal
Make booking a meeting feel like a modern scheduling tool (Calendly / Cal.com / Google
appointment scheduling / SavvyCal), not a list of buttons. The advisor is scheduling a meeting
with a client, so this is the advisor's side of scheduling — apply the same polished patterns.

## What's there now (baseline to replace)
A right-hand drawer with a **vertical day list** (Sat 18 / Sun 19 / …) beside a **time-slot
column**, three static pills (30 min / SAST / video call), a confirm button, and an "Already
booked" list. It works and the meeting loop is wired — but it is flat and missing the signature
scheduling patterns.

## House rules (keep; do not regress)
No em dashes in rendered copy · Rands via `money()` · `vault22.com` never `.io` · UK spelling ·
dates via `localISO()` (and note `Date.now()`/`new Date()` with no args are unavailable in some
contexts — build dates from fixed prototype values as the current code does) · **preserve every
working behaviour**. Seed data stays illustrative.

## MUST PRESERVE (the meeting loop — do not break any of it)
- Advisor picks day+time → **Send invite** opens the **editable email compose** (`data-bconfirm`
  → `composeDrawer`) → **Send** (`data-csend`) creates the meeting (`status:'invited'`) and
  builds the real `.ics` (`buildICS`/`downloadICS`/`icsModal`).
- The two-way loop: client sees the invite under **Your Advisor → Meetings** with **Accept /
  Decline**; accepting flips the advisor's view to **Confirmed** (`data-meetaccept`/`meetdecline`).
- Client → advisor request path and the dashboard "Confirm meeting requests" to-do.
- Keep `state.bookFor` / `state.bookDay` / `state.bookSlot` (or equivalent) and the meeting
  object shape; extend, don't replace, so nothing downstream breaks.

---

# THE REDESIGN — build these patterns

## 1. A real month calendar (the signature move)
Replace the vertical day list with a **month grid**: a 7-column week (Mon–Sun header), the
month's dates laid out, with **‹ / › month navigation** and the month + year shown. Behaviour:
- **Past dates disabled** (muted, not clickable); can't navigate before the current month.
- **Days with availability** get a subtle marker (e.g. a dot); **unavailable days** (seed
  weekends as unavailable, plus a couple of fully-booked days) are muted/disabled.
- **Today** is marked; the **selected date** is a clear filled highlight.
- Seed availability for the next ~6 weeks so navigating a month or two forward still shows slots.

## 2. A polished time-slot column
For the selected date, show the slots in a clean column with:
- A header naming the date ("Saturday, 18 July").
- Slots **grouped by Morning / Afternoon** (and Evening if used), with small group labels.
- **Already-booked / conflicting slots disabled** (greyed, labelled "Booked") — derive conflicts
  from that client's existing meetings so you can't double-book a taken time.
- A **12h / 24h toggle** (display AM/PM vs 24-hour); default to 24h SAST.
- The Calendly micro-interaction is a nice touch: clicking a slot can **split it to reveal a
  "Confirm" button inline** (or use a clear primary Confirm at the bottom) — pick one and do it
  cleanly.

## 3. Meeting controls (top of the surface)
Make the three static pills into **real selectors**:
- **Duration**: 15 / 30 / 60 min (changing it can change slot spacing/labels).
- **Timezone**: a small dropdown (SAST/GMT+2 default, a few others) — display-only is fine.
- **Location / type**: Video call / Phone / In person, each with an icon; the chosen one flows
  into the invite summary and the `.ics` LOCATION.

## 4. A confirmation summary before Send
After day + time are chosen, show a tidy **summary** — e.g. "30-minute Vault22 review with Lerato
Khumalo · Saturday 18 July, 10:00 (SAST) · Video call" — then continue to the existing editable
email compose and Send. The summary should read like a real booking confirmation.

## 5. Keep "Already booked" but integrate it
Keep showing the client's upcoming meetings, but reflect them in the calendar/slots (booked days
marked, booked slots disabled) so the advisor sees conflicts *in context*, not just as a list.

## 6. Polish + responsive + theming
- Generous whitespace, clear hover/selected states, smooth transitions.
- Works in **light, dark, and navy** (the calendar/slots must be legible in all three).
- **390px**: the calendar and slot column **stack** (calendar on top, slots below); no horizontal
  overflow; tap targets stay comfortable.

---

# VALIDATION PROTOCOL (before declaring done — UI only, LOOK/CLICK)
1. Open Book a meeting: a month calendar renders; ‹ › navigates months; past dates are disabled;
   unavailable/booked days are muted.
2. Pick a date → grouped time slots appear; already-booked slots are disabled; 12h/24h toggle
   works; duration/timezone/location selectors work.
3. Pick a slot → a clear summary → Send invite → editable compose → Send → meeting created and
   `.ics` downloads.
4. Full loop still closes: client sees the invite with Accept/Decline; accepting → advisor sees
   **Confirmed**; the client→advisor request path + dashboard to-do still work.
5. Legible and correct in light + dark + navy; stacks cleanly at 390px; no overflow.
6. House rules: zero em dashes / `.io` / `undefined` / `NaN`; zero console/page errors; nothing
   else in the app regressed.

# Definition of done
Booking looks and feels like a modern scheduling tool — a real month calendar, grouped and
conflict-aware time slots, real duration/timezone/location controls, and a clean confirm — while
the entire meeting loop (invite → accept/decline → confirmed → `.ics`) and the client/advisor
request paths keep working. Verify by clicking through a full booking, not by reading source.

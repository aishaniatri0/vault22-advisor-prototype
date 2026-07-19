# FIX PROMPT — Vault22 Advisor Prototype (Start Investing, registration loop, validation, booking)

> Paste everything below the line into a fresh Claude Code session on this repo.
> Self-contained. Each defect is **How it works now → How it must work → Fix → Verify**.
> Verify by DRIVING THE REAL UI and LOOKING (clicking, typing, watching counts change),
> never by reading source or dumping `innerText`.

---

You are a senior frontend engineer fixing the landing-page "Start Investing" flow, the
registration loop, phone validation, and the booking UI in the Vault22 Advisor prototype
(`/Users/aishaniatri/Vault22-Advisor-Prototype/index.html`, single-file vanilla JS, no build).
**Read it first.**

## Context: what "Start Investing" IS (Stephen's intent)
The landing page is the advisor's **PUBLIC page that a PROSPECT visits** via the advisor's
link/QR. **"Start Investing" is the *prospect's* register CTA** — a new person clicks it to
register and start the standard Vault22 investor onboarding, and because they came through this
advisor's link they are **tagged to that advisor** (they become the advisor's Registered User).
Stephen: "Prospect completes standard Vault22 onboarding. **No changes to investor journey**" —
so the actual signup hands off to the existing consumer flow; do not build a full new onboarding.
It is **never the advisor signing up for themselves.**

## House rules (keep; do not regress)
No em dashes in rendered copy · customer money is comma-grouped Rands via `money()` (subscription
fee is the only USD, via `usd()`) · `vault22.com` never `.io` · UK spelling · no internal/dev
references in the UI · dates via `localISO()` · **preserve every working behaviour and closed
loop** (meetings both ways, documents request→fulfil, portfolio submit→approve→live, Tara, firm
Admin/Member, consent, membership PDF, notification/qualification settings pages, three-view
theming + dark mode, CRM tab, analytics period incl. Custom). Seed data stays illustrative.

---

# DEFECT 1 — "Start Investing" is confusing and duplicated

**How it works now:** on the advisor's own Landing Page, "Start Investing" (`data-startinvest`)
opens a form headed *"Create your account. Aishani Atri will be your advisor."* Because the
advisor is viewing their **own** page, it reads as the advisor signing up with themselves — it
only makes sense from a prospect's point of view, and nothing frames it as a preview. Separately,
the **My Profile preview card** has a *different* "Start Investing" that is just a `data-stub`
explaining the journey is unchanged. Two buttons, two behaviours, both confusing.

**How it must work:** the Landing Page must clearly read as **the advisor's public page as a
prospect would see it** — add a short, explicit framing line (e.g. a "Preview — this is what a
prospect sees" label on the public card). "Start Investing" is the **prospect's** action; when
triggered in the demo it should be **honestly framed as a prospect signing up** (e.g. the
register drawer titled "Start investing with {advisor}" is fine, but make clear it is simulating
a prospect, not the advisor). **Unify** the two "Start Investing" buttons to the same behaviour
(both should do Defect 2's real registration, or the preview-card one should mirror it).

**Fix:** add the preview framing to `vLanding()` (and the My Profile preview). Point both
"Start Investing" buttons at the same handler. Keep the copy prospect-oriented ("Create your
account. {advisor} will be your advisor").

**Verify (look/click):** the Landing Page reads as a prospect preview; "Start Investing" from
either place opens the same prospect-signup, clearly framed as a prospect action.

# DEFECT 2 — 🔴 Registration does not create a Registered User (the loop does not close)

**How it works now:** submitting the register form (`data-register`) only `unshift`s a
**notification** and shows a toast. It does **not** add anyone to `state.leads`. Result (verified
by driving): the Dashboard "Registered users" count stays **10 → 10**, "New registered users this
month" does not change, the person **never appears in the Registered Users list**, and the
notification just routes to the Dashboard. So registering *appears to do nothing* — which is the
whole point of the product ("every signup through the link is tagged to the advisor").

**How it must work:** registering a prospect **creates a real Registered User** (spec §6 lifecycle):
- Push a new record into `state.leads` with a fresh id, the entered **name / email / phone**,
  `stage: 'Registered'`, `signup` = today (`localISO(new Date()).slice(0,10)`), `act` = now, and
  whatever fields the Registered Users list/table reads (nulls are fine for ffs/nw/etc. at the
  Registered stage, like the existing "Johan Botha" seed). Tag it to the advisor.
- Re-render so the **Dashboard "Registered users" count goes 10 → 11**, "New registered users
  this month" +1, and **conversion recalculates**; the person **appears in the Registered Users
  list** at "Registered".
- The signup **notification should open the new person's profile** (`go:['customer', newId]`),
  not the Dashboard.
- Keep the confirmation toast.

Apply the same "make it real" treatment to **"Get in touch"** enquiries if they claim to reach
the advisor: the enquiry should land somewhere the advisor actually sees it (a notification that
opens something real, and/or a Dashboard to-do), not a dead toast.

**Fix:** rewrite the `data-register` handler to build and unshift a new lead, tag to advisor,
add a profile-linking notification, then `render()`. Ensure the new lead flows through the same
count/conversion/list logic the seeded leads use.

**Verify (drive, watch the numbers):** note the Dashboard "Registered users" count → register a
prospect → the count increments, "New registered users" +1, the person is in the Registered Users
list at "Registered", and clicking the notification opens **their** profile.

# DEFECT 3 — 🟠 Phone/mobile inputs accept non-numeric input

**How it works now:** every phone field is a plain `<input class="inp">` with `type="text"` and
no restriction — `regPhone` (landing register), `r_phone` (onboarding register), and the Customise
advisor phone (`advisor.phoneFmt`) all accept letters/symbols ("abcXYZ!!!" submits fine). Email
fields *are* validated (`EMAIL_RE`); phone is the gap.

**How it must work:** phone fields accept **numbers only** (plus `+`, spaces, hyphens/parens as
formatting) and reject/strip letters, and validate on submit (a real phone or nothing). Apply to
**every** phone/mobile input in the prototype.

**Fix:** give phone inputs `type="tel" inputmode="numeric"` and filter input to digits and phone
punctuation (e.g. strip `[^\d+\s()-]` on input), and validate on submit with a clear message on
bad input. Audit the whole file for phone/mobile inputs so none is missed. Do not break the
existing email validation.

**Verify (type):** in each phone field, typing letters is rejected/stripped; submitting a
letters-only phone shows a validation message; a valid number works.

# DEFECT 4 — 🟡 Booking UI could be Calendly-grade (enhancement)

**How it works now:** Book a meeting is a row of day chips + a row of time-slot buttons.
**How it must work:** a more polished, **Calendly-inspired** booking surface — a clearer date
picker (a small month/week calendar or a cleaner day list), a tidy **time-slot column**, the
**meeting duration and timezone** shown, and a clear confirm step. **Preserve all existing
behaviour**: the two-way meeting loop (Send invite → client Accept/Decline → advisor sees
Confirmed), the editable-email compose, and the real `.ics` download must keep working exactly.

**Fix:** redesign `bookDrawer()` (and its confirm) for a Calendly-like feel; keep `state.bookDay`
/ `state.bookSlot` / the meeting object / `data-bconfirm` / `data-csend` / ICS logic intact.

**Verify (click):** the booking surface looks polished; picking a day+time, Send invite, client
Accept, advisor Confirmed still all work; the `.ics` still downloads.

---

# VALIDATION PROTOCOL (before declaring done — UI only, LOOK/DRIVE)
1. Landing reads as a prospect preview; "Start Investing" is unified and prospect-framed.
2. Register a prospect → Dashboard "Registered users" count increments, "New registered users"
   +1, person is in the Registered Users list at "Registered", notification opens their profile.
3. Every phone field rejects/strips letters and validates on submit; email validation intact.
4. Booking looks Calendly-grade; the full meeting loop + ICS still work.
5. Regression: all closed loops still close; three theme views + dark clean; analytics period
   (incl. Custom) drives all charts; no 390px overflow; zero em dashes / `.io` / `undefined` /
   `NaN` / retired labels; zero console errors.

# Definition of done
"Start Investing" is clearly the prospect's action on a previewed public page; registering a
prospect actually creates a Registered User that shows up in the count, the list and a
profile-linking notification; phone fields accept only valid phone input everywhere; and the
booking UI is polished without breaking the meeting loop. Verify by driving the UI and watching
the counts/lists change, not by reading source.

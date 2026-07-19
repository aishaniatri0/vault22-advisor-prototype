# FIX PROMPT — Vault22 Advisor Prototype (Product-Owner QA defects)

> Paste everything below the line into a fresh Claude Code session on this repo.
> Self-contained. This is a defect-fix pass to take the prototype from "the loops work"
> to "it behaves like a finished product." Fix behaviour, then verify it the way the bugs
> were found: by real typing and real clicks, never by setting state in code.

---

You are a Senior Frontend Engineer with a Product Owner's eye, fixing defects found in a
manual UI walkthrough of the Vault22 Advisor prototype (`/Users/aishaniatri/Vault22-Advisor-Prototype/index.html`,
single-file vanilla JS, no build). **Read it first.** These defects were found by driving
the real UI (typing with the keyboard, clicking real buttons, viewing at 390px), which is
exactly how you must verify each fix. Automated checks that set values atomically or poke
state will NOT catch these, and are not acceptable as verification.

## House rules (keep them; do not regress)
No em dashes in rendered copy · comma-grouped Rands via `money()` (subscription fee is the
only USD, via `usd()`) · `vault22.com` never `.io` · UK spelling · no internal/dev references
in the UI · dates via `localISO()` · right-hand panels use `.rdrawer`/`body.dw-open`, Tara is
a full-screen `body.onboarding`+`#onbRoot` takeover · **preserve every working behaviour and
every closed advisor↔client loop** (meetings, documents, portfolio approval, Tara suggest/
invest, firm page, consent, risk nudge, membership). Seed data stays clearly illustrative.

---

# DEFECT 1 — 🔴 HIGH: Tara "Total invested" (and any live-typed field) corrupts the value

**Symptom (reproduce first):** open a customer → Personalise portfolio → click the **Total
invested** field, press Ctrl+A, type `750000`. Observed result: the field shows garbage like
`57980000`, not `750000`. The advisor cannot reliably set the amount, on the headline AI moment.

**Root cause:** the `input` handler for `#taraTotal` mutates state and calls a full `render()`
on **every keystroke**, replacing the DOM; the focus/caret restore (`setSelectionRange`)
interleaves digits, so the typed value is mangled. The weight `+/-` steppers re-render too,
but that is a click (no caret to lose) so it is acceptable; the bug is specifically the
text/number **inputs** that re-render per keystroke.

**Required behaviour:** typing into Total invested updates the field naturally (what you type
is what you get), the caret never jumps, and each line's **amount** (and any total) recomputes
live as you type. No full-screen re-render on keystroke.

**How to fix (match the pattern already in the file):** the code already has a no-re-render
convention for live inputs (the theme colour picker writes to a CSS var without rendering;
the `data-bind` inputs deliberately "do NOT re-render"). Apply the same here: on input, update
`state.tara.total`, then recompute and write each line's amount into its own DOM node (by id/
class) and update the running-total pill **in place**, leaving the input element untouched.
Do not call `render()` from the Total-invested `input` handler.

**Sweep the same class of bug:** every field that calls `render()` on `input` — at minimum
`#taraTotal`, `#taraSrch`, `#portSearch`, `#advQ`, the Registered Users `#searchIn`, the
qualification-threshold inputs, and any Customise/firm text field. For each, verify by **real
keyboard typing** (multiple characters, mid-string edits, Ctrl+A replace) that the value is
never corrupted and focus/caret is stable. Convert any that corrupt or drop focus to the
in-place-update pattern (or a correct caret-preserving approach). Search/filter fields should
filter smoothly as you type without losing the cursor.

---

# DEFECT 2 — 🟠 MODERATE: Mobile topbar "Advisor" badge renders as vertical stacked letters

**Symptom:** at 390px width the "Advisor" tag beside the Vault22 logo wraps character-by-
character into a vertical `a/d/v/i/s/o/r` column. Looks broken.

**Required behaviour:** at every width the topbar brand reads cleanly. The badge stays on one
line (`white-space:nowrap` and enough room), or is hidden below a breakpoint if space is tight
— never wraps to vertical letters. Fix the flex/min-width so the logo, badge and right-hand
controls share the row without squeezing any child to near-zero width.

**Verify:** view the advisor app at 390px on Dashboard and a customer profile; the header is
clean, nothing is clipped, and there is no horizontal overflow.

---

# DEFECT 3 — 🟠 MODERATE: Landing page Contact block reads like empty form fields

**Symptom:** on the public Landing Page, the Contact section renders the advisor's email /
phone / website / LinkedIn as faint, underlined table rows that read like *blank input fields*.
The values are present but have almost no visual prominence, on the one page a prospect sees.

**Required behaviour:** the Contact block should look like polished, first-class contact
details a prospect would trust: clear labels with prominent values (consider a small icon per
row, or a clean label-value list with proper contrast), not underlined empty-looking rows.
Consistent with the card's visual language. It must look like something the advisor would be
proud to share.

**Also (product intent):** the landing page is "where new users register or get in touch."
The **Get in touch** button is currently an explanatory stub. Upgrade it to a real (prototype-
wired) mini contact form: name + email + message → on submit, show a believable confirmation
("Thanks, {advisor} will be in touch") and, if it fits the model, drop an enquiry into the
advisor's dashboard to-do or notifications so the acquisition loop closes like the others. Keep
**Start Investing** as the register CTA (the investor journey is deliberately unchanged; an
honest stub is fine there).

**Verify:** open Landing Page; the contact details are clearly legible in light and dark;
Get in touch opens a working form and confirms on submit.

---

# DEFECT 4 — 🟡 MINOR: Toasts linger and bleed across navigation

**Symptom:** confirmation toasts (e.g. "Requested Proof of address from Lerato") stay ~3s and
follow the user across navigation, appearing on unrelated screens (seen on the Tara and
Portfolios screens after a Documents action).

**Required behaviour:** a toast belongs to the action that fired it. Dismiss any visible toast
on navigation / scenario switch / full re-render, and prevent stacking (a new toast replaces or
queues rather than overlapping). Shorten slightly if needed. A toast should never appear over an
unrelated screen.

**Verify:** trigger a toast, immediately navigate; the toast does not appear on the next screen.

---

# DEFECT 5 — 🟡 MINOR: Portfolios table looks sparse

**Symptom:** "Total invested" and "Clients" show `n/a` on roughly 10 of 17 rows, so the flagship
Portfolios screen looks thin in a demo.

**Required behaviour:** seed believable "Total invested" and "Clients" values on more of the
SA-universe rows (keep them clearly illustrative and internally consistent with the book), so
the table reads as a live product catalogue. A few genuinely-empty rows are fine; most should
carry numbers. Keep the `// TODO: replace with Uday's real list` note.

**Verify:** open Portfolios; the majority of rows show real figures; nothing misaligns.

---

# VALIDATION PROTOCOL (do this before declaring done — UI only)

Drive the real UI; do not set state or call internal functions to fake a result.

1. **Defect 1:** in Tara, type into Total invested with the keyboard (fresh value, Ctrl+A
   replace, and a mid-string edit); confirm the value is exactly what you typed, the caret is
   stable, and line amounts recompute live. Repeat the keyboard test on every search/threshold/
   text field you touched.
2. **Defect 2:** resize to 390px; the topbar is clean on Dashboard and a customer profile; no
   vertical text; no horizontal overflow on any advisor route.
3. **Defect 3:** Landing Page contact details are clearly legible (light + dark); Get in touch
   form works and confirms.
4. **Defect 4:** fire a toast then navigate; it does not bleed onto the next screen; no stacking.
5. **Defect 5:** Portfolios reads full and aligned.
6. **Regression (click through, both scenarios):** confirm every closed loop still closes —
   meetings (invite→Accept/Decline→Confirmed; client request→advisor to-do→confirm), documents
   (request→client upload→Received), portfolio submit→Approve→live, Tara suggest→client tweak→
   invest→advisor reflects, firm Admin edit vs Member read-only, risk nudge, consent round-trip,
   membership PDF, notification settings changing the bell, qualification thresholds recomputing.
7. **House rules:** zero em dashes, `.io`, `undefined`/`NaN`/`[object Object]`, retired labels;
   zero console/page errors; clean in Navy + Dark; no 390px overflow.

# Definition of done
Every defect above is fixed and verified by real interaction, no closed loop or working
behaviour has regressed, and the prototype reads as a finished product: you can type into any
field and get exactly what you typed, the storefront looks polished, feedback is tidy, and
nothing looks broken at phone width. When done, note anything still seeded that needs real data
(Uday's securities/portfolio lists; final pricing with Greg).

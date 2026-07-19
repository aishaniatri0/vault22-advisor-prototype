# BUG-FIX PROMPT — Vault22 Advisor Prototype (all outstanding bugs)

> Paste everything below the line into a fresh Claude Code session on this repo.
> Self-contained. Fix the OPEN bugs listed in Part 2. Part 1 is the full ledger for context.
> Verify every fix by DRIVING THE REAL UI and LOOKING at the rendered result (screenshots),
> never by reading source or dumping `innerText` — several of these bugs are invisible to
> `innerText` (clipped-but-present DOM) and only show up visually.

---

You are a senior frontend engineer fixing outstanding bugs in the Vault22 Advisor prototype
(`/Users/aishaniatri/Vault22-Advisor-Prototype/index.html`, single-file vanilla JS, no build).
**Read it first.**

## House rules (keep; do not regress)
No em dashes in rendered copy · comma-grouped Rands via `money()` (subscription fee is the only
USD, via `usd()`) · `vault22.com` never `.io` · UK spelling · no internal/dev references in the
UI · dates via `localISO()` · **preserve every working behaviour and closed loop** (meetings
both ways, documents request→fulfil, portfolio submit→approve→live, Tara suggest→tweak→invest,
firm Admin/Member, consent, membership PDF download, notification settings, qualification
thresholds, three-view theming Green/Mid-navy/Navy, dark mode, the CRM tab). Seed data stays
clearly illustrative.

---

# PART 1 — Full bug ledger (context)

Already fixed in prior rounds (do NOT re-do; just don't regress them):
- Green/Navy/Dark theme toggles were dead → wired + three views + navy skin + Night-Owl dark.
- Tara "Total invested" typing → in-place update, `type=text inputmode=numeric` (clean).
- Mobile top-bar "Advisor" badge wrapped to vertical letters → single line at 390px.
- Landing "Contact" block looked like empty fields → icon + label + value; real Get-in-touch form.
- Toasts bled across navigation → one at a time, cleared on screen change.
- Portfolios list looked sparse → Total invested + Clients seeded on rows.
- Customers list missing "Last risk review" date (R43) → column + Due-soon flag added.
- Customer Audit tab was thin ("viewed profile" ×2) → real client history seeded.
- Analytics line charts stopped mid-axis → `preserveAspectRatio="none"`, span full width.
- Analytics charts ignored the period selector → `periodSeries()` per window; all charts follow.
- Analytics "Suggested additions" heading → folded into "Customer trends".
- CRM block rendered under every profile tab → CRM is now its own tab.
- Analytics "This month" retention showed a fake 100% → seeded a lost customer, now 83.3%.
- Right-hand drawer tables (fund View, invoice) clipped the value column → the global
  `table{min-width:820px}` floored the small `.ptbl` tables at 820px inside a ~460px drawer;
  dropped the floor with `.ptbl{min-width:0}` and let drawer cells wrap
  (`#modal .ptbl td{white-space:normal;overflow-wrap:normal}`) so values show and numbers stay whole.
- "Products owned" amounts were too faint (`--v4-navy-300`) → normal value colour (`--live-field-text`).
- Financial-report "Download PDF" was a dead stub → real one-page PDF via a shared `downloadPdf()`
  (same builder as the invoice), audited on download.
- Registered-user notifications routed to a dead `'lead'` view and landed on the Dashboard →
  treat `'lead'` like `'customer'` (both open the full profile), with the else guarded on
  `VIEWS[kind]` so no unknown kind can ever route to a non-existent view again.

# PART 2 — OPEN bugs

None. Every bug listed in this prompt has been fixed and verified by driving the real UI
(screenshotting drawers, clicking notifications, typing into fields). See the ledger above.

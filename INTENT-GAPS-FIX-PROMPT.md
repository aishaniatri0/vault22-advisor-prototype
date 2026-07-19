# FIX PROMPT — Vault22 Advisor Prototype (requirement gaps vs Stephen's exact words)

> Paste everything below the line into a fresh Claude Code session on this repo.
> Self-contained. Each defect is written as **Requirement (Stephen) → How it works now → How it
> must work → Fix → Verify**. Build to Stephen's intent, and verify by DRIVING THE REAL UI and
> LOOKING (clicking, switching, screenshotting), never by reading source or dumping `innerText`.

---

You are a senior frontend engineer fixing places where the Vault22 Advisor prototype
(`/Users/aishaniatri/Vault22-Advisor-Prototype/index.html`, single-file vanilla JS, no build)
does not match what the founder (Stephen) actually asked for. **Read it first.**

## House rules (keep; do not regress)
No em dashes in rendered copy · customer money is comma-grouped Rands via `money()` (the advisor
subscription fee is the only USD, via `usd()`) · `vault22.com` never `.io` · UK spelling · no
internal/dev references in the UI · dates via `localISO()` · **preserve every working behaviour
and closed loop** (meetings both ways, documents request→fulfil, portfolio submit→approve→live,
Tara suggest→tweak→invest, firm Admin/Member, consent, membership PDF, notification settings,
qualification thresholds, three-view theming Green/Mid-navy/Navy, dark mode, CRM tab). Seed data
stays clearly illustrative.

---

# DEFECT 1 — 🔴 Analytics "Custom" period is a stub (no date range)

**Requirement (Stephen):** "add a period selector at the top right – this month, last month,
last 90 days, YTD and **custom**."
**How it works now:** clicking **Custom** silently sets *"Custom, last 12 months"* — a fixed
12-month view. There is **no way to choose a date range**, so "custom" is not custom.
**How it must work:** selecting **Custom** reveals a **from / to date range picker** (two date
inputs, or a start+end); on change, **all** metrics, the funnel, Customer trends and the three
charts recompute for that exact range (consistent with how the other periods already recompute).
The header should read the chosen range (e.g. "1 Mar 2026 – 30 Jun 2026"), not "last 12 months".
**Fix:** when `state.anPeriod==='Custom'`, render two `<input type=date>` (default to a sensible
range), store them in state, and feed the chosen window into the existing `periodSeries()` /
period computation so every figure and chart follows it.
**Verify (click):** pick Custom, choose two dates, confirm every KPI and all three charts change
to that window and the header shows the range.

# DEFECT 2 — 🔴 Client "Request meeting" can only reach the appointed advisor

**Requirement (Stephen):** "From user view – add a request meeting button with **their advisor
and other advisors** – should open an email as the same as book meeting above."
**How it works now:** the client's **Request meeting** composes only to `state.linkedAdvisor`
(the appointed advisor). There is no way to request a meeting with any *other* advisor.
**How it must work:** Request meeting lets the client choose **which advisor** — their appointed
advisor **or** another advisor from the directory (`ADVISOR_DIR`) — then opens the same editable
email compose to the chosen advisor. Default the selection to the appointed advisor.
**Fix:** in the request-meeting flow, add an advisor selector (dropdown/list of `ADVISOR_DIR`,
appointed one pre-selected); set the compose `to`/`toName` from the chosen advisor. Keep the
existing compose/send behaviour.
**Verify (click):** as the client, Request meeting → change the advisor to a different one →
compose is addressed to that advisor → Send confirms.

# DEFECT 3 — 🔴 Landing page layout does not match Stephen's spec

**Requirement (Stephen):** "For the landing page – I was thinking more like this where **the
right is an image they upload, and the left is how they want to describe their business** –
should have same fields to the advisor profile. It should be the landing page **new users can
go to register or get in touch** with the advisor."
**How it works now:** the image is a **full-width banner across the top**; the **right column is
advisor admin tools** (Referral URL, referral code, "Share my Client Platform", QR download);
the page also shows admin controls (Edit firm page, Admin/Member view) intermixed. It reads as
the advisor's admin console, not the clean public page a prospect lands on.
**How it must work:** present the public landing as Stephen sketched — a **two-column prospect
page**: **left = the business description** (name, FSP licence, bio/description, services offered,
contact: email/phone/website/LinkedIn — same fields as the advisor profile), **right = the large
uploaded image** (the banner the advisor uploads), with a clear **register CTA ("Start
Investing") and "Get in touch"**. The advisor-only tools (referral URL/code, Share, QR download,
Edit firm page, Admin/Member toggle) must be **visually separated** from the public page — e.g.
a distinct "Your page / Share" panel or section clearly labelled as advisor tools — so the
public landing itself shows only what a prospect would see. Keep the firm page + Admin/Member
behaviour, just don't let it clutter the public preview.
**Fix:** restructure `vLanding()`: public two-column (description left, uploaded image right,
register + contact CTAs), then a clearly-separated advisor "Share this page" block for referral/
QR. Make "Start Investing" a believable register entry (open the onboarding/register flow or an
honest register affordance), since Stephen says the page is where new users **register**.
**Verify (look):** the landing reads as a clean public page — image right, description left,
register/contact — with the referral/QR tools clearly separated, not mixed into the public card.

# DEFECT 4 — 🟠 Cross-cutting: opening a drawer breaks dashboard numbers mid-character

**How it works now:** opening any right-hand drawer (bell/notification settings, etc.) on the
Dashboard shrinks the main area (`body.dw-open`), and `overflow-wrap:anywhere` then breaks text
**mid-character**: "R192k" → "R19 2k", "Revenue/commission" → "commissi on", "registered" →
"registere d". It looks broken.
**How it must work:** numbers and words must **never break mid-character**; the dashboard tiles
must degrade gracefully at the narrower width.
**Fix:** stop `overflow-wrap:anywhere` (and any `word-break:break-all`) from applying to the
dashboard KPI tiles / numbers — scope the anywhere-wrapping only to the drawer tables that needed
it, and use `overflow-wrap:normal` / `white-space:nowrap` on numeric values (`.num`, KPI values).
If needed, give the hero/tiles a sensible `min-width` or let them wrap between words only.
**Verify (look):** open the bell/settings drawer on the Dashboard — "R192k", "Revenue /
commission", "New registered users" all stay whole; no mid-character breaks.

# DEFECT 5 — 🟠 Tara per-asset reasons are templated and sometimes contradictory

**Requirement (Stephen):** "There should be a **reason** next to each asset suggested **why Tara
is suggesting it**."
**How it works now:** reasons are templated — an equity ETF gets "…**Suited to your Conservative
profile**" appended (contradictory for an aggressive fund held only as a small sleeve), and two
different bond ETFs share the identical reason "Adds income and stability to the mix."
**How it must work:** each asset's reason should genuinely explain **why it fits this client**,
and not contradict the client's risk profile. An equity sleeve in a conservative portfolio should
read like "a small growth sleeve for long-term upside" rather than "suited to your conservative
profile"; two different bonds should have distinct reasons.
**Fix:** derive the reason from the asset's role in *this* portfolio (its class + weight relative
to the client's risk profile), not a generic "suited to your {type} profile" suffix; ensure two
assets of the same class don't emit identical text.
**Verify (look):** open Tara for a Conservative client (Grant) and an Aggressive client (Zanele);
every reason is sensible for that client (no "suited to your Conservative profile" on Top 40) and
no two lines are identical.

# DEFECT 6 — 🟡 Settings open as drawers, but Stephen asked for a "page"

**Requirement (Stephen):** notifications — "this goes to a **page** where the advisor can select
which notifications they want"; qualification — "show me the **page** where the set the
qualification level."
**How it works now:** both open as **right-hand drawers**.
**How it must work:** present both as their own **full page** (a dedicated view/route), as Stephen
said, with the same controls (notification toggles + amount fields; the four qualification
thresholds). Reached from the same entry points (bell "Settings"; "Edit settings").
**Fix:** add `notifSettings` and `qualSettings` (or similar) routes/views and route the existing
entry points to them instead of `openModal(...)`. Keep all existing save/apply behaviour.
**Verify (click):** "Settings" (bell) and "Edit settings" (qualification) each open a full page;
changing values and saving still recomputes the bell / qualified count.
*(Note on "$x": Stephen wrote "investment over $x", but all customer money in this app is Rands,
so keeping "Investment over R…" is the consistent choice — leave it in Rands; do not switch
customer figures to USD.)*

---

# VALIDATION PROTOCOL (before declaring done — UI only, LOOK at it)
1. Custom period: pick a range → all KPIs + 3 charts follow it; header shows the range.
2. Client Request meeting: can pick a different advisor; compose addresses them.
3. Landing: clean public two-column (image right, description left, register/contact); advisor
   tools separated; "Start Investing" is a believable register entry.
4. Drawer-open on Dashboard: no mid-character breaks in any number/word.
5. Tara reasons: sensible per client, no contradiction, no duplicates.
6. Notification + qualification settings are full pages; save still works.
7. Regression: every closed loop still closes; three theme views + dark clean; no 390px overflow;
   zero em dashes / `.io` / `undefined` / `NaN` / retired labels; zero console errors.

# Definition of done
Custom period is truly custom; the client can request a meeting with any advisor; the landing
page is the clean prospect page Stephen sketched (image right, description left, register/contact,
tools separated); no number breaks mid-character; Tara's reasons are honest and non-duplicative;
settings are pages. Verify by looking at the rendered UI, not by reading source.

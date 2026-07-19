# FIX PROMPT — Vault22 Advisor Prototype (adversarial-QA findings)

> Paste everything below the line into a fresh Claude Code session on this repo.
> Self-contained. Fix the defects found by an adversarial UI critique. Verify each the
> way it was found: by driving the real UI (clicking, switching periods, viewing charts),
> never by trusting that an element exists.

---

You are a senior frontend engineer fixing defects found when a skeptical tester drove the
Vault22 Advisor prototype (`/Users/aishaniatri/Vault22-Advisor-Prototype/index.html`,
single-file vanilla JS, no build). **Read it first.** Each defect below is real and
reproducible. Fix behaviour, then re-verify by driving the UI.

## House rules (keep them; do not regress)
No em dashes in rendered copy · comma-grouped Rands via `money()` (subscription fee is the
only USD, via `usd()`) · `vault22.com` never `.io` · UK spelling · no internal/dev references
in the UI · dates via `localISO()` · **preserve every working behaviour and closed loop**
(meetings both ways, documents request→fulfil, portfolio submit→approve→live, Tara
suggest→tweak→invest, firm Admin/Member, consent, membership PDF, notification settings,
qualification thresholds, the three-view theming Green/Mid-navy/Navy, dark mode). Seed data
stays clearly illustrative.

---

# DEFECT 1 — 🔴 HIGH: Analytics line charts are visually broken (stop mid-axis)

**Symptom (reproduce):** Analytics page → the three area/line charts — **Monthly growth in
registered users / customers**, **AUM growth**, **Investment trends** — each plot a line that
**stops around May and leaves the right ~40% (Jun, Jul) blank**, even though the page header
says "July 2026" and the x-axis runs Feb→Jul. It reads as a half-rendered/broken chart.

**Root cause to check:** the series feeding `spark()` (e.g. `monthlySeries` / the arrays)
only contain data points up to ~May, or the point count doesn't match the labelled x-axis
span, so the polyline ends before the right edge.

**Required behaviour:** each chart's plotted line **spans the full x-axis** of the selected
period, ending at the current month with a real value. No blank right-hand gap. The area
fill and the line must reach the last labelled month.

**Verify:** open Analytics; each of the three charts draws all the way to the right edge
(July), no empty tail.

---

# DEFECT 2 — 🔴 HIGH: The period selector does not drive the charts (fails the requirement)

**Symptom (reproduce):** Analytics → switch **This month → Last month → Last 90 days → YTD →
Custom**. The KPI tiles, the funnel, "customers lost", and Customer trends **do** recompute —
but the **three line charts are pixel-identical across every period** (same Feb–May curve).
So the charts ignore the selector entirely.

**Why it matters:** Stephen was explicit — *"All the metrics should all follow the same
period."* Right now the charts are the one thing that doesn't. The requirement is only
half-met.

**Required behaviour:** when the advisor changes the period, **every** chart's data recomputes
for that window and redraws, consistent with the KPIs and trends. The x-axis labels and the
plotted range must reflect the selected period (e.g. YTD spans Jan→Jul; Last 90 days spans the
last three months; This month may show a short intra-month series). Seed period-appropriate
series so the change is visibly different, and keep it internally consistent with the KPI
numbers already shown for that period.

**Verify:** switch between at least three periods; each time all three charts visibly change
and match the period label; none stay frozen on the Feb–May curve.

---

# DEFECT 3 — 🟠 MODERATE: "Suggested additions" heading reads like an internal TODO

**Symptom:** the Analytics page ends with a section literally titled **"Suggested additions"**
listing "Change in assets under advice" and "Change in average portfolio." That phrasing reads
like a developer's internal note, not a product section an advisor should see.

**Required behaviour:** rename the section to something an advisor understands, or fold those
two rows into the existing **"Customer trends"** block so there is one coherent trends section.
Choose the option that reads cleanest; do not ship a section that sounds like a TODO. Confirm
no other internal-sounding headings remain on the page.

**Verify:** the Analytics page has no "Suggested additions" (or similarly internal) heading;
the trend rows sit under a clear, advisor-facing title.

---

# DEFECT 4 — 🟠 MODERATE: The CRM panel is bolted onto every customer-profile tab

**Symptom:** on a customer/registered-user full profile, the whole **CRM block (Notes / Tags /
Tasks / Follow-ups / Meetings / Files + "Add a note")** renders at the bottom of **every** tab
— Personal, Financial (even when consent-locked), Investment, Risk, Documents, Onboarding,
Activity, and Audit. On the Audit tab you get "Audit log for this client" followed by an
unrelated "Add a note" box. It's redundant and clutters, and it looks wrong on the compliance
and consent-locked tabs.

**Required behaviour:** the CRM should appear in **one** sensible place, not repeated under
every tab. Pick the cleaner of:
- make **CRM its own tab** (e.g. a "CRM" or "Notes" tab in the §8 tab row), or
- show the CRM block **only on the Personal/overview tab** (and not on Financial/Investment/
  Risk/Documents/Onboarding/Activity/Audit).
Keep all CRM functionality intact (add note/tag/task/follow-up/meeting/file); just stop
rendering it under every tab. Make sure whichever tabs no longer show CRM still look complete
(no empty trailing space).

**Verify:** open each profile tab; the CRM block appears in exactly one place, and the Audit /
Documents / consent-locked Financial tabs no longer show a stray "Add a note" box.

---

# DEFECT 5 — 🟡 MINOR: "Customer retention 100.0%" on the default period looks fake

**Symptom:** Analytics defaults to **This month**, where retention shows **100.0%** with "0
lost this period." Exactly 100% reads as fake-perfect in a live demo (YTD correctly shows
71.4% with two lost customers).

**Required behaviour (choose one):** either default the Analytics period to one that shows a
believable retention figure (e.g. YTD or Last 90 days), or seed at least one lost customer
into the current-month window so "This month" retention is realistic. Keep the math honest and
consistent with "customers lost this period."

**Verify:** the Analytics page, as first opened, shows a credible retention number backed by a
matching "customers lost" list.

---

# VALIDATION PROTOCOL (before declaring done — UI only)

Drive the real UI; do not judge by source or by element presence.
1. **Defect 1:** open Analytics; confirm all three charts draw to the right edge (July), no
   blank tail.
2. **Defect 2:** switch period across This month / Last month / Last 90 days / YTD; confirm
   every chart redraws and matches the period each time, consistent with the KPI numbers.
3. **Defect 3:** confirm no "Suggested additions" (or other internal-sounding) heading.
4. **Defect 4:** click through all customer-profile tabs; CRM shows in exactly one place; the
   Audit / Documents / locked-Financial tabs have no stray CRM box.
5. **Defect 5:** on first open, retention is a believable number with a matching lost-customer
   list.
6. **Regression:** re-drive the closed loops (meetings both ways, documents request→fulfil,
   portfolio submit→approve→live, Tara suggest→tweak→invest, firm Admin/Member, risk nudge,
   consent, membership PDF, notification settings, qualification thresholds), all three theme
   views, dark mode, and 390px — nothing regresses; zero console/page errors; no em dashes /
   `.io` / `undefined` / `NaN` / retired labels.

# Definition of done
The Analytics charts span the full period and follow the period selector; no internal-sounding
headings; CRM appears once, not under every tab; the default retention figure is believable;
and no working behaviour or closed loop has regressed.

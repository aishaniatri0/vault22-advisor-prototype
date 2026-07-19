# Stephen Round-2 Feedback — Build Prompt

Predicted follow-up feedback from Stephen after a fresh 30-minute review of the current build, converted into an actionable build prompt. This is the advisor prototype (`index.html`, single-file vanilla-JS SPA, no build step). Demos with SA IFA groups are imminent, so **demo-readiness and end-to-end loops matter more than new surface area.**

## Ground rules (unchanged from prior prompts)
- Source of truth = Stephen's intent. Fix behaviour, not just labels.
- Keep the architecture: in-memory `state`, `render()` re-render loop, `openModal`/`.rdrawer`, delegated click handler (~`:5757` selector list — any new `data-*` click target must be registered there), `audit(...)` logging, `sessionStorage` persistence keyed to `BUILD_VERSION`.
- No em dashes in rendered copy · Rands via `money()` · `vault22.com` never `.io` · UK spelling · dates via `localISO()` · no bare `Date.now()`/`new Date()`.
- **Do not regress** the verified P0–P3 QA fixes or the booking redesign. Extend, don't replace, meeting/consent/Tara state.
- Each item tagged **[Done]** (verify only), **[Partial]** (finish it), or **[New]** (build it). Line numbers are pointers; locate by nearby code.

---

## P0 — Demo safety (do first; Stephen explicitly asked for these)

### 1. "What's real vs mocked" one-pager — [New]
Produce a short doc (`DEMO-REALVSMOCK.md`) listing, per screen, what is interactive vs illustrative, and any button that only toasts/stubs. Not code — a reference so Stephen doesn't click a dead thing live.

### 2. "Reset demo" control — [New]
**Where:** Prototype settings gear (`~:1948`), alongside scenario/theme controls.
**Required:** A "Reset demo" button that clears the `sessionStorage` snapshot (`SESSION_KEY()`) and reloads the current scenario clean (`loadScenario(state.scen)` or reload). Confirm-before-wipe. Session persistence is already in (`saveSession`/`restoreSession`) — this is the escape hatch between meetings.
**Accept:** Make edits → Reset demo → back to a clean advisor/established-advisor with no leftover docs/notes/Tara drafts.

### 3. Mobile hold-up on the demo screens — [Partial/New]
**Required:** Ensure dashboard, a client full profile, booking, and the public landing page are usable on a phone (no horizontal scroll, tap targets, drawers become full-width sheets). Where a screen isn't mobile-ready, degrade gracefully rather than render broken. Stephen will show this on his phone.
**Accept:** Load those four at ~390px width — no overflow, everything reachable.

---

## P1 — End-to-end loops Stephen will click through

### 4. Risk-reassessment loop, both sides — [Partial]
Advisor trigger on the client header is [Done]. **Finish the user side:** when the advisor requests a reassessment, the user must see a prompt in their app (Your Advisor / notifications), and once they "redo" it, the client's **Last risk assessment** date updates and the advisor sees it. Make this demonstrable for the established-advisor client (C1) at minimum.
**Accept:** Trigger from advisor → switch to client scenario → user sees the request → completes it → date updates on both sides.

### 5. Tara — make it advisor-grade — [Partial]
Core Tara (full-screen, ZAR/USD, sections, weightings, Suggest→client) is [Done]. Add:
- **Strategy summary** line at the top (what Tara recommends and why) above the per-asset reasons.
- **Constrain to the client's risk profile** — do not suggest Aggressive assets to a Conservative client; show the suggested portfolio's **expected return + risk vs the client's profile**.
- **Before/after vs current portfolio** — what's changing, Rand amounts, and fees.
- **Advisor note to client** field + a compliance **disclaimer** on what's being sent.
- Verify the client receive→tweak→invest path end to end.
**Accept:** Open Tara on a Conservative client → suggestions respect the profile → strategy summary + before/after show → Suggest with a note → client sees note + disclaimer → tweaks → invests → advisor sees it.

### 6. "Get in touch" creates a real lead — [Partial]
**Where:** landing page `getInTouchDrawer` (exists). **Required:** submitting it should create a lead in the advisor's dashboard (Registered Users / a "new enquiry" to-do), not only compose an email.
**Accept:** Submit Get-in-touch → a new lead/enquiry appears for the advisor.

### 7. To-do list items all click through — [Partial]
Some rows are wired (`data-meetgo`, KPI tiles with `data-prefilter`). **Required:** every To-do row (Chase KYC, Chase risk assessments, Follow up with qualified users, Confirm meeting requests) navigates to its correctly filtered list.
**Accept:** Click each To-do row → lands on the right filtered view.

---

## P2 — Screen refinements

### 8. Global client search in the top bar — [New]
Top-bar search (name/email/phone) with a results dropdown that jumps to the client profile. Advisors live in this.

### 9. Client Overview at a glance — [New]
Add an **Overview** as the first profile tab (or a header strip): stage, risk profile, net worth, last contact, fees earned — so the advisor doesn't hunt across tabs.

### 10. Acquisition Source column — [New]
Add a **Source** column on Registered Users / Customers (which landing page or QR the user came through). This is the point of the advisor channel. Seed varied sources.

### 11. Bulk actions + export — [New]
Registered Users: multi-select → "Email" / "Export". Customers: add **Last contacted** (from CRM) and allow sort by **Fees earned**.

### 12. Notification channels + digest — [New]
On the notification-settings page (on/off toggles are [Done]), add a **channel per notification** (Email / Push / WhatsApp / Telegram — the app already has Telegram) and a **daily digest** option.

### 13. Documents: preview + expiry — [Partial]
Make documents **preview/download** (placeholder file is fine) and add an **Expiry date** field (KYC / proof of address expire — surface what's out of date).

### 14. Request-all consent — [New]
On the client profile, a **"Request all"** that asks for Financial + Investment + Risk in one go; the user sees a single consent screen.

### 15. Membership fee tier table on the card — [Partial]
Invoice wording is [Done]. Surface the **fee tier table** on the membership card (1-10 / 11-50 / 51+) with the advisor's current tier highlighted and next-tier cost. AUM-fee-to-client stays as-is pending Greg.

### 16. Reconcile Revenue/Commission with membership — [Partial]
Relabel the dashboard tile to "Estimated fees this month" and tie the number to the invoice logic (subscription tier + any advisor-side fees), so it's consistent with Membership. Add a small sparkline (sparkline util already exists in the portfolio drawer — reuse it).

### 17. Landing page testimonials/rating — [New]
Add a short testimonials / rating block to the public landing page.

### 18. Firm "Team" view shell — [New]
With admin/member in place, add a **Team** view for the firm: advisors at the firm, combined AUM + client count. Shell is enough; note deeper firm rollup as future.

### 19. Portfolios: openable factsheet + real return/vol — [Partial]
Make the factsheet **openable** (placeholder PDF), show expected return + volatility as numbers in the View drawer, and mock a **Vault22 moderation queue** for submitted portfolios. `@Uday` to supply the full SA portfolio list to replace placeholders.

### 20. Analytics export + breakdowns — [New]
Add **Export / Download report** (CSV/PDF) and a breakdown by **product and portfolio** (not only top products). Period selector is [Done].

### 21. Permissions: POA request flow + audit export — [Partial]
POA concept exists heavily [Partial]. Add an advisor-initiated **Power of attorney request** that the client approves, wired to the "requires authorisation" state. Add **Export** on the audit log for compliance.

### 22. Booking: availability + meeting types (+ real-build note) — [New]
Add advisor **availability settings** (working hours + buffer between meetings) and **meeting-type presets** (30/60 min · video/call/in person). Add a visible note that live calendar (Google/Outlook) sync comes in the real build. Calendar/slots/conflict-blocking are [Done].

---

## P3 — Theme decision + copy polish

### 23. Simplify themes to Green vs Navy — [Partial]
Stephen's decision: **keep navy as the advisor view, drop the "Mid navy" middle option** (currently three: Green / Mid navy / Navy). Remove `theme-midnavy` and its selector entry; keep Green (consumer) vs Navy (advisor). Verify **dark mode** holds up on every advisor screen including Tara and booking.

### 24. Copy consistency pass — [Partial]
- "Appoint/Appointed" everywhere; remove the residual "unlinked / re-link" strings (currently only in code comments — clean them anyway to be safe).
- Consistent table-header capitalisation; no placeholder text left in any rendered surface.

---

## Definition of done
- P0 (1–3) and P1 (4–7) complete and **clicked through end to end** — these are what Stephen said he'll look at first Sunday evening.
- Each item re-verified against its Accept line by driving the flow, not by assuming.
- No regression to the P0–P3 QA fixes or the booking redesign.
- Update `DEMO-REALVSMOCK.md` as items move from mocked to real.

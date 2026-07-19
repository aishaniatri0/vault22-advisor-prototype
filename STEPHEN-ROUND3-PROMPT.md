# Round-3 Build Prompt — FINAL implementation pass

Advisor prototype (`index.html`, single-file vanilla-JS SPA, no build). This is the **final** implementation round before QA and sharing. Implement everything below. Do not leave items half-wired; each has an Accept line — satisfy it by driving the UI, zero console errors.

## Ground rules
- Keep the architecture: in-memory `state`, `render()` loop, `openModal`/`.rdrawer`, delegated click handler (register every new click `data-*` in the selector list ~`:5757`), `audit(...)`, `sessionStorage` persistence keyed to `BUILD_VERSION`, `Reset demo` in the settings gear.
- No em dashes in rendered copy · Rands via `money()` · `vault22.com` never `.io` · UK spelling · dates via `localISO()` · no bare `Date.now()`/`new Date()`.
- Do not regress any prior P0–P3 QA fix, the booking redesign, the reassessment loop, or Tara.
- Tags: **[New]** build it · **[Partial]** finish/extend existing · **[Verify]** likely present, confirm. Line numbers are pointers.

---

## P0 — Demo story (Stephen leads the demo with these)

### 1. Advisor first-run setup checklist — [New]
In the empty "new advisor" scenario, replace the bare dashboard with a **setup checklist** card: "Set up your profile · Customise your landing page · Invite your first client · Share your QR" as 0/4, each row linking to that screen and ticking off as done (persist ticks in `state`). Once 4/4, collapse to the normal dashboard.
**Accept:** New-advisor scenario opens on the checklist; completing a step ticks it; 4/4 reveals the full dashboard.

### 2. Guided demo flow — [New]
Add `DEMO-FLOW.md`: the exact 10-minute click path (new advisor sets up → shares link/QR → lead arrives → converts to customer → Tara personalises a portfolio → shows on book + analytics), one line of narration per screen. Confirm the prototype supports that path with no dead ends. Update `DEMO-REALVSMOCK.md` if anything changes.

---

## P1 — Loops & features Stephen will click live

### 3. Global client search (top bar) — [New]
Top-bar search (name/email/phone) with a results dropdown that jumps to the client profile.
**Accept:** Type a partial name → matching clients listed → click → lands on their profile.

### 4. Client Overview (first tab / header strip) — [New]
Add an **Overview** as the first profile tab or a header strip: stage, risk profile, net worth, last contact, fees earned, last risk assessment.
**Accept:** Opening any client shows the Overview first, all fields populated.

### 5. Document preview + expiry — [Partial]
Make documents **open/preview** (placeholder PDF acceptable) and add an **Expiry date** field; flag expired/expiring docs (KYC, proof of address).
**Accept:** Click a document → it opens; an expired doc shows an "Expired" flag.

### 6. Advisor directory (mocked) — [New]
A browsable **"Browse advisors"** page (static/seeded cards using the advisor-profile fields: name, firm, FSP licence, services, location) with search/filter. Link the user-side "Search advisors" (appoint flow) to it.
**Accept:** Directory lists several advisors; search filters; selecting one flows into appoint.

### 7. Request loops: Document + KYC refresh — [Partial]
Reuse the reassessment-loop pattern for advisor-initiated **"Request a document"** and **"Request KYC refresh"**: advisor triggers → client sees an "Action needed" card → completing it updates the advisor view + notifies.
**Accept:** Trigger each from advisor → client scenario shows the request → completing updates both sides.

### 8. Power-of-attorney request flow + audit export — [Partial]
POA concept exists. Add an advisor-initiated **POA request** the client approves, wired to the portfolio "requires authorisation" state (advisor may change portfolio only once granted). Add **Export** on the audit log (CSV download).
**Accept:** Advisor requests POA → client approves → the portfolio action unlocks + is logged; audit log exports a file.

---

## P2 — Screens & refinements (ship these too; final round)

### 9. Source → Analytics — [Partial]
Source column is [Verify]. Add an **acquisition-by-source** breakdown in Analytics (link vs QR vs directory vs enquiry), respecting the period selector.

### 10. Estimated fees breakdown — [Partial]
On the dashboard tile, add an expand/hover breakdown (subscription tier vs any advisor fees). Keep the AUM 25bps client-billed.

### 11. Firm Team view — [New]
A **Team** view (admin/member exists): advisors at the firm with combined AUM + client count. Shell depth is fine.

### 12. Bulk actions + Last contacted — [New]
Registered Users: multi-select → "Email" / "Export". Customers: add **Last contacted** (from CRM); allow sort by Fees earned.

### 13. Notification channels + digest — [New]
On notification settings add a **channel per type** (Email / Push / WhatsApp / Telegram) and a **daily digest** toggle.

### 14. Request-all consent — [New]
Client profile: a **"Request all"** asking for Financial + Investment + Risk in one action; client sees a single POPIA-style consent screen.

### 15. Analytics export — [New]
**Download report** (CSV/PDF) plus a breakdown by product and portfolio.

### 16. Booking availability settings — [Partial]
Advisor **availability** (working hours + buffer) and **meeting-type presets** (30/60 min · video/call/in person). Calendar/slots/conflict-blocking already ship.

### 17. Portfolios: openable factsheet + moderation queue — [Partial]
Factsheet **opens** (placeholder), show expected return + volatility numbers in the View drawer, and mock a **Vault22 moderation queue** for submitted portfolios.

---

## P3 — Credibility, Tara depth, branding, polish

### 18. SA compliance credibility — [Partial]
- Show **FSP / FAIS licence number** prominently on advisor profile, landing page, and client-facing footer.
- Make data-sharing consent read as explicit **POPIA-style** consent: clear, revocable, "withdraw any time".
- Frame every Tara suggestion as a **record of advice** (recommendation, reason, disclaimer, timestamp, logged) with a clear "advice is the advisor's, not Vault22's" line.
**Accept:** Licence visible in the three places; consent copy reads POPIA-style; a sent Tara suggestion shows a logged record-of-advice with disclaimer.

### 19. Tara depth — goals + TFSA — [Partial]
Tara references the client's **goals** and SA **TFSA** allowance ("uses Rx of the tax-free allowance"), shows fit vs the **goals timeline** (not only risk profile), and the advisor can **save a suggestion as a draft** visible in the client's CRM/timeline.
**Accept:** Open Tara → suggestion cites goals + TFSA usage → save draft → draft appears in that client's timeline.

### 20. Co-branding — [New]
Advisor/firm **own logo alongside Vault22** ("powered by Vault22") on the landing page and client-facing surfaces.

### 21. Seed-number sanity pass — [Partial]
Review every Rand figure (net worth, AUM, fees, investment amounts) for South-African plausibility; fix anything that reads fake.

### 22. Dark/navy + mobile pass on new screens — [Partial]
Contrast pass on Tara before/after table, membership card, source column, checklist, directory, overview. Mobile pass specifically on **Tara** and the **client profile**.

### 23. Leave-behind — [New]
A **one-page shareable summary** (or a clean shareable link with 1–2 pre-loaded advisors) the IFA groups can revisit after the meeting.

---

## Definition of done
Every item's Accept line satisfied by driving the UI, zero console errors. No regressions to prior rounds. `DEMO-FLOW.md` + `DEMO-REALVSMOCK.md` current. This is the last build pass before final QA.

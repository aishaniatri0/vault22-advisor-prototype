# Vault22 Advisor — 10-Minute Guided Demo

The exact click path for a live demo. Switch scenarios with the bottom-right gear. Use
**Reset demo** (gear) between runs for a clean start. One line of narration per screen.

## 0. Setup (30s)
- Open the prototype. Gear → **New advisor** to start from scratch, or **Established advisor**
  to show a full book. For the full story, start on **New advisor**.

## 1. New advisor sets up (2 min) — *New advisor scenario*
- Complete the short onboarding (email, password, phone, region, policies). Any 6-digit code passes.
- Land on the **setup checklist** (0/4). Narrate: "Every advisor starts here."
  - **Set up your profile** → add FSP licence + firm (ticks step 1).
  - **Customise your landing page** → add photo/bio/services, Save (ticks step 2).
  - **Invite your first client / Share your QR** → open Share, download the QR (ticks 3 and 4).
- At 4/4 the checklist collapses to the dashboard.

## 2. Share the link / QR (30s)
- Dashboard → **Share my Client Platform**. Show the personal link and the real QR code.
  Narrate: "Every signup through this is tagged to the advisor."

## 3. A lead arrives (1 min)
- Gear → **Established advisor** (jumps to a live book so we can show conversion).
- Landing Page → **Start Investing** → register a prospect. Narrate: "They came through the
  link, so they land in this advisor's book as a Registered User."
- Or Landing → **Get in touch** → creates a real enquiry lead with the message as a CRM note.
- Dashboard: the **Registered users** count and **acquisition source** update.

## 4. Work the client (2 min)
- Registered Users → open a client. **Overview** tab shows stage, risk, net worth, fees, last
  contact at a glance.
- Header actions: **Request risk assessment**, **Request KYC refresh**, **Request all data access**.
- Gear → **Financial Advisor** (client view) → the client sees the **Action needed** cards and a
  **POPIA consent** request → complete them. Gear back to advisor: dates and consent update.

## 5. Tara personalises a portfolio (2 min) — the headline moment
- Open a **customer** → **Personalise portfolio** (Tara).
- Show the **strategy summary** (expected return, volatility vs profile, blended fee, within-profile),
  the **goals + TFSA** line, and the **before/after** table.
- Write a **note to the client**, then **Suggest to client** (or **Save draft** to log it in their timeline).
- Gear → **Financial Advisor** → the client sees the **record of advice** (note + disclaimer + licence + date),
  tweaks the weightings, and **Invests**. Gear back: the advisor sees it invested.

## 6. Booking (1 min)
- Any client → **Book meeting**. Show the **month calendar**, grouped **time slots** (booked slots
  disabled), **duration / timezone / location**, and the confirmation summary.
- **Send invite** → editable email → **Send**: a real `.ics` downloads and the mail client opens.
- Gear → client → **Accept** the meeting. Gear back: advisor sees **Confirmed**.

## 7. Analytics + book (1 min)
- **Analytics**: switch the period (incl. **Custom**), show **acquisition by source**, the
  **by-product / by-portfolio** breakdowns, and **Export CSV / Download report**.
- **Portfolios**: open a **fact sheet** (PDF). Advisors see their own submissions read-only; the
  **moderation queue** (Approve/Reject) is a Vault22 admin function, now in the settings gear under
  **Vault22 admin** (final home to be confirmed with Uday).
- **Audit Log**: **Export CSV** for compliance.

## No dead ends
Every button on this path is wired. Stubs (e.g. client-app sections other than Profile, "collapse
menu") show an explanatory modal, never a blank. See `DEMO-REALVSMOCK.md` for the full real/mock map.

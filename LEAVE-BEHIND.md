# Vault22 for Financial Advisors — Summary

*A leave-behind for IFA groups. The live prototype is pre-loaded with an established advisor
(Aishani Atri, Vault22 Wealth) and a full sample book, so you can revisit everything below.*

**Try it:** open the shared link, then use the bottom-right gear to switch between **New advisor**,
**Established advisor**, and the **client (Financial Advisor) view**. **Reset demo** restores a clean start.

## What Vault22 gives an advisor
- **Your own client-acquisition channel.** A personal landing page + QR. Every signup through it is
  tagged to you and appears in your book, with the **acquisition source** tracked.
- **One place to run the book.** Dashboard with pending-KYC / pending-risk tiles, a to-do list, and
  estimated fees; Registered Users and Customers with search, filters, source, last-contacted and
  bulk export; a global client search in the top bar.
- **Client profiles that close the loop.** Consent-gated Financial / Investment / Risk (POPIA-style,
  revocable), documents with request→fulfil and expiry flags, an at-a-glance Overview, and CRM.
- **Advisor-initiated requests that reach the client.** Risk reassessment, KYC refresh, power of
  attorney and full data-access requests: the client sees an action, completes it, and both sides update.
- **Tara, an AI co-pilot for advice.** Personalise a portfolio constrained to the client's risk profile,
  with expected return/volatility, blended fee, goals + tax-free-savings context, and a before/after.
  Sent as a **record of advice** (recommendation, reason, disclaimer, licence, timestamp, logged).
- **Calendly-grade booking.** Month calendar, conflict-aware slots, duration/timezone/location, a real
  `.ics`, and a two-way accept/confirm loop with the client.
- **Analytics + compliance.** Period-driven metrics (incl. custom range), acquisition-by-source and
  by-product/portfolio breakdowns, CSV/PDF export, and an exportable audit log.
- **Firm-ready.** Admin vs Member, a shared firm page, an advisor directory, and a firm Team view.

## Commercials
- **Advisor subscription** (per month, USD): 1–10 customers $10 · 11–50 $25 · 51+ $40.
- **AUM fee**: 25 bps p.a. on the client's non-Vault22 assets, **billed to the client**, not the advisor.
- The consumer investment journey is unchanged; the advisor layer sits on top of it.

## What is real in the prototype vs the full build
Everything in the demo is clickable front-end. The full build adds delivery and integrations:
live email/SMS/WhatsApp/Telegram, Google/Outlook calendar sync, secure document storage, the real
SA product universe (with Uday), and final pricing (with Greg). See `DEMO-REALVSMOCK.md`.

## Open decisions to confirm
- AUM 25 bps: keep client-billed (current) or roll into the advisor total — **Greg**.
- Final SA portfolio/securities list — **Uday**.

*Illustrative South African sample data throughout. Not financial advice.*

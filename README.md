# Vault22 — Advisor Prototype

High-fidelity, clickable **desktop web** prototype of the Vault22 advisor platform: advisors
self-onboard, get a branded page and a real QR code, share them, and every signup through the link
is tagged back to them as a **lead**. Leads who buy a product become **customers**, and the advisor
gets a dashboard over both.

Built as a natural extension of the live Vault22 web app — the shell, cards, brand green `#01C38D`,
Inter type and the split-screen onboarding were taken from `global-website.dev.vault22.com` and the
existing onboarding prototype, so this reads as the same product rather than a mock.

## Run it
Open `index.html` in any modern browser. No build step, no server, no dependencies.

## The two scenarios
Use the **Demo** switcher in the top bar (or the **Prototype controls** bar during onboarding):

- **New advisor** — the full onboarding journey, before a dashboard exists:
  entry → register → region → verify → **advisor name** → link & QR → share → dashboard (empty state).
- **Established** — Aishani Atri, 8 leads and 5 customers, populated dashboard.

`Sign in` from the entry screen drops straight into the established advisor.

## What to look at
- **The advisor name step** is mandatory — there is no link without it. Try `admin` (taken),
  `ab` (too short), `-x` (bad hyphen), then a real name; the URL assembles live as you type.
- **The QR code is real.** Not a decorative grid — a byte-mode, ECC-M encoder written from scratch,
  verified byte-identical to Apple's `CIQRCodeGenerator` reference and decoded end-to-end by
  `CIDetector` across versions 2–8. Scan it with your phone.
- **Consent masking** — on the dashboard, the amber dotted values are client financial data whose
  advisor visibility is still under privacy review. The toggle previews the portal without them.

## Structure
Everything is one self-contained file (`index.html`): tokens, QR encoder, mock data, views, router.

- Onboarding is a screen router (`ONB` / `ONB_AFTER`), one function per screen. It runs **before**
  the app shell exists — while onboarding is live there is no topbar, sidebar or nav.
- The dashboard is a `state.route` + `VIEWS` map.

## URLs
| What | Format |
|---|---|
| Advisor dashboard | `vault22.com/advisor` |
| Advisor's shareable link | `{advisor-name}.advisor.vault22.com` |

## Scope
Covers §1–§15 of the "Advisor in v22 platform" spec: registration, profile, landing page, QR &
referral, lifecycle (8 lead stages), dashboard KPIs, lead & customer management, customer profile,
notifications, CRM, search & filters, analytics, and permissions with an audit log.

**Prototype notes:** front-end only — no auth, no backend, no real email/OTP. Mock data throughout;
figures are illustrative placeholders to show states, not real client data. Nothing persists across
reload. The doc's "Future plans" table is listed in-UI as Roadmap and deliberately not built.

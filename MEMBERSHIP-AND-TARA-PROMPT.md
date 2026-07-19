# MASTER IMPLEMENTATION PROMPT — Vault22 Advisor Prototype (Stephen feedback, round 3)

> **How to use this file:** paste everything below the line into a fresh Claude Code
> session opened on this repo. It is self-contained. Execute all changes in one pass.

---

You are a senior frontend engineer implementing a founder's review feedback on a
high-priority, demo-ready prototype. Work entirely inside
`/Users/aishaniatri/Vault22-Advisor-Prototype/index.html`, a single-file, vanilla-JS,
no-build SPA (~4,700 lines). It is live at
`https://aishaniatri0.github.io/vault22-advisor-prototype/` and auto-updates when pushed.
**Read `index.html` in full before changing anything.**

This round adds two things Stephen asked for:
1. A **Membership plan** for the advisor (billing), in My Profile, with a downloadable
   PDF invoice.
2. **Tara for the advisor**: a "Personalise portfolio" flow that opens Tara full screen,
   suggests a portfolio from the client's profile, lets the advisor tweak it, and sends it
   to the client, who can then tweak and invest.

Do everything in this document in one go. Do not stop to ask questions, every decision you
need has been made below.

## Non-negotiable house rules (already enforced across the build, do not break them)
- **No em dashes** in rendered UI copy. Comments in code are fine. Gate on the rendered DOM.
- **Customer money uses comma-grouped Rands**: `R2,450,000`, via the existing `money()`
  helper. Never `toLocaleString('en-ZA')`. No decimals on headline money.
- **The membership subscription fee is priced in USD** (Stephen quoted `$10 / $25 / $40`,
  a Vault22 platform subscription). This is the one deliberate USD figure in the app. Add a
  small `usd(n)` helper (`'$' + Math.round(n).toLocaleString('en-US')`) and use it only for
  the subscription fee. Everything the customer sees stays in Rands. Add a code comment and a
  visible "final pricing to be confirmed" note.
- **Demo emails/domains use `vault22.com`**, never `.io`. **UK spelling** (personalise,
  customise, optimise, colour).
- **No internal/dev/reviewer references** in the UI.
- **Every control lands somewhere real.** No dead buttons; a stub must say so honestly.
- **Right-hand panels shrink the page beside them (no dark-scrim pop-ups)** via the existing
  `.rdrawer` / `body.dw-open` pattern. **Full-screen takeovers** (Tara) use the existing
  `body.onboarding`-style approach: hide the app shell and render into `#onbRoot` (or a
  dedicated root), with a Back control.
- **Pluralise correctly** (`1 customer`, not `1 customers`), via the existing `plural()`.
- **Never build dates from `toISOString()`**. Use the existing `localISO()` helper.
- **Preserve working functionality**: the consent round trip, QR encoder, onboarding, region
  select, two-way binding, themes (green/navy, light/dark), the customer profile tabs, the
  portfolio module, analytics period selector, notification settings, and everything else
  shipped in rounds 1 and 2. Prioritise demo readiness over completeness. Use realistic
  seeded data, clearly illustrative, never real client data.

## Key code anchors (confirm on read, names are stable)
- Data seeds: `ADVISOR_EST` / `ADVISOR_NEW`, `CUSTOMERS_EST`, `PORTS`, `CONSENT_SEED`,
  `PREV_PERIOD`, `NOTIF_PREFS`, `ADVISOR_DIR`.
- Money/format: `money()`, `short()`, `esc()`, `localISO()`, `plural()`, `dl()` (Detail/Value
  table), `ptbl` (client-side table), `spark()` (chart), `feesFor()` (fee basis).
- Advisor profile: `vProfile()`, `advisorPreview()`, `customiseDrawer()`.
- Customer profile: `vCustomer()`, `tabs8()`, `tab8Investment()`, `tab8Risk()`,
  `financialReport()`.
- Drawers/overlays: `openModal()` / `closeModal()` (right-hand `.rdrawer`), the
  `body.onboarding` full-screen takeover, `composeDrawer()` (an editable-email example).
- Client app: `state.scen === 'client'`, `vClientProfile()`, `clientShell()`, `clientNotifs()`,
  `CLIENT_ID` (the demo client is `C1`, Lerato Khumalo), `state.clientReminders`.
- Wiring: `render()`, `VIEWS`, `NAV`, `loadScenario()`, the delegated `click` handler (add new
  `data-*` keys to its selector string), the `input`/`change` handlers, `audit()`, `toast()`.
- Download pattern: `buildICS()` / `downloadICS()` build a real Blob and download it. Reuse
  this pattern for the PDF invoice.

---

# 1. MEMBERSHIP PLAN (advisor billing, in My Profile)

## 1.1 Data model
- Add `joined` to the advisor seeds: `ADVISOR_EST.joined = '2025-09-04'`;
  `ADVISOR_NEW.joined` should be set to today at onboarding completion via `localISO(new Date()).slice(0,10)`.
- Add a **non-Vault22 AUM** figure per customer in `CUSTOMERS_EST`: `extAum` = assets the
  advisor tracks for the client that are held outside Vault22 portfolios. Seed a realistic
  spread so some clients have external assets and some do not, e.g. C1 `extAum:850000`,
  C2 `extAum:0`, C3 `extAum:120000`, C4 `extAum:0`, C5 `extAum:640000`.
- Derived helpers (add near `feesFor`):
  - `custCountAsOf(monthEnd)` = customers whose `since` is on or before that month end.
  - `subFee(n)` = the USD subscription tier: `n <= 0 ? 0 : n <= 10 ? 10 : n <= 50 ? 25 : 40`.
  - `nonV22Aum()` = `sum(c.extAum)`; `totalAum()` = `sum(c.pv) + nonV22Aum()`.
  - `aumFeeMonthly(nonV22)` = `nonV22 * 0.0025 / 12` (25 bps per annum, charged monthly).

## 1.2 My Profile: Membership plan card
Add a **Membership plan** card to `vProfile()` (the right-hand column, near Account):
- **Date joined**: `state.advisor.joined`.
- **Current monthly fee**: `usd(subFee(currentCustomerCount))` with a sublabel naming the tier
  (e.g. `5 customers, 1 to 10 tier`).
- **This month's invoice** summary line, and a short list of the **last 4 to 6 monthly
  invoices** (month label + amount), each with a **Download PDF** button and a **View** action
  that opens the invoice in a right-hand `.rdrawer`.
- A small note: `Subscription pricing shown in USD and to be confirmed. The 25 bps AUM fee on
  non-Vault22 assets is paid by the customer, to be confirmed with Greg.`

## 1.3 Invoice content
Each invoice (seed the last 4 to 6 months, derived from the book) contains:
- Header: Vault22 wordmark text, `Invoice`, invoice number (e.g. `V22-ADV-2026-07`), the
  advisor's name + FSP licence, date joined, and the billing month.
- **Number of customers** in that month (`custCountAsOf(that month end)`).
- **Total AUM** (Rands).
- **Total non-Vault22 AUM** (Rands).
- **Subscription fee** for the month (`usd(subFee(count))`), with the tier shown.
- **AUM fee**: `money(aumFeeMonthly(nonV22))` for the month, labelled
  `25 bps per annum on non-Vault22 assets, charged monthly, paid by customer`.
- **Amount due from you** = the subscription fee (USD). The AUM fee is a separate,
  customer-paid line, presented clearly as not part of the advisor's amount due.
For older months, derive the customer count from `since` dates; approximate that month's AUM
by scaling `totalAum()` down slightly per month back (a seeded, monotonic curve is fine, add a
`// TODO: replace with real historic AUM` comment).

## 1.4 Downloadable PDF
Make **Download PDF** genuinely download a file, reusing the `buildICS`/`downloadICS` Blob
pattern. Two acceptable approaches, pick one:
- **Preferred:** hand-build a minimal, valid single-page PDF (plain text PDF structure: a
  `%PDF-1.4` header, one page object, a content stream drawing the invoice lines with `BT/ET`
  text operators, an xref table) and download it as `vault22-invoice-<month>.pdf`. Keep it to
  the fields in 1.3. No external libraries (CSP blocks them).
- **Fallback:** open a print-ready invoice view (`window.print()` on a clean invoice DOM) so
  the browser's "Save as PDF" produces the file.
Log an `audit('Downloaded membership invoice', '<month>')` entry on download.

---

# 2. TARA ADVISOR PERSONALISATION (Personalise portfolio)

## 2.1 Entry point
On the customer full profile (advisor view, `vCustomer`), add a **`Personalise portfolio`**
button (in the header action row next to Call / Email / WhatsApp / Book, and/or on the
Investment tab). It opens **Tara full screen**.

## 2.2 Full-screen Tara shell
Render Tara as a **full-screen takeover** (reuse the `body.onboarding` approach: hide the
advisor shell, render into `#onbRoot` or a dedicated `#taraRoot`). Include Tara branding
(reuse the Tara avatar asset `assets/img/tara-advisor.png`) and a **Back** control that returns
to the customer profile (`go('customer', <id>)`). Track state in `state.tara =
{custId, ccy:'ZAR', q:'', total, lines:[...]}`.

## 2.3 Layout
- **Top third (no more than one third of the page): client key info**, read from the customer
  record. Show: name, **age** (compute from `dob` against `NOW`), dependents, monthly income,
  monthly savings, net worth, emergency fund, total savings, and **risk profile + score**
  (from the Risk tab data). Compact, scannable, this is the context Tara reasons from.
- **Total invested** field at the **top of the suggestions**, prefilled with a sensible default
  (e.g. the client's current invested amount `inv`, or `sav * 12`), **editable by the advisor**.
  Changing it recomputes every line's amount.
- **Currency filter**: `ZAR` / `USD` chips (mutually exclusive, default `ZAR`). Filtering shows
  only assets available in that currency on the platform.
- **Search bar** to filter the asset universe by name or ticker.

## 2.4 Asset universe (seed)
Add an `ASSETS` seed, ~20 to 28 rows, clearly illustrative, spanning ETFs, single-name stocks
and a bond/cash sleeve, in both currencies. Each asset:
`{ticker, name, ccy:'ZAR'|'USD', type:'ETF'|'Stock'|'Bond'|'Cash', cls:'Equity'|'Bonds'|'Cash'|'Property', reason}`
where `reason` is a short standalone rationale. Examples (extend to a full set, add
`// TODO: replace seeded asset universe with the platform's real securities list`):
- ZAR: Satrix Top 40 (STX40, ETF, Equity), Satrix MSCI World (STXWDM, ETF, Equity), Sygnia
  Itrix S&P 500 (SYGSP5, ETF, Equity), Ashburton Govi Bond ETF (GOVI, ETF, Bonds), Naspers
  (NPN, Stock), Capitec (CPI, Stock), FirstRand (FSR, Stock), Sasol (SOL, Stock), a ZAR money
  market/cash sleeve.
- USD: Vanguard S&P 500 (VOO, ETF), Vanguard Total World (VT, ETF), iShares Core MSCI EAFE
  (IEFA, ETF), iShares 7 to 10 Year Treasury (IEF, ETF, Bonds), Apple (AAPL), Microsoft (MSFT),
  Amazon (AMZN), a USD cash sleeve.

## 2.5 Tara's suggestion
On open, build a **suggested portfolio** deterministically (no `Math.random`) from the client's
**risk profile / score** and profile:
- Derive **target asset-class weights** from the risk type (reuse the `PORTS` allocation idea,
  e.g. Conservative leans Bonds/Cash, Growth/Aggressive leans Equity). More dependents or a
  thinner emergency fund nudges toward income/cash; a strong fitness score and long horizon
  (younger age) nudges toward equity.
- Pick assets in the selected currency to fill each class, assign integer **% weightings** that
  **sum to 100%**, and compute each line's **amount** = `round(total * weight / 100)`.
- Each suggested line shows: asset (name, ticker, type), **weight %** with **`-` / `+`
  steppers** (step 1%, the advisor tweaks weightings inline), **amount** (recomputes live), and
  a **reason** (compose from `asset.reason` plus a profile-aware clause, e.g.
  `Adds offshore diversification, suited to your Growth profile.`).
- Show a running **total weight** indicator; warn when it is not 100%. The advisor can also
  **add** assets from the browsable universe (filtered by currency + search) and **remove**
  lines. Adding/removing rebalances or leaves the advisor to re-tweak to 100%.

## 2.6 Save and Suggest
- **Save** stores the working portfolio to `state.taraDraft[custId]` (survives navigation, does
  not send). Toast + `audit('Saved a personalised portfolio draft', <name>, custId)`.
- **Suggest** (enabled only when weights total 100%) **sends** the portfolio to the client:
  write it to `state.taraSuggestion[custId] = {total, lines, from: advisor, ts}`, push a client
  notification (extend `state.clientReminders` for `CLIENT_ID`), log
  `audit('Suggested a personalised portfolio', <name>, custId)`, toast, and return to the
  customer profile. Also surface the suggestion status on the customer's Investment tab
  (`Suggested portfolio sent, awaiting the client`).

## 2.7 Client side: receive, tweak, invest
In the **client scenario** (`state.scen === 'client'`, client is `C1`), if
`state.taraSuggestion[CLIENT_ID]` exists:
- Surface it prominently (a card in the **Your Advisor** tab and a client notification):
  `Your advisor suggested a portfolio`.
- The client can **view the suggested portfolio**, **tweak the weightings themselves** (the same
  `-` / `+` steppers, amounts recompute), and then **Invest**.
- **Invest** (prototype): mark it invested (`state.taraSuggestion[CLIENT_ID].invested = true`),
  show a confirmation, `auditShared('Client invested in the suggested portfolio', ...)`, and
  reflect it back so the advisor's Investment tab shows `Client invested`. No real order is
  placed; say so honestly in a note.

---

# 3. WIRING AND SEEDS

- Add all new `data-*` keys (e.g. `data-membership`, `data-invoice`, `data-invoicedl`,
  `data-tara`, `data-taraback`, `data-taraccy`, `data-taraweight`, `data-taraadd`,
  `data-tararm`, `data-tarasave`, `data-tarasuggest`, `data-taratotal`, `data-clientinvest`,
  `data-clienttweak`) to the delegated click-handler selector string, and handle them.
- Keep membership state and Tara drafts/suggestions **surviving scenario switches** where it
  matters for the demo (the same way consent, CRM, docs and portfolios already do in
  `loadScenario`), so a portfolio suggested in the advisor scenario is visible in the client
  scenario.
- Money: `money()` for all Rand figures; the new `usd()` only for the subscription fee.
- Seed: `extAum` per customer; 4 to 6 months of membership invoices; the `ASSETS` universe;
  one saved Tara suggestion for `C1` so the client-side receive/tweak/invest flow demos without
  the presenter having to build one first.

---

# 4. QA CHECKLIST (drive the running app, do not judge by reading source)

**Membership**
- [ ] My Profile shows a Membership plan card with **date joined** and the **current monthly
      fee** (correct USD tier for the customer count).
- [ ] Each monthly **invoice** lists number of customers, total AUM, total non-Vault22 AUM, the
      subscription fee (USD tier), and the 25 bps monthly AUM fee (Rands, paid by customer).
- [ ] **Download PDF** genuinely downloads a file that opens; download is audited.
- [ ] Subscription tiers are correct: 1 to 10 `$10`, 11 to 50 `$25`, 51+ `$40`.

**Tara**
- [ ] `Personalise portfolio` on the customer profile opens Tara **full screen** with a Back
      control that returns to the profile.
- [ ] Client key info sits in the **top third**; the asset list is below.
- [ ] **ZAR / USD** filter shows only that currency's assets; the **search bar** filters.
- [ ] Tara's suggested lines show **weight %** with `-` / `+`, **amount**, and a **reason**
      each; the **total invested** at the top is editable and recomputes amounts.
- [ ] Weights are tweakable and must total 100% before **Suggest**; **Save** stores a draft.
- [ ] **Suggest** sends to the client, notifies them, and is audited.
- [ ] In the client scenario the client sees the suggested portfolio, can **tweak** it, and can
      **Invest**; the advisor's Investment tab reflects the status.

**House rules**
- [ ] Zero em dashes in the rendered DOM (all new screens, both scenarios, both themes,
      light + dark). All customer money comma-grouped Rands; subscription fee in USD only.
- [ ] No `.io` domains, no internal references, no dead controls, no console errors, no
      `undefined` / `NaN` / `[object Object]`.
- [ ] No horizontal overflow at 390px on the new screens (Tara full screen included).
- [ ] Everything from rounds 1 and 2 still works: consent round trip, QR, onboarding, themes,
      portfolio module, analytics period selector, notification settings, document upload.

When every box is checked, summarise what changed and note what is seeded and needs real data
later (Uday's securities universe; final subscription pricing and the 25 bps AUM fee, to be
confirmed with Greg).

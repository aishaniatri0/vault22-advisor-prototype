# Round-6 Fix Prompt — manual-test findings + QA findings, merged

Target: `index.html`, single-file vanilla-JS SPA, build `2026-07-20.7`, deployed at
https://aishaniatri0.github.io/vault22-advisor-prototype/

Two sources merged here:
- **Manual testing** (Aishani) — the report rebuild, landing-page logo, button alignment, portfolio validation.
- **Independent QA** — three demo-breaking defects plus a set of smaller ones.

Every item is written as **How it is now → How it should work**, with the cause already traced, so
this can be worked in one pass. Line numbers are from build `2026-07-20.7`.

## Ground rules
- Architecture unchanged: in-memory `state`, `render()` loop, `openModal`/`.rdrawer`, delegated click
  handler selector list (`~:6814-6822`), `audit(...)`, `sessionStorage` keyed to `BUILD_VERSION`.
- UK spelling · no em dashes in rendered copy · Rands via `money()` · dates via `fmtDate()` (dd-mm-yyyy).
- Bump `BUILD_VERSION` (`:1896`). Verify deployed is byte-identical to local after pushing.
- Do not regress Stephen's 18 requirements — `qa-evidence/` is the baseline. Re-shoot any image whose
  screen changes.
- Verify by driving the UI, not by grepping for markers. Several past rounds passed a grep and failed
  in the browser.

---

# PART A — The financial report (the big one)

## A1. Rebuild the client financial report to the approved design

**Reference material, in order of authority:**
1. **Figma** — `figma.com/design/aW1gdmhSZu1qjkqIYJZvHt/Vault22-Design-System-3.5--In-progress-?node-id=31483-654141`
2. **Dev platform** — `https://global-website.dev.vault22.com/app/dashboard/` (Stephen: the report is
   *"already built in dev"*, so where dev and any other source disagree, **dev wins**)
3. **Approved prototype** — `~/Downloads/Vault22-User-Reports-Prototype/index.html`, plus its proof
   images in `proofs/` (`01-ffl-5-zone-bars.png`, `02-investments-assets-fees-module.png`,
   `03-fee-breakdown.png`, `04-field-formats-card.png`) and `FIELD-FORMATS.md`.

**How it is now.** `tab8Financial()` (`~:3341-3419`) renders a layout invented for this prototype:
four KPI tiles, a fitness progress bar, a four-bar net-worth composition, a cash-flow grid. It is not
the Vault22 report. It was approximated once already in Round 4 and flagged again on manual testing.

**How it should work.** The advisor's view of a client's financial report should be the
**Client Financial Intelligence Report** — the advisor-facing report in the reference prototype — not
a bespoke summary. That prototype contains three distinct reports; take the advisor one:

| Report | Audience | Use here? |
|---|---|---|
| User Profile Report ("My Financial Profile") | the user | no |
| Monthly Spending Report | the user | no |
| **Client Financial Intelligence Report** | **the advisor** | **yes — this is the one** |

Its section order, to reproduce:

> Executive summary · At a glance · Account & Onboarding Profile · Personal profile ·
> Employment & Income · Income verification & Stability · Net Worth ·
> Assets & Investment Profile · Debt Exposure & Liability Schedule · Debt Profile ·
> Budget Allocation (50/30/20) · Financial Fitness & Behavioural Signals ·
> Financial Fitness Level (FFL) · Insurance Coverage · Cover by type ·
> Investments, Assets & Fees · Affordability assessment · Flagged Transactions ·
> Cross-sell opportunities

Requirements:
- **Match the reference visually**, not just structurally — the FFL five-zone bar
  (`proofs/01-ffl-5-zone-bars.png`), the investments/assets/fees module
  (`proofs/02-...`) and the fee breakdown (`proofs/03-...`) are approved components, reproduce them.
- **Populate from the client record.** Where this prototype has no data for a section, render an
  honest empty state ("Not yet collected") rather than dropping the section, so the true shape of the
  report is visible. `FIELD-FORMATS.md` lists exactly which fields are not yet collected and how they
  are meant to be captured — use it for those empty states.
- **`reportPDF()` (`~:3634`) must mirror whatever the screen shows**, section for section. The two are
  deliberately kept in step and that must not regress.
- Keep the existing consent gate: the report stays behind **financial** consent (`consentOf(p.id,'financial')`).

**Accept:** Open a customer → Financial. The report is recognisably the same document as the dev
platform's, same section order and naming, populated with that client's data, with honest empty states
where data does not exist. Download PDF matches the screen.

**If blocked:** if the Figma or dev report cannot be accessed, say so and stop — do **not** approximate
again. That has now failed twice.

---

# PART B — Other issues from manual testing

## B1. Landing-page logo — wrong place, and easy to think it is broken

**How it is now.** The advisor's company logo renders in exactly one spot on the landing page: a small
co-brand strip at the **very bottom** of the page (`vLanding()`, the `.cobrand` block, logo measured at
108×26 px at y≈916 on a 1000px viewport). There is no logo in the header or hero. A prospective client
landing on that page sees a Vault22-branded page with the advisor's logo buried in the footer, and an
advisor who uploads a logo reasonably concludes it did not work.

Related: `vLanding()` renders a hero image from `a.banner || firm.banner` — with no banner uploaded
nothing renders there at all, so the top of the page has no advisor identity of any kind.

**How it should work.**
- The advisor/firm logo belongs in the **top of the landing page** — a header band alongside the
  advisor's name and firm, where a visitor sees it first.
- Keep the footer co-brand strip (that is where the "powered by Vault22" lockup belongs).
- Make the empty banner state look deliberate: if no banner is uploaded, show a styled placeholder
  band carrying the logo and advisor name, not blank space.
- In **My Profile → Customise**, state where each image appears ("Logo — shown at the top of your
  landing page and on your profile card", "Banner — the hero image at the top of your landing page,
  recommended 1200 × 300").

**Accept:** Upload a logo, open the landing page, and it is visible without scrolling. With no banner,
the top of the page still shows the advisor's identity rather than empty space.

## B2. Action-button alignment in the client view

**How it is now.** In the client's suggested-portfolio card, the action column mixes two element types
with **different heights**:
- `Invest` / `Sell` are `<button class="btn btn-sm">` — measured **29px** tall
- `Done` / `Sold` are `<span class="pill p-green" style="min-width:74px;text-align:center">` — **23px** tall

Left/right edges match (both 74px wide, right-aligned), so this reads as a vertical rhythm problem:
once some lines are actioned and others are not, the rows sit unevenly and the pill floats against the
amount/delta stack beside it.

**How it should work.**
- Actioned and un-actioned states must occupy **the same box** — same width **and** height, same
  vertical centring — so a row does not shift when its state changes. Either give the done-state pill
  the button's dimensions, or render it as a disabled button rather than a pill.
- The action control should be vertically centred against the amount + delta stack it sits beside.
- **Then sweep the whole prototype for the same class of problem**: anywhere a control is replaced by a
  status pill after an action, the two must be the same size. Known places to check: registered-user
  and customer table row actions, document rows (upload → uploaded), request loops (Request risk
  assessment → Requested), portfolio submissions (buttons → status pills), the moderation queue, and
  the setup checklist rows.

**Accept:** Action one line in the suggested portfolio and no row moves; the Done pill occupies exactly
the space the Invest button did. The same holds for every other action→status control in the app.

## B3. Add-a-portfolio accepts values it should not

**How it is now** (`addPortDrawer()`, submit handler at `data-portsub`):
- **Minimum investment** is `type="number"` but has **no `min` and no `step`**, so negatives and
  arbitrary decimals are accepted.
- **Ticker** is free text with no pattern — accepts digits, symbols, any length.
- **Only `name` and `ticker` are validated** on submit. Everything else is optional.
- Invalid numeric input is **silently coerced**: `min:+d.min||0` turns junk into `0` and submits it
  without warning. `fee` becomes `null`.
- Nothing tells the user what is required until they press submit, and then only for two fields.

**How it should work.** Use the validation conventions the team already approved in
`~/Downloads/Vault22-User-Reports-Prototype/FIELD-FORMATS.md` ("Number / stepper — min/max bounded",
"Free text — with an explicit max-character count shown to the user"):

| Field | Control | Validation |
|---|---|---|
| Portfolio name | text | required, max 80 chars, trimmed |
| Ticker | text | required, **uppercase A–Z and digits only**, 3–10 chars, pattern-enforced on input |
| Minimum investment | number | required, **integer ≥ 0**, `min="0" step="1"`, thousands-formatted on blur |
| Currency | select | required (ZAR / USD) |
| Exposure | text | required, max 80 chars |
| Risk profile | select | required |
| Fees | number | optional, `min="0" max="10" step="0.01"`, 2dp, shown as % a year |
| Factsheet | file | optional, PDF only, max 10 MB |

- **Validate on submit and show which field failed**, inline against the field — not a single generic
  toast. Never silently coerce an invalid value to `0`.
- Block non-numeric keystrokes in numeric fields rather than accepting and discarding them.

**Accept:** Try to submit with a blank or negative minimum, a lowercase or symbol-laden ticker, or a
fee above 10 — each is refused with an inline message naming the field. No submission ever stores a
silently-zeroed value.

---

# PART C — QA findings

## C1. Applying a Tara portfolio change is silently undone by a scenario switch — [demo-breaking]

**How it is now.** Reproduced through the UI:
```
after applying the plan:        holdings 5, allocation 70/14/10/6, invested = true
after client → advisor → client: holdings 6, allocation 63/23/6/8,  invested = true
```
`state.customers` is re-seeded on every `loadScenario()`, but `state.taraSuggestion` deliberately
survives. So the portfolio silently reverts while the card still says "Invested", the Current holding
column shows pre-investment figures again, and the flow cannot be re-run without a full reset. The
advisor↔client hop is the core demo path, so this will be hit.

**How it should work.** The applied portfolio must survive a scenario switch, exactly as consent
grants, documents and CRM notes already do. Either persist the mutated client alongside those stores,
or re-apply the plan on load when `sug.invested` is true. Whichever way, after a round trip the
holdings, allocation, Current holding column and the card state must all agree.

**Accept:** Apply the plan, switch to advisor and back, and the client still holds the new portfolio
with the card reading Invested. Switch again — still consistent.

## C2. Exporting the audit log ignores the filters — [demo-breaking]

**How it is now.** `vAudit()` filters with substring search on the client name and `auditCategory()` on
the action. The export handler (`~:7164-7166`) kept the **old** predicate — exact-match on both.
Reproduced: filter to `Lerato` → **7 rows on screen, 0 exported**. Select `Risk assessment` →
**4 on screen, 0 exported**, because `af.act` is now a category name that can never equal a raw action
string. The user gets "exported (0 events)" and an empty file.

**How it should work.** One predicate, used by both. Extract the filter from `vAudit()` into a single
function and call it from the view and the export.

**Accept:** Any filter combination shows N rows on screen and exports exactly those N rows.

## C3. "Reset demo" does not clear the portfolio-change record — [demo-breaking]

**How it is now.** The reset block (`~:6842-6848`) clears fourteen state fields — consent, poa, docs,
crm, clientReminders, taraSuggestion, taraDraft, riskReq, riskDates, setup, poaReq, kycReq,
linkedAdvisor, prevAdvisor — but **not `state.portfolioChanges`**. After a reset the client card shows
a dated "R112,000 invested, R80,000 sold" record next to an un-actioned amber "Suggested" pill.

**How it should work.** Add `state.portfolioChanges = {};` to the reset block. A reset must leave no
trace of a previous run.

**Accept:** Apply a plan, reset the demo, and the client card is back to a clean un-actioned suggestion
with no change record.

## C4. The landing/footer logo can show the wrong company's logo

**How it is now** (`~:5699`):
```js
const flogo = la.logo || (state.firm && state.firm.name === la.firm ? state.firm.logo : null) || logoOf(la);
```
By the third clause `la.logo` is already falsy, so `logoOf(la)` reduces to `state.firm.logo` — **the
logged-in user's firm logo, rendered as a different firm's co-brand**. The deliberate same-firm guard
in clause two is defeated by clause three.

**How it should work.** Drop the trailing `|| logoOf(la)`. A different firm's advisor with no logo of
their own should fall back to their firm name as text, never to another firm's logo.

**Accept:** Appoint an advisor from a different firm who has no logo; the client footer shows their
firm name as text, not your logo.

## C5. The audit log's "Invested" filter returns nothing

**How it is now.** `AUDIT_CATEGORIES` matches `Invested` with `/invested|sold/i`, which does not match
the seeded entries "First investment made" (×4) and "Recurring investment started". Five of the six
uncategorised rows are investment events, so clicking **Invested** on a log full of investment activity
returns zero.

**How it should work.** Widen to `/invest(ed|ment)|sold/i`. Then re-check the whole taxonomy against
every action string the app actually logs, and fix the multi-match mis-bucketing — currently
`Requested power of attorney` lands in *Portfolio change* while `Granted power of attorney` lands in
*Consent*, splitting one workflow across two categories.

**Accept:** Every category with matching events returns them; no category that should have events
returns zero.

## C6. The audit client search does not search all clients

**How it is now.** The control is a text input, but it filters **log rows only**. A real client with no
logged activity returns the identical "No matching events" state as a typo — so the user cannot tell
"this client has done nothing" from "you spelled it wrong".

**How it should work.** Resolve the query against all clients (`state.customers.concat(state.leads)`).
On an exact match with no events, show a distinct message — *"Grant Willemse has no recorded activity"*
— rather than the generic empty state. Offer type-ahead suggestions as the user types.

**Accept:** Search a client with no events and get a message naming them; search nonsense and get the
generic empty state.

## C7. Enter in the audit search steals focus

**How it is now.** The `change` listener fires `render()` for any `data-auditf` element that is not
`__clear`. On a text input `change` fires on blur or **Enter**, so pressing Enter repaints the page and
the caret is lost.

**How it should work.** Exclude `who` from the `change` → `render()` path; the `input` listener already
handles it via the live region.

**Accept:** Type in the search box and press Enter — results stay filtered and the caret stays put.

## C8. Cleanup

- **`renderKeepingScroll()` (`~:6721`) is dead code** — defined, never called; the working fix is the
  inline `state.chartScrollY` stash in the chart handler. Remove it and its comment.
- **Two stale comments**: `~:4179` still says the moderation queue "lives behind the admin surface in
  the settings gear"; a Round-4 comment near the chart handler still claims Tara places "no real order
  … state + audit only", which is no longer true.
- **Demo docs lag**: `DEMO-FLOW.md` and `DEMO-WALKTHROUGH.md` both still carry pre-move prose saying the
  moderation queue is behind the settings gear, before the new sections that say otherwise. A reader
  going top-to-bottom hits the old story first.
- **New holdings are written with `u:0, r:0`** and dated `TODAY_STR` (2026-07-17) while the change
  record uses today's real date — so a holding is dated three days before the transaction that created
  it. Give new holdings sensible unit counts and a consistent date.

## C9. Seed-data realism (not a defect, but it will show)

**How it is now.** C3, C4 and C5 each hold a **single fund** yet carry four-way allocations. Running
Tara against any of them produces a "sell everything you own" plan — the exact problem the C1 reseed
fixed.

**How it should work.** Give C2–C5 the same treatment as C1: holdings that use real `ASSETS` tickers,
summing to their `pv`, with allocations that actually describe those holdings, and at least one
untickered fund so a disposal still demonstrates. Then Tara can be demoed on any client.

---

# Blocked / needed
- **A1 cannot be completed without the Figma file or dev-platform access.** Both links are in this
  prompt; if neither opens, stop and report rather than approximating a third time.

# Definition of done
1. Every item satisfies its Accept line, driven in the UI, zero console errors.
2. Stephen's 18 requirements still pass; `qa-evidence/` images re-shot for every screen touched
   (at minimum TC-03 report, TC-09 logo, TC-13 audit, TC-15 Invest/Sell, TC-17 moderation).
3. JS parses; no dead buttons (cross-reference every rendered `data-*` against the delegated selector
   list and the change/input listeners).
4. All five clients' holdings sum to `pv` and allocations sum to 100, in the fresh seed **and** after a
   Tara plan is applied and reset.
5. `BUILD_VERSION` bumped; deployed re-checked byte-identical to local.
6. Independent QA pass before this goes to Stephen.

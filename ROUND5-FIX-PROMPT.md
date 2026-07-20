# Round-5 Fix Prompt — findings from manual testing of build 2026-07-20.5

Ten issues raised from hands-on testing of the deployed prototype. Each item below was traced in
source before being written up, so the diagnosis is the actual cause, not a restatement of the
symptom. Line numbers are from build `2026-07-20.5`.

Two items are **not** simple fixes and need input before implementation — see §Blocked at the end.

## Ground rules
- Architecture unchanged: in-memory `state`, `render()` loop, delegated click handler selector list
  (`~:6599-6605`), `audit(...)`, `sessionStorage` keyed to `BUILD_VERSION`.
- UK spelling · no em dashes in rendered copy · `money()` for Rands · dates via `fmtDate()`.
- Bump `BUILD_VERSION` (`:1885`). Re-verify deployed == local after pushing.
- Do not regress the 18 requirements from Stephen's 19 July email — the evidence pack in
  `qa-evidence/` is the baseline. Re-shoot any image whose screen you change.

---

# P0 — broken, misleading, or dead-ends

## 0.1 Two "Personalise portfolio" buttons on the client Overview — [defect]
`:3270` renders it in the profile **header action row**. `:3965` renders it *again* at the bottom of
the **Overview** tab body. Both are `data-tara`, so the user sees the same action twice on one screen.

This is the same duplication Stephen already asked to remove for Book meeting and Request risk
assessment ("as they are already at the top"). That fix removed those two but left `data-tara` behind.

**Fix:** delete the button at `:3965`. The header instance stays.

**Accept:** Open any customer → Overview shows the KPI tiles with no action button beneath them; the
header still has a working Personalise portfolio.

## 0.2 Closing a full-screen chart jumps back to the top of the page — [defect]
`:7106-7109`:
```js
if(t.dataset.chartfull){
  state.chartFull = t.dataset.chartfull === '__close' ? null : t.dataset.chartfull;
  return render();
}
```
`render()` rewrites `#view` wholesale, so the browser loses scroll position and lands at the top.
Analytics is a long page and the charts are near the bottom, so every open/close round trip costs the
user their place.

**Fix:** capture `window.scrollY` before `render()` and restore it after, for this handler. If other
handlers have the same problem (any in-place toggle that repaints), prefer a small shared helper
(`renderKeepingScroll()`) over repeating the pattern. Note `go()` (`:2002`) deliberately calls
`window.scrollTo(0,0)` on genuine navigation — that is correct and must stay.

**Accept:** Scroll to the charts, open full screen, close it, and the page is exactly where you left
it. Navigating to another page still starts at the top.

## 0.3 Two disconnected "company logo" uploads — [defect, this is the real cause]
This is why the logo behaviour looked wrong on manual testing. There are **two independent logo
fields**, each with its own uploader, writing to different state:

| Where | Control | Writes | Renders on | Remove? |
|---|---|---|---|---|
| **My Profile → Company logo** | `imgPicker('logo')` `:5284` → `data-img` handler `:2080` | `state.advisor.logo` | advisor profile card `:5282`, landing page `:4730`, client footer co-brand `:4649` | yes `:5286` |
| **Firm team → Shared company logo** | `data-firmimg="logo"` `:4799` → handler `:2097` | `state.firm.logo` | only inside the firm drawer `:4797`, plus a fallback for linked advisors `:5609` | **no** |

Upload via **Firm team** and the logo appears on none of the surfaces Stephen asked for, and cannot be
removed. Upload via **My Profile** and it works. Nothing on screen tells the user which is which.

**Fix — decide the model, then make the UI say it.** Recommended: keep both concepts (a firm-wide logo
and an advisor's own) but make the relationship explicit and consistent:
- Both uploaders get a **Remove** control.
- Label them distinctly: "Your company logo" (advisor) vs "Firm logo, shared by everyone at
  {firm name}" (firm).
- Make the fallback explicit and universal: any surface that renders a logo uses
  `state.advisor.logo || state.firm.logo`, so a firm logo actually shows through when the advisor has
  not set their own. Today only `:5609` does this.
- If the simpler answer is that advisors should not have a separate logo at all, collapse to one field.
  Either is defensible; two silently-diverging fields is not.

**Accept:** Upload on either screen and the logo appears on the advisor profile card and the landing
page. Remove works from both. With only a firm logo set, the advisor's surfaces show the firm logo.

## 0.4 Audit log filters are the wrong shape — [rework]
`:5335-5345`. Both filters are `<select>` elements whose options are built **from whatever happens to
be in the log**:
```js
const people  = [...new Set(state.audit.filter(a=>a.who).map(a=>crmWho(a.who)))].sort();
const actions = [...new Set(state.audit.map(a=>a.act))].sort();
```
Two consequences:
- **User name** is a dropdown of only the clients who already appear in the log. You cannot search for
  a client, and anyone with no logged activity is unfindable.
- **Action** is a list that grows every time a new action string is logged, so the filter's vocabulary
  changes as the demo is used and never matches a stable taxonomy.

**Fix:**
- **User name → a search input** (type-ahead over all clients, not just those in the log), matching the
  existing global-search pattern at `:1331`. Filter on partial match.
- **Action → a fixed, standard list** defined as a constant, independent of log contents. Derive the
  taxonomy from what the product actually logs and group the variants, e.g. *Viewed profile · Requested
  access · Consent granted / withdrawn · Document uploaded / viewed · KYC · Risk assessment · Meeting ·
  Portfolio change · Invested · Exported · Signed in*. Map each `audit()` call site to one of these
  categories rather than filtering on the raw free-text string.
- Keep the date filter as-is; it works and correctly compares ISO.

**Accept:** Type part of any client's name and the log filters to them, including clients with no
events (empty state). The action dropdown shows the same fixed list on a fresh session as after heavy
demo use.

## 0.5 Tara Invest / Sell / Action all is a dead end — [incomplete]
The buttons work mechanically (`:7121-7130`): they set `sug.actioned` / `sug.sold`, write an audit
entry, toast, and `taraSettle()` flips the card to Invested once everything is done. But nothing
downstream changes, so it reads as a button that does nothing:
- The client's **holdings do not change** — `CUSTOMERS_EST` is static, so the portfolio they supposedly
  just bought is not reflected anywhere.
- The **Current holding** column still shows the pre-investment figures.
- Portfolio value, AUM, the advisor's book and Analytics are all unchanged.
- There is no confirmation step, no summary of what is about to happen, and no record the client can
  look back at beyond an audit line.

**Fix — make the action have a consequence.** At minimum:
1. **Confirm before executing.** Clicking Invest / Sell / Action all opens a short confirmation showing
   what will be bought and sold, the amounts, and the resulting allocation. Nothing executes until
   confirmed.
2. **Apply the result to state.** Update the client's holdings so the suggested weights become the
   actual ones, disposals are removed, and portfolio value / Current holding / deltas all recompute.
3. **Reflect it on the advisor side.** The advisor's view of that client, their book totals and
   Analytics should move accordingly, so the demo can show the loop closing.
4. **Leave a record.** A dated "portfolio change" entry the client can see under Your Advisor, and the
   matching audit entry the advisor sees.

**Accept:** As the client, action the suggestion. Holdings change to the suggested allocation, Current
holding updates, the disposal is gone, and the advisor's view of that client and their AUM reflect it.

## 0.6 The moderation queue's "admin section" is not discoverable — [requirement not really met]
Stephen asked for the queue to move to "the admin section of project x ... should be a new section".
It currently lives in `protoDrawer()` at `:2244` under a **Vault22 admin** heading — but that drawer is
the prototype settings panel behind the gear, whose own header reads *"Prototype settings · Internal
tool · not part of the product"*. So the queue sits inside something explicitly labelled as not part
of the product, and a tester looking for an admin section will not find it.

**Fix:** this needs a real home, not a stopgap. Two options:
- **Preferred:** a proper **Vault22 admin** area — its own route and left-nav entry, visible only in an
  admin scenario (add one to the scenario switcher), containing the moderation queue as its own
  section. That matches "should be a new section" and is demonstrable.
- **Interim, if the destination is still Uday's call:** keep it behind the gear but move it out of the
  prototype-settings framing into a clearly separate, product-looking panel, and say plainly in
  `DEMO-WALKTHROUGH.md` where it is.

Either way, **write the navigation path into the demo docs** — right now nothing tells anyone how to
reach it.

**Accept:** A tester given no instructions can find the moderation queue. Approving there still flips a
submission to Approved and it appears in the portfolio list.

---

# P1 — layout and design

## 1.1 Global search is not aligned with the content column — [layout]
`:219` — `.gsearch{position:relative;flex:1;max-width:420px;margin:0 var(--v4-space-4)}`. It is a flex
child sitting between the logo and the right-hand controls, so it floats near the centre of the
topbar. The page content column starts at `var(--sidebar-w)` + `var(--v4-space-10)` (`:286`,
`--sidebar-w:264px` at `:103`), so the search bar's left edge does not line up with the client name or
any page heading beneath it.

**Fix:** align the search field's left edge to the content column. The topbar has `padding:0 28px`
(`:209`), so the offset is `calc(var(--sidebar-w) + var(--v4-space-10) - 28px)`. Either give `.gsearch`
that `margin-left` and drop the centring `flex:1`, or lay the topbar out as a grid whose second column
starts at the content edge. Keep it responsive — below the sidebar breakpoint it should go back to
filling the available width.

**Accept:** At desktop width the search bar's left edge lines up exactly with the page `<h1>` and the
client name beneath it. Nothing overlaps at narrow widths.

## 1.2 The financial report does not follow the Vault22 report design — [rework]
The current client Financial tab (`tab8Financial`, `:3327`) is a bespoke layout invented for this
prototype: four KPI tiles, a fitness bar, a net-worth composition bar chart, and a cash-flow grid. The
real Vault22 client report is a substantially richer document.

For reference, the **Client Financial Intelligence Report** in
`~/Downloads/Vault22-User-Reports-Prototype/index.html` is structured as:

> Executive summary · At a glance · Account & Onboarding Profile · Personal profile ·
> Employment & Income · Income verification & Stability · Net Worth · Assets & Investment Profile ·
> Debt Exposure & Liability Schedule · Debt Profile · Budget Allocation (50/30/20) ·
> Financial Fitness & Behavioural Signals · Financial Fitness Level (FFL) · Insurance Coverage ·
> Cover by type · Investments, Assets & Fees · Affordability assessment · Flagged Transactions ·
> Cross-sell opportunities · Monthly Spending Report · Monthly Summary

**Fix:** rebuild the advisor's view of the client financial report to match the real one, section for
section, populated from the client record. Sections with no data in this prototype should render an
honest empty state rather than being silently dropped, so the shape of the real report is visible.
`reportPDF()` (`:3568`) must mirror whatever the screen ends up showing — the two are currently kept in
step deliberately and that should not regress.

**Sources to reconcile before building** (see §Blocked): the Figma file, the dev platform's live
report, and the user-reports prototype above. Where they disagree, the dev platform wins, since Stephen
said the report is "already built in dev".

**Accept:** The advisor's financial report is recognisably the same document as the one in dev, with
the same section order and naming. Download PDF matches the screen.

## 1.3 Invoice formatting can be improved — [polish]
Every element Stephen asked for is present and correct (logo, bill-to name and address, invoice date,
VAT at 15% reconciling, no "Date joined"), so this is presentation, not compliance. Currently it is a
flat left-aligned text column because the PDF writer emits one text run per line.

**Fix suggestions, in rough order of value:**
- A proper header band: logo left, "TAX INVOICE" and invoice number right.
- Two-column block for **Bill to** (left) and **Invoice date / Billing month / Invoice no.** (right).
- The charges as a table with aligned right-hand amounts, and a ruled line above the total.
- **TOTAL DUE** visually dominant.
- Vault22 registration / VAT number in a footer, if there is one to quote.

The writer now supports positioned drawing (it gained `/XObject` and byte-accurate offsets for the
logo), so tables and rules are `re` fills and positioned text runs rather than a rewrite.

**Accept:** The invoice reads as a designed document. All figures still reconcile and no field is lost.

---

# Blocked — needed before P1.2 can be done properly

1. **The Figma link.** Referenced in testing but not in the thread. Without it the report will be
   rebuilt from the user-reports prototype alone, which may not be current.
2. **Access to the dev platform's financial report** — Stephen's own words are "as it is already built
   in dev", so that is the authoritative version. A screenshot set or a walkthrough would be enough.
3. **Where the Vault22 admin section lives in project x** (0.6) — still Uday's call. The preferred fix
   above does not depend on it, but the final destination does.

---

# Definition of done
1. Every P0 satisfies its Accept line, driven in the UI, zero console errors.
2. P1.1 and P1.3 done. P1.2 done if the reference material lands; otherwise raised explicitly as
   blocked rather than approximated again.
3. No regression in the 18 requirements evidenced in `qa-evidence/`. Re-shoot the images for any screen
   touched — at minimum TC-01 (charts), TC-03 (financial report), TC-09 (logo), TC-10 (invoice),
   TC-13 (audit filters), TC-15 (Invest/Sell/Action all), TC-17 (moderation queue).
4. JS still parses; no dead buttons (cross-reference every rendered `data-*` against the delegated
   selector list).
5. `BUILD_VERSION` bumped; deployed re-checked byte-identical to local.
6. Independent QA pass before replying to Stephen.

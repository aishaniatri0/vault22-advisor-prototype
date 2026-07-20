# Round-4 QA Fix Prompt #2

Second independent QA, against build `2026-07-20.2` (deployed byte-identical to local). Every finding
below was traced to source and verified — none is inferred from a comment or a marker.

**Headline: one previous fix is a regression that is worse than the bug it replaced (§A1). Do not send
the build to Stephen until that is resolved.**

## What is confirmed good — do not disturb
P0.1 (Tara ternary), P0.2 (moderation drawer refresh, count recomputed live), P0.3 (`financialReport`
deleted with zero references, `reportPDF` mirrors the designed report with every division guarded),
P1.1/1.2 (C1 reseed exact, tickers verified in `ASSETS`), P1.4 (BILL TO fix + an honest, specific logo
disclosure), and all of P2. Independently confirmed: **JS parses clean, no dead buttons, all five
clients' holdings reconcile to `pv`, and none of the nine previously-passing areas regressed.** The 8
removed selectors were all genuinely dead before removal.

## Ground rules
- Architecture unchanged. Bump `BUILD_VERSION` (`:1874`). Re-verify deployed == local after pushing.
- UK spelling · no em dashes in rendered copy · `money()` for Rands · `vault22.com` never `.io`.
- After each fix, drive the actual UI path. Several items below are things that *looked* fixed.

---

# A — Blocker

## A1. Analytics export reports zeros — [REGRESSED, fix first]
`:6773-6779`. The export and the screen now use **different populations**, so they still disagree — and
the disagreement is worse than the original all-time bug, because the file now carries a period stamp
that makes wrong numbers look authoritative.

- **Screen** (`vAnalytics`, `:4272-4282`) is cumulative **as-of window end**:
  `asOf = (d,end) => d && new Date(d) <= end`, driving `custByEnd`, `signedByEnd`, `aua`, `invTotalEnd`, `conv`.
- **Export** (`:6773-6779`) is **signed-up-inside-the-window**:
  `inWin = d => d && new Date(d) >= win.s && new Date(d) <= win.e`.

Traced on "Last month" (window June 2026, `NOW = 2026-07-17`; no customer signed up in June):

| Metric | Screen | Export |
|---|---|---|
| Registered on website | 7 | 2 |
| Customers | 4 | **0** |
| Conversion | 57.1% | 0.0% |
| Assets under advice | R2,036,000 | **R0** |
| Total invested | R1,687,000 | **R0** |

There is also a third semantic in play: `inWin(c.signup||c.since)` short-circuits on `signup`, which is
always populated, so `since` is dead in that expression — while the screen's `newCust` (`:4308`) uses
`inWin(c.since)`.

**Fix — do not patch the predicate, remove the duplication.** Extract the metric computation out of
`vAnalytics` into a single function, e.g. `analyticsFor(period)` returning
`{win, signedByEnd, custEnd, custByEnd, aua, invTotal, conv, bySource, newCust, ...}`. Call it from both
`vAnalytics` and the export handler. Two code paths computing the same advertised number will drift
again; one cannot.

Note that **each metric keeps its own correct basis** — this is not "make everything `asOf`":
- Headline population, AUA, invested, conversion → **as-of window end** (cumulative).
- Acquisition by source and "new customers this period" → **signed-up-within-window**. `bySource`
  already matches the screen (both `inWin`); preserve that.

Also restore `TODAY_STR` to the export filename alongside the period slug — dropping it means two
exports of the same period on different days collide.

**Accept:** For each of the five periods, export and compare every headline figure against the screen.
They match exactly. The filename carries both period and date.

---

# B — Incomplete fixes

## B1. Six more dates bypass `fmtDate` — [INCOMPLETE]
The ten previously-known sites are genuinely fixed and no ISO regression was introduced (ICS
`DTSTART`/`DTEND`, the audit date filter, sort keys and every `<input type="date">` verified intact —
leave those alone). But an independent sweep found six more.

| # | Line | Site | Renders |
|---|---|---|---|
| 1 | `:3306` | Advisor client Personal tab, `['Signup date', p.signup \|\| p.since]` | `2025-08-12` |
| 2 | `:3307` | Same array, `['Registration date', …]` | `2025-08-12` |
| 3 | `:5580` | `clientPreview()`, `['Risk Assessment Completed', lastRiskDate(c) …]` | raw ISO |
| 4 | `:6742` | Client meeting reminder, `m: day + ' at ' + slot` | raw ISO |
| 5 | `:6732`, `:6735`, `:7131` | `audit(...)` targets, `who.name + ' · ' + day + ' at ' + slot` | raw ISO in Audit Log Target column and the CSV |
| 6 | `:1699` | Seed literal `tgt:'2026-06-02 at 15:00'` | raw ISO |

Two things worth internalising about how the last pass missed these:
- **#1 and #2 sit seven lines below the DOB fix at `:3299`, in the same array literal.** The sweep fixed
  one row of a table and walked past two others.
- **#3's advisor-side twin at `:3246` was fixed and the client-side one was not.** Several date facts
  render on both sides; fixing one side is the recurring failure mode here.

For #5, format at the `audit()` call sites rather than at the renderers — `a.tgt` is free text and
`:5296` / `:6890` pass it through unformatted by design.

**Judgement call, decide rather than skip:** `:4191` renders analytics chart x-axis labels as `21 Jul`.
Chart axes conventionally abbreviate and `dd-mm-yyyy` will not fit at that width. Recommend leaving it
and noting it, rather than silently having a third format in the app.

**Accept:** Walk both scenarios end to end. No `yyyy-mm-dd` reaches the user anywhere, including the
Audit Log Target column, the audit CSV, and the client's meeting reminder. ICS files still open in a
calendar app.

## B2. Tara card consistency — inconsistency relocated, not removed — [INCOMPLETE]
`:7041-7056`. `data-tarainvest` and `data-tarasell` set `actioned` / `sold` but never re-check whether
everything is now done, so `sug.invested` stays false. Only `Action all` (`:7071`) and `clientinvest`
(`:7078`) set it. Result: action every row individually → every row reads Done/Sold while the card header
still shows the amber "Suggested · total 100%" pill and still offers both **Action all** and
**Invest R980,000**. That is the same defect P1.3 claimed to fix, moved from row level to card level.

Two more in the same handler:
- `sug.invested = true` (`:7071`) sits **outside** both loops, so clicking Action all when nothing remains
  flips the card to Invested, toasts the now-false "Everything is already actioned", and writes **no audit
  entry at all**. A state change with no compliance record.
- `Action all` never writes the portfolio-level `auditShared('Client invested in the suggested portfolio', …)`
  that `clientinvest` writes (`:7079`), so the log shows line items and no portfolio-level invest record.

**Fix:** derive the invested state at render time rather than storing it in three places —
`invested = sug.invested || (lines.every(l=>actioned[l.ticker]) && disposals.every(h=>sold[h.p]))` — or set
it at the end of the `tarainvest`/`tarasell` handlers. Move `sug.invested = true` inside the `n > 0` branch
and emit the portfolio-level audit entry alongside it.

**Accept:** Action every row one at a time; the card flips to Invested and stops offering actions. Click
Action all on an already-complete card; nothing changes state and nothing is audited. Every path that
sets Invested leaves a portfolio-level audit entry.

## B3. Un-appointing an advisor is silently reverted — [INCOMPLETE]
`:1858`: `state.linkedAdvisor = state.linkedAdvisor || ADVISOR_DIR[0]`. The un-appoint path (`:7175-7185`)
sets `state.linkedAdvisor = null`, and `null` is indistinguishable from "never set" to `||`. So: remove
advisor → switch to the advisor scenario → switch back → `ADVISOR_DIR[0]` is silently re-appointed, with
consents already wiped and an audit trail reading "Removed advisor".

The handler's own comment states the rule this breaks:
> "Unlinking also withdraws every grant — an unlinked advisor must not keep access — and that is exactly
> why re-linking cannot silently restore it."

**Fix:** distinguish "never set" from "deliberately cleared". Either test `=== undefined`, or carry an
explicit `state.advisorChosen` sentinel. `null` must mean *no advisor* and survive the round trip.

**Also:** Reset demo (`:6552-6565`) clears `consent`, `poa`, `docs`, `crm`, `taraSuggestion` and more, but
never clears `linkedAdvisor` — so a reset leaves a manually appointed advisor standing while wiping the
consents it promises to clear, contradicting its own confirm text. Add `linkedAdvisor` to the reset (to
`undefined`, not `null`, so the scenario default reapplies).

**Accept:** Remove advisor, round-trip scenarios, and no advisor is appointed. Appoint a different
advisor, round-trip, and the choice holds. Reset demo genuinely restores the seeded state.

---

# C — Issues introduced or exposed by this round

## C1. Consent list no longer matches what gates the report — [New, from P0.3]
P0.3 re-gated the financial report onto **financial** consent (`:3318`, `consentOf(p.id,'financial')`) —
which is the right gate for financial data. But the client-facing consent UI still advertises the report
under **Investment**: the field list at `:1725` and the rendered row at `:5569`
(`['Financial profile report', 'Your full report']`).

So the toggle the client sees is not the toggle that controls access. A client could grant Investment
believing they are sharing their report, or revoke Investment believing they have withdrawn it.

**Fix:** move `'Financial profile report'` out of the `investment` field list (`:1725`) into the
`financial` list (`:1722`), and move the corresponding `clientPreview()` row from the investment block
(`:5569`) into the financial block. Verify the preview and the actual gate then name the same section.

**Accept:** Toggling Financial consent off hides the report and the client's preview says so. Toggling
Investment off does not affect the report.

## C2. C1's allocation no longer describes C1's holdings — [New, from P1.1]
`:1592` still reads `{Equity:62, Bonds:18, Cash:8, Property:12}`, but the reseeded holdings are two equity
ETFs, a bond ETF and an income fund — no cash or property instrument at all. Nothing breaks (alloc sums to
100, holdings sum to `pv`), but the allocation bars and the Products-owned list render on the same screen
telling different stories. The old opaque `V22 …` names concealed this; explicit tickers expose it.

**Preferred fix — one reseed that also completes the Tara story.** `STXPRO` (Satrix Property) and `ZARMM`
(Vault22 ZAR Money Market) are already in `ASSETS` and are exactly the two suggested lines currently
showing `R0` current holding. Seeding them gives C1 a coherent allocation *and* makes all five suggested
lines meaningful, while keeping one untickered fund so the Sell/disposal path still demos:

| Holding | tk | mv | Class |
|---|---|---|---|
| Satrix Top 40 | `STX40` | 460,000 | Equity |
| Satrix MSCI World | `STXWDM` | 320,000 | Equity |
| Ashburton Govi Bond ETF | `GOVI` | 200,000 | Bonds |
| Satrix Property | `STXPRO` | 100,000 | Property |
| Vault22 ZAR Money Market | `ZARMM` | 80,000 | Cash |
| V22 Income Fund | *(none)* | 80,000 | → disposal |

Sum = 460 + 320 + 200 + 100 + 80 + 80 = **1,240,000** = `pv` ✓
Allocation = Equity 780/1240 = 63%, Bonds (200+80)/1240 = 23%, Property 100/1240 = 8%, Cash 80/1240 = 6%
→ `{Equity:63, Bonds:23, Cash:6, Property:8}` = 100 ✓

Check the knock-on: `productInvested()` (`:4017`) matches `h.p === p.n`, so fund-level "total invested"
figures shift with any reseed. Confirm the Portfolios page still reads sensibly afterwards.

**Accept:** C1's allocation bars agree with their Products-owned list; all five suggested lines show a
non-zero current holding; exactly one disposal remains; holdings still sum to `pv`.

## C3. Suggested amounts read as nonsense beside current holdings — [Residual from P1.1]
`sug.total` seeds from `_c1.inv` (980,000 — contributions) while the client's book is `pv` (1,240,000). So
a row renders **"Invest R411,600"** directly beside **"Current holding R520,000"**, with no delta language
— read literally it takes STX40 to R931,600 inside a R980,000 portfolio.

**Fix:** seed `sug.total` from `pv` rather than `inv`, and/or render a signed delta (`+R92,000` /
`−R108,400`) instead of a bare target amount, so the two numbers in a row tell one story. This is the
difference between the column being *present* and being *readable*.

---

# D — Small, cheap, do them while you are in there

- **`:5695-5696` stale comment** now contradicts the code it annotates: *"Status reflects the suggestion,
  never 'Invested'."* directly above the corrected ternary that renders "Invested". Delete it — this is
  precisely the comment that drives the next regression.
- **`:3506` invoice name line** is still unconditional, so a pre-registration `ADVISOR_NEW`
  (`name:''`, `licence:''`) emits one blank line in BILL TO. Same defect class as the fixed `company` /
  `address` lines; low reachability.
- **`:1251` stale comment** `/* P1.6: advisor directory grid */` now sits above the `.tara-cur` rule.
- **`data-status` (`:7452`)** is registered with no render site, so the account-status control is
  unreachable; **`data-firm` (`:7134`)** is shadowed by the `[data-firm-radio]` change listener. Both
  pre-existing, not this round's damage — clean up or note.

---

# E — Resolve, do not ask

**Q2 has an answer now.** The comment at `:1720-1721` asserts the fee figure is *"deliberately NOT in the
client's sharing list or the client-facing report"*. That is true only of the exact string
"Total fees earned". `'Fees earned'` is still in the Investment field list (`:1726`) and still renders a
live figure to the client at `:5577`. So the code comment misleads a future reader about what the client
can see.

Either remove `'Fees earned'` from the client-facing Investment section too — which matches Stephen's
stated reasoning, "doesn't make sense here" — or narrow the comment to say exactly which field was
removed and that the Investment one deliberately remains. Do not leave the comment as-is.

Q1 (is "Invested" acceptable as a genuine post-action state) and Q3 (confirm the audit-log filters back to
him) remain genuine questions for Stephen.

---

# Definition of done
1. **A1 fixed and verified across all five periods** — this is the ship blocker.
2. B1–B3 fixed, each verified by driving the UI path, not by grep.
3. C1–C3 fixed, or explicitly listed in the reply to Stephen as known and deferred.
4. No regression in the confirmed-good list at the top — re-run the nine-area sweep, the dead-button
   cross-reference, and the holdings-reconcile-to-`pv` check for all five clients.
5. JS still parses (`jsc` on the extracted script block).
6. `BUILD_VERSION` bumped; deployed re-checked byte-identical to local.
7. **Re-QA'd by someone other than the author before the reply goes out.** Two rounds running, the
   author's self-report has been more optimistic than the code: round one reported "all 20 items
   implemented and working" against 2 FAIL / 8 PARTIAL, and round two reported six P1 items fixed
   against 1 regression / 3 incomplete.

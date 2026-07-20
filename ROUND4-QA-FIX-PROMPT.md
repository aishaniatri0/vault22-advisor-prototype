# Round-4 QA Fix Prompt

Independent QA of build `2026-07-20.1` (deployed byte-identical to local) against Stephen's 19 July
21:33 email. Result: 9 requirements PASS, 2 FAIL, 8 PARTIAL. Everything below is a **verified** finding
— each was traced to the render path or handler and confirmed in source, not inferred from a marker or
a comment. Line numbers are from build `2026-07-20.1`.

What is already sound and must not regress: charts 2×2 + full-screen (`spark()` genuinely re-renders at
`h=260`), the fees removal from the client view, the New User rename and WhatsApp channel removal
(session key protects the rename), all three Registered Users items, the company-logo dual-surface fix,
the Change advisor removal, and the advisor-facing Approve/Reject removal. JS parses clean and there are
no dead buttons — keep it that way.

## Ground rules
- Architecture unchanged: in-memory `state`, `render()` loop, `openModal`/`.rdrawer`, the delegated click
  handler selector list (`~:6529-6535`), `audit(...)`, `sessionStorage` keyed to `BUILD_VERSION`.
- No em dashes in rendered copy · Rands via `money()` · `vault22.com` never `.io` · UK spelling.
- **Bump `BUILD_VERSION`** (`:1864`) so stale sessions cannot survive these changes.
- Re-verify with the same method QA used: drive the UI, confirm zero console errors, and for each fix
  satisfy its Accept line literally.

---

# P0 — fix before Stephen sees the build

## 0.1 Tara status pill: inverted ternary — [FAIL → fix]
`index.html:5695`. The ternary is backwards, so an **invested** portfolio renders the pill "Suggested",
directly above the paragraph "You invested in this portfolio. Your advisor can see it in their portal."
The comment on the line above (`:5694`) states the intended rule, so the code contradicts its own spec.

```js
/* now  */ <span class="pill p-amber">${sug.invested ? 'Suggested' : 'Total '+sugWeight+'%'}</span>
/* want */ <span class="pill ${sug.invested?'p-green':'p-amber'}">${sug.invested ? 'Invested' : 'Suggested · total '+sugWeight+'%'}</span>
```

Note the tension with Stephen's ask: he said the card must **not** say "Invested" when a portfolio is
merely *suggested*. He did not ask to hide a genuinely invested state. Restoring "Invested" for
`sug.invested === true` is the correct reading — see §Q1 if you want it confirmed before shipping.

**Accept:** A freshly suggested portfolio shows "Suggested · total 100%" in amber. After investing, the
pill reads "Invested" and no longer contradicts the paragraph beneath it.

## 0.2 Moderation Approve/Reject looks like a no-op — [PARTIAL → fix]
`index.html:7085-7098`. Both handlers call `render()` and return **without** `reopenProtoIfOpen()`.
`render()` only rewrites `#view`; the drawer lives in `#protoDrawer` and is written solely by
`openProto()` / `reopenProtoIfOpen()` (`:5388-5390`, `:7477-7478`). On stage: the presenter clicks
Approve, gets a toast, and the open drawer still shows "Pending approval" with live buttons and a stale
"N awaiting review" count.

Every sibling in-drawer handler already does this correctly — copy the pattern at `:6544-6546`:

```js
/* now  */ render(); toast(...); }  return;
/* want */ render(); reopenProtoIfOpen(); toast(...); }  return;
```

Apply to **both** `data-modq` (`:7085`, the Reject action) and `data-approveport` (`:7091`).

While here: `data-modq` is a misleading name for Reject. Rename to `data-rejectport` (update the
selector list at `:6535`) unless that risks churn.

**Accept:** With the drawer open, click Approve → the row flips to "Approved" **in the open drawer**
without reopening it, the awaiting-review count decrements, and the portfolio appears in the advisor's
portfolio list. Same for Reject.

## 0.3 The "quite basic" financial report is still on screen — [PARTIAL → fix]
Stephen's words: *"Could you use the financial report you designed (as it is already built in dev) and
this version is quite basic."*

`tab8Financial()` (`:3311-3386`) is a genuine redesign and its maths is correctly guarded. But the old
flat 7-row `dl()` list survives as `financialReport()` (`:3626-3646`) and is **still rendered**, under
the heading "Financial profile report" at `:3619-3620`, inside the client's **Investment** tab. So the
exact artifact he called basic is one tab away from its replacement, and the two contradict each other.

**Preferred fix — one report, not two:**
1. Delete the `Financial profile report` heading and the `${financialReport(p)}` call at `:3619-3620`.
2. Move the **Download PDF** button (`data-reportdl`, currently inside `financialReport()` at `:3644`)
   onto the designed report in `tab8Financial()`, so the download stays reachable.
3. Delete `financialReport()` itself once nothing calls it — do not leave it orphaned (see §2.3, we
   already have one orphan from this round).
4. Upgrade `reportPDF()` (`:3518-3537`) so the downloaded PDF reflects the designed report's sections
   (headline position, net worth composition, cash flow and resilience) rather than the old 7 flat lines.

If you would rather keep a report on the Investment tab, then `financialReport()` must render the
**designed** component, not the flat list — but two copies of the same report in one profile is worse.

**Accept:** The client profile contains exactly one financial report, the designed one. No flat
label/value financial report renders anywhere. Download PDF still works and its contents match the
on-screen report.

## 0.4 `dd-mm-yyyy` is not applied throughout — [FAIL → fix]
Stephen bolded *"throughout the app"*. `fmtDate()` (`:1875-1879`) is correct, and ISO is correctly
preserved for storage, sort keys and `<input type="date">` (`:4424-4426`, `:3709`, `:5286`) — do not
touch those. The failure is ten render sites that bypass it.

**Raw ISO leaking to the user:**

| # | Line | Site | Renders |
|---|---|---|---|
| 1 | `:5660` | Client scenario, own personal details | `${c.dob}` → `1991-03-19` |
| 2 | `:3293` | Advisor's client profile, Date of birth | `p.dob \|\| dash` |
| 3 | `:3849` | CRM Follow-ups pill | `${f.d}` |
| 4 | `:6198` | `icsModal` "Meeting booked" | `${day} at ${slot}` |
| 5 | `:6707` | Booking invite email body | `'portfolio review for ' + day` |
| 6 | `:7289` | Document preview PDF | `'Uploaded: '`, `'Expiry: '` |
| 7 | `:6787` | Analytics PDF | `'Generated ' + TODAY_STR` |
| 8 | `:6880` | Audit CSV `When` column | raw `a.ts` |

Items 1 and 2 are the worst — a client reading their own date of birth in ISO.

**Competing formatters (not dd-mm-yyyy):**
- 9. `:4251` — `fmtD()` produces `3 Apr 2025` and is rendered via the Custom period label at `:4261`,
  which propagates across Analytics as `${win.label}`. **This was introduced by this round** (its
  comment is tagged `R41`), i.e. the round that mandated dd-mm-yyyy added a second date format six lines
  from the period code. Delete `fmtD` and use `fmtDate`.
- 10. `:6035`, `:6073` — booking headings render `Monday, 21 July` / `Mon 21 July` via
  `DOW_FULL`/`MONTHS_FULL`. **Judgement call:** a human-readable day name in a booking UI is arguably
  correct and not what Stephen was complaining about. Recommend keeping the weekday but appending the
  dd-mm-yyyy date (`Monday, 21-07-2026`), so the rule holds without making the booking flow read oddly.

**Also decide:** the audit log (`:5293`), notifications (`:5380`), notes (`:3777`, `:3830`) and the
customers table (`:3159`) render `rel()` (`:1473`), an unbounded "200d ago". Not strictly a format
violation, but the audit log shows relative times directly above an ISO date filter. Recommend the audit
log's When column shows `fmtDate(a.ts)` with `rel()` as secondary text.

**Accept:** Walk every screen in **both** scenarios and confirm no `yyyy-mm-dd` and no `3 Apr 2025`
reaches the user. `grep -nE '[0-9]{4}-[0-9]{2}-[0-9]{2}'` over rendered template strings returns only
storage/sort/`<input type="date">` uses. Sorting and the audit date filter still work.

---

# P1 — fix if there is time before the reply

## 1.1 `currentHoldingOf` is structurally dead — [PARTIAL]
`:4912-4918` matches on `x.tk === ticker` or a case-insensitive name match. Seeded client holdings have
**no `tk` field at all** (`:1591-1593`; `tk:` exists only in the PORTFOLIOS catalog) and are named
`V22 Global Equity Fund` etc., while `ASSETS` (`:4769-4791`) are `Satrix Top 40`, `Sygnia Itrix S&P 500`.
Zero overlap by construction, so the Current holding column returns **R0 on every row, for every client,
always**. The requirement is satisfied structurally and is unfalsifiable in practice.

**Fix:** seed a real overlap so the column can demonstrate itself. Give at least one client (C1 Lerato,
the only client reachable in the client scenario) holdings that carry `tk` values present in `ASSETS` —
e.g. `{p:'Satrix Top 40', tk:'STX40', mv:…}`. Keep the totals reconciling with `pv`.

**Accept:** C1's suggested portfolio shows a non-zero Current holding on at least two lines, `R0` on
genuine new additions, and the holdings still sum to `pv`.

## 1.2 The disposal model implies liquidating the whole book — [PARTIAL, design]
Consequence of the same zero-overlap problem. `taraDisposals()` (`:4920-4923`) keeps every holding whose
name is not in the suggestion, so **every** suggestion renders every existing holding as a Sell. C1 shows
R980,000 suggested in against R1,240,000 of disposals out — an unexplained R260,000 gap, and advice that
reads as "sell everything you own".

Fixing §1.1 largely fixes this: with real overlap, only genuinely-dropped holdings become disposals. After
seeding, re-check that the suggested total and the disposal total tell a coherent story, and that the
Sell list is a subset, not the entire portfolio.

**Accept:** For C1, disposals are a strict subset of holdings, and the card's in/out figures reconcile or
carry an explanatory line.

## 1.3 Tara card state is inconsistent after actioning — [PARTIAL]
Three related defects in the same card:
- Per-row Invest (`:4944`) and Sell (`:5720`) render regardless of `sug.invested`, while the card-level
  `Action all` / `Invest Rx` block is gated on `!sug.invested` (`:5722`). After a full invest the card
  says done but every row still offers Invest.
- `Action all` does not set `sug.invested`, so actioning every row leaves the card still showing
  "Total 100%" and still offering "Invest R980,000".
- A green **"Invested"** pill appears per-row at `:4945` as the Invest done-state. Post-action rather
  than default, but it is the exact string and colour Stephen asked to remove — see §Q1.

**Accept:** After Action all, the card and every row show a consistent completed state; no row still
offers an action that has been taken.

## 1.4 Invoice PDF has no Vault22 logo — [PARTIAL]
Stephen asked for the logo. The **drawer** renders a real `<img>` (`:3551`) so it looks done on screen,
but `invoicePDF()` (`:3489`) emits the plain text `'V  VAULT22'`, and `downloadPdf()` (`:3461`) builds a
PDF whose only resource is `/F1 Helvetica` — there is **no `/XObject` image support in the file at all**
(`grep -c XObject` → 0). The artifact the advisor actually sends has a typed wordmark where the logo
should be.

**Fix options, in order of preference:**
1. Add minimal `/XObject` image support to `downloadPdf()` and embed the logo. Correct, more work.
2. Render the invoice to canvas and emit an image-based PDF.
3. If neither is worth it before the meetings: leave the text wordmark but **say so in the reply to
   Stephen** rather than letting him discover it. Do not claim the logo is in the PDF.

Also `:3497-3498` push `a.company || ''` and `a.address || ''` unconditionally, so a new advisor
(`ADVISOR_NEW`, `:1532`, `address:''`) gets two blank lines mid BILL-TO. The drawer guards these
correctly at `:3556-3557`; mirror that.

**Accept:** Either the downloaded PDF carries the logo, or the gap is explicitly disclosed. No blank
lines in BILL-TO for a new advisor.

*(The VAT work is correct and needs no change: 15% on the subscription fee only, `.toFixed(2)`
neutralising the `25 × 1.15 = 28.749999…` float hazard, reconciling at all three tiers, with the R335
AUM fee correctly excluded from both the VAT base and the total.)*

## 1.5 Appointing an advisor does not survive — [PARTIAL]
`:1848`: `if(s === 'client'){ state.advQ=''; state.linkedAdvisor = ADVISOR_DIR[0]; }` unconditionally
resets the appointed advisor on **every** switch into the client scenario. Since switching scenarios is
the only way to see the advisor side, any appointment is wiped on the round trip. The only surviving
cross-side effect is the audit line at `:7178`.

**Fix:** only default `linkedAdvisor` when it has never been set (`state.linkedAdvisor = state.linkedAdvisor || ADVISOR_DIR[0]`),
so a deliberate appointment persists. Keep `state.advQ=''` (clearing the search box is right).

**Accept:** Appoint a different advisor, switch to the advisor scenario and back — the appointment holds,
and the client's "Your Advisor" tab still names the chosen advisor.

## 1.6 Analytics export ignores the period selector — [PARTIAL]
`:6770-6791`. Export CSV and Download report recompute from `state.customers` / `state.leads` wholesale
(`L.length`, `C.length`, `aua = C.reduce(...)`, `bySource` from `[...L,...C]`) with no window filter, so
selecting "Last month" shows scoped figures on screen and exports all-time figures — from buttons sitting
in the same header as the period chips.

**Fix:** apply the same `win` filter the on-screen metrics use, and stamp the period into the export
header and filename.

**Accept:** Select "Last month", export, and the file's totals match what is on screen and name the period.

---

# P2 — cleanup, no user-visible risk

- **`.pc` column too narrow.** `bar()` (`:3327`) passes `money(pv)` (`R1,240,000`, ~75px) into
  `.arow .pc{width:42px}` (`:713`). Flex `min-width:auto` means it will not wrap, but it overflows and
  steals width from the bar, so the composition bars render short and uneven. Widen `.pc` for the
  currency variant, or add a modifier class.
- **Orphaned handler from the directory move.** `data-dircity` (`:2093-2095`) is rendered nowhere
  (`grep -c` → 0); it was live in `c17e3d4` and its select was removed in `b907730`. `state.dirCity` is
  now write-only. Delete the handler and the dangling comment at `:7552` that describes removed code.
- **8 phantom selectors** registered at `:6535` with no render site and no handler: `data-appt`,
  `data-avail`, `data-availset`, `data-editland`, `data-nsave2`, `data-portcancel`, `data-psearch`,
  `data-tarasrch`. Harmless (a `closest()` miss falls through) but misleading. Remove.
- **`.tara-cur`** (`:4940`, `:5716`) has no CSS rule; styling is inline only. Either add the rule or drop
  the class.
- **Rejected submissions cannot be re-approved** — the button block at `:2230` is gated on
  `status==='Pending approval'`. Fine for a demo; know it before someone clicks Reject on stage.
- **Docs.** The relocated advisor directory appears only in `DEMO-REALVSMOCK.md:17-18`; add it to
  `DEMO-FLOW.md` and `DEMO-WALKTHROUGH.md` — it is one of the two screens that moved this round and is
  currently undemoed. Add the moderation-queue-final-home blocker to `DEMO-WALKTHROUGH.md:80-83`, whose
  caveat list omits it.

---

# Open questions for Stephen — ask, do not assume

**Q1 — the word "Invested".** He asked that a *suggested* portfolio not show the green "Invested" label.
Two places still use it legitimately: the card pill once the client has actually invested (§0.1), and the
per-row done-state after clicking Invest (§1.3). Confirm he is happy for "Invested" to appear as a
genuine post-action state, or whether he wants that word gone from the card entirely.

**Q2 — "Fees earned" on the Investment tab.** §2.2 is implemented correctly as written: `'Total fees
earned'` is out of the Financial consent list (`:1714-1716`) and the client's financial report, and the
advisor's own block (`:3373-3383`) is intact. But `'Fees earned'` (singular) remains in the **Investment**
consent section (`:1719`) and still shows the client a live fee figure at `:5575`. Strictly he said
"financial reports", so leaving it is defensible — but it is the same number in front of the same person,
and his reason was "doesn't make sense here". Worth one line in the reply.

**Q3 — audit-log filters.** Not in `STEPHEN-ROUND4-PROMPT.md`, but his 19 July message re-raises it on
page 34 (he first asked on 18 July): *"Audit log – can you add the following filters in the recent
activity section so can filter by user name, action, date"*. It **is** implemented (`vAudit`, `:5262`) with
user/action/date selects, Clear filters, and a correct ISO comparison against `<input type="date">`. No
work owed — just confirm it back to him so the re-raise is visibly closed.

---

# Definition of done
1. Every P0 satisfies its Accept line, driven in the UI, zero console errors.
2. P1 items either fixed or explicitly listed in the reply to Stephen as known and deferred. Do not ship
   a silent gap — §1.4 in particular looks done on screen and is not.
3. No regression in the nine PASS areas listed at the top, especially the charts, the New User rename and
   the three Registered Users fixes.
4. JS still parses (`jsc` or `node --check` on the extracted script block) and there are still no dead
   buttons — re-run the delegated-selector cross-reference after removing the phantom selectors.
5. `BUILD_VERSION` bumped; deployed file re-checked as byte-identical to local.
6. Q1–Q3 raised with Stephen in the reply rather than decided unilaterally.

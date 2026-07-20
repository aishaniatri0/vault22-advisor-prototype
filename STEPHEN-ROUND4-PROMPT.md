# Round-4 Build Prompt — Stephen's "95% there" punch list

Source: Stephen Ong, 19 July 2026 21:33 (thread *Advisor Dashboard*), reviewing
`https://aishaniatri0.github.io/vault22-advisor-prototype/`. His words: **"this is great we are 95% there."**
Everything below is the remaining 5%. Nothing here is a new feature area — it is renames, moves,
removals and finishing on screens that already exist.

Two items from the 18 July round are still open and are carried forward at the bottom (§8).

## Ground rules (unchanged from Round 3)
- Architecture stays: in-memory `state`, `render()` loop, `openModal`/`.rdrawer`, the delegated click
  handler (register every new `data-*` in the selector list), `audit(...)`, `sessionStorage` keyed to
  `BUILD_VERSION`, `Reset demo` in the settings gear.
- No em dashes in rendered copy · Rands via `money()` · `vault22.com` never `.io` · UK spelling ·
  no bare `Date.now()`/`new Date()` outside the existing helpers.
- Do not regress any prior P0–P3 QA fix, the booking redesign, the reassessment loop, Tara, or the
  first-run checklist.
- Tags: **[New]** build it · **[Partial]** finish/extend existing · **[Verify]** likely present, confirm.
  Line numbers are pointers into `index.html` at time of writing, not guarantees.

---

## 1. Analytics and charts

### 1.1 Chart grid: 2 across × 2 high, with full-screen — [Partial]
> "charts make them 2 across and 2 high with a full screen button (when they are all full width it
> doesn't show the variation that well)"

The Analytics charts currently stack full-width, which flattens the curves. Lay them out as a
**2 × 2 grid** (Growth in registered users and customers · AUM growth · Investment trends · Customer
trends). Each chart card gets a **full-screen button** in its top right that expands that one chart to a
takeover with the same data at full height, and a close control back to the grid.
- Re-render the chart at the larger size so the line actually gains resolution. Do not just CSS-scale it.
- Grid collapses to one column under the existing mobile breakpoint.

**Accept:** Analytics shows four charts in a 2 × 2 grid; clicking full-screen on any one opens it large,
closing returns to the grid, no console errors.

### 1.2 Date format `dd-mm-yyyy` app-wide — [New]
> "Date format should always be dd-mm-yyyy **throughout the app**."

Emphasis is his. This is global and includes both advisor and user scenarios.
- Add a single display helper (e.g. `fmtDate(iso)` → `18-07-2026`) and route **every rendered date**
  through it: audit log, activity timeline, invites sent, meetings, documents and expiry, last risk
  assessment, customer `since`/joined, invoice date, notification timestamps, portfolio submissions.
- Keep `localISO()` as the internal/ISO storage and sort key. Only the display layer changes.
- Sweep for raw `.slice(0,10)`, template-interpolated ISO strings, and any `2026-07-18`-shaped literal
  in rendered copy.

**Accept:** Grep the rendered DOM across every screen in both scenarios; no `yyyy-mm-dd` is visible to
the user anywhere. Sorting by date still sorts correctly.

### 1.3 Analytics carry-overs — [Verify]
Confirm the 18 July items landed and did not regress: "How your book is related" renamed to
**customer acquisition**; advisor-name box replaced by a **registered on website** box; the *Monthly
growth in registered users / customers* chart sits at the bottom with the other charts but **above**
Customer trends; a period selector (this month, last month, last 90 days, YTD, custom) at top right
drives **all** metrics on the page.

---

## 2. Client profile

### 2.1 Use the real financial report — [Partial]
> "Could you use the financial report you designed (as it is already built in dev) and this version is
> quite basic"

The client **Financial** tab is currently a flat label/value list (`~:5459`). Replace it with the richer
financial report design already built in dev — same structure, sections and visual treatment as the
Vault22 user financial report, populated from the client's data.
- If a design reference is needed, take it from the existing user-side report / `reference/`; match it,
  do not invent a third layout.

**Accept:** Opening a client's Financial tab shows the designed report, not the basic list, with that
client's real numbers.

### 2.2 Remove "Total fees earned" from the user view — [Partial]
> "remove total fees earned from user view of the financial reports — doesn't make sense here"

The advisor keeps it. Remove it from anything the **client** sees.
- Drop `'Total fees earned'` from the consent/sharing field list (`~:1704`) and from the user-scenario
  financial report. Leave `feesFor()` and the advisor-side "Total fees earned" block (`~:3298`) intact.

**Accept:** User scenario → Profile → What your advisor can access → Financial lists no "Total fees
earned" row; the advisor's own fees view is unchanged.

### 2.3 Notifications: rename and de-scope — [Partial]
> "notifications – rename new lead to new user and remove whatsapp (as we don't have they yet)"

- Rename the notification type **New Lead → New User** everywhere it renders: the notification list
  (`~:1611`), the settings toggle list (`~:1652`, `~:1660`), and the runtime signup notification
  (`~:6774`). Keep the internal id stable or migrate it cleanly so persisted state does not break.
- Remove the **WhatsApp** channel chip from every notification row in notification settings. Keep
  Email, Push, Telegram. Leave the WhatsApp *contact* button on client profiles alone — he only meant
  the notification channel.

**Accept:** No "New Lead" string renders anywhere; no WhatsApp chip in notification settings; toggling
a renamed notification still persists.

---

## 3. Registered Users

### 3.1 Filter label `any` → `all` — [Partial]
> "Registered users > For the filter please rename **any** to **all**"

(Round 3 added "all" to the left of "qualified"; this is the remaining `any` label.)

**Accept:** The Registered Users filter row reads "all", no "any".

### 3.2 Remove duplicated actions below the boxes — [Partial]
> "Remove the book meeting and request risk assessment below the boxes (as they are already at the top)"

On the client **Overview** (the KPI box strip), remove the duplicate **Book meeting** and **Request risk
assessment** buttons underneath the tiles. Both already live in the profile header action row.

**Accept:** Overview shows the tiles with no duplicate action buttons; the header actions still work.

### 3.3 Move "Request all data access" to the top — [Partial]
> "Move request all data access to the top with the other actions"

Move `data-reqallconsent` (`~:3215`) out of the Overview body and into the profile **header action
row**, alongside Call / Email / Book / Request risk assessment / Request KYC refresh. Keep its existing
conditional visibility (only when some consent is not granted).

**Accept:** "Request all data access" renders in the header action row and still triggers the consent
request loop plus audit entry.

---

## 4. My Profile

### 4.1 Company logo on both surfaces, and removable — [Partial]
> "in my profile > add company logo – it adds to the advisor landing page but not the advisor profile
> card. Should add to both. Also add the ability to remove a company logo"

The logo picker (`~:4602`, `~:5063`) currently only feeds the landing page.
- Render the uploaded company logo on the **advisor profile card / preview profile** too.
- Add a **Remove logo** control next to the uploader; removing clears it from **both** surfaces and
  falls back to the current no-logo layout without breaking spacing.

**Accept:** Upload a logo → it appears on the landing page **and** the advisor profile card. Remove it →
it disappears from both, layout intact.

### 4.2 Invoice formatting — [Partial]
> "could you try to format the invoice better – add vault22 logo, client name, client address, invoice
> date, remove date joined, and VAT added"

Rework the generated advisor invoice (`~:3407`) into a properly laid-out document:
- **Add:** Vault22 logo · client name (the advisor being billed) · client address · **Invoice date** ·
  a **VAT** line (SA VAT at 15%, shown as its own line with subtotal → VAT → total).
- **Remove:** "Date joined".
- Keep: invoice number, billing month, number of customers, total AUM, total non-Vault22 AUM, the
  subscription tier fee, and the note that the 25 bps AUM fee is billed to the customer and is not part
  of the advisor's amount due.
- Invoice date in `dd-mm-yyyy` per §1.2.

**Accept:** Download the invoice → it carries the logo, client name and address, an invoice date, a VAT
line that arithmetically checks out against the subtotal and total, and no "Date joined".

---

## 5. User side — finding an advisor

### 5.1 Move "Browse advisors" into the user's Your Advisor tab — [Partial]
> "could you move this page to the user side > move it to the right tab of your advisor and call it
> **search for an advisor**"

Today `Browse advisors` is an **advisor**-scenario nav item (`~:1903`, `~:1947`, `~:5277`). It belongs to
the **user**.
- Move it into the user scenario, inside **Profile → Your Advisor**, titled **Search for an advisor**.
- Keep the directory itself as-is: cards with name, firm, FSP licence, services, location, search/filter.
- Selecting an advisor flows into the existing **Appoint** action (per the earlier rename of
  link/Linked → Appoint/Appointed).
- Remove it from the advisor-side left nav unless it also serves a purpose there; if it does, say so
  rather than silently keeping both.

**Accept:** User scenario → Profile → Your Advisor shows "Search for an advisor"; searching filters;
selecting appoints, and the appointment reflects on the advisor side.

### 5.2 Remove the old "Change advisor" section — [Partial]
> "remove the **change advisor** section as it will be replaced by the **search for an advisor** above"

Delete the `Change advisor` block (`~:5674`) and its search input. The "Appoint an advisor" empty state
is superseded by §5.1 as well — one entry point only.

**Accept:** No "Change advisor" section renders; changing advisor is done solely through Search for an
advisor.

---

## 6. Tara — advisor-suggested portfolio, user view

### 6.1 Remove the "Invested" label — [Partial]
> "When an advisor suggests a portfolio > user view – it should not say invested (green label) please
> remove."

A *suggested* portfolio has not been invested in. Remove the green **Invested** pill from the
"Your advisor suggested a portfolio" card in the user view. If a status pill is needed there, it should
reflect the suggestion state (e.g. **Suggested**), never "Invested".

**Accept:** A freshly suggested portfolio shows no green "Invested" label in the user view.

### 6.2 Per-asset Invest / Sell plus Action all — [New]
> "Please add a button on the right of each asset which says '**invest**' for any new additions, and any
> assets which are being suggested to sell should have a **sell** button and an **action all button**"

On the user's suggested-portfolio card, each asset row gets an action on the right:
- New/increased holdings → **Invest**
- Holdings Tara suggests reducing or exiting → **Sell**
- One **Action all** button at the card level that applies every row's action in one go.
Each action logs to the audit trail and updates the row to a done state. Prototype-level execution is
fine; the flow must not dead-end.

**Accept:** A suggested portfolio shows Invest on additions and Sell on disposals; individual actions
work; "Action all" applies them all and each is audited.

### 6.3 "Current holding" column — [New]
> "Add a column which shows **current holding**"

Add a **Current holding** column to the suggested-portfolio table showing what the client holds in that
asset today (`R0` where it is a new addition), so the suggested weighting reads against the current one.

**Accept:** Every asset row shows a current holding value; new additions read R0; the numbers reconcile
with the client's portfolio value.

---

## 7. Portfolios — moderation belongs to Vault22, not the advisor

### 7.1 Move the moderation queue to the admin section — [Partial]
> "When Advisors add their own portfolio > we need the moderation queue to the admin section of
> project x. (ask Uday where we should add it) should be a new section."

The **Vault22 moderation queue** (`~:3956`) currently sits inside the advisor's Add-a-portfolio screen.
It is a Vault22 admin function.
- Remove it from the advisor view.
- The advisor keeps: submit a portfolio, and see their own submissions with a read-only status
  (Pending approval / Approved / Rejected).
- **Open question for Uday:** where the moderation queue lives in project x. Until he answers, keep the
  queue implemented behind a clearly-labelled admin/presenter surface (settings gear → "Vault22 admin")
  so the demo can still show a submission going live, and note the dependency in `DEMO-REALVSMOCK.md`.

**Accept:** Advisor's Add-a-portfolio screen shows only their own submissions and statuses; the
moderation queue is reachable only from the admin surface.

### 7.2 Remove Approve / Reject from the advisor view — [Partial]
> "Remove the approve / reject button as it is not for advisors to moderate / approve new portfolios on
> our platform but Vault22 admin."

Remove the `data-approveport` / reject buttons (`~:3966`) and the explanatory caption (`~:3970`) from the
advisor view. The handlers (`~:6906`, `~:6914`) move with the queue to §7.1's admin surface.

**Accept:** No advisor-facing Approve or Reject control exists anywhere; approving from the admin
surface still flips a submission to Approved and it appears in the portfolio list.

---

## 8. Carried forward from 18 July (still open)

### 8.1 Full SA portfolio list — blocked on Uday
> "We should have all the portfolios we offer investors in SA … **get full list from uday**"

The Portfolios table still shows only the five model portfolios. Needs the real list from Uday, in the
same table style, with: total invested column, portfolio name first, a **view** button opening a
right-hand layer with fund details, and a **search bar**. Chase Uday; if the list has not arrived, seed a
clearly-marked placeholder set and flag it in `DEMO-REALVSMOCK.md`.

### 8.2 Membership pricing — blocked on Greg
> "start with 1-10 : $10, 11-50: $25, 50+: $40 (we can test this price point with advisors) …
> (To be confirmed with Greg)"

Tiers are implemented. Treat the numbers as provisional until Greg confirms; keep them in one constant
so a change is a one-line edit.

---

## Definition of done
1. Every section above satisfies its Accept line, driven in the UI, zero console errors.
2. `§1.2` verified by sweeping **both** scenarios screen by screen, not by spot check.
3. No regression in: booking, reassessment loop, Tara personalisation, consent/POA, audit export,
   first-run checklist, global search.
4. `DEMO-FLOW.md`, `DEMO-WALKTHROUGH.md` and `DEMO-REALVSMOCK.md` updated wherever a screen moved
   (§5.1 and §7.1 both change the click path).
5. `BUILD_VERSION` bumped so persisted `sessionStorage` state does not carry stale notification ids.
6. Reply to Stephen with the deployed link and a short list of what changed, calling out §7.1 and §8.1
   as waiting on Uday and §8.2 on Greg.

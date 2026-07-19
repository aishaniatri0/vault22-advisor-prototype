# MASTER IMPLEMENTATION PROMPT — Vault22 Advisor Prototype (Stephen feedback, round 2)

> **How to use this file:** paste everything below the line into a fresh Claude Code
> session opened on this repo. It is self-contained — the implementing session does
> **not** need to read the email thread. Execute all changes in one pass.

---

You are a senior frontend engineer implementing a founder's review feedback on a
high-priority, demo-ready prototype. Work entirely inside
`/Users/aishaniatri/Vault22-Advisor-Prototype/index.html` — a single-file, vanilla-JS,
no-build SPA (~4,400 lines). It is live at
`https://aishaniatri0.github.io/vault22-advisor-prototype/` and auto-updates when
pushed. **Read `index.html` in full before changing anything.**

Do everything in this document in one go. Do not stop to ask follow-up questions —
every decision you need has been made below. Where the founder's feedback conflicts
with an earlier instruction, the instruction in THIS document wins (it already
reflects the latest word in the thread).

## Non-negotiable house rules (these have been flagged repeatedly — do not break them)
- **No em dashes** anywhere in rendered UI copy. Use commas, "to", or restructure.
  Gate on the rendered DOM, not just source. (Comments in code are fine.)
- **Money uses comma grouping**: `R2,450,000`. Never `toLocaleString('en-ZA')` (that
  groups with spaces). No decimals on dashboard money.
- **Demo emails/domains use `vault22.com`**, never `.io`.
- **UK spelling**: personalised, customise, unrealised, colour, licence (noun).
- **No internal/dev/meeting references** in the UI (no "stand-up", "prototype note",
  "TO CONFIRM", reviewer names).
- **Every control lands somewhere real.** No dead buttons; a stub must say so honestly.
- **Right-hand panels shrink the page beside them (no dark-scrim pop-ups).** This is
  already the pattern via `.rdrawer` / `body.dw-open`; keep using it for the few
  drawers that remain.
- **Pluralise correctly** ("1 customer", not "1 customers").
- **Never build dates from `toISOString()`** (rolls back a day in SAST/IST). Use the
  existing `localISO()` helper.
- **Preserve working functionality** unless this document explicitly replaces it:
  the QR encoder, the consent round-trip (client grant → advisor tab unlocks), region
  select, onboarding, two-way binding. Prioritise **demo readiness** over engineering
  completeness. Use realistic seeded data wherever live data is unavailable.

---

# IMPLEMENTATION STRATEGY — do the work in this exact order

Complete each phase before starting the next. This ordering minimises rework because
later phases depend on the structural decisions in earlier ones.

1. **Structural / navigation changes** — remove the customer-detail right-hand drawer;
   fold Onboarding stage, Audit log into full-profile tabs; add the new "Permissions"
   left-nav item; add tabs to the client profile.
2. **Theme & settings changes** — user-vs-advisor navy toggle + light/dark toggle,
   wired globally.
3. **Profile / tab consolidation** — advisor customer §8 profile (add Onboarding +
   Audit tabs, merge activity timeline); client profile top-nav tabs.
4. **Document upload workflows** — advisor side and user side.
5. **Portfolio module redesign.**
6. **Landing page redesign** + move Preview into My Profile with Customise drawer.
7. **Analytics redesign.**
8. **Risk-assessment reminder workflow.**
9. **Final copy / label cleanup** (all renames listed in one place at the end).
10. **QA pass** against the checklist at the bottom.

---

# 1. PROJECT CONTEXT

This is the Vault22 **Financial Advisor Platform** prototype. Advisors self-onboard,
get a branded landing page + real QR code, share it, and every signup is tagged to
them. Registered Users who make a first investment become Customers. The advisor gets
a dashboard, client management (Personal / Financial / Investment / Risk tabs, each
consent-gated by the user), CRM, notifications, analytics, portfolios, and an audit
log. There is a separate **client-side** app where a user links an advisor and
controls what each of the three data sections shares.

Two demo scenarios via the settings/gear drawer: **New advisor** (full onboarding) and
**Established** (Aishani Atri, populated). A **client** scenario renders the user app.
Brand green `#01C38D`; navy hero `#12283F` with a full navy ramp already in `:root`
tokens. Data model is all in-memory JS seeds cloned into `state` by `loadScenario()`.

**Key code anchors** (approximate; confirm on read):
- `state` object ~1322; `loadScenario` ~1332; `VIEWS` map ~3918; `render()` ~3922.
- Advisor views: `vDashboard` ~2104, `vLeads` (Registered Users) ~2342,
  `vCustomers` ~2427, `vCustomer` (full detail page) ~2755, `tabs8` ~2683,
  `vRisk` ~2789, `vPortfolios` ~2826, `vAnalytics` ~2907, `vLanding` ~3075,
  `vProfile` ~3155, `vInvites` ~3844, `vAudit` ~3217.
- Client app: `clientShell` ~3419, `vClientProfile` ~3525, Financial Advisor section
  ~3620, consent toggles ~3660, POA toggle ~3693.
- Drawers: `openModal/closeModal` ~3315/3327, `bookDrawer` ~3727, `buildICS` ~3791,
  `nprefsDrawer` ~3892.
- Data seeds ~1134–1410 (`LEADS_EST`, `CUSTOMERS_EST`, `NOTIFS_EST`, `AUDIT_EST`,
  `CONSENT_SECTIONS`, `PORTS` ~2782, `ADVISOR_DIR` ~3380, `INVITES_SEED` ~3838).
- Tokens `:root` from ~line 26 (`--green`, `--hero`, navy ramp `--v4-navy-*`).
- Helpers: `dl()` ~2502, `money` ~1121, `rel` ~1124, `localISO` ~1394, `audit()` ~1402.

---

# 2. GLOBAL UI CHANGES

### 2.1 View-theme toggle (Vault22 green ⇄ navy)
Add a control (in the top bar, near the existing region/scenario controls) that
switches the **advisor** shell accent between the current green theme and a
**navy-forward** theme, so the advisor view is visually differentiated from the user
view. Implement as a `state.viewTheme` = `'green' | 'navy'`, applied by toggling a
class on the app root (e.g. `body.theme-navy`) that remaps the accent CSS variables
(sidebar active state, hero, primary buttons, active chips) to the navy ramp already
in `:root`. The user (client) app stays green; the advisor app is the one that can go
navy. Default: green. Persist within the session (no backend).

### 2.2 Light / dark mode toggle
Add a **light / dark** toggle available to both advisors and users. Implement
`state.mode` = `'light' | 'dark'`, applied via `document.documentElement.dataset.theme`
or a `body.dark` class. Add a dark palette: dark surfaces, adjusted text, borders, and
card backgrounds; keep green/navy accents legible on dark. Every screen (advisor +
client + onboarding) must render correctly in both modes with **no illegible text and
no white flashes**. Default: light. Put both toggles together in a small settings
cluster so the demo can flip them live.

---

# 3. NAVIGATION CHANGES

### 3.1 Remove the customer right-hand side window
When an advisor selects a Registered User or Customer, go **straight to the full
profile page** (`vCustomer`). **Delete the intermediate right-hand side window** that
currently shows Stage + CRM. Selecting a user must be a single step to the full
profile. (The `.rdrawer` mechanism itself stays for Book-a-meeting, notification
settings, and the new Customise/portfolio-detail drawers — only the customer
stage/CRM slide-over goes.)

### 3.2 Fold former drawer content into full-profile tabs
The full customer profile gets these tabs (see §4 for detail):
`Personal · Financial · Investment · Risk · Documents · Onboarding · Activity · Audit`
- **Onboarding** tab = the former right-hand "Stage" content (renamed, see §3.3).
- **Audit** tab = this client's audit log (moved off any drawer into the profile).

### 3.3 New left-nav "Permissions" section
The **top section of the current Audit Log page is actually Permissions** ("Permissions
in force", "Power of attorney"). Move it out of Audit Log into its **own new left-nav
item labelled `Permissions`** under the COMPLIANCE group. The Audit Log page keeps only
the activity log (see §11). Add `permissions` to `NAV`, `VIEWS`, `ROUTE_LABEL`, and a
`vPermissions()` view.

### 3.4 Client profile top navigation (see §5)
The client profile page is too long. Add a **top tab navigation**:
`Personal Information · Your Advisor · Documents library`.

---

# 4. ADVISOR DASHBOARD CHANGES

### 4.1 Dashboard copy / stats
- Rename the **"Your next actions"** block heading to **`To-do list`**.
- Next to **Assets under advice**, add a grey sublabel **`(x% increase from last
  month)`** styled like the other stat sublabels. Seed a realistic figure
  (e.g. `+4.3% from last month`) derived from `PREV_PERIOD` if available.

### 4.2 Customer full-profile tabs (`vCustomer` / `tabs8`)
- Add an **Onboarding** tab: the 10-stage lifecycle (Registered → Ready to Invest)
  with each stage marked Done / current / not-yet. **Rename the "current" state label
  to `Pending`** (the founder's words: "rename 'current' to 'pending', 'submitted' or
  'in review' as current doesn't mean anything"). Use **`Pending`** as the single
  chosen replacement, applied everywhere a stage shows the in-progress state
  (Registered Users table status included).
- Add an **Audit** tab: this client's own activity log (client-level audit, advisor
  view), reusing the audit row treatment.
- **Merge the Activity timeline** so it no longer duplicates the Onboarding stages:
  the Activity tab shows only post-onboarding, recurring events (first investment,
  SIP started, risk assessment last completed, portfolio review last completed, notes,
  meetings). Onboarding-stage milestones live only in the Onboarding tab.
- Add any **missing Personal fields** from the existing live profile so the new profile
  is a superset (§8 Personal list: First/Last/Customer Name, Email, DOB, Country,
  Nationality, Dependents, Phone, Contact, Home owner/tenant, Signup Date, Registration
  Date, Status, KYC Status, Last Login Date).
- **Show a worked example** of the user's **financial report**, and make **Investment**
  and **Risk** viewable by the advisor (seed granted-consent sample data for the demo
  customer so these tabs render fully populated rather than locked).

### 4.3 Risk-assessment controls on the customer profile (see §16)
On each customer's Risk tab: a **`Request risk assessment`** button and a **`Last risk
assessment: <date>`** line.

---

# 5. CLIENT PROFILE CHANGES

### 5.1 Top-nav tabs
Break the long client profile into tabs: **`Personal Information` · `Your Advisor` ·
`Documents library`**.
- **Personal Information** = the profile-completion card + Personal details + Account
  rows (existing content, plus any missing fields to match the advisor superset).
- **Your Advisor** = the existing Financial Advisor section (linked advisor, search +
  link/appoint, the three consent toggles, power of attorney).
- **Documents library** = the user document upload workflow (see §6.2).

### 5.2 Appoint / Appointed rename
Everywhere the client links an advisor, **rename "Link / Linked" to `Appoint /
Appointed`** (button label `Appoint`, status pill `Appointed`). Applies to the client
Financial Advisor section and any advisor-directory row.

### 5.3 Request meeting (see §7.2)
From the client's Your Advisor view, add a **`Request meeting`** button (with their
appointed advisor and other advisors) that opens the same editable-email flow as the
advisor's Book meeting.

---

# 6. DOCUMENT MANAGEMENT CHANGES

Build the same upload UX in two places (advisor customer profile, and client profile):

### 6.1 Advisor — customer Documents tab
Add an **`Upload document`** button and a **drag-and-drop area**. On adding a file the
user gives it a **name** and picks a **document type** from a dropdown:
`Identity · Agreement · Proof of Address · Statements · Other`. Show uploaded docs in a
list (name, type, date). Prototype-only: keep uploaded docs in `state` (array on the
customer), no real storage; accept the file, read its name, render the row. Log an
audit entry on upload.

### 6.2 Client — Documents library tab
Identical component: **`Upload document`** button + drag-and-drop + name +
document-type dropdown (same five types). Store on the client record in `state`.

---

# 7. MEETING WORKFLOW CHANGES

### 7.1 Advisor Book meeting → editable email
In the existing Book-a-meeting drawer, change the confirm action so **"Send invite"
opens an editable email containing the calendar invite**, letting the advisor edit
before sending. Show a compose view (To, Subject, Body prefilled with meeting details)
with the ICS attached/embedded, and an editable body, then a Send action. Keep the real
`buildICS()` / RFC-5545 output. Button copy: **`Send invite`** → opens **`Compose
email`** view with **`Send`**.

### 7.2 Client Request meeting → same flow
The client `Request meeting` button (see §5.3) opens the **same editable-email compose
flow**, addressed to the chosen advisor.

---

# 8. NOTIFICATIONS & SETTINGS CHANGES

Replace the notifications **"Thresholds"** button with **`Settings`**, opening a page
(or right-hand drawer) where the advisor **selects which notifications they want**.
Include all of these, each with an on/off toggle, and an editable **amount** field
where noted:
- New Lead
- Risk Assessment Completed
- Investment over $X — *advisor sets the amount*
- KYC Completed
- First Investment
- Payment Failed
- Birthdays
- Portfolio Review Due
- x% change in portfolio — *advisor sets the percentage*

Add a note that more can be added later. Wire toggles/amounts into `state.nprefs` /
`NOTIF_PREFS` so the notification list respects them (an off type does not appear; the
amount/percentage thresholds filter events as they already do via `notifShown`).
Keep existing threshold logic, just move it behind this settings surface.

---

# 9. QUALIFICATION FILTER CHANGES

### 9.1 Qualified-filtering settings page
When the advisor edits the qualified-user-filtering setting, open a **settings page**
where they set the qualification thresholds:
- **Minimum net worth**
- **Minimum monthly savings**
- **Minimum income**
- **Minimum fitness level**
Wire these into `QUAL` / `isQual()` so the Registered Users list and the dashboard
qualified count recompute live.

### 9.2 Filter chip
On the Registered Users filter row, add an **`All`** chip to the **left of `Qualified`**
(so the order reads `All · Qualified · Unqualified · ...`). `All` clears the
qualified/unqualified split.

---

# 10. ADVISOR PROFILE & LANDING PAGE CHANGES

The current "Landing Page" screen is really an **advisor profile card**, and the founder
wants a proper public landing page instead. Restructure as follows.

### 10.1 Move Preview into My Profile
- **Move the Preview (profile card) section out of the Landing Page screen and into the
  `My Profile` section.**
- In My Profile, add an **`Edit` button in the top right of the preview**, and **rename
  the preview to `Preview profile`** (Vault22 will build a directory of all advisors).
- Selecting `Preview profile` (or Edit) opens the **Customise** panel **in a right-hand
  layer** (`.rdrawer`) so the page does not scroll endlessly.

### 10.2 Customise panel — add all missing fields
The Customise section must include:
- **Services offered**
- **LinkedIn**
- **Website URL**
- **Email**
- **Phone**
- **FSP licence**
- **Description**
- **Upload banner** — label must include the recommended size in brackets, e.g.
  `Upload banner (recommended 1200 x 300)`
- **Advisor profile picture** upload
Plus the existing Website name, Welcome message, and Theme colour. Two-way bind all
fields to `state.advisor` so the Preview updates live.

### 10.3 Redesign the actual Landing Page
Rebuild the **Landing Page** screen as a real public page:
- **Right side = an image the advisor uploads** (the banner/hero image).
- **Left side = how they describe their business** — using the **same fields as the
  advisor profile** (name, description, services offered, contact info, FSP licence,
  LinkedIn/website).
- It is the page **new users land on to register or get in touch with the advisor** —
  include a **register / Start Investing CTA** and a **Get in touch / contact** action.
- Keep the QR code, referral URL, referral code, and share buttons (WhatsApp, Email,
  Facebook, LinkedIn, Copy Link).

---

# 11. ADVISOR INVITATION WORKFLOW

On the **Invite advisors** screen, add the ability to **invite advisors who work at the
same firm**:
- Invite form fields: email (required), **First name / Last name (optional)**.
- Add a **radio button: "Is this advisor at the same firm?"** (Yes / No).
- **If Yes**, show a **dropdown: `Admin` or `Member`**.
  - **Admin** can control the **landing page** (so all firm advisors share the same
    company page) but **not** the individual advisor profile card.
  - Add a **tooltip** on the role dropdown explaining admin rights, e.g. *"Admins can
    edit the firm's advisor landing page. Members cannot."*
- Keep the existing referral link / QR / invites-sent list. Seed a couple of same-firm
  invites (one Admin, one Member) in `INVITES_SEED` for the demo.

---

# 12. AUDIT LOG & PERMISSIONS CHANGES

### 12.1 Split Permissions out
As in §3.3: move **Permissions in force** and **Power of attorney** into a new
left-nav **`Permissions`** page. The **Audit Log** page keeps only **Recent activity**.

### 12.2 Audit log filters
On the Audit Log **Recent activity** section, add filters so it can be filtered by:
**`User name` · `Action` · `Date`.** Implement as three controls (a name search/select,
an action select, a date filter) that filter the `state.audit` rows live.

---

# 13. CLIENT TABLE & INVESTMENT DATA CHANGES

On the **Customers** table (and matching columns on Registered Users where applicable):
- **Rename "Investor Type" to `Risk profile`.**
- **Rename "Unrealised G/L" to `Profit/Loss`.**
- **Add a `% change` column.**
- **Add a `Net worth` column.**
Keep the table within width (the tables were trimmed to fit 1440px before — rebalance
columns so adding two does not cause horizontal overflow; drop or combine a low-value
column if needed, and verify at 390px there is no overflow).

---

# 14. PORTFOLIO MODULE REDESIGN

The Portfolio section needs a full rework.

### 14.1 Full portfolio list
Show **all portfolios Vault22 offers investors in SA**, not just the current five, in
the **same table style** as the current model-portfolio table.
- **Seed a realistic full list** (~12 to 18 rows) of SA-appropriate funds/portfolios,
  since the live list from Uday is not yet available. Add a code comment:
  `// TODO: replace seeded portfolio list with Uday's full SA product list`.
  Use realistic names spanning risk bands (e.g. Vault22 SA Savings, Vault22 Income
  Fund, Vault22 SA Balanced Fund, Vault22 Global Equity Fund, Vault22 High Conviction,
  plus recognisable SA-market style names like Ninety One Diversified Income, Allan
  Gray Balanced, Coronation Balanced Plus, Satrix Top 40, Prescient Income Provider,
  Nedgroup Core Global, etc. — clearly seeded, not real advice).
- **Put portfolio name first (leading column).**
- Add a **`Total invested`** column.
- Add a **`View` button** per row that opens a **right-hand layer** showing the **fund
  details** (like the current bottom-right fund detail card): description, best-for,
  minimum investment, currency, performance chart, fact sheet, and the **target
  allocation** (moved here, see 14.3).
- Add a **search bar** above the table (there will be many portfolios over time).

### 14.2 Add-portfolio (advisor-submitted) flow
Add an **`Add portfolio`** section/flow. Advisors submit portfolios their clients are
invested in, with fields:
**Portfolio name · Ticker · Minimum investment amount · Currency · Exposure · Risk
profile · Fees factsheet** (same fields as the internal add-portfolio form). On submit:
mark it **`Submitted for approval`**, and note it only appears in their platform **once
approved** (Vault22 moderates). For the demo, seed one portfolio in a
`Pending approval` state and show the status. Log an audit entry on submit.

### 14.3 Move target allocation
**Remove the Target allocation block from the main area** and place it **inside the
right-hand fund-detail layer, below the chart** (matching the current live platform's
portfolio description position).

---

# 15. ANALYTICS REDESIGN

On the **Analytics** page:
- **Rename "How your book is related" to `Customer acquisition`.**
- **Remove the advisor-name box** and **add a `Registered on website` box** so advisors
  understand how they are getting their users.
- **Move the "Monthly growth in registered users / customers" chart to the bottom**
  with the other charts, **but above Customer trends.**
- Add a **period selector at the top right** with options:
  **`This month · Last month · Last 90 days · YTD · Custom`.**
  **All metrics on the page must follow the selected period** (recompute every KPI,
  funnel, and chart against the chosen window; seed period-appropriate values so
  switching visibly changes the numbers). Default: `This month`.
- Keep the existing analytics content (conversion, average investment, top products,
  AUM growth, customer retention, customer trends: change in portfolios / savings /
  liabilities / net worth).

---

# 16. RISK ASSESSMENT REMINDER WORKFLOW

- On each customer (Risk tab and/or customer row), add a control for the advisor to
  **`Request risk assessment`**. Triggering it **notifies the user on the Vault22
  platform to redo the risk assessment** (prototype: push a notification into the
  client-side notification feed and log an audit entry; show a confirmation toast).
  Context: users normally redo this once a year.
- Show a **`Last risk assessment: <date>`** value **by each customer** (on the Risk tab
  and, space permitting, on the customer list). Seed realistic last-assessment dates.

---

# 17. SEEDED DEMO DATA REQUIREMENTS

Use realistic, clearly-illustrative seed data (never real client data). Specifically:
- **Portfolios:** full SA list per §14.1, plus one `Pending approval` advisor-submitted
  portfolio.
- **Documents:** a few seeded docs (name + type + date) on the demo customer and the
  demo client so the Documents workflows show populated lists.
- **Consent:** grant Financial/Investment/Risk on the demo customer so the advisor's
  Investment and Risk tabs and the financial-report example render fully (§4.2).
- **Notifications settings:** sensible defaults (most on; amount thresholds e.g.
  investment over `R100,000`, portfolio change `5%`).
- **Qualification thresholds:** seed defaults (e.g. min net worth `R500,000`, min
  monthly savings `R8,000`, min income, min fitness) matching the current dashboard copy.
- **Invites:** at least one same-firm Admin and one same-firm Member invite.
- **Risk assessment dates:** a last-assessment date per customer; at least one due/overdue.
- **Analytics:** period-appropriate figures for each of This month / Last month / Last
  90 days / YTD so the period selector visibly changes results.
- **Assets-under-advice delta:** a seeded month-on-month percentage.
Keep all money in comma-grouped Rands, all dates via `localISO()`.

---

# 18. FINAL QA CHECKLIST (verify every item before declaring done)

Drive the running app (do not judge by reading source). Confirm:

**Structural / navigation**
- [ ] Selecting any Registered User or Customer goes **straight to the full profile**;
      the old customer stage/CRM right-hand window is gone.
- [ ] Full customer profile shows tabs: Personal, Financial, Investment, Risk,
      Documents, **Onboarding**, Activity, **Audit**.
- [ ] Activity timeline no longer duplicates onboarding stages.
- [ ] New **Permissions** item exists in the left nav; Audit Log page shows only Recent
      activity; Permissions page shows Permissions in force + Power of attorney.
- [ ] Client profile has top tabs: Personal Information, Your Advisor, Documents library.

**Theme**
- [ ] Advisor view can toggle green ⇄ navy; user view stays green.
- [ ] Light/dark toggle works on **every** screen (advisor, client, onboarding) with no
      illegible text.

**Renames (exact wording)**
- [ ] "Your next actions" → **To-do list**
- [ ] Onboarding stage "current" state → **Pending** (everywhere, incl. Registered
      Users status)
- [ ] "Link / Linked" → **Appoint / Appointed**
- [ ] "Investor Type" → **Risk profile**
- [ ] "Unrealised G/L" → **Profit/Loss**
- [ ] Analytics "How your book is related" → **Customer acquisition**
- [ ] Notifications "Thresholds" → **Settings**
- [ ] Landing preview → **Preview profile**

**Additions**
- [ ] Assets under advice shows grey **(x% increase from last month)** sublabel.
- [ ] Customers table has **% change** and **Net worth** columns and does not overflow.
- [ ] Advisor + client Documents both have Upload button + drag-and-drop + name +
      type dropdown (Identity, Agreement, Proof of Address, Statements, Other).
- [ ] Book meeting "Send invite" opens an **editable email** with the calendar invite;
      client **Request meeting** opens the same flow.
- [ ] Notifications Settings page lists all 9 types with toggles; "investment over $X"
      and "x% change" have editable amounts.
- [ ] Qualified filtering settings page sets min net worth / monthly savings / income /
      fitness, and recomputes the qualified count. Filter row has **All** left of
      **Qualified**.
- [ ] My Profile holds the Preview with an Edit button top-right; Customise opens in a
      right-hand layer and includes services, LinkedIn, website, email, phone, FSP
      licence, description, banner upload (with recommended size), profile picture.
- [ ] Landing Page = image right / business description left, same fields as profile,
      with register + get-in-touch CTAs, QR, referral, share buttons.
- [ ] Invite advisors supports same-firm invites: optional first/last name, same-firm
      radio, Admin/Member dropdown with a tooltip on admin rights.
- [ ] Audit Log Recent activity filters by user name, action, date.
- [ ] Portfolios: full seeded SA list, name-first column, Total invested column, View
      button opening a right-hand fund-detail layer, search bar; target allocation
      moved into that layer below the chart; Add-portfolio flow with all fields +
      submitted-for-approval status.
- [ ] Analytics: advisor-name box replaced by "Registered on website"; monthly-growth
      chart moved to bottom above Customer trends; period selector (This month / Last
      month / Last 90 days / YTD / Custom) drives **all** metrics.
- [ ] Risk assessment: Request risk assessment control notifies the client + logs
      audit; Last risk assessment date shown per customer.

**House rules**
- [ ] Zero em dashes in the rendered DOM (all routes, tabs, drawers, both scenarios,
      both themes, light + dark).
- [ ] All money comma-grouped; no `en-ZA`; no decimals on dashboard money.
- [ ] No `.io` domains; no internal/dev references in UI.
- [ ] Zero dead controls; zero console errors; no `undefined` / `NaN` / `[object
      Object]`.
- [ ] No horizontal overflow at 390px on any route.
- [ ] Existing consent round-trip, QR encoder, onboarding, region select still work.

When every box is checked, summarise what changed and note anything seeded that needs
real data later (Uday's portfolio list; final commission/fee figures).

# Vault22 Advisor Prototype — spec audit

Item-by-item audit of `index.html` against the sources of truth, in precedence order:

1. **URL rulings** — dashboard at `vault22.com/advisor`; share link
   `{advisor-name}.advisor.vault22.com`; the advisor name is mandatory.
2. **`standup-transcript-17jul.txt`** — 17 Jul stand-up.
3. **The doc, §1–§15** (reproduced in `PROMPT.md` Part 3). Its "Future plans"
   table is out of scope.

`Implemented` = Yes / Partial / No · `Works` = Verified / Broken / N/A.
Evidence cites `index.html` line numbers (`L###`), a screenshot, or a named
assertion from the JSC harnesses (`partA.js`, `req.js`, `fix.js`, `onb.js`).

Audited 17 Jul 2026 against commit at time of writing (2,120 lines).

---

## Summary — everything not a clean Yes/Verified

| # | § | Item | State | What's wrong |
|---|---|---|---|---|
| 1.11 | §1 | Status: **Suspended** | **No** | The three statuses are Pending Verification / Active / Suspended. Only the first two ever render (L1073 shows Pending→Active; the profile shows only the current status). Suspended exists in no code path — it was lost when the static register form became the onboarding flow. |
| 10.26 | §10 | Timeline: **Portfolio Review** | **No** | The doc's timeline is Signed Up, Completed KYC, Risk Assessment, First Investment, SIP Started, **Portfolio Review**. The build renders the first five then "Last investment" instead (L1508–1513). 5 of 6. |
| 9.16 | §9 | Investment: **One-time investments** | **Partial** | `c.onetime` exists (L687) but surfaces only inside the timeline string "First investment — R350,000" (L1511). There is no one-time investments field in the snapshot or investment block. |
| 8.2 | §8 | Lead column: **Email** | **Partial** | Present but not its own column — rendered as a sub-line under Name (L1317). All 11 fields are visible; 10 are discrete columns. |
| 3.6 | §3 | Landing page: **QR code** | **Partial** | The landing *page preview* (L1777–1784) shows profile/about/services/contact/CTA but no QR. The QR is on the same screen in the Referral card (L1799). Per the doc the QR is *content of the landing page*; here it's advisor-side furniture. |
| 3.7 | §3 | Landing page: **Share buttons** | **Partial** | Same as above — "Share my Client Platform" (L1230) is advisor-side, not on the prospect-facing preview. |
| 2.6 | §2 | Profile: **Social media links** | **Partial** | Three inputs render (L1856–1860) but are decorative — not bound to state, not persisted, not shown on the landing preview. |
| 3.10 | §3 | Customisation: **Banner image** | **Partial** | Control present (L1818); opens an honest stub modal. The preview banner is a fixed gradient (the `.lp-ban` gradient) and cannot change. |
| 3.9 | §3 | Customisation: **Theme colour** | **Partial** | Colour input renders (L1814) but changing it does not restyle the preview. |
| 1.6 | §1 | Registration: **Company** | **Partial** | Field renders (L1020) but is not captured to `state.advisor` on submit — only name and email are (`goOnb` L1141). Same for Mobile (1.3), Licence (1.7), Referral code (1.8). |
| — | x-cut | **Investor-type labels** | **Flag** | Ships `Very Conservative / Conservative / Balanced / Growth / Aggressive` (`TYPES` L636). Vault22's own investor onboarding uses `Ultra-Low / Cautious / Balanced / Growth / Aggressive` (`Vault22-Onboarding-Prototype/app.js` ~L1995, *"aligned to the live investment-style tiers used by the quiz"*). Count (5) is right; the bottom two labels are invented. An advisor would read *Conservative* where the client's own app says *Cautious*. **Not guessed at — for Aishani to rule on.** |
| — | x-cut | **Notification bell glyph** | **Flag** | The bell is `&#9788;` — ☼, a sun (L497). Cosmetic, visible top-right on every dashboard screen. |
| — | x-cut | **Underlined action links** | **Flag** | `Call` / `Email` / `WhatsApp` on the customer profile render underlined (screenshot `A-cust-drill.png`) — `a.btn{text-decoration:none}` was scoped to `.onb` only, so the dashboard kept the default. |

Everything else below is Yes / Verified.

---

## Precedence 1 — URL rulings

| # | Source | Requirement (verbatim) | Implemented | Works | Evidence | Notes |
|---|---|---|---|---|---|---|
| 0.1 | ruling | Advisor dashboard lives at `vault22.com/advisor` | Yes | Verified | `DASH_URL` L785; shown in the onboarding URL bar (`urlbar`, L907) | Not just a string — surfaced in-UI on every onboarding screen. |
| 0.2 | ruling | Share link is `{advisor-name}.advisor.vault22.com` | Yes | Verified | `advisorUrl()` L783; assertion `advisorUrl -> aishani.advisor.vault22.com` (partA.js) | No fallback; an absent slug is a bug, not a state. |
| 0.3 | ruling | The advisor name is **mandatory** — no skip, no auto-generated fallback | Yes | Verified | `slugCheck()` L889; assertion `cannot advance past name without a name` (onb.js) | Continue disabled until valid *and* re-checked in `goOnb` L1141. |
| 0.4 | ruling | Validate: lowercase, alphanumeric + hyphens, no leading/trailing hyphen, 3–30, availability | Yes | Verified | L889–900; 17/17 assertions (onb.js) | Reserved: admin, www, test, advisor, api, support, help, app, vault22. Uppercase normalises rather than rejects (deliberate — subdomains are case-insensitive). |

## Precedence 2 — 17 Jul stand-up (binding)

| # | Source | Requirement | Implemented | Works | Evidence | Notes |
|---|---|---|---|---|---|---|
| T.1 | transcript | Onboarding is **name, email, password, region** | Yes | Verified | L1008/1011/991 (name/email/password); `REGIONS` L880; extras collapsed L1018–1024 | Uday named the four twice; the doc's extras are secondary. |
| T.2 | transcript | Link + QR created **during onboarding** | Yes | Verified | `ONB.link` L1110; screenshot `v3-link.png` | Not a settings page. |
| T.3 | transcript | Lead = signed up via the advisor's link | Yes | Verified | lead definition L1311; `LEADS_EST` L695 | |
| T.4 | transcript | Customer = *"once the user buys any product"* | Yes | Verified | L1397 — "bought at least one product" | Looser than the doc's "first successful investment" — transcript wins. |
| T.5 | transcript | Investor journey **unchanged** | Yes | N/A | "Start Investing" stub L1783 states the journey is unchanged | Deliberately not rebuilt. |
| T.6 | transcript | Financial data shown deliberately, pending consent design | Yes | Verified | `consent()` L853, `pii()` L861; assertion block `A5` (partA.js) | Mechanism kept; now scoped to views that have `.pii`. |
| T.7 | transcript | No lead assignment/coordination between advisors | Yes | N/A | Absent by design | *"each will have their own account"*. |

---

## §1 Advisor Registration

**Fields (8)**

| # | § | Requirement (verbatim) | Implemented | Works | Evidence | Notes |
|---|---|---|---|---|---|---|
| 1.1 | §1 | Full Name | Yes | Verified | L1008; assertion `name captured` (fix.js) | Drives greeting, initials, referral code. |
| 1.2 | §1 | Email Address | Yes | Verified | L1011; assertion `verify shows the TYPED email` (fix.js) | |
| 1.3 | §1 | Mobile Number | Partial | N/A | L1018 | Renders; not captured to state. |
| 1.4 | §1 | Password | Yes | N/A | L991 | Mock — no backend. |
| 1.5 | §1 | Country / Region | Yes | Verified | `ONB.region` L1034; `REGIONS` L880 | Gated: Continue disabled until picked (L1048). |
| 1.6 | §1 | Company (optional) | Partial | N/A | L1020 | Renders; not captured. |
| 1.7 | §1 | License Number (optional depending on region) | Partial | N/A | L1022 | Renders; not captured. Label carries "(Optional)" per A2 of the fix list. |
| 1.8 | §1 | Referral Code (optional) | Partial | N/A | L1024 | Renders; not captured. Distinct from the advisor's *own* referral code (4.3). |

**Verification (4)**

| # | § | Requirement | Implemented | Works | Evidence | Notes |
|---|---|---|---|---|---|---|
| 1.9 | §1 | Email verification | Yes | N/A | L1065 | Mock state — shown Verified. |
| 1.10 | §1 | Mobile OTP (optional) | Yes | N/A | L1067 | Shown Skipped, marked Optional. |
| 1.11 | §1 | Terms & Conditions | Yes | N/A | L1069 | Accepted state + legal note on register. |
| 1.12 | §1 | Privacy Policy | Yes | N/A | L1070 | |

**Statuses (3)**

| # | § | Requirement | Implemented | Works | Evidence | Notes |
|---|---|---|---|---|---|---|
| 1.13 | §1 | Pending Verification | Yes | Verified | L656 (`ADVISOR_NEW.status`), L1073 | Start state for a new advisor. |
| 1.14 | §1 | Active | Yes | Verified | L1074; set on completion in `goOnb` L1141 | |
| 1.15 | §1 | Suspended | **No** | **Broken** | Not present — grep for "Suspended" returns nothing | See summary. |
| 1.16 | §1 | Entry point `vault22.com/advisor` | Yes | Verified | `ONB.entry` L956 | Per ruling 0.1 this is the dashboard's address; onboarding happens on the way to it. |

## §2 Advisor Profile

| # | § | Requirement | Implemented | Works | Evidence | Notes |
|---|---|---|---|---|---|---|
| 2.1 | §2 | Profile photo | Yes | N/A | L1837 | Stub modal (honest terminal state). |
| 2.2 | §2 | Company logo | Yes | N/A | L1843 | Stub modal. |
| 2.3 | §2 | Biography | Yes | Verified | L1841; renders on the landing preview L1778 | |
| 2.4 | §2 | Contact details | Yes | Verified | L1847–1851; on preview L1784 | |
| 2.5 | §2 | Calendar booking link | Yes | Verified | L1851; used by the Schedule Meeting stub L1948 | |
| 2.6 | §2 | Social media links | Partial | N/A | L1856–1860 | Decorative — see summary. |

## §3 Personalised Landing Page

| # | § | Requirement | Implemented | Works | Evidence | Notes |
|---|---|---|---|---|---|---|
| 3.1 | §3 | `https://advisorname.vault22.com`, advisor can change the website name | Yes | Verified | L1810; ruling 0.2 supersedes the doc format | Doc format is wrong per the ruling — `.advisor.` segment added. |
| 3.2 | §3 | Landing page contains: advisor profile | Yes | Verified | L315 | |
| 3.3 | §3 | About advisor | Yes | Verified | L1778 | |
| 3.4 | §3 | Services offered | Yes | Verified | L1779 | |
| 3.5 | §3 | Contact information | Yes | Verified | L1784 | |
| 3.6 | §3 | "Start Investing" CTA | Yes | Verified | L1783 | Stub explains the investor journey is unchanged. |
| 3.7 | §3 | QR Code | Partial | Verified | L1799 (advisor-side card) | Not on the prospect-facing preview — see summary. |
| 3.8 | §3 | Share buttons | Partial | Verified | L1230 (advisor-side) | Same. |
| 3.9 | §3 | Customise: banner image | Partial | N/A | L1818 | Preview banner is fixed. |
| 3.10 | §3 | Customise: theme colour | Partial | N/A | L1814 | Doesn't restyle the preview. |
| 3.11 | §3 | Customise: welcome message | Yes | Verified | L1813; renders on preview L1777 | |

## §4 QR Code & Referral

| # | § | Requirement | Implemented | Works | Evidence | Notes |
|---|---|---|---|---|---|---|
| 4.1 | §4 | Auto-generate QR Code | Yes | Verified | `qrMatrix()` L537; `drawQR()` L606 | **Real encoder** — byte mode, ECC-M, v1–10. Byte-identical to Apple's `CIQRCodeGenerator`; `CIDetector` decodes the rendered pixels as `https://janesmith.advisor.vault22.com`. Passes v2–v8. |
| 4.2 | §4 | Auto-generate Referral URL | Yes | Verified | L1792 | |
| 4.3 | §4 | Auto-generate Referral Code | Yes | Verified | `refCodeOf()` L664; assertion `Aishani Atri -> AISH22` (fix.js) | Derived from the name, not hardcoded. |
| 4.4 | §4 | Channel: WhatsApp | Yes | Verified | `shareBtns()` L1198 | Real `wa.me` target with prefilled text. |
| 4.5 | §4 | Channel: Email | Yes | Verified | L1204 (`mailto:`) | Real `mailto:`. |
| 4.6 | §4 | Channel: Facebook | Yes | Verified | L1206 | Real sharer. |
| 4.7 | §4 | Channel: LinkedIn | Yes | Verified | L1205 | Real sharer. |
| 4.8 | §4 | Channel: Copy Link | Yes | Verified | L1122 + clipboard handler | |
| 4.9 | §4 | Every signup through this URL is tagged to that advisor | Yes | N/A | copy at L1311; the model is mock | Front-end prototype — no signup pipeline to tag. |

## §5 Prospect Journey

| # | § | Requirement | Implemented | Works | Evidence | Notes |
|---|---|---|---|---|---|---|
| 5.1 | §5 | Prospect clicks the link → completes standard Vault22 onboarding | Yes | N/A | Stub L1200 | Investor side deliberately not built (T.5). |
| 5.2 | §5 | **No changes to investor journey** | Yes | N/A | Absent by design | Honoured by omission. |
| 5.3 | §5 | Mapping: Advisor → Lead → Customer | Yes | Verified | leads L1311, customers L1397, conversion L1244 | |

## §6 Client Lifecycle

| # | § | Requirement | Implemented | Works | Evidence | Notes |
|---|---|---|---|---|---|---|
| 6.1 | §6 | Lead = registered but not yet invested | Yes | Verified | L1325 | |
| 6.2 | §6 | Customer = completed first successful investment | Yes | Verified | L1397 | Transcript loosens this to "buys any product" (T.4). |
| 6.3–6.10 | §6 | The 8 stages: Registered · Email Verified · KYC Started · KYC Submitted · KYC Approved · Risk Assessment Pending · Risk Assessment Completed · Ready to Invest | Yes | Verified | `STAGES` L632; rendered as a progression in the lead modal L1950 | All 8, in order, with per-stage pill colour L634. |

## §7 Advisor Dashboard — KPIs (10)

| # | § | Requirement | Implemented | Works | Evidence | Notes |
|---|---|---|---|---|---|---|
| 7.1 | §7 | Total Leads | Yes | Verified | L1261 | Hero tile. |
| 7.2 | §7 | Total Customers | Yes | Verified | L1262 | Hero tile. Both are the stand-up's headline numbers. |
| 7.3 | §7 | Conversion Rate | Yes | Verified | L1244 | Computed, not hardcoded. |
| 7.4 | §7 | Assets Under Advice (AUA) | Yes | Verified | L1245 | Computed from customers. |
| 7.5 | §7 | Total Investments | Yes | Verified | L1267 | Computed. |
| 7.6 | §7 | Monthly New Customers | Yes | Verified | L1268 | Static mock. |
| 7.7 | §7 | Monthly Investment Amount | Yes | Verified | L1269 | Static mock. |
| 7.8 | §7 | Revenue / Commission | Yes | Verified | L1270 | Static mock, labelled "estimated". |
| 7.9 | §7 | Pending KYC | Yes | Verified | L1273 | Computed from stages. |
| 7.10 | §7 | Pending Risk Assessments | Yes | Verified | L1274 | Computed. |

## §8 Lead Management — columns (11) + actions (6)

| # | § | Requirement | Implemented | Works | Evidence | Notes |
|---|---|---|---|---|---|---|
| 8.1 | §8 | Name | Yes | Verified | L1324 | |
| 8.2 | §8 | Email | Partial | Verified | L1324 | Sub-line under Name, not a column. |
| 8.3 | §8 | Phone | Yes | Verified | L1325 | |
| 8.4 | §8 | Signup Date | Yes | Verified | L1326 | |
| 8.5 | §8 | Lead Status | Yes | Verified | L1327 | All 8 stages. |
| 8.6 | §8 | Financial Fitness Score | Yes | Verified | L1322 | `.pii` — consent-pending. |
| 8.7 | §8 | Net Worth (estimated) | Yes | Verified | L1323 | `.pii`. |
| 8.8 | §8 | Monthly Savings | Yes | Verified | L1324 | `.pii`. |
| 8.9 | §8 | Investment Capacity | Yes | Verified | L1325 | `.pii`. |
| 8.10 | §8 | Assigned Portfolio (recommendation) | Yes | Verified | L1326 | |
| 8.11 | §8 | Last Activity | Yes | Verified | L1327 | Relative time. |
| 8.12 | §8 | Action: Call | Yes | Verified | L1945 (`tel:`) | Real `tel:`. |
| 8.13 | §8 | Action: Email | Yes | Verified | L1946 (`mailto:`) | Real `mailto:`. |
| 8.14 | §8 | Action: WhatsApp | Yes | Verified | L1947 (`wa.me`) | Real `wa.me` with prefilled text. |
| 8.15 | §8 | Action: Schedule Meeting | Yes | N/A | L1948 | Stub modal — appointment scheduling is Future plans. |
| 8.16 | §8 | Action: Add Notes | Yes | Verified | `crmTabs()` L1450 | In-memory, works. |
| 8.17 | §8 | Action: Follow-up Reminder | Yes | Verified | `crmTabs()` L1450 | In-memory, works. |

## §9 Customer Management

| # | § | Requirement | Implemented | Works | Evidence | Notes |
|---|---|---|---|---|---|---|
| 9.1 | §9 | Personal: Name | Yes | Verified | L1317 | |
| 9.2 | §9 | Personal: Email | Yes | Verified | L1317 | |
| 9.3 | §9 | Personal: Phone | Yes | Verified | L1318 | |
| 9.4 | §9 | Personal: Country | Yes | Verified | L1544 | On profile. |
| 9.5 | §9 | Personal: KYC Status | Yes | Verified | L1320 | |
| 9.6 | §9 | Financial: Financial Fitness Score | Yes | Verified | L1322 | `.pii`. |
| 9.7 | §9 | Financial: Net Worth | Yes | Verified | L1323 | `.pii`. |
| 9.8 | §9 | Financial: Monthly Income | Yes | Verified | L1568 | `.pii`. |
| 9.9 | §9 | Financial: Monthly Savings | Yes | Verified | L1565 | `.pii`. |
| 9.10 | §9 | Financial: Emergency Fund | Yes | Verified | L1567 | `.pii`. |
| 9.11 | §9 | Investment: Current Portfolio Value | Yes | Verified | L1245 | |
| 9.12 | §9 | Investment: Invested Amount | Yes | Verified | L1564 | |
| 9.13 | §9 | Investment: Unrealised Gain/Loss | Yes | Verified | L1323 | Colour-coded. |
| 9.14 | §9 | Investment: Asset Allocation | Yes | Verified | L1546 | Bar chart. |
| 9.15 | §9 | Investment: Products Owned | Yes | Verified | L1572 | Explicit block. |
| 9.16 | §9 | Investment: Recurring Investment | Yes | Verified | L1568 | |
| 9.17 | §9 | Investment: One-time Investments | Partial | Verified | `c.onetime` L687; used only at L1511 | Only inside the timeline string — see summary. |
| 9.18 | §9 | Risk: Risk Assessment Completed | Yes | Verified | L1579 | |
| 9.19 | §9 | Risk: Risk Score | Yes | Verified | L1580 | /100. |
| 9.20 | §9 | Risk: Investor Type (Conservative/Balanced/Growth/Aggressive) | Yes | Verified | L1582; `TYPES` L636 | Doc labels 4 as "Example"; build ships 5. **Labels flagged** — see summary. |

## §10 Customer Profile

| # | § | Requirement | Implemented | Works | Evidence | Notes |
|---|---|---|---|---|---|---|
| 10.1 | §10 | Summary: Client since | Yes | Verified | L1544 | |
| 10.2 | §10 | Summary: Portfolio Value | Yes | Verified | L1245 | |
| 10.3 | §10 | Summary: Last Login | Yes | Verified | L1544 | |
| 10.4 | §10 | Summary: Last Investment | Yes | Verified | L1544 | |
| 10.5 | §10 | Summary: Financial Fitness Score | Yes | Verified | L1322 | `.pii`. |
| 10.6 | §10 | Holdings: Product Name | Yes | Verified | L1544 | |
| 10.7 | §10 | Holdings: Units | Yes | Verified | L1544 | |
| 10.8 | §10 | Holdings: Market Value | Yes | Verified | L1544 | |
| 10.9 | §10 | Holdings: Return % | Yes | Verified | L1544 | |
| 10.10 | §10 | Holdings: Purchase Date | Yes | Verified | L1544 | |
| 10.11 | §10 | Snapshot: Net Worth | Yes | Verified | L1564 | `.pii`. |
| 10.12 | §10 | Snapshot: Savings | Yes | Verified | L1565 | `.pii`. |
| 10.13 | §10 | Snapshot: Investments | Yes | Verified | L1564 | |
| 10.14 | §10 | Snapshot: Insurance | Yes | Verified | L1567 | `.pii`. |
| 10.15 | §10 | Snapshot: Liabilities | Yes | Verified | L1568 | `.pii`. |
| 10.16 | §10 | Documents: KYC | Yes | N/A | L1588 | Stub modal; view-only per §15. |
| 10.17 | §10 | Documents: Signed Agreements | Yes | N/A | L1590 | Stub modal. |
| 10.18 | §10 | Timeline: Signed Up | Yes | Verified | L1508 | |
| 10.19 | §10 | Timeline: Completed KYC | Yes | Verified | L1509 | |
| 10.20 | §10 | Timeline: Risk Assessment | Yes | Verified | L1510 | |
| 10.21 | §10 | Timeline: First Investment | Yes | Verified | L1511 | |
| 10.22 | §10 | Timeline: SIP Started | Yes | Verified | L1512 | |
| 10.23 | §10 | Timeline: Portfolio Review | **No** | **Broken** | L1513 renders "Last investment" instead | See summary. |

## §11 Notifications (8)

| # | § | Requirement | Implemented | Works | Evidence | Notes |
|---|---|---|---|---|---|---|
| 11.1 | §11 | New Lead | Yes | Verified | `NOTIFS_EST` L719 | Routes to the lead on click. |
| 11.2 | §11 | KYC Completed | Yes | Verified | L723 | |
| 11.3 | §11 | Risk Assessment Completed | Yes | Verified | L721 | |
| 11.4 | §11 | First Investment | Yes | Verified | L724 | |
| 11.5 | §11 | SIP Failed | Yes | Verified | L725 | |
| 11.6 | §11 | Large Investment | Yes | Verified | L722 | |
| 11.7 | §11 | Birthday | Yes | Verified | L726 | |
| 11.8 | §11 | Portfolio Review Due | Yes | Verified | L727 | |
| 11.9 | §11 | Advisor is notified (surface) | Yes | Verified | bell + unread badge L497; `togglePanel()` | Empty state in Scenario A. **Bell glyph flagged** (☼). |

## §12 Advisor CRM (6) — every client

| # | § | Requirement | Implemented | Works | Evidence | Notes |
|---|---|---|---|---|---|---|
| 12.1 | §12 | Notes | Yes | Verified | `crmTabs` L1450 | Add/list, in-memory. |
| 12.2 | §12 | Tags | Yes | Verified | `crmTabs()` L1450 | Add/remove. |
| 12.3 | §12 | Tasks | Yes | Verified | `crmTabs()` L1450 | Add/complete. |
| 12.4 | §12 | Follow-ups | Yes | Verified | `crmTabs()` L1450 | Date + reminder. |
| 12.5 | §12 | Meeting History | Yes | Verified | `crmTabs()` L1450 | |
| 12.6 | §12 | File Uploads | Yes | Verified | `crmTabs()` L1450 | Adds a mock file (no backend). |
| 12.7 | §12 | On **every** client (leads *and* customers) | Yes | Verified | lead modal + customer profile both call `crmTabs()` L1450 | Same component both sides. |

## §13 Search & Filters

| # | § | Requirement | Implemented | Works | Evidence | Notes |
|---|---|---|---|---|---|---|
| 13.1 | §13 | Search by Name | Yes | Verified | `applyLeads()` L1291 | Live. |
| 13.2 | §13 | Search by Email | Yes | Verified | L1291 | |
| 13.3 | §13 | Search by Phone | Yes | Verified | L1291 | |
| 13.4 | §13 | Filter: Lead | Yes | Verified | Leads view is the lead set; `LEAD_FILTERS` L1281 | |
| 13.5 | §13 | Filter: Customer | Yes | Verified | Customers view; `CUST_FILTERS` L1373 | |
| 13.6 | §13 | Filter: KYC Pending | Yes | Verified | `LEAD_FILTERS` L1281 | |
| 13.7 | §13 | Filter: Risk Pending | Yes | Verified | `LEAD_FILTERS` L1281 | |
| 13.8 | §13 | Filter: High Net Worth | Yes | Verified | L1281 / L1373 | |
| 13.9 | §13 | Filter: Financial Fitness Score | Yes | Verified | L1281 / L1373 | "Fitness ≥ 600 / 700". |
| 13.10 | §13 | Filter: Portfolio Value | Yes | Verified | `CUST_FILTERS` L1373 | Customers. |
| 13.11 | §13 | Filter: Last Activity | Yes | Verified | L1281 / L1373 | "Active this week". |
| 13.12 | §13 | (support) Sortable columns + "N of M" + empty state | Yes | Verified | `th()` L1303; search box L1349 | Beyond spec; supports the above. |

## §14 Analytics (8)

| # | § | Requirement | Implemented | Works | Evidence | Notes |
|---|---|---|---|---|---|---|
| 14.1 | §14 | Lead Funnel | Yes | Verified | L1697 | Hero element; cumulative across the 8 stages. |
| 14.2 | §14 | Conversion Rate | Yes | Verified | L1244 | |
| 14.3 | §14 | Average Investment | Yes | Verified | L1722 | |
| 14.4 | §14 | Top Products Sold | Yes | Verified | L1739 | Derived from holdings. |
| 14.5 | §14 | Monthly Growth | Yes | Verified | L1743 | Inline SVG sparkline. |
| 14.6 | §14 | AUM Growth | Yes | Verified | L1724 | |
| 14.7 | §14 | Customer Retention | Yes | Verified | L1726 | Static 100% (no redemptions modelled). |
| 14.8 | §14 | Investment Trends | Yes | Verified | L1751 | Sparkline. |
| 14.9 | §14 | (support) No chart library — works offline from `file://` | Yes | Verified | `spark()` L1676 — inline SVG | Constraint honoured. |

## §15 Permissions & Security

| # | § | Requirement | Implemented | Works | Evidence | Notes |
|---|---|---|---|---|---|---|
| 15.1 | §15 | Advisor must NOT modify KYC information | Yes | Verified | stated L1883; KYC controls are view-only stubs L1588–1590 | Enforced by absence — no edit path exists. |
| 15.2 | §15 | Must NOT change customer portfolios without authorisation | Yes | Verified | L1883 | No portfolio-edit path on a customer. |
| 15.3 | §15 | Must NOT access clients belonging to other advisors | Yes | N/A | L1884 | Single-advisor dataset; nothing to cross into. |
| 15.4 | §15 | Must NOT view sensitive PII beyond servicing — mask identity document numbers | Yes | Verified | `maskId()` L630; rendered L1586 | ID numbers masked by default. |
| 15.5 | §15 | All advisor actions logged for audit | Yes | Verified | `audit()` L786; view L1889; appends live on real actions | Seeded + live entries. |

---

## Out of scope — confirmed absent

| Item | Status | Note |
|---|---|---|
| "Future plans" table (13 rows) | Correctly absent | The Roadmap panel that rendered 10 of them was **deleted** this pass (A7, Aishani's ruling). It was never a spec feature. |
| Investor/prospect side of the journey | Correctly absent | Unchanged per the stand-up. |

## Prototype-wide caveats

Front-end only: no auth, no backend, no email/OTP. Mock data throughout; money
figures are illustrative placeholders to show states. Nothing persists across a
reload. Every remaining stub opens an honest terminal state saying what the real
build would do — verified by assertion `no data-toast dead ends remain` (partA.js).

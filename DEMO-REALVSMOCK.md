# Vault22 Advisor Prototype — What's Real vs Mocked

A per-screen reference for live demos, so you never click a dead thing on stage. This is a
single-file front-end prototype: everything runs in the browser, nothing hits a server.
State lives in memory and is saved to `sessionStorage` for the tab (survives a refresh; the
gear's **Reset demo** clears it). Build: see the version in the Prototype settings gear.

**Legend:** ✅ interactive / real · 🟡 real UI, illustrative data · ⚪ stub (explains itself,
no dead end) · 🔒 intentionally out of scope for the prototype.

## Global
- ✅ **Scenario switch** (bottom-right gear): New advisor · Established advisor · Financial
  Advisor (client) view. ✅ **Green/Navy** theme + **dark mode**. ✅ **Reset demo** clears the
  session and reloads the current scenario clean.
- ✅ **First-run setup checklist** (new advisor): 0/4 steps tick as you do them, then collapse.
- ✅ **Global client search** (top bar): name/email/phone → dropdown → jumps to the profile.
- ✅ **Search for an advisor** directory now lives on the **user** side (Profile → Your Advisor);
  searching filters and selecting appoints. The advisor-side "Browse advisors" nav item is gone.
- ✅ **Firm team** view (roster + totals). Roster is seeded, not derived from the live book.
- ⚠️ **Portfolio moderation queue** moved to settings gear → **Vault22 admin**. Advisors cannot
  approve their own portfolios. Where this lives in project x is **open with Uday**.
- ⚠️ **Full SA portfolio list** still placeholder, **waiting on Uday**.
- ⚠️ **Membership pricing** tiers provisional, **waiting on Greg** (single constant, one-line change).
- ✅ **Session persistence** — a refresh keeps uploaded docs, notes, consent, meetings, Tara drafts.
- 🟡 All customers, registered users, portfolios and money are **illustrative SA seed data**.
- 🔒 No real email/SMS/calendar/payment service. "Send" opens your own mail client via `mailto:`.

## Onboarding (New advisor scenario)
- ✅ Multi-step onboarding, email + password + phone validation, region gate, policy accept.
- ⚪ OTP/verification codes: any 6-digit code passes (no SMS service).

## Dashboard
- ✅ **KPI tiles** (Pending KYC, Pending risk) click through to the filtered Registered Users list.
- ✅ **To-do rows** (Chase KYC, Chase risk, Follow up qualified, Confirm meeting requests) all
  navigate to the right filtered view.
- ✅ **Estimated fees this month** with trend sparkline (advisory fees, same basis as the
  client Financial tab). ✅ Largest clients + Latest activity open the relevant profile.
- 🟡 Numbers are computed live from the seed book; the sparkline trend is illustrative.

## Registered Users / Customers
- ✅ Search, filters (qualification segment, KYC/Risk pending, HNW, etc.), sort, row → full profile.
- ✅ **Source** column (acquisition channel). 🟡 Sources are seeded per user.
- ✅ Qualified-user filtering thresholds are editable and recompute the qualified count.

## Customer / Registered-user profile
- ✅ Tabs: **Overview** · Personal · Financial · Investment · Risk · Documents · Activity · CRM · Audit.
- ✅ **Overview** (first tab): stage, risk, net worth, fees earned, last contact, last risk assessment.
- ✅ **Consent gating**: Financial/Investment/Risk locked until the client grants access (POPIA-style,
  revocable) in their app. **Request all data access** asks for all three in one action.
- ✅ Advisor-initiated request loops, all round-trip with the client scenario and update both sides:
  **Request risk assessment**, **Request KYC refresh**, **Request power of attorney** (Permissions).
- ✅ **Documents**: request→fulfil loop, type required, **expiry dates** with Expired/Expiring flags,
  and **Open** builds a placeholder preview PDF.
- ✅ **CRM**: notes / tags / tasks / follow-ups / meetings / files. **Registered Users** support
  **multi-select → Email / Export CSV**; **Customers** show Last contacted and sort by Fees.
- ✅ **Download financial report** builds a real PDF; **Audit log** exports CSV.
- 🟡 Holdings, allocations, audit history, acquisition sources are seeded.

## Booking a meeting (advisor side)
- ✅ Month **calendar** (past/weekend/fully-booked days disabled, today + selected marked,
  availability dots), grouped **time slots** (Morning/Afternoon), **already-booked slots disabled**.
- ✅ **Duration** (15/30/60), **Timezone**, **Location** (Video/Phone/In person) flow into the
  summary and the `.ics`. ✅ 12h/24h toggle. ✅ **Confirmation summary** before send.
- ✅ **Send invite** → editable email compose → **Send** → real `.ics` downloads + `mailto:` opens
  → meeting logged. ✅ Two-way loop: client **Accept/Decline** → advisor sees **Confirmed**.
- 🟡 Availability is a fixed prototype schedule (weekdays 09:00–17:00 SAST). 🔒 Live Google/
  Outlook sync is a real-build feature (noted on the surface).

## Tara — personalise a portfolio
- ✅ Full-screen, ZAR/USD, labelled asset universe (ETFs / stocks / other), add/remove, weightings.
- ✅ **Strategy summary** (expected return, volatility, blended fee, profile-fit check),
  **before/after** vs the current portfolio, **note to client** + compliance disclaimer.
- ✅ **Suggest to client** → client sees it (with the note + disclaimer) → **tweak** weights →
  **Invest** → advisor sees it invested. 🟡 No real order is ever placed.

## Landing page (public)
- ✅ Prospect preview: business description + image, **Start Investing** (real registration →
  creates a Registered User tagged to the advisor) and **Get in touch** (creates a real
  enquiry lead + notification). ✅ Share panel: referral URL, QR download.
- 🟡 Testimonials/rating (if shown) are illustrative.

## Membership / billing
- ✅ **Fee-tier table** (1–10 / 11–50 / 51+) with the current tier highlighted. ✅ Invoices
  **View** (drawer) and **Download PDF** (real PDF). 🟡 Pricing illustrative.
- ⚠️ **Open product decision (Greg):** the 25 bps AUM fee on non-Vault22 assets is currently
  shown as **billed to the client**, so the advisor's amount due is the subscription only.

## Analytics
- ✅ Period selector (This month / Last month / 90 days / YTD / **Custom** date range) drives all
  KPIs, funnel, trends and the three charts. 🟡 Series are seeded per period.

## Firm / Permissions / Invites
- ✅ Admin vs Member view (Admin edits the shared firm page; Member is read-only).
- ✅ Invite advisors (same-firm radio → role), consent round-trip, power-of-attorney gating on
  portfolio switches (blocked without POA, and the attempt is logged).
- 🔒 Deeper firm rollup / team analytics: shell only, future.

## Known stubs (safe to click, they explain themselves)
- ⚪ "Open fact sheet" on a portfolio (explains it opens the real PDF in the live build).
- ⚪ Client-app sections other than Profile (out of scope: the prototype is the advisor portal).
- ⚪ Any `data-stub` button shows an explanatory modal, never a dead end.

# INTENT-DRIVEN REBUILD PROMPT — Vault22 Advisor Prototype

> Paste everything below the line into a fresh Claude Code session on this repo.
> Self-contained. This is a BEHAVIOUR rebuild, not a keyword pass.

---

You are the Senior Product Manager, UX Designer AND Senior Frontend Engineer for the
highest-priority prototype at Vault22: the Financial Advisor Dashboard, to be demoed to
large South African IFA (Independent Financial Advisor) groups. Work entirely in
`/Users/aishaniatri/Vault22-Advisor-Prototype/index.html` (single-file, vanilla JS, no
build). **Read it in full before changing anything.**

The previous rounds added the right screens but implemented them at a surface level:
several buttons look right but do nothing meaningful, and the advisor↔client workflow
loops do not close. Your job is to make every requirement **behave** the way Stephen
intended, so that when he clicks it live in front of a room of advisors, he says
"yes, that's exactly what I meant."

## The one rule that governs everything
**Behaviour over UI. A feature is done only when clicking it advances a believable advisor
workflow.** A screen that shows the right words but produces no real consequence is a
defect, not a feature. The canonical failure already found: the Navy/Dark toggles existed
and were "tested" by setting state in code, but were never wired to the click handler, so
they were dead in the actual UI. Do not repeat that class of error: **validate by clicking
real buttons in the rendered app, never by calling internal functions.**

## House rules (already enforced; keep them)
No em dashes in rendered copy · comma-grouped Rands via `money()` (the advisor subscription
fee is the only USD figure, via `usd()`) · `vault22.com` never `.io` · UK spelling · no
internal/dev references in the UI · pluralise via `plural()` · dates via `localISO()` ·
right-hand panels shrink the page (`.rdrawer`/`body.dw-open`), full-screen takeovers use
the `body.onboarding`+`#onbRoot` pattern · preserve the consent round-trip, QR encoder,
onboarding, region select, two-way binding, themes, and all working round-1/2/3 behaviour.
Seed realistic, clearly-illustrative data; never real client data.

---

# PART A — DEFECTS FOUND IN A MANUAL BEHAVIOURAL AUDIT (fix these first)

These were found by driving the real UI. Each is a workflow loop that does not close.

1. **Meetings are one-way dead-ends.** Advisor Book to Send invite only composes an email
   and downloads an `.ics`; the client never sees the meeting. Client "Request meeting"
   only fires a toast; the advisor never sees the request.
   **Required behaviour:** a meeting is a real object both sides see. When the advisor sends
   an invite, the meeting appears in that client's CRM Meetings AND in the client app (a
   notification plus an "Upcoming meeting" card in Your Advisor), with accept/decline. When
   the client requests a meeting, it appears for the advisor as a to-do on the dashboard and
   an item on that client's profile, which the advisor can then confirm (turning it into a
   booked meeting). The editable email is the covering message, not the whole feature.

2. **Documents are upload-only; there is no request to fulfil loop.** The advisor's real job
   is collecting KYC/agreements/proof-of-address from clients.
   **Required behaviour:** on a client's Documents tab the advisor can **Request a document**
   (name + type). It shows as "Requested, awaiting client". In the client app the request
   appears (notification + a clear "Your advisor asked you to upload: Proof of Address" row)
   and the client uploads against it. The advisor then sees it move to "Received" with the
   file. Passive upload stays, but the request/fulfil loop is the point.

3. **Invite Admin/Member role is cosmetic.** No shared company page exists, so "admins
   control the landing page" means nothing.
   **Required behaviour:** a firm has ONE shared company landing page. An advisor whose role
   is Admin sees an editable "Firm page" surface (company name, logo, description, shared
   banner); a Member sees it read-only with a note that only admins can edit. Each advisor
   still has their own personal profile card on top of the shared firm page. Make the role
   change what the advisor can actually do.

4. **Submitted portfolios never come back approved.** The governance story has no payoff.
   **Required behaviour:** after an advisor submits a portfolio it shows "Pending approval",
   and the demo provides a believable approval moment: either a visible "Vault22 review"
   step the presenter can approve (a compliance/admin action in the prototype) that flips it
   to "Approved" and makes it appear in the live portfolio list, or a seeded second submitted
   portfolio already "Approved" so the before/after is on screen. The advisor must see the
   full lifecycle: submitted to approved to available.

5. **Tara feels like a lookup, not reasoning.** Per-line reasons exist but there is no
   overall rationale tied to the client.
   **Required behaviour:** at the top of Tara's suggestion, show a short, profile-driven
   rationale in plain language, e.g. "Zanele is 34, aggressive risk (84/100), a strong
   fitness score and only one dependent, so I have tilted heavily to equity and kept a small
   cash buffer." The mix must visibly follow from the client's real numbers, and the rationale
   must update when the advisor changes currency or the client's inputs differ. It should feel
   like Tara reasoned, not filtered a table.

6. **A dead `leadModal` code path still renders the old "Current" pill.** The rename to
   "Pending" was not applied everywhere.
   **Required behaviour:** remove the dead code, or make it consistent. Sweep the whole
   rendered app so "Current" (as an onboarding state), "Investor Type", "Unrealised G/L",
   "Your next actions", "Link/Linked", "Thresholds" and "How your book is related" appear
   NOWHERE, in any reachable state, tab, drawer or scenario.

---

# PART B — EVERY REQUIREMENT, BY INTENT

For each: the intent (why Stephen asked), the required behaviour, and the demo test that
proves it. Build to the behaviour, not the wording.

## 1. Advisor vs investor identity — Green/Navy + Light/Dark
- Intent: an advisor must always know they are in the B2B portal, not the consumer app; and
  the demo must look polished in any room lighting.
- Behaviour: both toggles work on click, from the topbar, and persist across every route and
  scenario. Navy should make the portal read as a distinct, professional B2B surface (not
  just one recoloured accent). Client app stays green.
- Demo test: click Navy, navigate three pages, still navy; click Dark, open a customer and a
  drawer, still dark and legible.

## 2. One click to the client
- Intent: advisors triage hundreds of clients; every extra click is friction times hundreds.
- Behaviour: clicking any registered user or customer lands directly on the full, tabbed
  profile. No intermediate slide-over. No dead alternate path.
- Demo test: from Registered Users, one click reaches the full profile; nothing else opens.

## 3. Onboarding tab + "Pending"
- Intent: the advisor's job is to unstick clients in the funnel; the label must name an action.
- Behaviour: the ten-stage lifecycle lives in an Onboarding tab; the in-progress stage reads
  "Pending" everywhere (list status included). Activity shows only recent/recurring events,
  never the onboarding stages again.
- Demo test: open a mid-funnel client; Onboarding shows exactly where they are stuck as
  "Pending"; Activity does not repeat those stages.

## 4. Documents — request to fulfil (see Part A.2)

## 5. Meetings — two-way (see Part A.1)

## 6. Notifications settings
- Intent: an advisor with hundreds of clients must control signal vs noise.
- Behaviour: a settings surface lets them switch notification types on/off and set amount and
  percentage thresholds; the bell then shows fewer, relevant items. Turning a type off or
  raising a threshold visibly changes the list.
- Demo test: switch off New Lead and raise the investment threshold; the bell list shrinks
  accordingly.

## 7. Qualified filtering
- Intent: lead prioritisation, where do I spend my limited time.
- Behaviour: the advisor sets what "qualified" means (net worth, monthly savings, income,
  fitness); the Registered Users list and the dashboard qualified count recompute live. An
  "All" chip sits left of "Qualified".
- Demo test: raise the net worth threshold; the qualified count drops and the list re-splits.

## 8. Customers table language and columns
- Intent: the table must speak the advisor's language and show the numbers they judge on.
- Behaviour: columns include Risk profile, Net worth, Profit/Loss and % change; no jargon
  ("Investor Type", "Unrealised G/L") anywhere. No horizontal clipping at 1440px.
- Demo test: scan the table; every column is meaningful to an advisor and correctly populated.

## 9. Portfolio module (see also Part A.4)
- Intent: advisors need the full Vault22 SA universe, searchable, drillable, and the ability
  to contribute their own portfolios under Vault22 governance.
- Behaviour: full seeded SA list (name-first, ticker, total invested), a search that filters,
  a View drawer with fund detail and the target allocation, and an Add-portfolio flow that
  submits for approval and completes the approval lifecycle (Part A.4).
- Demo test: search narrows the list; View shows a real fund sheet with allocation; submit a
  portfolio and show it becoming approved and appearing.

## 10. Analytics
- Intent: how is my book growing and where do clients come from, over a period I choose.
- Behaviour: a period selector (This month, Last month, Last 90 days, YTD, Custom) recomputes
  EVERY metric, funnel and chart together. "Customer acquisition" names the funnel; a
  "Registered on website" box names the channel; the monthly-growth chart sits at the bottom
  above Customer trends.
- Demo test: switch period; all figures change coherently, not just one chart.

## 11. Landing page vs profile
- Intent: two jobs. The landing page is the public storefront a prospect uses to register or
  contact the advisor; the profile is the advisor's identity in a future directory.
- Behaviour: the Landing Page reads as real marketing (uploaded image on one side, business
  description and CTA on the other) and lets a prospect register or get in touch. My Profile
  holds the "Preview profile" with an Edit that opens Customise in a right-hand layer, with
  every field (services, LinkedIn, website, email, phone, FSP licence, description, banner
  with recommended size, profile picture); edits reflect in the preview on Save.
- Demo test: the landing page looks like something an advisor would proudly share; a Customise
  edit shows up in the preview immediately.

## 12. Invite advisors + firm roles (see also Part A.3)
- Intent: the customers are IFA groups; onboard whole firms with real hierarchy.
- Behaviour: invite by email with optional names; a same-firm radio; if same firm, an
  Admin/Member role that has real consequence (Part A.3); a tooltip explaining admin rights.
- Demo test: invite a same-firm Admin and a Member; show that the Admin can edit the shared
  firm page and the Member cannot.

## 13. Audit vs Permissions
- Intent: two compliance stories. Permissions states the guardrails (what an advisor cannot
  do); Audit is the filterable activity history.
- Behaviour: Permissions is its own left-nav page (permissions in force + power of attorney);
  Audit Log holds only Recent activity, filterable by user, action and date, and the filters
  actually change the rows.
- Demo test: pick an action in the filter; the log narrows to matching rows.

## 14. Risk-assessment reminder
- Intent: suitability requires refreshing risk profiles about yearly.
- Behaviour: the advisor triggers a request; the client is notified in their app to redo it;
  the advisor sees the last-assessment date per client and can spot who is overdue.
- Demo test: request a reassessment; switch to the client and see the nudge; back on the
  advisor side, the client shows a last-assessment date.

## 15. Dashboard
- Intent: the advisor's morning cockpit.
- Behaviour: "To-do list" (renamed), each row deep-links to the exact filtered worklist it is
  about; Assets under advice shows a month-on-month delta.
- Demo test: click a to-do row; land on the filtered list of exactly those clients.

## 16. Membership plan and invoices
- Intent: the commercial relationship, shown cleanly.
- Behaviour: My Profile shows the plan (date joined, current monthly fee at the correct USD
  tier for the customer count), a list of monthly invoices each viewable in a drawer and
  downloadable as a real PDF, with the 25 bps non-Vault22 AUM fee shown as customer-paid and
  never part of the advisor's amount due. "Pricing to be confirmed" is stated.
- Demo test: open an invoice, download the PDF, confirm it opens; tiers are $10 / $25 / $40.

## 17. Tara personalisation (see also Part A.5)
- Intent: the headline moment, AI-assisted advice that closes the loop.
- Behaviour: from a customer, "Personalise portfolio" opens Tara full screen with the client's
  key info in the top third, a profile-driven rationale, an editable total, ZAR/USD filter,
  searchable universe, and per-line weight/amount/reason. Save stores a draft; Suggest (only at
  100%) sends it to the client, who is notified, can tweak, and can invest; the advisor's
  Investment tab reflects the status.
- Demo test: open Tara for two different-risk clients and see genuinely different, explained
  mixes; suggest, invest as the client, and see it reflected for the advisor.

---

# PART C — VALIDATION PROTOCOL (do this before declaring done)

Walk the prototype as Stephen would, using only the UI (click real buttons; never call
internal functions to fake a result). For every requirement in Parts A and B:

1. Perform the action by clicking, in both scenarios where relevant (advisor and client).
2. Confirm the visible consequence is real and believable, and that state updates correctly.
3. For every advisor↔client loop (meetings, documents, risk reassessment, Tara suggestion,
   consent), verify the OTHER side actually sees the result.
4. Re-run in Navy and Dark, and at 390px width, checking legibility and no overflow.
5. Confirm zero em dashes, zero `.io`, zero `undefined`/`NaN`/`[object Object]`, zero console
   or page errors, and none of the retired labels anywhere.

A requirement passes only when the click produces the intended workflow outcome, not when the
element merely exists. When everything passes, write a short walkthrough script the presenter
can follow to tell the full advisor story in the demo, and list anything still seeded that
needs real data (Uday's securities and portfolio lists, final subscription pricing and the
25 bps fee with Greg).

# Definition of done
Stephen navigates every flow by clicking, and has minimal feedback because each interaction
does exactly what he meant, the advisor↔client loops close, and the whole thing reads as one
cohesive, production-quality product rather than a set of mocked screens.

# QA Report — Vault22 Advisor Prototype

**Build:** working tree (uncommitted) · **Live:** https://aishaniatri0.github.io/vault22-advisor-prototype/
(the live URL is still the OLD build — these fixes are local until pushed)

**Requirements source:** `SPEC-V2.md`. **Live UI ground truth:**
`../Vault22-Insights-Todo-Prototype/reference/dom/00-dashboard.html` (captured live DOM).

## How this was verified

Every fix is driven against the running app in headless Chrome, not judged by reading
source. **106 automated assertions across 8 harnesses, 0 failing**, plus four
independent audit passes (spec compliance, UI screen-by-screen, end-to-end flows,
and a cross-check of every prior piece of feedback).

## Status against all feedback

| Area | State |
|---|---|
| Every prior feedback item (16 tracked) | Applied and driven. The one that had been fixed in one place only — the Detail/Value **table** — is now unified through the shared `dl()` helper, so the advisor's §8 tabs and the client side use the identical treatment. |
| Back button on every page reached from the dashboard | Injected centrally in `render()`, so it cannot drift per-template. Verified on all 10 routes + the customer detail; none stray onto a top-level visit. |
| Em dashes in rendered copy | Zero, across every route, tab, modal, onboarding screen, and the client view. Gated on `innerText`; comments may keep them. |
| "Needs you / N open items" | Replaced with "Your next actions": each row names the action, the people, and lands on the FILTERED list. |
| Requirement Zero (client profile) | Rebuilt. The advisor section now sits INSIDE a real profile: completion score with the live app's point-weighted "5 pts" item, a picture upload, Personal details (the §8 fields, ID masked), Account rows, then the advisor sections. |
| Dashboard | Rebuilt to the live composition (gradient hero, promo card, tiles, largest clients, next-actions, activity). |
| Heading/type system | Corrected to the captured live DOM (#22252B headings), reversing a prior brief that had it wrong. |
| Tables | Registered Users trimmed 12→6 columns, Customers 11→6, so they fit 1440px without clipping. |
| Analytics | Relationship map made horizontal with big numbers; retention shown as a headline figure, formula demoted to a caption. |
| My Profile | Status switcher moved to the demo drawer (Suspended stays reachable per §1, but is no longer a customer-facing control); columns rebalanced. |
| Drawer bug | Right drawers no longer break numbers mid-digit ("R192k"). |
| Spec compliance | Every numbered section driven; every KPI/analytics figure recomputed from state and matched. No hardcoded/lying numbers. |
| Interaction integrity | 0 dead controls, 0 orphaned data-*, 0 exceptions across a full clickstorm, 0 console errors. |

## Open decisions for Stephen / Uday (not defects)

1. **Advisor URL literal.** The build uses `aishani.advisor.vault22.com`; the spec's
   examples read `advisorname.vault22.com` (no `.advisor` segment). Deliberate
   namespacing, but it is on nearly every screen and in the QR. One-line change if the
   spec's literal is preferred.
2. **Suspended enforcement.** The status is reachable and labelled "read-only", but the
   portal is not actually locked down when suspended. Either gate mutations on
   `status==='Suspended'`, or keep it as a status indicator only.
3. **Risk ladder.** TYPES has 5 levels (Very Conservative + the spec's 4). The doc names
   4. Which wins?
4. **PII masking scope.** The doc carries its own open question ("Udaykant, is this
   necessary?"). `maskId()` is built on the assumption it is.

## Honest limits

- The client Profile's Security / Notifications / Linked accounts / Subscription rows
  are honest stubs (out of scope: this prototype is the advisor portal). Same for the
  §8 Documents library, the Investment "financial profile report", and the landing
  "Start Investing" CTA (the investor journey is deliberately unchanged).
- A few figures are modest by design now that they are derived, not inflated — e.g.
  Revenue/commission is 0.75%/yr on the book, so ~R1,150/month rather than the old
  hardcoded R18,600. Correct, but a data choice worth confirming for the demo.

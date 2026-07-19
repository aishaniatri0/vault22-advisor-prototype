# Vault22 Advisor Prototype — QA Fix Prompt

You are fixing `index.html` (single-file SPA, ~6283 lines) for the Vault22 Financial Advisor prototype ahead of live demos with South African IFA groups. Every issue below was found in an adversarial QA pass against Stephen's original email requirements.

## Ground rules
- **Source of truth = Stephen's email requirements**, not the current implementation.
- Fix behaviour, not just labels. A button that exists but doesn't do the exact thing Stephen asked = still broken.
- Keep the existing architecture (in-memory `state`, `render()` re-render loop, `openModal`/`.rdrawer`, audit logging via `audit(...)`). Don't introduce a framework or a build step.
- After each fix, re-check the acceptance criterion by tracing the handler, not by assuming.
- Preserve all currently-passing behaviour (Analytics, Permissions split, qualification settings, doc upload, portfolio rework, landing/profile split, invite flow, themes, Tara core loop). Don't regress them.
- Line numbers are pointers from the QA pass; they will drift as you edit — locate by nearby code, not by absolute line.

---

## P0 — Demo blockers (fix first)

### 1. Risk reassessment must be triggerable for every client
**Where:** `tab8Risk` ~`:3172,3180`; handler `data-reqrisk` ~`:5625`.
**Now:** The "Request risk assessment" trigger and the "Last risk assessment" date live only inside the Risk tab, which renders `lockedTab(...)` when risk view-consent isn't `granted`. For clients C3 (pending) and C4 (none) the trigger is completely hidden.
**Required:** Triggering an annual reassessment is independent of financial-data view consent. Surface the "Request risk assessment" button **and** the "Last risk assessment" date somewhere reachable for **all** customers regardless of consent state — e.g. on the customer profile header or the Onboarding tab. Keep the existing handler (it already audits + notifies).
**Accept:** Open C3 and C4 → you can see the last-assessment date and click "Request risk assessment" without granting financial consent; action logs to audit.

### 2. "Send invite" / "Request meeting" should open a real editable email with calendar invite
**Where:** `bookDrawer` "Send invite" ~`:5107`; user-side "Request meeting" ~`:4935`; compose/send `data-csend` ~`:5577`; `.ics` chip ~`:5233`; `buildICS` ~`:5151`.
**Now:** Both open an in-app compose form (good — editable, nothing auto-sent), but **no `mailto:` ever opens**, and the "📎 Calendar invite attached" chip is cosmetic — the `.ics` is only a separate manual download.
**Required:** On final Send, actually open the user's mail client via a `mailto:` link prefilled with the edited To/Subject/Body, so the advisor lands in their own email to review and send. Keep booking the meeting internally + logging to audit. Since a raw `mailto:` can't carry an attachment reliably, also auto-trigger the `.ics` download (or make the "Calendar invite" chip a real download link) and add one line of body copy telling them to attach it. Update the chip copy so it's not claiming an attachment that isn't there.
**Accept:** Advisor books a meeting → edits compose → Send → mail client opens prefilled with their edits; the .ics is genuinely obtainable, not a fake chip. Same for the user-side request-meeting flow.

### 3. "Request meeting" must be available to users with no advisor yet
**Where:** user profile "Request meeting" gate ~`:4933-4935` (`${linked ? … : ''}`).
**Now:** The button only renders once an advisor is appointed, so an un-appointed user can't request a meeting — contradicting Stephen's "with their advisor **and other advisors**."
**Required:** Show "Request meeting" even when unlinked; the compose drawer's advisor `<select>` (`ADVISOR_DIR`) already lets them pick any advisor — just don't hide the entry point.
**Accept:** As an unlinked user, "Request meeting" is visible and opens compose with a choosable advisor.

### 4. Merge Activity timeline and Onboarding stages
**Where:** `TABS8` ~`:2852-2853`; `tab8Onboarding` ~`:3265`; `tab8Activity` ~`:3283`.
**Now:** They are two separate tabs. Stephen said: *"Activity timeline seems similar to the stages after onboarding — suggest to merge."*
**Required:** Combine into a single tab (e.g. "Activity") that shows the onboarding milestones and the post-onboarding recurring events on one timeline, deduplicated. Remove the redundant tab. Keep the onboarding-stage status pills.
**Accept:** One combined timeline tab; no duplicate stage/activity views; nothing lost.

---

## P1 — Should fix before sending to Stephen

### 5. Notification-settings: stop discarding unsaved amount edits
**Where:** toggle handler ~`:5513-5518` (calls full `render()`); amounts read from `state.nprefs`, only written on Save ~`:5520`.
**Now:** Flipping any toggle re-renders and reverts in-progress amount fields to the last saved value.
**Required:** Either (a) commit amount-field changes to `state.nprefs` on input so a re-render preserves them, or (b) make toggles persist without a full destructive re-render. Recommended: write both toggles and amounts to state immediately, and either drop the "Save" button or keep it purely as confirmation. Make the save model symmetric.
**Accept:** Set "Investment over" = 200,000 (no save) → toggle Birthdays → amount stays 200,000.

### 6. Add-portfolio form needs a distinct "Fees" field
**Where:** add-portfolio form ~`:3628-3630`; submit handler ~`:5774`.
**Now:** Only a single "Fees factsheet" file upload; no numeric/text fee (TER/annual fee %) captured or stored.
**Required:** Add a "Fees" input (e.g. annual fee %/TER) as its own field, persist it on the submitted portfolio object, and surface it in the portfolio View drawer.
**Accept:** Advisor enters a fee → it saves → shows in the fund detail drawer.

### 7. Confirm & apply the Membership AUM-fee billing decision
**Where:** `membershipCard`/invoice ~`:2953-2955,3040-3041,3089-3093`.
**Now:** The 25bps AUM fee is computed and shown but declared paid by the customer, so "amount due from you" = subscription only. Code says "to be confirmed with Greg."
**Required:** This is a product decision, not a code bug — flag it to Stephen/Greg. Once confirmed, either (a) keep as-is and label clearly that the AUM fee is customer-borne, or (b) include it in the advisor's amount-due total. Make the invoice's "amount due" unambiguous either way.
**Accept:** Invoice states plainly who each fee is billed to; total matches that statement.

### 8. Fix the "All" registered-users chip
**Where:** handler ~`:5904` (only deletes `qual`/`unqual`).
**Now:** Clicking "All" leaves other active filters (e.g. KYC Pending) applied, so the table stays filtered.
**Required:** Decide the intent — if "All" means "show everyone," clear all filters; if it only resets the qualified segment, rename it so it doesn't imply showing everyone. Recommended: "All" clears the qualified/unqualified segment only, and keep the separate "Clear all ✕" for everything (already present) — but make the two visually distinct so it's not confusing.
**Accept:** Behaviour matches the label; no stale hidden filtering.

---

## P2 — Copy, labels, small behaviour

### 9. Remove leftover "unlinked / Re-linking" copy in the Appoint context
**Where:** relink card ~`:4957-4958`.
**Now:** User-visible text still says "you unlinked them. Re-linking does not restore what you shared before."
**Required:** Rename to appoint/unappoint language: e.g. "you removed them as your advisor. Appointing them again does not restore what you previously shared." (Internal identifiers like `state.linkedAdvisor` can stay.)

### 10. Tara: don't wipe manual edits on currency switch
**Where:** `data-taraccy` handler ~`:5699-5703` (`state.tara.lines = taraBuild(...)`).
**Now:** Toggling ZAR⇄USD regenerates the default suggestion, discarding weight tweaks and added/removed assets.
**Required:** Preserve the advisor's current lines on currency toggle (only filter/relabel the available asset universe by currency), or prompt before regenerating.
**Accept:** Tweak weights → switch USD → switch back → edits intact.

### 11. Onboarding tab label + duplicate stage pill
**Where:** tab label ~`:2852` ("Onboarding"); Personal tab Status row ~`:2906`.
**Required:** Rename the tab/header to "Onboarding stage" (Stephen's exact word). Remove the duplicate onboarding-stage pill from the Personal tab (keep it in one place).

### 12. Document type should not silently default to "Identity"
**Where:** file-picker ~`:1854`; drag-drop ~`:6087`.
**Now:** A dropped file with an untouched dropdown saves as type "Identity."
**Required:** Default the draft type to empty/"Select type…" and require a selection before Save enables (both advisor-side and user-side upload).

### 13. Surface Pending KYC & Pending Risk Assessments as KPI tiles
**Where:** currently only to-do rows ~`:2432`.
**Required:** Spec §7 lists them as dashboard KPIs. Add them as proper KPI tiles (they can also remain as to-do rows), using the live `pkyc`/`prisk` counts.

### 14. Same-firm control as a real radio group (optional / low)
**Where:** invite same-firm toggle ~`:5271-5285`.
**Now:** Yes/No chip buttons. Behaviour is correct.
**Required (if time):** Use `<input type="radio">` for semantic/accessibility correctness, keeping the conditional Admin/Member dropdown + tooltip.

---

## P3 — Polish

- **15.** AUA delta sub-label: make it grey (use `--muted`, matching other stat sub-labels), wrap in parentheses `(+X.X% from last month)`, and place it beside/under the "Assets under advice" label consistently. `~:2513`
- **16.** Customer table headers: use spec casing "Risk Profile" and "Net Worth". `~:2828`
- **17.** Remove the redundant full-width "Customise profile" button (the top-right Edit already opens the same drawer). `~:4473`
- **18.** Membership tier boundary: confirm whether exactly 50 customers is $25 or $40; current code = $25 at 50, $40 at 51+. Align labels + logic once confirmed. `~:2960`
- **19.** Tara: split the asset universe into labelled sections (ETFs / single stocks / other securities) rather than one mixed list with per-row type labels. `~:4394`
- **20.** Demo persistence (nice-to-have): persist `state` to `sessionStorage` so an accidental refresh mid-demo doesn't wipe uploaded docs, consent grants, notes, and Tara drafts. Currently all state is in-memory (`~:1563`).
- **21.** Confirm Customer Trends "change in liabilities" color inversion (down = green) is intended; if so, leave a comment noting it. `~:2813`

---

## Definition of done
- P0 items 1–4 all verified by tracing the actual handler/flow.
- P1 items 5–8 fixed or (for 7) explicitly flagged with a clear invoice statement.
- No regression in any currently-passing area.
- A quick self-review pass: for each fixed item, would Stephen click it and say "yes, that's what I meant"? If not, it's not done.

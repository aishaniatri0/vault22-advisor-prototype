# Manual test pack — Stephen's 19 July requirements

Every requirement Stephen raised in his **19 July 2026, 21:33** email on the *Advisor Dashboard*
thread ("this is great we are 95% there"), with one before/after image each.

- **BEFORE** = build `2026-07-19.7` — the build Stephen actually reviewed.
- **AFTER** = build `2026-07-21.1` — currently deployed at
  https://aishaniatri0.github.io/vault22-advisor-prototype/

Each image is self-contained: the requirement in Stephen's own words, where to click, before on the
left, after on the right, and a red line at the bottom naming what changed. Images are numbered in
test-case order.

**Before testing:** hard refresh (**Cmd+Shift+R**). Confirm the build via the gear icon, bottom-right.
Scenario switching (Advisor ↔ Client) is also in that gear.

| # | Image | Requirement | Where to look | Result |
|---|---|---|---|---|
| TC-01 | `TC-01.png` | Charts 2 across and 2 high, with a full-screen button | Advisor → Analytics → scroll to charts | PASS |
| TC-02 | `TC-02.png` | Date format dd-mm-yyyy throughout the app | Advisor → Customers → Lerato Khumalo → Personal | PASS |
| TC-03 | `TC-03.png` | Use the designed financial report, not the basic list | Advisor → Customers → Lerato Khumalo → Financial | PASS |
| TC-04 | `TC-04.png` | Remove "Total fees earned" from the user view | Client → Profile → What your advisor can access | PASS |
| TC-05 | `TC-05.png` | Rename New Lead → New User; remove WhatsApp channel | Advisor → bell icon → Settings | PASS |
| TC-06 | `TC-06.png` | Registered Users filter: rename "any" to "all" | Advisor → Registered Users → filter chips | PASS |
| TC-07 | `TC-07.png` | Remove duplicate Book meeting / Request risk below the boxes | Advisor → Customers → Lerato → Overview | PASS |
| TC-08 | `TC-08.png` | Move "Request all data access" up with the other actions | Advisor → Customers → Chantal Meyer → header row | PASS |
| TC-09 | `TC-09.png` | Company logo on both surfaces, and removable | Advisor → My Profile → Company logo | PASS |
| TC-10 | `TC-10.png` | Invoice: logo, client name, address, invoice date, VAT; remove Date joined | Advisor → My Profile → Membership → Download invoice | PASS |
| TC-11 | `TC-11.png` | Move the directory to the user side as "Search for an advisor" | Client → Profile → Your Advisor | PASS |
| TC-12 | `TC-12.png` | Remove the "Change advisor" section | Client → Profile → Your Advisor | PASS |
| TC-13 | `TC-13.png` | Audit log filters by user name, action and date | Advisor → Audit Log | PASS |
| TC-14 | `TC-14.png` | Suggested portfolio must not show the green "Invested" label | Client → Profile → Your Advisor → suggestion card | PASS |
| TC-15 | `TC-15.png` | Invest per asset, Sell on disposals, and an "Action all" button | Client → Profile → Your Advisor → suggestion card | PASS |
| TC-16 | `TC-16.png` | Add a "Current holding" column | Client → Profile → Your Advisor → suggestion card | PASS |
| TC-17 | `TC-17.png` | Moderation queue belongs in the admin section | Advisor → Portfolios (before) · gear → Vault22 admin (now) | PASS |
| TC-18 | `TC-18.png` | Remove Approve / Reject from the advisor view | Advisor → Portfolios → your submissions | PASS |

## Notes on individual cases

**TC-13** — the filters existed before Stephen re-raised them, but have since been reworked: the client
filter is a search box over every client (not a dropdown of only those with activity), the action filter
is a fixed taxonomy that no longer grows as the demo is used, the row count updates live as you type,
and Export CSV now exports exactly what is on screen. Still worth confirming back to him so the
re-raise is visibly closed.

**TC-14** — in the 19.7 build the pill on a *not yet invested* suggestion read "Total 100%". It now
reads "Suggested · total 100%", naming the state explicitly. The green "Invested" pill still exists,
but only appears once the client has actually invested. **Worth confirming with Stephen** that he is
happy with "Invested" as a genuine post-action state, rather than gone entirely.

**TC-10** — the invoice logo was the last outstanding failure and is now fixed. The PDF writer had no
image support at all, so the file previously printed a typed "VAULT22" wordmark while the on-screen
drawer showed a real logo. It now embeds the actual mark.

## Still open — Stephen's calls, not ours

1. **TC-14 / "Invested"** — acceptable as a post-action state? (see above)
2. **TC-13 / audit filters** — confirm back to him that his re-raise is closed.
3. **Full SA portfolio list** — blocked on Uday.
4. **Membership pricing** ($10 / $25 / $40 tiers) — to be confirmed with Greg.
5. **Where the moderation queue finally lives in project x** — Uday's call. It now has its own
   **Vault22 admin** scenario, nav item and page (gear → scenario → Vault22 admin), so it is
   demonstrable wherever it eventually lands.

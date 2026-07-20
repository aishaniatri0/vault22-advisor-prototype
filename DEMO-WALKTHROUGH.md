# Vault22 Advisor Prototype — presenter demo walkthrough

A click-by-click script to tell the full advisor story, with the advisor↔client loops
closing live in front of the room. Every step below is driven by clicking real controls.
Switch scenarios with the gear (bottom-right): **New advisor**, **Established advisor**,
**Financial Advisor view** (the client app).

## 1. The pitch, in 30 seconds (New advisor)
- Open the app: it boots into **advisor onboarding**. Register, pick a region, verify, choose
  an advisor name, and land on your link + QR. Every signup through them is tagged to you.
- Gear → **Established advisor** to demo a populated book (Aishani Atri, 5 customers).

## 2. The morning cockpit (Dashboard)
- **To-do list**: Chase KYC, chase risk, follow up qualified users, and **Confirm meeting
  requests** (a client asked to meet). Click a row to land on exactly that filtered worklist.
- **Assets under advice** shows a month-on-month delta.
- Top-right: flip **Navy view** and **Dark** to show the same portal reads as a professional
  B2B surface in any room. Navigate a few pages, both persist. (The client app stays green.)

## 3. One click to a client, and unsticking the funnel
- **Registered Users** → click any row → straight to the full tabbed profile.
- **Onboarding** tab shows the ten-stage lifecycle; the in-progress stage reads **Pending**.
- On the list, **Edit settings** under Qualified user filtering: raise the net-worth threshold
  and the qualified count and split recompute live. The **All** chip clears the split.

## 4. Documents: request to fulfil (loop)
- Open **Lerato Khumalo** → **Documents**. You already requested a payslip: it shows
  **Awaiting client**. Click **Request a document**, ask for "Proof of address".
- Gear → **Financial Advisor view** → **Documents library**: the client sees
  "Your advisor asked you to upload: Proof of address", uploads a file.
- Gear → **Established advisor** → Lerato → Documents: it is now **Received** with the file.

## 5. Meetings: two-way (loop)
- On a customer, **Book** → pick a time → **Send invite** opens an editable email → **Send**.
- Financial Advisor view → **Your Advisor** → **Meetings**: the client sees the invite with
  **Accept / Decline**. Accept it.
- Back on the advisor side, the customer's **Meetings** shows **Confirmed**.
- Reverse: in the client app, **Request meeting**; the advisor's **Dashboard to-do** shows a
  request; open the client and **Confirm a time** to book it.

## 6. Tara: AI-assisted advice (the headline moment) (loop)
- On a customer, **Personalise portfolio** opens **Tara** full screen. Client key info sits up
  top; **Tara's thinking** explains the mix in plain language from the client's real numbers
  (open an Aggressive and a Conservative client to show genuinely different, explained mixes).
- Tweak weights with +/-, edit the **Total invested**, switch **ZAR/USD** (the rationale and
  universe update). **Suggest to client** (enabled at 100%).
- Financial Advisor view → **Your Advisor**: the client sees the suggested portfolio, can
  tweak the weights, and **Invest**. Back on the advisor side, the Investment tab reflects it.

## 7. Portfolios: submit to approved to available (loop)
- **Portfolios**: the full SA universe, searchable; **View** any fund for detail + allocation.
- **Client side → Profile → Your Advisor → Search for an advisor**: search the directory and
  **Appoint** an advisor. The choice persists across a scenario switch.
- **Add portfolio** → submit → it shows **Pending approval** (read-only for the advisor).
  Approve it from the settings gear → **Vault22 admin** moderation queue
  (the moderation stand-in) → it flips to **Approved** and appears in the live list, tagged
  **Yours**. A second submission is already Approved to show the before/after.

## 8. Firm hierarchy (loop)
- **Landing Page** leads with the **shared firm page** (one company page for the whole firm),
  with your personal card on top. As **Admin** you can **Edit firm page**; toggle **Member
  view** to show it read-only. **Invite advisors** sets same-firm Admin/Member roles.

## 9. The commercial story (Membership)
- **My Profile** → **Membership plan**: date joined, current monthly fee at the right USD tier
  ($10 / $25 / $40 by customer count), and monthly invoices. **View** one, **Download PDF**
  (a real file). The 25 bps non-Vault22 AUM fee is shown as customer-paid, never your amount due.

## 10. Analytics, risk, compliance
- **Analytics**: change the **period** (This month / Last month / Last 90 days / YTD / Custom)
  and every KPI, funnel and chart moves together. "Customer acquisition" + "Registered on
  website" name the channel.
- **Risk**: on a customer, **Request risk assessment** nudges the client (see it in their app);
  the last-assessment date shows per client.
- **Permissions** (guardrails + power of attorney) and **Audit Log** (filter by user / action /
  date) are separate compliance pages.

## The close
Every button advances a believable advisor workflow, and every advisor↔client loop closes.
No dead ends; no `undefined`/`NaN`; no jargon; works in Navy and Dark and at phone width.

## Seeded, needs real data later
- Uday's real SA securities universe (Tara assets) and portfolio list.
- Final subscription pricing and the 25 bps non-Vault22 AUM fee (to be confirmed with Greg).
- Historic AUM curve on older invoices.
- **Open with Uday:** where the portfolio **moderation queue** finally lives in project x. It sits
  behind the settings gear → **Vault22 admin** for now.
- **Invoice PDF** carries a typed Vault22 wordmark, not the logo image: the hand-rolled PDF writer
  has no image support.

## Vault22 admin (portfolio moderation)
Settings gear → Scenario → **Vault22 admin**. The left nav becomes **Vault22 admin → Portfolio
moderation**, listing advisor-submitted portfolios with Approve / Reject. Approving puts the
portfolio live on the advisor's platform (switch back to Established advisor → Portfolios to show it).
Advisors cannot approve their own submissions; their Add-a-portfolio screen is read-only.

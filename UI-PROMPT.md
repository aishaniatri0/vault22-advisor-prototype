# Advisor Prototype — UI parity with the live app

Paste below the line into a fresh Claude Code session.

---

Work on `/Users/aishaniatri/Vault22-Advisor-Prototype/index.html`. Single file,
vanilla JS, no build. **Read it first.** Live at
https://aishaniatri0.github.io/vault22-advisor-prototype/ — keep byte-identical.

**Scope: visual fidelity of every screen except onboarding.** The advisor dashboard,
the customer surface, and the client / Financial Advisor view all look wrong. The
functionality is done and verified — the consent round trip works in both directions
with no data leak. **Do not touch behaviour. This is a look-and-feel pass.**

> **CORRECTION (added after the fact).** The colour guidance below is WRONG and was
> disproved by the captured live DOM at
> `../Vault22-Insights-Todo-Prototype/reference/dom/00-dashboard.html`. The live app's
> card/section headings are `text-xl font-medium text-[#22252B]` (#22252B, 52 uses),
> and the page h1 is `text-[32px] font-semibold text-gray-900`. #1E3A5F appears 7
> times — as gradients and the fitness *level* label — NOT as a heading colour.
> Trust the capture, not this brief, on colour. The rest (density, composition,
> eyebrow tier) still holds.

## Read this before anything else: the previous briefs were wrong

Aishani has said four times that the UI does not match the live app. Each previous
brief handed over a **token table** — hex values, radii, a type scale — and each
session matched the tokens faithfully. The result still looks wrong, because the
briefs contained two actual errors and a wrong method.

### Error 1 — a previous brief deleted a real live pattern

A brief claimed: *"Our uppercase micro-labels ('HOLDINGS', 'FINANCIAL SNAPSHOT',
'LEAD FUNNEL') appear nowhere in the live app. Fixing that one class does more for
parity than everything else."*

**That is false.** The live app's own stylesheet — read off the live DOM — has **ten**
uppercase label classes:

```
.ins-mod       11px/700 uppercase .04em      card category eyebrow
.sect-h        12px/700 uppercase            section divider ("EVERYTHING ELSE")
.best-eyebrow  10px/700 uppercase .08em      hero card eyebrow
.stat-l        11px/700 uppercase .04em      stat tile label
.td-mod  .chip  .wz-lbl  .todo-h  .due-strip-l  .code-box
```

The original `.sec-t{font-size:11px;font-weight:700;letter-spacing:.09em;
text-transform:uppercase}` was **almost exactly `.ins-mod`**. It was correct. The
brief had it deleted and replaced with `20px/500` sentence case, which flattened
every heading in the prototype into one weight. **That is the single biggest cause of
the "scattered", "flat" look.** Restore the eyebrow tier.

### Error 2 — auth-page tokens were applied to app pages

The live **auth** page (`/register`, `/login`) and the live **app** pages have
different systems. A brief lifted auth tokens and applied them everywhere:

| Token | Belongs to | Was applied to |
|---|---|---|
| `h-14` (56px) fields | auth forms | **every input, including in-table search** |
| `text-[#22252B]` | auth **input text** | **our section headings** (`--live-head`) |
| `text-[#162B45]` | auth CTA | fine, it is the CTA |

The h-14 search bar was already caught and fixed. **The heading colour was not.**

### The colour bug you can see right now

> NOTE: this section is the part the correction above overturns — the captured live
> DOM shows #22252B IS the heading colour. Left here for history.

```
ours:  h1      { color: var(--text) }      → --g-900  → #111827   near-black
ours:  .sec-t  { color: var(--live-head) } → #22252B             near-black
live:  --heading: #1E3A5F                                        deep navy
```

### The method that has been failing

Token tables. Stop. **Open the live app and ours side by side and fix what you see.**

---

# P1 — DENSITY AND COMPOSITION

This is the real gap and no token will fix it. Render both and look.

**The live app is dense and layered. Ours is sparse and flat.** On one live screen:

- A **hero card** — dark navy/green gradient, a photo, a title, and *nested
  translucent cards inside it*, each with an eyebrow, a status pill, a bold headline,
  body copy, a divider, and a button pair.
- A **section divider** — uppercase, muted.
- A **stat tile row** — big number, label, sublabel.
- A **chip filter row** — the active chip solid green.
- A **3-column card grid** below.

Fix by composition, per screen. Lead with something. Tighten the vertical rhythm
(live gap is 14–18px). Give cards internal structure. Use stat tiles.

**Do not invent a Vault22 pattern.** Lift from the captured live DOM.

---

# CONSTRAINTS

**Behaviour is done. Do not touch** the consent round trip, the QR encoder, the Back
button logic, region selection, or onboarding. Keep passing: em dashes 0 in the
rendered DOM, money `R2,450,000`, 390px no overflow, zero orphaned `data-*`.

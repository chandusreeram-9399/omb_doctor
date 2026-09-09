# Homepage revamp — plan

We want a new homepage on the live customer site. Header, footer, and everything on the home page should look like the new design. When someone clicks Health or Motor, they go to the **same pages they use today** — we are not rebuilding how people buy insurance.

**Where we code:** `C:\Ombrela\customer`  
**Where we look for design:** `Ombrella Homepage Redesign` folder (reference only — we do not deploy this folder)

---

## The big picture

Right now the live site and the new design are two different codebases. The new one was built to show how the homepage should look. We take that look and build it **inside the customer repo**, using the tools that repo already uses (Next.js, SCSS, Bootstrap).

Think of it like this:

```
User opens ombrella.com
        │
        ▼
   NEW homepage  ← only this changes
        │
        │  clicks "Health Insurance"
        ▼
   OLD flow  /v2/health-insurance/prequotes  ← untouched
        │
        ▼
   same questions, plans, payment as today
```

The little **chatbot in the corner** stays. We do not remove it.

---

## What changes

- Header (new look, same login behaviour)
- Footer (new look, same links)
- Home page content (hero, categories, Dr. Ombrella box, risk score teaser, FAQ, etc.)
- One **new page**: `/risk-score` (free quiz — not on the old site yet)

## What we leave alone

- Every page under `pages/v2/` (health, motor, life, etc.)
- Gadget, travel, payment, login, sign-up
- The `services/` folder (all API calls for buying)
- The chatbot (`ChatWidget` + `pages/api/webchat/`)

If we only touch homepage + header + footer, we cannot break checkout.

---



---

## Homepage sections (top to bottom)

This is the order on the new design. Each row says if we need backend work.

1. **Hero** — rotating banners, main CTA  
   - Backend: no  
   - Links: scroll down or go to a prequote page  

2. **Category tiles** — Health, Motor, Life, Travel, etc.  
   - Backend: no  
   - Links: existing prequote URLs (see table below)  

3. **Partner logos** — insurer names/logos  
   - Backend: no  

4. **Dr. Ombrella** — user types "I had a baby" → sees suggested plans  
   - Backend: **yes** (plan list in DB + small API — explained later)  

5. **Popular plans** — a few cards with starting prices  
   - Backend: optional (can be static at first)  

6. **Ombrella Advantage, Why Ombrella, Testimonials, FAQ**  
   - Backend: no  

7. **Risk score teaser** — "Check my risk score in 60 seconds"  
   - Backend: no (links to `/risk-score`)  

8. **Bottom CTA + sticky bar on mobile**  
   - Backend: no  

All of this needs to work on phone and desktop.

---

## Where buttons must go (same as today)

These URLs already exist on the live site. The new homepage must use **exactly these**, not new ones.

| Button / product      | URL |
|-----------------------|-----|
| Health                | `/v2/health-insurance/prequotes` |
| Motor                 | `/v2/motor/prequotes` |
| Life                  | `/v2/life-insurance/prequotes` |
| Education             | `/v2/education/prequotes` |
| Savings               | `/v2/savings-insurance/prequotes` |
| Travel                | `/travel-insurance/prequotes` |
| Gadget                | `/gadget` |
| About                 | `/about-us` |
| Contact               | `/contact-us` |
| Login / Sign up       | `/login` , `/sign-up` |
| Risk score (new page) | `/risk-score` |

You can double-check these in `footer.js` and `insurance-menu.js` in the customer repo.

---

## Risk score — how it fits in

**On the homepage:** a band that says something like "Check my risk score" → user goes to `/risk-score`.

**On `/risk-score`:**  
- 8 simple tap questions (age, family, income, etc.)  
- Name + phone  
- A score out of 100 and a short "what you're missing" report  
- Buttons like "Get health cover" → normal prequote URLs above  
- Optional: download a report (built in the browser)

**Backend for launch:** none. The maths and questions live in a JS file we port from the redesign (`utils/risk-score.js`).

**Later (only if product wants it):** save name/phone/score to User Service or Insurance Service so sales can call people back. That is a separate small API — not needed to ship the new homepage.

Risk score and Dr. Ombrella are different:
- **Risk score** = "How exposed am I?" (fixed questions → one number)  
- **Dr. Ombrella** = "What plans fit my life?" (free text → plan suggestions)

---

## Dr. Ombrella — how it fits in (needs backend)

The prototype uses a fake list of plans in a file. Production should use **real plans you store in your database** (names, insurers, guide prices, which prequote link to open).

### Flow

```
User types on homepage
       ↓
Customer site calls our own API route  (pages/api/homepage/cover-finder.js)
       ↓
That route loads plans from Insurance Service DB
       ↓
AI (or simple keyword match) picks 2–3 plan IDs from that list only
       ↓
UI shows plans — prices always from DB, never invented by AI
       ↓
User clicks → prequote_url from DB → normal buy flow
```

### Insurance Service team builds

- A table for homepage plans (id, insurer, name, price from, tags, link to prequote, active yes/no)
- One read API: get all active plans for the homepage

This does **not** replace the existing plan APIs used during checkout. It is only for the homepage recommender.

### Customer site team builds

- `pages/api/homepage/cover-finder.js` — runs on the server, holds AI keys
- `CoverFinder.js` component on the homepage
- Production AI: either extend your existing **chatbot backend**, or call **Gemini/OpenAI** from that API route — team's choice. Keys live in server env only.

If AI is down, we still work: match keywords against plan tags in the DB (same idea as the prototype fallback).

---

## Chatbot

Already works. Lives in `website-layout.js` as `<ChatWidget />`.

We only need to make sure new homepage CSS does not cover it (z-index). No backend changes.

---

## Files we will add or change (customer repo)

**Change:**
- `component/layout/website-layout/header.js`
- `component/layout/website-layout/footer.js`
- `pages/index.js`

**Add:**
- `component/home/v2/` — one file per homepage section
- `pages/risk-score.js`
- `utils/risk-score.js`
- `styles/homepage-v2.scss`
- `pages/api/homepage/cover-finder.js` (when Dr. Ombrella is wired to DB)

**Do not open:**
- `pages/v2/**`
- `services/**`

Keep a backup of old `pages/index.js` before replacing.

---

## How to do the work (step by step)

**Step 1 — Branch and backup**  
Copy old homepage. Create a branch.

**Step 2 — Header and footer**  
New design, but keep login/logout and all the same links. Test footer links open the right prequote pages.

**Step 3 — Homepage without Dr. Ombrella**  
Build hero, categories, FAQ, etc. Hook every product tile to the URL table. Test on mobile. Check chatbot still shows.

**Step 4 — Risk score**  
Add the page and the teaser on the home page. Walk through the quiz once on a phone. Check result buttons go to prequotes.

**Step 5 — Plan catalog (backend)**  
Insurance Service adds the table + GET API. Someone fills in real plans product approves.

**Step 6 — Dr. Ombrella**  
Wire API route + homepage component to that catalog. Test a few prompts.

**Step 7 — Quick regression**  
Open homepage → health prequote → go a few steps into old flow. If that still works, we did not break buying.

**Step 8 — Deploy**  
Deploy customer repo to QA, then prod.

---

## Things to decide before we start

1. New header/footer on **every page**, or only homepage?  
2. Dr. Ombrella day one: full AI or keyword-only until AI is ready?  
3. Who adds rows to the plan catalog table?  
4. Risk score: save phone numbers now or later?  
5. Which AI service in production — chatbot platform or direct Gemini/OpenAI?

---

## Summary

We rebuild the homepage, header, and footer in the existing customer website. Buying insurance stays on the same URLs and same flows. The chatbot stays. We add a free risk score page (no backend needed to launch). Dr. Ombrella needs a small new database table and API for real plan data, plus a server route for recommendations. The redesign folder is the visual reference; we reimplement it in Next.js and SCSS, not as a file copy.

---


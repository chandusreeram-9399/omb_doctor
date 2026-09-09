# Homepage Redesign — Product & Delivery Plan

**Goal:** Ship a **completely new homepage** (header, footer, and everything in between) **without changing** how users buy insurance today.

**Repos involved:**

| Repo | Role |
|------|------|
| `C:\Ombrela\customer` | **Production site** — we change homepage + header + footer here |
| `Downloads/Ombrella Homepage Redesign` | **Design reference** — copy UI/UX from here; not a separate production deploy |

---

## 1. What we are delivering

### ✅ In scope (homepage experience only)

- New **header** (logo, nav, mobile menu, phone, Get Quote)
- New **homepage body** (hero, categories, partners, Dr. Ombrella, **Risk Score CTA**, testimonials, FAQ, etc.)
- New **footer** (links, contact, social, NAICOM note)
- **Free Risk Score** — homepage teaser + full page at `/risk-score` (8 tap questions → score + gap report)
- All category buttons route to **existing prequote URLs** (no new buy flow)
- **Chatbot widget stays** (bottom-right AI assistant — unchanged)
- **Dr. Ombrella** on homepage (prompt box → suggested plans → link to existing prequote)

### ❌ Out of scope (must not break)

- Health / motor / life / gadget / travel / education / savings **buy journeys**
- Payment (Paystack / Payaza)
- Login, OTP, sign-up
- Post-purchase, claims processing, policy certificates
- Insurance Service APIs used by buy flows (except new optional homepage endpoints — see Section 5)

---

## 2. How we avoid impacting other teams / flows

| Risk | How we prevent it |
|------|-------------------|
| Breaking buy flows | We **do not edit** `pages/v2/**`, `pages/gadget/**`, `services/**` buy logic |
| Losing chatbot | Keep `ChatWidget` in `website-layout.js` — only adjust CSS/z-index if needed |
| Wrong product links | Use **same URLs** as today’s footer/menu (see Section 4) |
| Login/session breaks | Header keeps same cookie/token logic as current `header.js` |
| SEO / analytics drop | Keep same homepage URL `/`, preserve meta tags & GA/Mixpanel hooks |

**Technical rule:** All changes live under homepage + layout shell. Buy flows stay frozen.

```
┌─────────────────────────────────────────┐
│  CHANGED: Header │ New homepage │ Footer │
├─────────────────────────────────────────┤
│  UNCHANGED: /v2/* prequotes → pay → cert │
│  UNCHANGED: Chatbot widget               │
└─────────────────────────────────────────┘
```

---

## 3. User journey (after launch)

1. User lands on **new homepage**.
2. Clicks **Health Insurance** → goes to **`/v2/health-insurance/prequotes`** (same as today).
3. Rest of journey = **exactly today’s flow** (questions → plans → OTP → pay).
4. Optional: user taps **Check my risk score** → **`/risk-score`** → answers 8 questions → gets score (0–100) + “what you’re missing” → CTAs link to **existing prequotes** (health, life, motor, etc.).
5. Optional: user types in **Dr. Ombrella** → sees **real curated plans from our DB** → clicks through to same prequote/plan pages.
6. User can still open **corner chatbot** for general questions (unchanged).

---

## 4. Route mapping (new homepage → existing flows)

These are the **official links** (from current customer site footer/menu):

| Product | Existing route (do not change) |
|---------|-------------------------------|
| Health Insurance | `/v2/health-insurance/prequotes` |
| Motor Insurance | `/v2/motor/prequotes` |
| Term Life | `/v2/life-insurance/prequotes` |
| Education | `/v2/education/prequotes` |
| Savings | `/v2/savings-insurance/prequotes` |
| Travel | `/travel-insurance/prequotes` |
| Gadget / Mobile | `/gadget` |
| About | `/about-us` |
| Contact / Claims | `/contact-us` |
| Blog | `/blog` |
| Terms | `/terms-and-conditions` |
| Privacy | `/privacy-policies` |
| Login | `/login` |
| Sign up | `/sign-up` |
| **Risk Score (new page)** | `/risk-score` → result CTAs use prequote URLs above |

**Acceptance criteria:** Every CTA on the new homepage uses this table. Zero new checkout paths.

---

## 4b. Risk Score — included in scope

The redesign has a **Risk Score** feature. It **is part of this project** and still **does not change buy flows**.

### What the user sees

| Piece | Where | What it does |
|-------|--------|--------------|
| **Risk Score CTA** | Homepage (blue band + animated gauge) | Teaser: “Check my risk score — free, 60 seconds” → links to `/risk-score` |
| **Risk Score page** | `/risk-score` (new page in customer repo) | 8 tap-only questions → name + phone → score 0–100 + personalised gap report |
| **Result actions** | End of quiz | “Close the gap” buttons → **existing prequote routes** (health, life, motor, etc.) |
| **Download report** | Result screen | HTML report download (client-generated, like redesign prototype) |

### How it works (simple)

```
Homepage CTA → /risk-score
    ↓
User taps through 8 questions (age, dependents, income, assets, etc.)
    ↓
Enters name + Nigerian phone (validation only in v1)
    ↓
Score calculated in the browser (rules in code — same logic as redesign)
    ↓
Shows: score, risk band, missing covers, suggested next steps
    ↓
"Get health cover" / "Get life cover" → /v2/.../prequotes (unchanged)
```

### Backend needed?

| Version | Backend | Notes |
|---------|---------|-------|
| **v1 (Phase 1)** | **None required** | Quiz + score + report run in frontend; no DB write |
| **v2 (optional later)** | Save lead when phone submitted | POST to User Service or Insurance Service for CRM/follow-up — **separate decision**, not blocking homepage launch |

**Important:** Risk Score **does not** replace Dr. Ombrella. They do different jobs:

| | Risk Score | Dr. Ombrella |
|--|------------|--------------|
| Input | Fixed 8 questions | Free-text life situation |
| Output | One score + gap list | 2–3 recommended plans |
| Data | Rules in code (v1) | Plans from **DB catalogue** (Phase 2) |
| Goal | “How protected am I?” | “What should I buy for my situation?” |

### Customer repo files (reference from redesign)

| Redesign file | Port to customer repo |
|---------------|----------------------|
| `src/components/home/RiskScoreCTA.tsx` | `component/home/v2/RiskScoreCTA.js` |
| `src/routes/risk-score.tsx` | `pages/risk-score.js` |
| `src/lib/risk-score.ts` | `utils/risk-score.js` (questions, scoring, report HTML) |

### QA checklist (Risk Score)

- [ ] Homepage CTA opens `/risk-score`
- [ ] All 8 questions work on mobile (tap only)
- [ ] Score displays after phone step
- [ ] Each “close the gap” CTA opens correct prequote URL
- [ ] Download report works
- [ ] Does not break chatbot or header/footer

---

## 5. Dr. Ombrella — real data (not mock)

### What PM should know

Today the redesign prototype uses a **fake JSON file** (`bundle-catalog.ts`). For production you want **real plans stored in your database**, managed by Ombrella (not hard-coded in frontend).

### Recommended approach (3 layers)

```
User prompt
    ↓
AI picks plan IDs (Gemini / existing chatbot stack)
    ↓
Plan details loaded from YOUR DB/API  ← real insurers, names, guide prices
    ↓
"View plans" → existing prequote route
```

### Backend work needed (Insurance Service team)

Add a **small, homepage-only API** — does **not** touch buy-flow APIs.

| Item | Description |
|------|-------------|
| **DB table** e.g. `homepage_plan_catalog` | Curated plans for Dr. Ombrella: `id`, `category`, `insurer`, `name`, `covers`, `monthly_from`, `tags[]`, `prequote_url`, `active`, `sort_order` |
| **GET** `/api/v1/homepage/plan-catalog` | Returns active plans for AI + UI |


**Why DB and not live quote API?**  
Live quotes need a **lead** and user answers. Dr. Ombrella is a **discovery** tool on the homepage — it should show **real product names and starting prices** you’ve approved, then hand off to prequote for exact premium.

### AI layer options

| Option | Tool | Notes |
|--------|------|-------|
| **A (fastest)** | Lovable AI Gateway / Gemini | Same logic as redesign; swap static file for API fetch |
| **B (align with chatbot)** | Existing `CHATBOT_API_BASE` | New “bundle recommender” agent with strict JSON output |
| **C (fallback always on)** | Keyword match on `tags` | Works if AI is down; uses same DB catalogue |

**Safety rules (non-negotiable):**

- AI may **only** return IDs that exist in `homepage_plan_catalog`
- Prices shown come **from DB**, never invented by AI
- Max 2–3 plans, max one plan per category

### Customer repo implementation

- New Next.js API route: `pages/api/homepage/cover-finder.js`
  - Loads catalogue from Insurance Service
  - Calls AI (or keyword fallback)
  - Returns `{ readback, plans[], note }`
- New component: `component/home/dr-ombrella/CoverFinder.js`
- Env vars: `LOVABLE_API_KEY` or reuse chatbot keys — **server-side only**

---

## 6. Delivery phases (for PM timeline)

### Phase 0 — Prep (2–3 days)

- [ ] Design sign-off on homepage, header, footer (from redesign reference)
- [ ] Confirm route mapping table (Section 4)
- [ ] Confirm Dr. Ombrella plan list with business (which insurers/plans appear)
- [ ] Create QA test checklist

**Owners:** PM + Design  
**Risk to others:** None

---

### Phase 1 — New shell without Dr. Ombrella DB (1–2 weeks)

**What ships:** New header, footer, homepage sections, **Risk Score page**; all links to old prequotes; chatbot unchanged.

| Task | Team |
|------|------|
| Rebuild header/footer in customer repo (SCSS + React, match redesign visually) | Frontend |
| Rebuild homepage sections as `component/home/v2/*` (incl. **RiskScoreCTA**) | Frontend |
| Add **`pages/risk-score.js`** + `utils/risk-score.js` (port from redesign) | Frontend |
| Replace `pages/index.js` content; keep `getServerSideProps` for categories if needed | Frontend |
| Keep `WebsiteLayoutComponent` + `ChatWidget` | Frontend |
| Visual QA mobile + desktop + full risk-score quiz | QA |

**Owners:** Frontend  
**Risk to others:** Low — isolated to `/` and layout  
**Rollback:** Restore `pages/index.old.js`

---

### Phase 2 — Dr. Ombrella with real DB data (1–2 weeks, parallel with backend)

**What ships:** Prompt box on homepage; plans from Insurance Service DB; AI recommendations.

| Task | Team |
|------|------|
| DB table + GET catalog API | Backend (Insurance Service) |
| Seed catalogue with approved plans | PM + Ops |
| Next.js API route + CoverFinder component | Frontend |
| AI integration + keyword fallback | Frontend + Backend review |
| Link each plan to correct `prequote_url` | PM verify |

**Owners:** Backend + Frontend  
**Risk to others:** None if new endpoints only  
**Depends on:** Phase 1 homepage live or on same branch behind feature flag

---

### Phase 3 — QA & release (3–5 days)

- [ ] Regression: each product prequote opens correctly from homepage
- [ ] Logged-in user: header account menu still works
- [ ] Chatbot opens and responds on homepage
- [ ] Dr. Ombrella: 10 sample prompts return sensible plans from DB
- [ ] Risk Score: complete quiz on mobile + desktop; prequote links from result page work
- [ ] Analytics events fire (homepage CTA clicks, risk score completions)
- [ ] Deploy QA → staging → production

**Owners:** QA + PM  
**Release:** Customer repo only (`C:\Ombrela\customer`)

---

## 7. Tech stack (customer repo — no new framework)

We **do not** migrate the site to TanStack Start. We **reuse** customer stack:

| Layer | Technology |
|-------|------------|
| Framework | Next.js 12 (Pages router) |
| UI | React 17, MUI, Bootstrap, SCSS |
| HTTP | axios + existing `services/*` |
| Layout | `component/layout/website-layout/` |
| New homepage | New SCSS module + React components under `component/home/v2/` |
| Dr. Ombrella API | `pages/api/homepage/cover-finder.js` |
| Chatbot | Existing `ChatWidget` + `pages/api/webchat/[...path].js` |

**Design reference:** Redesign repo for pixels, copy, section order — reimplemented in SCSS/React to match customer patterns.

---

## 8. Team responsibilities

| Team | Delivers |
|------|----------|
| **Product** | Sign-off, plan catalogue content, route mapping, UAT |
| **Design** | Final header/footer/homepage specs, mobile states |
| **Frontend (customer repo)** | Header, footer, homepage, Dr. Ombrella UI, API route |
| **Backend (Insurance Service)** | `homepage_plan_catalog` table + GET API |
| **QA** | Homepage + smoke test all prequotes + chatbot |
| **DevOps** | QA/staging/prod deploy customer repo |

**Not needed:** Changes to payment service, user service, or v2 buy-flow pages.

---

## 9. Success metrics (PM)

| Metric | Target |
|--------|--------|
| Buy flow completion rate | No drop vs baseline (± noise) |
| Homepage bounce rate | Improve vs old homepage |
| CTA click-through to prequote | Increase |
| Dr. Ombrella usage | Track prompts submitted + clicks to prequote |
| Risk Score completions | Track starts, finishes, downloads, clicks to prequote |
| Chatbot usage | Stable (not reduced) |
| P0 bugs on buy flows | Zero |

---

## 10. Open decisions for PM (please confirm)

1. **Header/footer globally or homepage only?**  
   - *Recommendation:* Apply new header/footer **site-wide** for consistent brand; still only **content** change on buy flows is zero.

2. **Dr. Ombrella v1 — AI required or keyword-only OK?**  
   - *Recommendation:* Ship with keyword fallback day 1; enable AI when key/API stable.

3. **Who maintains plan catalogue in DB?**  
   - *Recommendation:* Ops/Marketing monthly; engineering provides seed script.

4. **Feature flag?**  
   - *Recommendation:* `NEXT_PUBLIC_NEW_HOMEPAGE=true` for QA/staged rollout.

5. **Risk Score — save phone/leads to CRM in v1?**  
   - *Recommendation:* Ship **v1 without backend** (quiz + report only). Add lead capture in v2 if PM wants sales follow-up.

---

## 11. What happens next (engineering)

After PM approves this plan:

1. **Phase 1 coding** in `C:\Ombrela\customer`:
   - `component/home/v2/*` — homepage sections (incl. Risk Score CTA)
   - `pages/risk-score.js` + `utils/risk-score.js` — full quiz + report
   - `component/layout/website-layout/header-v2.js` (or refactor header)
   - `component/layout/website-layout/footer-v2.js`
   - Update `pages/index.js`
   - Preserve chatbot in layout

2. **Phase 2 coding** (when backend API ready):
   - Insurance Service: catalog endpoint
   - Customer: `pages/api/homepage/cover-finder.js` + Dr. Ombrella component

3. **No work** on `pages/v2/**` buy flows.

---

## 12. One-page summary for leadership

> We replace the Ombrella homepage look-and-feel (header, footer, hero, categories, **Risk Score**, Dr. Ombrella). Users get a free **Risk Score check** (8 questions → score + gap report) on a new `/risk-score` page; result buttons still go to **same prequote pages**. Every category click still goes to existing prequotes. The floating chatbot stays. Dr. Ombrella will show **real plans from our database**, with AI choosing the best match. Insurance purchase, payment, and login are **not modified**. Rollout is phased: new UI + Risk Score first, then Dr. Ombrella with DB, then QA and release.

---



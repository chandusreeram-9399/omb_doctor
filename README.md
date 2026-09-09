# Homepage Revamp — Technical Plan of Action

**Goal:** Replace the homepage (header, footer, and page content) on the **customer website** with the new design. All insurance buy flows stay exactly as they are today.

**Customer repo (where we code):** `C:\Ombrela\customer`  
**Design reference (where we copy layout/copy from):** `Ombrella Homepage Redesign` folder

---

## 1. What changes vs what stays the same

### Changes (customer repo only)

| Area | What we do |
|------|------------|
| **Header** | New look + nav; keep login/logout and existing behaviour |
| **Footer** | New look + links; same destination URLs as today |
| **Homepage (`/`)** | New sections (see list below) |
| **New page `/risk-score`** | Free risk quiz (does not exist on customer site today) |
| **Styles** | New SCSS for homepage/header/footer (responsive) |

### Does NOT change

| Area | Why |
|------|-----|
| `pages/v2/**` | Health, motor, life, education, savings prequotes → payment |
| `pages/gadget/**`, `pages/travel-insurance/**` | Other buy flows |
| `services/**` | API calls for insurance, payment, auth |
| Payment (Paystack / Payaza) | Unchanged |
| Login / sign-up / OTP | Unchanged |
| Post-purchase, renewal, claims pages | Unchanged |
| **Chatbot widget** | Stays in layout — do not remove |

---

## 2. Important: this is not “copy-paste”

The redesign folder and the customer repo use **different tech**:

| | Redesign (reference) | Customer (production) |
|--|---------------------|----------------------|
| Framework | TanStack Start | Next.js 12 (Pages router) |
| Styling | Tailwind CSS | Bootstrap + SCSS |
| UI kit | shadcn / Radix | MUI + Bootstrap |
| React | 19 | 17 |

**What we actually do:**

1. **Copy:** layout structure, text, images, section order, behaviour ideas.
2. **Rewrite:** components as `.js` files under `component/home/v2/` using customer patterns.
3. **Rewrite:** styles in SCSS (mobile-first breakpoints) — not Tailwind classes.
4. **Wire:** every button to **existing** `Link href="..."` URLs from current footer/menu.

So it is **fast if we treat redesign as a spec**, but it is **port/adapt**, not drag-and-drop files.

---

## 3. New homepage — sections to build

Order matches the redesign `src/routes/index.tsx`:

| # | Section | Links to buy flow? | Backend needed? |
|---|---------|-------------------|-----------------|
| 1 | Hero carousel | CTA → prequote or scroll | No |
| 2 | Category tiles (health, motor, etc.) | Yes → prequote URLs | No |
| 3 | Partner / insurer logo strip | No | No (static assets) |
| 4 | Dr. Ombrella (cover finder) | Yes → prequote URLs | **Yes** — see Section 6 |
| 5 | Popular plans strip | Yes → prequote URLs | Optional — static or DB |
| 6 | Ombrella Advantage | No | No |
| 7 | Why Ombrella | No | No |
| 8 | Risk Score CTA band | → `/risk-score` | No |
| 9 | Testimonials | No | No |
| 10 | Why buy here (comparison table) | No | No |
| 11 | FAQ accordion | No | No |
| 12 | CTA band | Scroll / prequote | No |
| 13 | Sticky mobile “Find cover” bar | Scroll to Dr. Ombrella | No |

**Responsive:** Each section needs SCSS for mobile, tablet, desktop (customer site already uses `.mobile` / `.laptop` patterns in places — we follow the same or use CSS media queries).

---

## 4. Route mapping — every click goes to existing flows

Use the **same URLs** already in `C:\Ombrela\customer\component\layout\website-layout\footer.js` and `component/home/insurance-menu.js`:

| User action | Route (unchanged) |
|-------------|-------------------|
| Health Insurance | `/v2/health-insurance/prequotes` |
| Motor Insurance | `/v2/motor/prequotes` |
| Term Life | `/v2/life-insurance/prequotes` |
| Education | `/v2/education/prequotes` |
| Savings | `/v2/savings-insurance/prequotes` |
| Travel | `/travel-insurance/prequotes` |
| Gadget | `/gadget` |
| About | `/about-us` |
| Contact | `/contact-us` |
| Blog | `/blog` |
| Login | `/login` |
| Sign up | `/sign-up` |
| Risk Score page | `/risk-score` **(new page)** |

**Rule:** No new checkout or payment URLs. Only `/` and `/risk-score` are new/changed pages.

---

## 5. Files we touch in customer repo

### Create or replace

```
component/home/v2/
  HeroCarousel.js
  HeroCategories.js
  PartnerMarquee.js
  CoverFinder.js          ← Dr. Ombrella (needs API)
  PopularPlans.js
  OmbrellaAdvantage.js
  WhyOmbrella.js
  RiskScoreCTA.js
  Testimonials.js
  WhyBuyHere.js
  FaqSection.js
  CtaBand.js

component/layout/website-layout/
  header.js               ← revamp (keep auth logic)
  footer.js               ← revamp (keep same links)

pages/
  index.js                ← compose v2 sections; keep getServerSideProps if still needed
  risk-score.js           ← new

utils/
  risk-score.js           ← port quiz logic from redesign

styles/
  homepage-v2.scss          ← new styles

pages/api/homepage/
  cover-finder.js           ← Dr. Ombrella server endpoint (prod)
```

### Do not touch

```
pages/v2/**
pages/gadget/**
pages/travel-insurance/**
services/**
component/v2/**          ← buy flow UI
pages/api/webchat/**     ← chatbot proxy (keep)
component/ChatWidget.js
component/layout/website-layout/website-layout.js  ← keep ChatWidget here
```

---

## 6. Backend work — per feature (what each team builds)

### 6.1 Header / footer / static homepage sections

| Backend | Required? |
|---------|-----------|
| Insurance Service | **No** |
| User Service | **No** (header already uses cookies/token for logged-in user — keep existing code) |

Optional: homepage already calls `HOME_PAGE_CATEGORIES` in `pages/index.js` (`GET .../home/blog/categories/home`). We can **keep** or **drop** that call if the new homepage does not use dynamic categories. PM decision.

---

### 6.2 Risk Score (`/risk-score`)

**What it does:** 8 tap questions → name + phone → score 0–100 → gap report → buttons link to prequote URLs. Can download HTML report.

| Layer | Work |
|-------|------|
| **Frontend (customer)** | Port `src/lib/risk-score.ts` → `utils/risk-score.js`; build `pages/risk-score.js` + homepage `RiskScoreCTA.js` |
| **Insurance Service** | **Not required for first release** — scoring runs in browser |
| **User Service** | **Not required for first release** |

**Optional later (if PM wants leads saved):**

| API | Purpose |
|-----|---------|
| New POST on User Service or Insurance Service | Save `{ name, phone, score, answers }` for sales follow-up |

This is **separate** from buy flows. Does not block homepage launch.

---

### 6.3 Dr. Ombrella (homepage cover finder)

**What it does:** User describes their situation → app shows 2–3 recommended plans with guide prices → user clicks through to prequote.

The redesign prototype used a **local JSON file** for plans. For production you want **real plan data in your database**.

#### Backend — Insurance Service (new, homepage-only)

| Item | Detail |
|------|--------|
| **Table** e.g. `homepage_plan_catalog` | Fields: `id`, `category`, `insurer`, `plan_name`, `covers_text`, `monthly_from`, `tags` (JSON array), `prequote_url`, `is_active`, `sort_order` |
| **GET** `/api/v1/homepage/plan-catalog` | Returns all active plans for UI + AI |
| **Admin / seed** | SQL seed or internal tool to load real insurers/plans PM approves |

This API is **read-only** and **does not** replace existing plan search on leads (`/lead/{id}/health/plans`, etc.).

#### Backend — Customer site (Next.js API route)

| Item | Detail |
|------|--------|
| **`pages/api/homepage/cover-finder.js`** | Server-side only: (1) fetch catalog from Insurance Service, (2) run recommender, (3) return JSON to frontend |
| **Env vars (production)** | e.g. `AI_API_KEY`, `AI_API_URL` — stored in server env, never in browser |

#### AI recommender — production options (pick one)

| Option | How it works | Backend owner |
|--------|--------------|---------------|
| **A. Extend existing chatbot API** | Add a “bundle recommend” endpoint on `CHATBOT_API_BASE`; strict JSON: plan IDs only | Chatbot / platform team |
| **B. Direct LLM call from Next API route** | Gemini or OpenAI from `cover-finder.js`; prompt includes catalog IDs only | Customer frontend + DevOps for keys |
| **C. No AI (keyword only)** | Match user text to `tags` in DB — same as redesign fallback | Insurance Service catalog only |

**Production safety rules:**

- AI returns **only IDs** that exist in `homepage_plan_catalog`
- **Prices and plan names** always come from **DB response**, never from AI text
- Max 3 plans, one per category
- If AI fails → keyword fallback on same DB data

#### Frontend — customer repo

| Item | Detail |
|------|--------|
| `CoverFinder.js` | Calls `/api/homepage/cover-finder`; renders results; links use `prequote_url` from DB |

---

### 6.4 Chatbot (corner widget)

| Backend | Required? |
|---------|-----------|
| **No new work** | Already uses `CHATBOT_API_BASE` + `pages/api/webchat/[...path].js` |

Only check: new homepage CSS does not hide the widget (z-index).

---

### 6.5 Popular plans strip (optional backend)

| Approach | Backend |
|----------|---------|
| **Static in frontend** | None — hard-code cards like redesign |
| **From same catalog as Dr. Ombrella** | Reuse `GET /api/v1/homepage/plan-catalog` with a `featured` flag |

---

## 7. Step-by-step execution order

Do these in order. Each step can be tested without breaking buy flows.

### Step 1 — Backup and branch

- Copy `pages/index.js` → `pages/index.backup.js`
- Work on a git branch in `C:\Ombrela\customer`

### Step 2 — Header and footer revamp

- Update `header.js` / `footer.js` (visual + responsive SCSS)
- **Keep:** token/cookie login logic, drawer, phone number, same nav links
- **Keep:** `WebsiteLayoutComponent` wrapping pages with `ChatWidget`
- Test: login, logout, open any prequote from footer

### Step 3 — Homepage static sections (no Dr. Ombrella yet)

- Add `component/home/v2/*` for hero, categories, partners, advantage, FAQ, etc.
- Replace body of `pages/index.js` to render these components
- Point all category CTAs to Section 4 URLs
- Add responsive SCSS; test mobile + desktop
- Test: every tile opens correct prequote; chatbot still visible

### Step 4 — Risk Score

- Add `utils/risk-score.js`, `pages/risk-score.js`, `RiskScoreCTA.js` on homepage
- Result page CTAs → prequote URLs from Section 4
- Test full quiz on phone

### Step 5 — Backend: plan catalog (Insurance Service)

- Create table + GET API + seed data (PM provides plan list)
- Test API in Postman

### Step 6 — Dr. Ombrella

- Add `pages/api/homepage/cover-finder.js`
- Connect to catalog API + AI or keyword recommender
- Add `CoverFinder.js` on homepage
- Test with/without AI; verify links go to prequote URLs

### Step 7 — QA smoke test

- [ ] Homepage responsive (320px, 768px, 1280px)
- [ ] Each product link → correct prequote page loads
- [ ] Complete one health prequote flow end-to-end (unchanged behaviour)
- [ ] Chatbot opens and replies
- [ ] Risk score completes; prequote links work
- [ ] Dr. Ombrella returns plans from DB
- [ ] Logged-in user header still works

### Step 8 — Deploy customer repo to QA → production

Deploy **only** `C:\Ombrela\customer`. Redesign folder is not deployed separately.

---

## 8. What “responsive” means in practice

- Mobile-first SCSS breakpoints (or reuse customer `.mobile` / `.laptop` split where team already does that)
- Touch targets ≥ 44px on CTAs
- Hero and category grid: 1 col mobile → 2–3 tablet → 6 desktop (match redesign behaviour)
- Sticky bottom bar on mobile (Find cover)
- Header: hamburger drawer on small screens (keep existing drawer pattern or match new design)

Reference breakpoints from redesign use Tailwind (`sm:`, `lg:`) — **translate** to SCSS, do not add Tailwind to customer repo unless team decides otherwise.

---

## 9. Environment variables (production customer site)

| Variable | Used for | Already exists? |
|----------|----------|-----------------|
| `API_BASE_URL_INSURANCE_SERVICE` | Catalog API, existing flows | Yes |
| `API_BASE_URL_USER_SERVICE` | Auth | Yes |
| `CHATBOT_API_BASE`, `CHATBOT_API_KEY` | Chatbot | Yes |
| `AI_API_KEY` / `AI_API_URL` (name TBD) | Dr. Ombrella recommender | **New** — server-side only |

No third-party prototype keys in production. Use your own AI provider or chatbot platform credentials.

---

## 10. Summary for PM / anyone reading this

1. We **revamp homepage, header, and footer** in the **customer repo**.
2. We **port** the new UI from the redesign folder (adapt styles, not literal copy-paste).
3. All **buy insurance flows stay untouched** — homepage only links to them.
4. **Chatbot stays** on every page.
5. **Risk Score** = new page, mostly frontend; no backend required to launch.
6. **Dr. Ombrella** = needs **new DB table + API on Insurance Service** + **Next.js API route** + production AI or keyword matching.
7. Work is **frontend-heavy** for layout; **backend is small and isolated** (catalog + optional lead capture).

---


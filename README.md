# Homepage-only plan (simple words)

**Your goal:** Replace the **old homepage** with the **new homepage design**.  
**Do NOT change:** insurance buy flows, payment, login, claims, or the floating chatbot widget.

---

## What we are doing vs not doing

| ✅ We WILL do | ❌ We will NOT do |
|--------------|-------------------|
| New homepage look (hero, categories, Dr. Ombrella box, etc.) | Rewrite health / motor / life buy journeys |
| Link buttons to **existing** customer routes | Change APIs or payment |
| Keep the **chatbot widget** on every page | Remove or replace the chatbot |
| Dr. Ombrella AI on homepage (mock/guide data for now) | Wire real insurer prices into Dr. Ombrella yet |

---

## The two codebases (remember this)

| Folder | What it is |
|--------|------------|
| `Downloads/Ombrella Homepage Redesign` | **New homepage UI** (this project) |
| `C:\Ombrela\customer` | **Live site** — old homepage + all buy flows + chatbot |

**End result:** User opens the site → sees **new homepage** → clicks "Health" or "Get started" → goes to **same old flow** as today.

---

## Step-by-step plan

### Step 1 — Run both projects locally (so you can test)

1. Install **Node 22 LTS**.
2. Run **customer repo** (old site):
   ```powershell
   cd C:\Ombrela\customer
   yarn install
   yarn dev
   ```
   Opens at `http://localhost:3000` — note the URLs for health, motor, life, etc.

3. Run **homepage redesign** (new site):
   ```powershell
   cd "C:\Users\LENOVO\Downloads\Ombrella Homepage Redesign"
   npm install
   npm run dev
   ```
   Opens at a different port (e.g. `http://localhost:5173`).

For now you use two tabs to compare. Later we merge into one site.

---

### Step 2 — List every link on the new homepage

Go through the new homepage and write down each button/link:

| New homepage action | Should go to (old customer route) |
|--------------------|-----------------------------------|
| Health category | `/v2/health-insurance/prequotes` or `/health-insurance/...` (match what old site uses) |
| Motor | `/v2/motor/prequotes` |
| Life | `/v2/life-insurance/prequotes` |
| Gadget | `/gadget/...` |
| Travel | `/travel-insurance/...` |
| Education | (check old `insurance-menu.js` links) |
| Savings | `/savings-insurance/...` |
| Claims | `/contact-us` or claims page on old site |
| About / Contact | `/about-us`, `/contact-us` |

**Where to find old links:**  
`C:\Ombrela\customer\component\home\insurance-menu.js` and `pages/index.js`.

---

### Step 3 — Put the new homepage **into** the customer repo (recommended way)

Because you must **keep chatbot + old flows**, the easiest path is:

**Copy the new homepage into `C:\Ombrela\customer`**, not the other way around.

Why?

- Chatbot already lives in `component/layout/website-layout/website-layout.js` → stays as-is.
- All buy flows stay under `pages/v2/`, `pages/gadget/`, etc. → untouched.
- Only `pages/index.js` (and maybe header/footer styling) changes.

**Sub-steps:**

1. **Backup** old homepage: copy `pages/index.js` → `pages/index.old.js`.
2. **Port new homepage** into customer repo:
   - Either rebuild the new sections as React components under `component/home/new/` using the redesign as a visual reference,
   - Or (if team agrees) add Tailwind to customer repo and import adapted components.
3. **Replace** `pages/index.js` content with the new homepage layout.
4. **Keep** `<WebsiteLayoutComponent>` wrapper so **Header + Footer + ChatWidget** still wrap the page.
5. **Point every CTA** to existing `Link href="/v2/..."` paths — same as old menu.

> **Important:** We are **not** moving buy flows into the redesign repo. We are **only** swapping the homepage inside the customer repo.

---

### Step 4 — Keep the chatbot widget

The chatbot is already in the customer layout:

```
C:\Ombrela\customer\component\layout\website-layout\website-layout.js
  → <ChatWidget ... />
```

**Rule:** Do not remove this file or component.  
When you change the homepage, the layout still wraps all pages → chatbot stays on every page including the new homepage.

No extra work unless the new homepage CSS hides the widget (fix z-index if needed).

---

### Step 5 — Dr. Ombrella AI on the homepage (without live data)

There are **two different AI things** — do not mix them up:

| Feature | What it is | Live data? |
|---------|------------|------------|
| **Floating chatbot** (bottom corner) | General Q&A, already on old site | Uses `CHATBOT_API_BASE` — keep as-is |
| **Dr. Ombrella box** (homepage prompt) | "I had a baby…" → suggests 2–3 plans | **No live API yet** — uses a **fixed plan list** in code |

#### How Dr. Ombrella works today (in the redesign code)

1. User types a life situation (or taps a sample prompt).
2. **Server** sends text to an **AI model** (Lovable AI Gateway → Gemini).
3. AI picks **2–3 plan IDs** from a **local catalogue** file (`bundle-catalog.ts`) — not from your insurance API.
4. App shows plan names, guide prices, and "why this plan" text.
5. If AI is off or fails → **keyword matching** on the same catalogue (still no live data).

So: **smart suggestions from a fixed menu**, not real quotes from insurers.

#### What you need for Dr. Ombrella to "respond"

**Option A — Quick (good for homepage-only launch)**  
Use the redesign logic as-is inside customer repo:

- Copy `bundle-catalog.ts` + cover-finder server logic (or a Next.js API route version).
- Set env: `LOVABLE_API_KEY` (from Lovable workspace) **or** skip key and use keyword fallback only.
- Buttons like "See plans" link to **old** prequote routes — no real bundle checkout.

**Option B — Same UI, your own AI later**  
Keep the same UI; swap the backend call to:

- Your existing **chatbot API** (`CHATBOT_API_BASE`), with a prompt that returns JSON plan IDs, **or**
- OpenAI / Gemini API directly with the same rules: *only pick from catalogue, never invent prices*.

**Option C — No AI at all (simplest)**  
Show the prompt box but only use **keyword matching** on `bundle-catalog.ts` — still feels helpful, zero API cost.

#### Recommended for your goal (homepage only, no live data)

Use **Option A or C**:

- **Catalogue** = static list in code (already in redesign).
- **AI** = optional (`LOVABLE_API_KEY`); fallback always works.
- **"Get this bundle"** → link to existing health/motor prequote pages, not a new checkout.

---

### Step 6 — Map new homepage buttons to old routes (checklist)

Before go-live, click every button on the new homepage:

- [ ] Each category tile opens the **same URL** as the old homepage menu.
- [ ] "Get a quote" / sticky mobile button scrolls or links correctly.
- [ ] Dr. Ombrella result → "See plans" goes to old health (or relevant) flow.
- [ ] Header links (Claims, Support, phone) unchanged or match old site.
- [ ] Chatbot still opens bottom-right on homepage.
- [ ] Login / sign-up still go to old `/login`, `/sign-up`.

---

### Step 7 — Test on QA

1. Deploy customer repo to **QA** branch (your team already uses `env/.env.qa`).
2. Test homepage on phone + desktop.
3. Test one full path: homepage → Health tile → old prequote flow → still works.
4. Test chatbot still answers.
5. Test Dr. Ombrella prompts (with and without AI key).

---

### Step 8 — Go live

1. Merge homepage change to production branch.
2. Deploy customer repo only (no separate redesign deploy needed if you merged into customer).
3. Monitor: homepage loads, old flows work, chatbot works.

---

## Simple architecture picture

```
User visits ombrella.com
        │
        ▼
┌─────────────────────────────┐
│  NEW HOMEPAGE (only this     │
│  page changed)               │
│  • Hero, categories          │
│  • Dr. Ombrella (mock plans) │
└─────────────┬───────────────┘
              │ clicks Health / Motor / etc.
              ▼
┌─────────────────────────────┐
│  OLD BUY FLOWS (unchanged)   │
│  /v2/health-insurance/...    │
│  /v2/motor/...               │
│  payment, OTP, etc.          │
└─────────────────────────────┘

Floating chatbot (unchanged) ──► CHATBOT_API_BASE
Dr. Ombrella box (new)       ──► AI + static catalogue (no live insurer API)
```

---

## Order of work (do in this sequence)

1. ✅ Run redesign locally — see the new homepage.
2. ✅ Run customer repo locally — note all old URLs.
3. 📝 Write link mapping table (Step 2).
4. 🔧 Port new homepage UI into `C:\Ombrela\customer\pages\index.js` (+ components).
5. 🔗 Wire all links to old routes only.
6. 🤖 Add Dr. Ombrella box (static catalogue + optional AI key).
7. ✅ Confirm chatbot still in layout.
8. 🧪 QA test homepage + one buy flow + chatbot.
9. 🚀 Deploy.

---

## FAQ in simple words

**Do we need the redesign repo in production?**  
Not necessarily. You can copy the homepage **into** the customer repo and deploy only customer.

**Will Dr. Ombrella show real prices?**  
Not until you connect to the insurance API later. For now it shows **guide prices** from the catalogue file — fine for homepage launch.

**Will we break insurance?**  
No — if you only change `index.js` / homepage components and use old `href`s.

**Can we use the old chatbot AND Dr. Ombrella?**  
Yes. Chatbot = corner widget. Dr. Ombrella = homepage search box. Different jobs.

---

## Files to touch (customer repo)

| File | Action |
|------|--------|
| `pages/index.js` | Replace with new homepage |
| `component/home/*` | Add new homepage sections |
| `component/layout/website-layout/website-layout.js` | **Do not remove ChatWidget** |
| `pages/v2/**`, `services/**` | **Do not touch** |

## Files to reference (redesign repo)

| File | Use for |
|------|---------|
| `src/routes/index.tsx` | Section order on homepage |
| `src/components/home/*` | UI/copy/layout reference |
| `src/lib/bundle-catalog.ts` | Dr. Ombrella plan list |
| `src/lib/cover-finder.server.ts` | Dr. Ombrella AI logic |

---

*Scope locked: homepage only. Insurance flows stay in `C:\Ombrela\customer`.*

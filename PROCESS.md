# Orente Grills — Process & Evidence

**Project:** Orente Grills — a functional campus food‑ordering web app
**Stack:** Next.js 15 (App Router) · React 19 · TypeScript · GSAP · Lenis
**Repo:** https://github.com/Ferousco-dev/oriente
**Tooling:** Built with Claude Code (agentic AI pair‑programming in the IDE)

This document is the evidence of process: the prompts used, the iterations,
the AI workflow, and how each step was verified. Attach the screenshots and the
SkillMD picture alongside this file.

---

## 1. The prompts I used (in order)

The project was built through an iterative conversation. Each prompt below moved
the app forward; the AI explored the codebase, planned, built, and **verified
(type‑check + production build + runtime check) before moving on.**

### Prompt 1 — Make the whole app functional (mock, no backend)
> "the order button… we are not connected to any backend or any database so we
> are to mock it. When someone clicks order it should take the user to a page to
> enter their email and name to verify, so they can see their record. In our menu
> add more UI — if I hover on a food it should show a preview, the price, and an
> order button. We want to make the whole app functional. And after order, see the
> image above (TikTok clip) — I love the animation. We've started but can't continue."

**What the AI did**
- Explored the project; **found the app didn't compile** — `layout.tsx` imported
  `StoreProvider` and `CartDrawer` but never rendered the provider, and
  `CartDrawer` didn't exist. That was the real blocker.
- Built the missing pieces:
  - `src/lib/menu.ts` — menu data with real items + prices (₦)
  - `src/components/CartDrawer.tsx` — slide‑out cart
  - Wired `Navbar` order button + live cart count to the store
  - Hover preview (price + "Add to bag") on every menu card (`OurMenu.tsx`)
  - `/order` page: **name+email verify → review → place order**
  - `src/components/OrderSuccess.tsx` — **GSAP order‑placed animation**
    (button fills → morphs to a badge → checkmark draws → confetti), inspired by
    the TikTok "order button morph" clip
  - `/account` page — look up past orders by name+email
- **Verified:** `tsc --noEmit` clean → `next build` (caught & fixed the layout
  provider bug) → dev server, all routes 200, no runtime errors.

### Prompt 2 — Mobile fix (no way back)
> "on mobile no button to go back" (with a screenshot of the empty cart on an
> iPhone 15 Pro Max)

**What the AI did**
- Diagnosed: the empty cart drawer was a dead end on mobile (only a greyed‑out
  Checkout). Added a **"Browse the menu"** button and replaced the dead disabled
  Checkout with a **"Continue shopping"** button — both close the drawer.
- Re‑ran type‑check + build to confirm.

### Prompt 3 — Research what wins, then build it
> "search anything else to win this competition — search"

**What the AI did**
- Ran web research (Awwwards 2026 restaurant trends, food‑ordering conversion
  studies, hackathon judging criteria) and used **AskUserQuestion** to confirm
  scope + context (school/coursework project).
- Built the 4 highest‑leverage features the research pointed to:
  1. **Live order tracking timeline** (`OrderTracker.tsx` + `order-status.ts`) —
     Placed → Preparing → Out for delivery → Delivered, **auto‑advancing on a
     timer**, shown on the success screen and the latest order in /account
  2. **One‑tap Reorder** on past orders
  3. **Menu category filter tabs + dietary/promo tags** (Popular, Spicy, Healthy…)
  4. **Checkout upsell** ("Make it a feast" add‑ons)
- **Verified:** unit‑checked the status time‑math at every boundary, type‑check
  clean, build passed all 6 routes, dev runtime check clean.

### Prompt 4 — Ship it
> "push to https://github.com/Ferousco-dev/oriente.git — don't add claude as
> co‑author"
>
> followed by: "wait the images — compress them, it's too big"

**What the AI did**
- `git init`, configured remote, staged. **Caught 15 MB of unused source PNGs**
  (`img/1–5.png`) + a redundant root `logo.png` — the app only uses
  `public/images/`. Used **AskUserQuestion**; on the user's choice, added them to
  `.gitignore` (kept on disk, out of the repo) so the repo stays ~3 MB.
- Committed **with no Claude co‑author** (verified trailers empty) and pushed to
  `main` → commit `a70481b`.

---

## 2. AI workflow / process discipline shown

- **Explore → Plan → Build → Verify** on every prompt — never edited blind.
- **Todo tracking** kept multi‑step work visible and ordered.
- **Verification before "done":** TypeScript `--noEmit`, Next.js production
  `build`, and a live dev‑server runtime check (all routes → 200) on each round.
- **Caught real bugs the user couldn't see:** the app wasn't compiling (missing
  provider), the empty cart was a mobile dead end, and 15 MB of dead images were
  about to be committed.
- **Research‑driven feature selection** (web search) instead of guessing what
  "wins," then confirmed scope with the user before building.
- **Clean git hygiene:** sensible `.gitignore`, no `node_modules`/build artifacts,
  honored the "no co‑author" instruction.

---

## 3. Files produced this session

**New components**
- `src/components/CartDrawer.tsx` (+ `cart-drawer.css`)
- `src/components/OrderSuccess.tsx` (+ `order-success.css`) — GSAP confirmation
- `src/components/OrderTracker.tsx` (+ `order-tracker.css`) — live status timeline

**New pages (App Router)**
- `src/app/order/page.tsx` (+ `order.css`) — verify → review → place
- `src/app/account/page.tsx` (+ `account.css`) — records + reorder

**New logic / data**
- `src/lib/store.tsx` — cart/account/orders state, localStorage‑persisted
- `src/lib/menu.ts` — menu items, prices, tags, upsell add‑ons
- `src/lib/order-status.ts` — time‑derived order status (mock, no backend)

**Modified**
- `src/app/layout.tsx` — wrapped app in `StoreProvider` + mounted `CartDrawer`
  (the compile fix)
- `src/components/Navbar.tsx`, `src/components/OurMenu.tsx` — wired to the store,
  filters, tags, hover preview

---

## 4. End‑to‑end demo path (what the screenshots show)

Hover a menu item (price + tag) → filter by category → **Add to bag** → cart
drawer → **Checkout** → enter name + email (verify) → **see the upsell** →
**Place order** → **GSAP animation + live tracker** → **My Orders** → watch the
status reach Delivered → **Reorder** in one tap.

---

## 5. How to run

```bash
npm install
npm run dev     # http://localhost:3000
npm run build   # production build (all routes prerender clean)
```

_Everything is mocked — cart, account, and orders persist in `localStorage`
(`orente:v1`). No backend or database required._

# Frontend Options (Web App) — Comparison for This Project

> **Status:** Planning Doc (high-level)  
> **Audience:** Product + engineering  
> **Goal:** Compare startup/industry-standard **dashboard-first** frontend approaches, aligned to a NestJS REST backend.

---

## 1. Your Key Requirements (from scope)

- Large authenticated **dashboard** surface (CRM, ERP, HR, Projects, Fleet, Reporting, Admin).
- Public/SEO pages exist (marketing, link-in-bio, public KB, storefront), but **are not the current focus**.
- Strong need for **forms**, tables, filters, and complex state.
- Real-time updates (notifications, tickets, conversations, fleet tracking).
- Solo founder + heavy AI assistance + TypeScript stack; maintainability matters.
- Backend is a modular monolith: **NestJS + REST**, durable events/jobs via **BullMQ + Redis**.

---

## 2. Main Options (Web App)

### Option A — Dashboard SPA (client-rendered) + REST API (recommended default)

**Pros**
- Simplest mental model for a **dashboard-heavy** product: classic client routing + API calls.
- Fast iteration as a solo dev; fewer SSR/server-client edge cases.
- Easy to keep boundaries clear: frontend = UI; backend = business logic.

**Cons**
- SEO/public pages are not covered by this app (not a problem if you postpone them).
- Need to design client-side caching, loading states, and error handling well.

**Fit**
- Best fit when the product is primarily an authenticated web app (your current focus).

---

### Option B — Single “full-stack React framework” app (SSR/SSG + routing + API)

This is the “one app does everything” approach.

**Pros**
- Can serve public/SEO routes and app routes in one project.
- Strong ecosystem and hiring pool.
- Useful when you later prioritize public surfaces.

**Cons**
- Higher complexity for dashboards: server/client boundaries, hydration, caching semantics.
- Some frameworks can feel “slow” in dev or introduce navigation waterfalls if misused (especially with dynamic routes and heavy layouts).

**Fit**
- Good if you want one codebase for everything and accept framework complexity.

---

### Option C — “Client-first meta-framework” (Vite-based) with optional server features

Examples in the ecosystem include Vite-based frameworks that emphasize client-first routing plus server/SSR as an option.

**Pros**
- Keeps the dashboard experience close to SPA while offering server features when needed.
- Often faster dev server/startup due to Vite-first toolchains.

**Cons**
- Newer ecosystem compared to the most established full-stack frameworks.
- You still need conventions to avoid mixing patterns everywhere.

**Fit**
- Strong option if you want “SPA first” now and optional SSR later without committing to heavier patterns.

---

## 3. What I’d Optimize For (given your scope)

Because your product is **dashboard-heavy**, optimize for:

- **Developer speed** on CRUD-heavy UI (forms, tables, filters, modals)
- **Predictable state management**
- **Fast iteration** (avoid fighting SSR boundaries)
- **Clear separation** of public SEO pages vs authenticated app

That usually points to **Option A** (dashboard SPA) or **Option C** (client-first meta-framework).

---

## 4. Concrete Technology Options (with your backend)

## Recommendation (current best option)

For your current focus (shipping an authenticated web app MVP fast as a solo developer), the best default choice is:

- **React + Vite (SPA dashboard)**

It keeps the dashboard architecture simple, pairs cleanly with your **NestJS REST** backend, and avoids SSR/SSG complexity until you actually need public/SEO pages.

### 4.1 Recommended default for your current focus (web app MVP)

- **React + Vite** (SPA dashboard)
  - Pair with your **NestJS REST** API.
  - Use a typed API client (OpenAPI-generated or hand-written) to keep TypeScript end-to-end.
  - Add a node-based workflow UI later using a MIT-licensed UI library (e.g. React Flow), while your backend runs automations.

### 4.2 “Unified toolchain” option: Vite+

If what you heard is “Vite+”: it’s a unified, open-source toolchain that bundles dev server, build, test, lint/format, and task running behind one CLI/config. It’s MIT-licensed and very new (alpha as of March 2026), so:

- **Pros**: potentially simpler toolchain and faster DX.
- **Cons**: early-stage; expect churn; you may hit rough edges.

**Guidance:** Great to watch and possibly adopt later; for a time-boxed MVP, plain **Vite** is the safer bet.

### 4.3 Next.js (when it fits, and why it can feel slow)

Next.js is strong when you need **SSR/SSG** for public pages. For dashboards, it can be great too, but it’s easier to accidentally introduce performance issues:

- Dynamic routes and heavy layouts can create **navigation waterfalls**
- Overusing client components can ship too much JS
- Prefetching behavior can be surprising for dynamic segments

**Guidance:** If you keep Next.js, treat authenticated routes as “app-like” (client-first), and be disciplined with data fetching and layout boundaries.

### 4.4 Other mainstream choices (keep generic for now)

- SPA frameworks/builders (client-rendered dashboards)
- Full-stack React frameworks (SSR/SSG + app routes)
- Vite-based client-first frameworks (SPA-first, optional server features)

Pick based on MVP speed and your comfort with SSR complexity.

---

## 5. Decision Criteria Checklist

- **Dashboard complexity**: is most time spent in authenticated UI? (Yes)
- **SEO importance now**: do you need SEO for MVP? (No — not your focus)
- **Team size**: will you have a large frontend team soon? (No; solo early)
- **Operational simplicity**: one deploy artifact vs multiple (trade-off)
- **Type safety**: can you keep types consistent between frontend and NestJS API?

---

## 6. Open Decisions (to choose later)

- Whether/when to build public pages (marketing, link-in-bio, public KB, storefront)
- Whether to keep one web app for everything or split dashboard/public later
- How to share UI components and API client code if split
- Auth strategy (session vs JWT) and how it works across multiple web frontends
- Real-time strategy on the web client (WebSocket handling, reconnection, channel subscriptions)


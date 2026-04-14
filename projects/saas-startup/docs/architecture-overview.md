# Architecture Overview

> **Last Updated:** March 8, 2026  
> **Architecture review (March 2026):** Durable events via BullMQ; Prisma+RLS spike priority; ERP phasing; file storage, notifications, caching; AI agent design; search and DB scaling notes.  
> **Status:** Living Document  
> **Audience:** Developers, future team members, architectural reference

---

## 1. System Architecture Pattern

**Decision: Modular Monolith with Dedicated Background Workers**

This platform is a **complete business solution** (CRM + full ERP + HR + projects + time tracking & timesheets + contracts & e-signature + content & marketing + booking/scheduling + e-commerce storefront + cross-module dashboards + help desk/support + internal knowledge base + link-in-bio + fleet management + feature-request feedback loop). A **unified AI agent** sits on top of all modules, interacting with them and making decisions from user prompts and business context. It uses a modular monolith architecture for the main API, with separate worker processes for AI orchestration and background jobs. This pattern fits our constraints because: (1) a solo founder cannot effectively operate distributed microservices, (2) NestJS's module system naturally supports bounded contexts with clear dependency rules, (3) AI and automation workloads are inherently async and must not block API responsiveness, and (4) the monolith deploys as a single unit, simplifying CI/CD and debugging.

**Evolution path:** Module boundaries are designed for future extraction. If a specific domain (e.g., AI-Agents or Integrations) becomes a performance bottleneck or requires independent scaling, it can be extracted to a standalone service. The internal event-driven communication pattern ensures this extraction requires minimal refactoring. Likely first extraction candidates: AI workers (compute-intensive) or Integration adapters (rate-limit isolation).

---

## 2. Multi-Tenancy Approach

**Decision: Row-Level Tenant Isolation with PostgreSQL Row-Level Security (RLS)**

All tenant data lives in a single database schema with `tenant_id` columns enforced via PostgreSQL RLS policies. This approach is chosen because: (1) Prisma ORM has limited multi-schema support, making schema-per-tenant operationally complex, (2) row-level isolation with RLS provides strong security guarantees without connection pool explosion, (3) migrations apply once globally rather than per-tenant, and (4) this scales efficiently to 10,000+ tenants. **Each tenant represents one customer of the platform** (one business using the product).

**Organisation structure (multi-entity and agencies):** Two patterns are supported. **(1) Multi-entity** — A tenant (one customer of the platform) can have multiple **entities** (e.g. departments, branches, legal entities) for accounting, reporting, inventory per location, and operations. One business, many departments/branches. Entity is a first-class concept in ERP (ledgers, P&L, consolidation) and can be used in other modules where relevant. **(2) Agencies** — An agency account can manage **multiple tenants** (multiple client businesses). The data model and RBAC support this via an optional parent-tenant (or organization) relationship and roles that allow switching context between child tenants. One account, many businesses. Data isolation remains per-tenant; agency users access one tenant at a time within their allowed set.

**Agency data access and permissions (for agency accounts):** Different agencies need different levels of access to their customers' data: some may be allowed to see and manage **all** data for the businesses they serve; others only **limited** data (e.g. contacts and deals but not billing or sensitive HR). The **business (tenant)** grants these permissions to the agency in a **simple, user-friendly way** (e.g. invite agency → choose access level or scopes per agency). Permissions are enforced in the API and RLS so that agency users never see more than what each business has explicitly granted. Details → `docs/data-model.md`, `docs/security.md`.

**Multi-tenant risk controls:** Tenant context is propagated on every API request (middleware) and in every job payload for workers, so RLS and application logic always run in a tenant scope. Cache keys (e.g. Redis) are tenant-namespaced to avoid cross-tenant leakage. Queues use per-tenant isolation or rate limits where needed to prevent one tenant from starving others. Details → `docs/security.md`.

**GDPR/Compliance:** Data residency is handled at the database cluster level—EU tenants connect to an EU-hosted PostgreSQL instance, US tenants to a US instance. Tenant assignment to region is determined at signup. Right-to-deletion is implemented via cascading soft deletes with a background hard-delete job after retention period; audit logs are in scope (retention and anonymization or deletion per policy). Data portability is a dedicated export API per tenant. Subprocessors (AI providers, email, ads) and data transfer controls are documented in `docs/security.md`.

---

## 3. Domain Boundaries

**Product scope:** A full list of features and capabilities is in → `docs/product-scope.md`.

### Bounded Contexts

| Domain | Responsibility |
|--------|----------------|
| **Core** | Tenant management, user authentication, RBAC, audit logging, platform settings, white-label (tenant branding, custom domain) |
| **CRM** | Contacts, companies, deals, pipeline stages, activities, notes, relationship tracking; customer segmentation (segments, tags, lifecycle stages, saved views); sales features (quotas, targets, pipeline forecasting, commission management, leaderboards, territories) |
| **Contracts & e-signature** | Cross-domain: contract templates, document generation (merge from CRM/ERP/HR), versions, e-signature; sales (quotes, deals, orders) and HR (employment, NDAs); status and renewal reminders |
| **ERP** | Full ERP: general ledger, AP/AR, fixed assets, expense management, budgeting and planning (budget vs actuals), bank reconciliation; inventory & warehouses; purchasing & POs; vendor portal (see Portals); sales orders & fulfillment; e-commerce storefront; subscription billing; reporting uses platform Reporting |
| **HR** | Employee profiles, leave/absence management, payroll basics, job postings (Indeed, LinkedIn), candidate tracking, interview scheduling (see Scheduling), training & learning (LMS: courses, assignments, completion, compliance) |
| **Projects** | Projects, tasks/subtasks, milestones, resource/capacity planning (link to HR), time tracking, timesheets (billing and payroll), link to CRM and ERP |
| **Automations** | Workflow engine, triggers (event/schedule/webhook), actions, execution logs, user-defined automation rules |
| **AI-Agents** | **Unified AI agent** on top of all modules: interacts with CRM, ERP, HR, Projects, Fleet, etc.; makes decisions from user prompts and business context; content generation, lead scoring, campaign optimization; thin gateway to OpenAI/Anthropic |
| **Content & marketing** | Content management (pages, posts, assets; content planning and calendar; event management), form builder (incl. surveys as secondary use), email marketing (campaigns, sequences, segments, A/B), campaigns, landing pages, booking (see Scheduling), SEO, marketing analytics |
| **Link in bio** | Content-creator feature: shareable profile URL, multiple links, custom branding, optional click analytics and CRM/ERP links |
| **Fleet management** | Vehicle register, drivers, location tracking, maintenance, fuel/mileage and fuel reports, issues/reports, orders management (e.g. deliveries/jobs per vehicle), routes/trips, fleet reporting |
| **Integrations** | Adapters for external services: Stripe (payments), Meta/Google/LinkedIn Ads, email providers, SMS, calendar sync, conversational channels (e.g. WhatsApp Business) |
| **Help desk / support** | Support tickets, SLA/priority, public knowledge base (help centre), surveys and feedback (NPS, CSAT; form builder for collection), link to CRM |
| **Content & knowledge** | Public knowledge base (help centre for support); internal knowledge base & documents (wiki, SOPs); versioning and access; AI assistant can use internal content for context-aware help |
| **Reporting & dashboards** | Single reporting capability: module-level and cross-module dashboards, saved/scheduled reports, export for BI |
| **Scheduling** | Shared availability and slot management; powers booking (Content & marketing) and interview scheduling (HR); can use Shifts for availability |
| **Shifts** | Employee/workforce scheduling: shift definitions, rosters, assignments; used by HR (payroll, leave), Fleet (driver roster), Projects (capacity), Scheduling (availability) |
| **Portals** | Customer portal (CRM + e-commerce self-service), vendor portal (supplier self-service); same auth model, different roles and data |
| **Organisation structure** | Multi-entity (one tenant, many entities: departments, branches, legal entities); agencies (one account, many tenants); data model and RBAC — see Section 2 |
| **Conversational channels** | Messaging (e.g. WhatsApp) across CRM (sales), support, marketing, and other domains; unified inbox; conversations linked to contacts, deals, tickets |
| **Feature requests & feedback** | User feature requests per module; shared list with upvotes/comments; internal approval; AI-assisted build/test; targeted rollout (requesters, similar users, or whole base) |
| **File storage** | Cross-cutting: tenant-isolated object storage (S3-compatible) for documents, assets, receipts, uploads; signed URLs; optional virus scanning |
| **Notifications** | Cross-cutting: in-app, email, push, optional SMS/channels; preferences, templates, batching; consumed by all modules and automations |

### Vertical Module Extensibility

Vertical-specific modules (real estate, law firms, medical practices, etc.) extend the core domains rather than replacing them. Each vertical is a plugin that: (a) adds domain-specific entities (e.g., "Properties" for real estate), (b) extends existing entities via JSONB custom fields, (c) provides vertical-specific UI components, and (d) registers vertical-specific automation triggers and actions. The core platform remains vertical-agnostic; verticals are opt-in feature flags per tenant.

- **Extension points only:** Verticals plug in via defined interfaces (new entities, custom fields, triggers/actions); they do not mutate core schemas or replace core domain logic.
- **No core schema mutations:** Core tables and migrations are owned by the platform; verticals add their own tables or use JSONB on existing entities.
- **Feature-flag isolation:** Vertical code paths are gated by tenant-level feature flags so rollout and rollback are safe and measurable.

---

## 4. Technology Choices Summary

| Layer | Choice | Rationale |
|-------|--------|-----------|
| **Backend** | NestJS + TypeScript (Express adapter) | Module system matches domain boundaries; decorators enable clean cross-cutting concerns (auth, tenancy); strong typing across stack. Default Express HTTP adapter (Fastify considered, not adopted to avoid extra complexity). |
| **Database** | PostgreSQL + Prisma | PostgreSQL for RLS, JSONB flexibility, mature ecosystem; Prisma for type-safe queries and migration management. **Prisma + RLS integration must be validated by a spike before any business logic:** session variable per request/connection, behavior with connection pooling and concurrent tenants, and survival of tenant context inside transactions and in worker jobs. If the spike fails, consider query-level SET or an ORM with finer connection control. |
| **Web Frontend** | React + Vite (current default) | Current default for the authenticated web app is a Vite-powered React SPA (faster iteration, simpler dashboard architecture). Open to revisiting alternatives later (e.g. SSR/SSG-capable frameworks) when public pages/SEO become a higher priority. |
| **Mobile** | React Native + Expo | Significant code sharing with web (API clients, business logic); Expo simplifies build/deploy for solo founder |
| **AI Integration** | External APIs (OpenAI, Anthropic) | Build vs. buy: external APIs for state-of-the-art models. Thin internal AI gateway: unified interface (e.g. text/image/score) plus provider adapters; LangChain.js optional later for complex chaining or agents. |
| **Job Queue** | BullMQ + Redis | Battle-tested, Redis-backed, supports delayed/scheduled jobs, retries, and priority queues; simpler than Temporal for current scale |
| **Real-time** | WebSockets + rooms/channels (TBD) | Push notifications, live updates (dashboards, tickets, fleet, conversations). Room/channel semantics are required (e.g. per-tenant, per-ticket, per-conversation). Choose implementation when building (native WebSockets + pub/sub, or a library that provides rooms). |
| **API style** | REST | Single, well-understood contract for web and mobile; business logic in domain services; consistent conventions (pagination/filtering/expands). **Note:** GraphQL can be reconsidered later if UI aggregation becomes a bottleneck, but it is not a current requirement. See `docs/api-design.md`. |
| **Public API** | REST, API keys / OAuth | Third-party apps (external CRMs, accounting, dashboards) integrate via a public API; auth, versioning, and rate limits specified in API design and security docs. |
| **Deployment** | Docker; Compose or PaaS initially, Kubernetes as evolution path | Docker for reproducible images (API, workers). MVP can use Docker Compose (single node) or a PaaS (Railway, Render, Fly.io); Kubernetes when multi-node scaling or operational requirements justify it. Full details → `docs/deployment.md`. |

---

## 5. Key Architectural Principles

1. **Tenant isolation is non-negotiable.** Every database query, API endpoint, and background job is tenant-scoped. RLS policies are the last line of defense, not the only line.

2. **External APIs over building complex systems.** Payments (Stripe), AI (OpenAI/Anthropic), ads (platform APIs), email (Resend/Postmark), SMS (Twilio) — we integrate, not rebuild. See `docs/external-integrations.md` for open source preferences and paid API subscription model.

3. **Domain communication: direct calls for request-scoped results, durable events for cross-domain side effects.** When the API response depends on another domain's result (e.g. "create deal and return it with first activity"), domains call each other via service interfaces (direct call). When something *happened* and one or more domains or the Automations engine should react (e.g. "DealWon" → create invoice in ERP, notify, run automations), the producing domain publishes a domain event **via BullMQ** (enqueue a job per event type). Subscriber workers process events; delivery is at-least-once with retries. This gives durability (no lost events on process crash), uses existing Redis/BullMQ infra, and keeps extraction options open (event transport can be swapped for another message broker if modules are split out). See `docs/ai-automation.md` or ADR for event catalog and conventions.

4. **JSONB for extensibility, not EAV.** Custom fields and vertical-specific data use PostgreSQL JSONB columns. Entity-Attribute-Value patterns are avoided due to query complexity.

5. **Background jobs for anything non-instant.** AI generation, email sending, report building, ad campaign sync — all async via BullMQ workers.

6. **Audit everything for compliance.** All data mutations log before/after state to an append-only audit table. This supports GDPR accountability and debugging.

7. **Mobile is a first-class citizen, not an afterthought.** Core workflows are designed mobile-first; the API serves both web and mobile without platform-specific hacks.

**Event semantics:** Domain events are published as **BullMQ jobs** (durable, Redis-backed). Subscriber workers process them; delivery is at-least-once with retries and optional dead-letter queues. Use direct calls when the API response depends on the result; use events when one or more domains or the Automations engine should react and the response does not depend on those side effects.

### Key Operational Guarantees

- **Tenant isolation:** Enforced at the database via RLS and in the application via tenant context on every request and job; ORM and middleware guardrails ensure no query or job runs without a tenant scope.
- **Data residency:** Tenant data and processing stay within the chosen region (EU/US); assignment at signup; subprocessors and transfer mechanisms documented in security/compliance.
- **Audit log retention and erasure:** Audit entries have a defined retention; right-to-erasure includes audit data (anonymization or deletion per policy). See `docs/security.md`.
- **Event and job delivery:** Events and background work are both enqueued to BullMQ; delivery is at-least-once with retries. No exactly-once guarantee across domains; idempotent handlers recommended where needed.
- **AI scope:** **Unified AI agent** interacts with all modules via tool calls (CRM, ERP, HR, Projects, Fleet, etc.); makes decisions from user prompts and business context; content generation (copy, images, video), lead scoring, campaign optimization. Implemented with narrow tools per domain, guardrails, and user confirmation for sensitive actions.

### Cross-cutting infrastructure and capabilities

- **File and object storage:** Tenant-isolated file storage is required by many modules (contracts, invoices, receipts, content assets, fleet photos, knowledge base uploads). Use object storage with an S3-compatible API; tenant-namespaced paths or buckets; time-limited signed URLs for access. Optional virus scanning on upload. Choice of provider (managed or self-hosted) is left to implementation; see `docs/external-integrations.md`.

- **Notifications:** A notification service is cross-cutting: in-app (real-time), email, push (mobile), and optionally SMS or conversational channels. Per-user preferences (which channels, which events), template system, and optional batching/digests so users are not flooded. Modules and automations trigger notifications via a single abstraction.

- **Caching:** Redis (already used for BullMQ) serves as the application cache. All cache keys are tenant-namespaced. Use TTL-based expiry for read-heavy data (e.g. product catalog, tenant settings); invalidate on write via domain events (e.g. "ProductUpdated" invalidates product cache). Dashboard and reporting endpoints can use short-lived response cache where appropriate.

### ERP phasing

Full ERP is large; build in phases so the product can ship and earn before the full general ledger is in place. Suggested phasing (order and scope can be adjusted): **(1) MVP** — Invoices, payments, products catalog, basic revenue tracking; integrate with existing payment provider. **(2)** — AP/AR, expense management, bank reconciliation. **(3)** — General ledger, chart of accounts, journal entries, multi-currency. **(4)** — Budgeting, fixed assets, fiscal periods, multi-entity consolidation. Document chosen phases in product roadmap or ADR.

### AI agent architecture (design points)

- **Tool registry:** Each module registers the tools the agent can call (e.g. search contacts, create deal, create invoice). Registry is tenant- and role-aware so the agent only exposes tools the current user is allowed to use.
- **Permission proxy:** The agent acts as the user; every tool call is authorized with the same RBAC as API calls. No privilege escalation.
- **Context builder:** For each prompt, assemble relevant context (current entity, recent activity, tenant settings) within a token budget. Avoid sending entire datasets.
- **Loop budget:** Cap the number of tool calls per agent invocation (e.g. max N per request) to prevent runaway cost and infinite loops.

### Search

Many entity types (contacts, deals, invoices, products, tickets, knowledge base, etc.) will need search. Start with PostgreSQL full-text where sufficient; plan for a **dedicated search engine** later (full-text or vector). Design domain events so that create/update/delete events can feed an index (e.g. "ContactCreated" → index document). When a search engine is introduced, add subscribers that maintain the index; no big-bang migration. Technology choice (e.g. which search engine) is left to implementation; see `docs/external-integrations.md` for options.

### Database scaling (future)

Single PostgreSQL carries all modules; at scale, access patterns differ (e.g. high write volume for fleet location, heavy analytical queries for reporting, append-only audit logs). Plan for (when needed): **partitioning** (e.g. audit log by month; time-series data); **read replicas** for reporting and dashboards so heavy queries do not block the API; **time-series extension or separate store** for fleet/location data if write volume grows. No change required for MVP; document as a known scaling path.

---

## 6. Critical Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| **Tenant data leakage** | Critical — legal, trust destruction | RLS policies as DB-level enforcement; tenant context injected at middleware level; comprehensive integration tests for isolation |
| **Prisma + RLS not validated** | Critical — leakage or broken queries | Run validation spike before building business logic; verify per-request and per-job tenant context with pool, transactions, and concurrency; have fallback (query-level SET or different ORM) in mind |
| **RBAC / permissions complexity underestimated** | High — data exposure, broken workflows, slow delivery | Design RBAC early and treat it as cross-cutting: role-based access control (RBAC) is the baseline; consider a hybrid with scoped permissions (per-entity, per-module) and attribute-based checks for sensitive domains (HR, finance). Ensure agency access is scope-granted by the business; AI agent must inherit the invoking user’s permissions (no privilege escalation). |
| **AI API costs spiral** | High — unprofitable unit economics | Per-tenant usage quotas; caching of repeated generations; user-facing cost visibility; tiered pricing aligned to AI usage |
| **Vertical modules become tangled** | Medium — technical debt, slow iteration | Strict plugin interface contracts; verticals cannot modify core schemas; feature flags isolate vertical code paths |
| **Solo founder bottleneck** | Medium — slow progress, burnout | Modular monolith keeps ops simple; heavy use of managed services (DB, Redis, hosting); AI-assisted development (Cursor) |
| **External API dependency failures** | Medium — degraded functionality | Circuit breakers on all external calls; graceful degradation (e.g., queue retries for transient failures); status page monitoring |

---

## 7. Architecture Review Notes (Feb 2026)

Decisions from architecture review, reflected in this document:

- **Backend:** NestJS retained with **default Express adapter**. Fastify adapter evaluated; not adopted to avoid a second HTTP layer and ecosystem split at current stage.
- **Real-time:** Rooms/channels are required for many features; choose implementation (native WebSockets with a pub/sub layer, or a library with rooms) when building.
- **Prisma + RLS:** Run a **validation spike as the first technical task**: set Postgres session variable (e.g. `app.current_tenant`) per request and per worker job; verify tenant isolation with connection pooling, concurrent requests, and transactions. Worker jobs must set tenant context before any query.
- **Verticals:** Build **2–3 verticals as concrete modules first**, then extract a plugin pattern; avoid a large plugin system before real use cases exist.
- **Frontend:** Current default is **React + Vite** for the authenticated web app. Alternative frontend stacks remain open and can be reconsidered as requirements evolve (especially when public SEO pages become a priority).
- **Alternatives considered and not chosen:** Vue for web (would split stack with React Native); Django/FastAPI (would split language with TS frontend); Fastify standalone (would lose NestJS module guardrails).
- **Approval workflow:** Expenses, POs, and timesheets each use approval workflows; a common approval engine (configurable steps, delegates) could serve these and other entities. Not documented as a separate first-class feature for now.

---

## 8. What's NOT Covered Here

This document provides the architectural overview and technology decisions from the Feb 2026 review (Section 7). Detailed specifications live in separate documents:

- **Product Scope** → `docs/product-scope.md`
- **External Integrations** → `docs/external-integrations.md` (open source preferences, paid API subscriptions)
- **Detailed Data Model** → `docs/data-model.md`
- **Architecture Decision Records** → `docs/adr/` (individual decisions with context and consequences)
- **API Design & Conventions** → `docs/api-design.md` (REST conventions, public API auth and versioning)
- **Deployment & Infrastructure** → `docs/deployment.md` (Docker, Compose/PaaS, Kubernetes path)
- **Security & Compliance Details** → `docs/security.md`
- **AI Automation Engine Design** → `docs/ai-automation.md`
- **Mobile Architecture** → `docs/mobile-architecture.md`
- **Resilience & Error Handling** → `docs/resilience-strategy.md`
- **Frontend options comparison** → `docs/frontend-options.md`
- **Module boundaries strategy** → `docs/module-boundaries-strategy.md`
- **Logging & observability** → `docs/logging-observability.md`
- **AI coding & prompting guidelines** → `docs/ai-coding-guidelines.md`

---

## Quick Reference Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        Clients                                   │
│   Web (Next.js)  ←→  Mobile (React Native)  ←→  Public API      │
│                    (third-party apps, API keys / OAuth)          │
└─────────────────────────┬───────────────────────────────────────┘
                          │ HTTPS / WebSocket
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │  Unified AI Agent — interacts with all modules; decisions   │ │
│  │  from user prompts and business context; tool calls         │ │
│  └─────────────────────────────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────────────┤
│                     API Gateway / BFF                            │
│                   (NestJS - REST)                                │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌───────────┐ │
│  │  Core   │ │   CRM   │ │   ERP   │ │   HR    │ │Automations│ │
│  │ Module  │ │ Module  │ │ Module  │ │ Module  │ │  Module   │ │
│  └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘ └─────┬─────┘ │
│  ┌─────────┐ ┌──────────┐ ┌──────┐ ┌──────────┐ ┌────────────┐ │
│  │Content/ │ │Link-bio  │ │Fleet │ │AI-Agents │ │Integrations│ │
│  │Marketing│ │(creators)│ │ Mgr  │ │(gateway) │ │            │ │
│  └────┬────┘ └────┬─────┘ └──┬───┘ └────┬─────┘ └─────┬──────┘ │
│       └──────────┬┴──────────┬┴──────────┬┴────────────┘       │
│                  ▼           ▼           ▼                      │
│         ┌─────────────────────────────────────┐                 │
│         │  Durable events (BullMQ) + workers  │                 │
│         └─────────────────────────────────────┘                 │
└─────────────────────────┬───────────────────────────────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│  PostgreSQL  │  │    Redis     │  │   Workers    │
│   (+ RLS)    │  │  (BullMQ)    │  │  (AI, Jobs)  │
└──────────────┘  └──────────────┘  └──────┬───────┘
                                           │
                          ┌────────────────┼────────────────┐
                          ▼                ▼                ▼
                   ┌───────────┐    ┌───────────┐    ┌───────────┐
                   │  OpenAI   │    │  Stripe   │    │ Ad APIs   │
                   │ Anthropic │    │           │    │Meta/Google│
                   └───────────┘    └───────────┘    └───────────┘
```

---

*This document should be updated when major architectural decisions change. For specific decision rationale, see the relevant ADR in `docs/adr/`.*

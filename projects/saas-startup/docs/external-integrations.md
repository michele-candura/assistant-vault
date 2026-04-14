a# External Integrations

> **Last Updated:** March 8, 2026  
> **Status:** Living Document  
> **Audience:** Developers, architecture decisions

---

## 1. Principles

- **Open source first (MIT preferred)** — Use truly open source libraries and services where possible to avoid reinventing the wheel and reduce vendor lock-in. MIT and similar permissive licenses (ISC, BSD-2, Apache 2.0) are preferred.
- **Single language stack** — Prefer TypeScript/JavaScript (Node.js) for all integrations to keep the codebase maintainable. Avoid adding Python, Go, or Rust services unless strictly necessary.
- **Build vs. integrate** — Integrate external APIs for complex domains (payments, AI, ads); build thin adapters and gateways in-house.
- **Paid API subscriptions** — The platform will offer services to customers via paid API subscriptions (e.g. AI models, premium features). These are resold or proxied; the platform owns billing and usage tracking.

---

## 2. Integration Categories

### 2.1 Payments & Billing

| Need | Open Source (MIT / permissive) | Paid API (integrate or resell) | Notes |
|------|--------------------------------|--------------------------------|-------|
| Payment processing | — | **Stripe** (primary) | Industry standard; SDK is MIT. Use for subscriptions, one-time, invoicing. |
| Payment orchestration | **PayKit** (ISC) — TypeScript, Prisma/Drizzle adapters | — | Abstraction over Stripe, PayPal, Polar; unified webhooks, own DB for subscriptions. Reduces vendor lock-in. |
| Alternative gateways | — | PayPal, Polar (open-source billing) | PayKit supports multiple providers; add as needed. |

**Recommendation:** Use **Stripe** directly for MVP (simplest). Consider **PayKit** when adding PayPal or regional providers. All SDKs are TypeScript-compatible.

---

### 2.2 AI & LLMs

| Need | Open Source (MIT / permissive) | Paid API (integrate or resell) | Notes |
|------|--------------------------------|--------------------------------|-------|
| Cloud AI (generation, scoring) | — | **OpenAI**, **Anthropic** | Primary; SDKs are MIT. Platform resells to customers via usage-based billing. |
| Local / self-hosted AI | **Ollama** (MIT) | — | Optional: run models locally for privacy-sensitive tenants. REST API; Node.js client available. |
| AI gateway / routing | **LangChain.js** (MIT) | — | Optional for complex chains, agents, multi-provider routing. |
| Embeddings / vector search | **pgvector** (PostgreSQL extension, PostgreSQL license) | OpenAI Embeddings, etc. | Use pgvector for embeddings in PostgreSQL; no extra service. |

**Unified AI agent:** The platform has an AI agent that interacts with all modules (CRM, ERP, HR, Projects, Fleet, etc.) and makes decisions from user prompts and business context. It uses tool calls to read/write domain data; LangChain.js or similar can orchestrate agentic flows. See `docs/architecture-overview.md` and `docs/product-scope.md`.

**Paid API subscription model:** Platform charges tenants for AI usage (tokens, generations). Internal gateway tracks usage per tenant, enforces quotas, and bills via Stripe. See Section 4.

---

### 2.3 Email

| Need | Open Source (MIT / permissive) | Paid API (integrate or resell) | Notes |
|------|--------------------------------|--------------------------------|-------|
| SMTP client | **Nodemailer** (MIT) | — | Universal SMTP client; works with any provider. |
| Email API (transactional) | — | **Resend**, **Postmark**, **SendGrid**, **Mailgun** | Resend has MIT SDK; others have official Node SDKs. Use Nodemailer + SMTP or provider SDK. |
| Email templates | **MJML** (MIT), **React Email** (MIT) | — | Build responsive templates; render to HTML. |

**Recommendation:** Use **Nodemailer** with Resend/Postmark SMTP, or Resend SDK directly. Single abstraction in code; swap provider via config.

---

### 2.4 SMS

| Need | Open Source (MIT / permissive) | Paid API (integrate or resell) | Notes |
|------|--------------------------------|--------------------------------|-------|
| SMS sending | — | **Twilio**, **Vonage**, **Resend** (SMS) | All have Node.js SDKs. |
| Self-hosted SMS gateway | **Textbee** (MIT), **Jasmin** | — | Textbee: Android phone as gateway. For low volume or specific use cases. |

**Recommendation:** Use **Twilio** or **Resend** (if already using Resend for email) for production. Abstract behind a `SmsService` interface.

---

### 2.5 Document Generation & E-Signature

| Need | Open Source (MIT / permissive) | Paid API (integrate or resell) | Notes |
|------|--------------------------------|--------------------------------|-------|
| Document merge (Word, PDF) | **Docxtemplater** (MIT) | — | Merge CRM/ERP/HR data into .docx, .pptx, .xlsx templates. |
| PDF generation | **pdfme** (MIT), **LibPDF** (MIT) | — | TypeScript-first; generate PDFs from templates or code. |
| E-signature | **DocuSeal** (AGPL-3.0), **OpenSign** (AGPL-3.0) | **DocuSign**, **HelloSign** | DocuSeal: self-host, API, webhooks. AGPL requires source disclosure if you distribute modified version. |
| HTML to DOCX | **TurboDocx** (MIT) | — | HTML-to-DOCX conversion; useful for reports. |

**Recommendation:** Use **Docxtemplater** for document generation (contracts, quotes, reports). For e-signature: **DocuSeal** if self-hosting and AGPL is acceptable; otherwise DocuSign/HelloSign API. Abstract behind `E-signatureService`.

---

### 2.6 File & Object Storage

| Need | Open Source (MIT / permissive) | Paid API (integrate or resell) | Notes |
|------|--------------------------------|--------------------------------|-------|
| Object storage (S3-compatible) | **MinIO** (AGPL-3.0) | **AWS S3**, **Cloudflare R2**, **Backblaze B2** | S3 API is the standard. Use `@aws-sdk/client-s3` (Apache 2.0) with any S3-compatible backend. |
| Local disk | — | — | For small deployments; use local path or bind mount. |

**Recommendation:** Use **AWS SDK v3** (`@aws-sdk/client-s3`) with S3-compatible storage. Configure backend via env (AWS, R2, MinIO). MinIO is AGPL if self-hosted.

---

### 2.7 Search

| Need | Open Source (MIT / permissive) | Paid API (integrate or resell) | Notes |
|------|--------------------------------|--------------------------------|-------|
| Full-text search | **Meilisearch** (MIT) | **Algolia**, **Typesense Cloud** | Meilisearch: typo-tolerant, instant search, REST API. Typesense is GPL. |
| PostgreSQL full-text | **pg_trgm**, built-in `tsvector` | — | For simple search within PostgreSQL; no extra service. |

**Recommendation:** Start with **PostgreSQL full-text** for MVP. Add **Meilisearch** when search UX (instant, facets, typo tolerance) justifies it. Meilisearch has official JS SDK.

---

### 2.8 Maps, Geocoding & Routing (Fleet)

| Need | Open Source (MIT / permissive) | Paid API (integrate or resell) | Notes |
|------|--------------------------------|--------------------------------|-------|
| Geocoding | **Nominatim** (GPL-2.0) | **Google Maps**, **Mapbox** | Nominatim: OpenStreetMap; self-host or public instance. |
| Routing / directions | **OSRM** (BSD-2), **Valhalla** (MIT) | **Google Directions**, **Mapbox** | OSRM/Valhalla: self-host with OSM data. Valhalla is MIT. |
| Map tiles / display | **Leaflet** (BSD-2), **MapLibre** (BSD-3) | — | Client-side; React wrappers available (react-leaflet, MIT). |

**Recommendation:** Use **Mapbox** or **Google Maps** for production fleet (reliability, support). **OSRM** or **Valhalla** for cost-sensitive or offline scenarios. Abstract behind `MapsService` / `RoutingService`.

---

### 2.9 Ads (Meta, Google, LinkedIn)

| Need | Open Source (MIT / permissive) | Paid API (integrate or resell) | Notes |
|------|--------------------------------|--------------------------------|-------|
| Ad platform APIs | — | **Meta Marketing API**, **Google Ads API**, **LinkedIn Marketing API** | Official SDKs or REST; OAuth for auth. No open source alternative. |

**Recommendation:** Integrate directly. Use official SDKs or `fetch` with OAuth. Rate limits and quotas apply; use background jobs for sync.

---

### 2.10 Conversational Channels (WhatsApp, etc.)

| Need | Open Source (MIT / permissive) | Paid API (integrate or resell) | Notes |
|------|--------------------------------|--------------------------------|-------|
| WhatsApp Business | **MultiWA** (MIT), **OpenWA** (MIT), **Whatomate** | **Meta WhatsApp Cloud API** | MultiWA: TypeScript/NestJS, multi-account, flow builder. Official API: reliable, compliant; paid per message. |
| Other channels | — | **Twilio** (SMS, WhatsApp), **Intercom**, **Crisp** | Depends on channel. |

**Recommendation:** Use **Meta WhatsApp Cloud API** for production (compliance, reliability). **MultiWA** or **OpenWA** for self-hosted, cost-sensitive scenarios. Abstract behind `ConversationalChannelService`.

---

### 2.11 Calendar Sync

| Need | Open Source (MIT / permissive) | Paid API (integrate or resell) | Notes |
|------|--------------------------------|--------------------------------|-------|
| Calendar APIs | — | **Google Calendar API**, **Microsoft Graph** (Outlook) | OAuth; read/write events. |
| iCal / CalDAV | **ical.js** (MPL-2.0), **tsdav** (MIT) | — | Parse/generate .ics; CalDAV for self-hosted. |

**Recommendation:** Integrate **Google Calendar** and **Microsoft Graph** via OAuth. Use `ical.js` or similar for parsing. No open source replacement for cloud calendars.

---

### 2.12 Job Boards (HR)

| Need | Open Source (MIT / permissive) | Paid API (integrate or resell) | Notes |
|------|--------------------------------|--------------------------------|-------|
| Job posting | — | **Indeed API**, **LinkedIn Job API** | Post jobs, sync applications. Official APIs. |

**Recommendation:** Integrate directly. Each has different auth and data model; build adapters per provider.

---

### 2.13 Analytics & BI Export

| Need | Open Source (MIT / permissive) | Paid API (integrate or resell) | Notes |
|------|--------------------------------|--------------------------------|-------|
| Export (CSV, Excel) | **ExcelJS** (MIT), **Papa Parse** (MIT) | — | Generate Excel; parse CSV. |
| Charts (client) | **Recharts** (MIT), **Chart.js** (MIT) | — | Dashboards, reports. |
| BI connectors | — | **Metabase** (AGPL), **Superset** (Apache 2.0) | Optional: connect BI tools to platform DB. |

**Recommendation:** Use **ExcelJS** for exports. **Recharts** or **Chart.js** for dashboards. All MIT.

---

## 3. Language & Stack Summary

| Category | Preferred Stack | Avoid |
|----------|------------------|-------|
| Backend adapters | NestJS + TypeScript | Python/Go services (unless isolated) |
| SDKs | Official Node.js / TypeScript | Unmaintained or Python-only |
| Search | Meilisearch (Rust binary, HTTP API) | — |
| AI local | Ollama (Go binary, HTTP API) | — |
| Storage | S3 SDK (TypeScript) | Language-specific clients |
| Maps | Mapbox/Google JS SDK + Node server | — |

**Rule:** External services run as binaries (Meilisearch, Ollama, PostgreSQL, Redis) or managed APIs. Application code stays in TypeScript.

---

## 4. Paid API Subscriptions (Platform → Customer)

The platform will offer services to tenants via **paid API subscriptions**:

| Service | Model | Implementation |
|---------|-------|----------------|
| **AI (LLMs)** | Usage-based (tokens, generations) | Internal AI gateway; track usage per tenant; bill via Stripe metered billing or usage add-ons. |
| **SMS** | Per-message or bundle | Proxy through Twilio/Resend; markup or flat fee. |
| **E-signature** | Per-document or monthly | If using DocuSign/HelloSign: resell. If DocuSeal: charge for platform tier. |
| **Storage** | Per GB | S3/R2 usage; aggregate and bill. |
| **Premium integrations** | Per integration / tier | Feature-flag; charge for Meta Ads, WhatsApp, etc. |

**Architecture:** A **UsageService** (or similar) records consumption per tenant and integration. Billing job aggregates and syncs to Stripe. Quotas enforced at gateway level (e.g. AI gateway rejects when limit exceeded).

---

## 5. Quick Reference: MIT / Permissive Libraries

| Purpose | Library | License |
|---------|---------|---------|
| Payments | Stripe SDK, PayKit | MIT, ISC |
| Email | Nodemailer, Resend SDK, MJML, React Email | MIT |
| Document merge | Docxtemplater | MIT |
| PDF | pdfme, LibPDF | MIT |
| Storage | @aws-sdk/client-s3 | Apache 2.0 |
| Search | Meilisearch client | MIT |
| AI | OpenAI SDK, Anthropic SDK, Ollama | MIT |
| Maps (client) | Leaflet, MapLibre, react-leaflet | BSD |
| Charts | Recharts, Chart.js | MIT |
| Excel | ExcelJS | MIT |
| WhatsApp (self-host) | MultiWA, OpenWA | MIT |

---

## 6. References

- **Architecture** → `docs/architecture-overview.md`
- **Product Scope** → `docs/product-scope.md`
- **API Design** → `docs/api-design.md`
- **Security (subprocessors, data transfer)** → `docs/security.md`

---

*Update this document when adding or changing integrations.*

# Cross-cutting — Product Scope

> **Module:** Cross-cutting capabilities  
> **Source:** [Product Scope](../product-scope.md)  
> **Last Updated:** March 8, 2026

---

Capabilities that span multiple modules or sit outside a single domain.

## Current Features

- **Unified AI agent** — Sits on top of all modules; interacts with CRM, ERP, HR, Projects, Fleet, Content & marketing, Help desk, etc.; makes decisions from user prompts and business needs; see AI-Agents.
- **Conversational channels** — Messaging (e.g. WhatsApp Business, or similar) across domains: chat with contacts from CRM (sales), support (tickets), marketing (leads), and other contexts; unified inbox or per-domain views; conversations linked to contacts, deals, tickets. Listed in Integrations for provider (e.g. WhatsApp).
- **Portals** — Single capability, two use cases: **(1) Customer portal** — logged-in customers see orders, invoices, support tickets, and account details (CRM + e-commerce self-service). **(2) Vendor portal** — logged-in suppliers see POs, submit invoices, update delivery status (procurement). Same auth and tenant model; different roles and data.
- **Organisation structure: multi-entity and agencies** — **(1) Multi-entity** — A tenant (one customer of the platform) can have multiple **entities** (e.g. departments, branches, legal entities) for accounting, reporting, and operations. **(2) Agencies** — An agency account can manage **multiple tenants** (multiple client businesses) with scoped permissions; each business is still a separate tenant. So: multi-entity = one business with many departments/branches; agency = one account managing many businesses. See architecture overview for data model.
- **Scheduling** — Shared availability and slot management; calendar sync. Powers **(1) Booking** (Content & marketing): public booking pages for demos, consultations, services; **(2) Interview scheduling** (HR): schedule interviews with candidates, link to candidate record.
- **Shifts (employee / workforce scheduling)** — Shift definitions, rosters, and assignments (who works when; optional role, location, or vehicle). Swap/cover requests and visibility. Single source of truth for scheduled work. **Used by:** HR (payroll, leave, compliance), Fleet (driver roster, vehicle assignment), Projects (capacity, who is on shift), Scheduling (availability can be derived from or constrained by shifts).
- **File storage** — Tenant-isolated object storage (S3-compatible) for documents, contracts, invoices, receipts, content assets, fleet photos, knowledge base uploads; signed URLs for access; optional virus scanning. Used by Contracts, ERP, Content & marketing, Fleet, Content & knowledge, and others.
- **Notifications** — In-app, email, push (mobile), and optionally SMS or conversational channels; per-user preferences and templates; batching/digests. Used by all modules and automations.
- **Reporting and dashboards** — Single place for all reporting: module-level (ERP, fleet, HR, projects, support, etc.) and cross-module dashboards; saved and scheduled reports; export for BI tools. Domain-specific reporting (e.g. financial, operational) uses this capability.
- **Public API** — REST, API keys / OAuth for third-party apps (CRMs, accounting, dashboards)
- **Real-time** — WebSockets for notifications and live updates
- **Mobile** — React Native app (first-class)
- **Verticals** — Extensions per industry (e.g. real estate, law firms, medical practices)
- **White-label** — Tenant branding (logo, colours, custom domain); see Core
- **Compliance** — GDPR, data residency (EU/US), right to erasure, portability

---

## Specific Features to Explore

<!-- Add detailed feature ideas, UX considerations, edge cases -->

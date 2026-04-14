# Product Scope

> **Last Updated:** March 8, 2026  
> **Status:** Living Document  
> **Audience:** Product, developers, stakeholders

---

The platform is a **complete business solution** combining CRM, **full ERP**, HR, **projects** (with time tracking and timesheets), **contracts & e-signature**, content & marketing (including **booking/scheduling**), **e-commerce storefront**, **cross-module dashboards**, **help desk/support**, **content & knowledge** (public and internal), link-in-bio, fleet management, and the feature-request feedback loop. A **unified AI agent** sits on top of all modules: it interacts with CRM, ERP, HR, Projects, Fleet, and others, and makes decisions based on user prompts and business needs. Features are grouped by domain below.

## Core

- Tenant management
- User authentication
- RBAC (roles and permissions)
- Audit logging
- Platform settings
- **White-label** — Tenant-level branding: logo, colours, custom domain for portals and app; rebrand the experience for each tenant (e.g. agencies or resellers)

## CRM

- Contacts
- Companies
- Deals
- Pipeline stages
- Activities
- Notes
- Relationship tracking (contact ↔ company ↔ deal)
- **Customer segmentation** — Segments and lists based on criteria (demographics, behaviour, purchase history, engagement, custom fields); saved views and filters; tags and labels; lifecycle stages; dynamic vs static segments; used for targeting (marketing, sales, support)
- **Sales features** — Quotas and targets (per rep, team, period); pipeline and revenue forecasting; **sales commission** (rules, calculation, pay or link to payroll); leaderboards and sales analytics; typical sales workflows (territories, round-robin, assignment rules)

## Contracts & e-signature (cross-domain)

- Contract templates and versioning
- **Document generation** — Merge data from CRM/ERP/HR (contacts, deals, orders, products, employees) into templates to generate quotes, proposals, contracts, employment documents, and reports (e.g. PDF/Word); reusable templates with merge fields; used across **sales** (quotes, proposals, order-related contracts) and **HR** (employment contracts, NDAs, etc.)
- Contracts linked to CRM deals and sales (quote → order → contract); HR contracts linked to employees and roles
- E-signature (built-in or integration) for proposals, orders, agreements, and HR documents
- Contract status and renewal/expiry reminders

## ERP (Enterprise Resource Planning)

Full ERP is integrated into the platform so businesses can run finance, operations, and reporting in one place.

### Finance & accounting

- General ledger (chart of accounts, journal entries, posting)
- Accounts payable and accounts receivable
- Fixed assets (register, depreciation)
- Bank reconciliation and bank feeds
- **Expense management** — Employee expense claims (receipts, categories, approval workflow, reimbursement); links to HR (employees submit) and ERP (approval, payment, GL posting)
- Financial reporting (P&L, balance sheet, cash flow)
- **Budgeting and planning** — Budgets by department, project, or entity; budget vs actuals reporting; planning and forecasts (linked to financial reporting)
- Multi-currency and exchange rates
- Fiscal periods, year-end close
- Invoices, payments, and revenue recognition linked to orders

### Inventory & operations

- Stock levels and warehouse/location management
- Inventory valuation (FIFO, average cost, etc.)
- Reorder points and stock alerts
- Lot and serial number tracking (where applicable)
- Stock movements (receipts, transfers, adjustments)

### Purchasing & procurement

- Purchase orders and approval workflows
- Supplier management (master data, terms)
- Goods receipt and matching to POs
- Three-way match (PO, receipt, invoice)
- **Vendor portal** — Suppliers log in to see POs, submit invoices, update delivery status (see Cross-cutting: Portals)

### Sales & order management

- Quotes and sales orders (linked to CRM deals where relevant)
- Order fulfillment and shipping
- Invoicing from orders
- Products/services catalog (shared with finance and sales)
- **E-commerce storefront** — Public product pages, cart, checkout; orders flow into ERP sales and fulfillment
- **Subscription billing** — Recurring plans, MRR, renewals, plan changes, optional usage-based billing; dunning and lifecycle (linked to Stripe where relevant)

## HR

- Employee profiles
- Leave/absence management
- Payroll basics
- **Job postings** — Publish and manage openings; **integrations with job boards** (Indeed, LinkedIn for now) to post and sync applications
- **Candidate tracking** — Applications, stages, notes; **interview scheduling** (see Cross-cutting: Scheduling)
- **Training & learning (internal platform)** — Courses and learning paths, assignments (by role or individual), completion tracking, optional compliance and certification; internal LMS for the organisation

## Projects

- Project list, milestones, and phases
- Tasks and subtasks with assignees and due dates
- **Resource / capacity planning** — View and plan who is assigned to what; capacity and utilization across projects and teams; links to HR (people) and projects (assignments)
- Optional Gantt/timeline view
- **Time tracking** — Log time per project, task, or client (for billing and cost)
- **Timesheets** — Submit and approve timesheets; link to payroll (HR) and/or project billing
- Link projects to CRM deals and ERP orders where relevant

## Automations

- Workflow engine
- Triggers (event, schedule, webhook)
- Actions (e.g. send email, create task, update record)
- Execution logs
- User-defined automation rules

## AI-Agents (unified agent on top of all modules)

- **Unified AI agent** — Interacts with all modules (CRM, ERP, HR, Projects, Fleet, Content & marketing, Help desk, etc.) via tool calls; makes decisions from user prompts and business context; answers questions, suggests actions, and executes operations within guardrails and user confirmation for sensitive steps.
- Content generation (copy, images; video later)
- Lead scoring and campaign optimization
- Thin gateway to OpenAI / Anthropic
- Uses **internal knowledge base and documents** (see Content & knowledge) for context-aware help
- Narrow tools per domain, guardrails, and user confirmation for sensitive actions (e.g. financial, HR)

## Content & marketing

- **Content management** — Pages, posts, and assets (images, videos, documents); **content planning and calendar** (plan, schedule, and publish content across channels); **event management** — webinars, events, registration, and promotion
- **Form builder** — Custom forms (lead gen, contact, applications); embeddable on site or shareable link; submissions feed into CRM (contacts, deals) or other modules; optional automations on submit. Can also be used for **surveys** (e.g. NPS, CSAT); surveys are not a primary focus but form builder can collect feedback that Help desk analyses.
- **Email marketing** — Email campaigns, segments and lists, sequences (drip/automated flows), templates, A/B testing; opens, clicks, and basic analytics; link to CRM and campaigns
- Campaigns and landing pages (linked to ads integrations where relevant)
- **Booking / scheduling** — Public booking pages for demos, consultations, services (see Cross-cutting: Scheduling)
- SEO and metadata management
- Marketing analytics (traffic, conversions) — complements ad integrations

## Link in bio (content creators)

- Single shareable URL (e.g. yourbrand.app/you) with a customizable profile page
- Multiple links (social, shop, links, media) with ordering and grouping
- Custom branding (themes, fonts, button styles)
- Optional analytics (clicks, visits) per link
- Optional integration with CRM/ERP (e.g. link to product or contact form)

## Fleet management

- Vehicle register (vehicles, type, plate, documents, insurance)
- Driver management and assignment to vehicles
- **Location tracking** — Real-time or periodic vehicle/driver location; optional geofencing and location history
- Maintenance scheduling and history (services, repairs, reminders)
- **Fuel and mileage tracking** — Refuel records, fuel consumption, fuel reports and receipts
- **Issues and reports** — Driver-reported issues (breakdowns, damage, incidents), fuel reports, and other operational reports; status and follow-up
- **Orders management** — Orders assigned to vehicles/drivers (e.g. deliveries, jobs); order status and completion linked to trips/routes
- Routes and trips (optional: basic route log or integration with maps)
- Fleet reporting (costs, utilization, compliance)

## Integrations

- Stripe (payments)
- Meta / Google / LinkedIn Ads
- Email providers
- SMS
- Calendar sync
- **Job boards** — Indeed, LinkedIn (post jobs and sync applications into HR candidate tracking)
- **Conversational channels** — WhatsApp Business (and similar) for messaging; see Cross-cutting

## Content & knowledge (public and internal)

- **Public knowledge base** — Help centre for support self-service; used by Help desk (see below). Same content capability as internal; different audience and access.
- **Internal knowledge base & documents** — Internal document library (policies, SOPs, playbooks, how-tos), wiki-style content; versioning and access control. **AI assistant integration** — AI agents can read and use this content for context-aware help.

## Help desk / support

- Support tickets (from customers or internal); status and assignment
- Optional SLA and priority rules
- Public knowledge base (help centre) for self-service — see Content & knowledge
- **Surveys and feedback** — NPS, CSAT, and custom surveys (forms can be built with Form builder; see Content & marketing); collection and analysis; link to support and CRM
- Link to CRM contacts/companies where relevant

## Feature requests & feedback loop

- **Request by module** — Users submit new feature requests tied to a specific module (CRM, ERP, Fleet, etc.).
- **Shared feature list** — Requests appear in a shared list (roadmap/board) so users see what others have asked for.
- **Upvote and comment** — Users can upvote existing requests and add comments (e.g. extra use cases or variants) instead of creating duplicates.
- **Internal approval** — Approved feature requests move to a build pipeline; internal workflow for prioritisation and approval.
- **AI-assisted build and test** — Approved requests can be implemented and tested by AI agents (within guardrails), then prepared for rollout.
- **Targeted rollout** — New features can be delivered to: (1) users who requested them, (2) similar users (e.g. same module/vertical/segment), or (3) the whole user base; with optional opt-in or gradual rollout.

## Cross-cutting

- **Unified AI agent** — Sits on top of all modules; interacts with CRM, ERP, HR, Projects, Fleet, Content & marketing, Help desk, etc.; makes decisions from user prompts and business needs; see AI-Agents.
- **Conversational channels** — Messaging (e.g. WhatsApp Business, or similar) across domains: chat with contacts from CRM (sales), support (tickets), marketing (leads), and other contexts; unified inbox or per-domain views; conversations linked to contacts, deals, tickets. Listed in Integrations for provider (e.g. WhatsApp).
- **Portals** — Single capability, two use cases: **(1) Customer portal** — logged-in customers see orders, invoices, support tickets, and account details (CRM + e-commerce self-service). **(2) Vendor portal** — logged-in suppliers see POs, submit invoices, update delivery status (procurement). Same auth and tenant model; different roles and data.
- **Organisation structure: multi-entity and agencies** — **(1) Multi-entity** — A tenant (one customer of the platform) can have multiple **entities** (e.g. departments, branches, legal entities) for accounting, reporting, and operations. **(2) Agencies** — An agency account can manage **multiple tenants** (multiple client businesses) with scoped permissions; each business is still a separate tenant. So: multi-entity = one business with many departments/branches; agency = one account managing many businesses. See architecture overview for data model.
- **Scheduling** — Shared availability and slot management; calendar sync. Powers **(1) Booking** (Content & marketing): public booking pages for demos, consultations, services; **(2) Interview scheduling** (HR): schedule interviews with candidates, link to candidate record.
- **Shifts (employee / workforce scheduling)** — Shift definitions, rosters, assignments (who works when; optional role, location, vehicle). Swap/cover requests. Used by HR (payroll, leave), Fleet (driver roster), Projects (capacity), Scheduling (availability).
- **File storage** — Tenant-isolated object storage (S3-compatible) for documents, contracts, invoices, receipts, content assets, fleet photos, knowledge base uploads; signed URLs for access; optional virus scanning. See architecture overview.
- **Notifications** — In-app, email, push (mobile), and optionally SMS or conversational channels; per-user preferences and templates; batching/digests; used by all modules and automations. See architecture overview.
- **Reporting and dashboards** — Single place for all reporting: module-level (ERP, fleet, HR, projects, support, etc.) and cross-module dashboards; saved and scheduled reports; export for BI tools. Domain-specific reporting (e.g. financial, operational) uses this capability.
- **Public API** — REST, API keys / OAuth for third-party apps (CRMs, accounting, dashboards)
- **Real-time** — WebSockets for notifications and live updates
- **Mobile** — React Native app (first-class)
- **Verticals** — Extensions per industry (e.g. real estate, law firms, medical practices)
- **White-label** — Tenant branding (logo, colours, custom domain); see Core
- **Compliance** — GDPR, data residency (EU/US), right to erasure, portability

---

**Note (approval workflow):** Expenses, POs, and timesheets each use approval workflows. The platform could implement a common approval engine (configurable steps, delegates) for these and other entities; this is not documented as a separate first-class feature for now.

---

**Module-specific product scopes** — Deep-dive docs per module are in `docs/product-scopes/`. Use them to explore specific features in detail.

*For how these map to architecture and bounded contexts, see `docs/architecture-overview.md`.*

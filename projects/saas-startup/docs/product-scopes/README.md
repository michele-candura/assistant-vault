# Module Product Scopes

> **Purpose:** Deep-dive product scope documents per module. Use these to think through specific features in detail.
> **Source:** Extracted from `docs/product-scope.md`. Update the main product-scope when features are finalised.

---

## Modules

| Module | Doc | Description |
|--------|-----|-------------|
| **Core** | [core.md](./core.md) | Tenant management, auth, RBAC, audit, settings, white-label |
| **CRM** | [crm.md](./crm.md) | Contacts, companies, deals, pipeline, segmentation, sales |
| **Contracts & e-signature** | [contracts-e-signature.md](./contracts-e-signature.md) | Templates, document generation, e-signature (cross-domain) |
| **ERP** | [erp.md](./erp.md) | Finance, inventory, purchasing, sales & orders, e-commerce |
| **HR** | [hr.md](./hr.md) | Employees, leave, payroll, recruiting, training |
| **Projects** | [projects.md](./projects.md) | Projects, tasks, time tracking, timesheets, resource planning |
| **Automations** | [automations.md](./automations.md) | Workflow engine, triggers, actions |
| **AI-Agents** | [ai-agents.md](./ai-agents.md) | Unified AI agent, content generation, lead scoring |
| **Content & marketing** | [content-marketing.md](./content-marketing.md) | Content, forms, email marketing, campaigns, booking |
| **Link in bio** | [link-in-bio.md](./link-in-bio.md) | Content-creator profiles and links |
| **Fleet management** | [fleet-management.md](./fleet-management.md) | Vehicles, drivers, location, maintenance, orders |
| **Scheduling** | [scheduling.md](./scheduling.md) | Shared availability, booking, interview scheduling |
| **Shifts** | (Cross-cutting) | Employee/workforce scheduling; see [cross-cutting.md](./cross-cutting.md) |
| **Integrations** | [integrations.md](./integrations.md) | External service adapters |
| **Content & knowledge** | [content-knowledge.md](./content-knowledge.md) | Public and internal knowledge base |
| **Reporting & dashboards** | [reporting-dashboards.md](./reporting-dashboards.md) | Cross-module dashboards, saved/scheduled reports |
| **Help desk / support** | [help-desk-support.md](./help-desk-support.md) | Tickets, SLA, surveys, CRM link |
| **Feature requests & feedback** | [feature-requests-feedback.md](./feature-requests-feedback.md) | User requests, upvotes, AI-assisted build, rollout |
| **Cross-cutting** | [cross-cutting.md](./cross-cutting.md) | AI agent, portals, scheduling, reporting, etc. |

---

## Cross-module connections

**Where to document:** References live **inside each product scope** (e.g. "Uses **Shifts** (see Cross-cutting) for payroll") and are summarised in **Cross-cutting** ("Used by: HR, Fleet, …"). There is no separate cross-module-connections file. When you add **per-module architecture docs**, put dependency and integration details there (how the module integrates); product scopes keep the high-level "uses X for Y" (what).

*Add specific features and ideas in each module doc. Sync back to `docs/product-scope.md` when decisions are made.*

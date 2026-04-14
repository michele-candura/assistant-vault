# ERP — Product Scope

> **Module:** ERP (Enterprise Resource Planning)  
> **Source:** [Product Scope](../product-scope.md)  
> **Last Updated:** March 8, 2026

---

Full ERP is integrated into the platform so businesses can run finance, operations, and reporting in one place.

## Finance & accounting

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

## Inventory & operations

- Stock levels and warehouse/location management
- Inventory valuation (FIFO, average cost, etc.)
- Reorder points and stock alerts
- Lot and serial number tracking (where applicable)
- Stock movements (receipts, transfers, adjustments)

## Purchasing & procurement

- Purchase orders and approval workflows
- Supplier management (master data, terms)
- Goods receipt and matching to POs
- Three-way match (PO, receipt, invoice)
- **Vendor portal** — Suppliers log in to see POs, submit invoices, update delivery status (see Cross-cutting: Portals)

## Sales & order management

- Quotes and sales orders (linked to CRM deals where relevant)
- Order fulfillment and shipping
- Invoicing from orders
- Products/services catalog (shared with finance and sales)
- **E-commerce storefront** — Public product pages, cart, checkout; orders flow into ERP sales and fulfillment
- **Subscription billing** — Recurring plans, MRR, renewals, plan changes, optional usage-based billing; dunning and lifecycle (linked to Stripe where relevant)

---

## ERP phasing

Full ERP is large; the architecture recommends building in phases so the product can ship before the full general ledger exists. See `docs/architecture-overview.md` (Cross-cutting infrastructure and capabilities → ERP phasing) for suggested phase order (MVP invoices/payments/products → AP/AR and expense → GL and multi-currency → budgeting and consolidation). Adjust order and scope in roadmap or ADR.

---

## Specific Features to Explore

<!-- Add detailed feature ideas, UX considerations, edge cases -->

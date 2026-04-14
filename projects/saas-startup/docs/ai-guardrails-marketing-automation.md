# AI Guardrails for Marketing Automation

> **Purpose:** Reference architecture for safe AI-assisted marketing and sales workflow creation  
> **Scope:** Content generation, campaign creation, sales pipeline setup, dashboard generation  
> **Input starting point:** Product image/video + product description  
> **Audience:** Product, engineering, AI, compliance, operations  
> **Last Updated:** March 24, 2026

---

## 1. Problem Statement

We want a non-technical user to generate marketing content, campaigns, pipeline setup, and performance dashboards from minimal input (image/video + description), while ensuring the AI output is useful, compliant, and aligned with brand and business policy.

The core risk is that free-form generation can produce:

- Hallucinated product claims or unsupported performance promises
- Brand-inconsistent messaging
- Legally risky language (regulated industries, guarantees, comparative claims)
- Privacy or data handling violations
- Incorrect business actions (wrong audience, wrong channel, wrong KPI setup)

Prompt engineering helps, but is not enough for production-grade behavior.

---

## 2. Guardrail Principles

1. **Layered controls over single control.** No single prompt or model call is trusted on its own.
2. **Generate in structure, render in text.** AI returns schema-validated objects first; final copy is derived from approved fields.
3. **Ground before generation.** AI must use approved product facts and policy documents.
4. **Risk-based automation.** Low-risk outputs can auto-publish; high-risk outputs require review.
5. **Separation of generation and execution.** "Draft" and "publish/send/create" are distinct permissions and actions.
6. **Auditability by default.** Every generation, edit, policy check, and publish action is traceable.

---

## 3. Why Prompt Engineering Alone Is Not Enough

Prompting is necessary but insufficient because:

- Model behavior shifts across provider/model versions
- Prompts cannot reliably enforce hard constraints in all edge cases
- Compliance requirements need deterministic checks and explainability
- Business operations need recoverability and audit logs, not just best-effort text quality

**Conclusion:** Prompting is the first layer, not the safety strategy.

---

## 4. Recommended Guardrail Architecture

### 4.1 High-level pipeline

1. **Ingest and extract facts**
   - Parse product description
   - Use vision/video models to extract visible attributes
   - Build a normalized `ProductFacts` record
2. **Grounding context assembly**
   - Brand voice guide
   - Legal/compliance snippets
   - Channel policy constraints
   - Pricing/inventory/truth-source fields
3. **Plan generation (structured)**
   - Generate campaign brief JSON (audience, channels, goals, budget assumptions)
4. **Asset generation (structured-first)**
   - Generate ad variants, email sequences, landing page sections, pipeline stages, dashboard specs
5. **Validation and policy checks**
   - Schema validation
   - Rule engine checks
   - LLM judge checks for nuanced policy or tone violations
6. **Auto-repair loop**
   - Feed violations back for bounded retries
7. **Risk scoring and approval routing**
   - Route high-risk outputs to human review
8. **Execution**
   - Publish/send/create only after gate passes
9. **Monitoring and feedback**
   - Track quality, compliance incidents, and campaign performance for continuous improvement

### 4.2 Control layers

- **Layer A: Prompt constraints**
  - Role, objective, forbidden claims, brand voice, output format
- **Layer B: Structured output contracts**
  - JSON schema for every artifact type
- **Layer C: Deterministic validators**
  - Regex/rules for banned terms, guarantees, missing disclaimers, unsupported superlatives
- **Layer D: Semantic policy evaluator**
  - Secondary model classifies risk categories and explains violations
- **Layer E: Human approval workflow**
  - Required for high-impact channels or high-risk content
- **Layer F: Action authorization**
  - RBAC + explicit user confirmations

---

## 5. Core Data Contracts (Suggested)

Use strict schemas for all generated artifacts:

- `ProductFacts`
- `CampaignBrief`
- `ContentAsset`
- `SalesPipelineTemplate`
- `DashboardSpec`
- `ComplianceCheckResult`
- `RiskScore`
- `ApprovalRecord`

### Example: `CampaignBrief` (conceptual fields)

- Objective (`lead_gen`, `sales_conversion`, `awareness`, ...)
- Target persona(s) and exclusions
- Core value proposition tied to `ProductFacts`
- Channel plan (email, paid social, landing page, etc.)
- Budget assumptions and constraints
- KPI definitions and expected measurement window
- Required disclaimers by channel/region

---

## 6. Guardrails by Feature Area

### 6.1 Marketing content generation

- Enforce claim-evidence linkage: every claim must map to a known product fact
- Block prohibited language classes (guarantees, medical/financial claims, unverified comparisons)
- Constrain reading level, tone, and prohibited style patterns per brand
- Require references to approved CTA patterns

### 6.2 Campaign generation

- Validate audience targeting against tenant consent and suppression rules
- Prevent invalid channel combinations for regulatory region
- Apply spend, frequency, and contact-pressure limits
- Require fallbacks if required fields are missing (no silent assumptions)

### 6.3 Sales pipeline generation

- Restrict stage templates to approved sales methodologies
- Prevent stages that imply legal commitment language without review
- Enforce required stage fields (owner, SLA, next-step date)
- Ensure stage transitions can be audited

### 6.4 Dashboard generation

- Restrict metrics to approved semantic layer definitions
- Block fabricated KPIs or derived metrics without formula source
- Require clear attribution window and data freshness metadata
- Validate chart recommendations against data type and cardinality

---

## 7. Human-in-the-Loop Design

Use risk tiers:

- **Tier 0 (low risk):** internal drafts, no external distribution -> auto-approve
- **Tier 1 (medium):** outbound but low legal risk -> optional review or spot checks
- **Tier 2 (high):** paid ads, legal-sensitive language, regulated domain -> mandatory approval

Approval UX should show:

- Generated output
- Violations and resolved warnings
- Evidence links for key claims
- Diff vs previous approved version
- One-click approve/reject/request-regenerate

---

## 8. Evaluation and Monitoring

### 8.1 Offline evaluation

- Maintain a red-team suite of adversarial prompts
- Track pass/fail by policy category
- Score tone consistency, factuality, and conversion quality proxy

### 8.2 Online monitoring

- Violation rate by artifact/channel
- Regeneration loop count (signals prompt/policy drift)
- Human override rate
- Time-to-approval
- Outcome metrics (CTR, open rate, conversion, pipeline velocity) with quality gates

### 8.3 Incident handling

- Immediate kill switch per tenant/channel
- Rollback to last approved template set
- Post-incident root cause tags (prompt, retrieval, schema, policy engine, reviewer gap)

---

## 9. Practical Techniques Beyond Prompt Engineering

- **RAG with policy docs and brand assets** for factual grounding
- **Schema-constrained decoding** to force valid structures
- **Multi-step generate -> critique -> revise** pipelines
- **Policy-as-code engine** for deterministic enforceable constraints
- **Secondary LLM judges** for semantic checks not covered by simple rules
- **Confidence/risk scoring** to drive approval routing
- **Model routing** (cheaper model for draft, stronger model for repair/high risk)
- **Limited fine-tuning/adapters** for brand voice consistency only after baseline stack is stable

---

## 10. Implementation Roadmap (Startup-Friendly)

### Phase 1: MVP Guardrails (ship quickly)

- Strong system prompts + strict JSON schemas
- Basic policy rules (banned claims, required disclaimers)
- Manual approval before publish
- Full audit logs

### Phase 2: Production baseline

- Add grounding (RAG over approved docs)
- Add semantic policy checker model
- Add risk scoring and tiered approvals
- Add offline red-team evaluation suite

### Phase 3: Optimization and scale

- Auto-repair loops with bounded retries
- Channel-specific policy packs by region
- Adaptive routing and quality/cost optimization
- Continuous policy tuning from incident and reviewer feedback

---

## 11. Integration Notes for This Platform

Given the architecture (modular monolith + workers + unified AI tools), implement guardrails as shared cross-cutting services:

- **AI Gateway:** provider abstraction, model routing, prompt templates
- **Policy Service:** deterministic + semantic checks
- **Approval Service:** routing, reviewer assignment, decision logs
- **Evidence Service:** claim-to-fact mapping and citation tracking
- **Risk Engine:** score aggregation and gating decisions

Worker jobs should execute long-running generation/check loops asynchronously, with explicit tenant context and idempotent job handlers.

---

## 12. Anti-Patterns to Avoid

- Letting AI directly publish/send without explicit gate
- Allowing free-text outputs to bypass schema validation
- Relying on one giant prompt as the only control
- Mixing test/demo prompts with production prompts
- No provenance for generated assets
- No human fallback path during model or policy failures

---

## 13. Minimal Go/No-Go Checklist

Before enabling auto-generated marketing workflows:

- [ ] Structured schemas are enforced for all generated artifact types
- [ ] Deterministic policy checks are active
- [ ] Semantic checks are active for nuanced policy classes
- [ ] Risk tiers and approval routes are configured
- [ ] Audit logs and trace IDs are available end-to-end
- [ ] Kill switch exists per tenant/channel
- [ ] Red-team suite run is passing threshold
- [ ] Clear ownership for policy updates and incidents

---

## 14. Related Docs

- `docs/architecture-overview.md`
- `docs/product-scopes/ai-agents.md`
- `docs/product-scopes/content-marketing.md`
- `docs/external-integrations.md`


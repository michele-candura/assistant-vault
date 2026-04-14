# AI Coding & Prompting Guidelines

> **Status:** Planning Doc (high-level)  
> **Audience:** Founder, future developers, AI-assisted coding workflows  
> **Goal:** Prevent “vibe coding” from turning the product into an unmaintainable mess.

---

## 1. Why This Document Exists

This product has a **very large scope** (many modules, strong multi-tenancy, cross-cutting concerns, AI features, and regulated data). AI can accelerate delivery, but without guardrails it will:

- create inconsistent patterns across modules
- break architectural boundaries
- ignore tenancy/RBAC constraints
- produce code that works locally but damages long-term maintainability

This document defines **how AI should be used** when building the product.

---

## 2. Core Principles

### 2.1 Architecture first, code second

AI should not invent architecture on the fly.

- Before implementing a feature, the relevant product/architecture docs must exist or be updated.
- If the prompt is vague, AI should first help clarify scope, boundaries, and risks.
- AI should prefer extending existing patterns rather than inventing new ones.

### 2.2 Small, scoped changes

Recent best practice for large codebases is to avoid huge “do everything” prompts.

- One objective per session/task when possible.
- Prefer small, reviewable diffs over large rewrites.
- Ask AI to work module-by-module, not “across the whole platform” unless strictly necessary.

### 2.3 Context must be explicit

AI works best when context is treated like an API: minimal, precise, and versioned.

- Prompts should reference the relevant docs/files/symbols explicitly.
- Prefer giving AI exact files and goals rather than asking it to “figure out the whole system”.
- If a task spans multiple modules, state why and list them.

### 2.4 Proof over confidence

AI suggestions are not trusted just because they sound good.

- Require code references, rationale, and risks for architectural changes.
- Require tests, validation steps, or explicit manual verification plans for code changes.
- Prefer compile-time / lint / test proof over “this should work”.

---

## 3. Project-Specific Rules AI Must Follow

### 3.1 Respect module boundaries

AI must follow the module-boundary strategy:

- No cross-module imports of internals.
- Cross-module communication should happen via:
  - durable domain events, or
  - small public interfaces for synchronous request-scoped needs.
- Do not create “god services” that centralize unrelated domain logic.

See `docs/module-boundaries-strategy.md`.

### 3.2 Never weaken tenant isolation

AI must treat multi-tenancy as a non-negotiable constraint.

- Every data access path must remain tenant-scoped.
- Worker jobs and background flows must carry tenant context.
- AI should not suggest shortcuts that bypass tenancy checks.

See `docs/architecture-overview.md`.

### 3.3 RBAC and permissions are cross-cutting

AI must assume permissions are difficult and important.

- Never assume a user can access cross-tenant or cross-entity data.
- AI-generated code must preserve RBAC constraints.
- The platform AI agent must inherit the invoking user’s permissions; no privilege escalation.

### 3.4 Prefer durable, idempotent workflows

Because cross-module reactions use durable events and jobs:

- Event handlers should be idempotent.
- AI should not design fragile synchronous chains when async side effects are more appropriate.
- Retries, failure modes, and logging must be considered.

See `docs/resilience-strategy.md`.

### 3.5 Don’t over-engineer early

AI often jumps too quickly to “platform” or “framework” solutions.

- Avoid introducing new infrastructure without clear need.
- Prefer generic, simple patterns unless a more advanced solution is justified.
- Defer optional abstractions (plugins, complex builders, extra services) until real use cases exist.

---

## 4. Prompting Best Practices for This Project

These are the preferred prompt patterns when using AI on this codebase.

### 4.1 Good prompt structure

Each prompt should try to include:

1. **Goal** — what problem are we solving?
2. **Scope** — which module(s) are involved?
3. **Constraints** — tenancy, RBAC, performance, product-phase limitations
4. **Expected output** — explanation, plan, code change, doc update, test, review, etc.
5. **Non-goals** — what should not be changed?

Example:

> Add customer segmentation filters to CRM list views. Scope: CRM only. Preserve tenant isolation and existing API conventions. Do not redesign shared filtering across all modules yet. Update docs if the scope changes.

### 4.2 Bad prompt patterns to avoid

- “Build the whole module”
- “Refactor this entire codebase”
- “Make this enterprise-ready”
- “Improve performance everywhere”
- “Figure out the best architecture and implement it”

These are too broad and produce low-quality, unreviewable output.

### 4.3 Ask AI to explain trade-offs

For architecture/product decisions, prefer prompts like:

- “Compare 2–3 options and explain trade-offs for *this project*”
- “What assumptions are you making?”
- “What are the risks if we choose option A?”

This reduces shallow, overconfident suggestions.

---

## 5. Recommended AI Workflow (Large Codebase Best Practice)

Recent guidance for AI in large codebases converges on a staged workflow:

### Stage 1 — Understand

- Read the relevant docs/files first.
- Summarize the current behavior and constraints.
- Confirm assumptions before editing.

### Stage 2 — Plan

- Propose the smallest safe change.
- Identify touched files, interfaces, and risks.
- Call out any boundary/security concerns.

### Stage 3 — Implement

- Make focused edits.
- Keep changes aligned with existing patterns.
- Avoid opportunistic unrelated refactors.

### Stage 4 — Validate

- Run lints/tests/build where relevant.
- Check edge cases and affected boundaries.
- Report limitations clearly if full validation was not possible.

### Stage 5 — Review

- Summarize what changed and why.
- Mention residual risks or follow-up work.

---

## 6. Review Guardrails for AI-Generated Changes

All non-trivial AI-generated changes should be reviewed with extra attention to:

- **Boundary violations** — cross-module imports or leaked internals
- **Tenant isolation** — missing `tenant_id` / context propagation
- **Permissions** — RBAC assumptions, portal/agency access, AI privilege escalation
- **Async correctness** — retries, idempotency, failure handling
- **Data model drift** — changes that conflict with product/architecture docs

Risk-tiered review is recommended:

- **Low risk**: copy changes, small UI tweaks, docs
- **Medium risk**: single-module CRUD/API changes
- **High risk**: auth, tenancy, RBAC, billing, AI tool execution, ERP/HR workflows, cross-module events

High-risk changes should get stricter review and stronger validation.

---

## 7. Persistent Instructions for AI Tools

Recent best practice for large codebases is to keep a **persistent AI instruction file** at repo root (for example `AGENTS.md`, `CLAUDE.md`, or equivalent tool-specific file).

For this project, that file should eventually contain:

- repo map / module map
- architecture rules
- commands for lint/test/build
- do-not-touch zones
- coding standards
- security/compliance notes
- review checklist

This document (`docs/ai-coding-guidelines.md`) is the planning source; a root-level AI instructions file can be created later once implementation starts.

---

## 8. What AI Is Best Used For Here

### Good uses

- generating boilerplate within existing patterns
- adding CRUD endpoints/forms in a single module
- writing tests around well-defined behavior
- comparing implementation options
- extracting repetitive code carefully
- drafting docs and ADRs

### Risky uses

- inventing architecture without constraints
- broad cross-module refactors
- security-sensitive changes without review
- RBAC/tenant logic without explicit checks
- autonomous “cleanup” or “simplify” prompts across the repo

---

## 9. Open Decisions (to revisit later)

- When to create root-level AI instruction files (`AGENTS.md` or tool-specific equivalents)
- Whether to enforce architecture rules with CI tests
- Whether to require stronger review gates for high-risk AI-generated diffs


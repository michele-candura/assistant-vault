# Module Boundaries & Cross-Module Communication (High-Level Strategy)

> **Status:** Planning Doc (high-level)  
> **Audience:** Developers, architecture reference  
> **Goal:** Prevent “dependency spaghetti” as modules grow, without locking into specific technology choices.

---

## 1. The Problem We Are Avoiding

With many modules, the default failure mode is **uncontrolled dependencies**:

- Modules import each other’s internals (entities, repositories, types)
- “Quick” direct calls become permanent coupling
- Circular dependencies appear
- Changes in one module break others
- Testing becomes expensive because everything depends on everything

This is a **governance and boundary** problem more than a transport problem (events vs HTTP vs direct calls).

---

## 2. Core Rules (Industry Standard in Modular Monoliths)

### 2.1 Dependency rules

- **No internal imports across modules**: Modules must not import other modules’ internal code (DB models, repositories, internal services).
- **Only depend on public contracts**:
  - **Domain events** (publish/subscribe), and/or
  - A **small public interface** (service API) when synchronous results are truly required.
- **Cross-cutting services** (e.g. auth/RBAC, notifications, file storage, scheduling/shifts) can be called by many modules because they are utilities.

### 2.2 Direct calls vs events

- **Direct calls**: only when the API response depends on the other module’s result (request-scoped).
- **Events**: for side effects and multi-consumer reactions (async, decoupled).

### 2.3 Orchestration vs choreography

- **Choreography** (events) is preferred for cross-module reactions.
- **Orchestration** is used when a user flow truly requires multi-step coordination (e.g., a workflow engine, or a thin application-layer orchestrator).
- Orchestrators must remain **thin** and avoid owning domain logic.

---

## 3. Practical Guardrails

### 3.1 Contract design

- Define event names and payload schemas (stable, versionable).
- Avoid leaking internal DB structure in events.
- Consider an anti-corruption layer (mapping) when consuming foreign events or APIs.

### 3.2 Preventing regressions

- Add **architecture tests** (CI checks) that fail when module dependency rules are violated.
- Enforce a shared convention: every cross-module dependency must be either:
  - a published event, or
  - an explicit entry in a module’s public interface.

---

## 4. How This Fits Our Current Architecture

- Durable cross-module events are delivered via the job/event system (BullMQ + Redis).
- Automations and AI agent tool-calls should operate through public contracts and permission checks, not through internal imports.
- RBAC is cross-cutting: module tool APIs must be role-aware; the AI agent must inherit the invoking user’s permissions.

---

## 5. Open Decisions (to revisit later)

- Exact dependency graph rules (which modules may call which, and in what direction)
- Event versioning approach
- Where orchestration lives for complex multi-step flows (workflow engine vs application orchestrator)


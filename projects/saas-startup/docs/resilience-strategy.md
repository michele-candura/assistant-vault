# Resilience & Error Handling Strategy

> **Status:** Planning Doc (high-level)  
> **Audience:** Developers, architecture reference  
> **Goal:** Define how the platform behaves when things fail (without locking into specific vendors).

---

## 1. Principles

- **Assume partial failure**: external APIs, Redis, network, and background workers can fail at any time.
- **Degrade gracefully**: keep core user flows working when possible; fail fast when not.
- **Make failures observable**: every failure path should produce actionable logs/metrics.
- **Prefer idempotency over exactly-once**: with at-least-once delivery, handlers must tolerate retries.

---

## 2. Failure Domains and Expected Behavior

### 2.1 External APIs (payments, AI, email, ads, calendars, messaging)

- **Strategy**
  - Timeouts, retries with backoff, and circuit breakers.
  - Rate-limit awareness; queue work when possible.
  - Feature-level degradation: if an external dependency is down, the core platform still works but the dependent feature is delayed or disabled.
- **User experience**
  - Clear status in UI (e.g., “sync pending”, “provider unavailable”, “retry scheduled”).
  - Avoid silent failures.

### 2.2 Redis / BullMQ (durable events + jobs)

- **Strategy**
  - Treat Redis as critical infrastructure. If unavailable, events/jobs cannot be enqueued.
  - API should either:
    - **Fail fast** for flows that require async side effects, or
    - Continue only for safe operations that don’t rely on background processing.
  - Use DLQ (dead-letter queue) patterns for jobs that exceed retries.
- **Operations**
  - Surface DLQ counts and failed job payloads in admin tooling.
  - Alerting on sustained failure rates.

### 2.3 Database

- **Strategy**
  - Connection pool limits, timeouts, and backpressure.
  - For heavy reporting endpoints, prefer cached / scheduled computations or read replicas (when needed).
  - RLS failures should be treated as security-critical (fail closed).

---

## 3. Idempotency & Retries (Industry Standard Pattern)

Because events and jobs are at-least-once:

- **Handlers must be idempotent**
  - Use a stable idempotency key per event/job (e.g., `event_id`).
  - Record processed keys to prevent duplicates (or use unique constraints).
- **Retries**
  - Safe to retry transient failures (network, timeouts).
  - Do not retry indefinitely; route to DLQ after max attempts.
- **Compensation**
  - For workflows that call external systems, prefer “compensating actions” rather than distributed transactions.

---

## 4. Error Taxonomy (Simplified)

- **User errors**: validation, permissions, missing resources → 4xx; no retries.
- **Transient system errors**: timeouts, 429s, temporary provider outages → retry/backoff; queue if async.
- **Permanent system errors**: misconfiguration, schema mismatch → fail fast; require operator intervention.

---

## 5. Observability Requirements (High-Level)

Without choosing specific tools, define requirements:

- **Structured logs** with `tenant_id`, `entity_id` (where relevant), `request_id`, `job_id`, and `event_id`.
- **Metrics**
  - API latency/error rates
  - Job/event throughput, retries, DLQ counts
  - External API call latency/error rates
- **Tracing** (optional later): end-to-end trace across request → event → job handlers.

---

## 6. Open Decisions (to revisit later)

- Retry policies per integration category (AI vs payments vs ads)
- DLQ retention and replay workflows
- Which endpoints can operate in “degraded mode” if Redis is down
- SLOs for core flows (CRM CRUD, invoice creation, scheduling, etc.)


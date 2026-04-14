# Logging & Observability (High-Level)

> **Status:** Planning Doc (high-level)  
> **Audience:** Developers, security/compliance planning  
> **Goal:** Define what we need to log/measure and why, without locking into specific tooling.

---

## 1. Goals

- **Debugging**: quickly answer “what happened?” and “why did it fail?”
- **Compliance & audit**: trace who changed what (especially for ERP/HR)
- **Security**: detect suspicious behavior and investigate incidents
- **Reliability**: measure error rates, retries, and failure hotspots
- **Product insights**: understand usage patterns (at a high level)

---

## 2. Log Types

### 2.1 Application logs

- Structured logs for API and worker execution (info/warn/error)
- Include errors from external API calls (redacting secrets)

### 2.2 Audit logs (business-level)

- Append-only record of business mutations (create/update/delete, approvals, payments)
- Must support compliance queries (who/when/what changed)

### 2.3 Security logs

- Authentication events (login, logout, failed attempts, token revocation)
- Permission failures and suspicious access patterns

### 2.4 Operational logs (jobs/events)

- Domain event publication, job enqueue/dequeue, retries, dead-letter routing
- Worker crashes and restart loops

---

## 3. Required Fields (Minimum Correlation)

Every log entry should include (when available):

- `tenant_id`
- `entity_id` (module-specific; optional)
- `user_id` (or “system/worker” identity)
- `request_id` (API)
- `job_id` and `event_id` (workers/events)
- `module` and `action` (what component did what)
- Timestamp and duration

---

## 4. GDPR & Data Handling

- Do not log sensitive payloads by default (HR/finance/PII).
- Define retention windows for each log type.
- Support right-to-erasure policies (often via anonymization for logs/audit).

---

## 5. Metrics (High-Level Requirements)

- API latency and error rates (by endpoint/module)
- Worker throughput, retries, and dead-letter queue counts
- External API latency/error rates (by integration)
- Cache hit/miss rates (tenant-scoped)

---

## 6. Tracing (Optional Later)

End-to-end correlation across:

API request → domain event → background jobs → external API calls → side effects

This can be implemented later once core flows stabilize.

---

## 7. Open Decisions (to revisit later)

- Which business events require audit entries vs “operational only”
- Async vs sync audit writes for hot paths
- Partitioning/archival strategy for audit log volume
- Admin UX for failed jobs, DLQ inspection, and replay


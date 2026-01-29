---
description: Production readiness with built-in review
---

# /ship

## Purpose

Comprehensive pre-release validation that combines stability patterns, observability coverage, and security checking. Built-in production review runs automatically. Run this before any production deployment.

## Input

- Feature name or scope of release
- Or: "full app" for complete check

## Workflow Overview

```
STABILITY AUDIT → OBSERVABILITY AUDIT → SECURITY SCAN → PRODUCTION REVIEW → VERDICT
        │                │                    │                │              │
   Timeouts,        Tracing,           Input validation,   production-   READY or
   retries,         metrics,           secrets,            reviewer      BLOCKED
   circuit breakers alerts             auth               (automatic)
```

---

## Embedded Knowledge: Stability Patterns

### Required Patterns

| Pattern | What | Check |
|---------|------|-------|
| **Timeouts** | Never wait forever | All external calls have timeout |
| **Circuit Breaker** | Stop calling failing services | Flaky dependencies have breakers |
| **Retry with Backoff** | Retry transient failures | Exponential backoff, jitter, max retries |
| **Bulkhead** | Isolate failures | Thread pools, connection limits |
| **Rate Limiting** | Protect downstream | Request limits configured |
| **Fallback** | Graceful degradation | Degraded service paths exist |

### Timeout Guidelines

```rust
// Configuration pattern
struct ClientConfig {
    connect_timeout: Duration,  // 1-5s typical
    read_timeout: Duration,     // 5-30s typical
    total_timeout: Duration,    // 30-60s typical
}
```

### Circuit Breaker States

```
CLOSED (normal) → failure count > threshold → OPEN (reject)
                                                    ↓
                                              timeout expires
                                                    ↓
CLOSED ←── success ←── HALF-OPEN (test calls)
```

---

## Embedded Knowledge: Observability Checklist

### Structured Logging

```rust
// REQUIRED: Structured key-value pairs
info!(
    user_id = %user_id,
    order_id = %order_id,
    amount_cents = amount,
    "order_created"
);

// NOT ACCEPTABLE: String interpolation
info!("User {} created order {} for ${}", user_id, order_id, amount);
```

### Correlation IDs

Every request must have a correlation ID that flows through:
- Entry point → all function calls → response
- All log entries for a request share the same ID
- Enables tracing a single request through logs

### State Transitions

```rust
info!(
    from_state = ?old_state,
    to_state = ?new_state,
    trigger = ?event,
    "state_transition"
);
```

### Error Context

```rust
error!(
    request_id = %request_id,
    user_id = %user_id,
    operation = "create_order",
    error_type = "validation",
    error_message = %e.to_string(),
    "operation_failed"
);
```

---

## Embedded Knowledge: Release Checklist

### Pre-Release Verification

1. **Stability Patterns** — Timeouts, retries, circuit breakers, bulkheads
2. **Observability** — Logging, tracing, metrics, health checks
3. **Error Handling** — Errors categorized, no sensitive data leaked
4. **Security** — Secrets not in code, input validated, auth checked
5. **Performance** — Load tested, resource limits set
6. **Rollback** — One-command rollback possible, feature flags work

### Critical Questions

1. **What could go wrong?** — List failure modes, document mitigations
2. **How will we know it's broken?** — Monitoring coverage, alert thresholds
3. **How will we fix it fast?** — Rollback procedure, feature flags
4. **What's the blast radius?** — Which users affected, isolation in place

---

## Phases

### Phase 1: Stability Audit

**Entry**: Feature/release scope provided
**Actions**:
1. Identify all external calls (HTTP, database, file I/O)
2. Check each for timeout configuration
3. Check retry logic and backoff
4. Identify circuit breaker usage
5. Check resource limits

**Output**:
```markdown
## Stability Audit

| Pattern | Status | Location | Notes |
|---------|--------|----------|-------|
| Timeouts on external calls | ✅/❌/⚠️ | file:line | [details] |
| Retry with backoff | ✅/❌/⚠️ | file:line | [details] |
| Circuit breakers | ✅/❌/⚠️ | file:line | [details] |
| Graceful degradation | ✅/❌/⚠️ | - | [details] |
| Resource limits | ✅/❌/⚠️ | config | [details] |

### Missing (BLOCKERS)
1. [What's missing and where to add it]
```

---

### Phase 2: Observability Audit

**Entry**: Stability audit complete
**Actions**:
1. Check for correlation IDs in request handlers
2. Verify error logging with context
3. Check state transition logging
4. Verify key metrics are emitted
5. Check health check endpoints

**Output**:
```markdown
## Observability Audit

| Aspect | Status | Location | Notes |
|--------|--------|----------|-------|
| Correlation IDs | ✅/❌/⚠️ | handlers | [details] |
| Error logging with context | ✅/❌/⚠️ | error paths | [details] |
| State transition logs | ✅/❌/⚠️ | state machines | [details] |
| Request tracing | ✅/❌/⚠️ | #[instrument] | [details] |
| Health check endpoints | ✅/❌/⚠️ | /health | [details] |

### Missing (BLOCKERS)
1. [What's missing and where to add it]
```

---

### Phase 3: Security Scan

**Entry**: Observability audit complete
**Actions**:
1. Check for secrets in code/config
2. Verify input validation at boundaries
3. Check authentication on protected routes
4. Verify no sensitive data in logs/errors

**Output**:
```markdown
## Security Scan

| Check | Status | Location | Notes |
|-------|--------|----------|-------|
| No exposed secrets | ✅/❌ | - | [details] |
| Input validation | ✅/❌/⚠️ | handlers | [details] |
| Auth checks | ✅/❌/⚠️ | routes | [details] |
| No PII in logs | ✅/❌/⚠️ | logging | [details] |

### Issues (BLOCKERS)
1. [Security issues found]
```

---

### Phase 4: Production Review (AUTOMATIC)

**Entry**: All audits complete
**Actions**:
1. Invoke `production-reviewer` agent automatically
2. Review all findings
3. Assess rollback readiness
4. Generate verdict

**Built-in Review Checklist** (production-reviewer checks these):
- [ ] All error paths logged
- [ ] Critical functions instrumented
- [ ] No hardcoded secrets
- [ ] Graceful degradation exists
- [ ] Rollback plan viable
- [ ] Blast radius contained

**Output**:
```markdown
## Production Review

### Stability: [PASS | PARTIAL | FAIL]
- Error handling: [Complete | Partial | Missing]
- Timeouts: [Configured | Missing]
- Circuit breakers: [In place | N/A | Missing]

### Observability: [PASS | PARTIAL | FAIL]
- Tracing: [Complete | Partial | Missing]
- Metrics: [Complete | Partial | Missing]
- Alerts: [Configured | Not configured]

### Security: [PASS | FAIL]
- Secrets management: [OK | Issue]
- Input validation: [Complete | Gaps]
- Auth coverage: [Complete | Gaps]

### Rollback Readiness
- **Detection**: [How to detect problems]
- **Trigger**: [When to rollback]
- **Process**: [Steps to rollback]
- **Verification**: [How to verify rollback worked]

### Verdict: [READY | BLOCKED]
```

---

### Phase 5: Final Verdict

**Entry**: Production review complete
**Actions**:
1. Summarize all findings
2. Categorize: Blockers, Warnings, Notes
3. Provide verdict

**Output**:
```markdown
# Ship Check: [Feature/Release Name]

## Overall Status: [READY | BLOCKED | WARNING]

## Summary

| Area | Status |
|------|--------|
| Stability | ✅/⚠️/❌ |
| Observability | ✅/⚠️/❌ |
| Security | ✅/❌ |
| Rollback Ready | ✅/❌ |

## Blockers (Must Fix Before Ship)
1. [Issue]: [Location] — [What to do]

## Warnings (Should Fix)
1. [Issue]: [Location] — [What to do]

## Notes (Track for Later)
1. [Issue]: [Ticket/issue link]

## Next Steps
[If READY: proceed with deployment]
[If BLOCKED: fix blockers, re-run /ship]
```

---

## Checklist

Before marking READY:
- [ ] All blockers resolved
- [ ] Warnings documented/ticketed
- [ ] Rollback plan reviewed
- [ ] On-call aware of deployment (if applicable)

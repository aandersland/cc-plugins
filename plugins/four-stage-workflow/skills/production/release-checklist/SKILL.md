---
name: release-checklist
description: Pre-release verification checklist ensuring systems can handle real-world conditions including failures and load.
---

# Release Checklist

## Source
- Book: Release It! Design and Deploy Production-Ready Software
- Author: Michael Nygard
- Key Concept: Production readiness is more than "it works"

## Core Principle
Before release, verify the system can handle real-world conditions: failures, load, and the unexpected.

## Pre-Release Checklist

### 1. Stability Patterns

| Pattern | Status | Notes |
|---------|--------|-------|
| Timeouts on all external calls | [ ] | Default: 5s connect, 30s read |
| Circuit breakers on dependencies | [ ] | Threshold, timeout configured |
| Retry with backoff | [ ] | Max retries, jitter enabled |
| Bulkheads for isolation | [ ] | Thread pools, connection limits |
| Rate limiting | [ ] | Protect downstream services |
| Fallback strategies | [ ] | Graceful degradation paths |

### 2. Observability

| Aspect | Status | Notes |
|--------|--------|-------|
| Structured logging | [ ] | JSON format, key-value pairs |
| Correlation IDs | [ ] | Flow through all operations |
| Error context | [ ] | Full state captured on errors |
| Request tracing | [ ] | Start, end, duration logged |
| Health check endpoints | [ ] | Dependency health included |
| Metrics exposed | [ ] | CPU, memory, custom business metrics |

### 3. Error Handling

| Aspect | Status | Notes |
|--------|--------|-------|
| Errors categorized | [ ] | Recoverable vs fatal |
| Error messages actionable | [ ] | What, why, how to fix |
| Graceful degradation | [ ] | Partial functionality on failure |
| No sensitive data in errors | [ ] | Redact PII, secrets |
| Error reporting configured | [ ] | Sentry, Rollbar, etc. |

### 4. Security

| Aspect | Status | Notes |
|--------|--------|-------|
| Secrets not in code | [ ] | Environment variables or vault |
| Input validation | [ ] | All user input sanitized |
| Authentication working | [ ] | Token validation, session handling |
| Authorization checked | [ ] | Permission checks on operations |
| Dependencies scanned | [ ] | No known vulnerabilities |
| HTTPS enforced | [ ] | TLS for all connections |

### 5. Performance

| Aspect | Status | Notes |
|--------|--------|-------|
| Load tested | [ ] | Expected traffic handled |
| Resource limits set | [ ] | Memory, CPU, connections |
| Database queries optimized | [ ] | Indexes, query plans reviewed |
| Caching configured | [ ] | Appropriate TTLs |
| Async operations | [ ] | Long tasks not blocking |

### 6. Data

| Aspect | Status | Notes |
|--------|--------|-------|
| Migrations tested | [ ] | Rollback plan exists |
| Backups configured | [ ] | Verified restore works |
| Data validation | [ ] | Corrupt data handling |
| GDPR compliance | [ ] | Data deletion, export capability |

### 7. Deployment

| Aspect | Status | Notes |
|--------|--------|-------|
| Rollback plan | [ ] | One-command rollback |
| Feature flags | [ ] | Critical features toggleable |
| Blue-green or canary | [ ] | Gradual rollout possible |
| Configuration separate | [ ] | No code changes for config |
| Database migrations | [ ] | Backward compatible |

### 8. Monitoring & Alerting

| Aspect | Status | Notes |
|--------|--------|-------|
| Dashboards created | [ ] | Key metrics visible |
| Alerts configured | [ ] | Error rates, latency, availability |
| On-call defined | [ ] | Who gets paged |
| Runbooks written | [ ] | Common issues documented |

## Critical Questions

Before each release, answer:

1. **What could go wrong?**
   - List failure modes
   - Document mitigations

2. **How will we know it's broken?**
   - Monitoring coverage
   - Alert thresholds

3. **How will we fix it fast?**
   - Rollback procedure
   - Feature flags

4. **What's the blast radius?**
   - Which users affected
   - Isolation in place

## Release Confidence Levels

| Level | Criteria | Action |
|-------|----------|--------|
| Green | All checks pass, low risk | Ship |
| Yellow | Minor gaps, manageable risk | Ship with monitoring |
| Red | Critical gaps, high risk | Do not ship |

## Post-Release

| Action | Timing | Notes |
|--------|--------|-------|
| Monitor dashboards | 15 min | Watch for anomalies |
| Check error rates | 1 hour | Compare to baseline |
| Review logs | 2 hours | Look for new errors |
| Canary validation | 24 hours | Full traffic safe |
| Retrospective | 1 week | What to improve |

## Anti-Patterns

- **YOLO deploys:** No checklist, no validation
- **Friday deploys:** No support during issues
- **Big bang releases:** All or nothing
- **No rollback:** Can't quickly revert
- **Deploy and forget:** No post-release monitoring

## Claude Code Integration
**Command:** /production-ready
**Input:** Feature or codebase scope
**Output:** Checklist evaluation with status and gaps

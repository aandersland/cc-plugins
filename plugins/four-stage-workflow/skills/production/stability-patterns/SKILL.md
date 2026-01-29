---
name: stability-patterns
description: Patterns for graceful failure handling, failure containment, and automatic recovery in production systems.
---

# Stability Patterns

## Source
- Book: Release It! Design and Deploy Production-Ready Software
- Author: Michael Nygard
- Key Concept: Stability patterns prevent cascade failures

## Core Principle
Systems should fail gracefully, contain failures, and recover automatically. Every integration point is a risk.

## Decision Framework

```
For each external dependency:
├── What if it's slow? → Timeout
├── What if it's down? → Circuit breaker
├── What if it fails? → Retry with backoff
├── What if it overloads us? → Bulkhead
└── What if we overload it? → Rate limiting
```

## Patterns

### Pattern 1: Timeouts

**What:** Never wait forever for a response
**When:** Every external call (HTTP, database, file I/O)

```rust
// DANGEROUS: No timeout
let response = client.get(url).await?;

// SAFE: Always timeout
let response = tokio::time::timeout(
    Duration::from_secs(5),
    client.get(url)
).await??;

// Configuration pattern
struct ClientConfig {
    connect_timeout: Duration,  // 1-5s typical
    read_timeout: Duration,     // 5-30s typical
    total_timeout: Duration,    // 30-60s typical
}
```

### Pattern 2: Circuit Breaker

**What:** Stop calling a failing service temporarily
**When:** Repeated failures to same dependency

```
States:
┌─────────┐  failure count > threshold  ┌──────────┐
│ CLOSED  │ ─────────────────────────→ │   OPEN   │
│(normal) │                             │ (reject) │
└─────────┘                             └──────────┘
     ↑                                        │
     │                                        │ timeout
     │        ┌─────────────┐                │
     └─────── │ HALF-OPEN   │ ←──────────────┘
    success   │(test calls) │
              └─────────────┘
```

```rust
use circuit_breaker::CircuitBreaker;

let breaker = CircuitBreaker::new()
    .failure_threshold(5)     // Open after 5 failures
    .success_threshold(2)     // Close after 2 successes
    .timeout(Duration::from_secs(30)); // Try again after 30s

let result = breaker.call(|| {
    external_service.request()
}).await;

match result {
    Ok(data) => process(data),
    Err(CircuitBreakerError::Open) => use_fallback(),
    Err(CircuitBreakerError::Failed(e)) => handle_error(e),
}
```

### Pattern 3: Retry with Exponential Backoff

**What:** Retry failed operations with increasing delays
**When:** Transient failures (network blips, temporary overload)

```rust
async fn retry_with_backoff<T, E, F, Fut>(
    mut operation: F,
    max_retries: u32,
) -> Result<T, E>
where
    F: FnMut() -> Fut,
    Fut: Future<Output = Result<T, E>>,
{
    let mut attempt = 0;
    loop {
        match operation().await {
            Ok(result) => return Ok(result),
            Err(e) if attempt < max_retries => {
                attempt += 1;
                let delay = Duration::from_millis(
                    100 * 2u64.pow(attempt)  // 200ms, 400ms, 800ms...
                );
                let jitter = rand::thread_rng().gen_range(0..100);
                tokio::time::sleep(delay + Duration::from_millis(jitter)).await;
            }
            Err(e) => return Err(e),
        }
    }
}
```

### Pattern 4: Bulkhead

**What:** Isolate failures to prevent cascade
**When:** Multiple independent workloads share resources

```rust
// Separate thread pools for different operations
struct Bulkheads {
    database: ThreadPool::new(10),    // Max 10 concurrent DB ops
    external_api: ThreadPool::new(5), // Max 5 concurrent API calls
    file_io: ThreadPool::new(3),      // Max 3 concurrent file ops
}

// If external_api pool is exhausted, database ops still work
let result = bulkheads.external_api.spawn(|| {
    slow_external_service()
}).await;
```

### Pattern 5: Rate Limiting

**What:** Limit request rate to protect systems
**When:** Protecting external services or your own resources

```rust
use governor::{Quota, RateLimiter};

// Allow 100 requests per second
let limiter = RateLimiter::direct(
    Quota::per_second(NonZeroU32::new(100).unwrap())
);

async fn make_request(&self) -> Result<Response, Error> {
    self.limiter.until_ready().await;  // Wait if limit reached
    self.client.request().await
}
```

### Pattern 6: Fallback

**What:** Provide degraded service when primary fails
**When:** You can serve something useful without the failed dependency

```rust
async fn get_user_data(id: &str) -> UserData {
    // Try primary source
    match primary_db.get_user(id).await {
        Ok(data) => data,
        Err(_) => {
            // Fall back to cache
            match cache.get_user(id).await {
                Ok(cached) => {
                    warn!("Serving stale data for user {}", id);
                    cached
                }
                Err(_) => {
                    // Final fallback: default/empty data
                    UserData::default()
                }
            }
        }
    }
}
```

## Anti-Patterns

### Patterns That Destroy Stability

- **Unbounded queues:** Memory exhaustion under load
- **Retry storms:** Overwhelming failing services
- **Cascading failures:** One component takes down all
- **Integration point ignorance:** Not handling failure
- **Blocked threads:** Waiting forever on responses

## Checklist

- [ ] All external calls have timeouts
- [ ] Circuit breakers on flaky dependencies
- [ ] Retry with backoff (not immediate retry)
- [ ] Bulkheads isolate critical paths
- [ ] Rate limiting protects downstream
- [ ] Fallbacks for critical features
- [ ] Health checks for dependencies

## Claude Code Integration
**Command:** /production-ready
**Input:** Feature or codebase
**Output:** Stability pattern audit and recommendations

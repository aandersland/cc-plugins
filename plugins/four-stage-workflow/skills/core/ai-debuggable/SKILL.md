---
name: ai-debuggable
description: Write code that makes its intentions, state transitions, and failure modes explicit and observable.
---

# AI-Debuggable Code Patterns

## Source
- Synthesized from: Observability Engineering, The Debugging Book, practical AI-assisted development
- Key insight: Code that's easy for AI to debug is also easy for humans to debug

## Core Principle

Write code that makes its intentions, state transitions, and failure modes explicit and observable.

## Why This Matters

Claude can only reason about what it can observe. Code patterns that make behavior implicit, hide state, or produce opaque errors dramatically reduce Claude's effectiveness at:
- Diagnosing bugs from logs
- Understanding code flow
- Suggesting targeted fixes
- Tracing causality through the system

## Decision Framework

```
When writing any code, ask:
├── Can Claude trace data flow from input to output?
│   └── No → Add structured logging at boundaries
├── Can Claude understand why a failure occurred?
│   └── No → Use typed errors with context fields
├── Can Claude identify the system state when an event occurred?
│   └── No → Add state to log entries
└── Can Claude correlate related operations?
    └── No → Add correlation IDs / request IDs
```

## Patterns

### Pattern 1: Structured Error Context

**When:** Any operation that can fail
**Apply:** Use typed errors with context fields instead of string messages
**Why:** Claude can pattern-match on structured fields to diagnose issues

```rust
// BAD: Claude can't reason about this
Err(anyhow!("Failed to process"))

// BAD: Context buried in string
Err(anyhow!("Failed to process user {} order {}", user_id, order_id))

// GOOD: Claude can trace causality and suggest fixes
#[derive(Debug, thiserror::Error)]
#[error("Failed to {operation} for {entity_type}:{entity_id}: {reason}")]
pub struct OperationError {
    pub operation: &'static str,
    pub entity_type: &'static str,
    pub entity_id: String,
    pub reason: String,
    #[source]
    pub source: Option<Box<dyn std::error::Error + Send + Sync>>,
}

// Usage:
Err(OperationError {
    operation: "process_payment",
    entity_type: "order",
    entity_id: order.id.to_string(),
    reason: "payment gateway timeout".into(),
    source: Some(Box::new(gateway_error)),
})
```

### Pattern 2: State Transition Logging

**When:** Any state machine or state-bearing struct
**Apply:** Log both the transition trigger and the resulting state change
**Why:** Claude can reconstruct the state timeline from logs

```rust
#[instrument(skip(self), fields(
    from_state = ?self.state,
    trigger = %event,
))]
fn transition(&mut self, event: Event) -> Result<(), TransitionError> {
    let new_state = match (&self.state, &event) {
        (State::Idle, Event::Start) => State::Running,
        (State::Running, Event::Pause) => State::Paused,
        (State::Running, Event::Complete) => State::Done,
        (from, event) => {
            warn!(
                invalid_transition = true,
                "attempted invalid state transition"
            );
            return Err(TransitionError::Invalid { from: *from, event });
        }
    };

    info!(to_state = ?new_state, "state_transition");
    self.state = new_state;
    Ok(())
}
```

### Pattern 3: Correlation Threading

**When:** Any multi-step operation or request handling
**Apply:** Generate a correlation ID at the entry point, thread it through all operations
**Why:** Claude can follow a single request through multiple log entries

```rust
use uuid::Uuid;

#[instrument(fields(request_id = %Uuid::new_v4()))]
pub async fn handle_request(req: Request) -> Response {
    // All child spans automatically inherit request_id
    let user = fetch_user(&req.user_id).await?;
    let order = create_order(&user, &req.items).await?;
    let payment = process_payment(&order).await?;

    info!(
        order_id = %order.id,
        payment_id = %payment.id,
        "request_completed"
    );

    Response::success(order)
}

#[instrument]  // Inherits request_id from parent
async fn fetch_user(user_id: &str) -> Result<User, UserError> {
    debug!(user_id = %user_id, "fetching_user");
    // ...
}
```

### Pattern 4: Decision Logging

**When:** Code makes a decision based on runtime data
**Apply:** Log the decision, the data that informed it, and the reason
**Why:** Claude can understand why the code took a particular path

```rust
// BAD: No visibility into decision
if user.subscription.is_expired() {
    return Err(AccessDenied);
}

// GOOD: Claude can see the decision logic
if user.subscription.is_expired() {
    info!(
        user_id = %user.id,
        subscription_expires_at = %user.subscription.expires_at,
        current_time = %Utc::now(),
        decision = "deny_access",
        reason = "subscription_expired",
        "access_decision"
    );
    return Err(AccessDenied::SubscriptionExpired {
        user_id: user.id.clone(),
        expired_at: user.subscription.expires_at,
    });
}
```

### Pattern 5: Invariant Documentation

**When:** Code depends on assumptions about data
**Apply:** Use debug_assert! with descriptive messages
**Why:** Claude learns what the code expects, even if assertions don't fire in production

```rust
fn calculate_percentage(part: u64, total: u64) -> f64 {
    debug_assert!(total > 0, "total must be positive, got 0");
    debug_assert!(
        part <= total,
        "part ({}) cannot exceed total ({})",
        part,
        total
    );

    (part as f64 / total as f64) * 100.0
}

fn process_items(items: &[Item]) -> ProcessResult {
    debug_assert!(
        items.iter().all(|i| i.is_valid()),
        "all items must be validated before processing"
    );
    // ...
}
```

### Pattern 6: Boundary Snapshots

**When:** Data enters or exits a module/service
**Apply:** Log a snapshot of the data at boundaries
**Why:** Claude can see data transformations and identify where corruption occurs

```rust
#[instrument(skip(input), fields(
    input_count = input.len(),
    input_hash = %hash_for_debug(&input),
))]
pub fn transform(input: Vec<RawData>) -> Vec<ProcessedData> {
    debug!(sample = ?input.first(), "transform_input");

    let output: Vec<ProcessedData> = input
        .into_iter()
        .filter_map(|raw| process_one(raw).ok())
        .collect();

    debug!(
        output_count = output.len(),
        output_hash = %hash_for_debug(&output),
        sample = ?output.first(),
        "transform_output"
    );

    output
}
```

### Pattern 7: Error Chain Preservation

**When:** Converting between error types or wrapping errors
**Apply:** Always preserve the source error chain
**Why:** Claude can trace errors back to their root cause

```rust
// BAD: Lost the original error
fn load_config(path: &Path) -> Result<Config, ConfigError> {
    let content = std::fs::read_to_string(path)
        .map_err(|_| ConfigError::LoadFailed)?;  // Source lost!
    // ...
}

// GOOD: Source preserved
#[derive(Debug, thiserror::Error)]
pub enum ConfigError {
    #[error("failed to read config file: {path}")]
    ReadFailed {
        path: PathBuf,
        #[source]
        source: std::io::Error,
    },

    #[error("failed to parse config")]
    ParseFailed {
        #[source]
        source: toml::de::Error,
    },
}

fn load_config(path: &Path) -> Result<Config, ConfigError> {
    let content = std::fs::read_to_string(path)
        .map_err(|source| ConfigError::ReadFailed {
            path: path.to_path_buf(),
            source,
        })?;
    // ...
}
```

## Anti-Patterns

### Stringly-Typed Errors
```rust
// AVOID: No structure for Claude to analyze
Err("something went wrong".to_string())
Err(format!("error processing {}", id))
```

### Silent Failures
```rust
// AVOID: Failure is invisible
let _ = risky_operation();
if let Ok(result) = maybe_fail() { /* use result */ }
```

### Magic Numbers Without Context
```rust
// AVOID: Why 3? What does timeout mean here?
if retries > 3 { return Err(Timeout); }

// BETTER: Document the reasoning
const MAX_RETRIES: u32 = 3;  // Based on p99 latency of 200ms × 3 = 600ms budget
if retries > MAX_RETRIES {
    warn!(
        retries = retries,
        max = MAX_RETRIES,
        "exceeded_retry_limit"
    );
    return Err(Timeout);
}
```

### State Hidden in Closures
```rust
// AVOID: State is invisible to logs
let processor = |item| {
    counter += 1;  // Hidden mutation
    process(item)
};

// BETTER: Make state explicit
struct Processor { counter: AtomicU64 }
impl Processor {
    fn process(&self, item: Item) -> Result<Output> {
        let count = self.counter.fetch_add(1, Ordering::SeqCst);
        debug!(item_number = count, "processing_item");
        // ...
    }
}
```

## Checklist

When writing new code, verify:
- [ ] All error types have structured context fields
- [ ] State transitions are logged with before/after
- [ ] Long-running operations have correlation IDs
- [ ] Decision points log the decision AND the reason
- [ ] Invariants are documented with debug_assert!
- [ ] Module boundaries log data snapshots
- [ ] Error chains preserve source errors
- [ ] No silent failures (unwrap/expect only where truly impossible)

## Claude Code Integration

**Command:** /instrument
**Input:** Function or module to add observability to
**Output:** Code with tracing spans, structured logs, and correlation IDs

**Command:** /diagnose
**Input:** Log output or error trace
**Output:** Root cause hypothesis based on patterns in this skill

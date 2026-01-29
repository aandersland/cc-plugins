---
name: error-strategy
description: Model operations as railways with success and failure tracks for explicit, composable, and testable error handling.
---

# Error Strategy (Railway Oriented Programming)

## Source
- Book: Domain Modeling Made Functional (Wlaschin)
- Concept: Railway Oriented Programming
- Application: Composable error handling without exception spaghetti

## Core Principle

Model operations as railways with two tracks: success and failure. Once on the failure track, subsequent operations are bypassed. This makes error handling explicit, composable, and testable.

## Decision Framework

```
How should this operation handle errors?
├── Can it fail?
│   ├── Yes → Return Result<T, E>
│   └── No → Return T directly (rare)
├── What kind of failure?
│   ├── Expected business failure → Domain error type
│   ├── Unexpected system failure → anyhow/eyre
│   └── Programmer error → panic (assert, unreachable)
├── Should caller distinguish failure modes?
│   ├── Yes → Specific error enum variants
│   └── No → Opaque error or single variant
└── Can the operation be retried?
    ├── Yes → Error should indicate retryability
    └── No → Error is terminal
```

## Patterns

### Pattern 1: Two-Track Model

**When:** Sequential operations where each can fail
**Apply:** Use Result and the ? operator for automatic track switching

```
   Input
     │
     ▼
┌─────────┐     Success     ┌─────────┐     Success     ┌─────────┐
│ Step 1  │────────────────▶│ Step 2  │────────────────▶│ Step 3  │───▶ Output
└─────────┘                 └─────────┘                 └─────────┘
     │                           │                           │
     │ Failure                   │ Failure                   │ Failure
     │                           │                           │
     ▼                           ▼                           ▼
   Error ◀──────────────────────────────────────────────────┘
```

```rust
fn process_order(input: OrderInput) -> Result<Receipt, OrderError> {
    // Each ? acts as a "switch" - failure diverts to error track
    let validated = validate_order(input)?;      // Switch to error if invalid
    let priced = calculate_pricing(&validated)?; // Switch if pricing fails
    let charged = charge_payment(&priced)?;      // Switch if payment fails
    let receipt = create_receipt(&charged)?;     // Switch if creation fails

    Ok(receipt)  // Only reached if all steps succeed
}
```

### Pattern 2: Adapter Functions

**When:** Connecting functions that don't naturally compose
**Apply:** Create small adapters that transform inputs/outputs

```rust
// "Dead-end" function that has side effects
fn log_order(order: &Order) {
    info!(order_id = %order.id, "order_logged");
}

// Adapter: Tee - perform side effect, continue with value
fn tee<T, F>(value: T, f: F) -> T
where
    F: FnOnce(&T),
{
    f(&value);
    value
}

// "Single-track" function that can't fail
fn add_timestamp(mut order: Order) -> Order {
    order.timestamp = Utc::now();
    order
}

// Adapter: Lift single-track to two-track
fn lift<T, U, F>(f: F) -> impl Fn(T) -> Result<U, Infallible>
where
    F: Fn(T) -> U,
{
    move |t| Ok(f(t))
}

// Composition
fn process(input: OrderInput) -> Result<Order, Error> {
    validate(input)
        .map(|order| tee(order, log_order))  // Side effect
        .map(add_timestamp)                   // Single-track lifted
        .and_then(verify_inventory)           // Two-track
        .map(|order| tee(order, notify_warehouse))
}
```

### Pattern 3: Error Type Design

**When:** Defining errors for a domain
**Apply:** Create a hierarchy that matches how callers will handle errors

```rust
// Top-level application error
#[derive(Debug, Error)]
pub enum AppError {
    #[error("validation error: {0}")]
    Validation(#[from] ValidationError),

    #[error("payment error: {0}")]
    Payment(#[from] PaymentError),

    #[error("inventory error: {0}")]
    Inventory(#[from] InventoryError),

    #[error("internal error")]
    Internal(#[from] anyhow::Error),
}

// Domain-specific errors with variants callers need
#[derive(Debug, Error)]
pub enum ValidationError {
    #[error("missing required field: {field}")]
    MissingField { field: &'static str },

    #[error("invalid email: {email}")]
    InvalidEmail { email: String },

    #[error("quantity must be positive, got {quantity}")]
    InvalidQuantity { quantity: i32 },
}

// Error classification helpers
impl AppError {
    pub fn is_retryable(&self) -> bool {
        matches!(
            self,
            AppError::Payment(PaymentError::Timeout) |
            AppError::Internal(_)
        )
    }

    pub fn is_user_error(&self) -> bool {
        matches!(self, AppError::Validation(_))
    }
}
```

### Pattern 4: Validation Accumulation

**When:** You want to collect all validation errors, not just the first
**Apply:** Use a validation type that accumulates errors

```rust
struct Validation<T> {
    value: Option<T>,
    errors: Vec<ValidationError>,
}

impl<T> Validation<T> {
    fn success(value: T) -> Self {
        Validation { value: Some(value), errors: vec![] }
    }

    fn failure(error: ValidationError) -> Self {
        Validation { value: None, errors: vec![error] }
    }

    fn and<U>(self, other: Validation<U>) -> Validation<(T, U)> {
        match (self.value, other.value) {
            (Some(t), Some(u)) => Validation {
                value: Some((t, u)),
                errors: vec![],
            },
            _ => Validation {
                value: None,
                errors: [self.errors, other.errors].concat(),
            }
        }
    }

    fn into_result(self) -> Result<T, Vec<ValidationError>> {
        match self.value {
            Some(v) if self.errors.is_empty() => Ok(v),
            _ => Err(self.errors),
        }
    }
}

// Usage: validate multiple fields, collect all errors
fn validate_user(input: UserInput) -> Result<ValidUser, Vec<ValidationError>> {
    let name = validate_name(&input.name);
    let email = validate_email(&input.email);
    let age = validate_age(input.age);

    name.and(email)
        .and(age)
        .map(|((name, email), age)| ValidUser { name, email, age })
        .into_result()
}
```

### Pattern 5: Context Enrichment

**When:** Errors need additional context as they propagate
**Apply:** Use .context() or .with_context() from anyhow

```rust
use anyhow::{Context, Result};

async fn load_user_profile(user_id: &str) -> Result<Profile> {
    let user = fetch_user(user_id)
        .await
        .with_context(|| format!("failed to fetch user {}", user_id))?;

    let preferences = load_preferences(&user.preferences_id)
        .await
        .context("failed to load preferences")?;

    let avatar = fetch_avatar(&user.avatar_url)
        .await
        .context("failed to fetch avatar")?;

    Ok(Profile { user, preferences, avatar })
}

// Error chain shows full context:
// Error: failed to fetch user usr_123
//
// Caused by:
//     0: database query failed
//     1: connection timeout
```

### Pattern 6: Recovery and Fallback

**When:** Some errors should be recovered from inline
**Apply:** Use .or_else() or pattern matching

```rust
fn get_user_name(user_id: &str) -> String {
    fetch_user(user_id)
        .map(|u| u.name.clone())
        .or_else(|e| {
            warn!(error = ?e, "falling back to guest name");
            Ok::<_, Infallible>("Guest".to_string())
        })
        .unwrap()  // Safe because we provided fallback
}

// With different fallback strategies
fn get_data(key: &str) -> Result<Data, FetchError> {
    cache_get(key)
        .or_else(|_| database_get(key))
        .or_else(|_| remote_fetch(key))
}

// With logging at each level
fn get_config(key: &str) -> Config {
    load_from_file(key)
        .inspect_err(|e| debug!(error = ?e, "config file not found"))
        .or_else(|_| load_from_env(key))
        .inspect_err(|e| debug!(error = ?e, "config env not found"))
        .unwrap_or_else(|_| {
            info!(key = %key, "using default config");
            Config::default()
        })
}
```

### Pattern 7: Async Railway

**When:** Async operations in a chain
**Apply:** Use async/await with Result, consider futures combinators

```rust
use futures::TryFutureExt;

async fn process_async(input: Input) -> Result<Output, Error> {
    // Simple async chain
    let step1 = async_step1(input).await?;
    let step2 = async_step2(step1).await?;
    let step3 = async_step3(step2).await?;
    Ok(step3)

    // Or with combinators for more complex flows
    async_step1(input)
        .and_then(|s1| async_step2(s1))
        .and_then(|s2| async_step3(s2))
        .await
}

// Parallel with error handling
async fn fetch_all(ids: &[String]) -> Result<Vec<Data>, Error> {
    let futures: Vec<_> = ids.iter()
        .map(|id| fetch_one(id))
        .collect();

    futures::future::try_join_all(futures).await
}
```

## Anti-Patterns

### Exception Tunneling
```rust
// AVOID: Hiding errors in panics
fn get_value() -> i32 {
    some_operation().expect("should never fail")  // Will panic!
}

// Better: Propagate the error
fn get_value() -> Result<i32, Error> {
    some_operation()
}
```

### Error Swallowing
```rust
// AVOID: Silently dropping errors
let _ = risky_operation();
if let Ok(v) = maybe_fail() { /* use v */ }

// Better: Handle or log
risky_operation().inspect_err(|e| warn!(?e, "non-critical failure"))?;
```

### Stringly Typed Errors
```rust
// AVOID: String errors lose structure
fn process() -> Result<(), String> {
    Err("something went wrong".into())
}

// Better: Typed errors
fn process() -> Result<(), ProcessError> {
    Err(ProcessError::ValidationFailed { field: "email" })
}
```

## Checklist

- [ ] All fallible operations return Result
- [ ] Error types match how callers will handle them
- [ ] Errors include context for debugging
- [ ] No silent error swallowing
- [ ] Validation errors accumulated where appropriate
- [ ] Recovery strategies explicitly defined
- [ ] Error chains preserved through conversions

## Claude Code Integration

**Command:** /error-design
**Input:** Operations that can fail in a workflow
**Output:** Error types and railway-style composition

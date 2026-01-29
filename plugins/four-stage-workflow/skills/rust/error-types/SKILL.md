---
name: error-types
description: Use typed errors for domain boundaries and library APIs; use anyhow for application-level error aggregation.
---

# Rust Error Types

## Source
- Book: Domain Modeling Made Functional (adapted for Rust)
- Crates: thiserror, anyhow
- Key concept: Make illegal states unrepresentable through the type system

## Core Principle

Use typed errors for domain boundaries and library APIs; use anyhow for application-level error aggregation.

## Decision Framework

```
What kind of code are you writing?
├── Library / Public API
│   └── Use thiserror with specific error variants
│       (callers need to handle different cases)
├── Application logic
│   └── Does the caller need to distinguish error types?
│       ├── Yes → Use thiserror enum
│       └── No → Use anyhow::Result with context
├── Internal module
│   └── Use thiserror for module boundary
│       Chain with anyhow at application level
└── Quick prototyping
    └── Use anyhow everywhere, refine later
```

## Patterns

### Pattern 1: Domain Error Enum (thiserror)

**When:** Library code or any boundary where callers must handle different failure modes
**Apply:** Create an enum with descriptive variants and context fields

```rust
use thiserror::Error;

#[derive(Debug, Error)]
pub enum OrderError {
    #[error("order {order_id} not found")]
    NotFound { order_id: String },

    #[error("cannot modify order {order_id} in state {state:?}")]
    InvalidState {
        order_id: String,
        state: OrderState,
        attempted_action: &'static str,
    },

    #[error("insufficient inventory for product {product_id}: requested {requested}, available {available}")]
    InsufficientInventory {
        product_id: String,
        requested: u32,
        available: u32,
    },

    #[error("payment failed for order {order_id}")]
    PaymentFailed {
        order_id: String,
        #[source]
        source: PaymentError,
    },

    #[error("database error")]
    Database(#[from] sqlx::Error),
}
```

### Pattern 2: Error Context Chaining (anyhow)

**When:** Application code where you want to add context without defining new types
**Apply:** Use `.context()` or `.with_context()` to add layers

```rust
use anyhow::{Context, Result};

async fn load_user_dashboard(user_id: &str) -> Result<Dashboard> {
    let user = fetch_user(user_id)
        .await
        .with_context(|| format!("failed to fetch user {}", user_id))?;

    let preferences = load_preferences(&user)
        .await
        .context("failed to load user preferences")?;

    let metrics = gather_metrics(&user)
        .await
        .context("failed to gather dashboard metrics")?;

    Ok(Dashboard::new(user, preferences, metrics))
}

// Error output shows full chain:
// Error: failed to fetch user abc123
//
// Caused by:
//     0: database connection failed
//     1: connection refused
```

### Pattern 3: Hybrid Approach (thiserror + anyhow)

**When:** Library exposes typed errors, application consumes them with context
**Apply:** Define domain errors with thiserror, wrap at application boundary

```rust
// In library crate
#[derive(Debug, Error)]
pub enum AuthError {
    #[error("invalid credentials for user {username}")]
    InvalidCredentials { username: String },

    #[error("account locked until {locked_until}")]
    AccountLocked { locked_until: DateTime<Utc> },

    #[error("token expired")]
    TokenExpired,
}

// In application
use anyhow::{Context, Result};

async fn handle_login(req: LoginRequest) -> Result<Session> {
    let auth_result = auth_service
        .authenticate(&req.username, &req.password)
        .await
        .context("authentication service call failed")?;

    match auth_result {
        Ok(token) => create_session(token).await,
        Err(AuthError::InvalidCredentials { .. }) => {
            // Log but don't expose details
            info!("login failed: invalid credentials");
            Err(anyhow!("login failed"))
        }
        Err(AuthError::AccountLocked { locked_until }) => {
            Err(anyhow!("account locked until {}", locked_until))
        }
        Err(e) => Err(e).context("unexpected auth error"),
    }
}
```

### Pattern 4: Error with Recovery Actions

**When:** Some errors have clear recovery paths
**Apply:** Include recovery information in the error type

```rust
#[derive(Debug, Error)]
pub enum ConfigError {
    #[error("config file not found at {path}")]
    NotFound {
        path: PathBuf,
        /// Suggested action for the user
        suggestion: &'static str,
    },

    #[error("invalid config format")]
    ParseError {
        #[source]
        source: toml::de::Error,
        line: Option<usize>,
        suggestion: String,
    },
}

impl ConfigError {
    pub fn not_found(path: PathBuf) -> Self {
        Self::NotFound {
            path,
            suggestion: "Run 'app init' to create a default config file",
        }
    }
}
```

### Pattern 5: Error Conversion Boundaries

**When:** Crossing module or crate boundaries
**Apply:** Implement From for clean error propagation

```rust
#[derive(Debug, Error)]
pub enum ServiceError {
    #[error("database error: {0}")]
    Database(#[from] DbError),

    #[error("cache error: {0}")]
    Cache(#[from] CacheError),

    #[error("external API error: {0}")]
    ExternalApi(#[from] ApiError),

    #[error("validation error: {message}")]
    Validation { message: String },
}

// Now you can use ? freely across these error types
async fn fetch_data(id: &str) -> Result<Data, ServiceError> {
    let cached = cache.get(id).await?;  // CacheError → ServiceError
    if let Some(data) = cached {
        return Ok(data);
    }

    let db_data = db.query(id).await?;  // DbError → ServiceError
    let enriched = api.enrich(&db_data).await?;  // ApiError → ServiceError

    cache.set(id, &enriched).await?;
    Ok(enriched)
}
```

### Pattern 6: Structured Logging Integration

**When:** Errors need to be logged with structured data
**Apply:** Implement helper methods for tracing integration

```rust
#[derive(Debug, Error)]
#[error("operation {operation} failed for {entity_type}:{entity_id}")]
pub struct OperationError {
    pub operation: &'static str,
    pub entity_type: &'static str,
    pub entity_id: String,
    pub reason: String,
    #[source]
    pub source: Option<Box<dyn std::error::Error + Send + Sync>>,
}

impl OperationError {
    /// Log this error with full structured context
    pub fn log(&self) {
        error!(
            operation = self.operation,
            entity_type = self.entity_type,
            entity_id = %self.entity_id,
            reason = %self.reason,
            error = ?self.source,
            "operation_failed"
        );
    }

    /// Create span fields for this error
    pub fn as_span_fields(&self) -> impl tracing::Value {
        format!(
            "{}:{}:{}",
            self.operation, self.entity_type, self.entity_id
        )
    }
}
```

## Anti-Patterns

### Stringly-Typed Errors
```rust
// AVOID
fn process() -> Result<(), String> {
    Err("something went wrong".to_string())
}

// Also AVOID: anyhow without context
fn process() -> anyhow::Result<()> {
    do_thing()?;  // No context if do_thing fails
    Ok(())
}
```

### Overly Broad Error Types
```rust
// AVOID: One variant catches everything
#[derive(Error)]
pub enum AppError {
    #[error("{0}")]
    General(String),  // This is just String with extra steps
}
```

### Losing Error Sources
```rust
// AVOID: Source error is lost
impl From<IoError> for MyError {
    fn from(e: IoError) -> Self {
        MyError::Io(e.to_string())  // Lost the original!
    }
}

// CORRECT: Preserve the source
impl From<IoError> for MyError {
    fn from(source: IoError) -> Self {
        MyError::Io { source }
    }
}
```

### Error Type Proliferation
```rust
// AVOID: One error type per function
pub enum FetchUserError { ... }
pub enum UpdateUserError { ... }
pub enum DeleteUserError { ... }

// BETTER: One error type per domain
pub enum UserError {
    NotFound { id: String },
    InvalidUpdate { reason: String },
    // ...
}
```

## Checklist

- [ ] Library/public APIs use thiserror with specific variants
- [ ] Each error variant includes enough context to diagnose
- [ ] Error sources are preserved with #[source] or #[from]
- [ ] Application code uses .context() to add information
- [ ] Errors at boundaries can be logged with structured data
- [ ] No stringly-typed errors in production code
- [ ] Error types match domain concepts, not implementation details

## Claude Code Integration

**Command:** /error-design
**Input:** Description of operations that can fail
**Output:** thiserror enum definition with appropriate variants and context fields

---
name: result-combinators
description: Chain operations on Result and Option types using combinators instead of match statements, creating a railway where success flows forward and errors automatically short-circuit.
---

# Rust Result Combinators

## Source
- Rust standard library documentation
- Book: Domain Modeling Made Functional (Railway Oriented Programming)
- Concept: Composable error handling through monadic operations

## Core Principle

Chain operations on Result and Option types using combinators instead of match statements, creating a "railway" where success flows forward and errors automatically short-circuit.

## Decision Framework

```
What do you want to do with a Result<T, E>?
├── Transform the success value (T → U)
│   └── Use .map(|t| ...)
├── Transform the error value (E → F)
│   └── Use .map_err(|e| ...)
├── Chain another fallible operation (T → Result<U, E>)
│   └── Use .and_then(|t| ...)
├── Provide a default on error
│   └── Use .unwrap_or(default) or .unwrap_or_else(|| ...)
├── Convert Option<T> to Result<T, E>
│   └── Use .ok_or(error) or .ok_or_else(|| error)
├── Convert Result<T, E> to Option<T>
│   └── Use .ok()
└── Early return on error
    └── Use ? operator
```

## Patterns

### Pattern 1: Map for Value Transformation

**When:** You have a Result and want to transform the success value
**Apply:** Use `.map()` to transform without changing error type

```rust
// Transform success value
fn get_user_name(id: &str) -> Result<String, DbError> {
    fetch_user(id)
        .map(|user| user.name.clone())
}

// Chain multiple maps
fn get_user_display(id: &str) -> Result<String, DbError> {
    fetch_user(id)
        .map(|user| user.name.clone())
        .map(|name| format!("User: {}", name))
        .map(|display| display.to_uppercase())
}

// Map with complex transformation
fn get_order_summary(id: &str) -> Result<OrderSummary, OrderError> {
    fetch_order(id).map(|order| OrderSummary {
        id: order.id,
        total: order.items.iter().map(|i| i.price).sum(),
        item_count: order.items.len(),
        status: order.status,
    })
}
```

### Pattern 2: Map_err for Error Transformation

**When:** Converting between error types at boundaries
**Apply:** Use `.map_err()` to transform errors while preserving success

```rust
// Convert error type
fn load_config() -> Result<Config, AppError> {
    std::fs::read_to_string("config.toml")
        .map_err(|e| AppError::ConfigLoad(e.to_string()))?
        .parse()
        .map_err(|e| AppError::ConfigParse(e.to_string()))
}

// Add context to errors
fn fetch_user(id: &str) -> Result<User, ServiceError> {
    db.query_user(id)
        .map_err(|e| ServiceError::Database {
            operation: "fetch_user",
            id: id.to_string(),
            source: e,
        })
}

// Chain map and map_err
fn process_payment(order_id: &str) -> Result<Receipt, AppError> {
    fetch_order(order_id)
        .map_err(AppError::from)?
        .validate()
        .map_err(|e| AppError::Validation(e.to_string()))?
        .charge()
        .map(|payment| Receipt::new(payment))
        .map_err(AppError::Payment)
}
```

### Pattern 3: And_then for Chaining Operations

**When:** Each step produces a Result and depends on the previous step's success
**Apply:** Use `.and_then()` to chain fallible operations

```rust
// Sequential operations that each might fail
fn create_order(user_id: &str, items: Vec<Item>) -> Result<Order, OrderError> {
    fetch_user(user_id)
        .and_then(|user| validate_user_can_order(&user))
        .and_then(|user| check_inventory(&items).map(|_| user))
        .and_then(|user| calculate_totals(&user, &items))
        .and_then(|totals| create_order_record(user_id, items, totals))
}

// With intermediate transformations
fn authenticate(username: &str, password: &str) -> Result<Session, AuthError> {
    find_user(username)
        .and_then(|user| verify_password(&user, password).map(|_| user))
        .and_then(|user| check_account_status(&user).map(|_| user))
        .and_then(|user| create_session(&user))
        .map(|session| {
            info!(user = %username, "authentication_successful");
            session
        })
}

// Async version with futures
async fn process_request(req: Request) -> Result<Response, Error> {
    validate_request(&req)
        .await
        .and_then(|valid_req| async {
            fetch_data(&valid_req).await
        })?
        .and_then(|data| async {
            transform_data(data).await
        })?
        .map(Response::success)
}
```

### Pattern 4: Or_else for Fallback Chains

**When:** You want to try alternative operations if the first fails
**Apply:** Use `.or_else()` to chain fallback attempts

```rust
// Try cache, fall back to database
fn get_user(id: &str) -> Result<User, FetchError> {
    cache.get(id)
        .or_else(|_| {
            info!(id = %id, "cache_miss");
            db.fetch(id)
        })
        .or_else(|_| {
            warn!(id = %id, "db_miss_trying_remote");
            remote_service.fetch(id)
        })
}

// Try multiple file locations
fn find_config() -> Result<Config, ConfigError> {
    load_config_from("./config.toml")
        .or_else(|_| load_config_from("~/.config/app/config.toml"))
        .or_else(|_| load_config_from("/etc/app/config.toml"))
        .or_else(|_| {
            info!("no config found, using defaults");
            Ok(Config::default())
        })
}
```

### Pattern 5: Unwrap_or_else for Default Values

**When:** You need a value and have a reasonable default if the operation fails
**Apply:** Use `.unwrap_or_else()` for computed defaults, `.unwrap_or()` for constants

```rust
// Simple default
let timeout = config.get_timeout().unwrap_or(Duration::from_secs(30));

// Computed default (lazy evaluation)
let user = fetch_user(id).unwrap_or_else(|e| {
    warn!(error = ?e, "using guest user due to fetch failure");
    User::guest()
});

// Default with logging
let config = load_config().unwrap_or_else(|e| {
    error!(error = ?e, "failed to load config, using defaults");
    Config::default()
});

// Pattern: try expensive operation, fall back to cheap
let data = fetch_from_remote()
    .unwrap_or_else(|_| fetch_from_cache())
    .unwrap_or_else(|_| compute_locally());
```

### Pattern 6: Option and Result Interop

**When:** Converting between Option and Result
**Apply:** Use `.ok_or()`, `.ok_or_else()`, `.ok()`, `.transpose()`

```rust
// Option → Result
fn get_required_header(req: &Request, name: &str) -> Result<&str, HeaderError> {
    req.headers()
        .get(name)
        .ok_or(HeaderError::Missing(name.to_string()))
}

// Option → Result with computed error
fn find_user(id: &str) -> Result<User, UserError> {
    users.get(id)
        .cloned()
        .ok_or_else(|| UserError::NotFound { id: id.to_string() })
}

// Result → Option (discard error info)
fn try_parse_config(path: &Path) -> Option<Config> {
    std::fs::read_to_string(path)
        .ok()
        .and_then(|content| content.parse().ok())
}

// Option<Result<T, E>> ↔ Result<Option<T>, E>
fn maybe_fetch(id: Option<&str>) -> Result<Option<User>, DbError> {
    id.map(|i| fetch_user(i))  // Option<Result<User, DbError>>
        .transpose()            // Result<Option<User>, DbError>
}
```

### Pattern 7: Combining Multiple Results

**When:** You have multiple independent Results and need all to succeed
**Apply:** Use tuple patterns or collect into Result<Vec<T>, E>

```rust
// Combine with tuple pattern
fn create_report(user_id: &str, order_id: &str) -> Result<Report, Error> {
    let user = fetch_user(user_id)?;
    let order = fetch_order(order_id)?;
    let metrics = fetch_metrics(user_id)?;

    Ok(Report { user, order, metrics })
}

// Collect iterator of Results
fn fetch_all_users(ids: &[&str]) -> Result<Vec<User>, DbError> {
    ids.iter()
        .map(|id| fetch_user(id))
        .collect()  // Stops at first error
}

// Process all, collecting errors separately
fn process_batch(items: Vec<Item>) -> (Vec<Output>, Vec<Error>) {
    let (successes, errors): (Vec<_>, Vec<_>) = items
        .into_iter()
        .map(process_item)
        .partition_map(|r| match r {
            Ok(v) => Either::Left(v),
            Err(e) => Either::Right(e),
        });
    (successes, errors)
}
```

### Pattern 8: Inspect for Side Effects

**When:** You want to log or perform side effects without changing the Result
**Apply:** Use `.inspect()` and `.inspect_err()` (nightly) or tap pattern

```rust
// With tap pattern (works on stable)
trait ResultExt<T, E> {
    fn tap<F: FnOnce(&T)>(self, f: F) -> Self;
    fn tap_err<F: FnOnce(&E)>(self, f: F) -> Self;
}

impl<T, E> ResultExt<T, E> for Result<T, E> {
    fn tap<F: FnOnce(&T)>(self, f: F) -> Self {
        if let Ok(ref v) = self { f(v); }
        self
    }

    fn tap_err<F: FnOnce(&E)>(self, f: F) -> Self {
        if let Err(ref e) = self { f(e); }
        self
    }
}

// Usage
fn process(data: &str) -> Result<Output, Error> {
    parse(data)
        .tap(|parsed| debug!(parsed = ?parsed, "parsed_successfully"))
        .tap_err(|e| warn!(error = ?e, "parse_failed"))
        .and_then(validate)
        .tap(|valid| info!("validation_passed"))
        .and_then(transform)
}
```

## Anti-Patterns

### Nested Match Pyramids
```rust
// AVOID
match fetch_user(id) {
    Ok(user) => match validate(&user) {
        Ok(valid) => match process(&valid) {
            Ok(result) => Ok(result),
            Err(e) => Err(e.into()),
        },
        Err(e) => Err(e.into()),
    },
    Err(e) => Err(e.into()),
}

// BETTER
fetch_user(id)
    .and_then(|user| validate(&user))
    .and_then(|valid| process(&valid))
```

### Unwrap in Production Code
```rust
// AVOID
let user = fetch_user(id).unwrap();  // Panics!

// BETTER
let user = fetch_user(id)?;  // Propagates error

// Or with default
let user = fetch_user(id).unwrap_or_else(|_| User::guest());
```

### Ignoring Errors Silently
```rust
// AVOID
let _ = risky_operation();

// BETTER: At least log
if let Err(e) = risky_operation() {
    warn!(error = ?e, "risky operation failed (non-critical)");
}

// Or make it explicit this is intentional
let _ignored = risky_operation();  // Deliberately ignoring result
```

## Checklist

- [ ] No nested match pyramids for Result handling
- [ ] No .unwrap() outside of tests or truly impossible cases
- [ ] Error types flow correctly through combinators
- [ ] Side effects (logging) don't consume the Result
- [ ] Option/Result conversions use appropriate methods
- [ ] Collection of Results collected properly

## Claude Code Integration

**Command:** /error-design
**Input:** Code with nested Result handling
**Output:** Refactored code using appropriate combinators

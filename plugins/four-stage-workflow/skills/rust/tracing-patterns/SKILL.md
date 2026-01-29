---
name: tracing-patterns
description: Log structured events with enough context that Claude can diagnose issues from logs alone, without needing to reproduce the problem.
---

# Rust Tracing Patterns

## Source
- Crate: tracing, tracing-subscriber
- Book: Observability Engineering (Charity Majors et al.)
- Focus: Structured, high-cardinality logging for AI-assisted debugging

## Core Principle

Log structured events with enough context that Claude can diagnose issues from logs alone, without needing to reproduce the problem.

## Decision Framework

```
What level should this log be?
├── ERROR: Operation failed, requires attention
│   └── Include: error, context, recovery suggestion
├── WARN: Unexpected but handled condition
│   └── Include: what happened, what was done about it
├── INFO: Significant business events
│   └── Include: operation completed, key identifiers
├── DEBUG: Useful for development/debugging
│   └── Include: data transformations, decision points
└── TRACE: Very verbose, usually disabled
    └── Include: function entry/exit, loop iterations

What should be in a span vs event?
├── Span: Wraps a unit of work with duration
│   └── Use for: requests, operations, function calls
└── Event: Point-in-time occurrence
    └── Use for: state changes, decisions, errors
```

## Patterns

### Pattern 1: Instrumented Functions

**When:** Any function worth tracking (most public functions, async operations)
**Apply:** Use #[instrument] with appropriate field selection

```rust
use tracing::{instrument, info, warn, error};

// Basic instrumentation - logs entry and exit with arguments
#[instrument]
pub fn process_order(order_id: &str, items: &[Item]) -> Result<Receipt> {
    // Function body
}

// Skip large/sensitive fields, add computed fields
#[instrument(
    skip(password, large_payload),
    fields(
        user_id = %user.id,
        item_count = items.len(),
    )
)]
pub async fn checkout(
    user: &User,
    password: &str,
    items: Vec<Item>,
    large_payload: &[u8],
) -> Result<Order> {
    // ...
}

// Set span level for less important functions
#[instrument(level = "debug")]
fn helper_function(data: &Data) -> ProcessedData {
    // ...
}

// Name the span explicitly
#[instrument(name = "db_query", skip(conn))]
async fn execute_query(conn: &DbConn, query: &str) -> Result<Rows> {
    // ...
}
```

### Pattern 2: Correlation IDs

**When:** Any request/operation that spans multiple functions or services
**Apply:** Generate ID at entry point, thread through all operations

```rust
use tracing::{instrument, info_span, Instrument};
use uuid::Uuid;

// Entry point generates the correlation ID
#[instrument(fields(request_id = %Uuid::new_v4()))]
pub async fn handle_request(req: Request) -> Response {
    let user = fetch_user(&req.user_id).await?;
    let result = process(&user, &req.data).await?;
    Ok(Response::new(result))
}

// Child functions automatically inherit request_id
#[instrument]
async fn fetch_user(user_id: &str) -> Result<User> {
    // Logs will include request_id from parent span
    info!("fetching user from database");
    // ...
}

// Manual span creation with correlation
async fn background_task(request_id: Uuid, data: TaskData) {
    let span = info_span!(
        "background_task",
        %request_id,
        task_type = "email_send"
    );

    async {
        // Work happens inside the span
        send_email(&data).await?;
        info!("background task completed");
    }
    .instrument(span)
    .await;
}
```

### Pattern 3: State Transition Events

**When:** System state changes in meaningful ways
**Apply:** Log the transition with before/after state and trigger

```rust
use tracing::{info, warn};

#[derive(Debug, Clone, Copy)]
pub enum ConnectionState {
    Disconnected,
    Connecting,
    Connected,
    Reconnecting,
}

impl Connection {
    fn transition(&mut self, new_state: ConnectionState, reason: &str) {
        let old_state = self.state;
        self.state = new_state;

        info!(
            from_state = ?old_state,
            to_state = ?new_state,
            reason = %reason,
            connection_id = %self.id,
            "connection_state_change"
        );
    }

    async fn handle_disconnect(&mut self, error: Option<&Error>) {
        if let Some(e) = error {
            warn!(
                error = ?e,
                state = ?self.state,
                "connection_lost_with_error"
            );
        }

        self.transition(ConnectionState::Reconnecting, "lost connection");
        self.reconnect().await;
    }
}
```

### Pattern 4: Decision Point Logging

**When:** Code makes runtime decisions based on data
**Apply:** Log the decision, the data that informed it, and alternatives considered

```rust
use tracing::{debug, info};

async fn select_payment_processor(order: &Order) -> PaymentProcessor {
    let amount = order.total_amount();
    let user_country = order.shipping_address.country;
    let preferred = order.user.preferred_processor;

    let processor = if amount > Money::new(10_000, Currency::USD) {
        debug!(
            amount = %amount,
            threshold = "10000 USD",
            decision = "high_value_processor",
            "selecting processor for high-value order"
        );
        PaymentProcessor::HighValue
    } else if let Some(p) = preferred {
        debug!(
            preferred = ?p,
            decision = "user_preferred",
            "using user's preferred processor"
        );
        p
    } else {
        debug!(
            country = %user_country,
            decision = "default_by_region",
            "using regional default processor"
        );
        PaymentProcessor::for_region(user_country)
    };

    info!(
        processor = ?processor,
        order_id = %order.id,
        "payment_processor_selected"
    );

    processor
}
```

### Pattern 5: Error Context Spans

**When:** An error might occur in a complex operation
**Apply:** Wrap error-prone sections in spans that capture diagnostic context

```rust
use tracing::{error, info_span, Instrument};

async fn import_data(source: DataSource) -> Result<ImportResult> {
    let validation_span = info_span!(
        "validation",
        source_type = %source.type_name(),
        record_count = source.estimated_count(),
    );

    let validated = async {
        let records = source.read_all().await?;

        for (idx, record) in records.iter().enumerate() {
            if let Err(e) = validate(record) {
                error!(
                    record_index = idx,
                    record_id = ?record.id,
                    error = ?e,
                    "validation_failed"
                );
                return Err(e.into());
            }
        }

        Ok(records)
    }
    .instrument(validation_span)
    .await?;

    // Continue with import...
}
```

### Pattern 6: Metric-Ready Events

**When:** Events should be aggregatable for metrics
**Apply:** Use consistent field names and values suitable for metric extraction

```rust
use tracing::info;
use std::time::Instant;

async fn process_request(req: Request) -> Response {
    let start = Instant::now();
    let result = handle(&req).await;
    let duration = start.elapsed();

    // These fields can be extracted by metrics systems
    info!(
        // Dimensions (low cardinality for grouping)
        endpoint = %req.path,
        method = %req.method,
        status = %result.status_code(),

        // Measurements
        duration_ms = duration.as_millis() as u64,
        response_size_bytes = result.body.len(),

        // High cardinality (for tracing, not metrics)
        request_id = %req.id,
        user_id = %req.user_id,

        "request_completed"
    );

    result
}
```

### Pattern 7: Span Events for Progress

**When:** Long-running operations should show progress
**Apply:** Emit events within a span to track progress

```rust
use tracing::{info, info_span, Instrument};

async fn batch_process(items: Vec<Item>) -> Result<BatchResult> {
    let span = info_span!(
        "batch_process",
        total_items = items.len(),
    );

    async {
        let mut processed = 0;
        let mut failed = 0;

        for (idx, item) in items.into_iter().enumerate() {
            match process_item(item).await {
                Ok(_) => processed += 1,
                Err(e) => {
                    error!(
                        item_index = idx,
                        error = ?e,
                        "item_processing_failed"
                    );
                    failed += 1;
                }
            }

            // Progress event every 100 items
            if (idx + 1) % 100 == 0 {
                info!(
                    progress = idx + 1,
                    processed = processed,
                    failed = failed,
                    "batch_progress"
                );
            }
        }

        info!(
            total_processed = processed,
            total_failed = failed,
            "batch_complete"
        );

        Ok(BatchResult { processed, failed })
    }
    .instrument(span)
    .await
}
```

## Anti-Patterns

### Unstructured Messages
```rust
// AVOID
info!("Processing user {} with {} items", user_id, items.len());

// BETTER
info!(
    user_id = %user_id,
    item_count = items.len(),
    "processing_user"
);
```

### High-Cardinality Dimensions
```rust
// AVOID: user_id as a dimension means unbounded cardinality
// This breaks metric aggregation
let counter = REQUESTS_BY_USER.with_label_values(&[&user_id]);

// BETTER: Use structured logging for high-cardinality data
info!(user_id = %user_id, "request_received");
// Use low-cardinality dimensions for metrics
let counter = REQUESTS_BY_ENDPOINT.with_label_values(&[&endpoint]);
```

### Missing Context in Errors
```rust
// AVOID
error!("database query failed");

// BETTER
error!(
    query_type = "user_fetch",
    user_id = %user_id,
    error = ?e,
    retry_count = retries,
    "database_query_failed"
);
```

### Over-Logging in Hot Paths
```rust
// AVOID: Logging in tight loops
for item in large_collection {
    debug!(item = ?item, "processing");  // Millions of logs
}

// BETTER: Log aggregates or sample
debug!(count = large_collection.len(), "starting batch");
for (idx, item) in large_collection.iter().enumerate() {
    if idx % 1000 == 0 {
        debug!(progress = idx, "batch_progress");
    }
}
```

## Checklist

- [ ] All public async functions have #[instrument]
- [ ] Request handlers generate correlation IDs
- [ ] State changes are logged with before/after
- [ ] Decision points log the decision AND the data
- [ ] Errors include diagnostic context
- [ ] Log levels are appropriate (not everything is INFO)
- [ ] High-cardinality data is in events, not span names
- [ ] Long operations emit progress events

## Claude Code Integration

**Command:** /instrument
**Input:** Function or module code
**Output:** Code with appropriate #[instrument], spans, and events

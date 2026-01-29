---
name: observability-design
description: Emit structured events with high-cardinality fields to understand any system behavior after the fact.
---

# Observability Design

## Source
- Book: Observability Engineering
- Authors: Charity Majors, Liz Fong-Jones, George Miranda
- Key Concept: High-cardinality, wide events for debugging

## Core Principle
Emit structured events with high-cardinality fields that allow you to understand any system behavior after the fact, without knowing in advance what questions you'll ask.

## Decision Framework

```
What to instrument:
├── Request/transaction start and end
├── State transitions
├── External calls (before and after)
├── Error conditions
├── Business logic decisions
└── Any "I wish I knew what happened here" moment
```

## Patterns

### Pattern 1: Wide Events

**What:** Single event with many fields vs many separate events
**When:** Always prefer wide events

```rust
// AVOID: Many small events
info!("Request started");
info!("User found");
info!("Processing order");
info!("Order complete");

// PREFER: One wide event per request
info!(
    request_id = %request_id,
    user_id = %user.id,
    user_email = %user.email,
    order_id = %order.id,
    items_count = order.items.len(),
    total_amount = %order.total,
    processing_time_ms = elapsed.as_millis(),
    status = "completed",
    "order_processed"
);
```

### Pattern 2: Correlation IDs

**What:** Unique ID that flows through all operations in a request
**When:** Every request or transaction

```rust
use uuid::Uuid;

// Generate at entry point
let request_id = Uuid::new_v4();

// Thread through all operations
#[instrument(fields(request_id = %request_id))]
async fn handle_request(request_id: Uuid, req: Request) -> Response {
    let user = fetch_user(&req.user_id).await?;
    // All child spans inherit request_id
    process_order(user, &req.order).await
}

// In logs, filter by request_id to see full trace
```

### Pattern 3: Structured Logging

**What:** Key-value pairs, not string interpolation
**When:** All logging

```rust
// AVOID: String interpolation
info!("User {} created order {} for ${}", user_id, order_id, amount);

// PREFER: Structured fields
info!(
    user_id = %user_id,
    order_id = %order_id,
    amount_cents = amount,
    currency = "USD",
    "order_created"
);

// Now you can query:
// - All orders for user X
// - All orders over $100
// - Average amount by currency
```

### Pattern 4: State Transition Logging

**What:** Log every state change with before/after
**When:** State machines, workflows, status changes

```rust
#[instrument(skip(self), fields(
    entity_id = %self.id,
    from_state = ?self.state,
))]
fn transition(&mut self, event: Event) -> Result<(), TransitionError> {
    let from = self.state.clone();
    let to = self.state.apply(event)?;

    info!(
        to_state = ?to,
        event = ?event,
        transition_time_ms = elapsed.as_millis(),
        "state_transition"
    );

    self.state = to;
    Ok(())
}
```

### Pattern 5: Error Context

**What:** Capture full context when errors occur
**When:** All error paths

```rust
// AVOID: Generic error
error!("Request failed");

// PREFER: Full context
error!(
    request_id = %request_id,
    user_id = %user_id,
    operation = "create_order",
    error_type = "validation",
    error_message = %e.to_string(),
    error_chain = ?e.source(),
    input_order_id = %order.id,
    input_items_count = order.items.len(),
    "order_creation_failed"
);
```

### Pattern 6: Timing and Performance

**What:** Measure duration of operations
**When:** External calls, expensive operations

```rust
// Using tracing spans for automatic timing
#[instrument(name = "db_query", skip(query))]
async fn execute_query(&self, query: &str) -> Result<Rows> {
    // Span automatically records duration
    self.pool.query(query).await
}

// Manual timing for custom metrics
let start = Instant::now();
let result = expensive_operation().await;
info!(
    operation = "expensive_op",
    duration_ms = start.elapsed().as_millis(),
    result_size = result.len(),
    "operation_completed"
);
```

### Pattern 7: High-Cardinality Fields

**What:** Fields with many possible values (user IDs, request IDs)
**When:** Any field useful for debugging specific instances

```rust
// High-cardinality (good for debugging)
info!(
    user_id = %user.id,           // Millions of values
    session_id = %session.id,     // Unique per session
    request_id = %request_id,     // Unique per request
    feature_flag = %flag_value,   // Varies per user
    "user_action"
);

// Low-cardinality only (bad for debugging)
info!(
    http_method = "POST",         // Few possible values
    status_code = 200,            // Limited set
    "request_completed"
);
// Can't filter to specific user or request!
```

## Anti-Patterns

- **Stringly-typed logs:** Can't query structured data
- **Log levels as only filter:** Use dimensions instead
- **Metrics without events:** Can't debug specific incidents
- **Missing correlation:** Can't trace across services
- **PII in logs:** Security and compliance risk

## Checklist

- [ ] Correlation IDs in all requests
- [ ] Structured key-value logging
- [ ] Wide events with high-cardinality fields
- [ ] State transitions logged with before/after
- [ ] Error context captures full state
- [ ] Timing on external calls
- [ ] Sensitive data redacted

## Claude Code Integration
**Command:** /instrument
**Input:** Function or module
**Output:** Instrumentation recommendations

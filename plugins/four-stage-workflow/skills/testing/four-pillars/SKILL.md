---
name: four-pillars
description: A good test balances four qualities: protection against regressions, resistance to refactoring, fast feedback, and maintainability.
---

# Four Pillars of Good Tests

## Source
- Book: Unit Testing Principles, Practices, and Patterns
- Author: Vladimir Khorikov
- Key Concept: Balance four qualities for valuable tests

## Core Principle
A good test balances four qualities: protection against regressions, resistance to refactoring, fast feedback, and maintainability.

## Decision Framework

```
Evaluating test quality:
├── Protection Against Regressions
│   └── Does it catch bugs when code changes?
├── Resistance to Refactoring
│   └── Does it survive internal changes that preserve behavior?
├── Fast Feedback
│   └── Can you run it frequently during development?
└── Maintainability
    └── Is it easy to understand and modify?
```

## The Four Pillars

### Pillar 1: Protection Against Regressions

**What:** Test catches bugs when code is modified
**Measure:** How much code is exercised × complexity of that code

```rust
// Good: Tests complex business logic
#[test]
fn calculate_discount_applies_tiered_rates() {
    let order = Order::new(vec![item(100.0), item(200.0)]);
    let discount = calculate_discount(&order, CustomerTier::Gold);
    assert_eq!(discount, 45.0); // 15% of 300
}

// Weak: Tests trivial code
#[test]
fn getter_returns_value() {
    let user = User::new("test");
    assert_eq!(user.name(), "test"); // Trivial, low value
}
```

### Pillar 2: Resistance to Refactoring

**What:** Test doesn't break when internals change
**Key:** Test behavior, not implementation

```rust
// FRAGILE: Tests implementation details
#[test]
fn user_service_calls_repository_save() {
    let mock_repo = MockRepository::new();
    mock_repo.expect_save().times(1);

    service.create_user("test");
    mock_repo.verify();
}
// Breaks if we batch saves or use different persistence

// ROBUST: Tests observable behavior
#[test]
fn created_user_can_be_retrieved() {
    service.create_user("test");
    let user = service.get_user("test");
    assert!(user.is_some());
}
// Survives any internal refactoring
```

### Pillar 3: Fast Feedback

**What:** Tests run quickly enough to run often
**Target:** Unit tests < 10ms each, full suite < 10 seconds

```rust
// FAST: In-memory, no I/O
#[test]
fn parse_config_extracts_values() {
    let config = parse_config(r#"{"port": 8080}"#);
    assert_eq!(config.port, 8080);
}

// SLOW: Network/disk I/O
#[test]
fn api_returns_users() {
    let response = client.get("/users").await;
    // Seconds to run, dependency on external service
}
```

### Pillar 4: Maintainability

**What:** Easy to understand and modify
**Measure:** Size and complexity of test code

```rust
// HARD TO MAINTAIN: Complex setup, unclear intent
#[test]
fn test_thing() {
    let a = Factory::create_with_defaults();
    a.set_x(1).set_y(2).set_z(3);
    let b = Processor::new(Config { /* 20 fields */ });
    let c = a.process(b.configure().unwrap());
    assert!(c.is_ok());
}

// MAINTAINABLE: Clear arrange-act-assert
#[test]
fn order_with_valid_items_calculates_total() {
    // Arrange
    let order = Order::with_items(vec![
        item("Widget", 10.00),
        item("Gadget", 20.00),
    ]);

    // Act
    let total = order.calculate_total();

    // Assert
    assert_eq!(total, 30.00);
}
```

## Trade-offs

You can't maximize all four. Choose based on test type:

| Test Type | Regression | Refactoring | Speed | Priority |
|-----------|------------|-------------|-------|----------|
| Unit | Medium | High | High | Speed + Refactoring |
| Integration | High | Medium | Medium | Regression + Refactoring |
| E2E | High | High | Low | Regression |

## Patterns

### Pattern 1: Test Behavior, Not Implementation

Focus on:
- What the code does (observable output)
- Not how it does it (internal steps)

### Pattern 2: Arrange-Act-Assert

```rust
#[test]
fn descriptive_test_name() {
    // Arrange: Set up preconditions
    let input = create_valid_input();

    // Act: Execute the behavior under test
    let result = system_under_test.do_something(input);

    // Assert: Verify expected outcome
    assert_eq!(result, expected_outcome);
}
```

### Pattern 3: One Assertion Per Concept

```rust
// Multiple asserts OK if testing one logical concept
#[test]
fn user_registration_creates_complete_profile() {
    let user = register("test@example.com", "password");

    assert_eq!(user.email, "test@example.com");
    assert!(user.password_hash.is_some());
    assert!(user.created_at <= Utc::now());
    // All assert the same concept: "complete profile"
}
```

## Anti-Patterns

- **Testing implementation:** Mocking internals excessively
- **Flaky tests:** Non-deterministic results
- **Slow unit tests:** I/O in what should be fast tests
- **Test interdependence:** Tests that must run in order
- **Over-specification:** Asserting too many details

## Checklist

- [ ] Does this test catch regressions in important code?
- [ ] Will it survive refactoring that preserves behavior?
- [ ] Is it fast enough to run frequently?
- [ ] Can another developer understand it quickly?
- [ ] Does it test behavior, not implementation?

## Claude Code Integration
**Command:** /test-type
**Input:** Code to test
**Output:** Test strategy based on four pillars analysis

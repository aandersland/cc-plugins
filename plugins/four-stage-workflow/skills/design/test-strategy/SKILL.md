---
name: test-strategy
description: Framework for creating test strategies during feature design.
---

# Test Strategy

Framework for defining test requirements before implementation begins. Testing is a first-class concern in the `/design` workflow.

## Core Principle

Define what tests prove the feature works BEFORE writing implementation code.

## Test Strategy Process

### 1. Identify Test Categories

For every feature, identify tests in these categories:

| Category | What to Test | Priority |
|----------|--------------|----------|
| **Happy Path** | Feature works correctly under normal conditions | Required |
| **Error Path** | Feature handles errors gracefully | Required |
| **Edge Cases** | Boundary conditions and limits | Required |
| **Integration** | Feature works with other components | As needed |

### 2. Choose Test Types

| What You're Testing | Test Type | Speed |
|---------------------|-----------|-------|
| Pure business logic (no dependencies) | Unit | Fast |
| Database/file operations | Integration | Medium |
| API endpoints | Integration | Medium |
| Complete user workflows | E2E | Slow |

**Decision Framework:**
```
What are you testing?
├── Pure logic with no dependencies?
│   └── Unit test
├── Integration with database/file system?
│   └── Integration test (use real DB)
├── External API interaction?
│   └── Integration test (mock or real)
└── Complete user workflow?
    └── E2E test (Playwright)
```

### 3. Define Specific Test Cases

Each test case should specify:
- **Scenario**: What conditions/inputs
- **Action**: What operation to perform
- **Expected Result**: What should happen
- **Test Type**: Unit/Integration/E2E

---

## Test Strategy Template

```markdown
## Test Strategy

### Happy Path
| Scenario | Action | Expected | Type |
|----------|--------|----------|------|
| Valid input | [operation] | Success with [result] | Unit |
| Normal workflow | [steps] | Completes successfully | Int |

### Error Paths
| Scenario | Action | Expected | Type |
|----------|--------|----------|------|
| Invalid input | [operation] | Returns error [type] | Unit |
| Missing dependency | [operation] | Graceful failure | Int |
| Network failure | [operation] | Retry/fallback | Int |

### Edge Cases
| Scenario | Action | Expected | Type |
|----------|--------|----------|------|
| Empty input | [operation] | [defined behavior] | Unit |
| Maximum size | [operation] | Handles correctly | Unit |
| Concurrent access | [operation] | No race conditions | Int |

### Integration Points
- [Component A] ↔ [Component B]: Verify [what]
```

---

## Four Pillars of Good Tests

Every test should balance:

### 1. Protection Against Regressions
Does it catch bugs when code changes?

```rust
// Good: Tests complex business logic
#[test]
fn calculate_discount_applies_tiered_rates() {
    let order = Order::new(vec![item(100.0), item(200.0)]);
    let discount = calculate_discount(&order, CustomerTier::Gold);
    assert_eq!(discount, 45.0);
}
```

### 2. Resistance to Refactoring
Does it survive internal changes that preserve behavior?

```rust
// FRAGILE: Tests implementation
mock_repo.expect_save().times(1);

// ROBUST: Tests behavior
service.create_user("test");
let user = service.get_user("test");
assert!(user.is_some());
```

### 3. Fast Feedback
Can you run it frequently during development?

- Unit tests: < 10ms each
- Full suite: < 10 seconds
- No network calls in unit tests

### 4. Maintainability
Is it easy to understand and modify?

```rust
// Clear arrange-act-assert
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

---

## Test Type Guidelines

### Unit Tests
- Test one unit of behavior in isolation
- No database, no network, no filesystem
- Very fast (< 10ms)
- Use for: calculations, parsing, validation logic

### Integration Tests
- Test component with real dependencies
- Use real database (in-memory SQLite)
- Slower (100ms - seconds)
- Use for: repositories, API handlers, file operations

### E2E Tests
- Test complete system from user perspective
- Full browser automation
- Slowest (seconds - minutes)
- Use for: critical user journeys, smoke tests

---

## Checklist

When creating test strategy:

- [ ] All happy paths have test cases
- [ ] All error modes have test cases
- [ ] Edge cases identified (empty, max, boundary)
- [ ] Test types appropriate for each case
- [ ] Test cases are specific and verifiable
- [ ] Integration points identified

---

## Common Patterns

### Testing Error Handling
```rust
#[test]
fn returns_error_on_invalid_input() {
    let result = validate_email("not-an-email");
    assert!(matches!(result, Err(ValidationError::InvalidEmail)));
}
```

### Testing State Machines
```rust
#[test]
fn transition_from_idle_to_running() {
    let mut machine = StateMachine::new(State::Idle);
    machine.transition(Event::Start);
    assert_eq!(machine.state(), State::Running);
}

#[test]
fn invalid_transition_returns_error() {
    let mut machine = StateMachine::new(State::Running);
    let result = machine.transition(Event::Start); // Already started
    assert!(result.is_err());
}
```

### Testing Async Operations
```rust
#[tokio::test]
async fn fetch_user_returns_user_when_exists() {
    let db = TestDb::new().await;
    let repo = UserRepo::new(db.pool());

    repo.create(&test_user()).await.unwrap();
    let result = repo.find_by_id("test-id").await.unwrap();

    assert!(result.is_some());
}
```

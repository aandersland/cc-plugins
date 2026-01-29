---
name: test-categorization
description: Use the right test type for what you're verifying - unit tests for logic, integration tests for boundaries, E2E tests for critical paths.
---

# Test Categorization

## Source
- Book: Unit Testing Principles, Practices, and Patterns
- Author: Vladimir Khorikov
- Key Concept: Choose test type based on what you're verifying

## Core Principle
Use the right test type for what you're verifying. Unit tests for logic, integration tests for boundaries, E2E tests for critical paths.

## Decision Framework

```
What are you testing?
├── Pure business logic (no dependencies)?
│   └── Unit Test
├── Integration with external system?
│   └── Integration Test (real dependencies)
├── Complete user workflow?
│   └── E2E Test
└── Contract with another service?
    └── Contract Test
```

## Test Types

### Unit Tests
**Scope:** Single unit of behavior in isolation
**Dependencies:** None (or simple stubs)
**Speed:** Very fast (< 10ms)
**When:** Business logic, algorithms, state transitions

```rust
#[test]
fn discount_calculator_applies_percentage() {
    let calc = DiscountCalculator::new(DiscountType::Percentage(10));
    let result = calc.apply(100.0);
    assert_eq!(result, 90.0);
}
```

### Integration Tests
**Scope:** Component + real external dependency
**Dependencies:** Real database, filesystem, etc.
**Speed:** Slower (100ms - seconds)
**When:** Verifying your code works with real infrastructure

```rust
#[tokio::test]
async fn user_repository_persists_and_retrieves() {
    let db = TestDatabase::new().await;
    let repo = UserRepository::new(db.pool());

    repo.save(&User::new("test@example.com")).await;
    let user = repo.find_by_email("test@example.com").await;

    assert!(user.is_some());
}
```

### E2E Tests
**Scope:** Complete system from user perspective
**Dependencies:** Fully deployed system
**Speed:** Slowest (seconds - minutes)
**When:** Critical user journeys, smoke tests

```rust
#[test]
fn user_can_complete_checkout_flow() {
    let browser = Browser::new();
    browser.navigate("/products");
    browser.click("#add-to-cart");
    browser.click("#checkout");
    browser.fill("#email", "test@example.com");
    browser.click("#place-order");

    assert!(browser.text_contains("Order confirmed"));
}
```

## The Testing Pyramid

```
        /\
       /  \     E2E (few)
      /----\    - Critical paths only
     /      \   - Expensive to maintain
    /--------\  Integration (some)
   /          \ - Verify boundaries
  /------------\ Unit (many)
 /              \ - Fast, focused, cheap
```

## Patterns

### Pattern 1: The London School (Mockist)
Mock all dependencies, test in complete isolation.

**Use when:**
- Complex collaborator interactions
- Need to verify specific calls
- Testing against unavailable services

### Pattern 2: The Detroit School (Classicist)
Use real dependencies when possible, mock only external systems.

**Use when:**
- Testing behavior, not interactions
- Dependencies are fast and deterministic
- Want high resistance to refactoring

### Pattern 3: Sociable vs Solitary

| Aspect | Solitary | Sociable |
|--------|----------|----------|
| Dependencies | All mocked | Real in-process |
| Speed | Fastest | Fast |
| Refactoring | Brittle | Robust |
| Isolation | Perfect | Some coupling |

**Recommendation:** Prefer sociable for domain logic, solitary for infrastructure boundaries.

### Pattern 4: Test Boundaries

```
┌─────────────────────────────────────────┐
│ Your Code                               │
│  ┌─────────────────────────────────┐   │
│  │ Domain Logic (Unit Test)        │   │
│  └─────────────────────────────────┘   │
│  ┌─────────────────────────────────┐   │
│  │ Repository (Integration Test)    │   │
│  │         ↓                        │   │
│  └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
         ↓
┌─────────────────────────────────────────┐
│ External System (Database, API, etc.)  │
└─────────────────────────────────────────┘
```

## When to Use Each Type

| Scenario | Test Type |
|----------|-----------|
| Calculate order total | Unit |
| Validate email format | Unit |
| Save user to database | Integration |
| Call external payment API | Integration (with mock API) |
| User login flow | E2E |
| Checkout process | E2E |

## Anti-Patterns

- **Ice Cream Cone:** More E2E than unit tests
- **Mocking everything:** Lose confidence in integration
- **Testing internals:** Breaks on refactoring
- **Slow unit tests:** I/O in unit tests

## Checklist

- [ ] Is this testing logic or integration?
- [ ] Am I using real deps where appropriate?
- [ ] Are unit tests fast (< 10ms each)?
- [ ] Do integration tests use real infrastructure?
- [ ] Are E2E tests limited to critical paths?

## Claude Code Integration
**Command:** /test-type
**Input:** Code to test
**Output:** Recommended test type with rationale

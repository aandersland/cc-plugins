---
description: Implementation with built-in code review
---

# /build

## Purpose

Implementation workflow for features or bug fixes with **mandatory completeness gates**. Built-in code review runs automatically. The main agent orchestrates task breakdown and execution.

## Input

One of:
- Text description of what to implement
- Output from `/design` command (approved spec)
- Diagnosis report from `/debug` command (for bug fixes)

---

## Workflow Overview

```
INPUT → TASK BREAKDOWN → IMPLEMENT → TEST GATE → INSTRUMENT GATE → WIRING GATE → ERROR GATE → CODE REVIEW → VERIFY → COMPLETE
  │           │              │            │             │              │             │             │           │         │
  │      Main agent     phase-impl    Tests pass    #[instrument]   Exports     No unwrap()   code-      cargo      Done
  │      breaks down    executes       required                     wired       on input      reviewer   test/fmt
  │      the work
```

**Review Loop**: Gate fails → fix → re-evaluate → repeat until pass

---

## Phases

### Phase 1: Task Breakdown (MAIN AGENT)

**Entry**: Input provided (text, /design spec, or /debug diagnosis)
**Actions**:

1. Parse input to understand scope
2. Identify what needs to be done
3. Break down into concrete tasks
4. Order by dependencies

**The main agent handles breakdown** — no separate work breakdown document needed.

For bug fixes from `/debug`:
- Parse diagnosis report
- Identify fix location
- Plan regression test

---

### Phase 2: Implementation (TASK LOOP)

**Entry**: Tasks identified
**Actions**:

1. Execute tasks via `phase-implementer` agent
2. Track progress
3. Handle blockers by adjusting approach

**Exit**: All implementation tasks complete

---

### Phase 3: Test Gate (BLOCKER)

**Entry**: Implementation complete
**Actions**:

1. Check test strategy exists (from /design or created)
2. Verify each required test case exists
3. Run `cargo test` to verify tests pass

**Gate: TEST_GATE**
| Check | Pass Condition | Severity |
|-------|----------------|----------|
| Required tests exist | All test cases have implementations | BLOCKER |
| Tests pass | `cargo test` succeeds | BLOCKER |
| No regression | Existing tests still pass | BLOCKER |
| Regression test (fixes) | Bug fix has regression test | BLOCKER |

**Fail Action**: Implement missing tests, fix failing tests

---

### Phase 4: Instrumentation Gate (BLOCKER)

**Entry**: Test Gate passed
**Actions**:
1. Identify public async functions in modified files
2. Check for `#[instrument]` attributes
3. Verify error paths have structured logging

**Gate: INSTRUMENT_GATE**
| Check | Pass Condition | Severity |
|-------|----------------|----------|
| Entry points instrumented | Public async fns have #[instrument] | BLOCKER |
| Errors logged with context | Error paths have structured logging | BLOCKER |
| State transitions logged | State changes emit events | WARNING |

**Fail Action**: Add instrumentation

---

### Phase 5: Wiring Gate (BLOCKER)

**Entry**: Instrumentation Gate passed
**Actions**:
1. Verify new public items are exported in mod.rs
2. Verify new API endpoints are registered in router
3. Check feature is reachable from entry point

**Gate: WIRING_GATE**
| Check | Pass Condition | Severity |
|-------|----------------|----------|
| Exports exist | New public items in mod.rs | BLOCKER |
| Routes registered | API endpoints in router | BLOCKER |
| Feature reachable | Can reach new code from entry | BLOCKER |

**Fail Action**: Add exports, register routes

---

### Phase 6: Error Handling Gate (BLOCKER)

**Entry**: Wiring Gate passed
**Actions**:

1. Find all `Result` returns in modified code
2. Verify no `unwrap()` on user input or external data
3. Check error types are properly defined
4. Verify error context is preserved

**Gate: ERROR_GATE**
| Check | Pass Condition | Severity |
|-------|----------------|----------|
| All failures handled | No unhandled Result types | BLOCKER |
| No panic on user input | No unwrap() on fallible operations | BLOCKER |
| Error types defined | Custom error enum exists | BLOCKER |
| Error context preserved | #[source] used appropriately | WARNING |

**Fail Action**: Fix error handling

---

### Phase 7: Code Review (AUTOMATIC)

**Entry**: All gates passed
**Actions**:

1. Invoke `code-reviewer` agent automatically
2. Review implementation quality, patterns, bugs
3. Generate verdict

**Built-in Review Checklist** (code-reviewer checks these):
- [ ] Tests cover happy path + error cases
- [ ] No obvious bugs or security issues
- [ ] Follows codebase patterns (reader/writer pools, typed IDs, etc.)
- [ ] Tracing/logging adequate
- [ ] Regression test added (if bug fix)
- [ ] No over-engineering beyond scope

**Output**:
```markdown
## Code Review

### Issues Found
1. [BLOCKER] {description}
2. [CONCERN] {description}

### Verdict: [APPROVE | REVISE]
```

**If REVISE:** Address blockers, re-run review.

---

### Phase 8: Verification (MANDATORY)

**Entry**: Code review passed
**Actions**:

Run all verification commands:

```bash
cargo test
cargo fmt --check
cargo build 2>&1 | grep -E "(warning|error)"
```

**Gate: VERIFICATION_GATE**
| Check | Pass Condition |
|-------|----------------|
| Tests pass | `cargo test` succeeds |
| Format clean | `cargo fmt --check` succeeds |
| Build clean | No errors, review warnings |

**Fail Action**: Fix issues, re-verify

---

### Phase 9: Completion

**Entry**: Verification passed
**Actions**:

1. Present summary
2. Offer next steps

**Output**:
```markdown
# Implementation Complete: [Name]

## Gates Passed
- [x] Test Gate ([X] tests)
- [x] Instrumentation Gate
- [x] Wiring Gate
- [x] Error Handling Gate
- [x] Code Review Gate
- [x] Verification Gate

## Summary
- **Files modified**: [count]
- **Tests added**: [count]
- **Verification**: All passing

## Verification Results
```
cargo test — [X] passed
cargo fmt --check — OK
cargo build — [warnings if any]
```

## Next Steps
1. Run `/commit` to commit changes
2. Or run `/ship` for production readiness check
```

---

## Bug Fix Mode

When input is a diagnosis report from `/debug`:

### Additional Requirements
- **Regression test is MANDATORY**
- Test must fail without fix, pass with fix
- Document what the test guards against

---

## Review Loop Mechanism

When any gate fails:

```
GATE FAIL
    │
    ├─ Identify issues
    │
    ├─ Fix issues
    │
    └─ Re-evaluate gate
        │
        ├─ PASS → Continue to next gate
        └─ FAIL → Repeat loop
```

---

## Checklist

Before marking complete:
- [ ] All tasks done
- [ ] All required tests exist and pass
- [ ] Entry points instrumented
- [ ] Components wired (exports + routes)
- [ ] No unwrap() on user input
- [ ] Code review passed
- [ ] Verification passed (test, fmt, build)

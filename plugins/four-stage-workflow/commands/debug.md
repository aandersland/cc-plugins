---
description: Unified debugging entry point (read-only diagnosis)
---

# /debug

## Purpose

**Read-only** systematic debugging workflow from error to root cause identification. Produces a diagnosis report with evidence. Does NOT fix issues — use `/build` with the diagnosis report to implement fixes.

## Input

- Error message, stack trace, or log output
- Or: Description of unexpected behavior
- Or: Failing test output
- Or: Issue/ticket reference

## Key Principle

**This command is read-only.** It analyzes, investigates, and reports. To fix the bug, run `/build` with the diagnosis report.

```
/debug → Diagnosis Report → /build → Fix Implemented
```

---

## Workflow Overview

```
CAPTURE → REPRODUCE → ANALYZE → HYPOTHESIZE → MINIMIZE → REPORT
    │          │          │           │           │         │
    │          │     Parse logs   Generate     Delta     Diagnosis
    │          │     Trace code   theories    debug      Report
    │          └────────────────────────────────────────────┘
                         (all read-only)
```

---

## Embedded Knowledge: AI-Debuggable Patterns

When analyzing code and logs, look for these patterns:

### What Makes Code Debuggable

1. **Structured Error Context**: Typed errors with context fields, not string messages
2. **State Transition Logging**: Before/after state with trigger information
3. **Correlation Threading**: Request IDs flow through all operations
4. **Decision Logging**: Decisions logged with data that informed them
5. **Error Chain Preservation**: Source errors preserved with #[source]

### What to Look For in Logs

- Correlation IDs to trace requests
- State transitions (from_state → to_state)
- Decision points with reasons
- Error chains showing root cause
- Timing information

### Common Bug Patterns

| Pattern | Symptoms | Investigation |
|---------|----------|---------------|
| Race condition | Intermittent, timing-dependent | Look for shared state, async boundaries |
| Resource exhaustion | Gradual degradation | Check pool sizes, connection counts |
| Null/None handling | Crashes on specific data | Trace data flow, find unchecked unwrap() |
| State corruption | Wrong behavior after sequence | Find state transitions, check invariants |
| Timeout cascade | Sudden failures under load | Check timeout configurations, backpressure |

---

## Embedded Knowledge: Delta Debugging

When minimizing reproduction cases:

### Binary Search Process

1. Start with full failing case
2. Remove half the elements
3. If still fails → keep smaller version
4. If passes → restore and try other half
5. Repeat until minimal

### What to Minimize

- Input data (find smallest input that fails)
- Code path (find shortest path to failure)
- Timing (find minimum delay/sequence)
- Configuration (find essential config)

---

## Phases

### Phase 1: Issue Capture

**Entry**: User provides bug report/error
**Actions**:
1. Parse the issue report
2. Extract: error message, stack trace, context
3. Document expected vs actual behavior
4. Identify affected component

**Output**:
```markdown
## Issue Capture

### Problem
**Expected**: [What should happen]
**Actual**: [What happens instead]
**Severity**: [Critical | High | Medium | Low]

### Error Details
```
[Error message / stack trace]
```

### Context
- **Component**: [Affected area]
- **Trigger**: [What causes it]
- **Frequency**: [Always | Sometimes | Rare]
- **First Seen**: [When]
- **Recent Changes**: [If known]

### Reproduction Steps
1. [Step 1]
2. [Step 2]
3. [Observe error]
```

---

### Phase 2: Reproduction

**Entry**: Issue captured
**Actions**:
1. Attempt to reproduce the issue
2. Document reproduction reliability
3. Note environmental factors

**Output**:
```markdown
## Reproduction

**Status**: REPRODUCED | INTERMITTENT | NOT REPRODUCED

### Verified Steps
1. [Step with specific commands]
2. [Step]
3. Error: [exact output]

### Environmental Factors
- [OS, versions, config that matters]
```

---

### Phase 3: Log Analysis

**Entry**: Issue reproducible (or documented as intermittent)
**Actions**:
1. Parse structured log output
2. Identify correlation IDs and request traces
3. Find error events and their context
4. Trace state transitions
5. Note timeline of events

**Output**:
```markdown
## Log Analysis

### Key Events (Chronological)

| Timestamp | Event | State | Notes |
|-----------|-------|-------|-------|
| [time] | [event] | [state] | [observation] |
| [time] | [ERROR] | [state] | ← FAILURE POINT |

### Error Chain
```
[Root cause]
└── [Caused by]
    └── [Which led to]
        └── [Resulting in visible error]
```

### Correlation Context
- Request ID: [if available]
- User/Session: [if available]
- Component: [where it originated]
```

---

### Phase 4: Hypothesis Generation

**Entry**: Logs analyzed
**Actions**:
1. Generate ranked hypotheses based on evidence
2. Note evidence for and against each
3. Plan verification steps for each hypothesis

**Output**:
```markdown
## Hypotheses

### H1: [Most Likely] ⭐ — Confidence: [High/Medium/Low]
**Theory**: [What might be wrong]
**Evidence For**:
- [Supporting observation]
**Evidence Against**:
- [Contradicting observation]
**Verification**: [How to test this hypothesis]

### H2: [Alternative] — Confidence: [Medium]
**Theory**: [Alternative explanation]
**Evidence For**: [...]
**Verification**: [How to test]

### H3: [Long Shot] — Confidence: [Low]
**Theory**: [Less likely but possible]
**Verification**: [How to test]
```

---

### Phase 5: Investigation

**Entry**: Hypotheses generated
**Actions**:
1. Systematically verify or rule out hypotheses
2. Start with highest confidence hypothesis
3. Trace code paths
4. Gather evidence
5. Apply delta debugging if applicable

**Output**:
```markdown
## Investigation

### H1 Verification: [CONFIRMED | RULED OUT]
**Steps Taken**:
1. [What was checked]
2. [What was found]

**Evidence**:
- [Specific finding at file:line]

### Minimization (if applicable)
**Original Case**: [Size/complexity]
**Minimal Case**:
```
[Minimal code/steps to trigger bug]
```
**Essential Elements**:
- [Element 1]: Required
- [Element 2]: Required
- [Element 3]: Removed (not needed)
```

---

### Phase 6: Diagnosis Report

**Entry**: Root cause identified
**Actions**:
1. Document the root cause with evidence
2. Explain why it wasn't obvious
3. Recommend fix approach
4. Specify regression test requirements

**Exit**: Complete diagnosis ready for `/build`

**Output**:
```markdown
# Diagnosis Report: [Brief Description]

## Summary
**Issue**: [One-line description]
**Root Cause**: [One-line description]
**Confidence**: [High | Medium | Low]

## Root Cause

### The Bug
**Location**: `file.rs:42`

**Code**:
```rust
// The problematic code
[snippet]
```

**Why It Fails**: [Detailed explanation]

**Why It Wasn't Obvious**: [What obscured the bug]

## Evidence Trail
1. [Evidence 1 at file:line]
2. [Evidence 2 at file:line]
3. [Evidence 3 at file:line]

## Recommended Fix

### Approach
[What to change]

### Code Change
```rust
// Before
[problematic code]

// After
[fixed code]
```

### Files to Modify
- `file.rs` — [change description]

### Side Effects
- [What else this might affect]

## Regression Test

### What to Test
[What the regression test should verify]

### Test Outline
```rust
#[test]
fn regression_[issue]_[description]() {
    // Guards against: [brief description]
    // Root cause was: [one-liner]

    // Arrange
    [setup that triggers the bug]

    // Act
    [operation that was failing]

    // Assert
    [verification that bug is fixed]
}
```

## Prevention

### How to Avoid Similar Issues
- [Recommendation 1]
- [Recommendation 2]

### Detection Improvements
- [How to catch this earlier next time]

---

## Next Step
Run `/build` with this diagnosis to implement the fix.
```

---

## Checklist

Diagnosis complete when:
- [ ] Root cause identified (not just symptoms)
- [ ] Evidence trail documented with file:line references
- [ ] Reproduction path understood
- [ ] Fix approach identified
- [ ] Regression test requirements specified
- [ ] Prevention measures noted

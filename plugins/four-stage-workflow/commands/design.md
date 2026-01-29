---
description: Feature design with testing as first-class concern
---

# /design

## Purpose

Feature design workflow from idea to approved spec with test strategy. Uses Claude's plan mode for spec creation. Testing is a first-class concern addressed before implementation begins.

## Input

Feature description including:
- What the feature does
- Who it's for
- Why it's needed
- (Optional) Technical constraints

## Workflow Overview

```
IDEA → CLARIFY → TEST STRATEGY → CLARIFY → SPEC → CLARIFY → DESIGN REVIEW → TEST REVIEW → APPROVE
  │        │           │             │        │        │           │              │          │
  │    Questions   Define tests  Questions  Create  Questions  design-      Verify        /build
  │    (if any)    FIRST         (if any)   spec    (if any)   reviewer     coverage
```

**Key principle:** Test strategy comes BEFORE spec creation.

---

## Phases

### Phase 1: Idea Capture (INTERACTIVE)

**Entry**: User provides feature description
**Actions**:
1. Parse and clarify the feature request
2. Ask clarifying questions if ambiguous
3. Document problem and proposed solution

**Exit**: Clear problem statement and proposed solution

**Output**:
```markdown
## Idea Capture

### Problem
[1-2 sentences: What pain point does this solve?]

### Proposed Solution
[1-2 sentences: What we're building]

### Target Users
[Who benefits from this feature?]
```

**Clarify if needed:**
- Scope unclear? Ask.
- Multiple interpretations? Ask.
- Technical constraints unknown? Ask.

---

### Phase 2: Test Strategy (FIRST CLASS)

**Entry**: Clear problem/solution
**Actions**:
1. Define what tests prove the feature works
2. Identify happy paths, error cases, edge cases
3. Determine test types (unit/integration/e2e) for each scenario

**Exit**: Complete test strategy

**Clarify if needed:**
- Error handling behavior unclear? Ask.
- Edge case behavior uncertain? Ask.
- Integration boundaries unclear? Ask.

**Output**:
```markdown
## Test Strategy

| Category | Test Case | Type | Priority |
|----------|-----------|------|----------|
| Happy Path | [scenario] | Unit/Int/E2E | Required |
| Error | [failure mode] | Unit/Int | Required |
| Edge Case | [boundary] | Unit | Required |

### Integration Points to Test
- [External systems/modules to verify]
```

**Gate: TEST_STRATEGY_GATE**
| Check | Pass Condition |
|-------|----------------|
| Happy path defined | At least 1 happy path test case |
| Error cases identified | All failure modes have test cases |
| Edge cases documented | Boundary conditions identified |
| Test types assigned | Unit/Integration/E2E for each test |

**Cannot proceed without test strategy.**

---

### Phase 3: Spec Creation (PLAN MODE)

**Entry**: Test strategy complete
**Actions**:
1. Enter Claude plan mode
2. Create spec document with:
   - Problem statement
   - Solution approach
   - Technical decisions
   - Files to modify/create
   - Test strategy (from Phase 2)
3. Present spec for review

**Clarify if needed:**
- Technical approach options? Ask for preference.
- Scope boundaries fuzzy? Ask.

**Output**: Plan file created via Claude's plan mode

---

### Phase 4: Design Review (AUTOMATIC)

**Entry**: Spec complete
**Actions**:
1. Invoke `design-reviewer` agent automatically
2. Review spec for completeness, architecture fit, scope creep
3. Generate verdict

**Built-in Review Checklist** (design-reviewer checks these):
- [ ] Scope creep check - Does this include things not requested?
- [ ] Architecture fit - Follows codebase patterns?
- [ ] Edge cases identified - Boundaries documented?
- [ ] State complexity assessed - State machine needed?
- [ ] Error handling specified - Failure modes clear?
- [ ] Dependencies clear - What this depends on?

**Output**:
```markdown
## Design Review

### Issues Found
1. [BLOCKER] {description}
2. [CONCERN] {description}
3. [NOTE] {description}

### Verdict: [APPROVE | REVISE]
```

**If REVISE:** Address blockers, return to relevant phase.

---

### Phase 5: Test Review (AUTOMATIC)

**Entry**: Design review passed
**Actions**:
1. Invoke `design-reviewer` agent with test focus
2. Review test strategy for completeness
3. Verify all scenarios are testable
4. Generate verdict

**Built-in Review Checklist:**
- [ ] All happy paths covered
- [ ] Error scenarios have tests
- [ ] Edge cases addressed
- [ ] Test types appropriate
- [ ] Tests are specific and verifiable

**Output**:
```markdown
## Test Review

### Coverage Assessment
- Happy paths: [✅ Covered | ⚠️ Gaps | ❌ Missing]
- Error cases: [✅ Covered | ⚠️ Gaps | ❌ Missing]
- Edge cases: [✅ Covered | ⚠️ Gaps | ❌ Missing]

### Gaps Found
1. [Missing test scenario]

### Verdict: [APPROVE | REVISE]
```

**If REVISE:** Add missing test cases, re-review.

---

### Phase 6: Final Approval (INTERACTIVE)

**Entry**: All reviews passed
**Actions**:
1. Present complete spec with test strategy
2. Request explicit approval to proceed

**Exit**: User approves, ready for `/build`

**Output**:
```markdown
# Design Complete: [Feature Name]

## Summary
- **Problem**: [one-line]
- **Solution**: [one-line]
- **Test Cases**: [count]
- **Files Affected**: [count]

## Reviews Passed
- [x] Design Review
- [x] Test Review

## Next Step
Run `/build` to begin implementation.

**Awaiting approval. Type 'approved' or provide feedback.**
```

---

## Checklist

Before approval:
- [ ] Test strategy complete with required cases
- [ ] Problem and solution are specific and clear
- [ ] Design review passed (no blockers)
- [ ] Test review passed (adequate coverage)
- [ ] User explicitly approved

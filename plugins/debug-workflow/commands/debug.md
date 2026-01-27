---
name: debug
description: Systematic debugging workflow with hypothesis-driven investigation
argument-hint: <test command or error description>
---

# Debug Workflow

A 5-phase systematic debugging workflow that investigates issues and produces actionable diagnosis reports. This workflow **does not make code changes** - it investigates, diagnoses, and reports findings for you to implement.

## Input

The user provides one of:
- A failing test command (e.g., `cargo test`, `pytest`, `npm test`)
- An error message or stack trace
- A description of unexpected behavior

## Phase 1: Symptom Collection

**Goal**: Understand what's failing and how.

### Actions

1. **Collect the Error Context**
   - If given a test command, run it to capture current output
   - Parse error messages, stack traces, and assertion failures
   - Identify the immediate failure point

2. **Launch Symptom Collector Agent**
   Use the `symptom-collector` agent to analyze the failure and produce a structured symptom profile.

3. **Review Symptom Profile**
   Present the symptom profile to the user. Ask for any missing critical information identified by the agent.

### Gate
Proceed when: Complete symptom profile established with failure location identified.

---

## Phase 2: Codebase Exploration

**Goal**: Understand the failing code path and intended behavior.

### Actions

1. **Launch Code Explorer Agent**
   Use the `code-explorer` agent to analyze:
   - The feature/component that's failing
   - Related code paths and dependencies
   - Recent changes to relevant files

2. **Review Exploration Report**
   The agent will provide:
   - Code flow tracing from entry points
   - Key components and their responsibilities
   - Architecture patterns in use
   - Data flow through the system

3. **Build Context**
   From the exploration report, understand:
   - How data flows to the failure point
   - Dependencies and state requirements
   - Assumptions the code makes

### Gate
Proceed when: Clear understanding of intended behavior vs actual behavior.

---

## Phase 3: Hypothesis Formation

**Goal**: Generate ranked theories about the root cause.

### Actions

1. **Launch Hypothesis Generator Agent**
   Use the `hypothesis-generator` agent with:
   - The symptom profile from Phase 1
   - The codebase context from Phase 2

2. **Present Hypotheses**
   Show the ranked hypotheses to the user. Include:
   - Description of each potential cause
   - Confidence scores
   - What evidence supports each hypothesis

3. **Gather User Input**
   Ask if the user:
   - Has additional suspicions to add
   - Wants to prioritize certain hypotheses
   - Has context that would eliminate any hypotheses

### Gate
Proceed when: Prioritized list of hypotheses ready for investigation.

---

## Phase 4: Investigation

**Goal**: Systematically validate or invalidate hypotheses.

### Actions

1. **Launch Root Cause Tracer Agents**
   For the top 2-3 hypotheses, use the `root-cause-tracer` agent to:
   - Trace execution paths
   - Examine state at key points
   - Gather evidence for/against each hypothesis

2. **Evaluate Results**
   - Which hypotheses were confirmed?
   - Which were refuted and why?
   - Is additional investigation needed?

3. **Iterate if Necessary**
   If no hypothesis confirmed:
   - Generate new hypotheses based on what was learned
   - Return to hypothesis formation with new context

### Gate
Proceed when: Root cause identified with supporting evidence.

---

## Phase 5: Diagnosis Report

**Goal**: Deliver actionable diagnosis with prevention recommendations.

### Actions

1. **Launch Similar Issue Scanner**
   Use the `similar-issue-scanner` agent to:
   - Find the same bug pattern elsewhere in the codebase
   - Identify related code at risk
   - Suggest tests to prevent regression

2. **Compile Final Report**
   Present the complete diagnosis to the user:

### Report Format

```
## Root Cause Summary
- **What's broken**: <1-2 sentence summary>
- **Location**: `file:line`
- **Why it fails**: <mechanism explanation>

## Recommended Fix
<description of what needs to change, without implementing>

**Potential side effects to watch for:**
- <consideration 1>
- <consideration 2>

## Similar Issues Found
<output from similar-issue-scanner agent>

## Suggested Tests
<test recommendations to prevent regression>

## Investigation Trail
<summary of hypotheses considered, evidence gathered, files examined>
```

---

## Important Notes

- **No Code Changes**: This workflow investigates only. You implement fixes based on the diagnosis.
- **Evidence Required**: Every diagnosis is backed by traced evidence, not guesses.
- **Similar Issue Scan**: After finding one bug, we scan for the same pattern elsewhere.
- **Test Suggestions**: Each diagnosis includes recommendations for regression tests.

## Example Usage

User: "My tests are failing: `pytest tests/test_auth.py::test_login`"

The workflow will:
1. Run the test command and parse the failure
2. Read the test and authentication code
3. Form hypotheses about why login fails
4. Trace through the code to find the root cause
5. Report the diagnosis with fix recommendations

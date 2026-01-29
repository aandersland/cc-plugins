# Four-Stage Workflow Plugin

A comprehensive workflow plugin for Claude Code that provides structured processes for design, implementation, debugging, and production readiness.

## Overview

This plugin implements a complete software development workflow with four interconnected stages:

```
/design → /build → /debug → /ship
    │         │         │        │
    │         │         │        └─ Production readiness check
    │         │         └─ Read-only diagnosis
    │         └─ Implementation with gates
    └─ Feature design with test strategy
```

Each command can be used independently or as part of a connected flow.

## Installation

Add this plugin to your Claude Code configuration:

```json
{
  "plugins": [
    "/path/to/cc-plugins/plugins/four-stage-workflow"
  ]
}
```

## Commands

### `/design` — Feature Design

Design workflow from idea to approved spec with test strategy as a first-class concern.

```
/design Add user authentication with OAuth2 support
```

**Workflow:**
```
IDEA → CLARIFY → TEST STRATEGY → SPEC → DESIGN REVIEW → TEST REVIEW → APPROVE
```

**Key principle:** Test strategy is defined BEFORE implementation planning.

**Output:** Approved spec with complete test strategy, ready for `/build`.

---

### `/build` — Implementation

Implementation workflow with mandatory completeness gates and built-in code review.

```
/build                    # Uses approved /design spec
/build Add caching layer  # Direct implementation
```

**Workflow:**
```
INPUT → TASK BREAKDOWN → IMPLEMENT → TEST GATE → INSTRUMENT GATE → WIRING GATE → ERROR GATE → CODE REVIEW → VERIFY
```

**Gates (all mandatory):**
- Test Gate — All required tests exist and pass
- Instrumentation Gate — Public async functions have `#[instrument]`
- Wiring Gate — Exports and routes registered
- Error Handling Gate — No `unwrap()` on user input, proper error types

**Bug Fix Mode:** When input is a diagnosis from `/debug`, regression test is mandatory.

---

### `/debug` — Read-Only Diagnosis

Systematic debugging workflow that produces a diagnosis report without making code changes.

```
/debug pytest tests/test_auth.py::test_login
/debug The API returns 500 errors on large payloads
```

**Workflow:**
```
CAPTURE → REPRODUCE → ANALYZE → HYPOTHESIZE → MINIMIZE → REPORT
```

**Key principle:** This command investigates only. Use `/build` with the diagnosis report to implement fixes.

**Output:** Diagnosis report with root cause, evidence trail, recommended fix, and regression test requirements.

---

### `/ship` — Production Readiness

Pre-release validation combining stability patterns, observability coverage, and security checking.

```
/ship user-auth-feature
/ship full app
```

**Workflow:**
```
STABILITY AUDIT → OBSERVABILITY AUDIT → SECURITY SCAN → PRODUCTION REVIEW → VERDICT
```

**Checks:**
- **Stability:** Timeouts, retries, circuit breakers, graceful degradation
- **Observability:** Correlation IDs, structured logging, tracing, health checks
- **Security:** No exposed secrets, input validation, auth coverage
- **Rollback:** Detection, trigger, process, verification

**Output:** READY or BLOCKED verdict with categorized issues.

## Agents

| Agent | Purpose |
|-------|---------|
| `design-reviewer` | Reviews specs for completeness, architecture fit, and scope creep |
| `codebase-explorer` | Explores codebase to understand patterns and structure |
| `code-reviewer` | Reviews implementation quality, patterns, and bugs |
| `test-explorer` | Explores existing tests and testing patterns |
| `phase-implementer` | Executes implementation tasks |
| `production-reviewer` | Reviews production readiness and rollback plans |

## Skills

The plugin includes 23 skills across 6 domains:

| Domain | Skills |
|--------|--------|
| **Architecture** | state-machines, deep-modules, error-strategy, complexity-signals |
| **Design** | test-strategy, data-viz, ui-conventions, ui-patterns, usability-heuristics |
| **Core** | ai-debuggable |
| **Production** | observability-design, stability-patterns, release-checklist |
| **Rust** | result-combinators, tracing-patterns, leptos-state, error-types |
| **Testing** | four-pillars, test-categorization |

## Design Philosophy

- **Test-first:** Test strategy is defined before implementation begins
- **Gate-driven:** Mandatory quality gates prevent shipping incomplete work
- **Read-only debugging:** Diagnosis is separate from fixing, ensuring thorough investigation
- **Production-aware:** Ship readiness is explicitly verified, not assumed
- **Evidence-based:** All reviews and diagnoses require concrete evidence

## Workflow Integration

The commands are designed to flow together:

1. **Feature development:** `/design` → `/build` → `/ship`
2. **Bug fixes:** `/debug` → `/build` (with diagnosis) → `/ship`
3. **Quick fixes:** `/build` directly (for well-understood changes)
4. **Pre-release:** `/ship` on any existing code

## License

See repository LICENSE file.

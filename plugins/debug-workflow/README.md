# Debug Workflow Plugin

A systematic debugging plugin for Claude Code that uses hypothesis-driven investigation to diagnose issues.

## Overview

This plugin provides a structured 5-phase debugging workflow that:

1. **Collects symptoms** - Parses errors, test failures, and gathers context
2. **Explores the codebase** - Understands the failing code path
3. **Forms hypotheses** - Generates ranked theories about root causes
4. **Investigates** - Validates hypotheses through code tracing
5. **Reports diagnosis** - Delivers actionable findings with fix recommendations

**Important**: This plugin **investigates only** - it does not make code changes. You implement fixes based on the diagnosis.

## Installation

Add this plugin to your Claude Code configuration:

```bash
# In your project's .claude/settings.json or global settings
{
  "plugins": [
    "/path/to/cc-plugins/plugins/debug-workflow"
  ]
}
```

## Usage

### Starting a Debug Session

Invoke the debug command with your failing test or error:

```
/debug pytest tests/test_auth.py::test_login
```

Or describe the issue:

```
/debug The API returns 500 errors when processing large payloads
```

### What You'll Get

1. **Structured symptom analysis** - Error classification, stack trace parsing, reproduction steps
2. **Ranked hypotheses** - Possible causes with confidence scores
3. **Evidence-based diagnosis** - Root cause identified through code tracing
4. **Similar issue scan** - Other locations with the same bug pattern
5. **Test suggestions** - Recommended tests to prevent regression

### Example Output

```
## Root Cause Summary
- **What's broken**: Null check missing before accessing user.email
- **Location**: `src/auth/login.py:47`
- **Why it fails**: When user lookup returns None, the code attempts to access .email

## Recommended Fix
Add a null check after user lookup, returning appropriate error before accessing properties.

## Similar Issues Found
- `src/auth/register.py:82` - HIGH RISK - Same pattern with user object
- `src/profile/update.py:34` - MEDIUM RISK - Similar but has partial guard

## Suggested Tests
- Add test for login with non-existent user
- Add test for register with duplicate email
```

## Agents

| Agent | Purpose |
|-------|---------|
| `symptom-collector` | Gathers error context, parses test output |
| `hypothesis-generator` | Forms ranked theories about root causes |
| `root-cause-tracer` | Validates hypotheses via code tracing |
| `similar-issue-scanner` | Finds same pattern elsewhere, suggests tests |

## Design Philosophy

- **Hypothesis-driven**: Form and test theories rather than trial-and-error
- **Evidence-based**: Require proof to confirm root cause
- **Systematic**: Follow structured process even for "obvious" bugs
- **Preventive**: Scan for similar issues and recommend regression tests

## Comparison with Feature-Dev

| Aspect | Feature-Dev | Debug Workflow |
|--------|-------------|----------------|
| Starting point | Feature request | Bug report / test failure |
| Goal | Build something new | Diagnose root cause |
| Output | Working feature | Diagnosis report |
| Code changes | Yes | No (investigation only) |

## License

See repository LICENSE file.

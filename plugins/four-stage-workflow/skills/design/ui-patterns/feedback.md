---
name: feedback
description: Feedback patterns ensuring users never wonder if an action worked through immediate, clear responses.
---

# Feedback Patterns

## Source
- Book: Designing Interfaces (3rd Edition)
- Author: Jenifer Tidwell
- Key Chapters: 5 (Feedback and Feedforward)

## Core Principle
Users should never wonder "did that work?" Every action should produce immediate, clear feedback.

## Patterns

### Pattern 1: Loading States
**When:** Any async operation
**Apply:** Immediate acknowledgment + progress

```
Instant feedback:       Progress indicator:
┌──────────────┐        ┌──────────────────────┐
│ [Button    ] │   →    │ [Loading...  ◠◡◠   ] │
└──────────────┘        └──────────────────────┘

For long operations:
┌────────────────────────────────────────┐
│ Uploading file...                      │
│ ████████████░░░░░░░░░░  60%           │
│ 3 of 5 MB • About 10 seconds left     │
└────────────────────────────────────────┘
```

Rules:
- < 100ms: No indicator needed
- 100ms-1s: Spinner
- > 1s: Progress bar with estimate
- Show what's happening, not just "loading"

### Pattern 2: Success Confirmation
**When:** Action completes successfully
**Apply:** Visible, temporary confirmation

```
Toast notification:
┌────────────────────────────────┐
│ ✓ Changes saved                │  ← Auto-dismiss after 3-5s
└────────────────────────────────┘

Inline confirmation:
┌────────────────────────────────┐
│ [Save]  ✓ Saved                │  ← Appears next to trigger
└────────────────────────────────┘
```

Rules:
- Position near the trigger action
- Auto-dismiss (but allow manual dismiss)
- Don't interrupt user flow
- Green/checkmark for success

### Pattern 3: Error Messages
**When:** Action fails
**Apply:** Specific, actionable error

```
Good:
┌────────────────────────────────────────────┐
│ ✗ Unable to save changes                   │
│   The file is too large (max 5MB).        │
│   [Try a smaller file] [Cancel]            │
└────────────────────────────────────────────┘

Bad:
┌────────────────────────────────────────────┐
│ Error: Operation failed                    │  ← Not helpful
└────────────────────────────────────────────┘
```

Rules:
- Say what went wrong
- Say how to fix it
- Provide actions (retry, alternative)
- Don't blame the user
- Persist until acknowledged

### Pattern 4: Confirmation Dialogs
**When:** Destructive or irreversible actions
**Apply:** Clear consequences + explicit confirmation

```
┌────────────────────────────────────────────┐
│ Delete "Project Alpha"?                    │
│                                            │
│ This will permanently delete the project   │
│ and all 47 tasks. This cannot be undone.   │
│                                            │
│           [Cancel]  [Delete Project]       │
│                      └── Red, destructive  │
└────────────────────────────────────────────┘
```

Rules:
- State the action and consequences
- Destructive button should be red
- Default to safe option (Cancel)
- Don't use confirmation for reversible actions

### Pattern 5: Progress Indicators
**When:** Multi-step processes
**Apply:** Show steps and current position

```
Linear progress:
Step 1 of 3: Uploading files...
━━━━━━━━━━━━━━━━━━━━░░░░░░░░░░  67%

Multi-step wizard:
  ①───────②───────③───────④
Upload   Process  Review   Done
  ●────────●────────○────────○
           ↑ Current step
```

### Pattern 6: Optimistic UI
**When:** Action will likely succeed
**Apply:** Show success immediately, rollback if fails

```
1. User clicks "Like"
2. UI immediately shows liked state
3. Request sent to server
4. If fails: Revert UI + show error

Benefits:
- Feels instant (no waiting)
- Most actions succeed anyway
- Graceful handling of failures
```

### Pattern 7: Empty States
**When:** No content to display
**Apply:** Explain and guide

```
┌────────────────────────────────────────────┐
│                  📭                        │
│           No notifications                 │
│                                            │
│   You're all caught up! New notifications  │
│   will appear here.                        │
└────────────────────────────────────────────┘
```

### Pattern 8: Skeleton Screens
**When:** Loading content-heavy pages
**Apply:** Show structure before data

```
Loading:
┌────────────────────────────────────────────┐
│ ████████████████                           │
│ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░              │
│ ░░░░░░░░░░░░░░░░░░░░                      │
│                                            │
│ ████████████                               │
│ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░        │
└────────────────────────────────────────────┘

Gray boxes where content will appear
Better than blank screen or spinner
```

## Anti-Patterns

- **Silent Failures:** Action fails with no indication
- **Modal Overload:** Confirmation for every action
- **Cryptic Errors:** "Error 500" or technical jargon
- **Success Buried:** User must hunt for confirmation
- **No Progress:** Long operation with no feedback
- **Blocking Loaders:** Full-page spinner for minor updates

## Checklist

- [ ] Every action has immediate visual feedback
- [ ] Loading states show what's happening
- [ ] Success messages are visible but not disruptive
- [ ] Error messages explain problem and solution
- [ ] Destructive actions require confirmation
- [ ] Progress shown for long operations
- [ ] Empty states are helpful, not just empty

## Claude Code Integration
**Command:** /ux-review
**Input:** Interaction or component
**Output:** Feedback pattern analysis and gaps

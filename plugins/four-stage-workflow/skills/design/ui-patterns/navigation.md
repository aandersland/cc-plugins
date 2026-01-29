---
name: navigation
description: Navigation patterns that answer where am I, where can I go, and how do I get back.
---

# Navigation Patterns

## Source
- Book: Designing Interfaces (3rd Edition)
- Author: Jenifer Tidwell
- Key Chapters: 3 (Getting Around)

## Core Principle
Navigation should answer: Where am I? Where can I go? How do I get back?

## Patterns

### Pattern 1: Global Navigation
**When:** App has 3-7 major sections
**Apply:** Persistent navigation bar

```
┌──────────────────────────────────────┐
│ Logo   Dashboard  Projects  Settings │
└──────────────────────────────────────┘
│                                      │
│         Page Content                 │
│                                      │
```

Rules:
- Highlight current section
- Keep labels short (1-2 words)
- Order by importance or frequency

### Pattern 2: Breadcrumbs
**When:** Deep hierarchical content
**Apply:** Show path from root

```
Home > Projects > Project Alpha > Settings
```

Rules:
- Use ">" or "/" as separator
- Current page is not a link
- Each item clickable back to that level

### Pattern 3: Tabs
**When:** 2-7 parallel sections within a page
**Apply:** Horizontal or vertical tabs

```
┌─────────┬─────────┬─────────┐
│ General │ Privacy │ Account │ ← Active tab connected to content
├─────────┴─────────┴─────────┤
│                             │
│     Tab Content Area        │
│                             │
└─────────────────────────────┘
```

Rules:
- Active tab visually connected to content
- Never nest tabs within tabs
- Consider pills for equal-weight options

### Pattern 4: Sidebar Navigation
**When:** Many sections or nested hierarchy
**Apply:** Vertical list with expandable sections

```
┌──────────────┬───────────────────────┐
│ Dashboard    │                       │
│ ▼ Projects   │                       │
│   Active     │     Main Content      │
│   Archived   │                       │
│ Settings     │                       │
└──────────────┴───────────────────────┘
```

Rules:
- Indicate expandable items with arrow
- Highlight current section and parents
- Collapse to icons on small screens

### Pattern 5: Command Palette
**When:** Power users need quick access
**Apply:** Keyboard-triggered search

```
┌─────────────────────────────────────┐
│ > search commands...                │
├─────────────────────────────────────┤
│ Go to Dashboard          ⌘D        │
│ New Project              ⌘N        │
│ Settings                 ⌘,        │
└─────────────────────────────────────┘
```

Trigger: Cmd+K or Cmd+P

### Pattern 6: Wizard/Stepper
**When:** Multi-step linear process
**Apply:** Progress indicator with steps

```
  (1)──────(2)──────(3)──────(4)
 Basic    Details   Review   Done
   ●─────────○─────────○─────────○

 ┌─────────────────────────────────┐
 │  Step 1: Basic Information      │
 │                                 │
 │  [Form fields...]               │
 │                                 │
 │        [Back]  [Next →]         │
 └─────────────────────────────────┘
```

Rules:
- Show all steps upfront
- Allow back navigation
- Mark completed steps

## Anti-Patterns

- **Mystery Meat Navigation:** Icons without labels
- **Hidden Navigation:** Buried in menus (vs. discoverable)
- **Deep Nesting:** More than 3-4 levels
- **Inconsistent Location:** Navigation moves between pages

## Checklist

- [ ] User always knows current location
- [ ] Major sections visible without clicking
- [ ] Back/home navigation always available
- [ ] Mobile navigation pattern appropriate (hamburger, bottom tabs)
- [ ] Active state clearly distinguished

## Claude Code Integration
**Command:** /ux-review
**Input:** Navigation structure
**Output:** Pattern recommendations and violations

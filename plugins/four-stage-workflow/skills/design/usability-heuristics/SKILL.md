---
name: usability-heuristics
description: Evaluate design using Steve Krug's "Don't Make Me Think" principles and Nielsen's 10 Usability Heuristics to ensure self-evident interfaces.
---

# Usability Heuristics

## Source
- Book: Don't Make Me Think (3rd Edition)
- Author: Steve Krug
- Key Chapters: 1-4 (Guiding Principles), 6-7 (Navigation), 9-10 (Testing)
- Additional: Nielsen's 10 Usability Heuristics

## Core Principle
Every page should be self-evident - users should be able to "get it" without expending any effort thinking about it.

## Decision Framework

```
Is this design self-evident?
├── Can users identify where they are? → Visibility of system status
├── Can users identify what they can do? → Clear affordances
├── Can users identify how to do it? → Recognition over recall
└── Do users have to think to use it?
    ├── Yes → Redesign to eliminate cognitive load
    └── No → Ship it
```

### The Trunk Test (5-Second Test)
User should answer in 5 seconds:
1. What site/app is this? (Logo/identity)
2. What page am I on? (Page title)
3. What are the major sections? (Navigation)
4. What are my options at this level? (Local navigation)
5. Where am I in the scheme of things? (Breadcrumbs)
6. How can I search? (Search box)

## Patterns

### Pattern 1: Visual Hierarchy
**When:** Laying out any page or component
**Apply:** Create clear visual hierarchy through size, color, position, and contrast
**Example:**
```
MOST IMPORTANT (largest, boldest, top)
├── Secondary information (medium size)
│   ├── Supporting details (smaller)
│   └── Related actions (styled as actions)
└── Tertiary content (smallest, lowest contrast)
```

### Pattern 2: Omit Needless Words
**When:** Writing any UI text
**Apply:** Cut text by 50%, then cut another 50%
**Example:**
```
Before: "Please enter your email address in the field below"
After:  "Email"

Before: "Click the Submit button to submit your form"
After:  "Submit"
```

### Pattern 3: Conventions Over Creativity
**When:** Designing any standard interface element
**Apply:** Use established conventions unless innovation provides clear 10x benefit
**Example:**
- Logo top-left, links to home
- Search top-right with magnifying glass icon
- Primary action = prominent button
- "X" = close
- Hamburger menu = navigation on mobile

### Pattern 4: Nielsen's Heuristics

| Heuristic | Application |
|-----------|-------------|
| H1: Visibility of System Status | Show loading states, progress, confirmations |
| H2: Match Real World | Use user's language, not jargon |
| H3: User Control | Provide undo, cancel, back |
| H4: Consistency | Same word = same meaning everywhere |
| H5: Error Prevention | Disable invalid actions, confirm destructive ones |
| H6: Recognition Over Recall | Show options, provide context |
| H7: Flexibility | Shortcuts for experts |
| H8: Aesthetic Minimalism | Every extra element competes |
| H9: Error Recovery | Plain language, precise, constructive |
| H10: Help | Searchable, task-focused, brief |

## Anti-Patterns

- **Happy Talk:** Intro text that says nothing useful ("Welcome to...")
- **Instructions That Could Be Eliminated:** If you need them, the design failed
- **Visual Noise:** Busy backgrounds, borders on everything, too many colors
- **Excise Tax:** Required steps that don't benefit the user

## Checklist

- [ ] Can I identify what this is instantly?
- [ ] Is it obvious what page I'm on?
- [ ] Can I see my main navigation options?
- [ ] Is there a clear visual hierarchy?
- [ ] Would removing any element make it better?
- [ ] Does it pass the 5-second trunk test?

## Claude Code Integration
**Command:** /ux-review
**Input:** Component name, screenshot, or code
**Output:** Heuristic evaluation with specific violations and fixes

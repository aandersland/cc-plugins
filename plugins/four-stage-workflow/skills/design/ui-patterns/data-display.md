---
name: data-display
description: Data display patterns that match the user's mental model and task requirements.
---

# Data Display Patterns

## Source
- Book: Designing Interfaces (3rd Edition)
- Author: Jenifer Tidwell
- Key Chapters: 6 (Showing Complex Data)

## Core Principle
Display data in the format that best matches the user's mental model and task.

## Patterns

### Pattern 1: Data Tables
**When:** Comparing multiple items with same attributes
**Apply:** Rows for items, columns for attributes

```
┌──────────────────────────────────────────────┐
│ Name          │ Status   │ Created   │ Actions│
├──────────────────────────────────────────────┤
│ Project Alpha │ Active   │ Jan 1     │ ⋮      │
│ Project Beta  │ Draft    │ Jan 15    │ ⋮      │
│ Project Gamma │ Complete │ Feb 1     │ ⋮      │
└──────────────────────────────────────────────┘
```

Features:
- Sortable columns (click header)
- Row hover highlight
- Fixed header on scroll
- Actions in last column

### Pattern 2: Card Grid
**When:** Items are visual or need rich preview
**Apply:** Cards in responsive grid

```
┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│   [Image]   │ │   [Image]   │ │   [Image]   │
│             │ │             │ │             │
│ Title       │ │ Title       │ │ Title       │
│ Description │ │ Description │ │ Description │
│ [Action]    │ │ [Action]    │ │ [Action]    │
└─────────────┘ └─────────────┘ └─────────────┘
```

Rules:
- Consistent card height (or masonry)
- Clear click target
- Key info visible without hover

### Pattern 3: List View
**When:** Scanning/selecting from many items
**Apply:** Single-column list with key info

```
┌─────────────────────────────────────────────┐
│ ○ Project Alpha                    Active   │
│   Created Jan 1 • 12 tasks                  │
├─────────────────────────────────────────────┤
│ ○ Project Beta                     Draft    │
│   Created Jan 15 • 5 tasks                  │
├─────────────────────────────────────────────┤
```

Features:
- Avatar/icon for quick identification
- Secondary info on second line
- Status badge aligned right

### Pattern 4: Detail View
**When:** Viewing single item in depth
**Apply:** Header + sections

```
┌─────────────────────────────────────────────┐
│ Project Alpha                    [Edit]     │
│ Created Jan 1 by @user                      │
├─────────────────────────────────────────────┤
│ Overview  │ Tasks  │ Settings  │ Activity   │
├─────────────────────────────────────────────┤
│                                             │
│ Description                                 │
│ Lorem ipsum dolor sit amet...               │
│                                             │
│ Metadata                                    │
│ ┌─────────────┬─────────────┐              │
│ │ Status      │ Active      │              │
│ │ Due Date    │ Mar 1, 2024 │              │
│ └─────────────┴─────────────┘              │
└─────────────────────────────────────────────┘
```

### Pattern 5: Master-Detail
**When:** Browsing list and viewing details simultaneously
**Apply:** Split view

```
┌────────────────┬─────────────────────────────┐
│ Items          │ Selected Item Detail        │
│                │                             │
│ > Item A       │ Item A                      │
│   Item B       │ ─────────────               │
│   Item C       │ Full details here...        │
│   Item D       │                             │
│                │                             │
└────────────────┴─────────────────────────────┘
```

Rules:
- List width 1/4 to 1/3 of screen
- Clear selection indicator
- Handle empty state (no selection)

### Pattern 6: Tree View
**When:** Hierarchical data
**Apply:** Expandable nested list

```
▼ Root
  ├─ ▼ Folder A
  │  ├─ Item 1
  │  └─ Item 2
  └─ ▶ Folder B (collapsed)
     └─ (hidden items)
```

Rules:
- Clear expand/collapse affordance
- Indentation shows hierarchy
- Keyboard navigation (arrows)

### Pattern 7: Empty States
**When:** No data to display
**Apply:** Helpful empty state

```
┌─────────────────────────────────────────────┐
│                                             │
│              📁                             │
│                                             │
│         No projects yet                     │
│    Create your first project to get        │
│              started.                       │
│                                             │
│         [+ Create Project]                  │
│                                             │
└─────────────────────────────────────────────┘
```

Rules:
- Explain why empty
- Provide action to fix
- Use illustration/icon

## Anti-Patterns

- **Information Overload:** Too many columns/fields visible
- **Truncation Without Tooltip:** User can't see full content
- **No Loading States:** Data appears to be missing
- **Pagination Over Infinite Scroll:** For browsing (scroll better); for searching (pagination better)

## Checklist

- [ ] Appropriate display pattern for data type
- [ ] Key information visible at a glance
- [ ] Sorting/filtering available for large datasets
- [ ] Empty states handled gracefully
- [ ] Loading states shown during fetch
- [ ] Responsive behavior on smaller screens

## Claude Code Integration
**Command:** /ux-review
**Input:** Data display component
**Output:** Pattern recommendations and issues

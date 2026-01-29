---
name: ui-patterns
description: Comprehensive UI patterns for dashboards, forms, navigation, data display, and feedback.
---

# UI Patterns

Consolidated reference for common UI patterns. Individual pattern files in `ui-patterns/` folder have detailed examples.

## Quick Reference

### Dashboard Patterns

| Pattern | When to Use |
|---------|-------------|
| KPI Cards | Key metrics at top, large number + context |
| Sparklines | Compact trend visualization |
| Status Indicators | Traffic light (green/yellow/red) for health |
| Time Range Selector | Global control for time-series data |
| Drill-down | Click through from overview to detail |

**Layout:**
```
┌─────────────────────────────────────────────────┐
│ KPI Cards (most important, top-left)             │
├─────────────────────────────────────────────────┤
│ Primary Charts (trends)                          │
├─────────────────────────────────────────────────┤
│ Detail Tables / Lists (drill-down)               │
└─────────────────────────────────────────────────┘
```

**Rules:**
- Most critical info top-left (Z-pattern reading)
- Limit to 5-9 key metrics
- Group related metrics
- Progressive detail (overview → detail)

---

### Form Patterns

| Pattern | When to Use |
|---------|-------------|
| Single-Column | Default layout, one field per row |
| Inline Validation | Validate on blur, not keystroke |
| Progressive Disclosure | Hide advanced options until needed |
| Smart Defaults | Pre-fill based on context/history |
| Input Masks | Format as user types (phone, card) |
| Autosave | Long forms, save on blur |
| Error Summary | Multiple errors at top + inline |

**Layout:**
```
Label
┌────────────────────────────────────┐
│ Input                              │
└────────────────────────────────────┘

Label
┌────────────────────────────────────┐
│ Input                              │
└────────────────────────────────────┘

        [Primary Action]
```

**Rules:**
- Labels above fields, not inline
- Error messages below field, specific and actionable
- Primary action clearly distinguished
- Optional fields marked (if minority)

---

### Navigation Patterns

| Pattern | When to Use |
|---------|-------------|
| Global Navigation | 3-7 major sections, persistent bar |
| Breadcrumbs | Deep hierarchical content |
| Tabs | 2-7 parallel sections within a page |
| Sidebar Navigation | Many sections or nested hierarchy |
| Command Palette | Power user quick access (Cmd+K) |
| Wizard/Stepper | Multi-step linear process |

**Questions navigation must answer:**
1. Where am I?
2. Where can I go?
3. How do I get back?

**Rules:**
- Highlight current section
- Keep labels short (1-2 words)
- Never nest tabs within tabs
- Active tab visually connected to content

---

### Data Display Patterns

| Pattern | When to Use |
|---------|-------------|
| Data Table | Compare items with same attributes |
| Card Grid | Visual items, rich preview |
| List View | Scanning/selecting from many items |
| Detail View | Single item in depth |
| Master-Detail | Browse list + view details simultaneously |
| Tree View | Hierarchical data |
| Empty State | No data to display |

**Empty State Template:**
```
┌─────────────────────────────────────────────┐
│              📁                             │
│                                             │
│         No projects yet                     │
│    Create your first project to get        │
│              started.                       │
│                                             │
│         [+ Create Project]                  │
└─────────────────────────────────────────────┘
```

**Rules:**
- Empty states: explain why, provide action
- Tables: sortable columns, fixed header on scroll
- Loading states for all async data

---

### Feedback Patterns

| Pattern | When to Use | Duration |
|---------|-------------|----------|
| Loading State | Any async operation | Until complete |
| Success Toast | Action succeeded | Auto-dismiss 3-5s |
| Error Message | Action failed | Until acknowledged |
| Confirmation Dialog | Destructive/irreversible action | Until user decides |
| Progress Indicator | Multi-step or long process | Until complete |
| Skeleton Screen | Loading content-heavy pages | Until content loads |

**Loading Timing:**
- < 100ms: No indicator needed
- 100ms-1s: Spinner
- > 1s: Progress bar with estimate

**Error Message Template:**
```
┌────────────────────────────────────────────┐
│ ✗ Unable to save changes                   │
│   The file is too large (max 5MB).        │
│   [Try a smaller file] [Cancel]            │
└────────────────────────────────────────────┘
```

**Rules:**
- Every action has immediate visual feedback
- Success messages visible but not disruptive
- Error messages explain problem AND solution
- Destructive actions require confirmation

---

## Anti-Patterns

### Dashboard
- Too many metrics (>9 causes overload)
- No context (numbers without trend/comparison)
- Stale data (no freshness indicator)

### Forms
- Inline labels (disappear when typing)
- Reset buttons (cause accidental data loss)
- Validation while typing (frustrating)

### Navigation
- Mystery meat (icons without labels)
- Deep nesting (>3-4 levels)
- Inconsistent location

### Data Display
- Information overload
- Truncation without tooltip
- No loading states

### Feedback
- Silent failures
- Modal overload
- Cryptic errors

---

## Detailed Patterns

For comprehensive examples and code patterns, see individual files in `ui-patterns/`:
- `dashboards.md` - Dashboard layout and KPI patterns
- `forms.md` - Form layout and validation patterns
- `navigation.md` - Navigation structures
- `data-display.md` - Tables, lists, cards, trees
- `feedback.md` - Loading, success, error, progress

---
name: data-viz
description: Apply Tufte and Few's data visualization principles to maximize data-ink ratio and minimize chartjunk.
---

# Data Visualization Principles

## Source
- Books: The Visual Display of Quantitative Information (Tufte), Show Me the Numbers (Few)
- Authors: Edward Tufte, Stephen Few
- Key Concepts: Data-ink ratio, information density, cognitive load

## Core Principle
Maximize the data-ink ratio: every drop of ink should convey information. Remove chartjunk and let the data speak.

## Decision Framework

```
Choosing Chart Type:
├── Comparison
│   ├── Among items → Bar chart
│   ├── Over time → Line chart
│   └── Part-to-whole → Pie (rarely), stacked bar
├── Relationship
│   ├── Two variables → Scatter plot
│   └── Correlation → Scatter with trend
├── Distribution
│   ├── Single variable → Histogram
│   └── Comparison → Box plot
├── Composition
│   ├── Static → Stacked bar, treemap
│   └── Changing → Stacked area
└── Geographic
    └── Location-based → Map
```

## Patterns

### Pattern 1: Data-Ink Ratio
**When:** Any chart or visualization
**Apply:** Remove non-data ink ruthlessly

```
Before:                    After:
┌─────────────────┐
│ ████████████    │        ████████████  Label A
│ ████████        │        ████████      Label B
│ ████            │        ████          Label C
│ Grid Grid Grid  │
│ Border Border   │        (no grid, no border, no box)
└─────────────────┘
```

Elements to remove:
- Unnecessary gridlines (or make very light)
- Chart borders
- Redundant labels
- 3D effects
- Excessive decimal places

### Pattern 2: Small Multiples
**When:** Comparing same measure across categories
**Apply:** Repeat small charts with same scale

```
Region A    Region B    Region C    Region D
┌──────┐    ┌──────┐    ┌──────┐    ┌──────┐
│ ╱╲   │    │  ╱╲  │    │╱    │    │    ╲│
│╱  ╲  │    │ ╱  ╲ │    │     │    │     │
└──────┘    └──────┘    └──────┘    └──────┘

Same scale, same size, easy to compare patterns
```

### Pattern 3: Sparklines
**When:** Inline data context
**Apply:** Word-sized graphics embedded in text

```
Sales trend █▃▅▇▆▄▂▃▅▇ shows recovery in Q4.
CPU usage ▁▁▂▇▇▅▂▁▁▁ spiked during deployment.
```

### Pattern 4: Dashboard Layout (Few's Principles)
**When:** Designing dashboards
**Apply:** Information density with visual hierarchy

```
┌────────────────────────────────────┐
│ MOST IMPORTANT KPIs (large)       │
│ ┌──────────┐ ┌──────────┐         │
│ │  $1.2M   │ │   847    │         │
│ │ Revenue  │ │  Users   │         │
│ └──────────┘ └──────────┘         │
├────────────────────────────────────┤
│ TRENDS (medium)                   │
│ ═══════════════════════           │
├────────────────────────────────────┤
│ DETAILS (smaller, if needed)      │
└────────────────────────────────────┘

Principles:
- Most important at top-left (F-pattern reading)
- Limit to 5-9 elements
- Group related metrics
- Use whitespace for separation, not lines
```

### Pattern 5: Color Usage
**When:** Any visualization
**Apply:** Color with purpose

```
Use color for:
- Categorical distinction (limited palette, 5-7 max)
- Quantitative encoding (sequential or diverging)
- Highlighting exceptions (red for alerts)

Avoid:
- Rainbow palettes (no natural order)
- Red/green only (colorblindness)
- Color for decoration
```

Sequential: Low ░░▒▒▓▓██ High (one hue, varying intensity)
Diverging:  Neg ████░░░░████ Pos (two hues, meeting at neutral)

### Pattern 6: Labeling
**When:** Any chart
**Apply:** Direct labeling over legends

```
Bad (requires legend lookup):
■ Series A  ● Series B  ▲ Series C
[Chart with symbols]

Good (labels on data):
              Series A ─────
    Series B ───────
Series C ──────

Labels directly on or next to data points
```

## Anti-Patterns

### Chartjunk
- 3D effects that distort data
- Decorative illustrations
- Excessive gridlines
- Unnecessary colors/gradients
- Rotated labels that are hard to read

### Misleading Visualizations
- Truncated y-axis (exaggerates differences)
- Inconsistent scales between charts
- Cherry-picked time ranges
- Dual y-axes (confuses comparison)
- Area/volume encoding (humans bad at comparing)

### Pie Chart Overuse
Pie charts are rarely the best choice:
- Hard to compare similar-sized slices
- Can't compare across charts
- Use bar chart instead for most cases

## Checklist

- [ ] Is this the right chart type for the data?
- [ ] Can any non-data ink be removed?
- [ ] Are axes labeled and scaled appropriately?
- [ ] Are colors meaningful and accessible?
- [ ] Can labels replace the legend?
- [ ] Is the most important insight immediately visible?
- [ ] Does the chart tell one clear story?

## Claude Code Integration
**Command:** /data-viz
**Input:** Data description and goal
**Output:** Recommended visualization with rationale

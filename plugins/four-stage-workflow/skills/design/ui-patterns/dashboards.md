---
name: dashboards
description: Dashboard patterns providing at-a-glance awareness of current state and items needing attention.
---

# Dashboard Patterns

## Source
- Books: Information Dashboard Design (Few), Show Me the Numbers (Few)
- Author: Stephen Few
- Key Concepts: Situational awareness, visual hierarchy, information density

## Core Principle
A dashboard should provide at-a-glance awareness of the current state and alert to items needing attention.

## Patterns

### Pattern 1: Dashboard Layout
**When:** Designing any dashboard
**Apply:** Z-pattern or F-pattern reading order

```
┌─────────────────────────────────────────────────┐
│ KPI Cards (most important metrics)              │
│ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐│
│ │ $1.2M   │ │  847    │ │  92%    │ │  12     ││
│ │ Revenue │ │ Users   │ │ Uptime  │ │ Alerts  ││
│ │ ↑ 12%   │ │ ↑ 5%    │ │ ↓ 0.1%  │ │ ⚠ 3 new ││
│ └─────────┘ └─────────┘ └─────────┘ └─────────┘│
├─────────────────────────────────────────────────┤
│ Primary Charts (trend analysis)                 │
│ ┌─────────────────────┐ ┌─────────────────────┐│
│ │                     │ │                     ││
│ │   Revenue Trend     │ │   User Growth       ││
│ │   ═══════════       │ │   ═══════════       ││
│ └─────────────────────┘ └─────────────────────┘│
├─────────────────────────────────────────────────┤
│ Detail Tables / Lists (drill-down)              │
│ ┌───────────────────────────────────────────┐  │
│ │ Recent Activity  │ Top Items │ Alerts     │  │
│ └───────────────────────────────────────────┘  │
└─────────────────────────────────────────────────┘
```

Rules:
- Most critical info top-left
- Limit to 5-9 key metrics
- Group related metrics
- Progressive detail (overview → detail)

### Pattern 2: KPI Cards
**When:** Showing single key metrics
**Apply:** Large number + context

```
┌─────────────────┐
│     $1.2M       │  ← Primary metric (large)
│     Revenue     │  ← Label
│                 │
│  ▲ 12% vs last  │  ← Comparison/trend
│     month       │
└─────────────────┘
```

Elements:
- Primary number (prominent)
- Clear label
- Trend indicator (↑↓ or sparkline)
- Comparison period

### Pattern 3: Trend Sparklines
**When:** Showing change over time in compact space
**Apply:** Mini line chart without axes

```
Revenue  $1.2M  ▁▂▃▄▅▆▇█▇▆
Users      847  ▁▁▂▃▃▄▅▆▇█
Errors      12  ▇▆▅▄▃▂▁▁▁▁
```

Rules:
- Same scale for comparison
- Recent data on right
- No need for labeled axes

### Pattern 4: Status Indicators
**When:** Showing system/item health
**Apply:** Traffic light or icon status

```
Status indicators:
● Green  = Good/Healthy
● Yellow = Warning/Attention
● Red    = Critical/Error
● Gray   = Unknown/Inactive

Example:
┌─────────────────────────────────┐
│ Service Status                  │
│ ● API Gateway     Healthy       │
│ ● Database        Healthy       │
│ ● Cache           Warning       │
│ ● Worker Queue    Error         │
└─────────────────────────────────┘
```

### Pattern 5: Alert/Exception Highlighting
**When:** Items need attention
**Apply:** Visual prominence for anomalies

```
Normal state:
│ Server 1  │ 45% CPU │ 2.1GB │

Alert state:
│ Server 2  │ ⚠ 95% CPU │ 7.8GB │  ← Highlighted row
  └─ Background color change
  └─ Icon indicator
  └─ Possibly sorted to top
```

### Pattern 6: Time Range Selector
**When:** Data is time-series
**Apply:** Consistent time controls

```
┌─────────────────────────────────────────────┐
│ Dashboard Title         [1H][1D][1W][1M][1Y]│
│                         └── Quick presets   │
│              or                             │
│         [Jan 1, 2024] to [Jan 31, 2024]     │
│                         └── Custom range    │
└─────────────────────────────────────────────┘
```

Rules:
- Apply to all widgets globally
- Show current selection clearly
- Provide presets for common ranges

### Pattern 7: Drill-Down
**When:** Users need to investigate anomalies
**Apply:** Click through to detail

```
Overview → Click metric → Detailed view

KPI: 12 Alerts ← Click
         ↓
┌─────────────────────────────────────────────┐
│ Alert Details                               │
│ ┌─────────────────────────────────────────┐│
│ │ [Critical] Server CPU at 95%  2min ago  ││
│ │ [Warning] Memory usage high   5min ago  ││
│ │ ...                                     ││
│ └─────────────────────────────────────────┘│
└─────────────────────────────────────────────┘
```

## Anti-Patterns

- **Too Many Metrics:** More than 9 key metrics causes overload
- **Decoration Over Data:** Gauge charts, 3D, unnecessary graphics
- **No Context:** Numbers without comparison or trend
- **Stale Data:** No indication of data freshness
- **Scroll to See KPIs:** Critical metrics should be above fold
- **Pie Charts for Comparison:** Use bar charts instead

## Checklist

- [ ] Most important metrics visible without scrolling
- [ ] Each metric has context (trend, comparison)
- [ ] Color used consistently (red=bad, green=good)
- [ ] Time range clear and controllable
- [ ] Drill-down available for investigation
- [ ] Loading states and data freshness shown
- [ ] Responsive on different screen sizes
- [ ] Limit to 5-9 key metrics maximum

## Claude Code Integration
**Command:** /ux-review
**Input:** Dashboard design or code
**Output:** Dashboard pattern analysis and recommendations

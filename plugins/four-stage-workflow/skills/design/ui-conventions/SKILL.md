---
name: ui-conventions
description: Follow the project's "Luxury Flat Design" system when creating UI components and pages in Leptos.
---

# UI Conventions

## Source

- Project analysis: `docs/UI_CONVENTIONS_ANALYSIS.md`
- Design tokens: `crates/frontend/input.css`
- Reference components: `crates/frontend/src/components/generic/`
- Reference page: `crates/frontend/src/pages/usage_analytics.rs`

## Core Principle

Follow the "Luxury Flat Design" system: dark backgrounds, muted accent colors, serif headings, generous spacing. Use CSS variables for all styling, the signal triplet pattern for async data, and pass signals (not values) to reactive components.

## Decision Framework

```
What are you building?
├── New page with async data
│   └── Use signal triplet (loading/error/data) + Effect::new
├── New page without async data
│   └── Use direct signals, access global state via use_app_state()
├── New reusable component
│   └── Add to components/generic/, follow StatCard pattern
├── New domain component
│   └── Add to components/{domain}/, compose from generic components
├── New chart/visualization
│   └── Add to components/charts/, use CSS variables for colors
└── Styling changes
    └── Edit input.css, use CSS variables, never inline styles
```

## Design Tokens

### Colors (use CSS variables, never hex values)

```css
/* Backgrounds */
var(--bg-deep)       /* #080a0c - root background */
var(--bg-primary)    /* #0e1216 - main surface */
var(--bg-secondary)  /* #0e1216 - cards, panels */
var(--bg-tertiary)   /* #151a1f - elevated */
var(--bg-hover)      /* #1a2026 - hover states */

/* Text */
var(--text-primary)   /* Headlines, primary text */
var(--text-secondary) /* Secondary text */
var(--text-muted)     /* Labels, hints */

/* Accents (muted luxury palette) */
var(--accent-blue)      /* #7da0d4 - primary */
var(--accent-blue-dim)  /* rgba(125, 160, 212, 0.6) - dimmed primary */
var(--accent-cyan)      /* #7dc4d4 - secondary */
var(--accent-sage)      /* #9caa9c - success */
var(--accent-gold)      /* #d4b87d - highlights */
var(--accent-purple)    /* #a37dd4 - alternative */
var(--accent-orange)    /* #d4a07d - warm accent */
var(--accent-error)     /* #d47d7d - errors */
var(--accent-teal)      /* #7dd4c4 - teal variant */

/* Status */
var(--success)  /* Green */
var(--warning)  /* Yellow */
var(--danger)   /* Red */

/* Borders */
var(--border)         /* rgba(200, 215, 235, 0.12) */
var(--border-subtle)  /* rgba(200, 215, 235, 0.08) */
```

### Typography

```css
var(--font-heading)  /* Cormorant Garamond - headings only */
var(--font-body)     /* Libre Franklin - body text */
var(--font-mono)     /* Monospace - code */

/* Heading styles */
h1: 3.5rem, weight 300, letter-spacing 0.15em
h2: 1.5rem, weight 400, letter-spacing 0.1em
h3: 1.25rem, weight 400

/* Special cases */
KPI values: 3rem, Cormorant Garamond, weight 300
KPI labels: 0.65rem, uppercase, letter-spacing 0.35em
Navigation: 0.75rem, uppercase, letter-spacing 0.25em
```

### Spacing

```css
var(--space-xs)   /* 0.25rem - 4px */
var(--space-sm)   /* 0.5rem - 8px */
var(--space-md)   /* 1rem - 16px */
var(--space-lg)   /* 1.5rem - 24px */
var(--space-xl)   /* 2rem - 32px */
var(--space-2xl)  /* 2.5rem - 40px */
var(--space-3xl)  /* 4rem - 64px (luxury spacing) */
```

### Border Radius

```css
var(--radius-sm)  /* 0.25rem */
var(--radius-md)  /* 0.5rem */
var(--radius-lg)  /* 0.75rem */
var(--radius-xl)  /* 1rem */
```

## Patterns

### Pattern 1: Page with Async Data

**When:** Creating any page that fetches data from the API
**Apply:** Signal triplet + Effect::new + proper loading/error states

```rust
use leptos::prelude::*;
use crate::api::{ApiClient, ApiError, API_BASE_URL};
use crate::components::generic::{LoadingSpinner, ErrorDisplay, EmptyState};

#[component]
pub fn MyPage() -> impl IntoView {
    // 1. Signal triplet - ALWAYS use this pattern
    let loading = RwSignal::new(true);
    let error: RwSignal<Option<ApiError>> = RwSignal::new(None);
    let data = RwSignal::new(MyData::default());

    // 2. Effect for data fetching
    Effect::new(move || {
        // Read dependencies here - triggers re-run when they change
        let period = time_period.get();

        spawn_local(async move {
            loading.set(true);
            error.set(None);  // Always clear previous errors

            let api = ApiClient::new(API_BASE_URL);
            match api.fetch_data(period).await {
                Ok(d) => data.set(d),
                Err(e) => error.set(Some(e)),
            }
            loading.set(false);  // Always set false, even on error
        });
    });

    // 3. View with proper state handling
    view! {
        <div class="page-container">
            <div class="page-header-luxury">
                <h1 class="page-title-luxury">"My Page"</h1>
            </div>

            // Error display (reactive)
            {move || view! { <ErrorDisplay error=error.get()/> }}

            // Loading spinner
            {move || {
                if loading.get() {
                    view! { <LoadingSpinner/> }.into_any()
                } else {
                    view! { <div></div> }.into_any()
                }
            }}

            // Content (hidden while loading)
            <div class:hidden=move || loading.get()>
                {move || {
                    let d = data.get();
                    if d.items.is_empty() {
                        view! { <EmptyState title="No data found".to_string()/> }.into_any()
                    } else {
                        view! {
                            <div class="section-card">
                                // Render data
                            </div>
                        }.into_any()
                    }
                }}
            </div>
        </div>
    }
}
```

### Pattern 2: Reusable Component with Enum Variants

**When:** Creating a component to be used across multiple pages
**Apply:** Full documentation, enum with helper methods, optional props, tests

```rust
//! # MyComponent
//!
//! Brief description of what this component does.
//!
//! ## Props
//! - `label`: The display label
//! - `value`: The primary value to display
//! - `variant`: Display variant (default, compact, large)
//! - `on_click`: Optional callback when clicked
//!
//! ## CSS Classes
//! - `.my-component`: Main container
//! - `.my-component-label`: Label element
//! - `.my-component-value`: Value element
//! - `.my-component--compact`: Compact variant
//! - `.my-component--large`: Large variant

use leptos::prelude::*;

/// Display variants for MyComponent
#[derive(Clone, Copy, Debug, Default, PartialEq, Eq)]
pub enum ComponentVariant {
    #[default]
    Default,
    Compact,
    Large,
}

impl ComponentVariant {
    pub fn css_class(&self) -> &'static str {
        match self {
            Self::Default => "",
            Self::Compact => "my-component--compact",
            Self::Large => "my-component--large",
        }
    }
}

/// A reusable component for displaying X.
///
/// # Example
/// ```rust,ignore
/// view! { <MyComponent label="Total".to_string() value="42".to_string()/> }
/// ```
#[component]
pub fn MyComponent(
    /// The display label
    label: String,
    /// The primary value
    value: String,
    /// Display variant
    #[prop(optional)]
    variant: Option<ComponentVariant>,
    /// Optional click handler
    #[prop(optional)]
    on_click: Option<Callback<()>>,
) -> impl IntoView {
    let variant_class = variant.unwrap_or_default().css_class();

    let handle_click = move |_| {
        if let Some(callback) = on_click {
            callback.run(());
        }
    };

    view! {
        <div class={format!("my-component {}", variant_class)} on:click=handle_click>
            <span class="my-component-label">{label}</span>
            <span class="my-component-value">{value}</span>
        </div>
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_variant_css_class() {
        assert_eq!(ComponentVariant::Default.css_class(), "");
        assert_eq!(ComponentVariant::Compact.css_class(), "my-component--compact");
        assert_eq!(ComponentVariant::Large.css_class(), "my-component--large");
    }
}
```

### Pattern 3: Builder Pattern for Complex Data

**When:** Component needs structured data with multiple optional fields
**Apply:** Builder struct with chainable methods

```rust
pub struct CardData {
    pub id: String,
    pub label: String,
    pub sublabel: Option<String>,
    pub value: i32,
    pub is_highlighted: bool,
    pub activity_percent: f64,
}

impl CardData {
    pub fn new(id: impl Into<String>, label: impl Into<String>, value: i32) -> Self {
        Self {
            id: id.into(),
            label: label.into(),
            sublabel: None,
            value,
            is_highlighted: false,
            activity_percent: 0.0,
        }
    }

    pub fn with_sublabel(mut self, sublabel: impl Into<String>) -> Self {
        self.sublabel = Some(sublabel.into());
        self
    }

    pub fn highlighted(mut self) -> Self {
        self.is_highlighted = true;
        self
    }

    pub fn with_activity(mut self, percent: f64) -> Self {
        self.activity_percent = percent.clamp(0.0, 100.0);
        self
    }
}

// Usage
let card = CardData::new("item-1", "Monday", 42)
    .with_sublabel("Jan 15")
    .highlighted()
    .with_activity(75.0);
```

### Pattern 4: Signal Passing to Child Components

**When:** Child component needs to react to parent state changes
**Apply:** Pass the signal itself, never `.get()`

```rust
// Parent component
#[component]
pub fn ParentPage() -> impl IntoView {
    let selected = RwSignal::new(FilterOption::All);

    let on_change = Callback::new(move |option: FilterOption| {
        selected.set(option);
    });

    view! {
        // CORRECT: Pass signal directly
        <FilterSelector selected=selected on_change=on_change/>

        // WRONG: This breaks reactivity!
        // <FilterSelector selected=selected.get() on_change=on_change/>
    }
}

// Child component
#[component]
pub fn FilterSelector(
    /// Currently selected option - accepts SIGNAL, not value
    selected: RwSignal<FilterOption>,
    /// Callback when selection changes
    on_change: Callback<FilterOption>,
) -> impl IntoView {
    view! {
        <div class="filter-selector">
            {move || {
                // Read signal inside reactive closure
                let current = selected.get();
                FilterOption::all().into_iter().map(|option| {
                    let is_selected = current == option;
                    view! {
                        <button
                            class:active=is_selected
                            on:click=move |_| on_change.run(option)
                        >
                            {option.label()}
                        </button>
                    }
                }).collect_view()
            }}
        </div>
    }
}
```

### Pattern 5: CSS Class Naming

**When:** Adding new CSS classes to `input.css`
**Apply:** Domain-prefixed BEM-like naming

```css
/* Pattern: {domain}-{component}-{element} */

/* Usage Analytics domain */
.ua-distribution-list-container { }
.ua-distribution-list-item { }
.ua-distribution-list-item-label { }

/* Session domain */
.session-card { }
.session-card-header { }
.session-card-content { }

/* Generic (no domain prefix) */
.stat-card { }
.stat-card-label { }
.stat-card-value { }

/* Modifiers use double-dash */
.btn-luxury--primary { }
.btn-luxury--secondary { }
.stat-card--clickable { }
.my-component--compact { }

/* State classes */
.active { }
.selected { }
.today { }
.loading { }
.expanded { }
.hidden { }
```

### Pattern 6: Page Layout Structure

**When:** Creating any new page
**Apply:** Standard container → header → content structure

```rust
view! {
    <div class="page-container">
        // Luxury header with serif title
        <div class="page-header-luxury">
            <h1 class="page-title-luxury">"Page Title"</h1>
            // Optional controls on the right
            <div class="page-header-controls">
                <DateRangePicker selected=period on_change=on_period_change/>
            </div>
        </div>

        // KPI cards at top
        <div class="section">
            <div class="grid-kpi">
                <StatCard label="Total".to_string() value=total/>
                <StatCard label="Active".to_string() value=active/>
            </div>
        </div>

        // Section with border/background
        <div class="section-card">
            <h2>"Section Title"</h2>
            // Content here
        </div>

        // Multi-column grid
        <div class="grid-primary">
            <div class="section-card">/* Column 1 */</div>
            <div class="section-card">/* Column 2 */</div>
        </div>
    </div>
}
```

### Pattern 7: Empty States

**When:** Data is loaded but empty
**Apply:** Use EmptyState component or domain-specific variant

```rust
{move || {
    let items = data.get();
    if items.is_empty() {
        view! {
            <EmptyState
                title="No sessions found".to_string()
                description=Some("Try adjusting your filters or date range.".to_string())
            />
        }.into_any()
    } else {
        view! {
            <SessionList sessions=items/>
        }.into_any()
    }
}}
```

### Pattern 8: Accessibility

**When:** Creating interactive elements
**Apply:** ARIA labels + keyboard handlers

```rust
view! {
    <div
        class="interactive-card"
        role="button"
        tabindex="0"
        aria-label={format!("{}: {} sessions", label, count)}
        on:click=handle_click
        on:keydown=handle_keydown
    >
        // Content
    </div>
}

// Keyboard handler
let on_keydown = move |e: web_sys::KeyboardEvent| {
    if e.key() == "Enter" || e.key() == " " {
        e.prevent_default();
        on_click.run(id.clone());
    }
};
```

### Pattern 9: Conditional CSS Classes

**When:** Component has multiple states
**Apply:** Build class string with conditions

```rust
// Pattern A: Simple conditional
let card_class = if is_selected {
    "session-card selected"
} else {
    "session-card"
};

// Pattern B: Move closure for reactive classes
<div class=move || {
    if is_active() {
        "header-nav-item active"
    } else {
        "header-nav-item"
    }
}>

// Pattern C: Vec builder for multiple conditions
let mut classes = vec!["ua-temporal-card".to_string()];
if data.is_current {
    classes.push("today".to_string());
}
if is_selected {
    classes.push("selected".to_string());
}
let card_class = classes.join(" ");

// Pattern D: Leptos class: directive
<button
    class="date-range-option"
    class:active=is_active
>
```

## Anti-Patterns

### Calling .get() at Prop Site

```rust
// WRONG: Evaluated once, no reactivity
<ChildComponent value=signal.get()/>

// CORRECT: Pass signal, read inside child
<ChildComponent value=signal/>
```

### Hardcoded Colors

```rust
// WRONG: Hardcoded hex values
view! { <div style="color: #7da0d4;"> }

// CORRECT: Use CSS classes that reference variables
view! { <div class="text-accent"> }
```

### Inline Styles

```rust
// WRONG: Inline styles
view! { <div style="padding: 16px; margin: 8px;"> }

// CORRECT: CSS classes
view! { <div class="section-card"> }
```

### Missing Loading State Reset

```rust
// WRONG: Forgetting to reset loading on error
Effect::new(move || {
    spawn_local(async move {
        match api.fetch().await {
            Ok(d) => {
                data.set(d);
                loading.set(false);  // Only on success
            }
            Err(e) => error.set(Some(e)),  // Loading stuck!
        }
    });
});

// CORRECT: Always set loading false
Effect::new(move || {
    spawn_local(async move {
        loading.set(true);
        error.set(None);
        match api.fetch().await {
            Ok(d) => data.set(d),
            Err(e) => error.set(Some(e)),
        }
        loading.set(false);  // Always runs
    });
});
```

### Hardcoded API URLs

```rust
// WRONG
let api = ApiClient::new("http://localhost:7890/api/v1");

// CORRECT
const API_BASE_URL: &str = "/api/v1";
let api = ApiClient::new(API_BASE_URL);
```

### Creating Components Without Documentation

```rust
// WRONG: No module doc, no prop docs
#[component]
pub fn MyCard(label: String, value: i32) -> impl IntoView {
    view! { <div>{label}: {value}</div> }
}

// CORRECT: Full documentation
//! # MyCard
//!
//! Displays a labeled value in a card format.

/// A card component for displaying labeled values.
#[component]
pub fn MyCard(
    /// The label to display
    label: String,
    /// The numeric value
    value: i32,
) -> impl IntoView {
    view! { <div class="my-card">{label}: {value}</div> }
}
```

### Enums Without Helper Methods

```rust
// WRONG: Enum without helpers
pub enum Size { Small, Medium, Large }

// CORRECT: Enum with helpers
#[derive(Clone, Copy, Default)]
pub enum Size {
    Small,
    #[default]
    Medium,
    Large,
}

impl Size {
    pub fn css_class(&self) -> &'static str {
        match self {
            Self::Small => "size--sm",
            Self::Medium => "",
            Self::Large => "size--lg",
        }
    }
}
```

## Checklists

### New Page Checklist

- [ ] Signal triplet defined (`loading/error/data`)
- [ ] Effect::new for data fetching with spawn_local
- [ ] Dependencies explicitly read in Effect (not in spawn_local)
- [ ] `loading.set(true)` at start, `loading.set(false)` always at end
- [ ] `error.set(None)` before fetching
- [ ] ErrorDisplay component for errors
- [ ] LoadingSpinner while loading
- [ ] Content hidden while loading (`class:hidden=move || loading.get()`)
- [ ] EmptyState for no data
- [ ] Page structure: `page-container` → `page-header-luxury` → `section-card`
- [ ] Uses `API_BASE_URL` constant, never hardcoded URLs
- [ ] Added to `pages/mod.rs` with `pub mod` and `pub use`
- [ ] Route added to `app.rs`

### New Component Checklist

- [ ] Module-level doc comment (`//!`)
- [ ] Component doc comment (`///`)
- [ ] All props documented with `///`
- [ ] CSS classes documented in module doc
- [ ] Optional props use `#[prop(optional)]`
- [ ] Boolean props use `#[prop(default = false)]`
- [ ] Reactive props accept `RwSignal<T>`, not `T`
- [ ] Callbacks use `Callback<T>` type
- [ ] Callbacks named `on_<event>` (e.g., `on_click`, `on_change`)
- [ ] Enums have `.css_class()` method
- [ ] Enums derive `Clone, Copy, Default, PartialEq, Eq`
- [ ] Unit tests for helper functions and enums
- [ ] Added to parent `mod.rs` with `pub mod` and `pub use`

### CSS Checklist

- [ ] Uses CSS variables, never hardcoded values
- [ ] Follows naming: `{domain}-{component}-{element}`
- [ ] Modifiers use double-dash (`--primary`, `--active`)
- [ ] Added to appropriate section in `input.css`
- [ ] Hover states use `var(--bg-hover)` or `var(--bg-tertiary)`
- [ ] Transitions on interactive elements (`transition: all 0.2s ease`)
- [ ] Border colors use `var(--border)` or `var(--border-subtle)`

## Reference Files

| Purpose | File |
|---------|------|
| Enum + builder + tests | `components/generic/stat_card.rs` |
| Multi-state UI + accessibility | `components/analytics/temporal_grid.rs` |
| Complex page pattern | `pages/usage_analytics.rs` |
| Simple component + enum | `components/generic/loading.rs` |
| Session display + callbacks | `components/session/session_card.rs` |
| Active state navigation | `components/layout/header.rs` |
| Design tokens | `crates/frontend/input.css` |
| Full analysis | `docs/UI_CONVENTIONS_ANALYSIS.md` |

## Claude Code Integration

**When to apply:** Any frontend work - creating pages, components, or styling changes.

**Auto-apply triggers:**
- Creating files in `crates/frontend/src/pages/`
- Creating files in `crates/frontend/src/components/`
- Editing `crates/frontend/input.css`
- Any task mentioning "UI", "frontend", "component", "page", or "styling"

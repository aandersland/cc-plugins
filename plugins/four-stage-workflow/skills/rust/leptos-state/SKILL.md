---
name: leptos-state
description: Use the finest-grained signal that meets your needs in Leptos reactive web applications.
---

# Leptos State Management

## Source
- Leptos documentation (https://leptos.dev)
- Reactive programming principles
- Focus: Fine-grained reactivity for Rust web applications

## Core Principle

Use the finest-grained signal that meets your needs: prefer read signals over read-write, derive computed values rather than duplicating state, and scope state to the smallest necessary context.

## Decision Framework

```
What kind of state do you need?
├── Single value, component-local
│   └── create_signal (ReadSignal + WriteSignal)
├── Single value, read-only in children
│   └── create_signal, pass only ReadSignal down
├── Computed/derived value
│   └── create_memo (cached, reactive computation)
├── Complex struct with multiple fields
│   └── create_rw_signal or Store (0.7+)
├── Global/app-wide state
│   └── provide_context + use_context
├── Async data fetching
│   └── create_resource
└── Side effects on state change
    └── create_effect
```

## Patterns

### Pattern 1: Basic Signals

**When:** Local component state that changes over time
**Apply:** Use `create_signal` for the finest-grained control

```rust
use leptos::*;

#[component]
pub fn Counter() -> impl IntoView {
    // create_signal returns (ReadSignal, WriteSignal)
    let (count, set_count) = create_signal(0);

    view! {
        <button on:click=move |_| set_count.update(|n| *n += 1)>
            "Count: " {count}
        </button>
    }
}

// Destructure for clarity
#[component]
pub fn Toggle() -> impl IntoView {
    let (is_open, set_is_open) = create_signal(false);

    let toggle = move |_| set_is_open.update(|v| *v = !*v);

    view! {
        <button on:click=toggle>
            {move || if is_open.get() { "Close" } else { "Open" }}
        </button>
        <Show when=move || is_open.get()>
            <div class="dropdown">"Dropdown content"</div>
        </Show>
    }
}
```

### Pattern 2: Derived Signals with Memos

**When:** A value computed from other signals
**Apply:** Use `create_memo` for cached, reactive computations

```rust
#[component]
pub fn ShoppingCart(items: ReadSignal<Vec<CartItem>>) -> impl IntoView {
    // Memo: recomputes only when items change
    let total = create_memo(move |_| {
        items.get()
            .iter()
            .map(|item| item.price * item.quantity as f64)
            .sum::<f64>()
    });

    let item_count = create_memo(move |_| {
        items.get().iter().map(|i| i.quantity).sum::<u32>()
    });

    // Derived without memo for simple cases
    let is_empty = move || items.get().is_empty();

    view! {
        <div class="cart">
            <Show
                when=move || !is_empty()
                fallback=|| view! { <p>"Your cart is empty"</p> }
            >
                <p>"Items: " {item_count}</p>
                <p>"Total: $" {move || format!("{:.2}", total.get())}</p>
            </Show>
        </div>
    }
}
```

### Pattern 3: RwSignal for Complex State

**When:** You need read and write in the same place, or for complex types
**Apply:** Use `create_rw_signal` for convenience

```rust
use leptos::*;

#[derive(Clone, Debug)]
pub struct FormState {
    pub username: String,
    pub email: String,
    pub password: String,
    pub errors: Vec<String>,
}

#[component]
pub fn RegistrationForm() -> impl IntoView {
    let form = create_rw_signal(FormState {
        username: String::new(),
        email: String::new(),
        password: String::new(),
        errors: vec![],
    });

    let update_username = move |ev| {
        form.update(|f| f.username = event_target_value(&ev));
    };

    let update_email = move |ev| {
        form.update(|f| f.email = event_target_value(&ev));
    };

    let submit = move |ev: ev::SubmitEvent| {
        ev.prevent_default();
        form.update(|f| {
            f.errors.clear();
            if f.username.is_empty() {
                f.errors.push("Username required".into());
            }
            if !f.email.contains('@') {
                f.errors.push("Invalid email".into());
            }
        });

        if form.get().errors.is_empty() {
            // Submit logic
        }
    };

    view! {
        <form on:submit=submit>
            <input
                type="text"
                prop:value=move || form.get().username
                on:input=update_username
            />
            <input
                type="email"
                prop:value=move || form.get().email
                on:input=update_email
            />
            <For
                each=move || form.get().errors
                key=|error| error.clone()
                children=|error| view! { <p class="error">{error}</p> }
            />
            <button type="submit">"Register"</button>
        </form>
    }
}
```

### Pattern 4: Context for Global State

**When:** State needs to be accessed across many components
**Apply:** Use `provide_context` at a parent, `use_context` in children

```rust
use leptos::*;

#[derive(Clone)]
pub struct AuthContext {
    pub user: RwSignal<Option<User>>,
    pub login: Callback<Credentials, ()>,
    pub logout: Callback<(), ()>,
}

#[component]
pub fn App() -> impl IntoView {
    let user = create_rw_signal(None::<User>);

    let login = Callback::new(move |creds: Credentials| {
        spawn_local(async move {
            if let Ok(u) = authenticate(creds).await {
                user.set(Some(u));
            }
        });
    });

    let logout = Callback::new(move |_| {
        user.set(None);
    });

    provide_context(AuthContext { user, login, logout });

    view! {
        <Router>
            <Nav />
            <main>
                <Routes>
                    // Routes here
                </Routes>
            </main>
        </Router>
    }
}

#[component]
pub fn Nav() -> impl IntoView {
    let auth = use_context::<AuthContext>()
        .expect("AuthContext not provided");

    view! {
        <nav>
            <Show
                when=move || auth.user.get().is_some()
                fallback=move || view! {
                    <button on:click=move |_| { /* show login modal */ }>
                        "Login"
                    </button>
                }
            >
                {move || {
                    let user = auth.user.get().unwrap();
                    view! {
                        <span>"Welcome, " {user.name}</span>
                        <button on:click=move |_| auth.logout.call(())>
                            "Logout"
                        </button>
                    }
                }}
            </Show>
        </nav>
    }
}
```

### Pattern 5: Resources for Async Data

**When:** Fetching data from an API or async source
**Apply:** Use `create_resource` for reactive async data

```rust
use leptos::*;

#[component]
pub fn UserProfile(user_id: ReadSignal<String>) -> impl IntoView {
    // Resource refetches when user_id changes
    let user_data = create_resource(
        move || user_id.get(),
        |id| async move {
            fetch_user(&id).await
        }
    );

    view! {
        <Suspense fallback=|| view! { <p>"Loading..."</p> }>
            {move || {
                user_data.get().map(|result| match result {
                    Ok(user) => view! {
                        <div class="profile">
                            <h1>{&user.name}</h1>
                            <p>{&user.bio}</p>
                        </div>
                    }.into_view(),
                    Err(e) => view! {
                        <p class="error">"Failed to load: " {e.to_string()}</p>
                    }.into_view(),
                })
            }}
        </Suspense>
    }
}

// With multiple dependencies
#[component]
pub fn Dashboard() -> impl IntoView {
    let (filter, set_filter) = create_signal(Filter::default());
    let (page, set_page) = create_signal(1);

    let data = create_resource(
        move || (filter.get(), page.get()),
        |(filter, page)| async move {
            fetch_dashboard_data(filter, page).await
        }
    );

    // ...
}
```

### Pattern 6: Effects for Side Effects

**When:** You need to perform side effects when signals change
**Apply:** Use `create_effect` sparingly, prefer derived state

```rust
use leptos::*;

#[component]
pub fn DocumentTitle(title: ReadSignal<String>) -> impl IntoView {
    // Effect: update document title when signal changes
    create_effect(move |_| {
        document().set_title(&title.get());
    });

    // Component renders nothing
    ().into_view()
}

#[component]
pub fn LocalStorageSync(key: &'static str, value: RwSignal<String>) -> impl IntoView {
    // Load initial value
    create_effect(move |_| {
        if let Ok(Some(storage)) = window().local_storage() {
            if let Ok(Some(stored)) = storage.get_item(key) {
                value.set(stored);
            }
        }
    });

    // Save changes
    create_effect(move |prev: Option<String>| {
        let current = value.get();
        // Only save if value changed (not initial load)
        if prev.is_some() && prev.as_ref() != Some(&current) {
            if let Ok(Some(storage)) = window().local_storage() {
                let _ = storage.set_item(key, &current);
            }
        }
        current
    });

    ().into_view()
}
```

### Pattern 7: Store for Nested State (Leptos 0.7+)

**When:** Complex nested state with fine-grained updates
**Apply:** Use Store for struct-level reactivity

```rust
use leptos::*;
use leptos::prelude::*;

#[derive(Clone, Store)]
pub struct AppState {
    pub user: Option<User>,
    pub settings: Settings,
    pub notifications: Vec<Notification>,
}

#[derive(Clone, Store)]
pub struct Settings {
    pub theme: Theme,
    pub notifications_enabled: bool,
    pub language: String,
}

#[component]
pub fn App() -> impl IntoView {
    let state = Store::new(AppState {
        user: None,
        settings: Settings::default(),
        notifications: vec![],
    });

    provide_context(state);

    view! {
        // ...
    }
}

#[component]
pub fn ThemeToggle() -> impl IntoView {
    let state = use_context::<Store<AppState>>()
        .expect("AppState not provided");

    // Fine-grained access: only rerenders when theme changes
    let theme = state.settings().theme();

    view! {
        <button on:click=move |_| {
            state.settings().theme().update(|t| {
                *t = match t {
                    Theme::Light => Theme::Dark,
                    Theme::Dark => Theme::Light,
                };
            });
        }>
            "Toggle theme: " {move || format!("{:?}", theme.get())}
        </button>
    }
}
```

## Anti-Patterns

### Signal in Loop
```rust
// AVOID: Creates new signal every render
#[component]
fn Bad() -> impl IntoView {
    view! {
        {(0..10).map(|i| {
            let (count, set_count) = create_signal(i);  // BAD!
            view! { <button>{count}</button> }
        }).collect_view()}
    }
}

// BETTER: Use For with stable keys
#[component]
fn Good(items: ReadSignal<Vec<Item>>) -> impl IntoView {
    view! {
        <For
            each=move || items.get()
            key=|item| item.id
            children=|item| view! { <ItemView item /> }
        />
    }
}
```

### Effect for Derived State
```rust
// AVOID: Using effect to sync derived state
let (items, set_items) = create_signal(vec![]);
let (total, set_total) = create_signal(0);

create_effect(move |_| {
    set_total.set(items.get().len());  // BAD!
});

// BETTER: Use memo
let total = create_memo(move |_| items.get().len());
```

### Prop Drilling Context
```rust
// AVOID: Passing through many layers
fn Parent(data: RwSignal<Data>) -> impl IntoView {
    view! { <Child data /> }
}
fn Child(data: RwSignal<Data>) -> impl IntoView {
    view! { <GrandChild data /> }
}

// BETTER: Use context for widely-shared state
provide_context(data);
// In any descendant:
let data = use_context::<RwSignal<Data>>().expect("...");
```

## Checklist

- [ ] Using finest-grained signal type for the use case
- [ ] Derived values use memo, not effect + signal
- [ ] Global state uses context pattern
- [ ] Async data uses resources with Suspense
- [ ] No signals created in loops or closures
- [ ] Effects only for true side effects (DOM, storage, etc.)
- [ ] Props prefer ReadSignal over RwSignal where possible

## Claude Code Integration

**Command:** /state-model
**Input:** User flow or feature description
**Output:** Recommended signal structure with Leptos code scaffold

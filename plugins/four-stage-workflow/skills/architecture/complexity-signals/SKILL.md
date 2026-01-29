---
name: complexity-signals
description: Recognize the early warning signs of complexity before it becomes entrenched, including shallow modules, information leakage, and temporal decomposition.
---

# Complexity Signals

## Source
- Book: A Philosophy of Software Design
- Author: John Ousterhout
- Key Chapters: 2-3, 9-11 (Complexity, Tactical vs Strategic, Consistency)

## Core Principle

Complexity is anything that makes software hard to understand or modify. Recognize the early warning signs before complexity becomes entrenched.

## Decision Framework

```
Complexity comes from:
├── Dependencies
│   └── Code in one place affects code in another
├── Obscurity
│   └── Important information is not obvious
└── Accumulation
    └── Many small issues compound over time

Red flags indicate complexity is forming:
├── Shallow modules → Split and recombine
├── Information leakage → Redesign boundaries
├── Temporal decomposition → Group by knowledge
├── Pass-through methods → Collapse layers
├── Repetition → Extract and parameterize
└── Special cases → Generalize the design
```

## Patterns

### Pattern 1: Recognizing Shallow Modules

**Signal:** Interface nearly as complex as implementation
**When:** A class/module has many methods but each does little work
**Response:** Combine into fewer, more powerful methods

```rust
// RED FLAG: Shallow module
pub struct StringProcessor {
    pub fn trim(&self, s: &str) -> String { s.trim().to_string() }
    pub fn lowercase(&self, s: &str) -> String { s.to_lowercase() }
    pub fn remove_punctuation(&self, s: &str) -> String { /* ... */ }
    pub fn split_words(&self, s: &str) -> Vec<&str> { s.split_whitespace().collect() }
}

// FIXED: Deep module with meaningful abstraction
pub struct TextNormalizer {
    /// Normalize text for search indexing: lowercase, remove punctuation,
    /// collapse whitespace, apply stemming
    pub fn normalize(&self, text: &str) -> NormalizedText { }

    /// Normalize with custom options
    pub fn normalize_with(&self, text: &str, opts: NormalizeOptions) -> NormalizedText { }
}
```

### Pattern 2: Information Leakage

**Signal:** Same knowledge appears in multiple places
**When:** Change in one module requires changes elsewhere
**Response:** Consolidate knowledge into one module

```rust
// RED FLAG: Leakage - both modules know about file format
mod writer {
    pub fn write_record(r: &Record) -> String {
        format!("{}|{}|{}", r.id, r.name, r.value)  // Knows format
    }
}
mod reader {
    pub fn read_record(line: &str) -> Record {
        let parts: Vec<_> = line.split('|').collect();  // Also knows format
        Record { id: parts[0], name: parts[1], value: parts[2] }
    }
}

// FIXED: Format knowledge in one place
mod record_format {
    const SEPARATOR: char = '|';

    pub fn serialize(r: &Record) -> String { /* ... */ }
    pub fn deserialize(s: &str) -> Result<Record, FormatError> { /* ... */ }
}
```

### Pattern 3: Temporal Decomposition

**Signal:** Code organized by when things happen, not by what they know
**When:** Related code scattered across init/process/cleanup phases
**Response:** Group by information, not by order of execution

```rust
// RED FLAG: Temporal decomposition
mod initialization {
    pub fn init_database(config: &Config) -> Pool { }
    pub fn init_cache(config: &Config) -> Cache { }
    pub fn init_logger(config: &Config) -> Logger { }
}
mod runtime {
    pub fn run_with(pool: Pool, cache: Cache, logger: Logger) { }
}
mod cleanup {
    pub fn close_database(pool: Pool) { }
    pub fn flush_cache(cache: Cache) { }
    pub fn shutdown_logger(logger: Logger) { }
}

// FIXED: Group by knowledge
mod database {
    pub struct Database { /* owns pool */ }
    impl Database {
        pub fn connect(config: &DbConfig) -> Result<Self> { }
        pub fn query<T>(&self, q: Query<T>) -> Result<T> { }
    }
    impl Drop for Database {
        fn drop(&mut self) { /* cleanup */ }
    }
}
```

### Pattern 4: Pass-Through Variables

**Signal:** Variables passed through many layers without use
**When:** Function signature cluttered with pass-through params
**Response:** Use context objects or eliminate unnecessary layers

```rust
// RED FLAG: Pass-through variable
fn process_request(req: Request, logger: &Logger, metrics: &Metrics, config: &Config) {
    validate(req, logger, metrics, config)?;
    transform(req, logger, metrics, config)?;
    store(req, logger, metrics, config)?;
}

// FIXED: Context object
struct RequestContext {
    logger: Logger,
    metrics: Metrics,
    config: Config,
}

fn process_request(req: Request, ctx: &RequestContext) {
    validate(req, ctx)?;
    transform(req, ctx)?;
    store(req, ctx)?;
}

// Or better: Context in struct, not passed around
impl RequestProcessor {
    fn process(&self, req: Request) -> Result<()> {
        // self.logger, self.metrics, self.config available
    }
}
```

### Pattern 5: Repetition

**Signal:** Same code pattern appears multiple times
**When:** Copy-paste with small variations
**Response:** Extract and parameterize, or find the right abstraction

```rust
// RED FLAG: Repetition
fn get_user(id: &str) -> Result<User, DbError> {
    let conn = pool.get()?;
    let row = conn.query_one("SELECT * FROM users WHERE id = $1", &[&id])?;
    Ok(User::from_row(row))
}
fn get_order(id: &str) -> Result<Order, DbError> {
    let conn = pool.get()?;
    let row = conn.query_one("SELECT * FROM orders WHERE id = $1", &[&id])?;
    Ok(Order::from_row(row))
}

// FIXED: Extract the pattern
fn get_by_id<T: FromRow>(&self, table: &str, id: &str) -> Result<T, DbError> {
    let conn = self.pool.get()?;
    let row = conn.query_one(&format!("SELECT * FROM {} WHERE id = $1", table), &[&id])?;
    Ok(T::from_row(row))
}

// Or better: Derive macro for entity queries
#[derive(Entity)]
#[table = "users"]
struct User { /* ... */ }

// Usage: repo.get::<User>(id)
```

### Pattern 6: Special Cases

**Signal:** Code littered with if-statements for edge cases
**When:** Special handling scattered throughout
**Response:** Design the normal case to handle special cases, or use the Null Object pattern

```rust
// RED FLAG: Special case proliferation
fn process_user(user: Option<User>) {
    if let Some(u) = user {
        if u.is_admin() {
            process_admin(&u);
        } else if u.is_guest() {
            process_guest(&u);
        } else {
            process_regular(&u);
        }
    } else {
        process_anonymous();
    }
}

// FIXED: Polymorphism absorbs special cases
trait UserProcessor {
    fn process(&self);
}

impl UserProcessor for AdminUser { fn process(&self) { /* ... */ } }
impl UserProcessor for GuestUser { fn process(&self) { /* ... */ } }
impl UserProcessor for RegularUser { fn process(&self) { /* ... */ } }
impl UserProcessor for AnonymousUser { fn process(&self) { /* ... */ } }

// Or: Null object pattern
fn get_user(id: Option<&str>) -> Box<dyn UserProcessor> {
    match id {
        Some(id) => load_user(id).unwrap_or_else(|_| Box::new(AnonymousUser)),
        None => Box::new(AnonymousUser),
    }
}
```

## Strategic vs Tactical Programming

### Tactical Mindset (Avoid)
- "I just need to get this working"
- "I'll clean it up later"
- "This is a special case"
- Adds complexity with each change

### Strategic Mindset (Prefer)
- "What's the best design for this?"
- "How can I make this simpler?"
- "Is there a general solution?"
- Reduces complexity over time

```rust
// Tactical: Quick fix, adds complexity
fn get_name(user: &User) -> String {
    if user.name.is_empty() {
        if user.email.contains('@') {
            user.email.split('@').next().unwrap().to_string()
        } else {
            "Unknown".to_string()
        }
    } else {
        user.name.clone()
    }
}

// Strategic: Proper design
impl User {
    fn display_name(&self) -> &str {
        if !self.name.is_empty() {
            &self.name
        } else {
            &self.fallback_name  // Computed once on creation
        }
    }
}
```

## Anti-Patterns

### Classitis
Creating many small classes/modules that each do very little, fragmenting the codebase.

### Configuration Explosion
Exposing every internal decision as a configuration option instead of making sensible defaults.

### Layer Proliferation
Adding layers "for abstraction" without each layer providing real value.

### Clever Code
Code that's short but obscure, requiring significant effort to understand.

## Checklist

When reviewing code for complexity:

- [ ] Could someone understand this without reading the implementation?
- [ ] Is the same information encoded in multiple places?
- [ ] Are there pass-through methods or variables?
- [ ] Is code organized by time of execution rather than knowledge?
- [ ] Are there special cases that could be generalized?
- [ ] Is there repeated code that could be extracted?
- [ ] Does each layer provide real value?

## Claude Code Integration

**Command:** /complexity-check
**Input:** Code or module to analyze
**Output:** Red flags identified, specific refactoring suggestions

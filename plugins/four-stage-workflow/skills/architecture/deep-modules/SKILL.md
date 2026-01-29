---
name: deep-modules
description: A deep module provides powerful functionality behind a simple interface, maximizing the ratio of implementation depth to interface complexity.
---

# Deep Modules

## Source
- Book: A Philosophy of Software Design
- Author: John Ousterhout
- Key Chapters: 4-6 (Modules Should Be Deep, Information Hiding, General-Purpose Modules)

## Core Principle

A deep module provides powerful functionality behind a simple interface. The best modules have small interfaces relative to the functionality they provide.

## Decision Framework

```
Evaluating a module:
├── Interface complexity (what callers must understand)
│   ├── Number of methods
│   ├── Number of parameters per method
│   ├── Types callers must construct
│   └── Exceptions/errors callers must handle
│
├── Implementation depth (functionality provided)
│   ├── Lines of code hidden
│   ├── Edge cases handled internally
│   ├── Complexity absorbed
│   └── Decisions made for the caller
│
└── Depth Ratio = Implementation Depth / Interface Complexity
    ├── High ratio → Deep module (good)
    └── Low ratio → Shallow module (bad)
```

## Patterns

### Pattern 1: Deep Interface Design

**When:** Designing a new module or refactoring an existing one
**Apply:** Maximize functionality per interface element

```rust
// SHALLOW: Exposes implementation details
pub struct FileCache {
    pub entries: HashMap<PathBuf, CacheEntry>,
    pub max_size: usize,
    pub eviction_policy: EvictionPolicy,
}

impl FileCache {
    pub fn insert(&mut self, path: PathBuf, entry: CacheEntry) { }
    pub fn get(&self, path: &Path) -> Option<&CacheEntry> { }
    pub fn remove(&mut self, path: &Path) { }
    pub fn evict_if_needed(&mut self) { }
    pub fn set_max_size(&mut self, size: usize) { }
    pub fn set_policy(&mut self, policy: EvictionPolicy) { }
}

// DEEP: Simple interface, complex behavior hidden
pub struct FileCache { /* private fields */ }

impl FileCache {
    /// Create a cache with sensible defaults
    pub fn new() -> Self { }

    /// Create with custom configuration
    pub fn with_config(config: CacheConfig) -> Self { }

    /// Get file contents, loading and caching automatically
    pub fn get(&self, path: &Path) -> Result<Arc<[u8]>, CacheError> { }

    /// Invalidate a specific entry
    pub fn invalidate(&self, path: &Path) { }
}
// Eviction, size management, LRU tracking all handled internally
```

### Pattern 2: Absorb Complexity

**When:** Callers would otherwise need to handle many edge cases
**Apply:** Handle edge cases inside the module, not at every call site

```rust
// SHALLOW: Every caller handles edge cases
pub fn read_config_file(path: &Path) -> io::Result<String> {
    std::fs::read_to_string(path)
}
// Caller must handle: file not found, permissions, encoding, etc.

// DEEP: Edge cases absorbed
pub fn load_config(path: &Path) -> Result<Config, ConfigError> {
    let content = match std::fs::read_to_string(path) {
        Ok(c) => c,
        Err(e) if e.kind() == ErrorKind::NotFound => {
            return Ok(Config::default())  // Absorbed: missing file = defaults
        }
        Err(e) => return Err(ConfigError::Read { path: path.into(), source: e }),
    };

    let config: Config = toml::from_str(&content)
        .map_err(|e| ConfigError::Parse { path: path.into(), source: e })?;

    // Absorbed: validation, defaults for missing fields
    config.with_defaults().validate()
}
```

### Pattern 3: Hiding Temporal Complexity

**When:** Operations have complex timing, ordering, or lifecycle requirements
**Apply:** Let the module manage the complexity internally

```rust
// SHALLOW: Caller manages lifecycle
pub struct Database {
    pool: Pool<Postgres>,
}

impl Database {
    pub fn new(url: &str) -> Self { }
    pub async fn connect(&self) -> Result<()> { }
    pub fn get_connection(&self) -> PooledConnection { }
    pub fn return_connection(&self, conn: PooledConnection) { }
    pub async fn health_check(&self) -> bool { }
    pub async fn reconnect_if_needed(&self) -> Result<()> { }
}

// DEEP: Lifecycle managed internally
pub struct Database { /* private */ }

impl Database {
    pub async fn connect(config: DatabaseConfig) -> Result<Self, DbError> {
        // Connection, pooling, retry, health checks all internal
    }

    pub async fn execute<T>(&self, query: Query<T>) -> Result<T, DbError> {
        // Automatically: get connection, execute, return connection,
        // handle transient failures, reconnect if needed
    }
}
```

### Pattern 4: General-Purpose Modules

**When:** Building a component that might be reused
**Apply:** Design for the general case, even if currently used specifically

```rust
// TOO SPECIFIC: Only works for one use case
pub struct UserValidator {
    pub fn validate_username(name: &str) -> bool { }
    pub fn validate_user_email(email: &str) -> bool { }
}

// GENERAL-PURPOSE: Reusable across domains
pub struct Validator<T> {
    rules: Vec<Box<dyn Rule<T>>>,
}

impl<T> Validator<T> {
    pub fn new() -> Self { }
    pub fn rule(mut self, rule: impl Rule<T> + 'static) -> Self { }
    pub fn validate(&self, value: &T) -> ValidationResult { }
}

// Specific validators built from general tool
let username_validator = Validator::new()
    .rule(MinLength(3))
    .rule(MaxLength(20))
    .rule(Pattern(r"^[a-zA-Z0-9_]+$"));
```

### Pattern 5: Somewhat General-Purpose

**When:** You're unsure how general to make something
**Apply:** Make it somewhat more general than the immediate need, but don't over-engineer

```rust
// Current need: Retry HTTP requests
// Over-general: Generic retry framework for anything
// Somewhat general: Retry for async operations

pub struct RetryPolicy {
    max_attempts: u32,
    initial_delay: Duration,
    backoff_factor: f64,
    max_delay: Duration,
}

impl RetryPolicy {
    pub async fn execute<T, E, F, Fut>(&self, operation: F) -> Result<T, E>
    where
        F: Fn() -> Fut,
        Fut: Future<Output = Result<T, E>>,
        E: std::fmt::Debug,
    {
        // Retry logic here
    }
}

// Works for HTTP, database, file operations, etc.
// Not over-engineered with callbacks, events, custom schedulers
```

## Anti-Patterns

### Shallow Modules (Classitis)

```rust
// BAD: One-liner methods that just forward
impl UserService {
    pub fn get_user(&self, id: &str) -> User {
        self.repository.get_user(id)
    }
    pub fn save_user(&self, user: User) {
        self.repository.save_user(user)
    }
}

// Why is this bad?
// - No value added over calling repository directly
// - Interface is as complex as implementation
// - Caller gains nothing from the abstraction
```

### Pass-Through Methods

```rust
// BAD: Parameters flow through unchanged
impl OrderProcessor {
    pub fn process(&self, order: Order, options: ProcessOptions) -> Receipt {
        self.validator.validate(&order, &options)?;
        self.calculator.calculate(&order, &options)?;
        self.executor.execute(&order, &options)
    }
}

// Better: Absorb the orchestration, simplify the interface
impl OrderProcessor {
    pub fn process(&self, order: Order) -> Result<Receipt, ProcessError> {
        // Options determined internally from order
    }
}
```

### Leaky Abstractions

```rust
// BAD: Internal types leak out
pub struct Parser {
    pub fn parse(&self, input: &str) -> Vec<Token> { }  // Token is internal
    pub fn set_lexer(&mut self, lexer: Lexer) { }       // Lexer is internal
}

// Better: Hide implementation types
pub struct Parser { /* private */ }

impl Parser {
    pub fn parse(&self, input: &str) -> Result<Document, ParseError> { }
}
```

## Checklist

When designing or reviewing a module:

- [ ] Can a caller use this with minimal understanding of internals?
- [ ] Are edge cases handled inside, not pushed to callers?
- [ ] Is the interface smaller than what I'd expect for this functionality?
- [ ] Are internal types hidden from the public interface?
- [ ] Does the module make decisions that callers shouldn't need to make?
- [ ] Could a new team member use this module correctly on first try?

## Claude Code Integration

**Command:** /complexity-check
**Input:** Module code to evaluate
**Output:** Depth assessment, shallow module warnings, refactoring suggestions

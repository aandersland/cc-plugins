---
name: state-machines
description: Make state explicit and transitions visible to ensure illegal states are unrepresentable and all transitions are intentional.
---

# State Machines

## Source
- Book: Practical UML Statecharts for C/C++ (Samek)
- Concept: Hierarchical State Machines (HSM) for complex behavior
- Application: UI state, connection management, workflows

## Core Principle

Make state explicit and transitions visible. A well-designed state machine makes illegal states unrepresentable and ensures all transitions are intentional.

## Decision Framework

```
Should this be a state machine?
├── Does the system have distinct modes of operation?
│   └── Yes → State machine candidate
├── Are there complex rules about valid transitions?
│   └── Yes → State machine prevents invalid transitions
├── Is debugging "how did we get here?" difficult?
│   └── Yes → State machine provides audit trail
├── Do you have boolean flags like is_loading, is_connected?
│   └── Yes → Replace with explicit state enum
└── Are there nested or hierarchical behaviors?
    └── Yes → Consider hierarchical state machine
```

## Patterns

### Pattern 1: Explicit State Enum

**When:** Boolean flags or status strings control behavior
**Apply:** Replace with a typed state enum

```rust
// BEFORE: Boolean flags
struct Connection {
    is_connecting: bool,
    is_connected: bool,
    is_authenticated: bool,
    has_error: bool,
    error_message: Option<String>,
}
// Problem: What does is_connecting=true, is_connected=true mean?

// AFTER: Explicit state
#[derive(Debug, Clone, Copy, PartialEq)]
enum ConnectionState {
    Disconnected,
    Connecting { attempt: u32 },
    Connected,
    Authenticating,
    Ready,
    Error { code: ErrorCode },
}

struct Connection {
    state: ConnectionState,
    // ... other fields that don't affect state
}
```

### Pattern 2: Transition Table

**When:** Complex transition rules need to be understood at a glance
**Apply:** Define transitions explicitly, reject invalid ones

```rust
#[derive(Debug, Clone, Copy)]
enum Event {
    Connect,
    Disconnect,
    AuthSuccess,
    AuthFailure,
    Timeout,
    Retry,
}

impl ConnectionState {
    /// Attempt to transition based on event.
    /// Returns new state or None if transition is invalid.
    fn transition(self, event: Event) -> Option<ConnectionState> {
        use ConnectionState::*;
        use Event::*;

        match (self, event) {
            // From Disconnected
            (Disconnected, Connect) => Some(Connecting { attempt: 1 }),

            // From Connecting
            (Connecting { .. }, Disconnect) => Some(Disconnected),
            (Connecting { attempt }, Timeout) if attempt < 3 => {
                Some(Connecting { attempt: attempt + 1 })
            }
            (Connecting { .. }, Timeout) => Some(Error { code: ErrorCode::Timeout }),

            // From Connected
            (Connected, Disconnect) => Some(Disconnected),
            (Connected, AuthSuccess) => Some(Ready),
            (Connected, AuthFailure) => Some(Error { code: ErrorCode::Auth }),

            // From Ready
            (Ready, Disconnect) => Some(Disconnected),

            // From Error
            (Error { .. }, Retry) => Some(Connecting { attempt: 1 }),
            (Error { .. }, Disconnect) => Some(Disconnected),

            // Invalid transitions return None
            _ => None,
        }
    }
}
```

### Pattern 3: State-Specific Data

**When:** Different states carry different associated data
**Apply:** Use enum variants with data fields

```rust
enum DownloadState {
    Idle,
    Fetching {
        url: String,
        started_at: Instant,
    },
    Downloading {
        url: String,
        total_bytes: u64,
        downloaded_bytes: u64,
        speed_bps: f64,
    },
    Paused {
        url: String,
        downloaded_bytes: u64,
        paused_at: Instant,
    },
    Complete {
        path: PathBuf,
        total_bytes: u64,
        duration: Duration,
    },
    Failed {
        url: String,
        error: DownloadError,
        downloaded_bytes: u64,
    },
}

impl DownloadState {
    fn progress_percent(&self) -> Option<f64> {
        match self {
            Self::Downloading { total_bytes, downloaded_bytes, .. } => {
                Some(*downloaded_bytes as f64 / *total_bytes as f64 * 100.0)
            }
            Self::Complete { .. } => Some(100.0),
            _ => None,
        }
    }
}
```

### Pattern 4: Entry/Exit Actions

**When:** Actions must happen when entering or leaving states
**Apply:** Implement actions in the transition method

```rust
impl Connection {
    fn transition(&mut self, event: Event) -> Result<(), TransitionError> {
        let old_state = self.state;
        let new_state = self.state.transition(event)
            .ok_or(TransitionError::Invalid { from: old_state, event })?;

        // Exit actions
        match old_state {
            ConnectionState::Ready => {
                self.stop_heartbeat();
            }
            ConnectionState::Connecting { .. } => {
                self.cancel_connect_timeout();
            }
            _ => {}
        }

        // Update state
        self.state = new_state;

        // Entry actions
        match new_state {
            ConnectionState::Connecting { attempt } => {
                self.start_connect_timeout(attempt);
                self.metrics.increment("connection.attempts");
            }
            ConnectionState::Ready => {
                self.start_heartbeat();
                self.metrics.increment("connection.established");
            }
            ConnectionState::Error { code } => {
                self.metrics.increment(&format!("connection.error.{:?}", code));
            }
            _ => {}
        }

        // Log transition
        info!(
            from = ?old_state,
            to = ?new_state,
            trigger = ?event,
            "state_transition"
        );

        Ok(())
    }
}
```

### Pattern 5: Hierarchical States

**When:** States share common behavior or transitions
**Apply:** Nest states within parent states

```rust
// Hierarchical: Active state contains sub-states
enum AppState {
    Starting,
    Active(ActiveState),
    ShuttingDown,
}

enum ActiveState {
    Idle,
    Processing { job_id: String },
    Paused { reason: PauseReason },
}

impl AppState {
    fn can_shutdown(&self) -> bool {
        // All Active sub-states can transition to ShuttingDown
        matches!(self, AppState::Active(_))
    }

    fn transition(&self, event: AppEvent) -> Option<AppState> {
        match (self, event) {
            // Parent-level transitions
            (AppState::Starting, AppEvent::Ready) => {
                Some(AppState::Active(ActiveState::Idle))
            }
            (AppState::Active(_), AppEvent::Shutdown) => {
                Some(AppState::ShuttingDown)
            }

            // Sub-state transitions
            (AppState::Active(active), event) => {
                active.transition(event).map(AppState::Active)
            }

            _ => None,
        }
    }
}
```

### Pattern 6: Guards and Conditions

**When:** Transitions depend on conditions beyond just current state
**Apply:** Add guard conditions to transitions

```rust
impl Order {
    fn transition(&mut self, event: OrderEvent) -> Result<(), OrderError> {
        let new_state = match (&self.state, &event) {
            // Guard: Can only confirm if items are in stock
            (OrderState::Pending, OrderEvent::Confirm) if self.items_in_stock() => {
                OrderState::Confirmed
            }
            (OrderState::Pending, OrderEvent::Confirm) => {
                return Err(OrderError::OutOfStock);
            }

            // Guard: Can only ship if payment received
            (OrderState::Confirmed, OrderEvent::Ship) if self.payment.is_complete() => {
                OrderState::Shipped { tracking: self.generate_tracking() }
            }
            (OrderState::Confirmed, OrderEvent::Ship) => {
                return Err(OrderError::PaymentIncomplete);
            }

            // ... other transitions
        };

        self.state = new_state;
        Ok(())
    }
}
```

## Anti-Patterns

### Boolean Flag Soup
```rust
// AVOID: Unclear what combinations are valid
struct Widget {
    is_visible: bool,
    is_enabled: bool,
    is_loading: bool,
    is_error: bool,
    is_selected: bool,
}

// Better: Explicit states
enum WidgetState {
    Hidden,
    Loading,
    Ready { selected: bool },
    Error { message: String },
    Disabled,
}
```

### Implicit Transitions
```rust
// AVOID: State changed without clear transition
fn process(&mut self) {
    if self.is_connected {
        self.is_processing = true;  // Implicit transition
        // ...
    }
}

// Better: Explicit transition
fn process(&mut self) -> Result<()> {
    self.transition(Event::StartProcessing)?;
    // ...
}
```

### God States
```rust
// AVOID: One state that means too many things
enum State {
    Active,  // But active doing what?
}

// Better: Specific states
enum State {
    ProcessingInput,
    WaitingForNetwork,
    RenderingOutput,
    Idle,
}
```

## Checklist

When implementing a state machine:

- [ ] All possible states are enumerated
- [ ] State carries only relevant data for that state
- [ ] All valid transitions are explicitly defined
- [ ] Invalid transitions are rejected with clear errors
- [ ] Entry/exit actions are implemented where needed
- [ ] State transitions are logged for debugging
- [ ] No boolean flags that could be states instead

## Claude Code Integration

**Command:** /state-model
**Input:** User flow or feature description
**Output:** State enum, transition table, and scaffolded implementation

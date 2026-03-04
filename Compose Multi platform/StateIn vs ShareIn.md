🔥 stateIn vs shareIn – Advanced Production Guide

यह guide Kotlin Flow के दो सबसे powerful operators
"stateIn" और "shareIn" को deep production level पर explain करती है।

यह सिर्फ difference नहीं —
बल्कि Architecture, Memory, Lifecycle, Performance, और Enterprise Patterns cover करेगी।

---

🧠 Why We Need Them?

By default:

val flow = repository.getPosts()

यह एक ❄️ Cold Flow है।

हर collector:

- API call दुबारा
- DB query दुबारा
- Execution दुबारा

👉 Multi-collector apps में inefficient।

Solution?

Convert Cold → Hot

- "stateIn"
- "shareIn"

---

🎯 Quick Definition

🔷 stateIn

«Cold Flow को StateFlow में convert करता है।»

✔ Always has a value
✔ Holds latest state
✔ Best for UI state

---

🔶 shareIn

«Cold Flow को SharedFlow में convert करता है।»

✔ No mandatory initial value
✔ Can replay multiple values
✔ Best for events / broadcast

---

📊 Core Difference Table

Feature| stateIn| shareIn
Returns| StateFlow| SharedFlow
Initial value| Required| Not required
Holds latest value| Yes| Optional
Replay count| Always 1| Configurable
Best for| UI State| Events

---

🏗 Architecture Position

Repository → Cold Flow
ViewModel → stateIn/shareIn
UI → Collect

---

🔥 stateIn Deep Dive

Example:

val uiState: StateFlow<PostState> =
    repository.getPosts()
        .map { PostState.Success(it) }
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = PostState.Loading
        )

---

🧠 What Happens Internally?

- Upstream flow shared
- Latest value stored
- New collectors get latest immediately
- Behaves like observable state

---

🎯 stateIn Best Use Cases

- Screen state
- UI state machine
- Form state
- Dashboard data
- Pagination state

---

⚠ stateIn Memory Behavior

StateFlow:

- Always keeps last value in memory
- Even when no collectors (depends on SharingStarted)

Use:

SharingStarted.WhileSubscribed(5000)

To avoid memory leaks.

---

🔶 shareIn Deep Dive

Example:

val events: SharedFlow<PostEvent> =
    repository.postEvents()
        .shareIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(),
            replay = 0
        )

---

🧠 What Happens Internally?

- Upstream flow shared
- No mandatory state storage
- Can replay N previous emissions
- Broadcast style

---

🎯 shareIn Best Use Cases

- Navigation events
- Snackbar messages
- One-time UI effects
- WebSocket streams
- Multi-subscriber analytics

---

🎭 Replay Behavior

stateIn

Always replay = 1
Cannot change

---

shareIn

Configurable:

replay = 0   // no replay
replay = 1   // last value
replay = 5   // last 5 values

---

🚨 Production Mistakes

❌ Using shareIn for UI state
❌ Using stateIn for one-time events
❌ Using Eagerly without reason
❌ Large replay causing memory issue
❌ Not understanding lifecycle

---

🧠 SharingStarted Strategies

1️⃣ Eagerly

SharingStarted.Eagerly

- Start immediately
- Never stops
- Risky for memory

---

2️⃣ Lazily

SharingStarted.Lazily

- Start on first collector
- Never stops

---

3️⃣ WhileSubscribed

SharingStarted.WhileSubscribed(5000)

- Start when collector exists
- Stop after timeout
- Best for Android

---

🏗 Enterprise Pattern

UI State Pattern

private val _uiState =
    repository.getPosts()
        .map { PostState.Success(it) }
        .stateIn(
            viewModelScope,
            SharingStarted.WhileSubscribed(5000),
            PostState.Loading
        )

val uiState = _uiState

---

Event Pattern

private val _events =
    repository.events()
        .shareIn(
            viewModelScope,
            SharingStarted.WhileSubscribed(),
            replay = 0
        )

val events = _events

---

🔥 Advanced Scenario: Multi Screen

Case:

- 3 screens observe same flow
- API should not be called 3 times

Solution:

val sharedPosts =
    repository.getPosts()
        .shareIn(
            appScope,
            SharingStarted.Eagerly,
            replay = 1
        )

App-level sharing.

---

🧠 stateIn vs shareIn Mental Model

stateIn = Database of current state
shareIn = Radio broadcast system

---

🎯 Performance Comparison

Concern| stateIn| shareIn
Memory| Holds 1 value| Holds replay count
CPU| Same| Same
Use in ViewModel| Yes| Yes
Use in Repository| Rare| Sometimes

---

🧪 Testing Differences

stateIn:

uiState.test {
    assertEquals(PostState.Loading, awaitItem())
}

shareIn:

events.test {
    awaitItem()
}

---

🔥 Advanced Cold → Hot Pattern

val hotFlow =
    coldFlow
        .onStart { emit(default) }
        .shareIn(...)

But for state → prefer stateIn.

---

🚦 Decision Guide

Use stateIn when:

- You need state
- UI observes it
- Latest value must be remembered

Use shareIn when:

- You need broadcast
- Events
- Multiple collectors
- Replay customization needed

---

🏁 Final Summary

Modern Android (2026 standard):

Repository → Cold Flow
ViewModel → stateIn (UI State)
ViewModel → shareIn (Events)
UI → Collect

Golden Rule:

- UI State = stateIn
- One-time Event = shareIn

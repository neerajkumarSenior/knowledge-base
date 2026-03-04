🔥 Hot vs Cold Flow Deep Dive (HotVsColdFlow.md)

यह guide Kotlin Coroutines में Hot Flow vs Cold Flow को production perspective से समझाती है —
Architecture, performance, lifecycle, और real-world Android use cases के साथ।

---

🧠 Basic Definition

❄️ Cold Flow

«Collector आने तक flow execute नहीं होता।»

हर collector के लिए नया execution होता है।

Example:

val flow = flow {
    println("Flow started")
    emit(api.getPosts())
}

जब collect होगा तभी run होगा।

---

🔥 Hot Flow

«Flow हमेशा active रहता है, चाहे कोई collect करे या नहीं।»

Example:

val state = MutableStateFlow(PostState())

StateFlow हमेशा value hold करता है।

---

🎯 Visual Comparison

Cold Flow:
Collector → Start → Emit → Complete

Hot Flow:
Start → Emit → Emit → Emit
           ↑
      Collector joins anytime

---

📊 Core Differences

Feature| Cold Flow| Hot Flow
Starts when| collect()| Creation time
Multiple collectors| New execution| Same stream
Holds state| ❌ No| Depends (StateFlow yes)
Replay| ❌| Configurable
Example| flow { }| StateFlow, SharedFlow

---

🧩 Android Architecture Context

Repository → Cold Flow
ViewModel → Convert to Hot Flow
UI → Collect

---

📦 Cold Flow Example (Repository Layer)

fun getPosts(): Flow<List<Post>> = flow {
    val posts = api.getPosts()
    emit(posts)
}

Every time collect:

- API call again
- Fresh execution

---

🔥 Convert Cold → Hot (stateIn)

val postsState =
    repository.getPosts()
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(),
            initialValue = emptyList()
        )

Now:

- API not re-called for each collector
- State shared
- Becomes hot StateFlow

---

🧠 Cold Flow Characteristics

✅ Lazy
✅ Re-executes per collector
✅ Ideal for API calls
✅ Ideal for database query
✅ No memory overhead until collected

---

🧠 Hot Flow Characteristics

✅ Always active
✅ Shared between collectors
✅ Can hold state (StateFlow)
✅ Broadcast style (SharedFlow)
❌ Can waste memory if misused

---

🔄 Real World Example (Search)

❄️ Cold

fun search(query: String): Flow<List<Post>> = flow {
    emit(api.search(query))
}

Every collect = new API call.

---

🔥 Hot

val searchResults =
    searchQuery
        .debounce(500)
        .flatMapLatest { query ->
            repository.search(query)
        }
        .stateIn(viewModelScope, SharingStarted.Lazily, emptyList())

Single shared stream.

---

🎯 When to Use Cold Flow?

Use Cold Flow when:

- API call
- DB query
- One-time computation
- Heavy operation
- Need fresh result each time

Repository layer mostly cold.

---

🎯 When to Use Hot Flow?

Use Hot Flow when:

- UI state
- Shared state
- Event broadcasting
- Continuous stream (location, websocket)
- Multiple collectors

ViewModel layer mostly hot.

---

🧪 Testing Perspective

Cold Flow:

val result = repository.getPosts().first()

Hot Flow:

viewModel.state.test {
    assertEquals(expected, awaitItem())
}

---

🚨 Common Mistakes

❌ Using Cold Flow for UI state
❌ Using Hot Flow for heavy API logic
❌ Forgetting stateIn
❌ Not understanding replay behavior
❌ Memory leak via forever-hot flow

---

🔥 shareIn vs stateIn

Both convert cold → hot.

Operator| Returns| Use Case
stateIn| StateFlow| UI state
shareIn| SharedFlow| Events / broadcast

---

🧠 Lifecycle & SharingStarted

SharingStarted.WhileSubscribed(5000)

Meaning:

- Stop upstream after 5s if no collector
- Prevent memory waste

Other options:

SharingStarted.Eagerly
SharingStarted.Lazily

---

🏗 Enterprise Architecture Pattern

Data Layer → Cold Flow
Domain Layer → Transform
ViewModel → stateIn / shareIn
UI → Collect

---

🎯 Performance Perspective

Cold Flow:

- Better memory usage
- Re-execution cost

Hot Flow:

- Less re-execution
- More memory retention
- Requires lifecycle awareness

---

🔥 Fintech / Ride App Scenario

Example: Live ride status

Cold:

- API polling per screen → wasteful

Hot:

- Single shared StateFlow
- All screens observe same stream

Correct approach → Hot Flow

---

🧠 Mental Model

Cold Flow = Netflix on demand
Hot Flow = Live TV broadcast

---

🏁 Final Summary

Cold Flow:

- Lazy
- Per collector execution
- Best for API/DB

Hot Flow:

- Active stream
- Shared state
- Best for UI & events

Modern Android Pattern (2026):

Repository → Cold Flow
ViewModel → Hot Flow (stateIn/shareIn)
UI → Collect

---

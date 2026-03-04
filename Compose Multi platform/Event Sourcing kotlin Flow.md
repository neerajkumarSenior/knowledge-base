🧾⚡ Event Sourcing + Kotlin Flow

Enterprise Reactive Architecture (Advanced)

यह guide समझाती है कि Event Sourcing क्या है और इसे Kotlin Flow + Offline-First + WebSocket architecture के साथ कैसे combine किया जाता है।

यह pattern use होता है:

- 🏦 Fintech apps
- 💬 Chat systems
- 📊 Trading dashboards
- 🛒 Order systems
- 🎮 Multiplayer state systems

---

🧠 Event Sourcing क्या है?

Traditional system:

Database → Stores Current State

Event Sourcing system:

Database → Stores Events (NOT state)
State = Derived from events

---

📦 Example (Bank Account)

Instead of storing:

Balance = 5000

We store:

Deposit 1000
Withdraw 500
Deposit 4500

Balance is calculated by replaying events.

---

🏗 Traditional vs Event Sourcing

Traditional| Event Sourcing
Stores current state| Stores all changes
Easy queries| Full history
Hard auditing| Perfect audit trail
Hard rollback| Easy replay

---

🔥 Why Combine With Flow?

Event stream naturally fits Kotlin Flow:

val eventFlow: Flow<DomainEvent>

Flow = Event stream
Event Sourcing = Event persistence

Perfect match.

---

🏗 High-Level Architecture

User Action
    ↓
Command
    ↓
Event Created
    ↓
Event Store (DB)
    ↓
Event Stream (Flow)
    ↓
Reducer
    ↓
Current State
    ↓
UI

---

🧩 Core Components

1️⃣ Domain Event

sealed class PostEvent {
    data class Created(val id: String, val title: String) : PostEvent()
    data class Updated(val id: String, val title: String) : PostEvent()
    data class Deleted(val id: String) : PostEvent()
}

---

2️⃣ Event Entity (Room)

@Entity
data class PostEventEntity(
    @PrimaryKey(autoGenerate = true) val eventId: Long = 0,
    val postId: String,
    val type: String,
    val payload: String,
    val version: Int,
    val timestamp: Long
)

---

3️⃣ Event DAO

@Query("SELECT * FROM PostEventEntity ORDER BY eventId ASC")
fun observeEvents(): Flow<List<PostEventEntity>>

---

🔄 State Reconstruction (Reducer)

Reducer converts events → state.

fun reduce(events: List<PostEvent>): List<Post> {
    val state = mutableMapOf<String, Post>()

    events.forEach { event ->
        when (event) {
            is PostEvent.Created ->
                state[event.id] = Post(event.id, event.title)

            is PostEvent.Updated ->
                state[event.id]?.let {
                    state[event.id] = it.copy(title = event.title)
                }

            is PostEvent.Deleted ->
                state.remove(event.id)
        }
    }

    return state.values.toList()
}

---

🌊 Reactive State Flow

val uiState =
    dao.observeEvents()
        .map { entities ->
            entities.map { it.toDomainEvent() }
        }
        .map { events ->
            reduce(events)
        }
        .stateIn(
            viewModelScope,
            SharingStarted.WhileSubscribed(5000),
            emptyList()
        )

Database → Flow → Reduce → StateFlow → UI

Fully reactive.

---

🔥 Command → Event Flow

User clicks "Update Post"

suspend fun updatePost(id: String, title: String) {
    val event = PostEvent.Updated(id, title)
    dao.insert(event.toEntity())
}

UI auto updates because DB changed.

---

🌐 WebSocket + Event Sourcing

When server sends event:

socket.observeEvents()
    .onEach { event ->
        dao.insert(event.toEntity())
    }
    .launchIn(appScope)

No direct merge logic needed.

Why?

Because:

Events are append-only
Replay always deterministic

---

🛡 Conflict Handling in Event Sourcing

Use:

Aggregate ID + Version

Each event has:

val version: Int

If incoming event.version <= currentVersion → ignore

Prevents duplication & conflict.

---

🔁 Snapshot Optimization (Very Important)

Problem:

100,000 events → replay slow

Solution:

Store snapshot:

@Entity
data class PostSnapshot(
    val postId: String,
    val stateJson: String,
    val lastEventId: Long
)

Load:

1. Snapshot
2. Replay only events after snapshot

Enterprise systems always use snapshots.

---

🧠 Full Enterprise Event Flow

Command
   ↓
Event
   ↓
Event Store (Append Only)
   ↓
Event Stream (Flow)
   ↓
Reducer
   ↓
Snapshot
   ↓
UI

---

📡 Offline-First + Event Sourcing

User offline:

- Events stored locally
- Marked PENDING
- Background sync pushes events
- Server broadcasts back
- Version verified
- No data loss

---

⚡ Advanced Pattern: CQRS + Flow

Separate:

- Write Model → Event Store
- Read Model → Projection table

Event Store → Projector → Read Table → UI

Projector listens via Flow.

---

🎯 When to Use Event Sourcing?

✔ Financial transactions
✔ Audit-critical systems
✔ Chat systems
✔ Collaborative editing
✔ Trading systems
✔ Order lifecycle tracking

---

🚨 When NOT to Use

❌ Simple CRUD app
❌ Small data
❌ No audit need
❌ No history requirement

Event sourcing increases complexity.

---

🧠 Mental Model

Traditional DB = Whiteboard
Event Store = CCTV camera

Whiteboard shows final state
CCTV shows entire history

---

🏁 Final Architecture (2026 Enterprise)

AppScope
   └── WebSocket (Event Stream)
          └── Append Event Store
                 └── Flow observeEvents()
                        └── Reducer
                               └── Snapshot
                                      └── stateIn
                                             └── UI

---

🎓 Key Benefits

✔ Full history
✔ Perfect auditing
✔ Time travel debugging
✔ Easy rollback
✔ Deterministic replay
✔ Reactive by design

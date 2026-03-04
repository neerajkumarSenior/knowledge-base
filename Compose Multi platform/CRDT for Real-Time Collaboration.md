🔗⚡ CRDT for Real-Time Collaboration

(Conflict-Free Replicated Data Types – Android + Flow + Offline-First)

यह guide explain करती है:

- CRDT क्या है
- Real-time collaboration में क्यों use होता है
- WebSocket + Offline-First के साथ कैसे integrate करें
- Kotlin Flow architecture में इसे कैसे design करें

Used in:

- 📝 Google Docs–style editors
- 💬 Chat apps
- 🗂 Shared task boards
- 🎨 Collaborative whiteboards
- 🧠 Notion-like apps

---

🧠 Problem: Real-Time + Offline + Multi-User

Scenario:

- User A offline edit करता है
- User B online same document edit करता है
- Network reconnect होता है
- Conflict आता है

Traditional merge strategies fail.

Need:

Automatic + Deterministic + No Data Loss

Solution → CRDT

---

🔥 What is CRDT?

«Conflict-Free Replicated Data Type
ऐसा data structure जो multiple devices पर independently update हो सकता है
और merge करने पर always same result देता है।»

Properties:

✔ Commutative
✔ Associative
✔ Idempotent

Meaning:

Order doesn’t matter
Repeat doesn’t matter
Merge always safe

---

🏗 High-Level CRDT Architecture

Device A         Device B
   ↓                 ↓
Local CRDT       Local CRDT
   ↓                 ↓
Event Log         Event Log
   ↓                 ↓
WebSocket Sync (Bidirectional)
   ↓
Merge CRDT
   ↓
Same Final State

---

🧩 Types of CRDT

1️⃣ G-Counter (Grow-only Counter)

Only increments.

class GCounter {
    private val counts = mutableMapOf<String, Int>()

    fun increment(nodeId: String) {
        counts[nodeId] = (counts[nodeId] ?: 0) + 1
    }

    fun value(): Int = counts.values.sum()

    fun merge(other: GCounter) {
        other.counts.forEach { (node, count) ->
            counts[node] = maxOf(counts[node] ?: 0, count)
        }
    }
}

Used in: likes, votes, analytics.

---

2️⃣ LWW-Register (Last Write Wins)

Keeps latest timestamp.

data class LWWRegister<T>(
    val value: T,
    val timestamp: Long
)

fun merge(a: LWWRegister<String>, b: LWWRegister<String>) =
    if (a.timestamp > b.timestamp) a else b

Used in: simple fields.

⚠ Clock sync issue possible.

---

3️⃣ OR-Set (Observed-Remove Set)

Add/remove without conflict.

Each item has unique ID.

Used in:

- Todo lists
- Shared tags
- Collaborative boards

---

4️⃣ RGA (Replicated Growable Array)

Used for collaborative text editors.

Each character has:

- Unique ID
- Position reference

Used in:

- Google Docs
- Notion
- Code editors

---

🔄 CRDT + Kotlin Flow

We store operations as events:

sealed class TextOperation {
    data class Insert(val id: String, val char: Char, val afterId: String?) : TextOperation()
    data class Delete(val id: String) : TextOperation()
}

Room:

@Query("SELECT * FROM TextOperationEntity ORDER BY timestamp ASC")
fun observeOperations(): Flow<List<TextOperationEntity>>

---

🧠 State Reconstruction via Reducer

fun reduce(operations: List<TextOperation>): String {
    val chars = mutableMapOf<String, Char>()

    operations.forEach {
        when (it) {
            is TextOperation.Insert -> chars[it.id] = it.char
            is TextOperation.Delete -> chars.remove(it.id)
        }
    }

    return chars.values.joinToString("")
}

Flow:

val documentState =
    dao.observeOperations()
        .map { reduce(it.map { e -> e.toDomain() }) }
        .stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), "")

---

🌐 WebSocket Sync with CRDT

When receiving remote operation:

socket.observeOperations()
    .onEach { op ->
        dao.insert(op.toEntity())
    }
    .launchIn(appScope)

No conflict resolution logic needed.

CRDT guarantees safe merge.

---

📡 Offline Support

Offline:

- Operations stored locally
- Marked as PENDING
- Sync later
- Merge automatically safe

No rollback required.

---

🛡 Why CRDT Works

Because:

Merge(A, B) = Merge(B, A)

No central authority required.

Perfect for peer-to-peer systems.

---

⚡ Advanced Architecture (Enterprise)

User Action
   ↓
Create Operation
   ↓
Store Locally (Room)
   ↓
Flow Emits
   ↓
Reducer Builds State
   ↓
UI Updates Instantly
   ↓
WebSocket Sync
   ↓
Other Devices Merge

---

🧠 CRDT vs Event Sourcing

Event Sourcing| CRDT
Centralized merge logic| Distributed merge
Version-based conflict| Math-based conflict free
Needs conflict strategy| No conflict logic
Better for backend| Better for collaboration

---

🚀 Real-World Example

Google Docs–style Editor

Each character:

ID: deviceId + counter

Insert:

Insert("A", after=ID123)

Delete:

Delete(ID123)

Merge safe across devices.

---

⚠ Challenges

❌ Complex implementation
❌ Larger metadata storage
❌ Hard debugging
❌ Text CRDTs are advanced
❌ Performance tuning needed

Enterprise apps often use:

- Automerge (JS)
- Yjs
- Custom CRDT engine

---

🏗 Full Android Blueprint

AppScope
   └── WebSocket (Operation Stream)
          └── Operation Store (Room)
                 └── Flow observeOperations()
                        └── Reducer
                               └── stateIn
                                      └── UI

---

🎯 When to Use CRDT?

✔ Real-time text collaboration
✔ Shared whiteboard
✔ Multiplayer document editing
✔ Distributed system without central authority
✔ Offline-first multi-device editing

---

🚫 When NOT to Use

❌ Simple CRUD app
❌ Backend-controlled systems
❌ No multi-user collaboration

CRDT increases complexity significantly.

---

🧠 Mental Model

Traditional merge = Judge deciding winner ⚖️
CRDT merge = Mathematics deciding result ➗

No arguments. Just math.

---

🏁 Final Summary

CRDT:

- Conflict-free by design
- Offline-friendly
- Peer-to-peer safe
- Perfect for collaborative apps
- Works beautifully with Flow

Modern 2026 Stack:

Room (Operation Log)
Flow (Reactive Stream)
Reducer (CRDT Engine)
stateIn (UI State)
WebSocket (Sync)

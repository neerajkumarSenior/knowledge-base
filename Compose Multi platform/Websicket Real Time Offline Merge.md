🔌🌐 WebSocket + Offline Merge Architecture

(Real-Time + Offline-First Enterprise Design)

यह guide explain करती है कि कैसे आप:

- ✅ WebSocket से real-time updates लें
- ✅ Offline-First architecture maintain करें
- ✅ Conflict-safe merge strategy implement करें
- ✅ Scalable enterprise-grade sync engine design करें

यह pattern chat apps, trading apps, ride apps, fintech dashboards में use होता है।

---

🧠 Core Problem

You want:

- Live real-time updates (WebSocket)
- App offline भी चले
- Data conflict न हो
- UI instantly update हो

Simple API polling enough नहीं है।

---

🏗 High-Level Architecture

             ┌───────────────┐
             │   WebSocket   │
             └───────┬───────┘
                     │
             ┌───────▼────────┐
             │ Remote Stream   │
             └───────┬────────┘
                     │
         ┌───────────▼───────────┐
         │  Merge Engine (Repo)  │
         └───────────┬───────────┘
                     │
             ┌───────▼────────┐
             │ Local Database  │  ← Single Source of Truth
             └───────┬────────┘
                     │
                 Flow (Room)
                     │
                ViewModel
                     │
                    UI

---

🔥 Golden Rules

1. Database = Single Source of Truth
2. WebSocket never directly updates UI
3. All remote events → Merge → DB
4. UI observes DB only

---

📦 Components Breakdown

1️⃣ WebSocket Data Source

Example:

class PostSocketDataSource(
    private val client: WebSocketClient
) {
    fun observeEvents(): Flow<PostEventDto> = callbackFlow {
        client.connect()

        client.onMessage { message ->
            trySend(parse(message))
        }

        awaitClose { client.disconnect() }
    }
}

Cold flow emitting real-time server events.

---

2️⃣ Local Database (SSOT)

@Query("SELECT * FROM posts")
fun observePosts(): Flow<List<PostEntity>>

UI always observes this.

---

3️⃣ Merge Engine (Repository)

This is the brain.

class PostRepository(
    private val dao: PostDao,
    private val socket: PostSocketDataSource
)

---

🔄 Real-Time Merge Flow

fun startRealtimeSync() {
    socket.observeEvents()
        .onEach { event ->
            mergeEvent(event)
        }
        .launchIn(appScope)
}

App-scope launch → survives screen changes.

---

🧠 Merge Strategies

Server sends:

{
  "id": "1",
  "title": "Updated Post",
  "version": 3,
  "updatedAt": 1710000000
}

Local entity:

data class PostEntity(
    val id: String,
    val title: String,
    val version: Int,
    val updatedAt: Long,
    val syncStatus: SyncStatus
)

---

⚔ Conflict Resolution Strategy

Strategy 1: Version-Based Merge (Recommended)

suspend fun mergeEvent(remote: PostDto) {
    val local = dao.getPost(remote.id)

    if (local == null || remote.version > local.version) {
        dao.insert(remote.toEntity())
    }
}

✔ Safe
✔ Deterministic
✔ Scalable

---

Strategy 2: Timestamp-Based

if (remote.updatedAt > local.updatedAt)

Less reliable (clock issues).

---

Strategy 3: Field-Level Merge (Advanced)

Used in collaborative apps:

- Only changed fields overwrite
- Merge nested objects

Enterprise-only.

---

📡 Handling Offline Writes

User edits offline:

suspend fun updatePost(post: Post) {
    dao.insert(post.toEntity(syncStatus = PENDING))
}

Background worker:

fun syncPending() {
    val pending = dao.getPendingPosts()
    api.sync(pending)
    dao.markSynced()
}

---

🔥 Optimistic UI + WebSocket Sync

Flow:

User Edit
  ↓
Update DB (Pending)
  ↓
UI instantly updates
  ↓
API request
  ↓
Server broadcasts via WebSocket
  ↓
Merge Engine updates DB
  ↓
Pending → Synced

---

🧩 Preventing Duplicate Events

Problem:

You send update
Server echoes back same update

Solution:

Add:

val originDeviceId: String

Ignore events from same device.

OR

Use version comparison.

---

🌊 Reactive Flow End-to-End

val uiState =
    dao.observePosts()
        .map { PostState.Success(it.map { e -> e.toDomain() }) }
        .stateIn(
            viewModelScope,
            SharingStarted.WhileSubscribed(5000),
            PostState.Loading
        )

---

🛡 WebSocket Lifecycle Handling

Use Application-level scope:

@Singleton
class RealtimeManager(
    private val repository: PostRepository,
    @ApplicationScope private val scope: CoroutineScope
)

Reconnect strategy:

.retryWhen { cause, attempt ->
    delay(2000)
    true
}

---

🔁 Reconnection + Recovery

When reconnecting:

1. Ask server for missed events
2. Send lastKnownVersion
3. Server sends diff only

Client → lastVersion=42
Server → send events > 42

Enterprise-grade approach.

---

🧠 Full Sync Engine Layers

WebSocket Stream
    ↓
Event Normalizer
    ↓
Merge Engine
    ↓
Local DB
    ↓
Reactive Flow
    ↓
UI

---

⚡ Performance Optimization

✔ Use "conflate()" on socket stream
✔ Batch DB writes
✔ Use transactions
✔ Avoid heavy work in onEach
✔ Avoid Main dispatcher

---

🚨 Common Mistakes

❌ Updating UI directly from socket
❌ No versioning
❌ No reconnect logic
❌ Clearing DB on reconnect
❌ No conflict strategy
❌ Not handling duplicate echoes

---

🏦 Real Enterprise Example

Trading App

- Stock prices via WebSocket
- Offline portfolio edits
- Version-based merge
- Reconnect diff fetch
- Optimistic local updates

---

🧠 Mental Model

WebSocket = Live News Feed
Database = Archive
Merge Engine = Editor
UI = Reader

Editor decides what goes into archive.

---

🏁 Final Blueprint (2026 Standard)

AppScope
   └── WebSocket Flow (Hot)
          └── Merge Engine
                 └── Room DB (SSOT)
                        └── Flow
                               └── stateIn (ViewModel)
                                      └── UI

---

🎯 When to Use This Architecture?

- Chat apps
- Social feed
- Fintech dashboard
- Crypto trading
- Multiplayer games
- Ride tracking
- Enterprise collaboration tools


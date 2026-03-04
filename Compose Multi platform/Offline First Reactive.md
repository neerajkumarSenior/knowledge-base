🌐 Offline-First Reactive Architecture (Android + Kotlin Flow)

यह guide बताती है कि Offline-First + Reactive (Flow based) architecture production apps में कैसे design किया जाता है —
Scalability, consistency, conflict handling और enterprise patterns के साथ।

---

🧠 Offline-First क्या है?

«App पहले local database से data दिखाएगा
Network sync background में होगा
User को हमेशा fast UI मिलेगा»

Golden Rule:

Single Source of Truth = Local Database

Network = Sync mechanism
Database = Truth

---

🏗 High-Level Architecture

UI (Compose)
     ↓
ViewModel (StateFlow)
     ↓
UseCase
     ↓
Repository
     ↓
Local DB (Room)
     ↕
Remote API

---

🔥 Core Principles

1️⃣ Single Source of Truth (SSOT)

UI कभी सीधे network से data नहीं लेती।

Wrong ❌

UI → API

Correct ✅

UI → DB → Repository syncs API

---

2️⃣ Reactive Database

Room Flow support देता है:

@Query("SELECT * FROM posts")
fun observePosts(): Flow<List<PostEntity>>

जब DB update होगी → UI auto update।

---

🧩 Data Layer Design

Local Entity

@Entity
data class PostEntity(
    @PrimaryKey val id: String,
    val title: String,
    val body: String,
    val updatedAt: Long
)

---

Remote DTO

data class PostDto(
    val id: String,
    val title: String,
    val body: String
)

---

Mapper

fun PostDto.toEntity(): PostEntity
fun PostEntity.toDomain(): Post

---

🏗 Repository Pattern (Offline-First)

class PostRepository(
    private val dao: PostDao,
    private val api: PostApi
)

---

Observe Data (Always from DB)

fun observePosts(): Flow<List<Post>> =
    dao.observePosts()
        .map { entities ->
            entities.map { it.toDomain() }
        }

---

Sync Function

suspend fun syncPosts() {
    val remote = api.getPosts()
    dao.insertAll(remote.map { it.toEntity() })
}

---

🔄 Reactive Sync Strategy

App Launch

viewModelScope.launch {
    repository.syncPosts()
}

UI already observing DB.

No manual refresh required.

---

🌊 Full Reactive Flow

val uiState =
    repository.observePosts()
        .map { PostState.Success(it) }
        .stateIn(
            viewModelScope,
            SharingStarted.WhileSubscribed(5000),
            PostState.Loading
        )

---

🔥 Network Bound Resource (Modern Pattern)

fun observePosts(): Flow<List<Post>> = flow {
    emitAll(
        dao.observePosts()
            .onStart {
                syncPosts()
            }
    )
}

But modern recommendation:

Separate observe & sync.

---

🌍 Sync Strategies

1️⃣ Pull-based

- User refresh
- Periodic sync

2️⃣ Push-based

- WebSocket
- Firebase
- Server events

3️⃣ WorkManager Sync

Background:

PeriodicWorker → syncPosts()

---

🛡 Conflict Handling

Real world:

User edits offline
Server updated same item

Need:

- Version field
- updatedAt timestamp
- Merge strategy

---

Conflict Resolution Strategy

1. Last Write Wins
2. Server Wins
3. Client Wins
4. Field-level merge

Enterprise apps use versioning:

val version: Int

---

🔄 Write Flow (Offline First)

User creates post

suspend fun createPost(post: Post) {
    dao.insert(post.toEntity(pendingSync = true))
}

Background sync:

val pending = dao.getPendingPosts()
api.createPosts(pending)
dao.markSynced()

---

📡 Sync Queue Pattern

Add column:

syncStatus = PENDING | SYNCED | FAILED

Worker retries failed sync.

---

🔥 Advanced Pattern: Optimistic Update

User likes post:

1. Update DB immediately
2. Update UI instantly
3. Send API request
4. If fail → rollback

---

🧠 Full Enterprise Flow

User Action
   ↓
Update Local DB
   ↓
UI Reacts Immediately
   ↓
Background Sync
   ↓
Server Response
   ↓
Update DB Again (if needed)

---

📦 stateIn in Offline First

ViewModel:

val uiState =
    repository.observePosts()
        .map { PostState.Success(it) }
        .stateIn(
            viewModelScope,
            SharingStarted.WhileSubscribed(5000),
            PostState.Loading
        )

Cold DB Flow → Hot StateFlow.

---

🚦 Error Handling Strategy

Network error?

- Don't crash UI
- Keep showing cached data
- Show Snackbar
- Retry option

---

📊 Caching Strategy Levels

Strategy| Use Case
Cache-Aside| Basic apps
Network Bound Resource| Standard
Full Sync Engine| Enterprise
CRDT / Conflict-free| Collaboration apps

---

🏗 Folder Structure Example

data/
   local/
   remote/
   repository/

domain/
   model/
   usecase/

ui/
   state/
   viewmodel/

---

🧪 Testing Strategy

Test:

- DB emits when updated
- Sync updates DB
- Conflict resolution logic
- Offline create → later sync

Use Fake API + InMemory DB.

---

🎯 When Offline-First is Mandatory

- Fintech apps
- Social media
- Ride booking
- Chat apps
- Enterprise dashboards
- Rural internet markets

---

🚨 Common Mistakes

❌ UI reading directly from API
❌ Clearing DB on every sync
❌ Not handling versioning
❌ No retry mechanism
❌ Blocking UI during sync
❌ No background worker

---

🧠 Mental Model

Database = Brain
API = Cloud Backup
UI = Eyes

Brain always active.
Cloud sync happens quietly.

---

🏁 Final Summary

Offline-First Reactive Pattern:

DB = Single Source of Truth
Flow = Reactive updates
stateIn = UI state holder
Worker = Background sync
Conflict strategy = Mandatory

Modern Android Enterprise Stack (2026):

- Room
- Kotlin Flow
- stateIn
- WorkManager
- Repository Pattern
- Sync Queue

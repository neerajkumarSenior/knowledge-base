🗂 Repository Documentation (Repository.md)

यह डॉक्यूमेंट Clean Architecture में Repository की पूरी role, purpose, structure, strategy और best practices को detail में explain करता है।

---

🎯 Repository क्या है?

Repository एक abstraction layer है जो:

«📡 Remote Data (API)
💾 Local Data (Database)
🔄 Cache
⚙️ Data Strategy»

इन सबको manage करता है — और Domain layer को clean data देता है।

Repository Data layer का brain होता है।

---

🏗 Clean Architecture में Position

UI
 ↓
ViewModel
 ↓
UseCase
 ↓
PostRepository (Interface - Domain)
 ↓
PostRepositoryImpl (Data Layer)
 ↓
RemoteDataSource + LocalDataSource
 ↓
API / Database

---

🧠 Repository क्यों जरूरी है?

1️⃣ Abstraction (Dependency Inversion Principle)

Domain layer केवल interface जानता है:

interface PostRepository {
    suspend fun getPosts(): List<Post>
}

Implementation Data layer में होती है।

👉 Domain layer को पता ही नहीं API या DB कैसे काम कर रहा है।

---

2️⃣ Strategy Control

Repository decide करता है:

- Offline first?
- Cache first?
- Network only?
- Merge data?
- Pagination?

UseCase सिर्फ "काम" बताता है
Repository "कैसे करना है" decide करता है

---

📂 Recommended Structure

domain/
 └── repository/
      └── PostRepository.kt

data/
 └── repository/
      └── PostRepositoryImpl.kt

---

📦 Domain Repository Interface

interface PostRepository {

    suspend fun getPosts(): List<Post>

    suspend fun createPost(post: Post): Post
}

👉 Interface में DTO नहीं होना चाहिए
👉 Domain model return करना चाहिए

---

📦 Basic Repository Implementation

class PostRepositoryImpl(
    private val remote: PostRemoteDataSource,
    private val mapper: PostDtoMapper
) : PostRepository {

    override suspend fun getPosts(): List<Post> {
        val dtos = remote.getPosts()
        return dtos.map { mapper.toDomain(it) }
    }

    override suspend fun createPost(post: Post): Post {
        val dto = mapper.toDto(post)
        val response = remote.createPost(dto)
        return mapper.toDomain(response)
    }
}

---

🔥 Advanced Repository (Offline First)

override fun getPosts(): Flow<Result<List<Post>>> = flow {

    emit(Result.Loading)

    // 1️⃣ Load cached data
    val cached = local.getPosts()
    emit(Result.Success(cached.map { entityMapper.toDomain(it) }))

    // 2️⃣ Fetch from network
    val remoteData = remote.getPosts()

    // 3️⃣ Save to DB
    local.insertPosts(remoteData.map { dtoMapper.toEntity(it) })

    // 4️⃣ Emit fresh data
    emit(Result.Success(remoteData.map { dtoMapper.toDomain(it) }))
}

---

🧠 Repository Responsibilities

Responsibility| Example
Data source choose करना| API या DB
DTO → Domain mapping| Mapper
Caching| Save to Room
Error handling| Wrap in Result
Retry logic| Try again
Pagination| Paging3

---

❌ Repository क्या नहीं करना चाहिए?

❌ UI logic
❌ ViewModel reference
❌ Compose state
❌ Direct JSON parsing
❌ Android Context dependency

---

🧩 Multiple Data Sources Example

class PostRepositoryImpl(
    private val remote: PostRemoteDataSource,
    private val local: PostLocalDataSource,
    private val dtoMapper: PostDtoMapper,
    private val entityMapper: PostEntityMapper
)

अब Repository smart बन गया:

- Internet हो तो sync करे
- Offline हो तो cache दे
- Merge करे
- Conflict resolve करे

---

🔁 Complete Production Flow

ViewModel
 ↓
UseCase
 ↓
Repository
   ├── RemoteDataSource
   └── LocalDataSource
 ↓
Mapper
 ↓
Domain Model
 ↓
UI

---

🧪 Testing Repository

Fake RemoteDataSource inject करो:

class FakeRemoteDataSource : PostRemoteDataSource {
    override suspend fun getPosts() = fakeList
}

अब Repository testable है बिना network के।

---

🚀 Pagination Repository Example

override fun getPagedPosts(): Flow<PagingData<Post>> {
    return Pager(
        config = PagingConfig(pageSize = 20),
        pagingSourceFactory = { postPagingSource }
    ).flow
}

---

🏆 Repository Types (Strategy Based)

Type| Use Case
Network Only| Simple app
Cache First| Medium app
Offline First| Production
Network Bound Resource| Enterprise

---

🔥 Network Bound Resource Pattern

Load from DB
If empty → Fetch from network
Save to DB
Emit DB

Production apps में common pattern है।

---

🧠 Repository Golden Rules

✅ Interface Domain layer में
✅ Implementation Data layer में
✅ Domain model return करें
✅ Mapping Data layer में करें
✅ Single Responsibility follow करें

---

⚠️ Common Mistakes

❌ Repository में 2000 lines code
❌ Direct API injection in ViewModel
❌ DTO return करना
❌ Business logic usecase की जगह repository में डाल देना

---

🎯 Why Repository is Critical

Without Repository:

- Tight coupling
- Hard refactor
- No scalability
- No offline support
- No testability

With Repository:

- Clean separation
- Flexible data strategy
- Easy replace API
- Easy add DB
- Enterprise ready

---

🏁 Final Summary

Repository:

- Data layer का brain है
- Data strategy control करता है
- Domain को clean data देता है
- Multiple sources manage करता है
- Production scalable apps का core है

---

📌 Related Docs:

- Architecture.md
- DTO.md
- Mapper.md
- RemoteDataSource.md
- OfflineFirstStrategy.md


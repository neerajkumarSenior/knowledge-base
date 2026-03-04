🏗 PostRepositoryImpl Documentation (RepositoryImpl.md)

यह डॉक्यूमेंट Clean Architecture में RepositoryImpl की पूरी role, design principles, structure, responsibilities और production-level implementation को detail में explain करता है।

---

🎯 RepositoryImpl क्या है?

"RepositoryImpl":

«Domain layer में defined Repository interface का
Data layer में actual implementation है।»

यह class decide करती है:

- डेटा कहाँ से लाना है (API / DB)
- DTO को Domain model में convert करना
- Cache करना या नहीं
- Error wrap करना
- Strategy apply करना

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

📂 Structure

domain/
 └── repository/
      └── PostRepository.kt

data/
 └── repository/
      └── PostRepositoryImpl.kt

---

📦 Step 1: Domain Interface

interface PostRepository {

    suspend fun getPosts(): List<Post>

    suspend fun createPost(post: Post): Post
}

⚠️ Important:

- DTO नहीं
- Entity नहीं
- Domain model return करना है

---

📦 Step 2: Basic RepositoryImpl

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

🧠 Responsibilities Breakdown

Responsibility| कहाँ होना चाहिए
API call| RemoteDataSource
DB call| LocalDataSource
Mapping| Mapper
Strategy| RepositoryImpl
Business rule| UseCase

---

🔥 Advanced RepositoryImpl (Offline First)

class PostRepositoryImpl(
    private val remote: PostRemoteDataSource,
    private val local: PostLocalDataSource,
    private val dtoMapper: PostDtoMapper,
    private val entityMapper: PostEntityMapper
) : PostRepository {

    override fun getPosts(): Flow<Result<List<Post>>> = flow {

        emit(Result.Loading)

        // 1️⃣ Emit cached data
        val cached = local.getPosts()
        emit(Result.Success(cached.map { entityMapper.toDomain(it) }))

        // 2️⃣ Fetch fresh data
        val remoteData = remote.getPosts()

        // 3️⃣ Save to DB
        local.insertPosts(
            remoteData.map { dtoMapper.toEntity(it) }
        )

        // 4️⃣ Emit updated data
        emit(Result.Success(
            remoteData.map { dtoMapper.toDomain(it) }
        ))
    }
}

---

🔁 Repository Strategy Patterns

1️⃣ Network Only

override suspend fun getPosts(): List<Post> {
    return remote.getPosts().map { mapper.toDomain(it) }
}

---

2️⃣ Cache First

if (cache not empty) return cache
else fetch from network

---

3️⃣ Offline First (Recommended Production Pattern)

Load from DB
Emit
Fetch from Network
Save to DB
Emit updated DB

---

🧩 Dependency Injection (Koin Example)

single<PostRepository> {
    PostRepositoryImpl(
        remote = get(),
        local = get(),
        dtoMapper = get(),
        entityMapper = get()
    )
}

---

🛡 Error Handling Version

override suspend fun getPosts(): Result<List<Post>> {
    return try {
        val dtos = remote.getPosts()
        Result.Success(dtos.map { mapper.toDomain(it) })
    } catch (e: Exception) {
        Result.Error(e.message ?: "Unknown Error")
    }
}

---

🧪 Testing RepositoryImpl

Fake dependencies inject करो:

val fakeRemote = FakePostRemoteDataSource()
val fakeLocal = FakePostLocalDataSource()

val repository = PostRepositoryImpl(
    fakeRemote,
    fakeLocal,
    dtoMapper,
    entityMapper
)

अब बिना network के test कर सकते हो।

---

❌ Common Mistakes

❌ DTO return करना
❌ UI state manage करना
❌ ViewModel reference रखना
❌ Mapping ViewModel में करना
❌ Repository 1000+ lines का बना देना

---

🧠 Golden Rules

✅ Interface domain में
✅ Implementation data में
✅ Domain model return करें
✅ Mapping data layer में करें
✅ Strategy RepositoryImpl में रखें
✅ Single Responsibility follow करें

---

🚀 Enterprise Level Enhancements

RepositoryImpl में add किया जा सकता है:

- Retry mechanism
- Rate limiting
- Background sync
- WebSocket merge
- Conflict resolution
- API version switching
- Feature flag based source switching

---

🏆 Why RepositoryImpl is Powerful?

Without RepositoryImpl:

- ViewModel direct API call करेगा ❌
- Tight coupling ❌
- Hard to scale ❌

With RepositoryImpl:

- Flexible data strategy ✅
- Easy API replace ✅
- Offline support ready ✅
- Testable architecture ✅
- Enterprise ready structure ✅

---

📌 Final Summary

"RepositoryImpl":

- Data layer का real brain है
- Strategy decide करता है
- Multiple data sources manage करता है
- Domain को clean data देता है
- Production scalable architecture का core हिस्सा है

---

📌 Related Docs:

- Architecture.md
- DTO.md
- Mapper.md
- RemoteDataSource.md
- Repository.md
- OfflineFirstStrategy.md

📡 RemoteDataSource Documentation (RemoteDataSource.md)

यह डॉक्यूमेंट Clean Architecture में RemoteDataSource की भूमिका, उद्देश्य, structure और best practices को explain करता है।

---

🎯 RemoteDataSource क्या है?

"RemoteDataSource" Data layer का वह हिस्सा है जो:

- 🌐 API call करता है
- 📦 Server से raw data (DTO) लाता है
- ⚠️ Network exceptions handle करता है
- 🚫 Business logic नहीं रखता

यह Repository और API के बीच एक abstraction layer है।

---

🏗 Clean Architecture में Position

UI
 ↓
ViewModel
 ↓
UseCase
 ↓
Repository
 ↓
RemoteDataSource
 ↓
API (Ktor / Retrofit)
 ↓
Server

---

🧠 RemoteDataSource क्यों जरूरी है?

1️⃣ Separation of Concerns

Layer| Responsibility
Repository| Data strategy (offline-first, merge, cache)
RemoteDataSource| Network execution
API| Endpoint definition

Repository का काम network call करना नहीं है।
उसका काम strategy decide करना है।

---

2️⃣ Scalability

अगर future में:

- Ktor → Retrofit
- REST → GraphQL
- Add WebSocket
- Add Firebase

तो सिर्फ RemoteDataSource बदलेगा, Repository नहीं।

---

3️⃣ Testing Friendly

Repository test करते समय:

- FakeRemoteDataSource inject कर सकते हैं
- Network dependency remove हो जाती है

Example:

class FakePostRemoteDataSource : PostRemoteDataSource {
    override suspend fun getPosts(): List<PostDto> {
        return listOf(PostDto(1, "Test", "Body"))
    }
}

---

📂 Recommended Folder Structure

data/
 ├── remote/
 │    ├── PostApi.kt
 │    └── PostRemoteDataSource.kt

---

📦 Basic Implementation Example

class PostRemoteDataSource(
    private val api: PostApi
) {

    suspend fun getPosts(): List<PostDto> {
        return api.getPosts()
    }

    suspend fun createPost(dto: PostDto): PostDto {
        return api.createPost(dto)
    }
}

---

🛡 Advanced Version (Error Handling Included)

class PostRemoteDataSource(
    private val api: PostApi
) {

    suspend fun getPosts(): List<PostDto> {
        return try {
            api.getPosts()
        } catch (e: Exception) {
            throw mapToDomainException(e)
        }
    }

    private fun mapToDomainException(e: Exception): Exception {
        return when (e) {
            is IOException -> Exception("No Internet Connection")
            is HttpException -> Exception("Server Error")
            else -> Exception("Unknown Error")
        }
    }
}

---

📌 Important Rules

✅ RemoteDataSource में business logic मत डालो
✅ Domain model return मत करो (DTO return करो)
✅ UI state manage मत करो
✅ Coroutine scope मत बनाओ
✅ ViewModel reference मत रखो

---

🔁 Repository + RemoteDataSource Example

class PostRepositoryImpl(
    private val remote: PostRemoteDataSource,
    private val mapper: PostDtoMapper
) : PostRepository {

    override suspend fun getPosts(): List<Post> {
        val dtos = remote.getPosts()
        return dtos.map { mapper.toDomain(it) }
    }
}

---

🧩 Multiple Remote Sources Scenario

Future में:

PostRestRemoteDataSource
PostFirebaseRemoteDataSource
PostWebSocketRemoteDataSource

Repository decide करेगा किसे use करना है।

---

🚀 Production Ready Features

RemoteDataSource में add किया जा सकता है:

- Token injection
- Header management
- Logging
- Retry mechanism
- Timeout handling
- API versioning

---

⚠️ Common Mistakes

❌ Repository में direct API call
❌ RemoteDataSource में mapper डाल देना
❌ RemoteDataSource से Domain model return करना
❌ Network + Cache logic mix करना

---

🏆 Best Practice Summary

Good Practice| Reason
DTO return करें| Domain clean रहे
Exception map करें| Consistent error handling
Small functions रखें| Testable code
Single Responsibility| Maintainable code

---

🎯 Final Conclusion

RemoteDataSource:

- Data layer को clean रखता है
- Repository को lightweight बनाता है
- Testing आसान बनाता है
- Future changes safe बनाता है
- Enterprise-level scalability देता है

अगर आपका app:

- Production level है
- Offline support चाहिए
- Large scale feature है
- Multiple API integrations हैं

👉 RemoteDataSource use करें।

---

📌 Related Docs:

- Architecture.md
- Repository.md
- ResultWrapper.md
- OfflineFirstStrategy.md

🔄 Mapper Documentation (Mapper.md)

यह डॉक्यूमेंट Clean Architecture में Mapper की पूरी role, purpose, design rules और best practices को detail में explain करता है।

---

🎯 Mapper क्या है?

Mapper एक class होती है जो:

«📦 Data Layer के model (DTO / Entity) को
🧼 Domain Layer के model में convert करती है»

और कभी-कभी reverse conversion भी करती है।

---

🏗 Clean Architecture में Position

Server JSON
   ↓
PostDto (Data Model)
   ↓
PostDtoMapper
   ↓
Post (Domain Model)
   ↓
UI

---

🧠 Mapper क्यों जरूरी है?

1️⃣ Server Structure Unstable होता है

Server response बदल सकता है:

{
  "post_id": 1,
  "post_title": "Hello",
  "post_body": "World",
  "created_at": "2025-03-01"
}

लेकिन UI को simple model चाहिए:

data class Post(
    val id: Int,
    val title: String,
    val body: String
)

👉 Mapper इस gap को bridge करता है।

---

2️⃣ Domain Layer को Clean रखना

Domain layer में:

- ❌ DTO नहीं होना चाहिए
- ❌ JSON annotations नहीं होने चाहिए
- ❌ Retrofit/Ktor annotations नहीं होने चाहिए

Mapper separation maintain करता है।

---

3️⃣ Future Proofing

अगर API structure बदल जाए
तो सिर्फ Mapper बदलना होगा, UI नहीं।

---

📂 Recommended Structure

data/
 ├── model/
 │    └── PostDto.kt
 │
 ├── mapper/
 │    ├── PostDtoMapper.kt
 │    └── PostEntityMapper.kt

---

📦 Basic Mapper Example

DTO

data class PostDto(
    val id: Int,
    val title: String,
    val body: String
)

Domain Model

data class Post(
    val id: Int,
    val title: String,
    val body: String
)

Mapper

class PostDtoMapper {

    fun toDomain(dto: PostDto): Post {
        return Post(
            id = dto.id,
            title = dto.title,
            body = dto.body
        )
    }

    fun toDto(domain: Post): PostDto {
        return PostDto(
            id = domain.id,
            title = domain.title,
            body = domain.body
        )
    }
}

---

🛡 Advanced Mapping Example

मान लो server response messy है:

data class PostDto(
    val post_id: Int?,
    val post_title: String?,
    val post_body: String?,
    val is_active: Int?
)

Domain clean चाहिए:

data class Post(
    val id: Int,
    val title: String,
    val body: String,
    val isActive: Boolean
)

Safe Mapper:

class PostDtoMapper {

    fun toDomain(dto: PostDto): Post {
        return Post(
            id = dto.post_id ?: 0,
            title = dto.post_title.orEmpty(),
            body = dto.post_body.orEmpty(),
            isActive = dto.is_active == 1
        )
    }
}

---

🔁 Entity Mapper (Offline Support)

Room Entity:

data class PostEntity(
    val id: Int,
    val title: String,
    val body: String
)

Entity Mapper:

class PostEntityMapper {

    fun toDomain(entity: PostEntity): Post {
        return Post(
            id = entity.id,
            title = entity.title,
            body = entity.body
        )
    }

    fun toEntity(domain: Post): PostEntity {
        return PostEntity(
            id = domain.id,
            title = domain.title,
            body = domain.body
        )
    }
}

---

🔥 Mapper Best Practices

Rule| Reason
DTO → Domain conversion जरूरी| Clean boundaries
Null safety handle करो| Crash avoid
Default values set करो| Stability
Business logic मत डालो| SRP follow
Simple functions रखो| Testable code

---

❌ Common Mistakes

❌ Repository में mapping कर देना
❌ DTO को UI में pass कर देना
❌ Mapper में API call डाल देना
❌ ViewModel में conversion करना
❌ Domain model में JSON annotation डालना

---

🧪 Testing Mapper

Mapper easily testable है:

@Test
fun `dto maps correctly to domain`() {
    val dto = PostDto(1, "Hello", "World")
    val mapper = PostDtoMapper()

    val domain = mapper.toDomain(dto)

    assertEquals(1, domain.id)
    assertEquals("Hello", domain.title)
}

---

🏗 Full Data Flow with Mapper

API
 ↓
PostDto
 ↓
PostDtoMapper
 ↓
Post (Domain)
 ↓
UseCase
 ↓
ViewModel
 ↓
UI

---

🧠 When Multiple Mappers Needed?

Large scale apps में:

UserDtoMapper
RideDtoMapper
PaymentDtoMapper
LocationMapper
NotificationMapper

हर feature का अलग mapper होना चाहिए।

---

🚀 Advanced Pattern (Generic Mapper Interface)

Reusable pattern:

interface Mapper<I, O> {
    fun map(input: I): O
}

Use:

class PostDtoMapper : Mapper<PostDto, Post> {
    override fun map(input: PostDto): Post {
        return Post(
            id = input.id,
            title = input.title,
            body = input.body
        )
    }
}

---

🎯 Why Mapper is Critical in Production

Without Mapper:

- Tight coupling
- Hard to change API
- Dirty domain models
- Difficult testing
- Unstable UI

With Mapper:

- Loose coupling
- Clean boundaries
- Future safe
- Easy refactor
- Enterprise ready

---

🏆 Final Conclusion

Mapper:

- Data layer और Domain layer के बीच firewall है
- Structure change absorb करता है
- App को stable बनाता है
- Clean Architecture maintain करता है
- Large scale systems में mandatory है

---

📌 Related Docs:

- Architecture.md
- RemoteDataSource.md
- Repository.md
- OfflineFirstStrategy.md

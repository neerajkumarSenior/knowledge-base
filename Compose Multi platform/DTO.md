📦 DTO Documentation (DTO.md)

यह डॉक्यूमेंट Clean Architecture में DTO (Data Transfer Object) की पूरी concept, purpose, structure और best practices को detail में explain करता है।

---

🎯 DTO क्या है?

DTO = Data Transfer Object

DTO एक simple data class होती है जो:

«🌐 Server से आने वाला raw data represent करती है
📤 Server को भेजे जाने वाला request body represent करती है»

DTO सिर्फ data transport के लिए होती है।
इसमें business logic नहीं होता।

---

🏗 Clean Architecture में Position

Server (JSON)
   ↓
PostDto (DTO)
   ↓
Mapper
   ↓
Post (Domain Model)
   ↓
UI

---

🧠 DTO क्यों जरूरी है?

1️⃣ Server Structure अलग होता है

Server response:

{
  "post_id": 10,
  "post_title": "Hello",
  "post_body": "World",
  "created_at": "2025-03-04T10:20:00Z"
}

लेकिन UI को इतना data नहीं चाहिए।

DTO server जैसा का तैसा structure रखता है।

---

2️⃣ Domain Layer को Clean रखना

Domain model में:

- ❌ JSON annotations नहीं
- ❌ Serialization library dependency नहीं
- ❌ API specific naming नहीं

DTO server-specific रहता है
Domain model clean रहता है

---

📂 Recommended Folder Structure

data/
 ├── model/
 │    ├── PostDto.kt
 │    ├── CreatePostRequestDto.kt
 │    └── BaseResponseDto.kt

---

📦 Basic DTO Example

data class PostDto(
    val id: Int,
    val title: String,
    val body: String
)

---

🛡 Real World DTO Example (Messy Server)

@Serializable
data class PostDto(
    val post_id: Int?,
    val post_title: String?,
    val post_body: String?,
    val created_at: String?,
    val is_active: Int?
)

👉 Notice:

- Nullable fields
- Server naming convention
- Extra fields

ये सब Domain model में नहीं जाने चाहिए।

---

🔁 Request DTO Example

जब नया post create करना हो:

data class CreatePostRequestDto(
    val post_title: String,
    val post_body: String
)

---

📥 Response Wrapper DTO

कई APIs ऐसा response देती हैं:

{
  "status": true,
  "message": "Success",
  "data": {
      "id": 1,
      "title": "Hello"
  }
}

DTO:

data class BaseResponseDto<T>(
    val status: Boolean,
    val message: String?,
    val data: T?
)

---

🔥 DTO vs Domain Model

DTO| Domain Model
Server dependent| App dependent
Nullable fields| Clean fields
JSON annotations| No annotations
Raw structure| Refined structure
Data layer only| Domain layer

---

🧠 DTO Golden Rules

✅ DTO सिर्फ Data layer में रहे
✅ Mapper से convert करो
✅ Nullable handle करो
✅ Business logic मत डालो
✅ Validation मत डालो

---

❌ Common Mistakes

❌ DTO को UI में pass कर देना
❌ DTO को Domain में use करना
❌ DTO में business logic डालना
❌ Domain model में @Serializable डाल देना
❌ ViewModel में JSON parsing करना

---

🧪 DTO Flow Example

API
 ↓
JSON
 ↓
PostDto
 ↓
Mapper
 ↓
Post (Domain)
 ↓
UseCase
 ↓
ViewModel
 ↓
UI

---

🚀 Advanced Production Tips

1️⃣ Separate Request & Response DTO

CreatePostRequestDto
CreatePostResponseDto

Mix मत करो।

---

2️⃣ API Versioning Ready रखो

v1/PostDto
v2/PostDto

Future changes safe रहेंगे।

---

3️⃣ Default Values Avoid करो

DTO में default value मत दो अगर server unreliable है।

Better:

val title: String?

और mapper में handle करो।

---

🏆 Why DTO is Critical

Without DTO:

- Tight coupling
- API change से app crash
- Domain dirty
- Hard refactor

With DTO:

- Clean separation
- API flexible
- Safe mapping
- Production ready

---

📌 Final Conclusion

DTO:

- Server data representation है
- Data layer का हिस्सा है
- Domain से अलग रहना चाहिए
- Always Mapper के साथ use होना चाहिए

DTO बिना mapper = Half Clean Architecture

---

📌 Related Docs:

- Architecture.md
- Mapper.md
- RemoteDataSource.md
- Repository.md

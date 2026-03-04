📦 Post Feature – Clean Architecture Documentation (Architecture.md)

यह डॉक्यूमेंट "features/post/" मॉड्यूल की पूरी आर्किटेक्चर को समझाता है।
यह Structure Clean Architecture + MVVM + Koin DI पर आधारित है।

---

🏗 Overall Architecture Pattern

हमने 3 लेयर फॉलो की हैं:

UI  →  Domain  →  Data

🔹 Rule:

- UI को Data layer का पता नहीं होना चाहिए
- Domain layer pure Kotlin logic होती है (no framework dependency)
- Data layer server / database से deal करती है
- Dependency Inversion Principle follow किया गया है

---

📂 Folder Structure Explanation

features/post/

---

1️⃣ 📂 DATA Layer (Implementation Layer)

👉 यह layer बाहरी दुनिया (API / DB) से बात करती है।

---

📂 data/remote/PostApi.kt

🔹 काम:

- Ktor API endpoints define करना
- Server से data fetch / send करना

🔹 Example Responsibility:

- getPosts()
- createPost()

यह सिर्फ network communication संभालता है।

---

📂 data/model/PostDto.kt

🔹 DTO (Data Transfer Object)

यह server से आने वाला raw JSON data represent करता है।

Example:

{
  "id": 1,
  "title": "Hello",
  "body": "World"
}

DTO हमेशा API structure जैसा होता है।

⚠️ DTO को UI में कभी directly use नहीं करना चाहिए।

---

📂 data/mapper/PostMapper.kt

🔹 काम:

DTO ➝ Domain Model में convert करना

Example:

PostDto → Post

🔹 क्यों जरूरी है?

- Server structure बदल सकता है
- UI को stable structure चाहिए
- Domain layer clean रहनी चाहिए

---

📂 data/repository/PostRepositoryImpl.kt

🔹 काम:

Domain में defined "PostRepository" interface का implementation

यह:

- API call करता है
- DTO को mapper से convert करता है
- Domain model return करता है

Flow:

UseCase → RepositoryImpl → API → DTO → Mapper → Domain Model

---

2️⃣ 📂 DOMAIN Layer (Business Logic Layer)

👉 यह application का दिमाग है
👉 Pure Kotlin (no Android dependency)

---

📂 domain/model/Post.kt

🔹 Clean Model

यह UI के लिए साफ-सुथरा model है।

- इसमें extra server fields नहीं होते
- Simple data class

---

📂 domain/repository/PostRepository.kt

🔹 Interface (Rule Book)

यह बताता है:

क्या काम होना चाहिए

Example:

- suspend fun getPosts(): List<Post>
- suspend fun createPost(post: Post)

यह implementation नहीं देता।

---

📂 domain/usecase/

🎯 UseCase = Single Responsibility Class

हर use case एक काम करता है।

---

🔹 GetPostsUseCase.kt

काम:

- Repository से posts लाना
- Business logic apply करना (अगर हो)

---

🔹 CreatePostUseCase.kt

काम:

- नया post create करना
- Validation logic add करना (अगर हो)

---

3️⃣ 📂 UI Layer (Presentation Layer)

👉 User से interact करने वाली layer
👉 Jetpack Compose + ViewModel

---

📂 ui/components/PostItem.kt

🔹 Reusable UI Component

छोटे composables:

- Card
- Title
- Body

Reusable UI blocks

---

📂 ui/PostScreen.kt

🔹 Main Screen

- ViewModel observe करता है
- State के आधार पर UI render करता है

Example states:

- Loading
- Error
- Success

---

📂 ui/PostViewModel.kt

🔹 Bridge Between UI & Domain

काम:

- UseCase call करना
- State manage करना
- Coroutine scope handle करना

Flow:

UI → ViewModel → UseCase → Repository

---

📂 ui/PostState.kt

🔹 Screen State Holder

Sealed class या data class हो सकता है:

Example:

- Loading
- Success(List<Post>)
- Error(String)

---

4️⃣ 📂 DI Layer (Dependency Injection)

📂 di/PostModule.kt

हमने Koin use किया है:

- ViewModel provide करना
- UseCase provide करना
- Repository bind करना
- API inject करना

Dependency Graph:

PostScreen
   ↓
PostViewModel
   ↓
UseCases
   ↓
PostRepository (Interface)
   ↓
PostRepositoryImpl
   ↓
PostApi

---

🔁 Complete Data Flow Diagram

User Click
   ↓
PostScreen
   ↓
PostViewModel
   ↓
GetPostsUseCase
   ↓
PostRepository (Interface)
   ↓
PostRepositoryImpl
   ↓
PostApi (Ktor)
   ↓
Server
   ↓
PostDto
   ↓
PostMapper
   ↓
Post (Domain Model)
   ↓
PostState.Success
   ↓
UI Render

---

🎯 Benefits of This Architecture

✅ Testable
✅ Scalable
✅ Maintainable
✅ Clear Separation of Concerns
✅ Easy to Replace API / DB
✅ Easy to Add New Features

---

🧠 Why This Architecture Is Powerful?

- अगर API बदल जाए → सिर्फ Data layer बदलेगी
- अगर UI बदले → Domain unaffected
- अगर Database add करना हो → Repository modify होगा
- Large scale app के लिए perfect

---

📌 Best Practices

1. DTO को कभी UI में use मत करो
2. Domain layer में Android imports मत डालो
3. UseCase single responsibility follow करे
4. ViewModel में business logic मत भरो
5. Mapper जरूर बनाओ

---

🚀 Future Scalability

आप आगे ये add कर सकते हैं:

- Local database (Room)
- Cache layer
- Pagination
- Offline support
- Error handling wrapper (Result class)
- BaseViewModel abstraction

---

📚 Conclusion

यह "features/post/" module:

- Clean Architecture follow करता है
- MVVM pattern implement करता है
- Koin से loosely coupled dependencies use करता है
- Scalable production-ready structure है


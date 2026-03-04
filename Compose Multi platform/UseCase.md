🎯 UseCase Documentation (UseCase.md)

यह डॉक्यूमेंट Clean Architecture में UseCase (या Interactor) की पूरी भूमिका, design principles, structure और best practices को detail में explain करता है।

---

🧠 UseCase क्या है?

UseCase एक class होती है जो:

«🎯 Application का एक specific काम (Single Responsibility) perform करती है»

Example:

- GetPostsUseCase → पोस्ट लाना
- CreatePostUseCase → नया पोस्ट बनाना
- DeletePostUseCase → पोस्ट हटाना

UseCase = “एक काम, एक class”

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
Data Layer

---

🎯 UseCase क्यों जरूरी है?

1️⃣ Business Logic Separation

अगर तुम ViewModel में सब logic डाल दोगे:

❌ Code messy हो जाएगा
❌ Reusable नहीं रहेगा
❌ Test करना मुश्किल होगा

UseCase business logic को isolate करता है।

---

2️⃣ Single Responsibility Principle (SRP)

हर UseCase सिर्फ एक काम करता है।

GetPostsUseCase → सिर्फ posts लाना
CreatePostUseCase → सिर्फ create करना

---

3️⃣ Testable Architecture

UseCase को easily unit test कर सकते हो:

- Fake Repository inject करो
- Result verify करो

---

📂 Recommended Structure

domain/
 └── usecase/
      ├── GetPostsUseCase.kt
      ├── CreatePostUseCase.kt
      └── DeletePostUseCase.kt

---

📦 Basic UseCase Example

class GetPostsUseCase(
    private val repository: PostRepository
) {

    suspend operator fun invoke(): List<Post> {
        return repository.getPosts()
    }
}

👉 "operator fun invoke()" use करने से:

getPostsUseCase()

ऐसे call कर सकते हैं।

---

📦 Create Post UseCase

class CreatePostUseCase(
    private val repository: PostRepository
) {

    suspend operator fun invoke(title: String, body: String): Post {

        if (title.isBlank()) {
            throw IllegalArgumentException("Title cannot be empty")
        }

        val post = Post(
            id = 0,
            title = title,
            body = body
        )

        return repository.createPost(post)
    }
}

---

🧠 UseCase Responsibilities

Responsibility| Example
Business rule apply करना| Validation
Multiple repository call| Merge data
Data transform करना| Sort / Filter
Input sanitize करना| Trim text
Combine data sources| User + Post

---

❌ UseCase क्या नहीं करना चाहिए?

❌ UI state manage करना
❌ DTO handle करना
❌ Android Context use करना
❌ Compose dependency
❌ Coroutine scope create करना

---

🔥 Advanced Flow Based UseCase

class GetPostsUseCase(
    private val repository: PostRepository
) {

    operator fun invoke(): Flow<Result<List<Post>>> {
        return repository.getPosts()
    }
}

अब ViewModel reactive बन सकता है।

---

🧩 Multiple Repository Use Example

class GetUserWithPostsUseCase(
    private val userRepository: UserRepository,
    private val postRepository: PostRepository
) {

    suspend operator fun invoke(userId: Int): UserWithPosts {

        val user = userRepository.getUser(userId)
        val posts = postRepository.getPosts()

        return UserWithPosts(user, posts)
    }
}

---

🧪 Testing UseCase

@Test
fun `title should not be empty`() = runTest {

    val fakeRepository = FakePostRepository()
    val useCase = CreatePostUseCase(fakeRepository)

    assertThrows<IllegalArgumentException> {
        useCase("", "body")
    }
}

---

🏆 UseCase vs Repository

UseCase| Repository
Business logic| Data strategy
Input validation| API/DB decide
Combine data| Fetch data
Domain layer| Data layer

---

🔁 Complete Flow

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
API

---

🚀 Enterprise Pattern

Large scale apps में:

auth/
ride/
payment/
post/

हर feature में:

GetXUseCase
CreateXUseCase
UpdateXUseCase
DeleteXUseCase

Clean modular structure बनता है।

---

🧠 Golden Rules

✅ Single responsibility
✅ Small class
✅ Domain layer में रहे
✅ Operator invoke use करें
✅ Business logic यहाँ रखें
✅ Repository को directly UI में inject न करें

---

⚠️ Common Mistakes

❌ ViewModel में repository call करना
❌ UseCase 500 lines का बना देना
❌ Multiple responsibilities डाल देना
❌ DTO use करना

---

🏁 Final Summary

UseCase:

- App का business action represent करता है
- Clean Architecture का core part है
- ViewModel को clean रखता है
- Testability improve करता है
- Enterprise level apps में mandatory है

---

📌 Related Docs:

- Architecture.md
- Repository.md
- RepositoryImpl.md
- DTO.md
- Mapper.md
- RemoteDataSource.md


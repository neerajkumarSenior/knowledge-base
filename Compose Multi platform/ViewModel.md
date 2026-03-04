🧠 ViewModel Documentation (PostViewModel.md)

यह डॉक्यूमेंट Clean Architecture + MVVM में ViewModel की पूरी role, responsibility, structure, best practices और production-level implementation को detail में explain करता है।

---

🎯 ViewModel क्या है?

ViewModel:

«🧩 UI और Domain layer के बीच का bridge है
📡 UseCase call करता है
📦 UI State manage करता है
🔄 Reactive updates देता है»

ViewModel का काम है:

- UseCase call करना
- Result collect करना
- UI state update करना
- Lifecycle-safe coroutine handle करना

---

🏗 Architecture Position

UI (Compose Screen)
 ↓
PostViewModel
 ↓
UseCase
 ↓
Repository
 ↓
Data Layer

---

📂 Recommended Structure

ui/
 ├── PostScreen.kt
 ├── PostViewModel.kt
 └── PostState.kt

---

📦 Step 1: UI State Class

data class PostState(
    val posts: List<Post> = emptyList(),
    val isLoading: Boolean = false,
    val error: String? = null
)

👉 Multiple boolean flags avoid करो
👉 Single state object use करो

---

📦 Step 2: Basic ViewModel

class PostViewModel(
    private val getPostsUseCase: GetPostsUseCase
) : ViewModel() {

    private val _state = MutableStateFlow(PostState())
    val state: StateFlow<PostState> = _state

    init {
        loadPosts()
    }

    fun loadPosts() {
        viewModelScope.launch {
            _state.value = _state.value.copy(isLoading = true)

            try {
                val posts = getPostsUseCase()
                _state.value = PostState(posts = posts)
            } catch (e: Exception) {
                _state.value = PostState(error = e.message)
            }
        }
    }
}

---

🔥 Flow Based ViewModel (Recommended Production)

अगर UseCase Flow return करता है:

class PostViewModel(
    private val getPostsUseCase: GetPostsUseCase
) : ViewModel() {

    private val _state = MutableStateFlow(PostState())
    val state: StateFlow<PostState> = _state

    fun loadPosts() {
        viewModelScope.launch {

            getPostsUseCase().collect { result ->

                when (result) {

                    is Result.Loading -> {
                        _state.value = _state.value.copy(
                            isLoading = true
                        )
                    }

                    is Result.Success -> {
                        _state.value = PostState(
                            posts = result.data
                        )
                    }

                    is Result.Error -> {
                        _state.value = PostState(
                            error = result.message
                        )
                    }
                }
            }
        }
    }
}

---

🧠 ViewModel Responsibilities

Responsibility| Example
UseCase call करना| getPostsUseCase()
UI state manage करना| PostState
Loading handle करना| isLoading
Error show करना| error message
Lifecycle safe coroutine| viewModelScope

---

❌ ViewModel क्या नहीं करना चाहिए?

❌ API direct call
❌ DTO handle करना
❌ Database access
❌ Business validation
❌ Mapping logic

---

🧩 Koin DI Example

viewModel {
    PostViewModel(
        getPostsUseCase = get()
    )
}

---

🔁 Compose Integration Example

@Composable
fun PostScreen(viewModel: PostViewModel = koinViewModel()) {

    val state by viewModel.state.collectAsState()

    when {
        state.isLoading -> LoadingUI()
        state.error != null -> ErrorUI(state.error)
        else -> PostList(state.posts)
    }
}

---

🛡 Advanced Production ViewModel

1️⃣ One-Time Events Handling

private val _event = MutableSharedFlow<String>()
val event = _event.asSharedFlow()

suspend fun showMessage(msg: String) {
    _event.emit(msg)
}

Use for:

- Toast
- Snackbar
- Navigation

---

2️⃣ BaseViewModel Pattern

abstract class BaseViewModel : ViewModel() {

    protected fun launchSafely(
        block: suspend () -> Unit
    ) {
        viewModelScope.launch {
            try {
                block()
            } catch (e: Exception) {
                handleError(e)
            }
        }
    }

    open fun handleError(e: Exception) {}
}

---

🧪 Testing ViewModel

@Test
fun `should load posts successfully`() = runTest {

    val fakeUseCase = FakeGetPostsUseCase()
    val viewModel = PostViewModel(fakeUseCase)

    viewModel.loadPosts()

    val state = viewModel.state.first()

    assertTrue(state.posts.isNotEmpty())
}

---

🏆 ViewModel vs UseCase

ViewModel| UseCase
UI state manage करता है| Business rule apply करता है
Lifecycle aware| Pure Kotlin
Coroutine scope रखता है| Scope नहीं रखता
UI specific| App logic specific

---

🔥 Production Flow

User Click
 ↓
PostScreen
 ↓
PostViewModel
 ↓
GetPostsUseCase
 ↓
PostRepository
 ↓
RemoteDataSource
 ↓
Server

---

🧠 Golden Rules

✅ UseCase inject करो
✅ StateFlow use करो
✅ Single state object रखो
✅ Business logic मत डालो
✅ Small methods रखो
✅ One-time events separate रखो

---

⚠️ Common Mistakes

❌ Repository directly UI में inject करना
❌ ViewModel में 1000 lines code
❌ MutableState expose करना
❌ DTO expose करना
❌ Compose state mix करना

---

🚀 Enterprise Enhancements

- MVI Pattern
- SavedStateHandle
- Paging3 Integration
- State Reducer Pattern
- Event Channel Pattern
- Background sync trigger

---

🏁 Final Summary

ViewModel:

- UI और Domain के बीच bridge है
- State management का center है
- Lifecycle safe execution देता है
- Clean Architecture maintain करता है
- Scalable production apps में mandatory है

---

📌 Related Docs:

- Architecture.md
- UseCase.md
- Repository.md
- RepositoryImpl.md
- DTO.md
- Mapper.md
- RemoteDataSource.md


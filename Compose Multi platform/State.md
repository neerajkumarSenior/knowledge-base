🧩 UI State Documentation (PostState.md)

यह डॉक्यूमेंट Clean Architecture + MVVM में UI State (PostState) की पूरी concept, design principles, patterns, और production-ready implementation को detail में explain करता है।

---

🎯 UI State क्या है?

UI State वह data structure है जो:

«🖥 Screen की current हालत (स्थिति) को represent करता है»

Example:

- Loading चल रहा है
- Data आ चुका है
- Error आया है
- Empty list है

UI को कभी भी direct data source से नहीं जुड़ना चाहिए।
UI सिर्फ State observe करती है।

---

🏗 Architecture Position

UI (Compose Screen)
    ↑
PostState (Single Source of Truth)
    ↑
PostViewModel

👉 UI सिर्फ state पढ़ती है
👉 ViewModel state update करता है

---

📦 Basic PostState

data class PostState(
    val posts: List<Post> = emptyList(),
    val isLoading: Boolean = false,
    val error: String? = null
)

---

🧠 State Design Principles

1️⃣ Single Source of Truth

पूरी screen की हालत एक ही state object में होनी चाहिए।

❌ Multiple LiveData / StateFlow
✅ Single StateFlow<PostState>

---

2️⃣ Immutable State

State हमेशा immutable होनी चाहिए।

_state.value = _state.value.copy(isLoading = true)

👉 MutableState expose मत करो
👉 "copy()" use करो

---

3️⃣ UI Driven Model

State UI के हिसाब से design होनी चाहिए
Database के हिसाब से नहीं

---

🔥 Recommended Production Pattern (Sealed State)

Better approach:

sealed interface PostState {

    data object Loading : PostState

    data class Success(
        val posts: List<Post>
    ) : PostState

    data class Error(
        val message: String
    ) : PostState
}

---

ViewModel Usage

private val _state = MutableStateFlow<PostState>(PostState.Loading)
val state = _state.asStateFlow()

fun loadPosts() {
    viewModelScope.launch {
        try {
            val posts = getPostsUseCase()
            _state.value = PostState.Success(posts)
        } catch (e: Exception) {
            _state.value = PostState.Error(e.message ?: "Unknown error")
        }
    }
}

---

Compose Usage

when (val state = stateFlow.collectAsState().value) {

    PostState.Loading -> LoadingUI()

    is PostState.Success -> PostList(state.posts)

    is PostState.Error -> ErrorUI(state.message)
}

---

🧠 Boolean Flags Problem

❌ Bad Example:

val isLoading: Boolean
val isError: Boolean
val isEmpty: Boolean

Problem:

- Multiple states true हो सकते हैं
- Impossible combinations बन सकते हैं

---

🏆 Why Sealed Class is Better?

Boolean State| Sealed State
Confusing| Clear
Multiple flags| Single state
Error prone| Safe
Hard to scale| Easy to extend

---

🧩 Advanced: Partial UI State

अगर screen complex हो:

data class PostState(
    val posts: List<Post> = emptyList(),
    val isRefreshing: Boolean = false,
    val isPaginating: Boolean = false,
    val error: String? = null
)

Used in:

- Pull to refresh
- Pagination
- Infinite scroll

---

🧪 Testing State

@Test
fun `should emit success state`() = runTest {

    viewModel.loadPosts()

    val state = viewModel.state.first()

    assertTrue(state is PostState.Success)
}

---

🔁 State Update Pattern

Always use:

_state.update { current ->
    current.copy(isLoading = true)
}

Instead of:

_state.value = ...

Safer for concurrency.

---

🎯 State vs Event Difference

State| Event
Persistent| One-time
Survive rotation| Not survive
Example: posts list| Example: Toast

---

Event Example

private val _event = MutableSharedFlow<String>()
val event = _event.asSharedFlow()

---

🛡 Enterprise Pattern (Reducer Style)

fun reduce(old: PostState, action: Action): PostState {
    return when(action) {
        Action.Loading -> PostState.Loading
        is Action.Success -> PostState.Success(action.posts)
        is Action.Error -> PostState.Error(action.message)
    }
}

Used in MVI architecture.

---

🧠 Golden Rules

✅ Single state object
✅ Immutable data class
✅ UI driven structure
✅ Sealed class for clear states
✅ StateFlow expose करो
✅ MutableStateFlow private रखो

---

⚠️ Common Mistakes

❌ MutableState expose करना
❌ DTO store करना
❌ Multiple LiveData use करना
❌ Business logic state में डालना
❌ 20 boolean flags रखना

---

🚀 Real World Flow

User Action
   ↓
ViewModel
   ↓
_state.update()
   ↓
Compose Recompose
   ↓
UI Update

---

🏁 Final Summary

UI State:

- Screen की पूरी हालत represent करता है
- ViewModel द्वारा manage होता है
- UI सिर्फ observe करती है
- Immutable होना चाहिए
- Production में sealed class better है
- Clean Architecture maintain करता है

---

📌 Related Docs:

- ViewModel.md
- UseCase.md
- Repository.md
- Mapper.md
- DTO.md
- Architecture.md


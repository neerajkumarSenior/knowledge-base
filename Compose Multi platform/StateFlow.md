🌊 StateFlow Deep Dive (StateFlow.md)

यह डॉक्यूमेंट StateFlow को MVVM / MVI architecture में deeply explain करता है —
क्या है, कैसे काम करता है, क्यों UI state के लिए best है, और production best practices क्या हैं।

---

🧠 StateFlow क्या है?

"StateFlow" Kotlin Coroutines का hot observable state holder है।

«📌 यह हमेशा एक current value रखता है
📌 New collector को latest value तुरंत मिलती है
📌 UI state management के लिए ideal है»

---

🎯 Why StateFlow for UI?

UI को चाहिए:

- Current state
- Survive configuration change
- Automatic recomposition (Compose)
- Always latest value

StateFlow इन सबको solve करता है।

---

🏗 Basic Structure (ViewModel)

private val _state = MutableStateFlow(PostState())
val state: StateFlow<PostState> = _state

Rule:

- "_state" → private
- "state" → public read-only

---

📦 Update State

_state.value = PostState(isLoading = true)

Better (thread-safe):

_state.update { current ->
    current.copy(isLoading = true)
}

---

🎨 Compose Integration

@Composable
fun PostScreen(viewModel: PostViewModel = koinViewModel()) {

    val state by viewModel.state.collectAsState()

    when {
        state.isLoading -> LoadingUI()
        state.error != null -> ErrorUI(state.error!!)
        else -> PostList(state.posts)
    }
}

👉 State change → UI automatically recompose

---

🔥 StateFlow Characteristics

Feature| Behavior
Type| Hot Flow
Holds value| ✅ Yes
Default value| Required
Replay| Always 1 (latest)
Multiple collectors| ✅ Yes
Lifecycle aware| With collect

---

🧠 StateFlow vs LiveData

Feature| StateFlow| LiveData
Kotlin-first| ✅| ❌
Coroutine support| Native| Limited
Cold/Hot| Hot| Hot
Null safety| Better| Less
Flow operators| Yes| No

Modern Android → Prefer StateFlow

---

🔄 StateFlow vs SharedFlow

Feature| StateFlow| SharedFlow
Holds state| ✅ Yes| ❌ No
Replay default| 1| Configurable
For UI state| ✅ Yes| ❌
For events| ❌| ✅

---

🧩 MVI Pattern with StateFlow

private val _state = MutableStateFlow(PostState())
val state = _state.asStateFlow()

private fun dispatch(action: PostAction) {
    _state.update { old ->
        reduce(old, action)
    }
}

Reducer produces new immutable state.

---

🛡 Production Best Practices

✅ 1. Always Immutable State

data class PostState(
    val posts: List<Post> = emptyList(),
    val isLoading: Boolean = false,
    val error: String? = null
)

Never mutate list inside state.

---

✅ 2. Never Expose MutableStateFlow

❌ Wrong:

val state = MutableStateFlow(...)

✅ Correct:

private val _state = MutableStateFlow(...)
val state = _state.asStateFlow()

---

✅ 3. Use update{} Instead of value=

Safer for concurrency.

---

✅ 4. Single State Per Screen

Don't create:

val loadingFlow
val errorFlow
val postFlow

Keep single unified state.

---

🚨 Common Mistakes

❌ Putting one-time events in StateFlow
❌ Exposing mutable state
❌ Using multiple state flows
❌ Storing DTO in state
❌ Huge state with business logic

---

🧪 Testing StateFlow

@Test
fun `should emit loading then success`() = runTest {

    viewModel.loadPosts()

    val states = viewModel.state.take(2).toList()

    assertTrue(states[0].isLoading)
    assertTrue(states[1].posts.isNotEmpty())
}

---

🔬 Internal Working (Conceptually)

StateFlow internally:

- Always keeps latest value
- Uses atomic updates
- Immediately pushes latest state to new collectors

Think of it as:

Observable State Container

---

🏆 Real World Flow

User Click
   ↓
ViewModel
   ↓
_state.update()
   ↓
StateFlow emits new value
   ↓
Compose collects
   ↓
Recomposition

---

🎯 When NOT to Use StateFlow?

Avoid for:

- Navigation events
- Toast messages
- Snackbar
- One-time triggers

Use SharedFlow instead.

---

🚀 Enterprise Architecture Pattern (2026 Standard)

State → StateFlow
Event → SharedFlow
Internal coroutine communication → Channel

---

🏁 Final Summary

StateFlow:

- UI state holder है
- Hot flow है
- Always latest value रखता है
- Compose के लिए ideal है
- Immutable pattern के साथ powerful है
- Clean Architecture का core component है


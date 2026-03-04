🔥 SharedFlow Deep Dive (SharedFlow.md)

यह डॉक्यूमेंट SharedFlow को MVVM / MVI architecture में deeply explain करता है —
क्या है, कैसे काम करता है, क्यों UI events के लिए best है, और production best practices क्या हैं।

---

🧠 SharedFlow क्या है?

"SharedFlow" Kotlin Coroutines का एक hot broadcast flow है।

«📡 Multiple collectors को same data broadcast करता है
📌 State hold नहीं करता (by default)
📌 UI events के लिए ideal है»

---

🎯 SharedFlow क्यों जरूरी है?

UI में कुछ actions होते हैं जो:

- One-time होते हैं
- Persist नहीं होने चाहिए
- Rotation survive नहीं करने चाहिए

Example:

- Toast
- Snackbar
- Navigation
- Dialog open
- Payment success message

इनके लिए SharedFlow best है।

---

🏗 Basic Setup (ViewModel)

private val _event = MutableSharedFlow<PostEvent>()
val event = _event.asSharedFlow()

Rule:

- "_event" → private
- "event" → public read-only

---

📦 Emit Event

viewModelScope.launch {
    _event.emit(
        PostEvent.ShowToast("Post Created")
    )
}

---

🎨 Compose Collection

LaunchedEffect(Unit) {
    viewModel.event.collect { event ->
        when (event) {
            is PostEvent.ShowToast -> {
                // show toast
            }
            PostEvent.NavigateBack -> {
                // navigate
            }
        }
    }
}

---

🔥 SharedFlow Characteristics

Feature| Behavior
Type| Hot Flow
Holds value| ❌ No (default)
Replay| Configurable
Multiple collectors| ✅ Yes
Lifecycle aware| With collect
Backpressure| Configurable

---

🧠 SharedFlow vs StateFlow

Feature| StateFlow| SharedFlow
Holds current state| ✅ Yes| ❌ No
Default replay| 1| 0
For UI state| ✅| ❌
For UI events| ❌| ✅
Requires initial value| ✅| ❌

---

🧩 SharedFlow Configuration

Default:

MutableSharedFlow<PostEvent>()

Equivalent to:

MutableSharedFlow<PostEvent>(
    replay = 0,
    extraBufferCapacity = 0
)

---

🔥 Recommended UI Configuration

MutableSharedFlow<PostEvent>(
    replay = 0,
    extraBufferCapacity = 1
)

Why?

- replay = 0 → old event repeat नहीं होगा
- extraBufferCapacity = 1 → fast emit safe

---

🛡 MVI Pattern (Effect)

MVI में SharedFlow को अक्सर Effect कहा जाता है।

private val _effect = MutableSharedFlow<PostEffect>()
val effect = _effect.asSharedFlow()

---

🔄 Event Flow Diagram

User Action
   ↓
Intent
   ↓
ViewModel
   ↓
UseCase
   ↓
_emit SharedFlow
   ↓
UI Collect
   ↓
Perform Action

---

🧪 Testing SharedFlow

@Test
fun `should emit navigation event`() = runTest {

    viewModel.createPost("Title", "Body")

    val event = viewModel.event.first()

    assertTrue(event is PostEvent.NavigateBack)
}

---

🚨 Common Mistakes

❌ replay = 1 रखना (duplicate event issue)
❌ SharedFlow को state के लिए use करना
❌ MutableSharedFlow expose करना
❌ collectAsState() से collect करना (event के लिए wrong)
❌ UI से emit करना

---

🎯 SharedFlow vs Channel

Feature| SharedFlow| Channel
Multiple collectors| ✅| ❌
Broadcast| ✅| ❌
Replay support| ✅| ❌
Recommended for UI| ✅| ⚠️ Limited
Coroutine primitive| Higher-level| Lower-level

Modern Android → Prefer SharedFlow

---

🧠 Advanced: Replay Feature

MutableSharedFlow<String>(replay = 1)

Use case:

- Last emitted value cache करना
- Analytics stream
- Configuration restore logic

⚠️ UI events में usually avoid करें

---

🏆 Enterprise Standard (2026)

State  → StateFlow
Event  → SharedFlow
Internal coroutine comm → Channel

---

🎯 Real World Example

Scenario:

User creates post:

1. API success
2. Emit toast
3. Navigate back

_event.emit(PostEvent.ShowToast("Success"))
_event.emit(PostEvent.NavigateBack)

UI performs both actions once.

---

🏁 Final Summary

SharedFlow:

- Hot broadcast flow है
- One-time UI events के लिए best है
- Multiple collectors support करता है
- Replay configurable है
- StateFlow का replacement नहीं है
- Channel से better choice है UI layer में

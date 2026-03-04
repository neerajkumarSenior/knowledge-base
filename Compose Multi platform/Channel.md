📡 Kotlin Channel for UI Events (Channel.md)

यह डॉक्यूमेंट Kotlin Channel को MVVM / MVI architecture में समझाता है —
कब use करें, कैसे use करें, और क्यों अक्सर "SharedFlow" से compare किया जाता है।

---

🧠 Channel क्या है?

"Channel" Kotlin Coroutines का primitive है:

«📬 Producer → Consumer communication mechanism»

It works like:

Send → Receive

एक coroutine send करता है
दूसरा coroutine receive करता है

---

🎯 UI Architecture में Channel का Use

Mainly used for:

- One-time events
- Navigation triggers
- Toast/Snackbar
- Dialog open

---

📦 Basic Channel Example (ViewModel)

private val _eventChannel = Channel<PostEvent>()
val events = _eventChannel.receiveAsFlow()

---

📦 Emit Event

viewModelScope.launch {
    _eventChannel.send(
        PostEvent.ShowToast("Post Created")
    )
}

---

📦 Collect in Compose

LaunchedEffect(Unit) {
    viewModel.events.collect { event ->
        when (event) {
            is PostEvent.ShowToast -> {
                // Show toast
            }
            PostEvent.NavigateBack -> {
                // Navigate
            }
        }
    }
}

---

🔥 Channel vs SharedFlow

Feature| Channel| SharedFlow
Multiple collectors| ❌ No| ✅ Yes
Replay support| ❌ No| ✅ Yes
Backpressure| Built-in| Manual
Cold/Hot| Cold-like| Hot
Recommended for UI| ⚠️ Limited| ✅ Yes

---

🧠 Channel Types

1️⃣ Rendezvous Channel (Default)

Channel<PostEvent>()

- Buffer size = 0
- Sender suspend होगा जब तक receiver consume न करे

---

2️⃣ Buffered Channel

Channel<PostEvent>(capacity = Channel.BUFFERED)

- Some buffer available
- Rarely used in UI

---

3️⃣ Conflated Channel

Channel<PostEvent>(capacity = Channel.CONFLATED)

- Only latest value रखता है
- Old events drop हो जाते हैं

⚠️ UI events के लिए risky

---

🚨 Channel Problems in UI

1️⃣ Single Collector Only

Channel multiple collectors support नहीं करता।

अगर:

- Two composables collect करें
- Or testing + UI collect करे

Problem आ सकता है।

---

2️⃣ Lifecycle Issue

अगर UI active नहीं है:

- Event lost हो सकता है

Channel events survive नहीं करते।

---

3️⃣ Harder to Debug

Channel low-level primitive है
SharedFlow higher-level API है

---

🏆 Why SharedFlow is Preferred Today

Google recommendation:

«Use "SharedFlow" for UI events instead of Channel.»

Because:

- Better multi-subscriber support
- Replay control
- Hot stream
- More flexible

---

🛡 When Should You Use Channel?

Use Channel when:

- One producer → One consumer guaranteed
- Strict backpressure needed
- Internal coroutine communication
- Not for UI layer primarily

Example:

Repository internal communication
Worker coordination
Pipeline processing

---

🧩 MVI + Channel Example (Advanced)

private val _effectChannel = Channel<PostEffect>()
val effect = _effectChannel.receiveAsFlow()

Emit:

viewModelScope.launch {
    _effectChannel.send(PostEffect.NavigateBack)
}

---

🔄 Channel vs SharedFlow in MVI

Pattern| Recommended
MVVM| SharedFlow
MVI| SharedFlow
Internal coroutine engine| Channel

---

🧪 Testing Channel

@Test
fun `should send navigation event`() = runTest {

    viewModel.createPost("Title", "Body")

    val event = viewModel.events.first()

    assertTrue(event is PostEvent.NavigateBack)
}

---

🧠 Deep Technical Difference

Channel

- Uses FIFO queue
- Suspends sender if no receiver
- Point-to-point communication

SharedFlow

- Broadcast style
- Multiple collectors allowed
- Configurable replay

---

🎯 Architecture Recommendation (2026 Standard)

State → StateFlow
Event → SharedFlow
Avoid Channel in UI layer

---

🏁 Final Summary

Channel:

- Coroutine communication primitive है
- Single consumer based है
- UI events के लिए limited है
- SharedFlow modern replacement है
- Mostly internal coroutine use के लिए best है

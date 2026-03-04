⚡ UI Event Handling Guide (Event.md)

यह डॉक्यूमेंट Clean Architecture + MVVM/MVI में Event Handling (One-Time Events) को deep level पर explain करता है।

---

🧠 Event क्या होता है?

Event = One-time action

Example:

- Toast दिखाना
- Snackbar दिखाना
- Navigation करना
- Dialog open करना
- Success message दिखाना

---

🎯 State vs Event Difference

State| Event
Persistent| One-time
Rotation survive करता है| Rotation survive नहीं करता
Example: posts list| Example: "Post Created" toast
UI observe करता है| UI consume करता है

---

❌ Common Mistake

Developers अक्सर error message को state में डाल देते हैं:

data class PostState(
    val error: String? = null
)

Problem:

- Screen rotate → Error फिर से दिखेगा
- Duplicate toast
- Navigation repeat हो सकता है

---

✅ Correct Approach: Separate Event Flow

Use:

MutableSharedFlow

---

📦 Step 1: Define Event

sealed interface PostEvent {

    data class ShowToast(val message: String) : PostEvent

    data class ShowSnackbar(val message: String) : PostEvent

    data object NavigateBack : PostEvent

    data object NavigateToDetails : PostEvent
}

---

📦 Step 2: ViewModel Event Setup

private val _event = MutableSharedFlow<PostEvent>()
val event = _event.asSharedFlow()

---

📦 Step 3: Emit Event

private fun createPost(title: String, body: String) {

    viewModelScope.launch {

        try {
            createPostUseCase(title, body)

            _event.emit(
                PostEvent.ShowToast("Post Created Successfully")
            )

            _event.emit(PostEvent.NavigateBack)

        } catch (e: Exception) {

            _event.emit(
                PostEvent.ShowSnackbar(
                    e.message ?: "Unknown Error"
                )
            )
        }
    }
}

---

🎯 Compose Event Collection

@Composable
fun PostScreen(viewModel: PostViewModel = koinViewModel()) {

    val context = LocalContext.current

    LaunchedEffect(Unit) {

        viewModel.event.collect { event ->

            when (event) {

                is PostEvent.ShowToast -> {
                    Toast.makeText(
                        context,
                        event.message,
                        Toast.LENGTH_SHORT
                    ).show()
                }

                is PostEvent.ShowSnackbar -> {
                    // show snackbar
                }

                PostEvent.NavigateBack -> {
                    // navController.popBackStack()
                }

                PostEvent.NavigateToDetails -> {
                    // navigate
                }
            }
        }
    }
}

---

🔥 Why SharedFlow?

Option| Why Not
LiveData| Re-deliver issue
Channel| Single collector
StateFlow| Replay problem
SharedFlow| Best for events

---

🧠 SharedFlow Configuration (Advanced)

Default:

MutableSharedFlow<PostEvent>()

Advanced:

MutableSharedFlow<PostEvent>(
    replay = 0,
    extraBufferCapacity = 1
)

Explanation:

- replay = 0 → old events repeat नहीं होंगे
- extraBufferCapacity → missed emission avoid

---

🛡 Enterprise Pattern: Effect (MVI Style)

In MVI:

Event को Effect कहते हैं।

sealed interface PostEffect {

    data class ShowToast(val message: String) : PostEffect

    data object NavigateBack : PostEffect
}

---

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
Emit Event (SharedFlow)
   ↓
UI Collect
   ↓
Perform Action (Toast/Navigation)

---

🧪 Testing Events

@Test
fun `should emit success toast`() = runTest {

    viewModel.createPost("Title", "Body")

    val event = viewModel.event.first()

    assertTrue(event is PostEvent.ShowToast)
}

---

🏆 Golden Rules

✅ State और Event अलग रखो
✅ SharedFlow use करो
✅ replay = 0 रखो
✅ UI में collect करो
✅ ViewModel से emit करो
✅ Business logic event में मत डालो

---

⚠️ Common Mistakes

❌ Event को State में डालना
❌ MutableSharedFlow expose करना
❌ replay = 1 रखना
❌ UI से event trigger करना
❌ Event consume logic ViewModel में डालना

---

🚀 Production Level Structure

post/
 ├── PostState.kt
 ├── PostIntent.kt
 ├── PostAction.kt
 ├── PostEffect.kt
 └── PostViewModel.kt

---

🧠 Real World Examples

Scenario| Type
API error toast| Event
Login success navigation| Event
List of posts| State
Loading spinner| State
Dialog open| Event

---

🏁 Final Summary

Event:

- One-time action represent करता है
- UI द्वारा consume किया जाता है
- State से अलग रखा जाता है
- SharedFlow best solution है
- MVI में Effect कहलाता है

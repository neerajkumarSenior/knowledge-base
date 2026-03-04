🚀 Flow Operators Production Guide (FlowOperators.md)

यह guide Kotlin Flow operators को production-level Android apps (MVVM / MVI + Clean Architecture) में कैसे सही तरीके से use करें — यह detail में समझाती है।

---

🧠 Flow Operators क्या होते हैं?

Flow operators वो functions हैं जो:

«🌊 Flow के data को transform, filter, combine, control या handle करते हैं।»

Example:

flow.map { }
flow.filter { }
flow.catch { }
flow.combine { }

---

🏗 Production Architecture Context

DataSource → Repository → UseCase → ViewModel → UI

Operators ज़्यादातर use होते हैं:

- Repository layer
- UseCase layer
- ViewModel layer

---

🔄 1️⃣ Transform Operators

✅ map

Data transform करने के लिए।

repository.getPosts()
    .map { dtoList ->
        dtoList.map { it.toDomain() }
    }

Use Case:

- DTO → Domain
- Domain → UI Model

---

✅ mapLatest

Previous emission cancel कर देता है।

searchQueryFlow
    .mapLatest { query ->
        api.search(query)
    }

Best for:

- Search
- Typing input
- Rapid UI actions

---

🔍 2️⃣ Filter Operators

✅ filter

flow.filter { it.isNotEmpty() }

---

✅ distinctUntilChanged

Duplicate emissions avoid करता है।

stateFlow
    .distinctUntilChanged()

Use for:

- Text input
- State optimization
- Avoid recomposition

---

🔀 3️⃣ Combination Operators

✅ combine

Multiple flows combine करने के लिए।

combine(
    userFlow,
    postFlow
) { user, posts ->
    UserWithPosts(user, posts)
}

Used in:

- Dashboard screens
- Profile + posts
- Multi-source UI

---

✅ zip

Parallel combine but waits both.

flow1.zip(flow2) { a, b ->
    a to b
}

Rare in UI, more in backend logic.

---

⏱ 4️⃣ Timing Operators

✅ debounce

Rapid emission control करता है।

searchQuery
    .debounce(500)

Best for:

- Search field
- Input validation
- Network optimization

---

✅ sample

Fixed interval sampling।

flow.sample(1000)

Used in:

- Location tracking
- Sensors
- Analytics

---

🔥 5️⃣ FlatMap Family (Advanced)

✅ flatMapLatest (Most Important)

Old flow cancel करता है।

searchQuery
    .debounce(500)
    .flatMapLatest { query ->
        repository.search(query)
    }

Best for:

- Search
- Dependent API calls
- Dynamic filtering

---

❌ flatMapMerge

Parallel execution.

Rare in UI.

---

❌ flatMapConcat

Sequential execution.

Mostly backend style logic.

---

🛡 6️⃣ Error Handling Operators

✅ catch

repository.getPosts()
    .catch { e ->
        emit(emptyList())
    }

⚠️ Note:

- "catch" upstream errors handle करता है
- Downstream errors नहीं

---

✅ retry

flow.retry(3)

Network retry use case।

---

✅ onCompletion

flow.onCompletion {
    hideLoader()
}

---

🎯 7️⃣ Lifecycle Operators

✅ onStart

flow.onStart {
    emit(Loading)
}

Best for:

- Show loading state

---

✅ launchIn (ViewModel)

flow
    .onEach { }
    .launchIn(viewModelScope)

Alternative to collect {}

---

🧪 8️⃣ Terminal Operators

These trigger execution.

collect

flow.collect { }

---

first

val value = flow.first()

---

toList

val list = flow.toList()

---

🧩 Production Example (Search Feature)

val searchResults =
    searchQuery
        .debounce(500)
        .distinctUntilChanged()
        .flatMapLatest { query ->
            repository.search(query)
                .onStart { emit(emptyList()) }
                .catch { emit(emptyList()) }
        }
        .stateIn(
            viewModelScope,
            SharingStarted.WhileSubscribed(5000),
            emptyList()
        )

---

🔥 stateIn (Very Important)

Flow को StateFlow में convert करता है।

flow.stateIn(
    scope = viewModelScope,
    started = SharingStarted.WhileSubscribed(),
    initialValue = PostState()
)

Use when:

- UI state directly derive करना हो
- Cold flow → Hot state

---

🔥 shareIn

Flow को SharedFlow में convert करता है।

flow.shareIn(
    scope = viewModelScope,
    started = SharingStarted.Lazily
)

Used in:

- Event broadcasting
- Multi-subscriber streams

---

🚨 Production Mistakes

❌ ViewModel में heavy map logic
❌ Nested collect calls
❌ flatMapLatest misuse
❌ No debounce in search
❌ No distinctUntilChanged
❌ catch at wrong position
❌ Not using stateIn for UI state

---

🏆 Operator Selection Cheat Sheet

Use Case| Operator
Search| debounce + flatMapLatest
API retry| retry
Show loading| onStart
Transform data| map
Merge flows| combine
Avoid duplicates| distinctUntilChanged
Cancel old request| flatMapLatest
Convert to state| stateIn
Convert to shared| shareIn

---

🧠 Enterprise Standard Pattern

Repository → Flow<Data>
UseCase → Transform with map/combine
ViewModel → stateIn / shareIn
UI → collectAsState

---

🎯 Golden Rules

✅ UI state = StateFlow
✅ Events = SharedFlow
✅ Search = debounce + flatMapLatest
✅ Always handle errors
✅ Avoid nested collects
✅ Prefer declarative chains

---

🏁 Final Summary

Flow operators:

- Reactive architecture की backbone हैं
- सही operator selection performance improve करता है
- Production apps में optimization critical है
- Search, combine, retry, loading — सब operator driven हैं
- stateIn और shareIn must-know operators हैं


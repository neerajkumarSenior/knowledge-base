🚦 Backpressure in Kotlin Flow (Backpressure.md)

यह guide Kotlin Coroutines Flow में Backpressure क्या है, क्यों important है, और production Android apps में इसे कैसे handle करें — यह detail में explain करती है।

---

🧠 Backpressure क्या होता है?

«जब producer data जल्दी emit कर रहा हो
लेकिन consumer उसे धीरे process कर रहा हो
तो जो pressure create होता है उसे Backpressure कहते हैं।»

---

🎯 Simple Example

Producer → 100 items/sec
Consumer → 10 items/sec

Result?

- Buffer भर जाएगा
- Memory grow करेगी
- App lag हो सकता है

---

🏗 Flow Context में Backpressure

DataSource → Repository → ViewModel → UI

Backpressure mostly होता है:

- Search input
- Scroll events
- Location updates
- Sensors
- WebSocket
- Rapid API calls

---

📦 Basic Problem Example

flow {
    repeat(1000) {
        emit(it)
    }
}
.collect {
    delay(1000)
}

Producer fast
Consumer slow

Pressure build होगा।

---

🔥 Flow Backpressure Handling Tools

Kotlin Flow built-in operators देता है।

---

1️⃣ buffer()

flow
    .buffer()
    .collect { }

What it does?

- Producer fast emit कर सकता है
- Consumer buffer से process करेगा
- Suspension reduce होती है

---

Custom Buffer Size

.buffer(capacity = 64)

---

2️⃣ conflate()

flow
    .conflate()
    .collect { }

What it does?

- Intermediate values drop कर देता है
- Only latest value रखता है

Best for:

- UI state updates
- Progress indicators
- Location tracking

---

3️⃣ collectLatest()

flow.collectLatest { value ->
    delay(2000)
}

What it does?

- New value आने पर previous block cancel
- Only latest emission process

Best for:

- Search
- Rapid UI input
- Network calls

---

4️⃣ debounce()

searchQuery
    .debounce(500)

What it does?

- Waits for pause
- Rapid emissions ignore

Best for:

- Search field
- Typing input
- Validation

---

5️⃣ sample()

flow.sample(1000)

What it does?

- Fixed interval पर value emit

Used in:

- Sensors
- Analytics
- Tracking

---

📊 Operator Comparison

Operator| Strategy| Use Case
buffer| Queue values| Heavy processing
conflate| Drop old| UI updates
collectLatest| Cancel old| Search/API
debounce| Wait for pause| Text input
sample| Time-based pick| Sensors

---

🧠 Real Production Example (Search)

Without backpressure handling:

searchQuery
    .flatMapLatest { query ->
        api.search(query)
    }

With proper handling:

searchQuery
    .debounce(500)
    .distinctUntilChanged()
    .flatMapLatest { query ->
        repository.search(query)
    }

This prevents:

- 50 API calls per second
- UI freezing
- Server overload

---

🔥 UI Recomposition Backpressure

Compose + StateFlow:

stateFlow
    .distinctUntilChanged()

Prevents:

- Unnecessary recomposition
- Performance drop

---

🛡 Buffer Strategy Types

Flow internally supports:

buffer(capacity, onBufferOverflow = BufferOverflow.DROP_OLDEST)

Options:

- SUSPEND (default)
- DROP_OLDEST
- DROP_LATEST

---

🚨 Common Mistakes

❌ No debounce in search
❌ Not using collectLatest
❌ Large buffer without need
❌ Ignoring cancellation
❌ Blocking inside collect

---

🎯 Backpressure in Hot vs Cold Flow

Cold Flow:

- Backpressure limited to collector

Hot Flow:

- Shared among collectors
- More risk of overflow

Use "buffer()" carefully.

---

🧪 Testing Backpressure

flow
    .buffer()
    .test {
        expectMostRecentItem()
    }

Ensure:

- Old values dropped when needed
- Latest value preserved

---

🏗 Enterprise Scenario

Example: Live Ride Tracking

Location updates: 10/sec
UI refresh needed: 1/sec

Solution:

locationFlow
    .conflate()
    .sample(1000)

Efficient + smooth UI.

---

🧠 Mental Model

Backpressure = Traffic control system 🚦

If traffic heavy:

- Add lane (buffer)
- Drop old cars (conflate)
- Cancel previous route (collectLatest)
- Wait for pause (debounce)

---

🏁 Final Summary

Backpressure:

- Producer-consumer speed mismatch problem
- Flow provides built-in tools
- Search → debounce + flatMapLatest
- UI updates → conflate
- Heavy processing → buffer
- Sensors → sample
- Always consider performance

Modern Android Reactive Rule:

Fast Producer + Slow Consumer = Handle Backpressure

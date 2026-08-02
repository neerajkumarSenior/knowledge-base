# Structured Concurrency in Kotlin: Using `coroutineScope` and `supervisorScope` Correctly

> **Concurrency केवल कई Coroutines चलाने का नाम नहीं है। सही Scope चुनना भी उतना ही महत्वपूर्ण है। `coroutineScope` और `supervisorScope` अलग-अलग Failure Strategies को दर्शाते हैं।**

---

# 📌 समस्या (Problem)

कई Applications में एक Business Operation के दौरान कई Services को Call किया जाता है।

उदाहरण:

- Payment Authorization
- Risk Engine
- Fraud Detection

अक्सर Code कुछ इस प्रकार लिखा जाता है:

```kotlin
suspend fun authorize(req: Request): Outcome {

    val auth = acquirer.authorize(req)

    val risk = try {
        riskEngine.score(req)
    } catch (e: Exception) {
        null
    }

    val fraud = try {
        fraudService.check(req)
    } catch (e: Exception) {
        null
    }

    return Outcome.from(auth, risk, fraud)
}
```

पहली नज़र में यह Code सही लगता है।

लेकिन इसमें कई गंभीर समस्याएँ छिपी हुई हैं।

---

# ❌ इसमें समस्या क्या है?

तीनों Services एक-दूसरे से स्वतंत्र (Independent) तरीके से Call हो रही हैं।

```
Authorization

Risk Engine

Fraud Service
```

यदि इनमें से कोई Fail हो जाए,

तो हर Failure अलग-अलग Handle करनी पड़ती है।

उदाहरण:

- Risk Engine Fail
- Fraud Service Fail
- Authorization Fail

अब Code में कई `try-catch` Blocks आ जाते हैं।

---

# एक और बड़ी समस्या

मान लीजिए

Risk Engine Fail हो गया।

Code ने Exception पकड़ ली।

```kotlin
risk = null
```

अब आगे Processing जारी रहती है।

```
Authorization ✔

Risk ❌

Fraud ✔

↓

Approve Payment ❌
```

यानी Risk Check Fail होने के बावजूद Payment Approve हो सकती है।

यह Business Rule का उल्लंघन है।

---

# ऐसा क्यों होता है?

क्योंकि Code की Structure Business Rules को Represent नहीं करती।

Developer स्वयं Decide कर रहा है कि कौन-सी Failure Ignore करनी है।

---

# ✅ समाधान (Solution)

Kotlin की **Structured Concurrency** का उपयोग करें।

Business Requirement के अनुसार Scope चुनें।

---

# `coroutineScope`

यदि सभी Operations सफल होना आवश्यक हैं,

तो `coroutineScope` का उपयोग करें।

```kotlin
suspend fun authorize(
    req: Request
) = coroutineScope {

    val auth = async {
        acquirer.authorize(req)
    }

    val risk = async {
        riskEngine.score(req)
    }

    val fraud = async {
        fraudService.check(req)
    }

    Outcome.from(
        auth.await(),
        risk.await(),
        fraud.await()
    )
}
```

---

# इसका व्यवहार

यदि कोई भी Child Coroutine Fail होती है,

तो

- बाकी सभी Cancel हो जाती हैं।
- पूरा Operation Fail हो जाता है।

```
Authorization ✔

Risk ❌

Fraud (Cancelled)

↓

Authorization Failed
```

इसे **Fail Fast** कहते हैं।

---

# कब उपयोग करें?

जब सभी Operations आवश्यक हों।

उदाहरण:

- Payment Authorization
- Banking Transactions
- Money Transfer
- OTP Verification
- Account Creation

इनमें Partial Success स्वीकार नहीं किया जा सकता।

---

# `supervisorScope`

कुछ Operations Independent होती हैं।

उदाहरण:

User Signup के बाद

- SMS भेजना
- Analytics भेजना
- Activity Feed Update करना

यदि SMS Fail हो जाए,

तो Signup Cancel नहीं होना चाहिए।

ऐसी स्थिति में

```kotlin
suspend fun fanOut(event: Event) =
    supervisorScope {

        listOf(

            async { sms(event) },

            async { analytics(event) },

            async { feed(event) }

        ).awaitAll()
    }
```

---

# इसका व्यवहार

यदि एक Child Fail होती है,

बाकी Coroutines चलती रहती हैं।

```
SMS ❌

Analytics ✔

Feed ✔
```

बाकी Tasks प्रभावित नहीं होतीं।

इसे **Failure Isolation** कहते हैं।

---

# `coroutineScope` बनाम `supervisorScope`

| विशेषता | coroutineScope | supervisorScope |
|---------|----------------|-----------------|
| Child Fail होने पर | सभी Cancel | बाकी चलते रहते हैं |
| Failure Strategy | Fail Fast | Failure Isolation |
| Partial Success | स्वीकार नहीं | स्वीकार है |
| उपयोग | Critical Operations | Independent Tasks |

---

# वास्तविक उदाहरण

## Payment Processing

```
Authorize

Risk Check

Fraud Check
```

यदि Fraud Check Fail हो जाए,

तो पूरा Payment Fail होना चाहिए।

✅ `coroutineScope`

---

## Notification System

```
Send SMS

Send Email

Send Push Notification
```

यदि SMS Fail हो जाए,

तो Email और Push फिर भी भेजे जाने चाहिए।

✅ `supervisorScope`

---

# इस Approach के लाभ

## ✅ 1. Business Rules Code में दिखाई देती हैं

Scope देखकर ही समझ आता है कि Failure कैसे Handle होगी।

---

## ✅ 2. कम `try-catch`

हर जगह Exception Handle करने की आवश्यकता नहीं रहती।

---

## ✅ 3. Automatic Cancellation

`coroutineScope` में Fail होने पर बाकी Coroutines स्वतः Cancel हो जाती हैं।

---

## ✅ 4. Failure Isolation

`supervisorScope` में एक Failure बाकी Tasks को प्रभावित नहीं करती।

---

## ✅ 5. Better Readability

Code देखकर तुरंत समझ आता है कि

- Fail Fast चाहिए
- या Partial Success

---

## ✅ 6. बेहतर Resource Management

अनावश्यक Coroutines चलती नहीं रहतीं।

---

## ✅ 7. Cleaner Concurrent Code

Concurrency Business Logic के अनुसार Structured रहती है।

---

# यह Pattern कहाँ उपयोग करें?

**`coroutineScope`**

- Payment Gateway
- Banking
- Authentication
- Order Placement
- Money Transfer
- Identity Verification

---

**`supervisorScope`**

- Notification System
- Analytics
- Logging
- Activity Feed
- Background Jobs
- Metrics Collection

---

# Before

```text
Multiple Independent Calls

↓

Multiple try-catch

↓

Hidden Failures

↓

Complex Code
```

---

# After

```text
Structured Concurrency

↓

Correct Scope

↓

Predictable Failure

↓

Cleaner Code
```

---

# मुख्य सीख (Key Takeaway)

> **Concurrency केवल Parallel Execution नहीं है। यह Failure Strategy भी है।**

- यदि सभी Operations सफल होना आवश्यक हैं → **`coroutineScope`**
- यदि Partial Success स्वीकार है → **`supervisorScope`**

Scope का चुनाव आपकी Business Requirement के अनुसार होना चाहिए।

---

# निष्कर्ष (Conclusion)

Structured Concurrency Kotlin Coroutines की सबसे महत्वपूर्ण विशेषताओं में से एक है।

सही Scope चुनकर आप:

- ✔ Business Rules को Code में स्पष्ट रूप से व्यक्त कर सकते हैं।
- ✔ Failure Handling को सरल बना सकते हैं।
- ✔ Resource Leaks से बच सकते हैं।
- ✔ अधिक सुरक्षित और Maintainable Concurrent Applications बना सकते हैं।

> **Golden Rule:**  
> **`coroutineScope` = Fail Fast**  
> **`supervisorScope` = Isolate Failures**

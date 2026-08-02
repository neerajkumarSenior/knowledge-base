# Timeout as a Named Outcome in Kotlin

> **Timeout को केवल एक Exception की तरह मत मानिए। उसे अपने Domain का एक स्पष्ट (Named) Outcome बनाइए।**

---

# 📌 समस्या (Problem)

जब किसी External Service (जैसे Payment Gateway, Bank API या Third-Party API) को Call किया जाता है, तब अक्सर `withTimeout()` का उपयोग किया जाता है।

उदाहरण:

```kotlin
suspend fun processAuth(
    req: Request
) = withTimeout(hostTimeoutMs) {

    acquirer.authorize(req)
}
```

यदि निर्धारित समय के भीतर Response नहीं मिलता,

तो Kotlin

```kotlin
TimeoutCancellationException
```

Throw कर देता है।

---

# ❌ इसमें समस्या क्या है?

Timeout एक Business Event नहीं बनता,

बल्कि केवल एक Exception बन जाता है।

अब यह इस बात पर निर्भर करता है कि

ऊपर वाले Layer ने Exception Catch किया या नहीं।

```
Payment Service

↓

Timeout Exception

↓

Controller

↓

Service

↓

Caller

↓

Maybe Catch
```

यदि किसी Layer ने Exception Handle नहीं की,

तो पूरी Request Fail हो सकती है।

---

# वास्तविक उदाहरण

मान लीजिए

Payment Gateway का Timeout

```
30 Seconds
```

है।

लेकिन

Bank

```
31 Seconds
```

में Payment Approve कर देता है।

Application को क्या पता?

उसे केवल Timeout मिला।

लेकिन वास्तव में

```
Customer Money

↓

Deducted ✔

↓

Application

↓

Timeout ❌
```

अब System सोचता है कि Payment Fail हुई।

लेकिन Bank ने पैसा काट लिया।

इसे

**Silent Money Loss**

या

**Unknown Transaction State**

कहा जाता है।

---

# ऐसा क्यों होता है?

क्योंकि Timeout को Exception माना गया है।

जबकि वास्तव में

Timeout भी Business Outcome है।

उदाहरण:

- Approved
- Declined
- Timed Out

तीनों समान रूप से Valid Results हैं।

---

# ✅ समाधान (Solution)

Timeout को Exception की जगह

एक Named Outcome बनाइए।

```kotlin
sealed interface AuthorizationOutcome
```

अब सभी संभावित परिणाम Define कीजिए।

```kotlin
data class Approved(
    val authCode: AuthCode
) : AuthorizationOutcome

data class Declined(
    val reason: DeclineReason
) : AuthorizationOutcome

data object TimedOut
    : AuthorizationOutcome
```

अब Timeout भी Domain Model का हिस्सा है।

---

# Business Logic

अब Result Handle करना आसान हो जाता है।

```kotlin
when (
    val outcome = processAuth(req)
) {

    is Approved ->
        settleTransaction(
            outcome.authCode
        )

    is Declined ->
        notifyCardHolder(
            outcome.reason
        )

    is TimedOut ->
        initiateReversal(req)
}
```

अब Timeout कोई Unexpected Exception नहीं है।

बल्कि एक Expected Business Outcome है।

---

# पहले

```
Request

↓

Timeout Exception

↓

Maybe Catch

↓

Maybe Crash

↓

Maybe Retry
```

पूरा Flow Unpredictable था।

---

# बाद में

```
Request

↓

AuthorizationOutcome

├── Approved

├── Declined

└── TimedOut
```

हर स्थिति स्पष्ट रूप से Handle होती है।

---

# Timeout का अर्थ

Timeout का मतलब हमेशा

```
Payment Failed
```

नहीं होता।

इसका मतलब केवल इतना है:

```
Response समय पर नहीं मिला।
```

हो सकता है

- Payment Success हो चुकी हो।
- Payment Fail हुई हो।
- Gateway अभी भी Processing कर रहा हो।

इसीलिए Timeout को Business Outcome मानना अधिक सही है।

---

# वास्तविक उदाहरण

## Payment Gateway

Possible Outcomes

```
Approved

Declined

TimedOut
```

---

## OTP Service

Possible Outcomes

```
Sent

Rejected

TimedOut
```

---

## Bank Transfer

Possible Outcomes

```
Completed

Failed

TimedOut
```

---

# इस Approach के लाभ

## ✅ 1. Timeout Business Model का हिस्सा बन जाता है

अब वह Hidden Exception नहीं रहता।

---

## ✅ 2. बेहतर Error Handling

हर Outcome स्पष्ट रूप से Handle किया जाता है।

---

## ✅ 3. Compile-Time Safety

`sealed interface` के कारण Compiler सुनिश्चित करता है कि सभी Outcomes Handle किए जाएँ।

---

## ✅ 4. Silent Failures समाप्त

Timeout कहीं Lost नहीं होता।

---

## ✅ 5. बेहतर Recovery Logic

Timeout होने पर

- Retry
- Reversal
- Compensation
- Status Polling

जैसी Business Logic आसानी से लिखी जा सकती है।

---

## ✅ 6. अधिक स्पष्ट Domain Model

Code देखकर तुरंत समझ आता है कि

Business में कौन-कौन से परिणाम संभव हैं।

---

## ✅ 7. Production Bugs कम होते हैं

Unexpected Exceptions कम होती हैं।

Recovery पहले से Defined होती है।

---

# यह Pattern कहाँ उपयोग करें?

- Payment Gateway
- Banking APIs
- UPI Transactions
- OTP Services
- SMS Providers
- Email Services
- External REST APIs
- Message Brokers
- Inventory Services
- Shipping APIs

---

# Before

```kotlin
withTimeout {

    gateway.call()
}
```

समस्याएँ:

- Exception-Based Flow
- Hidden Timeouts
- Missed Catch Blocks
- Unknown Transaction State

---

# After

```kotlin
sealed interface AuthorizationOutcome

Approved

Declined

TimedOut
```

फायदे:

- Explicit Outcomes
- Better Recovery
- Compile-Time Safety
- Clear Business Logic
- Predictable Behaviour

---

# मुख्य सीख (Key Takeaway)

> **Timeout कोई Programming Error नहीं है।**

वह आपके Business Domain का एक Valid Outcome है।

उसे Exception की तरह Throw करने के बजाय

एक Named Result के रूप में Return कीजिए।

---

# निष्कर्ष (Conclusion)

Distributed Systems में Timeout होना सामान्य बात है।

यदि Timeout को केवल Exception माना जाएगा,

तो Recovery Logic बिखर जाएगी और Bugs बढ़ेंगे।

लेकिन यदि Timeout को Domain Model का हिस्सा बनाया जाए,

तो आपका Code अधिक स्पष्ट, सुरक्षित और Maintainable बन जाता है।

> **Golden Rule:**  
> **Timeout is a Business Outcome, not just an Exception.**

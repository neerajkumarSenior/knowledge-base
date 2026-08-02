# Exhaustive `when` in Kotlin: Why You Should Avoid `else`

> **यदि आप `sealed class` या `sealed interface` के साथ `when` का उपयोग कर रहे हैं, तो `else` का उपयोग मत कीजिए। Compiler को सभी States Handle करवाने दीजिए।**

---

# 📌 समस्या (Problem)

Kotlin में `when` Expression का उपयोग अक्सर Business Logic लिखने के लिए किया जाता है।

उदाहरण:

```kotlin
sealed interface AuthorizationRequest

data class PurchaseRequest(...) : AuthorizationRequest

data class FullRefundRequest(...) : AuthorizationRequest
```

अब Business Logic:

```kotlin
fun authorize(
    request: AuthorizationRequest
): Authorization =
    when (request) {

        is PurchaseRequest ->
            authorizePurchase(request)

        is FullRefundRequest ->
            authorizeRefund(request)

        else ->
            error("Unsupported request")
    }
```

ऊपर का Code पहली नज़र में सही लगता है।

लेकिन इसमें एक गंभीर समस्या छिपी हुई है।

---

# ❌ इसमें समस्या क्या है?

मान लीजिए कुछ समय बाद Project में एक नई Request जोड़ दी गई।

```kotlin
data class PartialRefundRequest(...)
    : AuthorizationRequest
```

अब `when` Expression को Update करना चाहिए।

लेकिन क्योंकि पहले से `else` मौजूद है,

Compiler कोई Error नहीं देगा।

नई Request सीधे `else` में चली जाएगी।

```
PurchaseRequest
FullRefundRequest
PartialRefundRequest
        │
        ▼
else ❌
```

यानी Bug Runtime में मिलेगा।

---

# इसे "Graveyard Else" क्यों कहा जाता है?

`else` एक ऐसी जगह बन जाती है जहाँ भविष्य में आने वाली सभी नई States चुपचाप पहुँच जाती हैं।

इसलिए इसे अक्सर

> **Graveyard Else**

कहा जाता है।

नई States Handle नहीं होतीं,

बस `else` में गिर जाती हैं।

---

# ऐसा क्यों होता है?

क्योंकि `else` लिखने के बाद Compiler मान लेता है कि

> "बाकी सभी Cases Developer स्वयं Handle करेगा।"

इसलिए Compiler अब नई States की जाँच नहीं करता।

---

# वास्तविक उदाहरण

आज Application में केवल दो Requests हैं।

```
PurchaseRequest

FullRefundRequest
```

कुछ महीनों बाद

```
PartialRefundRequest
```

जोड़ दी गई।

लेकिन `when` नहीं बदला।

Application Compile हो जाएगी।

Bug सीधे Production में जाएगा।

---

# ✅ समाधान (Solution)

`else` को पूरी तरह हटा दीजिए।

```kotlin
fun authorize(
    request: AuthorizationRequest
): Authorization =
    when (request) {

        is PurchaseRequest ->
            authorizePurchase(request)

        is FullRefundRequest ->
            authorizeRefund(request)
    }
```

बस इतना ही।

---

# अब क्या होगा?

यदि बाद में कोई नई State जोड़ता है।

```kotlin
data class PartialRefundRequest(...)
    : AuthorizationRequest
```

तो Compiler तुरंत Error देगा।

```
'when' expression must be exhaustive
```

यानी

नई State Handle किए बिना Code Compile नहीं होगा।

---

# Compiler आपका Reviewer बन जाता है

पहले

```
Developer

↓

Code Review

↓

Production

↓

Bug
```

अब

```
Developer

↓

Compiler ❌

↓

Fix

↓

Production
```

Bug Production तक पहुँचने से पहले ही रुक जाता है।

---

# पहले

```kotlin
when(request) {

    is PurchaseRequest -> ...

    is FullRefundRequest -> ...

    else -> ...
}
```

नई State

↓

```
else
```

↓

Silent Bug ❌

---

# बाद में

```kotlin
when(request) {

    is PurchaseRequest -> ...

    is FullRefundRequest -> ...
}
```

नई State

↓

Compiler Error

↓

Developer Handle करेगा

↓

Safe Code ✅

---

# Sealed Classes की असली ताकत

मान लीजिए

```kotlin
sealed interface PaymentStatus
```

States

```text
Pending

Captured

Refunded
```

अब

```kotlin
when(status)
```

लिखते समय Compiler सुनिश्चित करेगा कि

- Pending
- Captured
- Refunded

तीनों Handle किए गए हों।

यदि बाद में

```text
Failed
```

State जोड़ दी गई,

तो Compiler तुरंत Error देगा।

---

# Real World Example

❌ गलत तरीका

```kotlin
when(order.status) {

    is Pending -> ...

    is Paid -> ...

    else -> ...
}
```

यदि बाद में

```text
Cancelled
```

State जोड़ दी गई,

तो वह `else` में चली जाएगी।

---

✅ सही तरीका

```kotlin
when(order.status) {

    is Pending -> ...

    is Paid -> ...

    is Cancelled -> ...
}
```

या यदि `Cancelled` बाद में जोड़ी जाती है,

तो Compiler स्वयं बताएगा कि इसे Handle करना बाकी है।

---

# इस Approach के लाभ

## ✅ 1. Compile-Time Safety

नई State Handle किए बिना Code Compile नहीं होगा।

---

## ✅ 2. Silent Bugs समाप्त

नई States चुपचाप `else` में नहीं जाएँगी।

---

## ✅ 3. Better Maintainability

Domain Model बदलते ही Compiler प्रभावित स्थानों की जानकारी दे देता है।

---

## ✅ 4. Safer Refactoring

नई State जोड़ते समय किसी Logic के छूटने की संभावना लगभग समाप्त हो जाती है।

---

## ✅ 5. Compiler Becomes Your Reviewer

Compiler स्वयं सुनिश्चित करता है कि कोई भी State छूट न जाए।

---

## ✅ 6. Cleaner Business Logic

कोई अनावश्यक `else` Branch नहीं रहती।

केवल Valid States दिखाई देती हैं।

---

## ✅ 7. Future-Proof Code

भविष्य में State बढ़ने पर भी Code सुरक्षित रहता है।

---

# यह Pattern कहाँ उपयोग करें?

- Payment Status
- Transaction Status
- Order Status
- Delivery Status
- Booking Status
- OTP Status
- Workflow Engine
- Subscription Status
- User Verification Status
- Ride Status
- Approval Process

---

# Before

```kotlin
when(state) {

    is Pending -> ...

    is Approved -> ...

    else -> ...
}
```

समस्याएँ:

- Silent Bugs
- Hidden Logic
- Compiler मदद नहीं करता
- नई States आसानी से छूट जाती हैं

---

# After

```kotlin
when(state) {

    is Pending -> ...

    is Approved -> ...

}
```

फायदे:

- Exhaustive Checking
- Compile-Time Safety
- Better Refactoring
- Safer Code
- No Hidden Bugs

---

# मुख्य सीख (Key Takeaway)

> **यदि `when` किसी `sealed class` या `sealed interface` पर लिखा गया है, तो `else` का उपयोग नहीं करना चाहिए।**

`else` हटाने से Compiler:

- सभी States Handle करवाता है।
- नई State जोड़ने पर Compile Error देता है।
- Hidden Bugs को रोकता है।
- Business Rules को सुरक्षित बनाता है।

---

# निष्कर्ष (Conclusion)

**`else` लिखना आसान है, लेकिन यह भविष्य के Bugs छिपा सकता है।**

जब आप `sealed class` या `sealed interface` के साथ `when` लिखते हैं, तब `else` को हटाइए और Compiler को आपकी सहायता करने दीजिए।

इससे आपका Code अधिक सुरक्षित, स्पष्ट, Maintainable और Future-Proof बनता है।

> **Golden Rule:**  
> **Sealed Class + `when` = No `else` ✅**

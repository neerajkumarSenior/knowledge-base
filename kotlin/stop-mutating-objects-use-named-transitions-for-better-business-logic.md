# Immutability + Named Transitions

> **Object की State को सीधे बदलने (Mutate) की अनुमति मत दीजिए। प्रत्येक State Change एक Named Business Transition के माध्यम से होनी चाहिए।**

---

# 📌 समस्या (Problem)

कई Applications में Business Objects Mutable होते हैं। अर्थात उनकी State को Project के किसी भी हिस्से से बदला जा सकता है।

उदाहरण:

```kotlin
data class Transaction(
    val id: String,
    var status: String,
    var amount: Long
)
```

अब Application में कहीं से भी कोई लिख सकता है:

```kotlin
transaction.status = "REFUNDED"
```

या

```kotlin
transaction.status = "REVERSED"
```

या

```kotlin
transaction.amount = 0
```

यानी Object की State बिना किसी नियंत्रण (Control) के बदल सकती है।

---

# ❌ इसमें समस्या क्या है?

मान लीजिए Business Rule है कि Refund तभी हो सकता है जब:

- Payment Captured हो।
- Refund Amount Valid हो।
- Transaction पहले से Refunded न हो।
- Transaction Reversed न हो।
- Payment Gateway से Refund Successful हो।

लेकिन यदि कोई सीधे लिख दे:

```kotlin
transaction.status = "REFUNDED"
```

तो इनमें से कोई भी Validation नहीं होगी।

परिणाम:

- Business Rules टूट जाते हैं।
- Invalid State Transition हो जाती है।
- Audit Information नहीं बनती।
- Bugs Production तक पहुँच जाते हैं।

---

# वास्तविक उदाहरण

Business Flow

```text
Captured
    │
    ▼
Refund Requested
    │
    ▼
Refund Completed
```

लेकिन Mutable Object होने पर कोई सीधे लिख सकता है:

```text
Captured
    │
    ▼
Refunded ❌
```

बीच की पूरी प्रक्रिया Skip हो गई।

---

# ऐसा क्यों होता है?

क्योंकि State Public है और कहीं से भी बदली जा सकती है।

Project में 100 Files हों,

तो किसी भी File में लिखा जा सकता है:

```kotlin
transaction.status = "REFUNDED"
```

अब यह पता लगाना कठिन हो जाता है कि State आखिर बदली कहाँ गई।

---

# ✅ समाधान (Solution)

Object को Immutable बनाइए।

```kotlin
@ConsistentCopyVisibility
data class PurchaseAuthorization private constructor(
    // properties
)
```

अब:

- Object बाहर से Create नहीं किया जा सकता।
- State सीधे Modify नहीं की जा सकती।
- प्रत्येक State Change Controlled होगी।

---

# Named Transition

State बदलने के लिए एक Business Method बनाई जाती है।

```kotlin
fun fullRefund(
    refundReason: RefundReason
): FullRefundOutcome
```

ध्यान दीजिए,

यह Method किसी Property को सीधे Change नहीं करती,

बल्कि एक Business Action को Represent करती है।

---

# पहले

```kotlin
transaction.status = "REFUNDED"
```

---

# बाद में

```kotlin
authorization.fullRefund(reason)
```

अब Refund करने का केवल एक ही अधिकृत (Authorized) तरीका है।

---

# Named Transition क्यों?

Method का नाम Business Language जैसा होना चाहिए।

उदाहरण:

Business Requirement

> Customer can request a Full Refund.

Code

```kotlin
authorization.fullRefund(reason)
```

Business Requirement

> Order can be Shipped.

Code

```kotlin
order.ship()
```

Business Requirement

> Payment can be Captured.

Code

```kotlin
payment.capture()
```

इससे Code और Business Specification लगभग एक जैसे दिखाई देते हैं।

---

# State Change के दौरान क्या होगा?

अब सारी Business Logic एक ही Method में होगी।

```text
fullRefund()
        │
        ▼
Status Validation
        │
        ▼
Refund Amount Validation
        │
        ▼
Fraud Checks
        │
        ▼
Payment Gateway Call
        │
        ▼
Audit Log
        │
        ▼
Database Update
        │
        ▼
Result
```

कोई Step भूलने की संभावना बहुत कम हो जाती है।

---

# Meaningful Return Type

Method केवल `Boolean` Return नहीं करती।

बल्कि एक Meaningful Result Return करती है।

```kotlin
sealed class FullRefundOutcome
```

उदाहरण:

```kotlin
sealed class FullRefundOutcome {

    data class Succeeded(
        val refundAuthorization: RefundAuthorization
    ) : FullRefundOutcome()

    data object AlreadyRefunded : FullRefundOutcome()

    data object NotAllowed : FullRefundOutcome()

    data object GatewayFailure : FullRefundOutcome()
}
```

अब Caller को स्पष्ट रूप से पता चलता है कि Refund का परिणाम क्या रहा।

---

# पहले

```kotlin
transaction.status = "REFUNDED"
```

कोई Validation नहीं।

कोई Audit नहीं।

कोई Business Rule नहीं।

---

# बाद में

```kotlin
authorization.fullRefund(reason)
```

↓

Business Validation

↓

Gateway Processing

↓

Audit Logging

↓

Database Update

↓

Meaningful Result

---

# इस Approach के लाभ

## ✅ 1. Immutability

Object की State बाहर से Modify नहीं की जा सकती।

---

## ✅ 2. Controlled State Changes

State केवल Approved Methods से बदलती है।

---

## ✅ 3. Business Rules एक स्थान पर रहती हैं

Validation अलग-अलग Files में बिखरती नहीं।

---

## ✅ 4. Code Business Language जैसा दिखता है

```kotlin
payment.capture()

payment.refund()

order.ship()

order.cancel()

subscription.pause()
```

इन Methods को देखकर तुरंत समझ आता है कि Business में क्या हो रहा है।

---

## ✅ 5. बेहतर Audit

हर State Change एक Method के माध्यम से होती है।

इसलिए Tracking और Logging आसान हो जाती है।

---

## ✅ 6. Runtime Bugs कम होते हैं

गलत State सीधे Assign नहीं की जा सकती।

---

## ✅ 7. बेहतर Maintainability

यदि Business Rule बदलती है,

तो केवल Transition Method को Update करना होता है।

पूरे Project में Changes करने की आवश्यकता नहीं पड़ती।

---

# यह Pattern कहाँ उपयोग करें?

यह Pattern लगभग हर Domain में उपयोगी है।

- Payment Processing
- Banking Systems
- Order Management
- Booking Systems
- Ride Applications
- Subscription Platforms
- Invoice Systems
- Approval Workflows
- Inventory Management
- Logistics Systems

---

# Before

```kotlin
transaction.status = "REFUNDED"
```

समस्याएँ:

- Mutable State
- Business Rules Bypass
- Validation Missing
- Audit Missing
- Hidden Bugs

---

# After

```kotlin
authorization.fullRefund(reason)
```

फायदे:

- Controlled Transition
- Business Validation
- Better Audit
- Clear Intent
- Safer Code
- Immutable Objects

---

# मुख्य सीख (Key Takeaway)

> **State को सीधे बदलने की अनुमति मत दीजिए।**

हर State Change को एक **Named Transition Method** के माध्यम से कीजिए।

जैसे:

```kotlin
authorize()

capture()

fullRefund()

partialRefund()

reverse()

cancel()

expire()

ship()

deliver()
```

इन Methods के माध्यम से आपका Code Business Process को स्पष्ट रूप से दर्शाता है और Invalid State Changes को रोकता है।

---

# निष्कर्ष (Conclusion)

**Immutability + Named Transitions** Domain-Driven Design (DDD) और Clean Architecture की एक महत्वपूर्ण तकनीक है।

जब Object Immutable होता है और State केवल Named Business Methods के माध्यम से बदलती है, तब:

- ✔ Business Rules सुरक्षित रहती हैं।
- ✔ Invalid State Transitions रुक जाती हैं।
- ✔ Code अधिक स्पष्ट और Maintainable बनता है।
- ✔ Debugging और Security Audit आसान हो जाते हैं।
- ✔ Compiler और Domain Model मिलकर Bugs को Production तक पहुँचने से पहले ही रोकने में सहायता करते हैं।

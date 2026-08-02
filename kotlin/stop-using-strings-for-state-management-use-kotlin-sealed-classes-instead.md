# Sealed Hierarchies for State Machines

> **Application की State को साधारण `String` की तरह Store मत कीजिए। उसे `Sealed Class` या `Sealed Interface` के माध्यम से Model कीजिए।**

---

# 📌 समस्या (Problem)

अधिकांश Applications में किसी Entity की State को `String` के रूप में Store किया जाता है।

उदाहरण:

- Payment Status
- Order Status
- Transaction Status
- OTP Status
- Subscription Status
- Booking Status

एक सामान्य Implementation कुछ इस प्रकार होती है:

```kotlin
data class Transaction(
    val id: String,
    val status: String,
    val amount: Long
)
```

Business Logic:

```kotlin
fun refund(tx: Transaction, amount: Long) {
    if (amount <= tx.amount) {
        issue(tx.id, amount)
    }
}
```

---

# ❌ इसमें समस्या क्या है?

ऊपर दिए गए Code में केवल Refund Amount की जाँच की गई है।

लेकिन यह नहीं देखा गया कि Transaction किस State में है।

मान लीजिए Transaction पहले से ही Reverse हो चुका है।

```text
Status = REVERSED
```

फिर भी Refund हो जाएगा।

```
REVERSED
      │
      ▼
Refund Again ❌
```

इसे **Invalid State Transition** कहते हैं।

यानी Business Rules का उल्लंघन हो गया।

---

# ऐसा क्यों होता है?

क्योंकि `status` केवल एक `String` है।

Compiler के लिए ये सभी Values समान हैं।

```kotlin
status = "CAPTURED"

status = "captured"

status = "Refunded"

status = "Done"

status = "Anything"
```

Compiler यह नहीं जानता कि इनमें से कौन-सी Value सही है और कौन-सी गलत।

सारी Validation Developer के भरोसे छोड़ दी जाती है।

---

# दूसरी समस्या

पूरा Business Logic String Comparison पर निर्भर हो जाता है।

```kotlin
if (status == "CAPTURED") {
    ...
}
```

या

```kotlin
when(status) {

    "CAPTURED" -> ...

    "REFUNDED" -> ...

    "REVERSED" -> ...
}
```

इस Approach की समस्याएँ:

- Typing Mistakes होने की संभावना
- Invalid Values Store हो सकती हैं
- Runtime Bugs
- नई State Handle करना भूल सकते हैं
- Compiler कोई सहायता नहीं करता

---

# ✅ समाधान (Solution)

हर Valid State के लिए एक अलग Type बनाईए।

```kotlin
sealed interface TransactionStatus
```

अब प्रत्येक State अपनी अलग Class होगी।

```kotlin
data class Captured(
    val amount: AmountMinor
) : TransactionStatus

data class FullyRefunded(
    val refundedAt: Instant
) : TransactionStatus

data class Reversed(
    val reason: ReversalReason
) : TransactionStatus
```

अब केवल यही तीन States अस्तित्व में हो सकती हैं।

कोई Invalid State बन ही नहीं सकती।

---

# प्रत्येक State अपना Data साथ रखती है

हर State केवल वही जानकारी रखती है जिसकी उसे आवश्यकता है।

### Captured

```text
Captured
└── amount
```

### Fully Refunded

```text
FullyRefunded
└── refundedAt
```

### Reversed

```text
Reversed
└── reason
```

इससे Domain Model अधिक स्पष्ट और Meaningful बन जाता है।

---

# Business Logic

अब Refund Logic कुछ इस प्रकार होगा:

```kotlin
fun TransactionStatus.fullRefund(
    reason: RefundReason
) = when(this) {

    is Captured ->
        Ok()

    is FullyRefunded ->
        Error("Already refunded")

    is Reversed ->
        Error("Cannot refund a reversed transaction")
}
```

ध्यान दीजिए कि अब कहीं भी String Comparison नहीं है।

---

# Compiler सभी States Handle करवाता है

मान लीजिए बाद में एक नई State जोड़ दी गई।

```kotlin
data object Pending : TransactionStatus
```

अब Compiler Error देगा।

क्यों?

क्योंकि जहाँ-जहाँ

```kotlin
when(status)
```

लिखा गया है, वहाँ `Pending` को Handle नहीं किया गया।

जब तक नई State Handle नहीं होगी, Code Compile नहीं होगा।

इसे **Exhaustive Pattern Matching** कहते हैं।

---

# पहले

```kotlin
when(status) {

    "CAPTURED" -> ...

    "REFUNDED" -> ...
}
```

यदि `"REVERSED"` Handle करना भूल गए,

तो Compiler कुछ नहीं कहेगा।

Bug सीधे Production में पहुँच जाएगा।

---

# बाद में

```kotlin
when(status) {

    is Captured -> ...

    is FullyRefunded -> ...

    is Reversed -> ...
}
```

यदि नई State जोड़ दी,

तो Compiler तुरंत Error देगा।

---

# इस Approach के लाभ

## ✅ 1. Compile-Time Safety

केवल Valid States ही बन सकती हैं।

---

## ✅ 2. Invalid Strings समाप्त

अब कोई गलती से नहीं लिख सकता:

```text
captured
done
refund
abc
processing
```

---

## ✅ 3. Invalid State Transition रुक जाती है

उदाहरण:

```
Reversed
     │
     ▼
Refund Again ❌
```

ऐसी गलतियाँ आसानी से रोकी जा सकती हैं।

---

## ✅ 4. प्रत्येक State अपना आवश्यक Data रखती है

कोई अनावश्यक या Nullable Fields नहीं रहतीं।

---

## ✅ 5. Code अधिक पढ़ने योग्य बनता है

पहले:

```kotlin
status == "CAPTURED"
```

अब:

```kotlin
is Captured
```

Intent तुरंत समझ में आता है।

---

## ✅ 6. Maintenance आसान हो जाती है

यदि नई State जोड़ते हैं,

तो Compiler स्वयं बता देता है कि किन-किन जगहों पर बदलाव करना होगा।

कोई Hidden Bug नहीं रहता।

---

## ✅ 7. Domain Model अधिक मजबूत बनता है

अब आपका Code केवल Data Store नहीं करता,

बल्कि वास्तविक Business Rules को भी Represent करता है।

---

# यह Pattern कहाँ उपयोग करें?

यह Pattern उन सभी जगहों पर उपयोगी है जहाँ State Machine होती है।

- Payment Status
- Order Status
- Transaction Status
- Delivery Status
- Ride Status
- Booking Status
- Subscription Status
- Invoice Status
- OTP Verification Status
- User Verification Status
- Workflow Engine

---

# पहले

```kotlin
data class Order(

    val status: String

)
```

समस्याएँ:

- Runtime Validation
- Invalid Values
- String Comparison
- Missing States
- Production Bugs

---

# बाद में

```kotlin
sealed interface OrderStatus

data object Pending : OrderStatus

data object Paid : OrderStatus

data object Shipped : OrderStatus

data object Delivered : OrderStatus

data class Cancelled(
    val reason: String
) : OrderStatus
```

अब प्रत्येक State Type System का हिस्सा बन जाती है।

---

# मुख्य सीख (Key Takeaway)

> **State केवल एक String नहीं होती, बल्कि आपके Domain Model का महत्वपूर्ण भाग होती है।**

इसे `Sealed Class` या `Sealed Interface` के माध्यम से Model करने से:

- ✔ Compile-Time Validation मिलती है।
- ✔ Invalid States नहीं बन सकतीं।
- ✔ Compiler सभी States Handle करवाता है।
- ✔ Business Rules अधिक सुरक्षित बनते हैं।
- ✔ Runtime Bugs कम हो जाते हैं।
- ✔ Code अधिक स्पष्ट, सुरक्षित और Maintainable बनता है।

---

# निष्कर्ष (Conclusion)

**Business Rules को केवल `if` और String Comparison में मत लिखिए।**

उन्हें **Type System** का हिस्सा बनाइए।

जब प्रत्येक State का अपना Type होता है, तब Compiler स्वयं आपकी सहायता करता है और Invalid State Transitions को Production में पहुँचने से पहले ही रोक देता है।

# Compliance in the Type System

> **Don't represent sensitive data as `String`. Represent it as a dedicated type.**

## 📌 Problem

कई Applications में Sensitive Data जैसे:

- Credit Card PAN
- CVV
- Aadhaar Number
- Password
- API Key
- JWT Token
- Bank Account Number

को सीधे `String` के रूप में Store किया जाता है।

### Example

```kotlin
data class Card(
    val pan: String,
    val expiry: YearMonth
)

logger.info("Processing card $card")

throw PaymentException("Invalid card: $pan")
```

### ❌ Problem

ऊपर वाले Code में `pan` केवल एक `String` है।

अब अगर कोई Developer गलती से Logging कर दे:

```kotlin
logger.info(card)
```

तो Output कुछ ऐसा होगा:

```text
Processing card Card(
    pan=4111111111111111,
    expiry=2028-12
)
```

यानी पूरा Card Number Log में चला गया।

इसी तरह यह Sensitive Data पहुँच सकता है:

- Log Files
- Stack Trace
- Error Messages
- Monitoring Systems
- Crash Reports

यह Security Risk भी है और Payment Systems में PCI DSS Compliance का Violation भी हो सकता है।

---

# 🤔 Root Cause

समस्या Logging नहीं है।

समस्या यह है कि Compiler को पता ही नहीं है कि यह `String` एक Sensitive Value है।

Compiler के लिए ये दोनों एक जैसे हैं:

```kotlin
val username: String
val pan: String
```

इसलिए गलती से PAN भी Print हो सकता है।

---

# ✅ Solution

Sensitive Data के लिए एक Dedicated Type (Value Object) बनाइए।

```kotlin
@JvmInline
value class Pan private constructor(
    private val value: String
) {

    override fun toString(): String = "PAN[****]"

    fun masked(): String =
        "************" + value.takeLast(4)

    fun raw(): String = value
}
```

अब Model कुछ ऐसा होगा:

```kotlin
data class Card(
    val pan: Pan,
    val expiry: YearMonth
)
```

---

# Logging

अब यदि कोई लिखता है:

```kotlin
logger.info(card)
```

Output

```text
Card(
    pan=PAN[****]
)
```

अब Real PAN कभी Log में नहीं जाएगा।

---

# जब वास्तव में PAN चाहिए

Payment Gateway को PAN भेजना है।

```kotlin
paymentGateway.pay(
    pan.raw()
)
```

अब Developer को Explicit Method Call करनी पड़ेगी।

इससे Sensitive Data का Access Intentional हो जाता है।

---

# Benefits

## 1. Accidental Logging रुक जाती है

गलती से भी पूरा PAN Log में नहीं जाएगा।

---

## 2. Type Safety

पहले

```kotlin
fun pay(pan: String)
```

कोई भी String Pass कर सकता था।

```kotlin
pay("Hello")
```

Compile हो जाता।

अब

```kotlin
fun pay(pan: Pan)
```

सिर्फ `Pan` Object ही Accept होगा।

---

## 3. Better Code Readability

पहले

```kotlin
String
```

देखकर पता नहीं चलता था कि उसमें क्या है।

अब

```kotlin
Pan
```

देखते ही समझ आ जाता है कि यह Credit Card Number है।

---

## 4. Easy Security Audit

जहाँ भी Real PAN Access किया गया होगा

```kotlin
pan.raw()
```

बस Project में Search करो

```
raw()
```

या

```
Pan
```

और सभी Sensitive Access Points मिल जाएंगे।

---

## 5. Compliance Friendly

यह Approach कई Security Standards के लिए अच्छी Practice मानी जाती है।

- PCI DSS
- GDPR
- ISO 27001
- SOC 2

---

# Real World Example

❌ Bad

```kotlin
data class Payment(
    val cardNumber: String,
    val cvv: String
)
```

गलती से

```kotlin
logger.info(payment)
```

तो पूरा Data Leak हो सकता है।

---

✅ Good

```kotlin
@JvmInline
value class CardNumber(private val value: String) {

    override fun toString(): String =
        "CardNumber(****)"

    fun raw() = value
}

@JvmInline
value class CVV(private val value: String) {

    override fun toString(): String =
        "***"

    fun raw() = value
}
```

अब

```kotlin
logger.info(payment)
```

Output

```text
Payment(
    card=CardNumber(****),
    cvv=***
)
```

---

# Use This Pattern For

- Credit Card Number
- CVV
- Aadhaar Number
- Bank Account Number
- IFSC
- UPI ID
- Password
- API Key
- JWT Token
- Refresh Token
- Access Token
- Secret Key
- OTP

---

# Key Takeaway

> **Sensitive data should never be modeled as a plain `String`.**

Instead, create a dedicated Value Object that:

- Masks itself by default
- Prevents accidental logging
- Exposes raw value only through an explicit method
- Improves type safety
- Makes security audits easier
- Helps achieve compliance requirements

---

## Before

```kotlin
val pan: String
```

❌ Just another String.

---

## After

```kotlin
val pan: Pan
```

✅ A secure, self-protecting type.

---

# Conclusion

**Move security from documentation to the type system.**

अगर Compiler को पता होगा कि कौन-सा Data Sensitive है, तो वह Developer की गलतियों से होने वाले Data Leaks को काफी हद तक रोक सकता है।

यही **Compliance in the Type System** का मूल विचार है।

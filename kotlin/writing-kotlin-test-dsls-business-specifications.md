# DSLs that Read Like Specifications

> **Tests ऐसे लिखिए कि वे केवल Developers ही नहीं, बल्कि Business Analysts, QA Engineers और Domain Experts भी आसानी से पढ़ और समझ सकें।**

---

# 📌 समस्या (Problem)

कई Projects में Unit Tests या Integration Tests बहुत जटिल (Complex) होते हैं।

एक साधारण Business Scenario को Test करने के लिए सैकड़ों Lines का Setup लिखना पड़ता है।

उदाहरण:

```kotlin
val card = Card(
    pan = Pan.parse("4111...")
)

val terminal = Terminal(
    capabilities = setOf(...)
)

val transaction = Transaction(
    ...
)

val result = runEmv(card, terminal, transaction)
```

कई बार केवल Object Create करने में ही 100–200 Lines का Code लिखना पड़ता है।

---

# ❌ इसमें समस्या क्या है?

Test पढ़कर यह समझना मुश्किल हो जाता है कि वास्तव में Business Rule क्या Verify किया जा रहा है।

Developer को पहले पूरा Setup समझना पड़ता है।

```
Card

↓

Terminal

↓

Transaction

↓

Configuration

↓

Result
```

Business Logic Code के अंदर छिप जाती है।

---

# वास्तविक उदाहरण

मान लीजिए Business Rule है:

> **"Floor Limit से अधिक Amount होने पर Contactless Payment Online Authorization के लिए जाएगी।"**

लेकिन Test कुछ ऐसा दिखता है:

```kotlin
val card = ...
val terminal = ...
val transaction = ...
val config = ...
val result = ...
```

इतना Setup देखकर यह समझना कठिन है कि Test वास्तव में किस Rule को Verify कर रहा है।

---

# ऐसा क्यों होता है?

क्योंकि Test Infrastructure और Business Logic एक साथ लिखे जाते हैं।

Business Rule Code के शोर (Boilerplate) में खो जाती है।

---

# ✅ समाधान (Solution)

ऐसा DSL (Domain Specific Language) बनाइए जो Business Specification की तरह पढ़ा जा सके।

उदाहरण:

```kotlin
scenario(
    "Purchase with tap card above floor limit must go online"
) {

    terminal {
        floorLimit = 50.EUR
    }

    posRequest {
        purchase(75.EUR)
    }

    tap {
        card = TestCards.visaWithOnlinePIN
    }

    expect {
        terminalDecision mustBe GoOnline
        outcome mustBe Approved
    }
}
```

अब Test पढ़ते ही Business Rule स्पष्ट दिखाई देती है।

---

# DSL क्या है?

DSL (Domain Specific Language) ऐसी भाषा या API होती है जो किसी विशेष Domain की भाषा में Code लिखने की सुविधा देती है।

उदाहरण:

```kotlin
order.ship()

payment.capture()

subscription.pause()
```

ये Methods Business Language जैसी लगती हैं।

---

# Test DSL का उद्देश्य

Test ऐसा होना चाहिए कि

- Developer
- QA Engineer
- Business Analyst
- Product Owner

सभी उसे आसानी से पढ़ सकें।

---

# पहले

```kotlin
val card = ...

val terminal = ...

val request = ...

val transaction = ...

val result = ...
```

क्या Test हो रहा है?

समझना कठिन।

---

# बाद में

```kotlin
scenario("Purchase above floor limit") {

    purchase(75.EUR)

    expect {
        outcome mustBe Approved
    }
}
```

अब उद्देश्य तुरंत समझ में आता है।

---

# Specification जैसी Readability

Business Requirement

```
Purchase above Floor Limit

↓

Go Online

↓

Approved
```

DSL

```kotlin
scenario {

    purchase(75.EUR)

    expect {
        outcome mustBe Approved
    }
}
```

Code और Requirement लगभग समान दिखाई देते हैं।

---

# इस Approach के लाभ

## ✅ 1. Readability

Test पढ़ना आसान हो जाता है।

---

## ✅ 2. Business Friendly

Non-Developers भी Test समझ सकते हैं।

---

## ✅ 3. कम Boilerplate Code

बार-बार Object Construction नहीं करना पड़ता।

---

## ✅ 4. Maintainability

Infrastructure बदलने पर केवल DSL Update करनी होती है।

Tests नहीं बदलते।

---

## ✅ 5. Better Communication

QA, Product Team और Developers एक ही भाषा में बात कर सकते हैं।

---

## ✅ 6. Reusable Test Components

Common Setup एक जगह Define किया जा सकता है।

---

## ✅ 7. Documentation जैसा Test

Tests Documentation की तरह पढ़े जा सकते हैं।

---

# कहाँ उपयोग करें?

- Payment Systems
- Banking Software
- Order Management
- Healthcare Applications
- Insurance Platforms
- Workflow Engines
- Financial Systems
- Rule Engines
- Complex Business Applications

---

# Before

```text
200 Lines of Setup

↓

Business Rule Hidden

↓

Hard to Understand
```

---

# After

```text
Scenario

↓

Business Language

↓

Readable Test

↓

Easy Maintenance
```

---

# मुख्य सीख (Key Takeaway)

> **Tests केवल मशीन के लिए नहीं, बल्कि इंसानों के लिए भी लिखे जाते हैं।**

यदि Test पढ़ने के लिए पहले 200 Lines का Setup समझना पड़े,

तो Test अच्छी नहीं है।

एक अच्छी Test DSL Business Specification की तरह पढ़ी जानी चाहिए।

---

# निष्कर्ष (Conclusion)

DSL (Domain Specific Language) का उपयोग करके आप Tests को अधिक स्पष्ट, संक्षिप्त और Business-Oriented बना सकते हैं।

जब Tests Business Specification की तरह पढ़ी जाती हैं, तब:

- ✔ Readability बढ़ती है।
- ✔ Boilerplate कम होता है।
- ✔ QA और Business Team भी Tests समझ सकती है।
- ✔ Maintenance आसान हो जाती है।
- ✔ Tests Documentation का कार्य भी करने लगती हैं।

> **Golden Rule:**  
> **A good test should read like a business specification, not an object construction script.**

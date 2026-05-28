# Modular Monolith में Multiple Tables Insert और Transactions Handling

# Problem Statement

जब हमें:

* एक साथ कई tables में data insert करना हो
* और यह ensure करना हो कि:

  * या तो सब save हो जाए
  * या कुछ भी save न हो

तब हमें चाहिए:

# Database Transactions (ACID)

---

# सबसे Important Rule

> “Half data save नहीं होना चाहिए।”

मतलब:

❌ Order save हो गया लेकिन items नहीं हुए
❌ Payment save हो गया लेकिन subscription नहीं बना

ऐसा कभी नहीं होना चाहिए।

---

# Transaction क्या करता है?

Transaction database को कहता है:

> “सारा काम एक package में करो।”

अगर बीच में कुछ भी fail हुआ:

* सब rollback
* database पुरानी state में वापस

---

# Two Main Cases

| Case                    | Complexity              |
| ----------------------- | ----------------------- |
| Same Module Tables      | Easy ✅                  |
| Different Module Tables | Careful Architecture ⚠️ |

---

# CASE 1:

# Multiple Tables SAME MODULE में

यह सबसे simple और recommended case है।

---

# Example

## Orders Module

Tables:

* orders
* order_meals
* order_extras

---

# Goal

जब नया order बने:

* orders table में entry हो
* order_meals में items save हों
* order_extras में extras save हों

और अगर कोई भी step fail हो जाए:

* कुछ भी save न हो

---

# Solution:

# Database Transaction ✅

---

# AdonisJS Example

```ts id="txa11"
import db from '@adonisjs/lucid/services/db'

async function createOrder(orderData, meals) {

  const trx = await db.transaction()

  try {

    // Create Order
    const order = new Order()
    order.fill(orderData)

    order.useTransaction(trx)

    await order.save()

    // Create Meals
    for (const meal of meals) {

      const orderMeal = new OrderMeal()

      orderMeal.fill({
        orderId: order.id,
        ...meal
      })

      orderMeal.useTransaction(trx)

      await orderMeal.save()
    }

    // SUCCESS
    await trx.commit()

    return order

  } catch (error) {

    // FAILURE
    await trx.rollback()

    throw error
  }
}
```

---

# Flow Understanding

```text
Start Transaction
        ↓
Insert Order
        ↓
Insert Meals
        ↓
Everything Success?
        ↓
YES → Commit
NO  → Rollback
```

---

# Benefits

| Benefit          | Why Important                 |
| ---------------- | ----------------------------- |
| Data Consistency | Half data save नहीं होगा      |
| Safe Operations  | Crash होने पर corruption नहीं |
| Easy to Manage   | Single transaction            |
| Fast             | Single DB connection          |

---

# Same Module Rule

> “Same Module = Direct Transaction Allowed”

---

# CASE 2:

# Multiple Tables DIFFERENT MODULES में

यहाँ architecture important हो जाता है।

---

# Example

## Subscription Module

Table:

* subscriptions

## Payments Module

Table:

* payments

---

# Problem

जब user subscription खरीदे:

* subscription भी बने
* payment record भी बने

लेकिन:

❌ Subscription module सीधे Payments table को touch नहीं करेगा

क्योंकि:

* Modules tightly coupled हो जाएंगे
* Architecture टूट जाएगा

---

# Solution Approaches

| Approach                  | Recommended For     |
| ------------------------- | ------------------- |
| Transaction Sharing       | Small/Medium Apps   |
| Event Driven Architecture | Large/Scalable Apps |

---

# APPROACH A:

# Transaction को Service के पार Pass करना

यह Modular Monolith में बहुत common और practical तरीका है।

---

# Architecture

```text
Subscription Service
        ↓
Payment Service
        ↓
Same Transaction Shared
```

---

# AdonisJS Example

## Subscription Service

```ts id="txb22"
import db from '@adonisjs/lucid/services/db'

export class SubscriptionService {

  async buySubscription(subData, paymentData) {

    const trx = await db.transaction()

    try {

      // Create Subscription
      const sub = new Subscription()

      sub.fill(subData)

      sub.useTransaction(trx)

      await sub.save()

      // Call Payment Module
      await PaymentService.createPayment(
        paymentData,
        trx
      )

      await trx.commit()

      return sub

    } catch (error) {

      await trx.rollback()

      throw error
    }
  }
}
```

---

# Payment Service

```ts id="txc33"
export class PaymentService {

  async createPayment(data, trx) {

    const payment = new Payment()

    payment.fill(data)

    payment.useTransaction(trx)

    await payment.save()
  }
}
```

---

# Important Point

यहाँ:

✅ Modules अलग हैं
✅ Services अलग हैं
✅ Business logic अलग है

लेकिन:

✅ Transaction shared है

---

# यह तरीका कब सही है?

| Condition        | Suitable? |
| ---------------- | --------- |
| Single Database  | ✅ YES     |
| Modular Monolith | ✅ YES     |
| Small Team       | ✅ YES     |
| Fast Development | ✅ YES     |

---

# Drawback

अगर Payment Service crash हो जाए:

* पूरा transaction rollback होगा

यह अच्छा भी है और limitation भी।

---

# APPROACH B:

# Event Driven Architecture (Professional Way) 🌟

यह बड़े systems में use होता है।

---

# Core Idea

Modules एक-दूसरे को direct call नहीं करते।

वे सिर्फ events emit करते हैं।

---

# Example Flow

```text
User Buys Subscription
        ↓
Subscription Module
        ↓
Save subscription (pending_payment)
        ↓
Emit Event:
subscription:created
        ↓
Payments Module listens
        ↓
Create Payment
        ↓
Payment Success
        ↓
Emit Event:
payment:success
        ↓
Subscription Module listens
        ↓
Update Status → active
```

---

# Visual Flow

```text
[Subscription Module]
        │
        ├── emit(subscription:created)
        │
        ▼
[Payments Module]
        │
        ├── payment success
        │
        ├── emit(payment:success)
        │
        ▼
[Subscription Module]
```

---

# Benefits

| Benefit        | Why Important                               |
| -------------- | ------------------------------------------- |
| Loose Coupling | Modules independent रहते हैं                |
| Scalable       | Future microservices ready                  |
| Fault Tolerant | One module fail होने पर पूरा app नहीं टूटता |
| Async Friendly | Background jobs possible                    |

---

# Drawbacks

| Drawback             | Reason                        |
| -------------------- | ----------------------------- |
| More Complex         | Events manage करने पड़ते हैं  |
| Hard Debugging       | Flow distributed हो जाता है   |
| Eventual Consistency | Data instantly sync नहीं होता |

---

# Real Industry Usage

| Company Style  | Approach           |
| -------------- | ------------------ |
| Small SaaS     | Shared Transaction |
| Medium Startup | Mixed              |
| Enterprise     | Event Driven       |
| Microservices  | Event Driven       |

---

# Your Best Starting Point

यदि:

* Single Database है
* Modular Monolith है
* Team छोटी है
* Fast development चाहिए

तो:

# ✅ Approach A Best है

यानी:

> “Transaction object को services के बीच pass करो।”

---

# Golden Rule

| Situation                   | Best Practice      |
| --------------------------- | ------------------ |
| Same Module Tables          | Direct Transaction |
| Different Modules + Same DB | Shared Transaction |
| Fully Independent Modules   | Events             |
| Microservices               | Events + Queues    |

---

# Most Important Understanding

> “Modules अलग हो सकते हैं,
> लेकिन transaction same हो सकता है।”

---

# Final Architecture Thinking

## Same Module

```text
Orders Module
 ├── orders
 ├── order_meals
 └── order_extras
```

✅ Direct JOIN
✅ Direct Transaction

---

## Different Modules

```text
Subscription Module
        ↕
Payment Module
```

⚠️ Direct table access avoid
✅ Service communication
✅ Shared transaction OR events

---

# Final One-Line Rule

> “Database transaction data consistency संभालता है,
> जबकि module boundaries architecture consistency संभालती हैं।”

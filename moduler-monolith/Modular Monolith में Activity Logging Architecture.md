# Modular Monolith में Activity Logging Architecture

# Problem Statement

जब application में:

* User login करे
* Order place हो
* Meal skip हो
* Payment success हो
* Subscription activate हो

तो हमें उनका Activity Log रखना होता है।

---

# सबसे बड़ा सवाल

> “क्या हर module अपना अलग logging system बनाएगा?”

या

> “पूरा application एक central logging system use करेगा?”

---

# Correct Architecture ✅

# Single Shared Logging Module

Modular Monolith में:

✅ एक central logging module बनाया जाता है।

उदाहरण:

```text id="2zbx4m"
app/modules/logs
```

या

```text id="2t3g9q"
app/shared/logging
```

---

# ऐसा क्यों?

क्योंकि:

* हर module duplicate logging code नहीं लिखेगा
* सारे logs same format में store होंगे
* Admin dashboard बनाना आसान होगा
* Future scaling आसान होगी

---

# Recommended Architecture

```text id="5y7w0x"
app/modules
│
├── auth
├── users
├── orders
├── subscriptions
├── payments
└── logs
```

---

# Golden Rule

> “कोई भी module सीधे activity_logs table को touch नहीं करेगा।”

सिर्फ:

# Logs Module

ही logging database operations करेगा।

---

# Logging Flow (Event Driven)

```text id="zv6q1d"
Any Module
     │
     ├── emit event
     │
     ▼
Event Emitter
     │
     ▼
Logs Listener
     │
     ▼
activity_logs table
```

---

# Step 1:

# Module Event Emit करेगा

उदाहरण:

Subscription module में user meal skip करता है।

---

# WRONG WAY ❌

```ts id="tw4t9l"
await db.table('activity_logs').insert(...)
```

यह गलत है क्योंकि:

* Module logs table के बारे में जानने लगा
* Tight coupling हो गया

---

# RIGHT WAY ✅

```ts id="r7q8kx"
import emitter from '@adonisjs/core/services/emitter'

export class SubscriptionService {

  async skipMeal(userId: number, date: string) {

    // Main business logic
    // meal skipped

    // Emit logging event
    emitter.emit('activity:log', {
      userId,
      module: 'subscriptions',
      action: 'MEAL_SKIPPED',
      description: `Meal skipped for ${date}`,
      newValue: {
        date,
        status: 'skipped'
      }
    })
  }
}
```

---

# Important Understanding

यहाँ:

✅ Subscription module को यह नहीं पता:

* logs table कहाँ है
* logging कैसे हो रही है
* कौन सा database use हो रहा है

उसका काम सिर्फ:

# Event emit करना है

---

# Step 2:

# Logs Module Event Listen करेगा

---

# Activity Listener

```ts id="v0e4sa"
export default class ActivityListener {

  async handle(payload: any) {

    await db.table('activity_logs').insert({

      user_id: payload.userId,

      module: payload.module,

      action: payload.action,

      description: payload.description,

      new_value: JSON.stringify(payload.newValue),

      created_at: new Date()
    })
  }
}
```

---

# Final Flow

```text id="6v4epu"
User Action
     ↓
Subscription Module
     ↓
Event Emit
     ↓
Activity Listener
     ↓
activity_logs Table
```

---

# Why This Architecture is Powerful

# 1. Loose Coupling ✅

Orders module को logs database structure नहीं पता।

Subscriptions module को logs table नहीं पता।

सब independent हैं।

---

# 2. Clean Architecture ✅

हर module सिर्फ अपना काम करता है।

| Module        | Responsibility     |
| ------------- | ------------------ |
| Orders        | Order logic        |
| Payments      | Payment logic      |
| Subscriptions | Subscription logic |
| Logs          | Logging            |

---

# 3. Better Performance ✅

User request fast complete हो जाती है।

Logging background में हो सकती है।

---

# 4. Future Proof ✅

आज:

```text id="1kz0ro"
activity_logs table
```

कल:

```text id="jlwmvw"
Elasticsearch
Kafka
CloudWatch
Logstash
```

सिर्फ listener बदलना होगा।

बाकी modules untouched रहेंगे।

---

# 5. Microservice Ready ✅

आज:

* Modular Monolith

कल:

* Microservices

Migration आसान हो जाएगी।

क्योंकि architecture पहले से event-driven है।

---

# Recommended activity_logs Table

```sql id="n1fdw2"
CREATE TABLE activity_logs (

  id BIGINT PRIMARY KEY,

  user_id BIGINT NULL,

  module VARCHAR(50),

  action VARCHAR(100),

  entity_type VARCHAR(100),

  entity_id BIGINT NULL,

  description TEXT,

  old_value JSON NULL,

  new_value JSON NULL,

  ip_address VARCHAR(100) NULL,

  user_agent TEXT NULL,

  created_at TIMESTAMP
);
```

---

# Recommended Log Payload Structure

```ts id="9kw0dr"
{
  userId: 1,

  module: 'orders',

  action: 'ORDER_CREATED',

  entityType: 'order',

  entityId: 55,

  description: 'New order created',

  oldValue: null,

  newValue: {
    total: 250
  }
}
```

---

# Common Activities to Log

| Action          | Log? |
| --------------- | ---- |
| Login           | ✅    |
| Logout          | ✅    |
| Order Created   | ✅    |
| Payment Success | ✅    |
| Meal Skip       | ✅    |
| Profile Update  | ✅    |
| Role Change     | ✅    |
| Password Change | ✅    |
| Failed Login    | ✅    |

---

# What NOT to Log

| Avoid             | Reason            |
| ----------------- | ----------------- |
| Raw Passwords     | Security Risk     |
| OTP Codes         | Sensitive         |
| Full Card Details | Compliance Issues |
| Secrets/Tokens    | Dangerous         |

---

# Best Practice

## Emit Business Events

```text id="dc0kgj"
order:created
payment:success
subscription:paused
meal:skipped
user:logged_in
```

---

# Then Logs Module decides:

* क्या store करना है
* कहाँ store करना है
* किस format में store करना है

---

# Final Architecture Philosophy

> “Business modules should not care about logging implementation.”

---

# One-Line Golden Rule

> “Modules emit events,
> Logs module records history.”

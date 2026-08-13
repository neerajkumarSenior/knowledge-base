# Eventual Consistency — Problems & Solutions

## 1. Overview

**Eventual Consistency** distributed systems में एक consistency model है जिसमें data update होने के बाद सभी services, servers या replicas पर बदलाव **तुरंत दिखाई देना जरूरी नहीं होता**।

कुछ समय के लिए अलग-अलग systems में अलग data हो सकता है, लेकिन यदि नए updates आना बंद हो जाएँ और replication सफल हो, तो eventually सभी replicas में latest data आ जाता है।

### Simple Example

```text
MySQL  → ₹500
Redis  → ₹300
```

MySQL में balance update हो चुका है, लेकिन Redis में अभी पुराना value है।

कुछ समय बाद:

```text
MySQL  → ₹500
Redis  → ₹500
```

यही **Eventual Consistency** है।

---

# 2. Strong Consistency vs Eventual Consistency

| Feature | Strong Consistency | Eventual Consistency |
|---|---|---|
| Latest data | तुरंत सभी जगह | थोड़ी delay हो सकती है |
| Read accuracy | बहुत high | temporary stale data possible |
| Performance | comparatively lower | generally higher |
| Availability | कम हो सकती है | generally higher |
| Complexity | कम | ज्यादा |
| Best for | Payments, critical transactions | Cache, analytics, feeds |

---

# 3. Common Problems

## 3.1 Stale Data

एक service में latest data होता है जबकि दूसरी service में पुराना data।

```text
MySQL  → ₹500
Redis  → ₹300
```

Application अगर Redis से read करती है तो user को गलत/पुराना result मिल सकता है।

### Solution

- Cache invalidation करें।
- TTL रखें।
- Critical reads के लिए source-of-truth database से read करें।
- Important updates के बाद cache refresh/invalidate करें।

---

# 4. Race Condition

दो requests एक ही data को लगभग एक साथ update कर सकती हैं।

```text
Request A → Balance ₹500 → ₹400
Request B → Balance ₹500 → ₹300
```

अगर proper concurrency control नहीं है तो एक update दूसरे को overwrite कर सकता है।

### Solution

Use:

- Database transactions
- Row-level locking
- Optimistic locking
- Version numbers
- Atomic database operations

Example:

```sql
UPDATE accounts
SET balance = balance - 100
WHERE id = 'user-123'
  AND balance >= 100;
```

---

# 5. Lost Update

दो services एक ही record को update करती हैं और बाद वाला update पहले वाले बदलाव को overwrite कर देता है।

### Solution — Optimistic Locking

Record में version रखें:

```text
id       balance   version
user-1   ₹500      7
```

Update करते समय version verify करें:

```sql
UPDATE accounts
SET balance = 400,
    version = 8
WHERE id = 'user-1'
  AND version = 7;
```

अगर affected rows `0` हैं, तो किसी दूसरे process ने record पहले ही बदल दिया है।

---

# 6. Duplicate Events

Message queue में कभी-कभी same event दो बार process हो सकता है।

```text
ORDER_PAID
     ↓
Consumer
     ↓
Processed

ORDER_PAID
     ↓
Consumer
     ↓
Processed again ❌
```

Payment, wallet या order जैसे systems में यह dangerous हो सकता है।

### Solution — Idempotency

हर event को unique ID दें:

```text
event_id = evt_123456
```

Processed events को store करें:

```text
event_id
---------
evt_123456
```

अगर वही event दोबारा आए:

```text
Already processed?
       ↓
      YES
       ↓
    Ignore
```

इससे duplicate processing से बचा जा सकता है।

---

# 7. Out-of-Order Events

Events हमेशा उसी order में process हों, यह जरूरी नहीं है।

Example:

```text
Event A → Order Pending
Event B → Order Paid
```

लेकिन consumer को मिल सकता है:

```text
B → Paid
A → Pending
```

इससे final state गलत हो सकती है।

### Solution

Event में version/sequence number रखें:

```json
{
  "eventId": "evt-101",
  "aggregateId": "order-123",
  "version": 5
}
```

Consumer पुराने version वाले events को reject/ignore कर सकता है।

---

# 8. Message Loss

Database में update हो गया लेकिन event publish नहीं हुआ:

```text
MySQL
  ↓
UPDATE successful ✅

RabbitMQ
  ↓
Message publish failed ❌
```

अब बाकी services को update की जानकारी नहीं मिलेगी।

### Solution — Transactional Outbox Pattern

Database transaction के अंदर business update और outbox event दोनों save करें।

```sql
BEGIN;

UPDATE orders
SET status = 'paid'
WHERE id = 'order-123';

INSERT INTO outbox_events
(event_type, aggregate_id)
VALUES
('ORDER_PAID', 'order-123');

COMMIT;
```

फिर background worker outbox से event पढ़कर RabbitMQ में publish करता है।

```text
MySQL
 ├── Business Data
 └── Outbox Event
          ↓
       Worker
          ↓
      RabbitMQ
          ↓
      Consumers
```

इससे database update और event creation के बीच inconsistency का risk कम होता है।

---

# 9. Consumer Failure

Consumer message receive करने के बाद processing fail हो सकती है।

```text
RabbitMQ
   ↓
Consumer
   ↓
Processing ❌
```

### Solution

Use:

- Retry
- Exponential backoff
- Dead Letter Queue (DLQ)

Example:

```text
Message
   ↓
Consumer ❌
   ↓
Retry 1
   ↓
Retry 2
   ↓
Retry 3
   ↓
DLQ
```

DLQ में failed messages रखे जाते हैं ताकि उन्हें बाद में inspect और reprocess किया जा सके।

---

# 10. Cache Inconsistency

Example:

```text
MySQL:
User name = Rahul

Redis:
User name = Amit
```

Database सही है लेकिन cache stale है।

### Solution

### Cache-Aside Pattern

```text
Application
    ↓
Check Redis
    ↓
Cache Hit → Return
    ↓
Cache Miss
    ↓
Read MySQL
    ↓
Update Redis
    ↓
Return
```

Update के समय:

```text
UPDATE MySQL
     ↓
DELETE Redis Cache
```

अगली read पर cache फिर से populate हो सकता है।

---

# 11. Recommended Architecture

अगर system में:

- MySQL
- NestJS
- RabbitMQ
- Redis / Dragonfly
- Background Workers

का इस्तेमाल हो रहा है, तो यह architecture useful है:

```text
                  ┌───────────────┐
                  │   NestJS API  │
                  └───────┬───────┘
                          │
                    Transaction
                          │
                  ┌───────▼───────┐
                  │     MySQL     │
                  │ Source Truth  │
                  └───────┬───────┘
                          │
                    Outbox Event
                          │
                  ┌───────▼───────┐
                  │     Worker    │
                  └───────┬───────┘
                          │
                  ┌───────▼───────┐
                  │   RabbitMQ    │
                  └───────┬───────┘
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
          Redis        Search      Notification
```

---

# 12. Source of Truth

Critical data के लिए एक authoritative source रखें।

Example:

```text
MySQL = Source of Truth
Redis = Cache
RabbitMQ = Event Transport
Search = Read Model
Analytics = Derived Data
```

Redis या search index को primary source of truth नहीं बनाना चाहिए अगर data critical है।

---

# 13. Where Eventual Consistency Is Good

Eventual consistency इन use cases में अच्छी हो सकती है:

- Redis cache
- Search indexes
- Analytics
- Notification
- Activity feeds
- Reports
- Recommendation systems
- Background processing
- Counters
- Non-critical read models

Example:

```text
Order Created
     ↓
MySQL updated immediately
     ↓
Analytics updated after 1–2 seconds
```

Analytics में कुछ seconds की delay generally acceptable हो सकती है।

---

# 14. Where Strong Consistency Is Better

इन operations में stronger consistency की जरूरत हो सकती है:

- Payment
- Wallet balance
- Bank transactions
- Inventory deduction
- Financial ledger
- Critical authorization
- Subscription payment status

Example:

```text
Wallet Balance = ₹1000

User spends ₹800

Available balance check
        ↓
Database transaction
        ↓
₹1000 → ₹200
```

ऐसे operations को केवल eventually consistent cache के भरोसे नहीं रखना चाहिए।

---

# 15. Practical Rules

### Rule 1

**Database को Source of Truth रखें।**

### Rule 2

Critical operations के लिए database transaction इस्तेमाल करें।

### Rule 3

Distributed events के लिए **Transactional Outbox** consider करें।

### Rule 4

Consumers को **idempotent** बनाएं।

### Rule 5

Events में version/sequence number रखें जहाँ ordering important हो।

### Rule 6

Failed messages के लिए retry + DLQ रखें।

### Rule 7

Cache को authoritative data न मानें।

### Rule 8

Critical reads में stale cache से बचें।

### Rule 9

Monitoring और reconciliation jobs रखें।

### Rule 10

हर system में strong consistency की जरूरत नहीं होती।

---

# 16. Reconciliation

Eventual consistency systems में periodic reconciliation बहुत useful है।

Example:

```text
MySQL Orders
     ↓
Compare
     ↓
Redis / Search
     ↓
Mismatch?
     ↓
Repair
```

Example:

```text
MySQL Order Status = PAID
Redis Order Status = PENDING
```

Reconciliation job mismatch detect करके Redis को correct कर सकती है।

---

# 17. Final Architecture Principle

एक practical distributed application में:

```text
                    WRITE
                      │
                      ▼
              ┌──────────────┐
              │    MySQL     │
              │ Source Truth │
              └──────┬───────┘
                     │
                 Outbox
                     │
                     ▼
                RabbitMQ
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
        Redis      Search    Analytics
          │          │          │
          └──────────┴──────────┘
                     │
                   READ
```

**Core principle:**

> **Strong consistency जहाँ correctness critical है, और eventual consistency जहाँ scalability, performance और availability ज्यादा important हैं।**

इस approach से distributed system को scalable रखते हुए data inconsistency के risks को controlled तरीके से handle किया जा सकता है.

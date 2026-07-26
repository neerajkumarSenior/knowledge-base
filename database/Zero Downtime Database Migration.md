# Zero Downtime Database Migration
## बड़े Production Database में बिना Downtime के नया Column कैसे जोड़ें?

> **Level:** Intermediate → Senior Backend
>
> **Topics:** Database, PostgreSQL, MySQL, Production, System Design, Backend Engineering

---

# समस्या

मान लीजिए आपके पास एक `users` टेबल है जिसमें **10 लाख (1M+) रिकॉर्ड** हैं।

साथ ही,

- हर 10 सेकंड में नया रिकॉर्ड Insert हो रहा है।
- आपकी Application Production में Live है।
- Users लगातार API इस्तेमाल कर रहे हैं।
- आपको एक नया Column जोड़ना है।

उदाहरण:

```sql
ALTER TABLE users
ADD COLUMN phone_verified BOOLEAN;
```

क्या आप सीधे Migration चला देंगे?

**नहीं।**

क्योंकि Production Database में ऐसा करना कई समस्याएँ पैदा कर सकता है।

---

# सीधे Migration करने पर क्या समस्याएँ आ सकती हैं?

यदि आप एक ही बार में

```sql
ALTER TABLE ...
```

और उसके बाद

```sql
UPDATE users
SET phone_verified = false;
```

चलाते हैं, तो

- Table Lock हो सकती है।
- Database बहुत Slow हो सकता है।
- CPU और Disk I/O बढ़ सकते हैं।
- Replication Lag हो सकती है।
- API Response Time बढ़ सकती है।
- Users को Timeout मिल सकता है।
- कभी-कभी Application कुछ समय के लिए Down भी हो सकती है।

यही कारण है कि बड़ी कंपनियाँ **Zero Downtime Migration Strategy** का उपयोग करती हैं।

---

# Zero Downtime Migration क्या है?

ऐसी Migration जिसमें

- Application बंद नहीं होती।
- Users को कोई Error नहीं मिलता।
- Database Lock नहीं होती।
- Data सुरक्षित रहता है।
- Migration धीरे-धीरे Background में पूरी होती है।

---

# Zero Downtime Migration के 4 चरण

```
Schema Change
      │
      ▼
Deploy Application
      │
      ▼
Background Backfill
      │
      ▼
Cleanup & Constraints
```

---

# चरण 1 : Nullable Column जोड़ें

सबसे पहले केवल Column जोड़ें।

उसे Nullable रखें।

```sql
ALTER TABLE users
ADD COLUMN phone_verified BOOLEAN NULL;
```

ध्यान दें:

❌ Default Value मत दीजिए।

❌ NOT NULL मत लगाइए।

---

## ऐसा क्यों?

Database को केवल Schema Update करना पड़ता है।

उसे 10 लाख रिकॉर्ड Update नहीं करने पड़ते।

इसलिए Migration बहुत जल्दी पूरी हो जाती है।

---

### Migration के बाद

| id | name | phone_verified |
|----|------|----------------|
|1|Rahul|NULL|
|2|Amit|NULL|
|3|Neha|NULL|

सभी पुराने रिकॉर्ड में NULL रहेगा।

---

# चरण 2 : नया Backend Deploy करें

अब Application का नया Version Deploy करें।

अब से जब भी नया User बने

```text
phone_verified = false
```

Save करें।

Pseudo Code

```ts
createUser({
    name: "Rahul",
    phoneVerified: false
})
```

अब स्थिति होगी

| id | name | phone_verified |
|----|------|----------------|
|1|Rahul|NULL|
|2|Amit|NULL|
|3|Neha|NULL|
|1000001|Neeraj|false|

ध्यान दें

पुराना Data अभी भी NULL है।

नया Data सही Value के साथ Insert हो रहा है।

Application पूरी तरह Live है।

---

# चरण 3 : पुराने Data को Batch में Update करें

यहीं सबसे अधिक गलती होती है।

---

## गलत तरीका

```sql
UPDATE users
SET phone_verified = false;
```

यदि Table में 10 लाख रिकॉर्ड हैं,

तो Database एक साथ सभी रिकॉर्ड Update करेगी।

इससे

- Lock
- Slow Queries
- CPU Spike
- High Disk I/O
- Replication Delay

जैसी समस्याएँ आ सकती हैं।

---

## सही तरीका

छोटे Batch में Update करें।

उदाहरण

```sql
UPDATE users
SET phone_verified = false
WHERE phone_verified IS NULL
LIMIT 1000;
```

फिर

- 1000 रिकॉर्ड Update
- कुछ सेकंड प्रतीक्षा
- अगला Batch

```
Batch 1

1 → 1000

↓

Batch 2

1001 → 2000

↓

Batch 3

2001 → 3000

↓

...
```

इस प्रक्रिया को Background Worker या Cron Job चला सकता है।

---

# Background Worker का उदाहरण

```
while (true)

    Update 1000 rows

    Sleep 2 seconds

Repeat
```

Users को पता भी नहीं चलेगा कि Migration चल रही है।

---

# चरण 4 : Constraint जोड़ें

जब

```
SELECT COUNT(*)
FROM users
WHERE phone_verified IS NULL;
```

का Result

```
0
```

आ जाए,

तब

```sql
ALTER TABLE users
ALTER COLUMN phone_verified
SET NOT NULL;
```

यदि Default चाहिए

```sql
ALTER TABLE users
ALTER COLUMN phone_verified
SET DEFAULT false;
```

अब Migration पूरी हो चुकी है।

---

# पूरा Flow

```
                 Start

                   │

                   ▼

        Add Nullable Column

                   │

                   ▼

       Deploy New Application

                   │

                   ▼

      New Records नई Value लिखेंगे

                   │

                   ▼

 Background Worker पुराने Data को
      छोटे Batch में Update करेगा

                   │

                   ▼

     NULL Records = 0 ?

          │             │

        No              Yes

        │                │

        ▼                ▼

Continue Backfill    Add NOT NULL

                          │

                          ▼

                    Migration Complete
```

---

# Batch Size कितना होना चाहिए?

यह आपके Database और Traffic पर निर्भर करता है।

आम तौर पर

- 500 Rows
- 1000 Rows
- 5000 Rows

Production में उपयोग किए जाते हैं।

यदि Traffic अधिक है,

तो Batch छोटा रखें।

---

# बड़ी कंपनियाँ क्या करती हैं?

Netflix

- Background Backfill

Uber

- Expand → Migrate → Contract Pattern

Google

- Online Schema Changes

Amazon

- Progressive Deployment

GitHub

- Batched Migration

Facebook (Meta)

- Online Data Migration

---

# Expand → Migrate → Contract Pattern

यह Zero Downtime Migration का सबसे लोकप्रिय Pattern है।

```
Expand

↓

नई Schema जोड़ो

↓

Deploy Code

↓

Backfill

↓

Verify

↓

Contract

↓

पुराना Code और पुराना Column हटाओ
```

इसे **Expand and Contract Pattern** भी कहा जाता है।

---

# Interview Answer (Senior Backend)

यदि Interviewer पूछे:

> "आपके Database में 10 लाख रिकॉर्ड हैं और हर 10 सेकंड में नया रिकॉर्ड Insert हो रहा है। बिना Downtime नया Column कैसे जोड़ेंगे?"

तो उत्तर होगा

### Step 1

नई Nullable Column जोड़ूँगा।

### Step 2

Application Deploy करूँगा ताकि नए रिकॉर्ड उसी समय नई Value Save करें।

### Step 3

Background Worker से पुराने Data को छोटे Batch में Backfill करूँगा।

### Step 4

सभी रिकॉर्ड Update होने के बाद NOT NULL और Default Constraint जोड़ूँगा।

इस पूरी प्रक्रिया में

- Downtime नहीं होगी।
- Table Lock नहीं होगी।
- Users प्रभावित नहीं होंगे।
- Application लगातार चलती रहेगी।

---

# महत्वपूर्ण सीख

✅ Production Database पर कभी भी बड़े UPDATE एक साथ मत चलाइए।

✅ पहले Schema बदलिए।

✅ फिर Application Deploy कीजिए।

✅ उसके बाद धीरे-धीरे Data Backfill कीजिए।

✅ अंत में Constraints जोड़िए।

---

# याद रखने का आसान Formula

```
Expand

↓

Deploy

↓

Backfill

↓

Validate

↓

Contract
```

---

# निष्कर्ष

Zero Downtime Migration केवल SQL लिखने का विषय नहीं है, बल्कि Production Systems की समझ का हिस्सा है।

एक अनुभवी Backend Engineer हमेशा ऐसी Migration डिज़ाइन करता है जिसमें:

- Database सुरक्षित रहे
- Application बंद न हो
- Users प्रभावित न हों
- Migration धीरे-धीरे और नियंत्रित तरीके से पूरी हो

यही दृष्टिकोण बड़े स्तर (Large Scale) के Backend Systems में अपनाया जाता है।

---

## यदि यह लेख उपयोगी लगा हो

⭐ इस Repository को Star करें

🍴 Fork करें

📢 अपने Backend Developer साथियों के साथ साझा करें।

Happy Coding! 🚀

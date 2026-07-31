# Database Scaling: Partitioning vs Sharding vs Replication

जब किसी Application के Users और Data बहुत अधिक बढ़ जाते हैं, तब केवल एक Database Server पर्याप्त नहीं होता। ऐसी स्थिति में Database Scaling की आवश्यकता होती है।

Database Scaling की तीन प्रमुख तकनीकें हैं:

1. Partitioning (पार्टिशनिंग)
2. Sharding (शार्डिंग)
3. Replication (रेप्लिकेशन)

---

# 1. Partitioning (पार्टिशनिंग)

## क्या है?

Partitioning का अर्थ है **एक ही Database के अंदर Data को छोटे-छोटे हिस्सों (Partitions) में बाँटना।**

> ध्यान दें: सभी Partitions एक ही Database Server पर हो सकते हैं।

### उदाहरण

मान लीजिए `bus_locations` टेबल में 100 करोड़ रिकॉर्ड हैं।

इसे वर्ष के आधार पर विभाजित किया गया।

```
bus_locations_2025
bus_locations_2026
bus_locations_2027
```

या

```
ID 1 - 1000000
ID 1000001 - 2000000
```

---

## फायदे

- Query तेज़ चलती है।
- Index छोटे हो जाते हैं।
- Maintenance आसान होती है।
- पुराना Data आसानी से Archive किया जा सकता है।

---

## नुकसान

- Server वही रहता है।
- Hardware Limit समाप्त नहीं होती।
- बहुत अधिक Load होने पर समस्या बनी रहती है।

---

# 2. Sharding (शार्डिंग)

## क्या है?

Sharding का अर्थ है **Data को कई अलग-अलग Database Servers में बाँटना।**

हर Server Data का केवल एक भाग रखता है।

### उदाहरण

```
Server 1
Users 1 - 10 लाख

Server 2
Users 10 - 20 लाख

Server 3
Users 20 - 30 लाख
```

या

```
North India  → DB-1

South India  → DB-2

West India   → DB-3
```

---

## फायदे

- Horizontal Scaling
- बहुत अधिक Users Handle कर सकता है।
- Write Performance बेहतर होती है।
- प्रत्येक Server पर Load कम होता है।

---

## नुकसान

- Design जटिल हो जाती है।
- Cross-Shard Query कठिन होती है।
- Data Migration चुनौतीपूर्ण होती है।

---

# 3. Replication (रेप्लिकेशन)

## क्या है?

Replication में **एक ही Database की कई Copies बनाई जाती हैं।**

```
             Primary Database
                    │
      ┌─────────────┼─────────────┐
      │             │             │
 Replica-1      Replica-2     Replica-3
```

आमतौर पर

- Write केवल Primary Database में होती है।
- Read Replica Databases से होती है।

---

## फायदे

- Read Performance बढ़ती है।
- Backup उपलब्ध रहता है।
- High Availability मिलती है।
- Failover आसान होता है।

---

## नुकसान

- Write केवल Primary में होती है।
- Replication Delay हो सकता है।
- Storage अधिक लगता है।

---

# तीनों में अंतर

| Feature | Partitioning | Sharding | Replication |
|----------|-------------|----------|-------------|
| Data Split | ✅ हाँ | ✅ हाँ | ❌ नहीं |
| Data Copy | ❌ नहीं | ❌ नहीं | ✅ हाँ |
| Multiple Servers | ❌ सामान्यतः नहीं | ✅ हाँ | ✅ हाँ |
| Read Performance | थोड़ा बेहतर | बेहतर | बहुत बेहतर |
| Write Performance | थोड़ा बेहतर | बहुत बेहतर | लगभग समान |
| High Availability | ❌ नहीं | आंशिक | ✅ हाँ |
| Horizontal Scaling | ❌ नहीं | ✅ हाँ | ❌ नहीं |

---

# सरल उदाहरण

## Partitioning

एक अलमारी है जिसमें 10 दराज़ हैं।

```
Drawer 1
Drawer 2
Drawer 3
```

सभी दराज़ एक ही अलमारी में हैं।

---

## Sharding

अब 10 अलग-अलग अलमारियाँ हैं।

```
Almirah 1

Almirah 2

Almirah 3
```

प्रत्येक अलमारी अलग स्थान पर रखी है।

---

## Replication

एक ही किताब की 3 प्रतियाँ हैं।

```
Original Book

Copy 1

Copy 2

Copy 3
```

यदि एक खो जाए, तब भी बाकी उपलब्ध रहती हैं।

---

# Bus Tracking System में उपयोग

## Partitioning

```
bus_locations_2025

bus_locations_2026

bus_locations_2027
```

---

## Sharding

```
North India Server

South India Server

East India Server

West India Server
```

---

## Replication

```
Primary Database

↓

Replica Delhi

↓

Replica Mumbai

↓

Replica Bangalore
```

Read Request Replica से होगी और Write Primary में होगी।

---

# कब क्या उपयोग करें?

| स्थिति | उपयुक्त तकनीक |
|---------|---------------|
| Data बहुत बड़ा है लेकिन एक Server पर्याप्त है | Partitioning |
| करोड़ों Users और बहुत अधिक Writes हैं | Sharding |
| Read Traffic बहुत अधिक है | Replication |
| Backup और High Availability चाहिए | Replication |
| Global Scale Application | Sharding + Replication |
| Enterprise Database | Partitioning + Replication |
| Google, Facebook, Amazon जैसी बड़ी Systems | Sharding + Replication + Partitioning |

---

# याद रखने की ट्रिक

- **Partitioning = एक Database के अंदर Data बाँटना।**
- **Sharding = Data को कई Database Servers में बाँटना।**
- **Replication = उसी Data की कई Copies बनाना।**

---

# निष्कर्ष

- **Partitioning** का उद्देश्य Performance बढ़ाना है।
- **Sharding** का उद्देश्य बड़े Scale पर Data और Traffic को कई Servers में बाँटना है।
- **Replication** का उद्देश्य Backup, High Availability और Read Performance बढ़ाना है।

बड़ी कंपनियाँ (Google, Amazon, Meta आदि) अक्सर इन तीनों तकनीकों का संयोजन (Combination) उपयोग करती हैं।

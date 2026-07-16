# Leaky Bucket Algorithm (लीकी बकेट एल्गोरिथ्म)

> **शुरुआती (Beginner) से लेकर एडवांस स्तर तक** लीकी बकेट एल्गोरिथ्म की पूरी जानकारी। यह एल्गोरिथ्म **Rate Limiting**, **Traffic Shaping**, **API Gateway**, **Networking** और **Distributed Systems** में व्यापक रूप से उपयोग किया जाता है।

---

# 📚 विषय सूची

- लीकी बकेट एल्गोरिथ्म क्या है?
- इसकी आवश्यकता क्यों पड़ती है?
- वास्तविक जीवन का उदाहरण
- यह कैसे काम करता है?
- आर्किटेक्चर डायग्राम
- एल्गोरिथ्म
- Pseudocode
- उदाहरण
- फायदे
- नुकसान
- Leaky Bucket vs Token Bucket
- वास्तविक उपयोग (Use Cases)
- System Design Interview Questions
- सारांश

---

# लीकी बकेट एल्गोरिथ्म क्या है?

**Leaky Bucket Algorithm** एक **Traffic Shaping** और **Rate Limiting** एल्गोरिथ्म है।

इसका मुख्य उद्देश्य यह सुनिश्चित करना है कि सिस्टम से **रिक्वेस्ट (Requests)** या **नेटवर्क पैकेट्स** एक **निश्चित (Constant) गति** से बाहर जाएँ।

यदि अचानक बहुत अधिक रिक्वेस्ट आ जाएँ, तो यह एल्गोरिथ्म उन्हें पहले **Queue (Bucket)** में रखता है और धीरे-धीरे निश्चित गति से प्रोसेस करता है।

यदि Bucket पूरी तरह भर जाती है, तो नई आने वाली अतिरिक्त रिक्वेस्ट **Reject (Drop)** कर दी जाती हैं।

---

# इसकी आवश्यकता क्यों पड़ती है?

मान लीजिए आपकी API एक सेकंड में केवल

```
100 Requests
```

सुरक्षित रूप से संभाल सकती है।

लेकिन अचानक

```
5000 Requests
```

एक साथ आ जाती हैं।

यदि Rate Limiting न हो तो—

- CPU का उपयोग बहुत बढ़ जाएगा।
- Database पर अत्यधिक Load आएगा।
- Memory भर सकती है।
- API Crash हो सकती है।
- पूरा सिस्टम Slow या Down हो सकता है।

Leaky Bucket Algorithm इस समस्या को रोकता है।

---

# वास्तविक जीवन का उदाहरण

मान लीजिए आपके पास एक पानी की बाल्टी है जिसमें नीचे एक छोटा सा छेद है।

```
        पानी

   ↓ ↓ ↓ ↓ ↓ ↓

   +------------+
   |            |
   |   बाल्टी   |
   |            |
   +------------+
         |
         |
     छोटा छेद
         |
         ▼
```

चाहे ऊपर से कितना भी पानी डालें,

नीचे से पानी **एक निश्चित गति** से ही निकलेगा।

यही सिद्धांत Leaky Bucket Algorithm में भी लागू होता है।

---

# यह कैसे काम करता है?

## चरण 1

सभी Incoming Requests Bucket में आती हैं।

```
Client

   │

   ▼

+-----------+

|  Bucket   |

+-----------+
```

---

## चरण 2

Bucket रिक्वेस्ट को Queue में स्टोर करती है।

उदाहरण

```
Bucket Capacity = 10 Requests

[1]

[2]

[3]

[4]

[5]

...
```

---

## चरण 3

Bucket से Requests निश्चित गति से बाहर निकलती हैं।

उदाहरण

```
Leak Rate

2 Requests / Second
```

यदि

```
100 Requests
```

एक साथ भी आ जाएँ,

फिर भी केवल

```
2 Requests
```

हर सेकंड प्रोसेस होंगी।

---

## चरण 4

यदि Bucket पूरी भर जाए

```
Capacity = 10

Incoming = 15
```

तो

```
10 Requests
```

स्टोर होंगी

और

```
5 Requests
```

Drop कर दी जाएँगी।

---

# आर्किटेक्चर

```
              Incoming Requests

         R R R R R R R R R R

                 │

                 ▼

      +-----------------------+

      |     Leaky Bucket      |

      |       (Queue)         |

      +-----------------------+

                 │

      निश्चित गति (Leak Rate)

                 │

                 ▼

          Application Server

                 │

                 ▼

              Database
```

---

# एल्गोरिथ्म

1. प्रत्येक नई Request Bucket में डालो।
2. यदि Bucket में जगह है तो Request Store करो।
3. यदि Bucket भर चुकी है तो Request Reject कर दो।
4. निश्चित समय के अंतराल पर Queue से Request निकालो।
5. Server को भेजो।

---

# Pseudocode

```text
Bucket Capacity = 10

Leak Rate = 2 Requests/Second

यदि Bucket में जगह है

    Request जोड़ दो

अन्यथा

    Request Reject कर दो

हर सेकंड

    Leak Rate के अनुसार Request निकालो

    Server को भेज दो
```

---

# उदाहरण

मान लीजिए

```
Bucket Capacity = 10

Leak Rate = 2/sec

Incoming = 15 Requests
```

तो परिणाम होगा

```
Store

10 Requests

Drop

5 Requests

पहला सेकंड

2 Requests Process

बाकी

8

दूसरा सेकंड

2 Process

बाकी

6
```

धीरे-धीरे सभी 10 Requests प्रोसेस हो जाएँगी।

---

# फायदे

✅ सिस्टम को Overload होने से बचाता है।

✅ Traffic को Smooth बनाता है।

✅ Implementation बहुत आसान है।

✅ Server Crash होने की संभावना कम होती है।

✅ Database पर अचानक Load नहीं आता।

✅ CPU का उपयोग नियंत्रित रहता है।

---

# नुकसान

❌ Burst Traffic को Handle नहीं कर सकता।

❌ Bucket भरने पर Requests Drop हो जाती हैं।

❌ Waiting Time बढ़ सकता है।

❌ Token Bucket जितना Flexible नहीं है।

---

# Leaky Bucket और Token Bucket में अंतर

| विशेषता | Leaky Bucket | Token Bucket |
|----------|--------------|--------------|
| Output Rate | हमेशा निश्चित | आवश्यकता अनुसार बदल सकती है |
| Burst Traffic | ❌ नहीं | ✅ हाँ |
| Overflow | Requests Drop | Tokens जमा रहते हैं |
| उपयोग | Smooth Traffic | Bursty Traffic |
| Complexity | आसान | थोड़ा जटिल |

---

# वास्तविक उपयोग (Use Cases)

## API Gateway

```
Client

↓

Leaky Bucket

↓

API Server

↓

Database
```

उदाहरण

- Login API
- Payment API
- OTP API
- SMS API

---

## Network Router

Network Routers इसका उपयोग करते हैं—

- Traffic Shaping
- Congestion Control
- Packet Flow Control

---

## Reverse Proxy

- Nginx
- HAProxy
- Envoy
- Kong API Gateway

---

## Microservices

```
Service A

↓

Leaky Bucket

↓

Service B

↓

Database
```

यह Downstream Services को Overload होने से बचाता है।

---

# Time Complexity

| Operation | Complexity |
|-----------|------------|
| Insert | O(1) |
| Remove | O(1) |
| Space | O(n) |

जहाँ

```
n = Bucket Capacity
```

---

# System Design Interview Questions

### प्रश्न 1

Leaky Bucket Algorithm का मुख्य उद्देश्य क्या है?

**उत्तर**

Traffic को नियंत्रित करना और Server को Overload होने से बचाना।

---

### प्रश्न 2

Bucket भर जाने पर क्या होता है?

**उत्तर**

नई आने वाली Requests Reject (Drop) कर दी जाती हैं।

---

### प्रश्न 3

क्या Leaky Bucket Burst Traffic को Handle कर सकता है?

**उत्तर**

नहीं।

यह हमेशा निश्चित गति से Output देता है।

---

### प्रश्न 4

Leaky Bucket कहाँ उपयोग किया जाता है?

- API Gateway
- Routers
- Reverse Proxy
- Cloud Platforms
- Distributed Systems

---

### प्रश्न 5

Leaky Bucket और Token Bucket में सबसे बड़ा अंतर क्या है?

**उत्तर**

Leaky Bucket हमेशा निश्चित गति से Requests भेजता है, जबकि Token Bucket Burst Traffic को भी Allow करता है।

---

# सारांश

- Leaky Bucket एक **Traffic Shaping** और **Rate Limiting** एल्गोरिथ्म है।
- यह Requests को Queue में रखता है।
- निश्चित गति से Requests को Process करता है।
- Bucket भरने पर अतिरिक्त Requests Drop कर देता है।
- इसका उपयोग API Gateway, Routers, Reverse Proxy और Distributed Systems में किया जाता है।
- इसका मुख्य उद्देश्य Server और Database को Overload होने से बचाना है।

---

# मुख्य बिंदु (Key Takeaways)

- निश्चित आकार (Fixed Size) की Bucket
- निश्चित गति (Constant Leak Rate)
- Overflow होने पर Requests Drop
- Traffic हमेशा Smooth रहता है
- Server सुरक्षित रहता है
- Networking और System Design में अत्यंत महत्वपूर्ण एल्गोरिथ्म

---

## 📖 आगे क्या सीखें?

Leaky Bucket सीखने के बाद निम्नलिखित विषय अवश्य पढ़ें—

1. Token Bucket Algorithm
2. Sliding Window Algorithm
3. Fixed Window Counter
4. Rate Limiting
5. API Gateway
6. Load Balancer
7. Circuit Breaker
8. Retry Pattern
9. Queue (Kafka, RabbitMQ)
10. Distributed Rate Limiting

---

⭐ **यदि यह नोट्स आपके लिए उपयोगी रहे हों, तो इस Repository को Star ⭐ करना न भूलें।**

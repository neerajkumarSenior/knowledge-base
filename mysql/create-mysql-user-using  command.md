
#Create new User
CREATE USER 'super_app_u'@'%' IDENTIFIED BY 'x45ctr565778';

GRANT ALL PRIVILEGES
ON super_app.*
TO 'super_app_u'@'%';

FLUSH PRIVILEGES;


#Now checking user created or not 

SELECT user, host
FROM mysql.user
WHERE user='super_app_u';

# Database User Security Guide

## उद्देश्य

यह दस्तावेज़ बताता है कि किसी एप्लिकेशन को सीधे MySQL के `root` उपयोगकर्ता से कनेक्ट करने के बजाय एक Dedicated Database User का उपयोग क्यों करना चाहिए।

---

## समस्या

कई डेवलपर्स विकास (Development) के दौरान एप्लिकेशन को सीधे `root` उपयोगकर्ता से MySQL से जोड़ देते हैं।

उदाहरण:

```env id="dxd6m7"
DB_USER=root
DB_PASSWORD=your_password
```

हालाँकि यह काम करता है, लेकिन Production Environment में यह एक सुरक्षित तरीका नहीं माना जाता।

---

## Dedicated User क्या है?

Dedicated User एक ऐसा MySQL उपयोगकर्ता होता है जिसे केवल एप्लिकेशन के लिए बनाया जाता है और उसे केवल आवश्यक Permissions दी जाती हैं।

उदाहरण:

```sql id="zyy9md"
CREATE USER 'super_app_user'@'%' IDENTIFIED BY 'strong_password';
```

---

## Dedicated User बनाने के लाभ

### 1. बेहतर सुरक्षा

यदि एप्लिकेशन से जुड़े Credentials किसी कारण से लीक हो जाते हैं, तो हमलावर (Attacker) केवल उसी Database तक सीमित रहेगा जिसके लिए Permissions दी गई हैं।

---

### 2. Root Account सुरक्षित रहता है

Root User के पास पूरे MySQL Server पर नियंत्रण होता है।

Dedicated User उपयोग करने से Root Account को एप्लिकेशन से अलग रखा जा सकता है।

---

### 3. Principle of Least Privilege

सिर्फ उतनी ही Permissions दी जाती हैं जितनी एप्लिकेशन को वास्तव में चाहिए।

---

### 4. Production Best Practice

लगभग सभी Production Systems, Cloud Platforms और Enterprise Applications Dedicated Database Users का उपयोग करते हैं।

---

## Dedicated User बनाना

```sql id="w9ph6l"
CREATE USER 'super_app_user'@'%' IDENTIFIED BY 'strong_password';
```

---

## Database Permissions देना

```sql id="4rk5yo"
GRANT ALL PRIVILEGES
ON super_app.*
TO 'super_app_user'@'%';
```

या अधिक सुरक्षित तरीके से:

```sql id="zovlzg"
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, ALTER, INDEX
ON super_app.*
TO 'super_app_user'@'%';
```

---

## Permissions Reload करना

```sql id="p0rjlwm"
FLUSH PRIVILEGES;
```

---

## Environment Configuration

```env id="wxbv4j"
DB_URL=jdbc:mysql://mysql:3306/super_app
DB_USER=super_app_user
DB_PASSWORD=strong_password
```

---

## Root User क्यों नहीं?

Root User:

* सभी Databases देख सकता है
* Databases Delete कर सकता है
* Users Create/Delete कर सकता है
* Server Configuration बदल सकता है

इसलिए Application को Root User से Connect नहीं करना चाहिए।

---

## निष्कर्ष

Production Environment में Dedicated Database User का उपयोग करना अधिक सुरक्षित, बेहतर प्रबंधनीय (Maintainable) और Industry Best Practice माना जाता है। इससे Application केवल उन्हीं Resources तक सीमित रहती है जिनकी उसे वास्तव में आवश्यकता होती है, जबकि Root Account सुरक्षित बना रहता है।

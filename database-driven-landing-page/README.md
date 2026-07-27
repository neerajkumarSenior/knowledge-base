# Landing Page Settings System
#Database-Driven Landing Page Architecture (settings_type + key_name Pattern)
## Overview

यह सिस्टम पूरी Landing Page को **Database Driven** बनाता है।

हर Section का Content Database में Store होता है, इसलिए Developer को बार-बार Code Change करके Deploy करने की जरूरत नहीं पड़ती।

---

# Table Structure

| Column | Description |
|---------|-------------|
| id | Unique Record ID |
| key_name | Section के अंदर Content की Unique Key |
| value | JSON Data |
| settings_type | किस Section का Data है |
| order_index | Display Order |
| is_active | Enable / Disable |
| created_at | Created Time |
| updated_at | Last Updated |

---

# settings_type क्या है?

`settings_type` Landing Page का Section बताता है।

उदाहरण:

```
header
hero
features
clients
stats
reviews
footer
seo
```

अगर Frontend को Hero Section बनाना है तो वह केवल

```
settings_type = hero
```

का Data Fetch करेगा।

---

## Example

```
settings_type = hero
```

Database

```
intro_contents
primary_cta
banner_1
banner_2
banner_3
banner_4
```

Frontend इन्हें Hero Section में Render करेगा।

---

# key_name क्या है?

`key_name` Section के अंदर Component की पहचान है।

उदाहरण

Header Section

```
logo
menu
cta
```

Footer Section

```
logo
quick_links
service_links
event_links
contacts
social_links
copyright
```

Features

```
intro_contents
feature_1
feature_2
feature_3
```

---

## क्यों जरूरी है?

अगर केवल

```
settings_type = header
```

हो,

तो Database को कैसे पता चलेगा कि

```
Logo कौन सा है?

Menu कौन सा है?

CTA कौन सा है?
```

इसीलिए

```
key_name
```

की जरूरत होती है।

---

## Example

```
settings_type = header

key_name = logo
```

```
{
    "imageUrl":"..."
}
```

---

```
settings_type = header

key_name = menu
```

```
[
   {
      "label":"Home",
      "href":"/"
   }
]
```

---

```
settings_type = header

key_name = cta
```

```
{
   "text":"Booking",
   "href":"/contact"
}
```

Frontend आसानी से समझ जाता है कि

```
logo

↓

Header Logo
```

```
menu

↓

Navigation Menu
```

```
cta

↓

Button
```

---

# value

Actual Content JSON Format में Store होता है।

Example

```
{
    "title":"Welcome",
    "description":"Lorem ipsum"
}
```

या

```
[
   {
      "title":"Feature One"
   }
]
```

इससे Structure Flexible रहता है।

---

# order_index

Render Order तय करता है।

Example

```
Hero

1
2
3
4
```

यदि

```
Banner 2
```

को ऊपर दिखाना है

तो केवल

```
order_index
```

बदलना होगा।

Code Change नहीं होगा।

---

# is_active

```
1
```

मतलब दिखाना है।

```
0
```

मतलब Hide करना है।

Frontend केवल Active Records दिखाएगा।

---

# Complete Flow

```
Frontend

      │

      ▼

GET /landing-page

      │

      ▼

Database

      │

      ▼

settings_type

      │

      ├── header
      ├── hero
      ├── features
      ├── clients
      ├── reviews
      ├── footer

      │

      ▼

Each Section

      │

      ├── key_name
      │      ├── logo
      │      ├── menu
      │      ├── cta
      │      ├── intro_contents
      │      ├── feature_1
      │      └── ...

      ▼

Frontend Components

      │

      ▼

Landing Page
```

---

# Benefits

## 1. No Code Change

Content Update करने के लिए Deployment नहीं करना पड़ता।

---

## 2. Dynamic Website

पूरा Landing Page Database से Generate होता है।

---

## 3. Reusable Components

एक ही Component कई Projects में उपयोग किया जा सकता है।

---

## 4. CMS Friendly

Admin Panel से Content Manage किया जा सकता है।

---

## 5. Easy Ordering

Section या Cards का Order Database से बदल सकते हैं।

---

## 6. Enable / Disable

कोई भी Section तुरंत Hide या Show किया जा सकता है।

---

## 7. Scalable Architecture

नया Section जोड़ने के लिए Database में नया `settings_type` और संबंधित `key_name` जोड़ना पर्याप्त है। Existing Code में बड़े बदलाव की आवश्यकता नहीं होती।

---

# Example Records

| settings_type | key_name | Purpose |
|---------------|----------|----------|
| header | logo | Website Logo |
| header | menu | Navigation Menu |
| header | cta | Header Button |
| hero | intro_contents | Hero Text |
| hero | primary_cta | Hero CTA |
| banners | banner_1 | Hero Banner |
| features | feature_1 | Feature Card |
| clients | client_1 | Client Logo |
| stats | stat_1 | Statistics |
| reviews | review_1 | Customer Review |
| footer | quick_links | Footer Links |
| footer | social_links | Social Media |
| seo | meta | SEO Metadata |

---

# Conclusion

`settings_type` बताता है कि **डेटा किस Section का है**, जबकि `key_name` बताता है कि **उस Section के अंदर कौन-सा Component या Content Block है**। दोनों मिलकर Landing Page को पूरी तरह Database Driven, CMS Friendly और Scalable बनाते हैं।

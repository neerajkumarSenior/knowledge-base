

# Windows पर PostgreSQL और Adminer इंस्टॉल करने की गाइड

यह गाइड Windows पर PostgreSQL डेटाबेस सर्वर और Adminer database management UI को इंस्टॉल और चलाने के लिए है। इसमें PHP/XAMPP पहले से इंस्टॉल होना आवश्यक नहीं है, क्योंकि नीचे PHP के लिए XAMPP का उपयोग किया गया है।

## 1. PostgreSQL इंस्टॉल करें

1. PostgreSQL की आधिकारिक Windows डाउनलोड साइट खोलें:
   https://www.postgresql.org/download/windows/
2. आधिकारिक Windows installer डाउनलोड करें।
3. Installer चलाएँ और सामान्य विकल्पों के साथ आगे बढ़ें।
4. PostgreSQL का `postgres` user password सेट करें। इसे सुरक्षित रखें।
5. Port `5432` रहने दें, जब तक यह किसी दूसरे PostgreSQL instance द्वारा उपयोग न हो रहा हो।
6. Installation पूरी करें। जरूरत न हो तो अतिरिक्त Stack Builder packages इंस्टॉल करना आवश्यक नहीं है।

### PostgreSQL service जाँचें

PowerShell खोलकर चलाएँ:

```powershell
Get-Service *postgres*
```

अगर PostgreSQL service चल रही है, तो उसका status `Running` दिखाई देना चाहिए।

### psql से कनेक्शन जाँचें

PostgreSQL 18 के सामान्य install path के लिए:

```powershell
& "C:\Program Files\PostgreSQL\18\bin\psql.exe" -U postgres -h 127.0.0.1 -p 5432 -d postgres
```

- Password माँगे तो installation के दौरान सेट किया गया password डालें।
- सफल कनेक्शन के बाद `postgres=#` prompt दिखाई देगा।
- यह SQL चलाएँ:

```sql
SELECT version();
```

SQL statement के अंत में semicolon (`;`) लगाएँ। बाहर निकलने के लिए:

```sql
\q
```

> यदि आपके PostgreSQL का version 18 नहीं है, तो command में `18` को अपने installed version से बदलें।

## 2. PHP और XAMPP इंस्टॉल करें

Adminer एक PHP application है। इस गाइड में उसे चलाने के लिए XAMPP का PHP इस्तेमाल किया गया है।

1. XAMPP की आधिकारिक साइट खोलें:
   https://www.apachefriends.org/
2. Windows के लिए XAMPP इंस्टॉल करें।
3. XAMPP Control Panel में Apache चलाना अनिवार्य नहीं है; नीचे PHP का built-in development server इस्तेमाल होगा।

### PHP extensions enable करें

XAMPP के `php.ini` को खोलें। सामान्य path:

```text
C:\xampp\php\php.ini
```

फ़ाइल में `pdo_pgsql` और `pgsql` खोजें। यदि उनके सामने semicolon है, तो उसे हटाएँ:

```ini
extension=pdo_pgsql
extension=pgsql
```

फ़ाइल save करें। यदि Apache/PHP process चल रहा हो तो उसे restart करें।

PowerShell में extensions verify करें:

```powershell
C:\xampp\php\php.exe -m | findstr /i "pgsql"
```

Output में ये नाम आने चाहिए:

```text
pdo_pgsql
pgsql
```

अगर extensions नहीं दिखते, तो `php.ini` में extension entries, PHP extension directory और PHP version की जाँच करें।

## 3. Adminer डाउनलोड करें

1. Adminer की आधिकारिक वेबसाइट खोलें:
   https://www.adminer.org/
2. Adminer का PHP file डाउनलोड करें।
3. एक folder बनाएँ:

```powershell
New-Item -ItemType Directory -Force -Path C:\adminer
```

4. डाउनलोड की गई PHP file को इस path पर रखें:

```text
C:\adminer\adminer.php
```

यदि डाउनलोड की फ़ाइल का नाम अलग है, तो उसे `adminer.php` नाम दे दें।

Folder में फ़ाइल की मौजूदगी जाँचें:

```powershell
Test-Path C:\adminer\adminer.php
```

Output `True` होना चाहिए।

## 4. Adminer चलाएँ

PowerShell में चलाएँ:

```powershell
C:\xampp\php\php.exe -S 127.0.0.1:8080 -t C:\adminer
```

यह terminal खुला रहने दें। Adminer इस्तेमाल करते समय server इसी terminal में चलता रहेगा।

Browser में खोलें:

http://127.0.0.1:8080/adminer.php

> यह PHP का built-in development server है। इसे development/local use के लिए रखें; इसे public internet पर expose न करें।

## 5. Adminer में PostgreSQL login करें

Adminer login screen पर ये details भरें:

| Field | Value |
|---|---|
| System | PostgreSQL |
| Server | `127.0.0.1:5432` |
| Username | `postgres` |
| Password | PostgreSQL installation के समय सेट किया गया password |
| Database | `postgres` |

फिर **Login** पर क्लिक करें।

सफल login के बाद database और tables manage कर सकते हैं।

## 6. अगर समस्या आए तो

### `pdo_pgsql` या `pgsql` नहीं दिख रहा

- `C:\xampp\php\php.ini` में दोनों extensions enable करें।
- ऊपर दिया गया `php -m` command फिर चलाएँ।
- जाँचें कि आप उसी PHP executable और उसी `php.ini` को इस्तेमाल कर रहे हैं।

### PostgreSQL connection नहीं हो रहा

PowerShell में:

```powershell
Get-Service *postgres*
Test-NetConnection 127.0.0.1 -Port 5432
```

- PostgreSQL service का status `Running` होना चाहिए।
- `TcpTestSucceeded` में `True` आना चाहिए।
- यदि port अलग configure किया है, तो command और Adminer दोनों में सही port डालें।
- Password सही होना चाहिए।

### Port `8080` इस्तेमाल में है

किसी दूसरे local port, जैसे `8081`, पर server चलाएँ:

```powershell
C:\xampp\php\php.exe -S 127.0.0.1:8081 -t C:\adminer
```

फिर खोलें:

http://127.0.0.1:8081/adminer.php

### `MySQL server has gone away` दिख रहा है

- Adminer login screen पर **System** में `PostgreSQL` चुनें, `MySQL` नहीं।
- Server `127.0.0.1:5432` रखें।
- सुनिश्चित करें कि PostgreSQL service चल रही है और PHP में `pdo_pgsql` तथा `pgsql` enabled हैं।

### Browser में Adminer नहीं खुल रहा

- PHP server वाला PowerShell terminal खुला है या नहीं, जाँचें।
- `C:\adminer\adminer.php` मौजूद है या नहीं, जाँचें।
- Browser में सही URL और port खोलें।

## 7. Modern White CSS theme (वैकल्पिक)

यदि Adminer के लिए custom white theme चाहिए, तो `C:\adminer\adminer.css` नाम की CSS file बना सकते हैं। CSS customization का तरीका आपके Adminer version/design setup पर निर्भर हो सकता है; file बनाने के बाद browser Developer Tools में जाँचें कि stylesheet वास्तव में load हो रही है। केवल file मौजूद होने से यह मानकर न चलें कि theme लागू हो गई है।

## 8. सुरक्षा नोट्स

- PostgreSQL superuser password सुरक्षित रखें; उसे source code या logs में plain text में न रखें।
- PHP built-in server को `127.0.0.1` पर bind रखें, ताकि वह केवल local machine पर उपलब्ध हो।
- Adminer को बिना authentication या access restrictions के public internet पर expose न करें।
- Production use के लिए PHP development server के बजाय उचित web-server configuration और security hardening करें।

## Quick checklist

- [ ] PostgreSQL install है और service `Running` है।
- [ ] `psql` से PostgreSQL login सफल है।
- [ ] PHP में `pdo_pgsql` और `pgsql` enabled हैं।
- [ ] Adminer file `C:\adminer\adminer.php` पर मौजूद है।
- [ ] PHP server `127.0.0.1:8080` पर चल रहा है।
- [ ] Adminer में System `PostgreSQL` चुनकर login सफल है।




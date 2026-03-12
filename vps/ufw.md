# VPS Firewall (UFW) Issue – Troubleshooting Guide

## समस्या का सार

मेरे VPS server पर database और Adminer access करते समय **Connection timed out** error आ रहा था।
पहले सब कुछ सही काम कर रहा था, लेकिन कुछ server configuration बदलने के बाद database connect नहीं हो रहा था।

मुख्य कारण firewall (UFW) और port rules थे।

---

# Server Environment

* Server: Linux VPS
* Containers: Docker
* Reverse Proxy: Traefik
* Database: MySQL
* DB Management Tool: Adminer
* Firewall: UFW

---

# मुख्य समस्या

Adminer login page खुल रहा था लेकिन database connect करते समय यह error आ रहा था:

```
Connection timed out
```

Database port:

```
3301
```

---

# Troubleshooting Steps

## 1. Docker Containers Check किया

सबसे पहले verify किया कि containers चल रहे हैं।

```
docker ps
```

इससे confirm हुआ कि database container running है।

---

## 2. Firewall Status Check किया

```
sudo ufw status
```

कभी-कभी firewall inactive भी हो सकता है।

```
Status: inactive
```

या active होने पर rules दिखते हैं।

---

## 3. Database Port Allow करना

Database access के लिए port allow करना जरूरी है।

```
sudo ufw allow 3301/tcp
```

यह command TCP protocol के लिए port 3301 खोल देती है।

---

## 4. Specific IP Allow करना (More Secure)

अगर database public access के लिए नहीं है तो specific IP allow करना बेहतर है।

```
sudo ufw allow from YOUR_IP to any port 3301
```

यह केवल उसी IP को database access देगा।

Example:

```
sudo ufw allow from 49.xx.xx.xx to any port 3301
```

---

# Useful UFW Commands

## Firewall Enable

```
sudo ufw enable
```

---

## Firewall Disable

```
sudo ufw disable
```

---

## Firewall Status

```
sudo ufw status
```

---

## Port Allow

```
sudo ufw allow 3301/tcp
```

---

## Port Remove

```
sudo ufw delete allow 3301/tcp
```

---

## Specific IP Allow

```
sudo ufw allow from YOUR_IP to any port 3301
```

---

# Important Security Notes

Production server पर यह ports normally open होने चाहिए:

| Port | Purpose   |
| ---- | --------- |
| 22   | SSH Login |
| 80   | HTTP      |
| 443  | HTTPS     |

Database ports public internet पर open रखना risky हो सकता है।

Better approach:

* database port private network में रखें
* या specific IP allow करें

---

# Final Fix

Issue fix हुआ जब:

1. Firewall rules verify किए गए
2. Database port allow किया गया
3. Network configuration confirm किया गया

इसके बाद Adminer से database successfully connect हो गया।

---

# Key Learning

* Firewall rules database connectivity को affect कर सकते हैं
* Port allow करना जरूरी होता है
* Production servers में minimal ports open रखने चाहिए
* Specific IP allow करना सबसे secure तरीका है

---

# Author Notes

यह troubleshooting guide VPS पर database connection issues और firewall configuration समझने के लिए बनाई गई है।



# UFW Firewall Guide (Beginner से Advanced) – Linux VPS Security

## परिचय

**UFW (Uncomplicated Firewall)** Linux servers के लिए एक आसान firewall tool है जो server के ports और network access को control करता है।

इसका मुख्य काम है:

* Server को unauthorized access से बचाना
* केवल जरूरी ports को open रखना
* unwanted traffic को block करना

UFW mainly **iptables** का simplified version है।

---

# Firewall क्यों जरूरी है

अगर firewall नहीं है तो:

* कोई भी server ports scan कर सकता है
* database hack हो सकता है
* brute-force attack हो सकता है
* malware access मिल सकता है

इसलिए production server पर firewall होना जरूरी है।

---

# UFW Install करना

Ubuntu / Debian servers में अक्सर UFW पहले से installed होता है।

Check करने के लिए:

```
sudo ufw status
```

अगर installed नहीं है:

```
sudo apt update
sudo apt install ufw
```

---

# UFW Status Check

```
sudo ufw status
```

Possible result:

```
Status: inactive
```

या

```
Status: active
```

---

# Firewall Enable करना

⚠️ Important: Enable करने से पहले SSH allow करना जरूरी है।

```
sudo ufw allow 22
```

फिर firewall enable करें:

```
sudo ufw enable
```

---

# Firewall Disable करना

```
sudo ufw disable
```

---

# Basic Rules (Beginner)

## SSH Allow

```
sudo ufw allow 22
```

या

```
sudo ufw allow ssh
```

---

## HTTP Allow

```
sudo ufw allow 80
```

---

## HTTPS Allow

```
sudo ufw allow 443
```

---

## Specific Port Allow

Example: database port

```
sudo ufw allow 3301/tcp
```

---

# Port Block करना

```
sudo ufw deny 3301
```

---

# Rule Delete करना

```
sudo ufw delete allow 3301
```

---

# Current Rules देखना

```
sudo ufw status numbered
```

Example output:

```
[1] 22 ALLOW Anywhere
[2] 80 ALLOW Anywhere
[3] 443 ALLOW Anywhere
```

Rule delete:

```
sudo ufw delete 2
```

---

# Specific IP Allow करना (Advanced Security)

अगर आप database public internet से hide रखना चाहते हैं।

```
sudo ufw allow from 49.xxx.xxx.xxx to any port 3301
```

इससे केवल वही IP access कर पाएगा।

---

# Specific IP Block करना

```
sudo ufw deny from 49.xxx.xxx.xxx
```

---

# Subnet Allow करना

Example:

```
sudo ufw allow from 192.168.1.0/24
```

---

# Rate Limiting (SSH Attack Protection)

SSH brute-force attack रोकने के लिए:

```
sudo ufw limit ssh
```

---

# Default Policies

Default firewall behaviour set करना।

```
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Meaning:

* incoming traffic block
* outgoing traffic allowed

---

# Logs Enable करना

Firewall logs enable करें:

```
sudo ufw logging on
```

Logs location:

```
/var/log/ufw.log
```

---

# Firewall Reset करना

अगर configuration खराब हो जाए:

```
sudo ufw reset
```

यह सभी rules remove कर देगा।

---

# Docker और UFW

अगर server पर Docker use कर रहे हैं तो ध्यान रखें:

* Docker ports automatically open कर सकता है
* firewall rules bypass भी हो सकते हैं

Check ports:

```
docker ps
```

---

# Production Server Recommended Rules

Typical secure setup:

```
sudo ufw allow 22
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
```

अगर database external access चाहिए:

```
sudo ufw allow from YOUR_IP to any port 3301
```

---

# Common Mistakes

### 1. SSH allow किए बिना firewall enable करना

Server lock हो सकता है।

### 2. Database port public open करना

Security risk।

### 3. Rules check नहीं करना

Always verify:

```
sudo ufw status
```

---

# Best Security Practices

Production server पर:

* केवल जरूरी ports open रखें
* database public internet पर expose न करें
* SSH port change करें
* fail2ban use करें
* strong passwords रखें

---

# Quick Command Summary

| Task        | Command                              |
| ----------- | ------------------------------------ |
| Status      | `sudo ufw status`                    |
| Enable      | `sudo ufw enable`                    |
| Disable     | `sudo ufw disable`                   |
| Allow port  | `sudo ufw allow 3301`                |
| Delete rule | `sudo ufw delete allow 3301`         |
| Allow IP    | `sudo ufw allow from IP to any port` |
| Reset       | `sudo ufw reset`                     |

---

# Conclusion

UFW एक simple लेकिन powerful firewall tool है जो Linux servers को secure रखने में मदद करता है।

अगर सही तरीके से configure किया जाए तो:

* unauthorized access block हो जाता है
* server security काफी improve हो जाती है

Production server security के लिए UFW basic लेकिन बहुत important layer है।


# Linux VPS Firewall Safety Guide

## (Firewall Enable करने से पहले SSH Allow क्यों जरूरी है)

---

# परिचय

जब हम Linux VPS पर **UFW firewall enable** करते हैं तो server incoming connections को filter करना शुरू कर देता है।
अगर सही rules पहले से configure नहीं किए गए तो server access **lock** हो सकता है।

सबसे common mistake होती है:

**Firewall enable करने से पहले SSH allow न करना।**

---

# SSH क्या होता है

**SSH (Secure Shell)** एक protocol है जिससे हम remote server को control करते हैं।

जब आप VPS login करते हैं:

```
ssh root@server-ip
```

तो connection **port 22** से establish होता है।

Default SSH port:

```
22
```

---

# Firewall Enable करते समय क्या होता है

जब आप firewall enable करते हैं:

```
sudo ufw enable
```

तो firewall incoming traffic को control करना शुरू कर देता है।

अगर कोई rule defined नहीं है तो default policy अक्सर होती है:

```
deny incoming
```

मतलब:

सभी incoming connections block हो जाते हैं।

---

# Problem Scenario

अगर आपने यह किया:

```
sudo ufw enable
```

लेकिन SSH allow नहीं किया तो:

* SSH connection terminate हो जाएगा
* server से disconnect हो जाओगे
* दुबारा login नहीं कर पाओगे

Error:

```
Connection timed out
```

---

# Correct Safe Method

Firewall enable करने का safe तरीका यह है:

### Step 1 — SSH Allow करें

```
sudo ufw allow 22
```

या

```
sudo ufw allow ssh
```

---

### Step 2 — Firewall Enable करें

```
sudo ufw enable
```

---

### Step 3 — Rules Verify करें

```
sudo ufw status
```

Expected output:

```
22 ALLOW Anywhere
```

---

# Production Server Recommended Rules

Production server के लिए minimum rules:

```
sudo ufw allow 22
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
```

Meaning:

| Port | Purpose       |
| ---- | ------------- |
| 22   | SSH login     |
| 80   | HTTP website  |
| 443  | HTTPS website |

---

# Example Output

```
Status: active

To                         Action      From
--                         ------      ----
22                         ALLOW       Anywhere
80                         ALLOW       Anywhere
443                        ALLOW       Anywhere
```

---

# Extra Security (Advanced)

SSH को और secure करने के लिए:

### Rate limiting enable करें

```
sudo ufw limit ssh
```

यह brute-force attacks को reduce करता है।

---

# अगर SSH block हो जाए तो क्या करें

अगर गलती से SSH block हो जाए तो server access के लिए:

* VPS provider console
* rescue mode
* web console login

use करना पड़ता है।

फिर firewall rules fix किए जाते हैं।

---

# Common Mistakes

### 1️⃣ SSH allow किए बिना firewall enable करना

Server lock हो सकता है।

---

### 2️⃣ Database ports public open करना

Security risk बढ़ जाता है।

---

### 3️⃣ Firewall status check न करना

हमेशा verify करें:

```
sudo ufw status
```

---

# Best Practice

Firewall enable करने से पहले हमेशा:

```
sudo ufw allow 22
```

यह rule server access को सुरक्षित रखता है।

---

# Quick Summary

| Step            | Command             |
| --------------- | ------------------- |
| SSH allow       | `sudo ufw allow 22` |
| Firewall enable | `sudo ufw enable`   |
| Status check    | `sudo ufw status`   |

---

# Conclusion

Firewall server security के लिए बहुत जरूरी है, लेकिन इसे enable करने से पहले सही rules configure करना भी उतना ही जरूरी है।

सबसे important rule:

**SSH allow करना।**

अगर यह rule नहीं होगा तो server access permanently block हो सकता है।

# UFW Firewall Quick Guide (Linux VPS)

## 1️⃣ Firewall क्या करता है

Firewall server के **incoming connections control** करता है।
यह decide करता है कि कौन-सा port open रहेगा और कौन-सा block।

---

# 2️⃣ सबसे बड़ी गलती

Firewall enable करने से पहले **SSH allow न करना**।

अगर ऐसा किया तो:

* server से disconnect हो जाओगे
* दुबारा login नहीं कर पाओगे
* error आएगा:

```
Connection timed out
```

---

# 3️⃣ SSH क्यों जरूरी है

VPS server में login करने के लिए SSH use होता है।

Default port:

```
22
```

इसलिए firewall enable करने से पहले **port 22 allow करना जरूरी है**।

---

# 4️⃣ Safe तरीका (Important)

Step 1

```
sudo ufw allow 22
```

Step 2

```
sudo ufw enable
```

Step 3

```
sudo ufw status
```

---

# 5️⃣ Production Server Basic Rules

```
sudo ufw allow 22
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
```

| Port | Purpose       |
| ---- | ------------- |
| 22   | SSH Login     |
| 80   | Website HTTP  |
| 443  | Website HTTPS |

---

# 6️⃣ Extra Security

SSH brute force attack रोकने के लिए:

```
sudo ufw limit ssh
```

---

# 7️⃣ Firewall Status Check

```
sudo ufw status
```

---

# 8️⃣ अगर SSH block हो जाए

Server access करने के लिए:

* VPS console
* rescue mode
* provider panel

use करना पड़ता है।

---

# 9️⃣ Golden Rule

⚠️ **Firewall enable करने से पहले हमेशा SSH allow करो**

```
sudo ufw allow 22
```

---

# Quick Commands

```
sudo ufw allow 22
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
sudo ufw status
```

---

✅ यह VPS firewall का **safe basic setup** है।



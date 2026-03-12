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

# Authentication Architecture (JWT + Access Token + Refresh Token + Device Sessions)

> Production Ready Authentication Flow for Android, iOS & Web Applications

---

# Overview

Authentication Architecture:

```text
                +----------------------+
                |      Android App     |
                +----------------------+
                          │
                          ▼
                Access Token (45 min)
                Refresh Token (90 Days)
                          │
                          ▼
                 Authentication API
                          │
        ┌─────────────────┴─────────────────┐
        │                                   │
        ▼                                   ▼
   PostgreSQL                          Redis Cache
(users, device_sessions)           (Session Cache)
```

---

# Components

## 1. Users Table

Stores permanent user information.

```sql
users
-----
id
name
email
phone
password_hash
status
created_at
updated_at
```

---

## 2. Device Sessions Table

Stores login sessions for every device.

```sql
device_sessions
------------------------
id
user_id
device_id
refresh_token_hash
status
expires_at
last_seen_at
created_at
updated_at
```

Status

```
ACTIVE
REVOKED
```

---

# Token Configuration

| Token         | Validity   | Purpose                   |
| ------------- | ---------- | ------------------------- |
| Access Token  | 45 Minutes | API Authentication        |
| Refresh Token | 90 Days    | Generate New Access Token |

Refresh Token uses **Rotation**.

Meaning:

Every refresh request generates

* New Access Token
* New Refresh Token

Old Refresh Token becomes invalid immediately.

---

# Complete Authentication Story

---

# Step 1

## User Opens App

App shows Login Screen.

User enters

```
Email
Password
```

Clicks

```
Login
```

---

# Step 2

Backend Receives Request

```
POST /auth/login
```

Request

```json
{
    "email":"john@gmail.com",
    "password":"********",
    "device_id":"ANDROID-DEVICE-001"
}
```

---

# Step 3

Backend Verifies User

```
users
```

Search user by email.

If user not found

```
401 Unauthorized
```

If password mismatch

```
401 Unauthorized
```

Otherwise

```
Login Success
```

---

# Step 4

Create Device Session

Backend creates session.

```
device_sessions
```

```
id = SESSION_001

user_id = 101

device_id = ANDROID-DEVICE-001

refresh_token_hash = hash(R1)

status = ACTIVE

expires_at = 90 Days

last_seen_at = Current Time
```

---

# Step 5

Cache Session in Redis

```
Redis

session:SESSION_001

status=ACTIVE

user_id=101
```

Redis is only a cache.

PostgreSQL remains Source of Truth.

---

# Step 6

Generate Tokens

Backend generates

```
Access Token
```

Validity

```
45 Minutes
```

And

```
Refresh Token
```

Validity

```
90 Days
```

Response

```json
{
    "access_token":"A1",
    "refresh_token":"R1"
}
```

---

# Step 7

App Stores Tokens

App securely stores

```
Access Token

Refresh Token
```

---

# Step 8

User Calls APIs

Example

```
GET /profile

GET /rides

POST /ride/book
```

Authorization Header

```
Bearer A1
```

Backend

```
Verify JWT

↓

Valid

↓

Execute API
```

No database lookup required.

JWT verification is enough.

---

# Step 9

Access Token Expires

45 minutes later

```
Access Token Expired
```

App receives

```
401 Unauthorized
```

Automatically calls

```
POST /auth/refresh
```

Request

```json
{
    "refresh_token":"R1"
}
```

---

# Step 10

Refresh Flow

Backend

First

```
Redis
```

Search Session.

```
session:SESSION_001
```

## Cache Hit

Session found.

Verify

```
status == ACTIVE
```

Verify

```
refresh_token_hash
```

Matches

```
hash(R1)
```

Everything valid.

---

## Cache Miss

If Redis doesn't contain session

↓

Read from PostgreSQL

↓

Load Session

↓

Store again in Redis

↓

Continue authentication.

---

# Step 11

Generate New Tokens

Backend creates

```
Access Token = A2

Refresh Token = R2
```

Updates PostgreSQL

```
refresh_token_hash = hash(R2)

expires_at = now + 90 days

last_seen_at = now
```

Updates Redis

Returns

```json
{
    "access_token":"A2",
    "refresh_token":"R2"
}
```

Old Refresh Token

```
R1
```

becomes

```
INVALID
```

---

# Step 12

Rotation

```
Login

A1 + R1

↓

45 Minutes

↓

A2 + R2

↓

45 Minutes

↓

A3 + R3

↓

45 Minutes

↓

A4 + R4

↓

45 Minutes

↓

A5 + R5
```

Every refresh replaces previous Refresh Token.

---

# Step 13

User Uses App Daily

Every

```
45 Minutes
```

App silently refreshes token.

User never sees login screen.

---

# Step 14

User Opens App After 20 Days

Refresh Token

Still valid.

Backend generates

```
A20

R20
```

No Login Required.

---

# Step 15

User Opens App After 95 Days

Scenario

User never opened app during last 95 days.

Refresh Token

Expired.

Backend

```
401 Unauthorized
```

App redirects

```
Login Screen
```

---

# Step 16

Logout

```
POST /logout
```

Backend

PostgreSQL

```
status = REVOKED
```

Redis

```
Delete Session Cache
```

Future refresh requests

```
401 Unauthorized
```

---

# Redis Usage

Redis is **NOT** the primary storage.

Redis is only a cache.

Used for

✅ Refresh Token Verification

✅ Session Lookup

✅ Last Seen Update

✅ Logout

Not used for

❌ Every API Call

❌ JWT Verification

---

# JWT Payload

Access Token contains only minimal information.

```json
{
    "sub":"101",
    "session_id":"SESSION_001",
    "role":"USER",
    "iat":123456789,
    "exp":123456999
}
```

Never store

* Password
* Refresh Token
* Email
* Phone
* Wallet Balance

inside JWT.

---

# Why Refresh Token Hash?

Database stores

```
hash(R1)
```

Not

```
R1
```

If database leaks

Attacker cannot use Refresh Token.

---

# Why JWT?

* Stateless
* Fast
* No Database Hit
* Scalable

---

# Why Refresh Token?

* Long Login
* Silent Authentication
* Better User Experience

---

# Why Device Sessions?

* Multiple Devices
* Logout Single Device
* Logout All Devices
* Session Tracking
* Token Revocation

---

# Why Redis?

* Faster Session Lookup
* Lower Database Load
* Better Performance

---

# Final Architecture

```text
                        Login
                          │
                          ▼
                 Verify User (PostgreSQL)
                          │
                          ▼
               Create Device Session
                          │
                          ▼
           Save Refresh Hash (PostgreSQL)
                          │
                          ▼
             Cache Session (Redis)
                          │
                          ▼
     Generate Access + Refresh Tokens
                          │
                          ▼
                    Return Tokens
                          │
                          ▼
                  Protected APIs
                          │
                   Verify JWT Only
                          │
                          ▼
                 Access Token Expires
                          │
                          ▼
                  /auth/refresh
                          │
                          ▼
               Redis → PostgreSQL
                          │
                          ▼
        Verify Refresh Token & Session
                          │
                          ▼
      Generate New Access + Refresh
                          │
                          ▼
 Update Refresh Hash + Redis Cache
                          │
                          ▼
              Continue Using App
                          │
                          ▼
                     User Logout
                          │
                          ▼
       Revoke Session + Delete Cache
```

---

# Final Production Stack

* JWT Access Token (45 Minutes)
* Rotating Refresh Token (90 Days)
* PostgreSQL (Users + Device Sessions)
* Redis (Session Cache)
* Refresh Token Hashing
* Device-based Login Sessions
* Multi-device Support
* Logout from Single Device
* Logout from All Devices
* Production Ready Architecture

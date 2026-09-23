# Authentication & Authorization — Tables Purpose

यह पूरा system दो मुख्य हिस्सों में बंटा है:

* **Authentication (Auth)** → User कौन है?
* **Authorization (RBAC)** → User क्या कर सकता है?

---

## 1. `users`

**Purpose:** User की identity/account रखने के लिए।

इसमें user की basic information रहती है।

```text
users
```

उदाहरण:

```text
User ID
Name
Email
Password Hash
Status
```

**Simple meaning:**

> "यह user कौन है?"

---

## 2. `sessions`

**Purpose:** User के login sessions manage करने के लिए।

एक user के multiple devices/browser sessions हो सकते हैं।

```text
users
  ↓
sessions
```

**Simple meaning:**

> "यह user अभी किन login sessions में active है?"

---

# RBAC (Role-Based Access Control)

## 3. `roles`

**Purpose:** Users को logical groups/roles में रखने के लिए।

उदाहरण:

```text
Admin
Manager
Editor
Viewer
```

**Simple meaning:**

> "User किस role में है?"

---

## 4. `permissions`

**Purpose:** System में छोटे-छोटे allowed actions define करने के लिए।

उदाहरण:

```text
user.read
user.create
user.update
user.delete

post.read
post.create
post.update
post.delete
```

**Simple meaning:**

> "System में कौन-कौन से actions possible/allowed हैं?"

---

## 5. `user_roles`

**Purpose:** User और Role के बीच relationship रखने के लिए।

```text
user_roles

user_id → role_id
```

उदाहरण:

```text
Neeraj → Admin
Rahul  → Editor
Amit   → Viewer
```

**Simple meaning:**

> "किस user को कौन-सा role मिला है?"

---

## 6. `role_permissions`

**Purpose:** Role को permissions देने के लिए।

```text
role_permissions

role_id → permission_id
```

उदाहरण:

```text
Admin
 ├── user.read
 ├── user.create
 ├── user.update
 └── user.delete
```

**Simple meaning:**

> "इस role को क्या-क्या करने की permission है?"

---

# API Authorization

## 7. `access_policies`

**Purpose:** यह define करने के लिए कि कौन-सी API/route को कौन-सी permission चाहिए।

Example:

```text
GET    /users       → user.read
POST   /users       → user.create
DELETE /users/:id   → user.delete
```

इसके मुख्य fields:

```text
type
method
matcher
permission_id
priority
effect
```

**Simple meaning:**

> "किस API/route पर कौन-सी permission required है?"

इसलिए `access_policies` और `role_permissions` अलग हैं।

```text
role_permissions
→ Role को permission देता है

access_policies
→ API/route को permission से protect करता है
```

---

# Admin UI

## 8. `sidebar_items`

**Purpose:** Admin/dashboard में कौन-से sidebar menu दिखाई देंगे और किस order/hierarchy में दिखाई देंगे, यह manage करने के लिए।

Example:

```text
Dashboard

Users
 ├── All Users
 └── Roles

Posts
 ├── All Posts
 └── Categories

Settings
```

यह **security permission नहीं है**।

यह सिर्फ UI/navigation है।

```text
sidebar_items
→ UI में क्या दिखाना है?

permissions
→ User को वास्तव में क्या करने देना है?
```

---

# पूरा System एक साथ

```text
                    AUTHENTICATION

                    ┌───────────┐
                    │   users   │
                    └─────┬─────┘
                          │
                    ┌─────▼─────┐
                    │ sessions  │
                    └───────────┘


                    AUTHORIZATION

                    ┌───────────┐
                    │   roles   │
                    └─────┬─────┘
                          │
                    ┌─────▼───────────┐
                    │ role_permissions│
                    └─────┬───────────┘
                          │
                    ┌─────▼──────────┐
                    │  permissions   │
                    └────────────────┘

users
  │
  └──────→ user_roles ──────→ roles


                    API SECURITY

                    ┌─────────────────┐
                    │ access_policies │
                    └────────┬────────┘
                             │
                    API/Route → Permission


                    ADMIN UI

                    ┌───────────────┐
                    │ sidebar_items │
                    └───────────────┘
```

# सबसे आसान याद रखने वाला Formula

```text
users
→ User कौन है?

sessions
→ User का login session कौन-सा है?

roles
→ User किस group/role में है?

user_roles
→ किस User को कौन-सा Role मिला?

permissions
→ कौन-कौन से actions available हैं?

role_permissions
→ किस Role को कौन-कौन से actions मिले?

access_policies
→ कौन-सी API को कौन-सी permission चाहिए?

sidebar_items
→ Admin UI में कौन-सा menu दिखाना है?
```

# Important

Normal RBAC के लिए:

```text
users
sessions
roles
permissions
user_roles
role_permissions
```

ये core tables हैं।

`access_policies` API/route authorization के लिए है।

`sidebar_items` UI/navigation के लिए है।

`user_permissions` अभी आवश्यक नहीं है। इसे तभी जोड़ना है जब किसी **specific user को उसके role से अलग direct permission/override** देना हो।

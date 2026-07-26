# Template Literal Types for Permission Generation

## Problem

हमारे सिस्टम में Permissions को Manual तरीके से define किया जा रहा था।

```ts
type Permission =
  | 'user:read'
  | 'user:write'
  | 'order:read'
  | 'order:write'
  | 'product:read'
  | 'product:write'
```

### Issues

- हर Permission manually लिखनी पड़ती है।
- Code में duplication (Repeat) होता है।
- Typo होने की संभावना रहती है।
- नई Entity जोड़ने पर सभी permissions manually add करनी पड़ती हैं।
- नई Action जोड़ने पर सभी Entities के लिए permissions लिखनी पड़ती हैं।
- Maintain करना कठिन हो जाता है।

---

## Solution

Permissions को Template Literal Types की मदद से generate किया जाए।

```ts
type Entity =
  | 'user'
  | 'order'
  | 'product'

type Action =
  | 'read'
  | 'write'

type Permission = `${Entity}:${Action}`
```

TypeScript compile time पर सभी combinations generate कर देता है।

```ts
type Permission =
  | 'user:read'
  | 'user:write'
  | 'order:read'
  | 'order:write'
  | 'product:read'
  | 'product:write'
```

---

## Benefits

### ✅ Single Source of Truth

Entities और Actions केवल एक जगह define होते हैं।

```ts
type Entity = ...
type Action = ...
```

---

### ✅ DRY Principle

बार-बार Permissions लिखने की आवश्यकता नहीं रहती।

---

### ✅ Automatic Permission Generation

नई Entity जोड़ें—

```ts
type Entity =
  | 'user'
  | 'order'
  | 'product'
  | 'category'
```

Automatically:

```text
category:read
category:write
```

generate हो जाएंगे।

---

### ✅ Easy Action Expansion

यदि नई Action जोड़ते हैं—

```ts
type Action =
  | 'read'
  | 'write'
  | 'delete'
```

तो Automatically:

```text
user:delete
order:delete
product:delete
```

भी generate हो जाएंगे।

---

### ✅ Type Safety

Invalid Permission compile-time पर ही reject हो जाती है।

```ts
const p: Permission = 'user:update'
```

❌ TypeScript Error

---

### ✅ Better Maintainability

Code छोटा, साफ़ और maintain करना आसान हो जाता है।

---

## Comparison

### Manual Approach

```text
Entity × Action = Manual Permissions
```

हर combination developer को लिखना पड़ता है।

---

### Template Literal Approach

```text
Entity + Action
        │
        ▼
`${Entity}:${Action}`
        │
        ▼
TypeScript Automatically Generates
All Valid Permissions
```

---

## Conclusion

Template Literal Types का उपयोग करने से:

- Boilerplate Code कम होता है।
- DRY Principle follow होता है।
- Type Safety मिलती है।
- Typos कम होते हैं।
- Future Scaling आसान हो जाती है।
- Code अधिक Maintainable और Enterprise Friendly बनता है।

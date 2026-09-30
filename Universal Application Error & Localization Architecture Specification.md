# Universal Application Error & Localization Architecture Specification

**Version:** 1.0  
**Status:** Recommended Architecture Standard  
**Scope:** Language & Framework Independent  
**Purpose:** Universal Error Handling, Localization, API Contract and AI Coding Standard

---

# 1. परिचय

यह document किसी एक programming language, framework, database या platform के लिए नहीं है।

इसका उद्देश्य एक ऐसा **universal application error architecture** define करना है जिसे किसी भी modern software project में लागू किया जा सके।

यह architecture निम्न प्रकार के systems में उपयोग किया जा सकता है:

- Mobile Applications
- Web Applications
- Backend APIs
- Admin Panels
- Desktop Applications
- Serverless Applications
- Monolithic Applications
- Modular Monoliths
- Microservices
- Multi-client Applications
- Multi-language Applications
- AI-assisted Software Projects

इस architecture का मुख्य उद्देश्य है:

```text
Consistent Errors
        +
Stable Error Codes
        +
Structured Error Data
        +
Localization
        +
Security
        +
Backward Compatibility
        +
AI-friendly Development
```

---

# 2. मुख्य समस्या

छोटे projects में error handling अक्सर इस प्रकार शुरू होती है:

```text
"User not found"
"User does not exist"
"No user found"
"user missing"
```

अलग-अलग developers अलग-अलग messages लिखते हैं।

कहीं:

```json
{
  "message": "User not found"
}
```

कहीं:

```json
{
  "error": "Invalid OTP"
}
```

और कहीं:

```json
{
  "status": false,
  "message": "Something went wrong"
}
```

समय के साथ system inconsistent हो जाता है।

इसके अलावा multilingual application में समस्या और बढ़ जाती है।

उदाहरण:

```text
English:
User not found

Hindi:
उपयोगकर्ता नहीं मिला

Bengali:
ব্যবহারকারী পাওয়া যায়নি
```

यदि business logic में सीधे messages लिखे गए हैं तो language बदलना कठिन हो जाता है।

इसलिए application में **Error Identity और User Presentation को अलग करना आवश्यक है।**

---

# 3. Core Principle

पूरे architecture का सबसे महत्वपूर्ण principle:

> **Error Identity, Error Metadata और User Presentation को अलग-अलग concerns माना जाए।**

उदाहरण:

```text
ERROR IDENTITY
    USER_NOT_FOUND

        ↓

ERROR METADATA
    category = NOT_FOUND
    message_key = errors.user_not_found
    retryable = false

        ↓

TRANSPORT
    HTTP = 404

        ↓

PRESENTATION
    English = User not found
    Hindi   = उपयोगकर्ता नहीं मिला
```

इस separation के कारण:

- Business logic language-independent रहता है।
- Frontend message text पर निर्भर नहीं होता।
- Backend transport-independent रह सकता है।
- नई languages जोड़ना आसान होता है।
- AI generated code को standard rules दिए जा सकते हैं।

---

# 4. Error Architecture के मुख्य Components

एक mature application में error system के निम्न components होने चाहिए:

```text
Error System
│
├── Error Code
├── Error Definition
├── Error Category
├── Message Key
├── Localization
├── Error Details
├── Original Cause
├── Transport Mapping
├── Public/Private Visibility
├── Severity
├── Retryability
├── Logging
├── Request/Trace ID
└── API Response Contract
```

हर component का अलग responsibility होना चाहिए।

---

# 5. Error Code

## 5.1 Definition

Error Code एक **machine-readable stable identifier** है।

उदाहरण:

```text
VALIDATION_ERROR
UNAUTHORIZED
FORBIDDEN
NOT_FOUND
CONFLICT
USER_NOT_FOUND
ORDER_NOT_FOUND
INVALID_OTP
OTP_EXPIRED
PAYMENT_FAILED
```

---

## 5.2 Error Code के Rules

Error Code:

- stable होना चाहिए।
- language-independent होना चाहिए।
- user-facing sentence नहीं होना चाहिए।
- predictable naming convention follow करना चाहिए।
- API contract का हिस्सा माना जा सकता है।
- frontend/business logic द्वारा safely consume किया जा सके।
- unnecessary duplication नहीं करनी चाहिए।

---

## 5.3 सही उदाहरण

```text
USER_NOT_FOUND
```

## गलत उदाहरण

```text
USER_NOT_FOUND_HINDI
USER_NOT_FOUND_MESSAGE
USER_NOT_FOUND_TEXT
USER_NOT_FOUND_ENGLISH
```

---

# 6. Error Code Naming Convention

Recommended naming:

```text
UPPER_SNAKE_CASE
```

उदाहरण:

```text
VALIDATION_ERROR
AUTHENTICATION_REQUIRED
FORBIDDEN
USER_NOT_FOUND
ORDER_NOT_FOUND
INVALID_OTP
OTP_EXPIRED
PAYMENT_FAILED
SERVICE_AREA_NOT_FOUND
```

एक project में naming convention consistent रहना चाहिए।

---

# 7. Generic बनाम Domain-Specific Error Codes

दोनों प्रकार के error codes valid हैं।

## Generic

```text
VALIDATION_ERROR
UNAUTHORIZED
FORBIDDEN
NOT_FOUND
CONFLICT
```

## Domain Specific

```text
USER_NOT_FOUND
ORDER_NOT_FOUND
SERVICE_AREA_NOT_FOUND
INVALID_OTP
PAYMENT_FAILED
```

---

## 7.1 Domain-Specific Code कब बनाएं?

जब application या client को exact error type के आधार पर अलग behavior करना हो।

उदाहरण:

```text
ORDER_NOT_FOUND
```

frontend को order-specific UI दिखाने की अनुमति देता है।

लेकिन हर छोटी internal condition के लिए नया public error code बनाना आवश्यक नहीं है।

---

# 8. Error Definition

हर error code की central definition होनी चाहिए।

Conceptual structure:

```text
ErrorDefinition
│
├── code
├── message_key
├── category
├── public
├── severity
└── retryable
```

उदाहरण:

```json
{
  "code": "USER_NOT_FOUND",
  "message_key": "errors.user_not_found",
  "category": "NOT_FOUND",
  "public": true,
  "severity": "INFO",
  "retryable": false
}
```

---

# 9. Error Category

Error Category errors को broad classification देती है।

Recommended categories:

```text
VALIDATION
AUTHENTICATION
AUTHORIZATION
NOT_FOUND
CONFLICT
BUSINESS_RULE
RATE_LIMIT
TIMEOUT
DEPENDENCY
CONFIGURATION
INTERNAL
```

उदाहरण:

```text
Code:
USER_NOT_FOUND

Category:
NOT_FOUND
```

दूसरा:

```text
Code:
ORDER_NOT_FOUND

Category:
NOT_FOUND
```

Category और Code अलग concepts हैं।

---

# 10. Message Key

User-facing message को सीधे Error Code से जोड़ना आवश्यक नहीं है।

इसके बजाय message key रखी जा सकती है।

उदाहरण:

```text
Code:
USER_NOT_FOUND

Message Key:
errors.user_not_found
```

फिर translation catalogs:

```text
en → errors.user_not_found
hi → errors.user_not_found
bn → errors.user_not_found
ta → errors.user_not_found
```

---

# 11. Localization का उद्देश्य

Application को language-independent होना चाहिए।

Business logic में:

```text
User not found
```

या:

```text
उपयोगकर्ता नहीं मिला
```

hard-code नहीं होना चाहिए।

Business logic को केवल semantic information पता होनी चाहिए:

```text
USER_NOT_FOUND
```

---

# 12. Internationalization और Localization

## i18n

Internationalization का अर्थ है application को विभिन्न languages/locales के लिए तैयार करना।

## l10n

Localization का अर्थ है किसी specific language/locale के लिए actual translation और formatting देना।

Architecture दोनों को support करने योग्य होना चाहिए।

---

# 13. Translation Catalog

Translation files conceptual रूप से:

```text
locales/
├── en.json
├── hi.json
├── bn.json
├── mr.json
├── ta.json
├── te.json
├── gu.json
├── kn.json
├── ml.json
├── pa.json
└── or.json
```

हो सकती हैं।

Exact storage implementation project के अनुसार बदल सकती है।

---

# 14. English Translation Example

```json
{
  "errors.user_not_found": "User not found",
  "errors.invalid_otp": "Invalid OTP",
  "errors.otp_expired": "OTP has expired",
  "errors.payment_failed": "Payment failed",
  "errors.validation": "Validation failed"
}
```

---

# 15. Hindi Translation Example

```json
{
  "errors.user_not_found": "उपयोगकर्ता नहीं मिला",
  "errors.invalid_otp": "अमान्य OTP",
  "errors.otp_expired": "OTP की समय सीमा समाप्त हो गई है",
  "errors.payment_failed": "भुगतान विफल हो गया",
  "errors.validation": "इनपुट मान्य नहीं है"
}
```

---

# 16. Language Selection

Language selection के लिए application निम्न signals का उपयोग कर सकती है:

```text
1. Explicit user preference
2. Account/profile preference
3. Application setting
4. Request language
5. Device language
6. Browser language
7. System default
```

Recommended priority:

```text
Explicit User Preference
        ↓
Account/Application Preference
        ↓
Request Locale
        ↓
Device/Browser Locale
        ↓
Default Locale
```

Exact priority project requirements के अनुसार बदली जा सकती है।

---

# 17. Locale

Language और Locale को अलग concepts माना जाए।

उदाहरण:

```text
hi
hi-IN

en
en-IN
en-US

bn
bn-IN
```

जहाँ आवश्यक हो वहाँ locale-specific:

- Date
- Time
- Currency
- Number
- Pluralization
- Formatting

का ध्यान रखा जाए।

---

# 18. Fallback Strategy

Translation missing होने पर application fail नहीं होनी चाहिए।

Recommended flow:

```text
Requested Locale
       ↓
Translation Available?
    │
    ├── YES → Use Translation
    │
    └── NO
         ↓
    Default Locale
         ↓
    Translation Available?
         │
         ├── YES → Use Default
         │
         └── NO
              ↓
         Safe Fallback
```

उदाहरण:

```text
Requested:
hi-IN

hi-IN missing
    ↓
hi missing
    ↓
en
    ↓
safe fallback
```

---

# 19. Translation Missing होने पर Rules

Application को:

```text
panic
crash
blank message
raw translation key
```

नहीं दिखाना चाहिए।

Safe fallback message होना चाहिए।

उदाहरण:

```text
Something went wrong.
```

या project का defined default message।

---

# 20. Transport Independence

Application error को transport protocol से tightly couple नहीं करना चाहिए।

### गलत

```text
Domain
  ↓
HTTP 404
```

### बेहतर

```text
Domain
  ↓
USER_NOT_FOUND
  ↓
Application Error
  ↓
Transport Adapter
  ↓
HTTP 404
```

इससे application को future में अलग transports में reuse किया जा सकता है।

---

# 21. Transport Mapping

एक ही application error को अलग transports अलग तरीके से represent कर सकते हैं।

उदाहरण:

```text
Application Error:
USER_NOT_FOUND
```

HTTP:

```text
404
```

gRPC:

```text
NOT_FOUND
```

GraphQL:

```text
application error code
```

CLI:

```text
application-specific exit status
```

इसलिए:

```text
Application Error Code
        ≠
Transport Status Code
```

---

# 22. HTTP Status और Error Code

HTTP status और application error code अलग रखें।

एक HTTP status के अंदर कई application errors हो सकते हैं।

उदाहरण:

```text
HTTP 404
│
├── USER_NOT_FOUND
├── ORDER_NOT_FOUND
└── SERVICE_AREA_NOT_FOUND
```

और:

```text
HTTP 400
│
├── VALIDATION_ERROR
├── INVALID_OTP
└── INVALID_REQUEST
```

इसलिए frontend को केवल HTTP status देखकर business condition assume नहीं करनी चाहिए।

---

# 23. Application Error Object

Implementation language अलग हो सकती है, लेकिन conceptual structure:

```text
ApplicationError
│
├── code
├── message_key
├── category
├── details
├── cause
├── metadata
└── diagnostic/reference information
```

---

# 24. `details`

`details` structured additional information के लिए है।

Validation example:

```json
{
  "fields": {
    "phone": {
      "code": "INVALID_FORMAT"
    },
    "name": {
      "code": "REQUIRED"
    }
  }
}
```

---

# 25. Validation Errors

Validation errors को generic string की बजाय structured होना चाहिए।

Recommended:

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "details": {
      "fields": {
        "phone": {
          "code": "INVALID_FORMAT",
          "message_key": "validation.phone.invalid"
        },
        "name": {
          "code": "REQUIRED",
          "message_key": "validation.name.required"
        }
      }
    }
  }
}
```

Client अपनी localization strategy के अनुसार `message_key` से message generate कर सकता है।

---

# 26. Message और Business Logic

Business logic को message text पर निर्भर नहीं होना चाहिए।

### गलत

```text
if message == "User not found"
```

### सही

```text
if error.code == "USER_NOT_FOUND"
```

क्योंकि message बदल सकता है:

```text
User not found
```

से:

```text
We couldn't find this user.
```

लेकिन:

```text
USER_NOT_FOUND
```

stable रह सकता है।

---

# 27. Internal Cause

Application error में original cause preserve करना उपयोगी है।

Flow:

```text
Database Error
       ↓
Repository
       ↓
Application Error
       ↓
API Response
```

लेकिन database error को सीधे client को expose नहीं करना चाहिए।

---

# 28. Public Message बनाम Internal Cause

उदाहरण:

### Internal

```text
database connection failed
```

### Public

```text
INTERNAL_SERVER_ERROR
```

Client:

```json
{
  "success": false,
  "error": {
    "code": "INTERNAL_SERVER_ERROR",
    "message": "Something went wrong"
  }
}
```

Internal logs:

```text
error_code=INTERNAL_SERVER_ERROR
cause=database connection failed
```

---

# 29. Security Principle

Public error response में कभी भी बिना sanitization के expose न करें:

```text
Passwords
OTP
Access Tokens
Refresh Tokens
JWT
API Keys
Database Credentials
Stack Traces
SQL Queries
Internal Service URLs
Cloud Credentials
Private User Data
Payment Secrets
Infrastructure Information
```

---

# 30. Public और Private Errors

Error definition में visibility metadata useful हो सकता है।

उदाहरण:

```json
{
  "code": "DATABASE_CONNECTION_FAILED",
  "public": false
}
```

Application इसे internally log कर सकती है।

Client को safe error मिल सकता है:

```text
INTERNAL_SERVER_ERROR
```

---

# 31. Error Mapping

Low-level errors को public application errors में map किया जाना चाहिए।

उदाहरण:

```text
Database: no row
       ↓
Repository
       ↓
USER_NOT_FOUND
```

दूसरा:

```text
External Payment Timeout
       ↓
Payment Adapter
       ↓
PAYMENT_PROVIDER_TIMEOUT
```

फिर transport layer:

```text
PAYMENT_PROVIDER_TIMEOUT
       ↓
HTTP 503
```

---

# 32. Error Wrapping

जब error layers के बीच propagate हो तो original cause preserve किया जाए।

Concept:

```text
Public Error
    USER_NOT_FOUND

Cause
    database no-row condition
```

इससे:

- debugging आसान होती है।
- logs meaningful रहते हैं।
- root cause खोता नहीं है।
- client को internal implementation नहीं दिखाई जाती।

---

# 33. API Error Response Contract

पूरे API में एक standard structure होना चाहिए।

Recommended:

```json
{
  "success": false,
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "User not found",
    "details": null
  }
}
```

Hindi:

```json
{
  "success": false,
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "उपयोगकर्ता नहीं मिला",
    "details": null
  }
}
```

---

# 34. Request ID

Production systems में request/correlation ID उपयोगी है।

उदाहरण:

```json
{
  "success": false,
  "error": {
    "code": "INTERNAL_SERVER_ERROR",
    "message": "Something went wrong",
    "details": null
  },
  "request_id": "01JXXXXXXXX"
}
```

इससे support और debugging में मदद मिलती है।

---

# 35. Error Response के Required और Optional Fields

Recommended:

```text
success
error.code
error.message
error.details
request_id
```

Optional:

```text
error.category
error.retryable
```

लेकिन client-facing contract में केवल वही fields expose करें जिनकी वास्तविक आवश्यकता हो।

---

# 36. Error Message Parameters

Dynamic values के लिए interpolation support होना चाहिए।

Example:

```text
errors.minimum_age
```

English:

```text
Minimum age is {age}.
```

Hindi:

```text
न्यूनतम आयु {age} वर्ष है।
```

Runtime:

```text
age = 18
```

Result:

```text
न्यूनतम आयु 18 वर्ष है।
```

---

# 37. Pluralization

Localization architecture को pluralization support करने योग्य होना चाहिए।

Example:

```text
1 item
2 items
```

लेकिन सभी languages के plural rules समान नहीं होते।

इसलिए application को locale-aware translation/pluralization mechanism का उपयोग करना चाहिए।

हर language के लिए manually:

```text
if count == 1
```

जैसे rules hard-code नहीं करने चाहिए।

---

# 38. Date, Time और Number Formatting

Multilingual architecture केवल error messages तक सीमित नहीं होनी चाहिए।

Locale-aware formatting के लिए architecture future-ready होना चाहिए:

```text
Date
Time
Currency
Number
Decimal
Percentage
Plural
Units
```

लेकिन error system और general localization system को unnecessarily tightly couple नहीं करना चाहिए।

---

# 39. Server-Side Localization

Architecture:

```text
Client
  ↓
Requested Locale
  ↓
Server
  ↓
Translation
  ↓
Localized Response
```

Example:

```http
Accept-Language: hi-IN
```

Response:

```json
{
  "success": false,
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "उपयोगकर्ता नहीं मिला"
  }
}
```

---

# 40. Client-Side Localization

दूसरा approach:

```text
Server
  ↓
USER_NOT_FOUND
  ↓
Client
  ↓
Translation Catalog
  ↓
Localized UI
```

Server:

```json
{
  "success": false,
  "error": {
    "code": "USER_NOT_FOUND"
  }
}
```

Client:

```text
USER_NOT_FOUND
        ↓
Hindi
        ↓
उपयोगकर्ता नहीं मिला
```

---

# 41. Hybrid Localization

Multi-client applications के लिए hybrid model भी अपनाया जा सकता है।

उदाहरण:

```text
Server
    ↓
code
details
optional fallback message
    ↓
Client
    ↓
localized presentation
```

यह architecture को flexible रखता है।

---

# 42. कौन-सा Localization Model चुनें?

Architecture को किसी एक model पर hard-code नहीं करना चाहिए।

Decision factors:

```text
Number of clients
Translation ownership
Offline support
Native UI requirements
Server-side rendering
API consumers
Consistency requirements
```

लेकिन चाहे server localization हो या client localization:

> **Business logic को translated message से स्वतंत्र रखना अनिवार्य है।**

---

# 43. Error Registry

Large application में centralized error registry उपयोगी है।

Concept:

```text
Error Registry
│
├── VALIDATION_ERROR
├── UNAUTHORIZED
├── FORBIDDEN
├── NOT_FOUND
├── CONFLICT
├── USER_NOT_FOUND
├── ORDER_NOT_FOUND
├── INVALID_OTP
├── OTP_EXPIRED
└── PAYMENT_FAILED
```

Registry का implementation:

```text
Static Code
Configuration
Database
Generated Metadata
```

में से कोई भी हो सकता है।

लेकिन runtime behavior deterministic होना चाहिए।

---

# 44. Error Registry में Duplicate Prevention

AI और developers दोनों को नया error code बनाने से पहले existing registry inspect करनी चाहिए।

उदाहरण:

यदि पहले से:

```text
USER_NOT_FOUND
```

है तो नया:

```text
USER_DOES_NOT_EXIST
```

नहीं बनाना चाहिए यदि दोनों का semantic meaning समान है।

---

# 45. Error Code Lifecycle

Public error codes को API contract माना जाना चाहिए।

Lifecycle:

```text
NEW
 ↓
ACTIVE
 ↓
DEPRECATED
 ↓
MIGRATION PERIOD
 ↓
REMOVED
```

किसी public error code को अचानक rename/delete नहीं करना चाहिए।

---

# 46. Backward Compatibility

Existing clients इस प्रकार हो सकते हैं:

```text
Android
iOS
Web
Admin
Third-party API consumer
```

यदि error code बदल दिया गया:

```text
USER_NOT_FOUND
```

से:

```text
USER_MISSING
```

तो पुराने clients टूट सकते हैं।

इसलिए public error codes के लिए compatibility policy होनी चाहिए।

---

# 47. Error Code Versioning

यदि error behavior बदलता है तो project को evaluate करना चाहिए:

```text
Is code still semantically valid?
        │
        ├── Yes
        │    ↓
        │ Keep same code
        │
        └── No
             ↓
        Introduce new code
             ↓
        Deprecate old code
```

---

# 48. Error Severity

Internal observability के लिए severity उपयोगी हो सकती है:

```text
DEBUG
INFO
WARNING
ERROR
CRITICAL
```

उदाहरण:

```text
USER_NOT_FOUND
severity = INFO
```

और:

```text
DATABASE_UNAVAILABLE
severity = ERROR
```

Severity user-facing message नहीं है।

---

# 49. Retryability

कुछ errors retry योग्य हो सकते हैं:

```text
TIMEOUT
TEMPORARY_DEPENDENCY_FAILURE
RATE_LIMITED
SERVICE_UNAVAILABLE
```

कुछ generally retry योग्य नहीं होते:

```text
VALIDATION_ERROR
FORBIDDEN
USER_NOT_FOUND
INVALID_OTP
```

इसलिए metadata में:

```text
retryable = true/false
```

रखा जा सकता है।

लेकिन automatic retry केवल इस field के आधार पर blindly नहीं करना चाहिए।

इसके साथ:

```text
Backoff
Retry limit
Idempotency
Request semantics
```

का भी ध्यान रखना चाहिए।

---

# 50. Logging

Error system को logging system के साथ integrate किया जा सकता है।

Useful fields:

```text
error_code
category
module
use_case
endpoint
request_id
trace_id
locale
environment
timestamp
```

लेकिन logs में sensitive information automatically capture नहीं होनी चाहिए।

---

# 51. Observability

Production environment में error codes metrics और traces के लिए useful dimensions बन सकते हैं।

उदाहरण:

```text
USER_NOT_FOUND count
PAYMENT_FAILED count
OTP_EXPIRED count
```

इससे application behavior को समझना आसान होता है।

---

# 52. Error Response और Log Response अलग रखें

### Client

```json
{
  "code": "PAYMENT_FAILED",
  "message": "Payment failed"
}
```

### Internal Log

```text
code=PAYMENT_FAILED
provider=...
transaction_id=...
cause=...
trace_id=...
```

दोनों का उद्देश्य अलग है।

---

# 53. Error Handling Layering

Recommended conceptual flow:

```text
Infrastructure
      ↓
Low-level Error
      ↓
Application Mapping
      ↓
Application Error
      ↓
Transport Mapping
      ↓
Localization
      ↓
API Response
```

---

# 54. Architecture Diagram

```text
┌────────────────────────────────────────┐
│                CLIENT                  │
│                                        │
│ Android / iOS / Web / Admin / Others │
└───────────────────┬────────────────────┘
                    │
                    ▼
┌────────────────────────────────────────┐
│              TRANSPORT                 │
│                                        │
│ HTTP / GraphQL / gRPC / etc.          │
└───────────────────┬────────────────────┘
                    │
                    ▼
┌────────────────────────────────────────┐
│             APPLICATION                │
│                                        │
│ Use Cases / Services                   │
└───────────────────┬────────────────────┘
                    │
                    ▼
┌────────────────────────────────────────┐
│                DOMAIN                  │
│                                        │
│ Business Rules / Domain Errors         │
└───────────────────┬────────────────────┘
                    │
                    ▼
┌────────────────────────────────────────┐
│             INFRASTRUCTURE             │
│                                        │
│ Database / Network / External Services │
└────────────────────────────────────────┘
```

Error presentation flow:

```text
Infrastructure Error
        ↓
Application Mapping
        ↓
Application Error Code
        ↓
Transport Mapping
        ↓
Localization
        ↓
Public Response
```

---

# 55. Recommended Folder Structure

Technology-independent conceptual structure:

```text
project/
│
├── docs/
│   ├── error-handling.md
│   ├── localization.md
│   └── api-contract.md
│
├── error/
│   ├── codes
│   ├── definitions
│   ├── mapping
│   └── serialization
│
├── localization/
│   ├── translator
│   └── locales/
│       ├── en
│       ├── hi
│       ├── bn
│       └── ...
│
├── domain/
│
├── application/
│
├── infrastructure/
│
└── transport/
    ├── http
    ├── grpc
    └── ...
```

Actual folder structure technology के अनुसार बदल सकती है।

---

# 56. Recommended Separation

Error system को कम से कम इन concerns में logically separate रखना चाहिए:

```text
Error Identity
       ↓
Error Definition
       ↓
Error Mapping
       ↓
Localization
       ↓
Serialization
       ↓
Logging
```

इन सभी को एक giant error utility में डालना avoid करें।

---

# 57. Anti-Pattern: Random Error Strings

### गलत

```text
"User not found"
"user doesn't exist"
"No user"
"Could not find user"
```

### सही

```text
USER_NOT_FOUND
```

Message translation layer संभाले।

---

# 58. Anti-Pattern: Language in Business Logic

### गलत

```text
if user == null:
    return "उपयोगकर्ता नहीं मिला"
```

### सही

```text
return USER_NOT_FOUND
```

---

# 59. Anti-Pattern: Message-Based Business Logic

### गलत

```text
if error.message contains "OTP":
```

### सही

```text
if error.code == "INVALID_OTP":
```

---

# 60. Anti-Pattern: Raw Database Error

### गलत

```json
{
  "message": "SQLSTATE[42S02]: table doesn't exist..."
}
```

### सही

```json
{
  "code": "INTERNAL_SERVER_ERROR",
  "message": "Something went wrong"
}
```

Internal database error logs में रहे।

---

# 61. Anti-Pattern: HTTP Status as Business Error

### गलत

```text
if response.status == 404:
    show user-not-found
```

क्योंकि 404 के कई कारण हो सकते हैं।

### सही

```text
if response.error.code == USER_NOT_FOUND:
```

---

# 62. Anti-Pattern: Translation Inside Domain

Domain code में:

```text
"User not found"
"उपयोगकर्ता नहीं मिला"
```

जैसी strings नहीं होनी चाहिए।

---

# 63. Anti-Pattern: हर Error का नया Code

यह गलत है:

```text
USER_NOT_FOUND
USER_DOES_NOT_EXIST
USER_MISSING
USER_NOT_AVAILABLE
NO_USER
```

यदि सभी का semantic meaning एक ही है तो एक canonical code पर्याप्त है।

---

# 64. AI-Assisted Development के लिए Architecture Rules

यह section अत्यंत महत्वपूर्ण है।

Project में AI से code generate करवाते समय AI को निम्न rules follow करने चाहिए।

---

## AI Rule 1 — Existing Architecture पहले पढ़ो

AI को नया error handling system बनाने से पहले:

```text
Existing error package
Existing error codes
Existing API response
Existing localization
Existing transport layer
Existing tests
```

inspect करना चाहिए।

---

## AI Rule 2 — Duplicate Error Code मत बनाओ

नया code बनाने से पहले existing registry में semantic match खोजें।

यदि existing code पर्याप्त है:

```text
Reuse existing code.
```

---

## AI Rule 3 — Error Message Hard-code मत करो

Business/application code में:

```text
"User not found"
```

hard-code न करें।

इसके बजाय:

```text
USER_NOT_FOUND
```

और:

```text
errors.user_not_found
```

का उपयोग करें।

---

## AI Rule 4 — HTTP Dependency नीचे की layers में मत डालो

यदि architecture transport-independent है तो domain/application layer को:

```text
HTTP 400
HTTP 404
HTTP 500
```

से unnecessarily couple न करें।

---

## AI Rule 5 — Internal Cause Preserve करो

Original error debugging के लिए preserve करें।

लेकिन public response sanitize करें।

---

## AI Rule 6 — Existing API Contract न तोड़ो

यदि project में पहले से:

```json
{
  "success": false,
  "error": {}
}
```

का contract है तो बिना explicit requirement के उसे:

```json
{
  "status": "error"
}
```

में न बदलें।

---

## AI Rule 7 — Localization Update करो

नया user-facing error जोड़ते समय:

```text
Error Code
Message Key
Default Translation
Supported Locales
Tests
```

update करें।

---

## AI Rule 8 — Existing Translation Keys Reuse करो

यदि:

```text
errors.user_not_found
```

पहले से मौजूद है तो:

```text
errors.user_missing
```

जैसी duplicate key न बनाएं।

---

## AI Rule 9 — Sensitive Information Protect करो

AI-generated error response में निम्न चीजें expose नहीं होनी चाहिए:

```text
Secrets
Tokens
Passwords
OTP
SQL
Stack Traces
Internal URLs
Credentials
Private Data
```

---

## AI Rule 10 — Tests लिखो

नए error handling behavior के साथ tests होने चाहिए।

---

# 65. AI के लिए Standard Implementation Prompt

किसी भी project में AI को यह instruction दिया जा सकता है:

```text
Implement or extend the project's application error handling and localization architecture.

Requirements:

1. Inspect the existing error handling architecture before making changes.
2. Reuse existing error codes whenever the semantic meaning matches.
3. Do not create duplicate error codes.
4. Every public application error must have a stable machine-readable code.
5. Keep error codes language-independent.
6. Separate error identity from user-facing messages.
7. Use localization/message keys for user-facing text.
8. Do not hard-code localized messages inside domain or business logic.
9. Keep application/domain errors independent from transport-specific protocols.
10. Map application errors to HTTP/gRPC/other transport status codes only at the appropriate transport layer.
11. Preserve the original internal cause for debugging and logging.
12. Never expose secrets, credentials, tokens, OTPs, SQL queries, stack traces, internal URLs, or sensitive information to clients.
13. Use a consistent public API error response structure.
14. Support structured validation details.
15. Support locale selection and deterministic fallback behavior.
16. Support message parameters where required.
17. Support locale-aware pluralization where required.
18. Maintain backward compatibility for existing public error codes.
19. Do not rename or remove existing public error codes without an explicit migration/deprecation strategy.
20. Add or update translations for all supported locales.
21. Add tests for error creation, mapping, serialization, localization, fallback, validation details, security filtering, and backward compatibility.
22. Follow the existing project's architecture, naming conventions, dependency rules, coding style, and folder structure.
23. Do not introduce a parallel error-handling system if an existing one can be extended.
24. Keep implementation modular so new languages and transports can be added without changing business logic.
25. Explain any architectural changes before making large structural modifications.
```

---

# 66. AI को नया Error Add करने का Standard Workflow

AI को नया error जोड़ते समय:

```text
STEP 1
Existing error registry खोजो
        ↓
STEP 2
Semantic match खोजो
        ↓
STEP 3
Existing code reusable है?
        │
        ├── YES
        │    ↓
        │  Reuse
        │
        └── NO
             ↓
STEP 4
New stable error code
             ↓
STEP 5
Error definition
             ↓
STEP 6
Message key
             ↓
STEP 7
Translations
             ↓
STEP 8
Transport mapping
             ↓
STEP 9
Serialization
             ↓
STEP 10
Tests
             ↓
STEP 11
Security verification
             ↓
STEP 12
Backward compatibility verification
```

---

# 67. Error Registry Example

Documentation में registry इस प्रकार maintain की जा सकती है:

| Code | Category | Message Key | Public | Retryable |
|---|---|---|---|---|
| VALIDATION_ERROR | VALIDATION | errors.validation | Yes | No |
| UNAUTHORIZED | AUTHENTICATION | errors.unauthorized | Yes | No |
| FORBIDDEN | AUTHORIZATION | errors.forbidden | Yes | No |
| USER_NOT_FOUND | NOT_FOUND | errors.user_not_found | Yes | No |
| ORDER_NOT_FOUND | NOT_FOUND | errors.order_not_found | Yes | No |
| INVALID_OTP | VALIDATION | errors.invalid_otp | Yes | No |
| OTP_EXPIRED | VALIDATION | errors.otp_expired | Yes | No |
| PAYMENT_FAILED | BUSINESS_RULE | errors.payment_failed | Yes | Usually No |

यह table documentation के लिए है।

Actual runtime source of truth project architecture के अनुसार:

```text
Code
Configuration
Generated Registry
Database
```

में से कोई हो सकता है।

---

# 68. Public Error Contract

यदि कोई error external client को भेजा जाता है तो उसे public API contract की तरह treat करें।

उदाहरण:

```json
{
  "success": false,
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "User not found",
    "details": null
  }
}
```

Client को यह assume नहीं करना चाहिए कि message हमेशा exactly वही रहेगा।

Client को stable field:

```text
error.code
```

पर depend करना चाहिए।

---

# 69. Client Behavior

Client logic:

```text
error.code
      ↓
Business/UI behavior
```

उदाहरण:

```text
USER_NOT_FOUND
    ↓
Show user-not-found UI
```

लेकिन:

```text
error.message
```

का उपयोग केवल display के लिए किया जाना चाहिए।

---

# 70. Translation और Error Code का संबंध

Concept:

```text
USER_NOT_FOUND
       │
       ▼
errors.user_not_found
       │
       ├── en → User not found
       ├── hi → उपयोगकर्ता नहीं मिला
       ├── bn → ...
       ├── mr → ...
       └── ta → ...
```

इससे एक error को अनेक भाषाओं में represent किया जा सकता है।

---

# 71. Language Add करने का लाभ

यदि architecture सही है तो नई language जोड़ने पर:

```text
Domain Code
        ↓
NO CHANGE

Application Logic
        ↓
NO CHANGE

Database
        ↓
Usually NO CHANGE

Translation Catalog
        ↓
NEW LANGUAGE
```

यानी:

```text
English
Hindi
```

से:

```text
English
Hindi
Bengali
Marathi
Tamil
Telugu
Gujarati
Kannada
Malayalam
Punjabi
Odia
```

करने पर business logic बदलने की जरूरत नहीं होनी चाहिए।

---

# 72. Transport Add करने का लाभ

इसी तरह:

```text
HTTP
```

से:

```text
HTTP
gRPC
GraphQL
WebSocket
CLI
Background Jobs
```

जोड़ने पर domain error codes वही रह सकते हैं।

केवल transport adapters बदलेंगे।

---

# 73. Multi-Client Architecture

एक backend:

```text
                 ┌── Android
                 │
Backend ─────────┼── iOS
                 │
                 ├── Web
                 │
                 └── Admin
```

सभी clients को stable error codes दे सकता है।

उदाहरण:

```text
USER_NOT_FOUND
ORDER_NOT_FOUND
INVALID_OTP
```

हर client अपनी presentation strategy इस्तेमाल कर सकता है।

---

# 74. Offline Applications

Offline/mobile application में भी यही architecture लागू हो सकता है।

उदाहरण:

```text
NETWORK_UNAVAILABLE
```

Client इसे:

```text
English:
No internet connection.

Hindi:
इंटरनेट कनेक्शन उपलब्ध नहीं है।
```

में दिखा सकता है।

---

# 75. Domain Errors और Technical Errors

दो broad प्रकार के errors को distinguish करना useful है।

## Domain/Application Error

```text
USER_NOT_FOUND
INVALID_OTP
ORDER_ALREADY_CANCELLED
INSUFFICIENT_BALANCE
```

## Technical/Infrastructure Error

```text
DATABASE_UNAVAILABLE
CACHE_UNAVAILABLE
EXTERNAL_SERVICE_TIMEOUT
NETWORK_FAILURE
```

Technical errors को अक्सर public-safe application error में map करना चाहिए।

---

# 76. Business Rule Errors

Business rules के लिए अलग category उपयोगी हो सकती है।

उदाहरण:

```text
ORDER_ALREADY_CANCELLED
RIDE_ALREADY_STARTED
INSUFFICIENT_BALANCE
SERVICE_NOT_AVAILABLE
```

इनका HTTP status project semantics के अनुसार निर्धारित किया जा सकता है।

Error code और transport mapping को अलग रखें।

---

# 77. Conflict Errors

उदाहरण:

```text
RESOURCE_ALREADY_EXISTS
EMAIL_ALREADY_REGISTERED
PHONE_ALREADY_REGISTERED
ORDER_ALREADY_EXISTS
```

इनका generic category:

```text
CONFLICT
```

हो सकती है।

---

# 78. Authentication और Authorization

इन concepts को अलग रखें।

### Authentication

```text
UNAUTHORIZED
TOKEN_EXPIRED
INVALID_CREDENTIALS
```

### Authorization

```text
FORBIDDEN
INSUFFICIENT_PERMISSION
```

एक user authenticated हो सकता है लेकिन authorized नहीं।

---

# 79. Rate Limiting

Rate limit errors के लिए standard code हो सकता है:

```text
RATE_LIMITED
```

Metadata:

```text
retryable = true
```

और transport layer appropriate status mapping कर सकती है।

---

# 80. Dependency Errors

External service failures के लिए:

```text
PAYMENT_PROVIDER_UNAVAILABLE
SMS_PROVIDER_UNAVAILABLE
EMAIL_PROVIDER_UNAVAILABLE
```

जैसे codes हो सकते हैं।

Internal provider-specific details client को expose नहीं करनी चाहिए।

---

# 81. Timeout Errors

Timeout को generic या domain-specific रूप से model किया जा सकता है।

उदाहरण:

```text
DEPENDENCY_TIMEOUT
PAYMENT_TIMEOUT
```

Architecture को यह distinguish करने की क्षमता होनी चाहिए कि error retryable है या नहीं।

---

# 82. Error Details के लिए Design Rules

`details`:

- structured होना चाहिए।
- predictable schema follow करे।
- sensitive data नहीं रखे।
- client के लिए आवश्यक information तक सीमित रहे।
- arbitrary internal object dump नहीं होना चाहिए।

---

# 83. Error Serialization

Application error को API response में serialize करते समय:

```text
Application Error
        ↓
Public Error Serializer
        ↓
Safe Response
```

Serializer को यह सुनिश्चित करना चाहिए कि:

```text
cause
stack
internal metadata
private fields
```

accidentally response में न जाएं।

---

# 84. Error Handler

हर transport में centralized error handling होना चाहिए।

Concept:

```text
Controller / Handler
        ↓
Use Case
        ↓
Error
        ↓
Central Error Handler
        ↓
Mapping
        ↓
Localization
        ↓
Response
```

इससे हर controller में अलग error serialization लिखने की जरूरत नहीं पड़ती।

---

# 85. Error Handling का Golden Rule

> **Error जितना नीचे generate होता है, उतना technical हो सकता है; जैसे-जैसे वह ऊपर जाता है, उसे appropriate abstraction में map किया जाना चाहिए।**

उदाहरण:

```text
Database Exception
        ↓
Repository Error
        ↓
Application Error
        ↓
Public API Error
```

---

# 86. Error Handling का दूसरा Golden Rule

> **Internal cause को preserve करो, लेकिन public representation को control करो।**

इससे:

```text
Debugging
+
Security
+
User Experience
```

तीनों maintain रहते हैं।

---

# 87. Error Handling का तीसरा Golden Rule

> **Frontend को message नहीं, error code पर भरोसा करना चाहिए।**

Message बदल सकता है:

```text
User not found
```

से:

```text
We couldn't find this user.
```

लेकिन code:

```text
USER_NOT_FOUND
```

stable रह सकता है।

---

# 88. Error Handling का चौथा Golden Rule

> **Localization presentation concern है, business rule नहीं।**

Business logic:

```text
USER_NOT_FOUND
```

Presentation:

```text
उपयोगकर्ता नहीं मिला
```

---

# 89. Error Handling का पाँचवाँ Golden Rule

> **Transport status और application error identity को अलग रखें।**

```text
HTTP 404
```

transport information है।

```text
USER_NOT_FOUND
```

application information है।

---

# 90. Testing Requirements

Error architecture के लिए कम से कम निम्न tests होने चाहिए।

---

## 90.1 Error Creation

```text
Given a valid error code
Then the correct definition should be returned.
```

---

## 90.2 Unknown Error

```text
Given an unknown/internal error
Then the public response should use a safe generic error.
```

---

## 90.3 Localization

```text
Given locale = hi
Then Hindi translation should be returned.
```

---

## 90.4 Fallback

```text
Given unsupported locale
Then configured fallback locale should be used.
```

---

## 90.5 Missing Translation

```text
Given missing translation
Then fallback translation should be used.
```

---

## 90.6 Serialization

```text
Given an application error
Then it should serialize to the standard API error contract.
```

---

## 90.7 Security

```text
Given an internal database error
Then SQL/database details must not appear in the public response.
```

---

## 90.8 Validation

```text
Given multiple invalid fields
Then structured validation details should be returned.
```

---

## 90.9 Backward Compatibility

```text
Given an existing public error code
Then existing clients should continue receiving the compatible contract.
```

---

# 91. Translation Testing

Translation system में verify करें:

```text
All required keys exist
No unexpected keys
No duplicate keys
Fallback works
Parameters resolve correctly
Pluralization works where applicable
```

---

# 92. AI Generated Code के लिए Validation Checklist

AI द्वारा generated code accept करने से पहले verify करें:

```text
[ ] Existing error architecture inspected
[ ] Existing error code reused where possible
[ ] No duplicate error code
[ ] Error code is language-independent
[ ] Message is not hard-coded in business logic
[ ] Message key exists
[ ] Required translations exist
[ ] Fallback exists
[ ] Transport mapping is correct
[ ] Internal cause is preserved
[ ] Internal details are not exposed
[ ] API response contract is unchanged
[ ] Validation details follow existing schema
[ ] Tests added
[ ] Backward compatibility checked
```

---

# 93. AI को Existing Project में काम करने का नियम

AI को हमेशा यह sequence follow करना चाहिए:

```text
Understand
    ↓
Inspect
    ↓
Reuse
    ↓
Extend
    ↓
Test
    ↓
Refactor only when necessary
```

न कि:

```text
Read requirement
    ↓
Create completely new system
```

---

# 94. AI को नया Error बनाने से पहले क्या करना चाहिए?

AI को कम से कम यह खोज करना चाहिए:

```text
Existing error codes
Existing message keys
Existing translations
Existing error handler
Existing response serializer
Existing transport mapping
Existing tests
```

फिर निर्णय लेना चाहिए कि:

```text
Reuse
```

या:

```text
Create New
```

---

# 95. Error Code Governance

Large projects में error codes के लिए governance useful है।

हर नया public error code:

```text
Unique
Meaningful
Stable
Documented
Tested
Localized
```

होना चाहिए।

---

# 96. Error Registry Documentation

हर public error code की documentation में ideally:

```text
Code
Category
Meaning
Public/Private
Message Key
Retryable
Transport Mapping
Example
```

हो।

---

# 97. Example Registry Entry

```text
Code:
USER_NOT_FOUND

Category:
NOT_FOUND

Message Key:
errors.user_not_found

Public:
true

Retryable:
false

Meaning:
Requested user does not exist or is not available to the requester.

HTTP Mapping:
404

Client Action:
Display user-not-found state.
```

---

# 98. Public Error Code के लिए Documentation Rule

Error code का meaning स्पष्ट होना चाहिए।

उदाहरण:

```text
PAYMENT_FAILED
```

का अर्थ केवल:

```text
Payment operation failed.
```

हो।

इसमें provider-specific implementation details नहीं जोड़नी चाहिए।

---

# 99. Error Code और Privacy

Error code भी sensitive information leak कर सकता है।

उदाहरण:

```text
USER_EXISTS
```

का उपयोग user enumeration की समस्या पैदा कर सकता है।

इसलिए public error codes design करते समय security/privacy implications evaluate करें।

---

# 100. Authentication Errors और Security

कुछ systems में:

```text
INVALID_EMAIL
USER_NOT_FOUND
WRONG_PASSWORD
```

जैसी अलग-अलग responses user existence leak कर सकती हैं।

जहाँ आवश्यक हो वहाँ generic response:

```text
INVALID_CREDENTIALS
```

जैसी strategy बेहतर security दे सकती है।

यह product/security requirements पर निर्भर करता है।

---

# 101. Error Architecture को Overengineer न करें

Universal architecture का अर्थ यह नहीं है कि हर project में:

```text
Database Error Registry
Dynamic Error Service
Remote Translation Service
Complex Error Broker
```

बनाना जरूरी है।

Small project:

```text
Static Error Definitions
+
Simple Translation Files
+
Central Error Handler
```

से शुरू कर सकता है।

Large project में आवश्यकता के अनुसार complexity बढ़ाई जा सकती है।

---

# 102. Progressive Architecture

Recommended growth:

```text
Level 1
Stable Error Codes
        ↓
Level 2
Central Definitions
        ↓
Level 3
Localization
        ↓
Level 4
Structured Details
        ↓
Level 5
Transport Mapping
        ↓
Level 6
Logging/Tracing
        ↓
Level 7
Error Registry Governance
        ↓
Level 8
AI Automation
```

हर project को सीधे Level 8 complexity में जाने की जरूरत नहीं है।

---

# 103. Minimal Implementation

एक छोटा project भी निम्न चार चीजों से शुरुआत कर सकता है:

```text
1. Error Codes
2. Error Definitions
3. Translation Catalog
4. Central Error Response
```

उदाहरण:

```text
USER_NOT_FOUND
        ↓
errors.user_not_found
        ↓
Hindi/English translation
        ↓
Standard JSON response
```

---

# 104. Production Implementation

बड़े project में:

```text
Error Registry
+
Localization
+
Transport Mapping
+
Structured Details
+
Logging
+
Tracing
+
Metrics
+
Security Filtering
+
Versioning
+
AI Rules
```

जोड़ा जा सकता है।

---

# 105. Universal Error Flow

पूरा flow:

```text
┌────────────────────────────┐
│       Failure Occurs       │
└─────────────┬──────────────┘
              ↓
┌────────────────────────────┐
│      Technical/Domain      │
│          Error             │
└─────────────┬──────────────┘
              ↓
┌────────────────────────────┐
│      Error Mapping         │
└─────────────┬──────────────┘
              ↓
┌────────────────────────────┐
│       Error Code           │
│      USER_NOT_FOUND        │
└─────────────┬──────────────┘
              ↓
┌────────────────────────────┐
│     Error Definition       │
│ category / metadata        │
└─────────────┬──────────────┘
              ↓
┌────────────────────────────┐
│       Localization         │
│ en / hi / bn / ...         │
└─────────────┬──────────────┘
              ↓
┌────────────────────────────┐
│     Transport Mapping      │
│       HTTP / gRPC          │
└─────────────┬──────────────┘
              ↓
┌────────────────────────────┐
│      Public Response       │
└────────────────────────────┘
```

---

# 106. Final API Example

Request:

```http
GET /users/123
Accept-Language: hi-IN
```

Internal application error:

```text
USER_NOT_FOUND
```

Transport:

```text
HTTP 404
```

Localized response:

```json
{
  "success": false,
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "उपयोगकर्ता नहीं मिला",
    "details": null
  },
  "request_id": "01JXXXXXXXX"
}
```

---

# 107. Internal Logging Example

Client response:

```json
{
  "success": false,
  "error": {
    "code": "INTERNAL_SERVER_ERROR",
    "message": "कुछ गलत हो गया"
  },
  "request_id": "01JXXXXXXXX"
}
```

Internal log:

```text
request_id=01JXXXXXXXX
error_code=INTERNAL_SERVER_ERROR
category=INTERNAL
cause=database connection timeout
service=user-service
```

Client को:

```text
database connection timeout
```

नहीं दिखाई देता।

---

# 108. Recommended Universal Rules

Project में निम्न rules baseline माने जाएं:

```text
1. Error Code = stable machine identity
2. Message = presentation
3. Message Key = localization identity
4. HTTP Status = transport concern
5. Domain Error ≠ HTTP Error
6. Internal Cause ≠ Public Message
7. Details = structured and sanitized
8. Frontend logic = Error Code पर आधारित
9. Translation = presentation layer
10. Fallback Locale = mandatory
11. Error Response = consistent
12. Sensitive Data = never expose
13. Existing Codes = reuse
14. Duplicate Codes = avoid
15. Public Codes = backward-compatible
16. New Languages = business logic change के बिना add हो सकें
17. New Transports = domain change के बिना add हो सकें
18. AI = existing architecture inspect करके code generate करे
19. New Errors = documented and tested हों
20. Error System = unnecessarily overengineered न हो
```

---

# 109. Architecture Decision Summary

| Concern | Responsibility |
|---|---|
| Error Code | Machine-readable identity |
| Error Definition | Error metadata |
| Category | Broad classification |
| Message Key | Localization identity |
| Translation | User-facing language |
| Details | Structured additional data |
| Cause | Internal debugging |
| Transport Mapping | HTTP/gRPC/etc. |
| Serializer | Public response |
| Logger | Internal diagnostics |
| Trace ID | Request correlation |
| Frontend | Presentation & UI behavior |
| Domain | Business meaning |
| AI | Architecture-compliant implementation |

---

# 110. सबसे महत्वपूर्ण Mental Model

पूरे architecture को इस तरह याद रखें:

```text
ERROR
│
├── IDENTITY
│   └── USER_NOT_FOUND
│
├── SEMANTICS
│   ├── category
│   ├── severity
│   └── retryability
│
├── PRESENTATION
│   └── message_key
│
├── LOCALIZATION
│   ├── en
│   ├── hi
│   ├── bn
│   └── ...
│
├── TRANSPORT
│   ├── HTTP
│   ├── gRPC
│   └── ...
│
├── DIAGNOSTICS
│   ├── cause
│   ├── metadata
│   └── trace_id
│
└── CLIENT CONTRACT
    ├── code
    ├── message
    ├── details
    └── request_id
```

---

# 111. Final Principle

Modern application में error को केवल:

```text
"एक string"
```

नहीं समझना चाहिए।

Error एक **structured application contract** है।

```text
Error Identity
      ↓
Stable Error Code
      ↓
Error Metadata
      ↓
Localization
      ↓
Transport Mapping
      ↓
Safe Public Response
      ↓
Logging / Observability
```

इस architecture का सबसे बड़ा फायदा यह है कि:

```text
Business Logic
       ↓
Language से Independent

Business Logic
       ↓
Transport से Independent

Business Logic
       ↓
UI से Independent
```

और इसी कारण एक ही architecture:

```text
Go
Kotlin
Java
Rust
Python
TypeScript
C#
PHP
Dart
Swift
```

जैसी अलग-अलग technologies में implement किया जा सकता है।

---

# 112. AI Implementation Contract

किसी भी project में AI को इस document को पढ़कर निम्न principle follow करना चाहिए:

> **Do not invent a new error architecture when an existing one can be extended.**

AI को:

```text
Inspect → Understand → Reuse → Extend → Test
```

workflow follow करना चाहिए।

AI को:

```text
Hard-coded Messages
Duplicate Error Codes
Transport Coupling
Sensitive Error Exposure
Breaking API Changes
Duplicate Translation Keys
```

से बचना चाहिए।

---

# 113. अंतिम Checklist

किसी project को production-ready मानने से पहले:

```text
ERROR SYSTEM
[ ] Stable error codes
[ ] Central definitions
[ ] Categories
[ ] Public/private distinction
[ ] Structured details
[ ] Cause preservation
[ ] Transport mapping
[ ] Central serialization

LOCALIZATION
[ ] Message keys
[ ] Default locale
[ ] Supported locales
[ ] Fallback strategy
[ ] Parameter interpolation
[ ] Pluralization where needed
[ ] Translation validation

SECURITY
[ ] No secrets in errors
[ ] No tokens in errors
[ ] No OTP in errors
[ ] No SQL exposure
[ ] No stack traces in public response
[ ] No internal infrastructure details
[ ] Privacy-sensitive error codes reviewed

API
[ ] Standard response contract
[ ] Stable error.code
[ ] Structured validation errors
[ ] Request/correlation ID
[ ] Backward compatibility

AI
[ ] Existing architecture inspected
[ ] Existing errors reused
[ ] Duplicate errors avoided
[ ] Translation updated
[ ] Tests added
[ ] Security checked
[ ] Breaking changes avoided
```

---

# 114. One-Line Architecture Rule

यदि इस पूरे document से केवल एक rule याद रखना हो:

```text
Stable Error Code → Error Metadata → Localization → Transport Mapping → Safe Public Response
```

और business logic में:

```text
NEVER depend on translated message text.
ALWAYS depend on stable error identity/code.
```

---

# 115. अंतिम निष्कर्ष

एक scalable और multilingual application में error architecture का उद्देश्य केवल यह बताना नहीं है कि:

```text
"क्या गलत हुआ?"
```

बल्कि system को अलग-अलग concerns में स्पष्ट रूप से divide करना है:

```text
क्या हुआ?
    ↓
Error Code

इसका business meaning क्या है?
    ↓
Category / Definition

User को क्या दिखाना है?
    ↓
Localization

किस transport में कैसे भेजना है?
    ↓
Transport Mapping

Developer को असली समस्या क्या मिली?
    ↓
Cause / Logs / Trace

Frontend को किस condition पर क्या करना है?
    ↓
Stable Error Code
```

इस separation के कारण application:

```text
Multi-language
+
Multi-client
+
Multi-platform
+
Multi-transport
+
Secure
+
Observable
+
Backward-compatible
+
AI-friendly
```

बन सकती है।

**यह specification किसी भी नए project में एक reusable architectural baseline के रूप में इस्तेमाल की जा सकती है।**
# Amazon Rekognition Permission Issue Fix (Console Initialization)

## Overview

इस document में बताया गया है कि **Amazon Rekognition configure करते समय आने वाले permission issue को कैसे fix किया गया**।

Issue का root cause यह था कि AWS account में **Rekognition service initialize नहीं हुई थी**।
जब Rekognition console manually open किया गया, तब service activate हो गई और API requests सही काम करने लगीं।

Services used:

* Amazon Rekognition
* AWS IAM (Identity and Access Management)

---

# Problem

Application में face verification feature configure करते समय Rekognition API calls fail हो रही थीं।

Possible errors:

```
AccessDeniedException
UnrecognizedClientException
InvalidSignatureException
```

हालांकि IAM user correctly configured था।

---

# IAM User Configuration

AWS IAM में एक user पहले से create किया गया था।

User details:

```
User name: rapidplot
ARN: arn:aws:iam::610472461996:user/rapidplot
Console access: Disabled
```

Note:

Console access disabled होना normal है क्योंकि user केवल **API access** के लिए use किया जा रहा था।

---

# Access Key Status

IAM user के पास active access key मौजूद थी।

```
Access Key: Active
Key Age: 2 days
Last Used: 2 days ago
```

इससे confirm हुआ कि credentials valid हैं।

---

# Permission Policy

User के पास Rekognition access policy attached थी।

```
Policy Name: AmazonRekognitionFullAccess
Policy Type: AWS Managed Policy
Attached Via: Directly
```

यह policy Rekognition APIs access करने की full permission देती है।

---

# Region Configuration

Application configuration में AWS region set किया गया था:

```
Region: us-east-1
```

AWS console भी इसी region को reference कर रहा था।

---

# Root Cause

Investigation के बाद पाया गया कि **Rekognition service account में initialize नहीं हुई थी**।

AWS की कुछ services पहली बार use करते समय तब तक fully active नहीं होतीं जब तक:

* console open न किया जाए
* या first API request complete न हो

---

# Fix Applied

Rekognition service console manually open किया गया:

```
https://console.aws.amazon.com/rekognition/
```

इस step के बाद AWS account में Rekognition service initialize हो गई।

---

# Result

Console open करने के बाद Rekognition API calls successful हो गईं।

Example operation:

```
DetectFaces
```

Example response:

```
Face detected
Confidence: 97%
```

इससे confirm हुआ कि Rekognition properly configured है।

---

# Lessons Learned

1. AWS services को पहली बार use करते समय console open करके verify करना useful होता है
2. IAM permissions verify करना जरूरी है
3. Region configuration हमेशा confirm करनी चाहिए
4. Access keys secure तरीके से store करनी चाहिए

---

# Final Working Configuration

```
Service: Amazon Rekognition
Region: us-east-1
Access Key: ************
Secret Key: ************
```

---

# Conclusion

Amazon Rekognition permission issue resolve करने के लिए:

1. IAM user verify किया गया
2. Rekognition permission policy check की गई
3. Region configuration confirm किया गया
4. Rekognition console manually open किया गया

इन steps के बाद Rekognition service successfully work करने लगी और face detection functionality correctly operate करने लगी।

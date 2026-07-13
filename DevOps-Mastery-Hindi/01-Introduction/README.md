# 📖 Chapter 01 - DevOps का परिचय

> **"DevOps कोई Tool नहीं, बल्कि एक Culture, Mindset और Practices का समूह है, जो Software Development और Operations Teams को साथ मिलकर तेज़, सुरक्षित और भरोसेमंद Software Deliver करने में मदद करता है।"**

---

# 📑 विषय सूची

* DevOps क्या है?
* DevOps की आवश्यकता क्यों पड़ी?
* Software Development का इतिहास
* Traditional Software Development
* Waterfall Model
* Agile Methodology
* DevOps Culture
* DevOps Lifecycle
* Continuous Integration (CI)
* Continuous Delivery (CD)
* Continuous Deployment
* Infrastructure as Code (IaC)
* Configuration Management
* Monitoring
* Logging
* Feedback Loop
* DevSecOps
* GitOps
* Platform Engineering
* SRE (Site Reliability Engineering)
* DevOps Engineer की भूमिका
* DevOps Roadmap

---

# 📌 DevOps क्या है?

**DevOps = Development + Operations**

पहले Software बनाने वाले Developers और उसे Production में चलाने वाली Operations Team अलग-अलग काम करती थीं।

उदाहरण:

```text
Developer:
"मेरे सिस्टम पर तो सही चल रहा है।"

↓

Operations Team:
"Production में नहीं चल रहा।"
```

यहीं से समस्याएँ शुरू होती थीं।

---

# 😓 पहले क्या समस्या थी?

मान लीजिए आपने एक Ride Booking App बनाई।

Developer ने:

* Backend बनाया
* Mobile App बनाई
* Testing की

अब Application Operations Team को दे दी गई।

Production में जाने के बाद:

* Server Crash
* Database Error
* Memory Leak
* Configuration Missing
* SSL Missing

अब Developer और Operations Team एक-दूसरे को दोष देने लगते थे।

---

# DevOps क्यों आया?

DevOps का उद्देश्य है कि Development और Operations मिलकर काम करें।

अब पूरी टीम की ज़िम्मेदारी होती है:

* Build
* Test
* Deploy
* Monitor
* Improve

---

# Traditional Workflow

```text
Requirement

↓

Development

↓

Testing

↓

Operations

↓

Production
```

समस्या:

* बहुत Slow
* Manual Process
* Human Errors
* Late Bug Detection

---

# DevOps Workflow

```text
Plan

↓

Code

↓

Build

↓

Test

↓

Deploy

↓

Monitor

↓

Feedback

↓

Improve
```

यह एक Continuous Cycle है।

---

# Software Development का इतिहास

## 1. Waterfall Model

Waterfall Model में हर चरण एक के बाद एक पूरा किया जाता था।

```text
Requirement

↓

Design

↓

Development

↓

Testing

↓

Deployment

↓

Maintenance
```

### फायदे

* Simple
* Documentation अच्छी

### नुकसान

* Changes करना कठिन
* Feedback देर से मिलता है
* Bug अंत में पता चलते हैं

---

# 2. Agile Methodology

Agile में Software छोटे-छोटे हिस्सों (Sprint) में बनाया जाता है।

```text
Sprint 1

↓

Sprint 2

↓

Sprint 3

↓

Sprint 4
```

हर Sprint के बाद ग्राहक से Feedback लिया जाता है।

### फायदे

* Faster Delivery
* Early Feedback
* Continuous Improvement

---

# DevOps कहाँ फिट होता है?

Agile Software बनाना सिखाता है।

DevOps Software को जल्दी, सुरक्षित और लगातार Production तक पहुँचाना सिखाता है।

दोनों एक-दूसरे के पूरक हैं।

---

# DevOps Lifecycle

```text
Plan
 │
 ▼
Code
 │
 ▼
Build
 │
 ▼
Test
 │
 ▼
Release
 │
 ▼
Deploy
 │
 ▼
Operate
 │
 ▼
Monitor
 │
 ▼
Feedback
 │
 └───────────────┐
                 ▼
               Plan
```

---

# Continuous Integration (CI)

CI का मतलब है कि Developers अपने Code को बार-बार Main Branch में Merge करें।

हर Commit पर:

* Build
* Test
* Code Quality Check

अपने आप हो जाए।

---

# Continuous Delivery (CD)

Code हमेशा Production के लिए तैयार रहे।

Deployment का अंतिम निर्णय Developer या Release Manager लेता है।

---

# Continuous Deployment

अगर सभी Tests पास हो जाएँ तो Code बिना Manual Approval के सीधे Production में Deploy हो जाए।

---

# Infrastructure as Code (IaC)

पहले:

AWS Console खोलो

↓

Server बनाओ

↓

Network बनाओ

↓

Security Group बनाओ

अब:

Infrastructure भी Code में लिखी जाती है।

उदाहरण:

* Terraform
* OpenTofu
* Pulumi

---

# Configuration Management

Server बनने के बाद:

* Docker Install
* Nginx Install
* Node.js Install
* Java Install
* SSL Configure

इन सबको Automation से करना Configuration Management कहलाता है।

लोकप्रिय Tools:

* Ansible
* Puppet
* Chef

---

# Monitoring

Production में Application को लगातार Monitor करना आवश्यक है।

उदाहरण:

* CPU Usage
* Memory Usage
* Disk Space
* API Response Time
* Error Rate

लोकप्रिय Tools:

* Prometheus
* Grafana

---

# Logging

अगर Application Crash हो जाए तो Logs देखकर कारण पता लगाया जाता है।

लोकप्रिय Tools:

* Loki
* Elasticsearch
* Kibana

---

# DevSecOps

DevOps + Security = DevSecOps

Security को अंत में नहीं, बल्कि Development के हर चरण में शामिल किया जाता है।

उदाहरण:

* Secret Scanning
* Container Scanning
* Dependency Scanning
* Static Code Analysis

---

# GitOps

Infrastructure और Kubernetes Configuration को Git Repository में रखा जाता है।

Git ही "Single Source of Truth" बन जाता है।

लोकप्रिय Tools:

* Argo CD
* Flux CD

---

# Platform Engineering

Platform Team Developers के लिए Internal Platforms बनाती है।

जैसे:

* One Click Deployment
* Self Service Infrastructure
* Internal Developer Portal

---

# Site Reliability Engineering (SRE)

SRE का लक्ष्य है:

* Reliability
* Scalability
* Automation
* High Availability

Google ने SRE की अवधारणा को लोकप्रिय बनाया।

---

# DevOps Engineer क्या करता है?

एक DevOps Engineer की सामान्य ज़िम्मेदारियाँ:

* Linux Administration
* Networking
* Cloud Infrastructure
* Docker
* Kubernetes
* CI/CD Pipelines
* Infrastructure Automation
* Monitoring
* Logging
* Security
* Incident Response

---

# DevOps में उपयोग होने वाले प्रमुख Tools

| Category                 | Examples                           |
| ------------------------ | ---------------------------------- |
| Version Control          | Git, GitHub, GitLab                |
| CI/CD                    | GitHub Actions, Jenkins, GitLab CI |
| Containers               | Docker                             |
| Container Orchestration  | Kubernetes                         |
| Infrastructure as Code   | Terraform, OpenTofu                |
| Configuration Management | Ansible                            |
| Monitoring               | Prometheus, Grafana                |
| Logging                  | Loki, Elasticsearch, Kibana        |
| Cloud                    | AWS, Azure, GCP                    |

---

# DevOps सीखने का रोडमैप

```text
Computer Basics
        │
        ▼
Linux
        │
        ▼
Networking
        │
        ▼
Git
        │
        ▼
Shell Scripting
        │
        ▼
Docker
        │
        ▼
Docker Compose
        │
        ▼
Terraform
        │
        ▼
Ansible
        │
        ▼
AWS
        │
        ▼
Kubernetes
        │
        ▼
Helm
        │
        ▼
GitHub Actions
        │
        ▼
Monitoring
        │
        ▼
Security
        │
        ▼
Production Projects
```

---

# इस अध्याय का सार

इस अध्याय में आपने सीखा:

* DevOps क्या है।
* DevOps की आवश्यकता क्यों पड़ी।
* Waterfall और Agile का अंतर।
* CI/CD की मूल अवधारणा।
* Infrastructure as Code क्या है।
* Configuration Management क्या है।
* Monitoring और Logging का महत्व।
* DevSecOps, GitOps, Platform Engineering और SRE का परिचय।
* DevOps सीखने का रोडमैप।

---

# अभ्यास (Practice)

1. DevOps और Agile में क्या अंतर है?
2. Continuous Delivery और Continuous Deployment में क्या अंतर है?
3. Infrastructure as Code क्या है?
4. Configuration Management किसे कहते हैं?
5. Monitoring और Logging में क्या अंतर है?

---

# अगले अध्याय में

➡️ **Chapter 02 - Linux Fundamentals**

इसमें हम सीखेंगे:

* Linux क्या है?
* Linux Architecture
* File System
* Commands
* Users
* Groups
* Permissions
* Processes
* Services
* SSH
* Cron
* Networking Basics
* Bash Scripting की शुरुआत

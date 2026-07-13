# 🐧 Chapter 02 - Linux Fundamentals

> **"यदि DevOps सीखना है, तो सबसे पहले Linux सीखना होगा।"**

---

# 📖 इस अध्याय में आप क्या सीखेंगे?

- Linux क्या है?
- Linux का इतिहास
- Linux क्यों बनाया गया?
- Linux Kernel क्या है?
- GNU क्या है?
- Shell क्या है?
- Terminal क्या है?
- Linux Distribution क्या है?
- Linux Architecture
- Linux File System Overview
- Linux कहाँ उपयोग होता है?
- DevOps में Linux क्यों आवश्यक है?
- आगे क्या सीखेंगे?

---

# Linux क्या है?

**Linux एक Open Source Operating System Kernel है।**

ध्यान दें।

बहुत लोग Linux को Operating System कहते हैं।

असल में

Linux = Kernel

और

Ubuntu, Debian, Fedora आदि = Operating System (Linux Distribution)

---

# पहले Operating System कैसे थे?

1990 के आसपास अधिकतर लोग उपयोग करते थे

- MS-DOS
- UNIX
- Windows

UNIX बहुत शक्तिशाली था लेकिन महंगा था।

उसका Source Code उपलब्ध नहीं था।

यदि कोई छात्र सीखना चाहता था

तो वह उसके अंदर क्या चल रहा है

नहीं देख सकता था।

---

# Linux क्यों बनाया गया?

1991 में

**Linus Torvalds** नाम के एक छात्र ने सोचा

> "क्यों न एक Free UNIX जैसा Kernel बनाया जाए?"

उन्होंने अपना Kernel Internet पर डाल दिया।

दुनिया भर के Developers उसमें योगदान देने लगे।

आज Linux

दुनिया का सबसे ज्यादा उपयोग होने वाला Server Operating System है।

---

# Linux Open Source क्यों है?

Open Source का मतलब

- Source Code सभी देख सकते हैं।
- Modify कर सकते हैं।
- Improve कर सकते हैं।
- Share कर सकते हैं।

इसी कारण

Linux लगातार बेहतर होता गया।

---

# Kernel क्या होता है?

Kernel Operating System का सबसे महत्वपूर्ण भाग होता है।

यह Hardware और Software के बीच Communication करवाता है।

```text
Application

↓

Shell

↓

Kernel

↓

CPU
Memory
Disk
Network
```

यदि Kernel न हो

तो Application

Hardware से बात नहीं कर पाएगी।

---

# Kernel की जिम्मेदारियाँ

Kernel

- Process Management
- Memory Management
- Device Drivers
- File System
- CPU Scheduling
- Networking
- Security

सब कुछ संभालता है।

---

# GNU क्या है?

Linux केवल Kernel है।

Operating System बनाने के लिए

बहुत सारे Tools चाहिए।

जैसे

- ls
- cp
- mv
- rm
- cat
- grep
- bash

ये अधिकांश GNU Project से आते हैं।

इसलिए कई लोग इसे

GNU/Linux

कहते हैं।

---

# Linux Distribution क्या है?

Linux Kernel + GNU Tools + Package Manager + Desktop

=

Linux Distribution

उदाहरण

- Ubuntu
- Debian
- Fedora
- CentOS
- Rocky Linux
- AlmaLinux
- Arch Linux
- Kali Linux

---

# सबसे लोकप्रिय Distributions

## Ubuntu

सबसे आसान

Beginner Friendly

बहुत Documentation

---

## Debian

बहुत Stable

Servers में लोकप्रिय

Ubuntu भी Debian पर आधारित है।

---

## Rocky Linux

Enterprise Servers

Red Hat Compatible

---

## Fedora

Latest Packages

Developers के लिए अच्छा

---

## Arch Linux

Advanced Users

Rolling Release

---

## Kali Linux

Cyber Security

Penetration Testing

---

# Linux Architecture

```text
+-------------------------+
|     User Applications   |
+-------------------------+
            │
            ▼
+-------------------------+
|         Shell           |
+-------------------------+
            │
            ▼
+-------------------------+
|         Kernel          |
+-------------------------+
            │
            ▼
+-------------------------+
|       Hardware          |
+-------------------------+
```

---

# Shell क्या है?

Shell

User और Kernel के बीच Interface है।

जब आप लिखते हैं

```bash
ls
```

तो

Shell

Kernel को Command भेजता है।

Kernel

Filesystem से Data लाता है।

फिर

Shell

आपको Output दिखाता है।

---

# लोकप्रिय Shell

- Bash
- Zsh
- Fish
- Ksh
- Tcsh

Production Servers में

सबसे अधिक Bash उपयोग होती है।

---

# Terminal क्या है?

Terminal

एक Program है

जिसमें हम Commands लिखते हैं।

Windows

- PowerShell
- CMD
- Windows Terminal

Linux

- GNOME Terminal
- Konsole
- XTerm

Terminal

Shell को चलाता है।

---

# Linux कहाँ उपयोग होता है?

Linux आज लगभग हर जगह है।

- AWS
- Azure
- GCP
- Docker
- Kubernetes
- Android
- Raspberry Pi
- Smart TV
- Super Computers
- Web Servers

---

# क्या Android भी Linux है?

हाँ।

Android

Linux Kernel का उपयोग करता है।

लेकिन

Android

GNU Tools का उपयोग नहीं करता।

इसलिए Android

GNU/Linux नहीं है।

---

# DevOps में Linux क्यों जरूरी है?

लगभग

95%

Cloud Servers

Linux पर चलते हैं।

यदि आपको

- AWS
- Docker
- Kubernetes
- Nginx
- Terraform
- Ansible

सीखना है

तो Linux आना अनिवार्य है।

---

# DevOps Engineer प्रतिदिन Linux में क्या करता है?

- SSH से Login
- Logs पढ़ना
- Services Restart करना
- Files Edit करना
- Docker चलाना
- Kubernetes Debug करना
- Disk Check करना
- Memory Check करना
- Network Troubleshoot करना

---

# Linux सीखने के बाद क्या-क्या आसान हो जाएगा?

- Docker
- Kubernetes
- AWS EC2
- Terraform
- Ansible
- Jenkins
- GitHub Actions
- Nginx
- Apache
- Redis
- PostgreSQL
- MySQL

---

# Linux File System का परिचय

Linux में सब कुछ File माना जाता है।

उदाहरण

```
/

├── home

├── etc

├── var

├── usr

├── tmp

├── boot

├── dev

├── proc

└── opt
```

अगले अध्यायों में

हम हर Directory को विस्तार से समझेंगे।

---

# इस Module में आगे क्या सीखेंगे?

✅ Installation

✅ Linux Architecture

✅ File System

✅ Basic Commands

✅ Advanced Commands

✅ Users

✅ Groups

✅ Permissions

✅ Processes

✅ Services

✅ Systemd

✅ SSH

✅ Cron Jobs

✅ Networking

✅ Logs

✅ Bash Scripting

✅ Disk Management

✅ Memory Management

✅ Performance

✅ Security

---

# याद रखने योग्य बातें

✔ Linux Open Source है।

✔ Linux केवल Kernel है।

✔ Ubuntu एक Linux Distribution है।

✔ Kernel Hardware को Manage करता है।

✔ Shell Commands को Execute करता है।

✔ Terminal वह Program है जिसमें Commands लिखते हैं।

✔ अधिकांश Servers Linux पर चलते हैं।

---

# अभ्यास

1. Linux और Ubuntu में क्या अंतर है?

2. Kernel क्या करता है?

3. Shell और Terminal में क्या अंतर है?

4. Distribution क्या होती है?

5. DevOps Engineer को Linux क्यों सीखना चाहिए?

---

# Interview Questions

### Linux क्या है?

### Kernel क्या है?

### Shell क्या है?

### Bash क्या है?

### Ubuntu और Debian में अंतर?

### Open Source क्या होता है?

### Linux Server क्यों लोकप्रिय हैं?

### Android Linux कैसे है?

---

# अगले अध्याय में

➡ **01-Installation.md**

हम सीखेंगे

- VirtualBox Install करना
- VMware Install करना
- WSL2
- Ubuntu Installation
- Dual Boot
- Cloud VM
- SSH से Connect करना
- पहला Linux Login

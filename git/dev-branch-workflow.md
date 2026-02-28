# 🌿 Dev Branch Setup & Workflow Guide

यह documentation Git project में `dev` branch बनाने, manage करने और proper development workflow follow करने के लिए बनाई गई है।

`main` branch हमेशा stable और production-ready code के लिए होती है, जबकि `dev` branch development और नई features के लिए use की जाती है।

---

## 📌 Branch Strategy Overview

```
main   → Stable / Production code
dev    → Development branch
feature/* → Individual feature development
```

---

## ✅ Step 1: Current Status Check

सबसे पहले check करें कि आप किस branch पर हैं और सभी changes committed हैं।

```bash
git status
git branch
```

अगर uncommitted changes हैं:

```bash
git add .
git commit -m "Save current changes"
```

---

## ✅ Step 2: Dev Branch Create करें

### Recommended Method (main से)

```bash
git checkout main
git pull origin main
git checkout -b dev
```

यह सुनिश्चित करता है कि dev branch latest stable code से बने।

---

## ✅ Step 3: Remote Repository पर Push करें

```bash
git push -u origin dev
```

यह command dev branch को remote repository पर push करती है और upstream automatically set कर देती है।

---

## ✅ Step 4: Upstream Branch Manual Setup (Optional)

अगर upstream automatically set न हो:

```bash
git branch --set-upstream-to=origin/dev
```

---

## 🚀 Feature Development Workflow

नई feature हमेशा dev branch से बनाएं।

### Feature Branch Create

```bash
git checkout dev
git checkout -b feature/login-system
```

### Work Complete होने के बाद

```bash
git checkout dev
git merge feature/login-system
```

---

## 🔄 Dev से Main में Merge

Testing complete होने के बाद:

```bash
git checkout main
git pull origin main
git merge dev
git push origin main
```

---

## ⚠️ Important Rules

* ❌ main branch पर direct development न करें।
* ✅ हमेशा dev branch पर काम करें।
* ✅ Merge से पहले testing करें।
* ✅ Small और meaningful commits करें।
* ✅ Regularly branches update रखें।

---

## 📦 Daily Development Commands

```bash
# Latest code fetch करें
git pull

# Current branch check
git branch

# Changes add करें
git add .

# Commit करें
git commit -m "your message"

# Push करें
git push
```

---

## 🎯 Benefits of Dev Branch Workflow

* Stable production environment
* Safe development process
* Easy bug tracking
* Better collaboration
* Clean project history

---

## 📌 Best Practice Routine

Development शुरू करने से पहले:

```bash
git checkout main
git pull origin main
git checkout dev
git pull origin dev
```

---

## ✅ Summary

* main → Production ready code
* dev → Active development
* feature/* → New features
* Proper branching strategy = Safe & scalable development

---

⭐ Tip: हर project में शुरुआत से branch workflow follow करें ताकि future में deployment और collaboration आसान रहे।

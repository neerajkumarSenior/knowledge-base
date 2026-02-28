# 🔀 Dev Branch से Main Branch में Changes Merge करने की Guide

यह documentation बताती है कि `dev` branch में किए गए development changes को सुरक्षित तरीके से `main` branch में कैसे लाया जाए।

`main` branch हमेशा stable और production-ready code के लिए होती है, इसलिए merge process सही तरीके से करना जरूरी है।

---

## 📌 कब Merge करना चाहिए?

* जब feature development complete हो जाए
* Testing successfully हो जाए
* Code production के लिए ready हो

---

## ✅ Step 1: Main Branch पर Switch करें

सबसे पहले `main` branch पर जाएं:

```bash
git checkout main
```

---

## ✅ Step 2: Main Branch को Latest करें

Remote repository से latest changes लाएं:

```bash
git pull origin main
```

यह step conflicts avoid करने के लिए बहुत important है।

---

## ✅ Step 3: Dev Branch को Merge करें

अब `dev` branch के changes को `main` में merge करें:

```bash
git merge dev
```

इस command के बाद dev branch के सभी commits main branch में जुड़ जाएंगे।

---

## ✅ Step 4: Changes Remote Repository पर Push करें

```bash
git push origin main
```

अब GitHub पर main branch update हो जाएगी।

---

## 🔄 Complete Merge Flow (Quick Commands)

```bash
git checkout main
git pull origin main
git merge dev
git push origin main
```

---

## ⚠️ Important Rules

* ❌ main branch पर direct development न करें।
* ✅ हमेशा dev branch पर development करें।
* ✅ Merge से पहले dev branch को test करें।
* ✅ Conflicts आने पर carefully resolve करें।

---

## 🧠 Visual Workflow

```
Before Merge:

main  ----A------B
                 \
dev    ----C----D----E

After Merge:

main  ----A----B----C----D----E
```

---

## 🛠️ Conflict आने पर क्या करें?

अगर merge conflict आता है:

1. Conflict files open करें
2. सही code manually select करें
3. File save करें
4. फिर commands चलाएं:

```bash
git add .
git commit -m "Resolve merge conflict"
```

---

## 🎯 Benefits

* Stable production code maintain रहता है
* Development process safe रहता है
* Project history clear रहती है
* Deployment आसान हो जाता है

---

## ✅ Best Practice

Merge करने से पहले dev branch update करें:

```bash
git checkout dev
git pull origin dev
```

---

## 📌 Summary

* `dev` → Development work
* `main` → Production ready code
* Dev complete → Main में merge
* Proper workflow = Safe deployment

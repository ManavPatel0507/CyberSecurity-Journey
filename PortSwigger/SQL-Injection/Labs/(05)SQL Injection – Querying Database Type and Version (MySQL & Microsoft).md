
# 🛡️ Lab: SQL Injection – Querying Database Type and Version (MySQL & Microsoft)

## 📌 Platform

* PortSwigger Web Security Academy

---

## 🎯 Objective

Retrieve the database version string using SQL injection.

---

## 🔍 Initial Testing

In this lab, the goal is similar to the previous lab:

* Fetch the database version
* But this time from **MySQL / Microsoft SQL Server**, not Oracle

---

### 🧪 Step 1: Break the Query

Tried:

```sql
'
```

### 🔍 Observation:

* Page returned **Internal Server Error**

### 📌 Conclusion:

* SQL query is breaking → SQL injection is possible

---

## 🚫 Step 2: Initial Failures

After confirming injection, tried:

* `UNION SELECT`
* `ORDER BY`
* Other payloads

### ❌ Result:

* Page kept breaking
* No valid output

---

### 🧠 Reason:

* The query was not properly terminated
* Remaining part of the original query was still executing

---

## 🔐 Step 3: Fixing the Query using Comments

Learned that:

```sql
--
```

* Is used to comment out the rest of the SQL query

---

### ⚠️ Important Detail

In URL-based injection:

```text
--      ❌ may not work
--+     ✅ adds space
--%20   ✅ URL encoded space
```

👉 SQL requires a **space after `--`** to activate comment

---

### 🧪 Tried:

```sql
'--+
'--%20
```

### ✅ Result:

* Page loaded normally
* Query successfully terminated

---

## 📊 Step 4: Finding Number of Columns

Tried:

```sql
' UNION SELECT NULL,NULL--+
```

### ✅ Result:

* No error

### 📌 Conclusion:

* Query has **2 columns**

---

## 🧠 Step 5: Understanding Output Placement

Observed:

* First column → appears in middle (messy)
* Second column → appears clearly at the end

👉 So better to place payload in **second column**

---

## 💥 Step 6: Extract Database Version

Used:

```sql
' UNION SELECT NULL,@@version--+
```

### 🔍 Explanation:

* `@@version` → returns database version (MySQL / Microsoft)
* No `FROM` required (it's a system variable)
* Matches 2-column structure

---

### ✅ Result:

* Database version displayed on page
* Lab solved

---

## 🧠 Key Learnings

* Always properly terminate queries using comments
* `--` requires a space → use `--+` or `--%20` in URLs
* UNION-based SQLi requires:

  * Correct number of columns
  * Matching data types
* Some values (like `@@version`) are:

  * System variables
  * Do NOT require `FROM`

---

## 🛠️ Payload Summary

### Injection Test

```sql
'
```

### Fix Query

```sql
'--+
```

### Find Columns

```sql
' UNION SELECT NULL,NULL--+
```

### Extract Version

```sql
' UNION SELECT NULL,@@version--+
```

---

## 📌 Conclusion

This lab demonstrates how to:

* Properly terminate SQL queries
* Identify column count
* Use UNION-based SQL injection
* Extract database version using system variables

---

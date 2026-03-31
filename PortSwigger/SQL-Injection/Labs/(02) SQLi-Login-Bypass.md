# 🛡️ Lab: SQL Injection – Login Bypass (Authentication Bypass)

## 📌 Platform

* PortSwigger Web Security Academy

---

## 🎯 Objective

Exploit an SQL injection vulnerability in the login functionality to log in as the `administrator` user without knowing the password.

---

## 🔍 Vulnerability Overview

The application has a login form:

* Username
* Password

The backend query is likely:

```sql
SELECT * FROM users WHERE username = 'input' AND password = 'input'
```

The input is not properly sanitized, making it vulnerable to **SQL Injection**.

---

## 🧪 Method 1: Comment-Based Authentication Bypass (Your Approach)

### 🔹 Payload Used

```text
Username: administrator--
Password: anything
```

### ✅ Result:

* Successfully logged in as administrator

---

### 🧠 Why This Works

Injected query becomes:

```sql
SELECT * FROM users WHERE username = 'administrator--' AND password = 'anything'
```

But actually interpreted as:

```sql
SELECT * FROM users WHERE username = 'administrator'-- AND password = 'anything'
```

### 🔍 Explanation:

* `--` → comments out the rest of the query
* Password condition is ignored
* Only username is checked → login successful

---

## 💥 Method 2: OR-Based Authentication Bypass (Alternative / Official Method)

### 🔹 Payload Used

```text
Username: administrator' OR 1=1--
Password: anything
```

---

### ✅ Result:

* Login bypass successful

---

### 🧠 Why This Works

Injected query becomes:

```sql
SELECT * FROM users WHERE username = 'administrator' OR 1=1-- AND password = 'anything'
```

### 🔍 Explanation:

* `OR 1=1` → always TRUE
* `--` → ignores password condition
* Query returns valid user → login granted

---

## ⚖️ Comparison of Both Methods

| Method        | Payload                   | Behavior                    |
| ------------- | ------------------------- | --------------------------- |
| Comment-based | `administrator--`         | Removes password check      |
| OR-based      | `administrator' OR 1=1--` | Makes condition always true |

---

## 🔓 Exploitation Result

* Logged in as administrator without knowing password
* Authentication mechanism bypassed
* Lab successfully solved

---

## 🧠 Key Learnings

* Login forms are common SQL injection targets
* SQL comments (`--`) can remove security checks
* `OR 1=1` is a classic authentication bypass technique
* Always test both:

  * Comment-based payloads
  * Boolean-based payloads

---

## 🛠️ Tools Used

* Browser (manual testing)
* (Optional) Burp Suite for interception

---

## 🚀 Payload Summary

### Method 1 (Simplest)

```text
administrator--
```

### Method 2 (Classic)

```text
administrator' OR 1=1--
```

---

## 📌 Real-World Relevance

* Can lead to:

  * Full account takeover
  * Admin access without credentials
* Very common vulnerability in poorly secured login systems
* Frequently found in bug bounty programs

---

## ✅ Conclusion

This lab demonstrates how improper input validation in authentication systems can allow attackers to bypass login mechanisms using SQL injection.

---

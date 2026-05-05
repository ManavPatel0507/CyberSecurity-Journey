# 🛡️ Lab: Stored Cross-Site Scripting (XSS) – Basic

## 📌 Platform
- PortSwigger Web Security Academy

---

## 🎯 Goal
This lab contains a **stored XSS vulnerability** in the comment functionality.

👉 Objective:
- Submit a comment that executes JavaScript  
- Trigger `alert()` when the blog post is viewed  

---

## 🔍 Vulnerability Type
- **Stored XSS (Persistent XSS)**
- User input is:
  - Stored in the database  
  - Rendered later to other users  
  - Not sanitized or encoded  

---

## ⚙️ Steps to Solve

### 1. Open Target Page
- Go to the website  
- Open any **blog post**  

---

### 2. Submit Malicious Comment

Enter the following payload in the comment field:

```html
<script>alert(1)</script>
```

---

### 3. Fill Required Fields
- Name  
- Email  
- Website (optional or any value)

---

### 4. Post Comment
- Submit the comment  

---

### 5. Trigger Execution
- Reload or view the blog post  

---

## ✅ Result
- The script executes when the page loads  
- `alert(1)` popup appears  
- Lab marked as solved  

---

## 🧠 Explanation
- The application stores user input without sanitization  
- When the blog is viewed, the stored input is rendered as HTML  
- Browser executes the injected JavaScript  

---

## 🔄 Alternative Payloads

```html
<script>prompt(1)</script>
```

```html
<script>print()</script>
```

👉 These can also confirm XSS  

---

## ⚠️ Key Learnings
- Stored XSS is more dangerous than reflected XSS  
- Payload persists in the application database  
- Affects all users who view the page  
- Always test input fields like:
  - Comments  
  - Profiles  
  - Forms  

---

## 🛠️ Payload Summary

```html
<script>alert(1)</script>
```

---

## 📌 Conclusion
This lab demonstrates a basic **stored XSS attack**, where:
1. Malicious input is stored in the database  
2. Input is rendered when the page is viewed  
3. Browser executes injected JavaScript automatically  

---

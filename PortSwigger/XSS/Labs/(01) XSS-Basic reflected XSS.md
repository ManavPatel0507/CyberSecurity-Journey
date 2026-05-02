# 🛡️ Lab: Reflected Cross-Site Scripting (XSS) – Basic

## 📌 Platform
- PortSwigger Web Security Academy

---

## 🎯 Goal
This lab contains a **simple reflected XSS vulnerability** in the search functionality.

👉 Objective:
- Perform a cross-site scripting attack  
- Execute the `alert()` function in the browser  

---

## 🔍 Vulnerability Type
- **Reflected XSS**
- User input from the search bar is:
  - Reflected directly in the response  
  - Not sanitized or encoded  

---

## ⚙️ Steps to Solve

### 1. Locate Input Field
- Use the **search bar** on the website  

### 2. Inject Payload

```html
<script>alert(1)</script>
```

### 3. Observe Behavior
- The script executes in the browser  
- A popup (`alert(1)`) appears  

---

## ✅ Result
- JavaScript successfully executed  
- Lab marked as solved  

---

## 🧠 Explanation
- The application reflects user input directly into the HTML response  
- No input sanitization or output encoding is applied  
- Browser interprets injected input as executable JavaScript  

---

## 🔄 Alternative Payloads

```html
<script>prompt(1)</script>
```

```html
<script>print()</script>
```

👉 These can also be used to confirm XSS  

---

## ⚠️ Key Learnings
- Reflected XSS occurs when:
  - User input is returned immediately in the response  
- `<script>` tag is a common injection method  
- Always test input fields for reflection and execution  

---

## 🛠️ Payload Summary

```html
<script>alert(1)</script>
```

---

## 📌 Conclusion
This lab demonstrates a basic **reflected XSS attack**, where:
1. User input is not sanitized  
2. Input is reflected in the response  
3. Browser executes injected JavaScript  

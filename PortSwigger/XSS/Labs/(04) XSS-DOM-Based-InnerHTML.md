# 🛡️ Lab: DOM-Based XSS in innerHTML (location.search)

## 📌 Platform
- PortSwigger Web Security Academy

---

## 🎯 Goal
This lab contains a **DOM-based XSS vulnerability**.

👉 Objective:
- Perform a DOM XSS attack  
- Execute `alert()` using input from the URL  

---

## 🔍 Vulnerability Type
- **DOM-Based XSS**
- Source: `location.search` (URL input)
- Sink: `innerHTML`

👉 Flow:
```
User Input (URL) → JavaScript → innerHTML → Executed in Browser
```

---

## 🧠 Key Concept

- The application reads data from the URL (`location.search`)
- It inserts it into the page using:
```javascript
element.innerHTML
```

❌ No sanitization → allows injection

---

## ⚙️ Steps to Solve

### 1. Identify Input Source
- Search functionality uses URL parameter  
- Example:
```
?search=test
```

---

### 2. Inject Payload

Enter this payload in the search box:

```html
<img src=1 onerror=alert(1)>
```

---

### 3. Execute

- Click **Search**  
- Page loads with injected HTML  

---

## ✅ Result
- Image fails to load (`src=1` is invalid)  
- `onerror` event triggers  
- `alert(1)` executes  
- Lab marked as solved  

---

## 🧠 Explanation

- Input is taken from URL (`location.search`)  
- Inserted into DOM using `innerHTML`  
- Browser parses it as HTML  
- `<img>` tag is created  
- Invalid `src` triggers `onerror`  
- JavaScript executes  

---

## 🔄 Alternative Payloads

```html
<img src=x onerror=alert(1)>
```

```html
<svg onload=alert(1)>
```

---

## ⚠️ Key Learnings

- DOM XSS happens fully on client-side  
- `innerHTML` is a dangerous sink  
- No need to break out of attributes (unlike previous lab)  
- Event handlers like:
  - `onerror`
  - `onload`
  are commonly used  

---

## 🛠️ Payload Summary

```html
<img src=1 onerror=alert(1)>
```

---

## 📌 Conclusion

This lab demonstrates a **DOM-based XSS attack**, where:

1. Input comes from URL (`location.search`)  
2. Inserted into DOM via `innerHTML`  
3. No sanitization is applied  
4. Malicious HTML executes automatically  

---

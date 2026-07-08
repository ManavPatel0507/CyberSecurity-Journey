# 🛡️ Lab: DOM-Based XSS in jQuery `href` Attribute Sink using `location.search`

## 📌 Platform
- PortSwigger Web Security Academy

---

## 🎯 Goal

This lab contains a **DOM-Based XSS vulnerability** in the **Submit Feedback** page.

👉 Objective:
- Make the **Back** link execute:
```javascript
alert(document.cookie)
```

---

## 🔍 Vulnerability Type

- **DOM-Based XSS**
- Source: `location.search`
- Sink: jQuery `.attr('href', ...)`

👉 Data Flow:

```
User Input (URL) → location.search → jQuery → href attribute → JavaScript executes
```

---

## 🧠 Key Concept

The application reads the value of the `returnPath` URL parameter and assigns it directly to the `href` attribute of the **Back** link using jQuery.

Conceptually, the code behaves like:

```javascript
$('a').attr('href', returnPath);
```

Because no validation is performed, an attacker can supply a dangerous URI such as:

```text
javascript:alert(document.cookie)
```

Instead of a normal URL.

---

## ⚙️ Steps to Solve

### 1. Open the **Submit Feedback** page.

---

### 2. Confirm where the input is used.

Change the URL parameter:

```
returnPath=/test123
```

Inspect the page and observe that it becomes:

```html
<a href="/test123">Back</a>
```

This confirms that **returnPath** is directly inserted into the `href` attribute.

---

### 3. Inject the Payload

Replace the parameter with:

```text
javascript:alert(document.cookie)
```

Final URL:

```
?returnPath=javascript:alert(document.cookie)
```

---

### 4. Trigger Execution

Click the **Back** link.

---

## ✅ Result

- Clicking **Back** executes:

```javascript
alert(document.cookie)
```

- Browser displays the current cookies.
- Lab marked as solved.

---

## 🧠 Explanation

Normally, an anchor (`<a>`) points to another webpage:

```html
<a href="/home">Back</a>
```

However, browsers also support the special **JavaScript URI scheme**:

```html
<a href="javascript:alert(1)">Back</a>
```

Instead of navigating to another page, clicking the link executes the JavaScript code.

Since the application copies user-controlled input directly into the `href` attribute, an attacker can replace the URL with malicious JavaScript.

---

## ⚠️ Why User Interaction is Required

Unlike previous DOM XSS labs:

- `document.write()` executes immediately.
- `innerHTML` executes when the HTML is parsed.
- `href="javascript:..."` **does not execute automatically.**

The JavaScript runs **only when the user clicks the link**.

---

## 🛠️ Payload Summary

```text
javascript:alert(document.cookie)
```

---

## ⚠️ Key Learnings

- DOM XSS can occur through HTML attributes, not just HTML content.
- `location.search` is a common **source** of user-controlled input.
- `href` is a dangerous **sink** if user input is written directly into it.
- Browsers execute JavaScript when a link uses the `javascript:` URI scheme.
- Some DOM XSS vulnerabilities require **user interaction** (such as clicking a link) before the payload executes.

---

## 📌 Conclusion

This lab demonstrates a **DOM-Based XSS vulnerability** where:

1. User input is taken from `location.search`.
2. jQuery assigns the value directly to an anchor's `href` attribute.
3. A malicious `javascript:` URI is injected.
4. Clicking the **Back** link executes arbitrary JavaScript in the victim's browser.

---

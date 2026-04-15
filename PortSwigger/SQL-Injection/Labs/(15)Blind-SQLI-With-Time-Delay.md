# Lab: Blind SQL Injection with Time Delays

**Difficulty:** Practitioner  
**Platform:** PortSwigger Web Security Academy  
**Topic:** Blind SQL Injection — Time-Based  
**Database:** PostgreSQL  

---

## Objective

The application uses a tracking cookie for analytics and performs a SQL query with its value. The results are never returned, and the application responds identically whether the query succeeds or fails — meaning **neither output nor errors are visible**. The goal is to confirm SQL injection exists using time delays.

---

## Key Concepts

### What is Time-Based Blind SQLi?

This is the third flavour of blind SQLi. When you have:
- No query output visible
- No error messages visible
- No difference in HTTP response between true/false conditions

...you have **nothing left to observe except time**. If you can make the database deliberately pause, and the HTTP response is delayed by the same amount, you've confirmed injection.

| SQLi Type | Signal Used |
|---|---|
| Visible error-based | Error message contains data |
| Conditional errors (blind) | HTTP 500 vs 200 |
| Time-based blind (this lab) | Response time delay |

### `pg_sleep()` — PostgreSQL's Sleep Function

```sql
pg_sleep(10)  -- tells PostgreSQL to pause for 10 seconds
```

If your injected `pg_sleep(10)` causes the HTTP response to take 10 seconds longer than usual, the database executed your code. That's your confirmation signal.

### Database-Specific Sleep Functions

Different databases have different sleep syntax — important to know for fingerprinting:

| Database | Sleep Syntax |
|---|---|
| PostgreSQL | `pg_sleep(10)` |
| MySQL | `SLEEP(10)` |
| Microsoft SQL Server | `WAITFOR DELAY '0:0:10'` |
| Oracle | `dbms_pipe.receive_message(('a'),10)` |

---

## Step-by-Step Walkthrough

### Step 1: Intercept the Request in Burp Suite

1. Open Burp Suite → **Proxy → Intercept**
2. Open the Burp browser, access the lab, browse to any product category
3. Intercept the GET request containing the `TrackingId` cookie
4. Send to **Repeater** (right-click → Send to Repeater)

---

### Step 2: Observe the Baseline Behaviour

Try injecting anything — errors, quotes, anything:
```sql
TrackingId=xyz'
TrackingId=xyz' AND 1=1--
TrackingId=xyz' AND 1=2--
```

**All return HTTP 200 OK** with identical responses.

> This is what makes this lab harder than the previous two — there is **no visible signal at all**. No errors, no content difference. Time is the only channel left.

---

### Step 3: Inject a Time Delay

Append `pg_sleep()` using PostgreSQL's concatenation operator `||`:
```sql
TrackingId=xyz'||pg_sleep(10)--
```

Send the request and **watch the response time** in Burp Repeater (bottom right shows milliseconds).

- Response takes ~10 seconds longer than usual → **injection confirmed** ✅
- Response returns instantly → syntax wrong or different database

> **Why `||`?** Just like in the Oracle lab, `||` is the string concatenation operator. In PostgreSQL it works the same way — it appends your injected expression to the TrackingId value. The database evaluates `pg_sleep(10)` as part of processing the concatenation.

---

### Step 4: Solve the Lab

The lab only requires you to **prove** that time-based injection works. Send this final payload:

```sql
TrackingId=xyz'||(SELECT pg_sleep(10))--
```

Wrapping `pg_sleep` in a `SELECT` subquery is cleaner and more reliable. The 10-second delay confirms blind time-based SQLi — **lab solved** ✅

---

## Full Payload Reference

```sql
-- Basic time delay (simplest form)
TrackingId=xyz'||pg_sleep(10)--

-- Cleaner subquery form (preferred)
TrackingId=xyz'||(SELECT pg_sleep(10))--

-- Conditional time delay (used in next lab for data extraction)
TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END)--
```

---

## How This Compares to Previous Labs

| Feature | Conditional Errors (Lab 1) | Visible Errors (Lab 2) | Time Delays (This Lab) |
|---|---|---|---|
| Database | Oracle | PostgreSQL | PostgreSQL |
| Signal | HTTP 500 vs 200 | Error message text | Response time |
| Data extraction | Character brute force | Full value in error | Time-based brute force |
| Complexity | Medium | Easy | Medium |
| Burp tool needed | Intruder | Repeater only | Repeater (+ Intruder for next lab) |

---

## Important Notes

**Why is time-based injection useful even if you can only confirm injection exists?**  
This lab just proves the concept. The **next lab** (Blind SQLi with time delays and information retrieval) builds on this — you use conditional `CASE WHEN` logic with `pg_sleep` to extract data character by character, similar to the conditional errors lab but using delay instead of errors as your signal.

**Watch out for network latency** — if your internet connection is slow or unstable, a normal request might sometimes take a few extra seconds. Always test with a clear baseline response time before injecting, and use a long enough delay (10 seconds) so it's unmistakably different from normal latency.

**Burp Repeater shows response time** in the bottom right corner of the response panel — keep an eye on it when testing time-based payloads.

---

## Key Takeaways

- When no output and no errors are visible, **response time becomes your only signal**
- `pg_sleep(N)` pauses PostgreSQL execution for N seconds
- Every database has its own sleep function — knowing which one to use helps fingerprint the database
- Time-based injection is the **last resort** technique when all other channels are blind
- This lab is a foundation for the next one where you'll use time delays to actually **extract data**

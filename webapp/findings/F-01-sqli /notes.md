# F-01 — SQL Injection | Finding Notes

## Finding Summary

| Field             | Detail                                      |
|-------------------|---------------------------------------------|
| ID                | F-01                                        |
| Title             | SQL Injection — User Lookup Endpoint        |
| Application       | DVWA                                        |
| Module            | SQL Injection                               |
| Security Level    | Low                                         |
| URL               | http://localhost:8080/vulnerabilities/sqli/ |
| Parameter         | `id` (GET)                                  |
| OWASP 2021        | A03 — Injection                             |
| Severity          | Critical                                    |
| Tester            | Indrajit Singh                              |

---

## Root Cause

The `id` parameter from the user's GET request is concatenated directly
into a SQL query string on the server side without any sanitisation,
escaping, or parameterisation — so whatever the user types becomes part
of the executable SQL sent to the database.

The vulnerable PHP code pattern looks like this:
```php
$query = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
```
There is nothing between the user's input and the database engine.

---

## Business Impact

At Meridian FinServe, this vulnerability on the customer portal's
account-lookup or EMI-servicing endpoint would allow an unauthenticated
attacker to extract the names, account numbers, and loan balances of all
180,000 borrowers from the database in a single automated request,
directly enabling identity fraud and regulatory breach under RBI data
protection guidelines.

---

## Remediation (Summary)

Replace string concatenation with a **prepared statement / parameterised
query**. In PHP with PDO:

```php
// VULNERABLE (current)
$query = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";

// FIXED
$stmt = $pdo->prepare("SELECT first_name, last_name FROM users WHERE user_id = ?");
$stmt->execute([$id]);
```

The database engine now treats `$id` as pure data, never as SQL logic,
regardless of what characters it contains.

Code-level remediation will be implemented during the SAST and remediation phase

---

## Evidence

- Screenshot: `screenshots/day3-sqli-payload-result.png`
- Request/response: `webapp/findings/F-01-sqli/request.txt`
- Payload log: `webapp/findings/F-01-sqli/payload.md`

---

## References

- OWASP Top 10 2021 — A03 Injection:
  https://owasp.org/Top10/A03_2021-Injection/
- OWASP SQL Injection Prevention Cheat Sheet:
  https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
- CWE-89: Improper Neutralization of Special Elements in SQL Command:
  https://cwe.mitre.org/data/definitions/89.html

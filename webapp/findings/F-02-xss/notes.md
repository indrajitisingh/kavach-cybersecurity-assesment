# F-02 — Reflected XSS | Finding Notes

## Finding Summary

| Field             | Detail                                           |
|-------------------|--------------------------------------------------|
| ID                | F-02                                             |
| Title             | Reflected Cross-Site Scripting (XSS)             |
| Application       | DVWA                                             |
| Module            | XSS (Reflected)                                  |
| Security Level    | Low                                              |
| URL               | http://localhost:8080/vulnerabilities/xss_r/     |
| Parameter         | `name` (GET)                                     |
| OWASP 2021        | A03 — Injection                                  |
| Severity          | High                                             |
| Tester            | Indrajit Singh                                   |

---

## Root Cause

The application takes the `name` parameter from the GET request and
concatenates it directly into the HTML response using PHP's echo
statement, with no output encoding applied — so any HTML or JavaScript
characters in the input are interpreted by the browser as markup rather
than displayed as text.

The vulnerable server-side pattern:
```php
// VULNERABLE
echo "<pre>Hello " . $_GET['name'] . "</pre>";
```

---

## Business Impact

At Meridian FinServe, an XSS vulnerability on the customer portal's
loan-status or EMI-servicing pages would allow an attacker to craft a
malicious link and send it to a borrower — when clicked, the attacker's
script runs silently in the borrower's browser, stealing their session
token and giving the attacker full authenticated access to that
borrower's account, loan history, and payment details without needing
their password.

---

## Remediation (Summary)

Apply `htmlspecialchars()` with the correct encoding flag to every
variable that is reflected into HTML output:

```php
// FIXED
echo "<pre>Hello " . htmlspecialchars($_GET['name'], ENT_QUOTES, 'UTF-8') . "</pre>";
```

`htmlspecialchars()` converts `<`, `>`, `"`, `'`, and `&` into their
HTML entity equivalents (`&lt;`, `&gt;`, etc.) so the browser displays
them as text rather than executing them as markup.

Additionally, a Content Security Policy (CSP) header should be set to
restrict which scripts the browser will execute:
```
Content-Security-Policy: default-src 'self'
```

Full code-level patch is on branch: `fix/F-02-xss-output-encoding`

---

## Evidence

- Screenshot: `screenshots/day4-xss-alert-execution.png`
- Request/response: `webapp/findings/F-02-xss-reflected/request.txt`
- Payload log: `webapp/findings/F-02-xss-reflected/payload.md`

---

## References

- OWASP Top 10 2021 — A03 Injection:
  https://owasp.org/Top10/A03_2021-Injection/
- OWASP XSS Prevention Cheat Sheet:
  https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html
- CWE-79: Improper Neutralization of Input During Web Page Generation:
  https://cwe.mitre.org/data/definitions/79.html

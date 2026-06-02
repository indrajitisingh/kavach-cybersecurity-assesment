# F-03 — Brute Force Attack | Finding Notes

## Finding Summary

| Field             | Detail                                              |
|-------------------|-----------------------------------------------------|
| ID                | F-01                                                |
| Title             | Authentication Failure via Brute Force              |
| Application       | DVWA                                                |
| Module            | Brute Force                                         |
| Security Level    | Low                                                 |
| URL               | http://localhost:8080/vulnerabilities/brute/        |
| Parameter         | `username`, `password` (GET)                        |
| OWASP 2021        | A07 — Identification and Authentication Failures    |
| Severity          | High                                           |
| Tester            | Indrajit Singh                                      |

---

## Root Cause

The application's login endpoint accepts unlimited authentication
attempts with no rate limiting, account lockout, CAPTCHA, or
multi-factor authentication enforcement. Credentials are submitted through URL 
query parameters using a GET request, casuing username and passwords to appears
in browser history, proxy logs, and server logs.

Manual testing demonstrated that repeated failed login attempts were processed normally 
without any warning , delays , throttling, or temporary account locakout .
As a result, the authentication mechansim is vulnerable to password-guessing and brute-force
attacks, allowing an attackers to repeatedly test credentials until valid accounts access is obtained.
---

## Business Impact

At Meridian FinServe, an authentication brute-force vulnerability
on any customer-facing or internal portal would allow an attacker
to systematically guess passwords for known usernames (harvested
from phishing, OSINT, or previous data breaches), gain unauthorised
access to customer accounts or staff portals, and use those
sessions to initiate fraudulent transactions, exfiltrate KYC
documents, or escalate privileges to administrative functions —
all without requiring any prior knowledge of the actual password.

---

## Remediation (Summary)

Implement progressive account lockout, rate limiting, and detection
on the authentication endpoint:

```php
// FIXED — Enforce account lockout after N failed attempts
session_start();
$max_attempts  = 5;
$lockout_time  = 15 * 60; // 15 minutes in seconds

if (!isset($_SESSION['failed_attempts'])) {
    $_SESSION['failed_attempts'] = 0;
    $_SESSION['first_attempt']   = time();
}

// Reset window if lockout period has passed
if (time() - $_SESSION['first_attempt'] > $lockout_time) {
    $_SESSION['failed_attempts'] = 0;
    $_SESSION['first_attempt']   = time();
}

if ($_SESSION['failed_attempts'] >= $max_attempts) {
    die("Account temporarily locked. Please try again later.");
}

// ... perform credential check ...

if ($login_failed) {
    $_SESSION['failed_attempts']++;
    sleep(2); // Add artificial delay to slow automation
    die("Invalid credentials.");
} else {
    $_SESSION['failed_attempts'] = 0; // Reset on success
}
```

Additional hardening measures:
- Switch login form to POST and enforce HTTPS to prevent credentials
  appearing in server logs, browser history, and HTTP Referer headers
- Integrate CAPTCHA or proof-of-work challenge after 3 failed
  attempts from the same IP / session
- Enable server-side rate limiting at the web server or WAF layer
  (e.g. nginx `limit_req_zone`, ModSecurity rule)
- Implement multi-factor authentication for privileged accounts
- Alert the account owner via email/SMS on repeated failures




---

## Evidence

- Screenshot: `screenshots/ authentication-failure-success.png`
- Request/response: `webapp/findings/F-03-brute-force/request.txt`
- Payload log: `webapp/findings/F-03-brute-force/payload.md`

---

## References

- OWASP Top 10 2021 — A07 Identification and Authentication Failures:
  https://owasp.org/Top10/A07_2021-Identification_and_Authentication_Failures/
- OWASP Authentication Cheat Sheet:
  https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
- OWASP Credential Stuffing Prevention Cheat Sheet:
  https://cheatsheetseries.owasp.org/cheatsheets/Credential_Stuffing_Prevention_Cheat_Sheet.html
- CWE-307: Improper Restriction of Excessive Authentication Attempts:
  https://cwe.mitre.org/data/definitions/307.html
- CWE-521: Weak Password Requirements:
  https://cwe.mitre.org/data/definitions/521.html

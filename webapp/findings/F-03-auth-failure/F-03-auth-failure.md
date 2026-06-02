# F-03 — Brute Force Attack | Payload Progression

## Target
- Application: DVWA (Damn Vulnerable Web Application)
- Module: Brute Force
- Security Level: Low
- Login URL: `http://localhost:8080/vulnerabilities/brute/`

---

## Step 1 — Reconnaissance: Understanding the Login Mechanism

**What I tried first:**
Submitted the login form with an obviously wrong password for user
`admin` while observing the URL in the browser address bar.

**What happened:**
The page reloaded and the URL became:
```
http://localhost:8080/vulnerabilities/brute/?username=admin&password=wrongpass&Login=Login
```

**Key observations:**
1. The login form uses a GET request — credentials appear in the URL
2. The query string parameters are `username` and `password`
3. The server returned the page with the message:
   ```
   Username and/or password incorrect.
   ```
4. Because credentials travel in the GET query string, they are
   visible in server access logs, browser history, and any
   intermediate proxies — a secondary security weakness

---

## Step 2 — Identifying the Success Indicator

**What I tried:**
Logged in with the correct credentials (`admin` / `password`) to
record what a successful response looks like.

**What happened:**
The page displayed:
```
Welcome to the password protected area admin
```
accompanied by a user image. The URL pattern for success is:
```
http://localhost:8080/vulnerabilities/brute/?username=admin&password=password&Login=Login
```

This success/failure distinction in the response body is the
oracle that an automated tool uses to detect when the correct
password has been found.

---

## Step 3 — Manual Confirmation of No Rate Limiting

**What I tried:**
Submitted the login form ten times in rapid succession with
incorrect passwords.

**What happened:**
Every request received a normal HTTP 200 response with the
"incorrect" message — no lockout, no CAPTCHA, no delay, no
warning. The application imposes zero throttling or account
lockout.

---



---

## Step 4 — Manual Verification

**What I tried:**
Navigated directly to:
```
http://localhost:8080/vulnerabilities/brute/?username=admin&password=password&Login=Login
```

**What happened:**
```
Welcome to the password protected area admin
```

The brute-forced credential set was valid and granted full access
to the protected area. Screenshot saved as:
`screenshots/bruteforce-login-success.png`

---

## Why DVWA Low Security Has No Defences

At Low security, the DVWA Brute Force module uses a bare PHP
`mysqli` query with no session-based attempt counter, no IP-based
throttle, and no lockout mechanism. The login endpoint is
functionally equivalent to:

```php
// VULNERABLE — no rate limiting, no lockout
$query  = "SELECT * FROM `users` WHERE user = '$user' AND password = '$pass';";
$result = mysqli_query($GLOBALS["___mysqli_ston"], $query);

if ($result && mysqli_num_rows($result) == 1) {
    echo "<p>Welcome to the password protected area {$user}</p>";
} else {
    echo "<pre><br />Username and/or password incorrect.</pre>";
}
```

Any loop or automation tool can send thousands of requests per
minute with no consequence.

---

## Final Working Payload

**Successful credential pair discovered:**
```
username = admin
password = password
```

**Verified execution URL:**
```
http://localhost:8080/vulnerabilities/brute/?username=admin&password=password&Login=Login
```

- **Effect:** Unauthorised access to a password-protected area
- **Vulnerability:** No account lockout / no rate limiting on login
- **OWASP Category:** A07:2021 — Identification and Authentication Failures
- **DVWA Security Level tested:** Low

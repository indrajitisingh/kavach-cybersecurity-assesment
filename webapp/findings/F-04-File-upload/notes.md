# F-04 — Unrestricted File Upload | Finding Notes

## Finding Summary

| Field             | Detail                                           |
|-------------------|--------------------------------------------------|
| ID                | F-04                                             |
| Title             | Unrestricted File Upload → Remote Code Execution |
| Application       | DVWA                                             |
| Module            | File Upload                                      |
| Security Level    | Low                                              |
| Upload URL        | http://localhost:8080/vulnerabilities/upload/    |
| Shell URL         | http://localhost:8080/hackable/uploads/shell.php |
| Parameter         | `uploaded` (POST multipart form)                 |
| OWASP 2021        | A05 — Security Misconfiguration /CW-434                                |
| Severity          | Critical                                         |
| Tester            | Indrajit Singh                                   |

---

## Root Cause

The file upload handler accepts any file type without validating
the file extension, MIME type, or file contents, and stores uploaded
files in a directory that is directly served by the web server —
so an uploaded PHP file is immediately accessible as an executable
URL, giving an attacker a persistent command execution channel on
the server.

The vulnerable PHP pattern:
```php
// VULNERABLE — no validation, stores in web-accessible path
move_uploaded_file($_FILES['uploaded']['tmp_name'],
    DVWA_WEB_PAGE_TO_ROOT . "hackable/uploads/" . $_FILES['uploaded']['name']);
```

---

## Business Impact

At Meridian FinServe, an unrestricted file upload vulnerability on
the customer portal or partner merchant onboarding interface would
allow an attacker to deploy a persistent web shell on the server,
granting ongoing remote command execution under the web application's
OS account — from which they could exfiltrate the entire customer
database, pivot to internal data center systems, or install ransomware
affecting all 22,000 merchant and 180,000 borrower records, with no
authentication required after the initial upload.

---

## Remediation (Summary)

Three controls must work together — any one alone is bypassable:

**1. Validate file extension on the server side (allowlist, not blocklist)**
```php
$allowed_extensions = ['jpg', 'jpeg', 'png', 'gif'];
$ext = strtolower(pathinfo($_FILES['uploaded']['name'], PATHINFO_EXTENSION));
if (!in_array($ext, $allowed_extensions)) {
    die("File type not permitted.");
}
```

**2. Validate actual file content using PHP's finfo (magic bytes)**
```php
$finfo = new finfo(FILEINFO_MIME_TYPE);
$mime = $finfo->file($_FILES['uploaded']['tmp_name']);
$allowed_mimes = ['image/jpeg', 'image/png', 'image/gif'];
if (!in_array($mime, $allowed_mimes)) {
    die("File content does not match an allowed type.");
}
```

**3. Store uploads outside the web root and serve via a controller**
```
/var/uploads/          ← outside /var/www/html, not web-accessible
/var/www/html/serve.php?file=abc123  ← controlled serving script
```
Even if an attacker uploads a PHP file, it cannot be executed
because the web server has no path to it.



---

## Evidence

- Screenshot: `screenshots/day4-file-upload-shell-execution.png`
- Request/response: `webapp/findings/F-04-file-upload/request.txt`
- Payload log: `webapp/findings/F-04-file-upload/payload.md`

---

## References

- OWASP File Upload Cheat Sheet:
  https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html
- OWASP Top 10 2021 — A03 Injection:
  https://owasp.org/Top10/A03_2021-Injection/
- CWE-434: Unrestricted Upload of File with Dangerous Type:
  https://cwe.mitre.org/data/definitions/434.html

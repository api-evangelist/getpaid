---
name: Onboard a seller
description: Create a Getpaid onboarding application for a seller, drive it to completion, and confirm the provisioned account.
api: openapi/getpaid-openapi-original.yml
operations: [create_application, get_application, refresh_application, query_accounts]
---

# Onboard a seller

Use this to bring a new merchant/seller onto the platform (KYC/KYB) before they can be paid.

## Auth
- OAuth 2.0 client-credentials token, scope `accounts:read_write`.
- Send `Getpaid-Trace-Id` on every request.

## Steps
1. **create_application** — `POST /v2/applications`. Provide the seller's legal details and country. Send a `Getpaid-Idempotency-Key`. The response returns an `ApplicationId` and the hosted onboarding URL.
2. Direct the seller through the hosted onboarding (identity verification / KYB). Await the `application_submitted` then `application_completed` webhooks.
3. **get_application** — `GET /v2/applications/{application_id}`. Track `status` through `initiated → identity_verified → submitted → screened → approved → completed`.
4. If the application expires before completion, **refresh_application** — `POST /v2/applications/{application_id}/refresh` to extend it.
5. **query_accounts** — `POST /v2/accounts/query` to confirm the provisioned `Account` for the completed application.

## Rules
- In sandbox, onboarding is mocked (no real KYC/KYB checks).
- Use `cancel_application` (`POST /v2/applications/{application_id}/cancel`) to abandon an in-flight application.
- Errors are RFC 7807 `application/problem+json`.

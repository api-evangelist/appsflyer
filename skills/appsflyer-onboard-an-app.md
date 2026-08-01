---
name: Onboard an app and register a test device
description: Add a new app to an AppsFlyer account, confirm it appears in the app list, then register a test device
  and read back the events it produces.
api: openapi/appsflyer-app-management-api-v20-openapi.yml
operations:
- app-mng-v2-post
- app-list-ad-nets-api-get
- test-console-api-post
- test-console-api-get
- test-console-api-delete
also:
- openapi/appsflyer-app-list-api-openapi.yml
- openapi/appsflyer-test-console-api-openapi.yml
generated: '2026-07-31'
method: generated
---

# Onboard an app and register a test device

Add a new app to an AppsFlyer account, confirm it appears in the app list, then register a test device and read back the events it produces.

## Authentication

Every call below uses the AppsFlyer **API V2 token** (a JWT), issued by an account admin from the AppsFlyer
dashboard Security Center and shown exactly once:

```
Authorization: Bearer <API_V2_TOKEN>
```

Do not fall back to the legacy `api_token` query parameter — those V1 endpoints are superseded, and every
API V2 token issued before 2026-03-10 19:00 UTC was revoked. Full profile: `authentication/appsflyer-authentication.yml`.

## Steps

1. **Add the app** — `app-mng-v2-post` → `POST https://hq1.appsflyer.com/api/app/v2.0/apps/`.
   The request body is a `oneOf` over three shapes: `available-app` (already live in a store, send
   `app_url`), `pending-app` (not live yet, send `app_id` + `country`) and `out-of-store-app`.
   All three require `status`, `platform`, `time_zone` and `currency`.
2. **Confirm it landed** — `app-list-ad-nets-api-get` → `GET https://hq1.appsflyer.com/api/mng/apps`.
   This is paginated with `limit`/`offset` and returns at most 1,000 records per page.
3. **Register a test device** — `test-console-api-post` →
   `POST https://hq1.appsflyer.com/api/test-console/v1.0/app/{app-id}/devices`. Until a device is registered
   its installs are attributed as production traffic, so do this before any manual testing.
4. **Read the test events** — `test-console-api-get` → `GET .../{app-id}/events`.
5. **Clean up** — `test-console-api-delete` → `DELETE .../{app-id}/devices/{device-id}`.

Prefer a dedicated **debug app** (its own app ID and dashboard instance, never published to a store) over
testing against the production app: https://dev.appsflyer.com/hc/docs/integration-testing

## Failure handling

- `409` on step 1 means the app already exists on the account — go to step 2 rather than retrying.
- `429`/`402` on step 1 means the app-creation rate limit or the account app quota is exhausted.

## Rules that apply to every step

- **No idempotency.** AppsFlyer supports no idempotency key on any operation. Never blind-retry a write —
  re-read state first (`conventions/appsflyer-conventions.yml`).
- **Errors** are a proprietary JSON envelope `{status, code, message, request_id}`, not RFC 9457
  `application/problem+json`. Always capture `request_id`; it is what AppsFlyer support asks for.
- **Rate limits** surface as HTTP 429 with no `Retry-After` and no `X-RateLimit-*` headers. Report generation
  is quota'd per report, per app, per day — back off exponentially and treat 402 as "quota exhausted", not
  "payment required".
- **Versioning is in the path** (`/v2.0`, `/v5`, `/v3`). There are no `Sunset`/`Deprecation` headers, so pin
  the version you tested against.

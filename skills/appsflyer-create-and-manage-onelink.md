---
name: Create, inspect and retire a OneLink short link
description: Mint a OneLink attribution/deep-link short link, fetch its data and QR code, check the account quota,
  update it, and delete it when done.
api: openapi/appsflyer-onelink-api-v20-openapi.yml
operations:
- get-onelink-v2-link-quota
- onelink-v2-create-link
- get-onelink-v2-link
- get-onelink-v2-link-qr
- update-onelink-v2-link
- delete-onelink-v2-link
generated: '2026-07-31'
method: generated
---

# Create, inspect and retire a OneLink short link

Mint a OneLink attribution/deep-link short link, fetch its data and QR code, check the account quota, update it, and delete it when done.

## Authentication

Every call below uses the AppsFlyer **API V2 token** (a JWT), issued by an account admin from the AppsFlyer
dashboard Security Center and shown exactly once:

```
Authorization: Bearer <API_V2_TOKEN>
```

Do not fall back to the legacy `api_token` query parameter — those V1 endpoints are superseded, and every
API V2 token issued before 2026-03-10 19:00 UTC was revoked. Full profile: `authentication/appsflyer-authentication.yml`.

## Steps

Base: `https://onelink.appsflyer.com/api/v2.0/`. This API authenticates with the **OneLink REST API token in an
`authorization` header**, not the account-wide V2 bearer token — see
https://support.appsflyer.com/hc/en-us/articles/360001250345-OneLink-API

1. **Check headroom first** — `get-onelink-v2-link-quota` → `GET /shortlinks-quota/{account-id}`.
   Short links are quota'd per account; a 429 on create means you have exhausted it.
2. **Create the link** — `onelink-v2-create-link` → `POST /shortlinks/{onelink-id}`.
   The body carries the deep-link `data` map, an optional `ttl`, and an optional branded domain.
3. **Read it back** — `get-onelink-v2-link` → `GET /shortlinks/{onelink-id}/{shortlink-id}`.
   Do this before any update: there is no idempotency key, so a re-issued create makes a *second* link.
4. **QR code**, when you need a printable/scannable artifact — `get-onelink-v2-link-qr` →
   `GET /shortlinks/{onelink-id}/{shortlink-id}/qr`.
5. **Update** — `update-onelink-v2-link` → `PUT /shortlinks/{onelink-id}/{shortlink-id}`.
6. **Delete** — `delete-onelink-v2-link` → `DELETE /shortlinks/{onelink-id}/{shortlink-id}`.

## Failure handling

- `429` on create/update/delete is documented as "Limit exceeded" — re-check the quota from step 1.
- `404` on read means the short link ID is wrong or the link was already deleted; do not recreate blindly.

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

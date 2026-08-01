---
name: Send server-to-server and web events
description: Send install, session and in-app events to AppsFlyer from a backend, a web property, or a PC/console/CTV
  client, using the sandbox host first.
api: openapi/appsflyer-server-to-server-events-api-for-mobile-openapi.yml
operations:
- s2s-events-api3-post
- web-s2s-api-event-post
- web-s2s-api-setcuid-post
- ctv-c2s-events-inapps-post
- ctv-c2s-events-sessions-post
also:
- openapi/appsflyer-web-server-to-server-api-openapi.yml
- openapi/appsflyer-pcconsolectv-events-api-openapi.yml
- openapi/appsflyer-pcconsolectv-client-app-events-api-openapi.yml
generated: '2026-07-31'
method: generated
---

# Send server-to-server and web events

Send install, session and in-app events to AppsFlyer from a backend, a web property, or a PC/console/CTV client, using the sandbox host first.

## Authentication

Every call below uses the AppsFlyer **API V2 token** (a JWT), issued by an account admin from the AppsFlyer
dashboard Security Center and shown exactly once:

```
Authorization: Bearer <API_V2_TOKEN>
```

Do not fall back to the legacy `api_token` query parameter — those V1 endpoints are superseded, and every
API V2 token issued before 2026-03-10 19:00 UTC was revoked. Full profile: `authentication/appsflyer-authentication.yml`.

## Steps

These APIs do **not** use the account V2 bearer token. Mobile S2S authenticates with the app **Dev Key** in an
`authentication` header; the PC/Console/CTV endpoints use a per-product token in an `authorization` header.

1. **Test against the sandbox first.** PC/Console/CTV has real sandbox hosts that mirror production exactly:
   `https://sandbox-events.appsflyer.com/v1.0/s2s/` and `https://sandbox-events.appsflyer.com/v1.0/c2s/`.
   There is no sandbox for the mobile S2S API — use a debug app instead.
2. **Mobile in-app event** — `s2s-events-api3-post` → `POST https://api3.appsflyer.com/inappevent/{app_id}`.
   Send `appsflyer_id` (or `customer_user_id`), `eventName`, `eventValue`, `eventTime` and the device/ad
   identifiers. `api2.appsflyer.com` is the **legacy** host — do not build against it.
3. **Web** — `web-s2s-api-setcuid-post` → `POST https://webs2s.appsflyer.com/v1/{bundleId}/setcuid` to bind your
   customer user ID to the AppsFlyer user ID, then `web-s2s-api-event-post` →
   `POST https://webs2s.appsflyer.com/v1/{bundleId}/event`.
4. **PC/Console/CTV** — `ctv-c2s-events-sessions-post` → `POST /session/app/{platform}/{app-id}` for sessions and
   `ctv-c2s-events-inapps-post` → `POST /inapp/app/{platform}/{app-id}` for in-app events; the S2S variant adds
   `POST /first_open/app/{platform}/{app-id}`.

## Failure handling

There is no idempotency key. **Send a stable, caller-generated event identifier and an accurate `eventTime`**
so AppsFlyer's own deduplication can absorb a retry — a naive retry after a timeout will otherwise double-count.
A `401` here almost always means the Dev Key, not the V2 token, is the wrong credential for this endpoint.

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

---
name: Roll out and rotate Protect360 click signing
description: Generate a click-signing secret, configure signature versions and excluded apps, test signing, read
  the report, and revoke a compromised key.
api: openapi/appsflyer-click-signing-api-openapi.yml
operations:
- click-signing-config-get
- click-signing-secret-post
- click-signing-config-update-put
- click-signing-test-post
- click-signing-config-excluded-apps-add
- click-signing-config-excluded-apps-delete
- click-signing-config-circuit-breaker-put
- click-signing-report-get
- click-signing-secret-delete
generated: '2026-07-31'
method: generated
---

# Roll out and rotate Protect360 click signing

Generate a click-signing secret, configure signature versions and excluded apps, test signing, read the report, and revoke a compromised key.

## Authentication

Every call below uses the AppsFlyer **API V2 token** (a JWT), issued by an account admin from the AppsFlyer
dashboard Security Center and shown exactly once:

```
Authorization: Bearer <API_V2_TOKEN>
```

Do not fall back to the legacy `api_token` query parameter — those V1 endpoints are superseded, and every
API V2 token issued before 2026-03-10 19:00 UTC was revoked. Full profile: `authentication/appsflyer-authentication.yml`.

## Steps

Base: `https://hq1.appsflyer.com/api/p360-click-signing/v2.0`.

1. **Read current state** — `click-signing-config-get` → `GET /config`.
2. **Generate a secret** — `click-signing-secret-post` → `POST /secret`. A **429 here means "too many active
   keys"**, not throttling — revoke an old key first. Capture the returned key material immediately.
3. **Exclude any app that is not ready** — `click-signing-config-excluded-apps-add` →
   `POST /config/excluded-apps/{app-id}`. Do this *before* enforcing, or unsigned clicks from that app get
   rejected.
4. **Test before enforcing** — `click-signing-test-post` → `POST /test`.
5. **Set the signature version** — `click-signing-config-update-put` →
   `PUT /config/signatures/{signature_version}`.
6. **Watch the outcome** — `click-signing-report-get` → `GET /report`.
7. **Emergency stop** — `click-signing-config-circuit-breaker-put` → `PUT /config/circuit-breaker` disables
   enforcement without tearing down the configuration. Reach for this before deleting keys.
8. **Rotate out** — `click-signing-secret-delete` → `DELETE /secret/{secret-key-id}`, and
   `click-signing-config-excluded-apps-delete` → `DELETE /config/excluded-apps/{app-id}` when an app is ready.

## Failure handling

This is an **enforcement** surface: a wrong configuration silently drops real attribution traffic. Always
`GET /config` and `GET /report` after a write, and prefer the circuit breaker over key deletion when
something looks wrong.

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

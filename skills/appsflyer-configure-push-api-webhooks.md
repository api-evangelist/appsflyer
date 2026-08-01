---
name: Configure Push API webhooks (postbacks)
description: Validate a receiving endpoint, set the Push API authentication token, and enable real-time install
  and in-app event postbacks for an app.
api: openapi/appsflyer-push-api-configuration-api-openapi.yml
operations: []
generated: '2026-07-31'
method: generated
---

# Configure Push API webhooks (postbacks)

Validate a receiving endpoint, set the Push API authentication token, and enable real-time install and in-app event postbacks for an app.

## Authentication

Every call below uses the AppsFlyer **API V2 token** (a JWT), issued by an account admin from the AppsFlyer
dashboard Security Center and shown exactly once:

```
Authorization: Bearer <API_V2_TOKEN>
```

Do not fall back to the legacy `api_token` query parameter — those V1 endpoints are superseded, and every
API V2 token issued before 2026-03-10 19:00 UTC was revoked. Full profile: `authentication/appsflyer-authentication.yml`.

## Steps

Base: `https://hq1.appsflyer.com/api/pushapi/v1.0/`. These operations carry **no `operationId`** in the published
spec — address them by method + path.

1. **Discover what you can subscribe to** —
   `GET /event-types/{attributing-entity}` where the entity is the regular (iOS/Android) or `skadnetwork` attributor, and
   `GET /fields/{platform}` for the message fields available on that platform.
2. **Validate your receiver before saving it** — `POST /validate-url`. The response is
   `{tested_endpoint_url_http_response: <int>}`; only proceed on a 2xx. URLs are capped at 2,048 characters.
3. **Set the authentication token** — `PUT /tokens/app/{app_id}`. AppsFlyer sends this token on every postback
   so your receiver can verify the sender. Do this **before** enabling delivery.
4. **Write the configuration** — `PUT /app/{app-id}` with `url`, `method` (`GET` or `POST`),
   `attributing_entity`, `event_types` and `enabled`. `selected_fields` accepts the literal `"all"` or an
   explicit field list.
   - Regular event types: `install`, `organic-install`, `reinstall`, `organic-reinstall`, `re-engagement`,
     `re-attribution`, `install-in-app-event`, `organic-install-in-app-event`, `re-engagement-in-app-event`,
     `re-attribution-in-app-event`.
   - SKAdNetwork event types: `install`, `re-download`, `in-app-event`, `postback`, `postbacks-copy`.
5. **Read back** — `GET /app/{app-id}` and confirm the configuration you intended is what is stored.
6. **Rotate or revoke** — re-`PUT /tokens/app/{app_id}` to rotate, `DELETE /tokens/app/{app_id}` to revoke.

There is **no AsyncAPI document** for this surface; the event catalog above is the enum set from the published
OpenAPI. See `asyncapi/appsflyer-push-api-webhooks.yml`.

## Failure handling

`PUT /app/{app-id}` is not idempotent-keyed but it is a full replacement — re-sending the same body is safe.
Adding an event type requires sending the **whole** `event_types` array, not a delta.

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

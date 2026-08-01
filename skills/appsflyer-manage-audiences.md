---
name: List, import and pause audiences
description: Read the active audiences on an account, import audience membership and user attributes, attach additional
  identifiers, and pause an audience.
api: openapi/appsflyer-audience-external-api-openapi.yml
operations:
- audience-external-active-audiences
- audience-external-pause
- audience-import-post
- audiences-user-attr-import-post
- audience-put-additional-identifier
also:
- openapi/appsflyer-audience-import-api-openapi.yml
- openapi/appsflyer-audiences-user-attribution-import-api-openapi.yml
- openapi/appsflyer-additional-identifiers-api-openapi.yml
generated: '2026-07-31'
method: generated
---

# List, import and pause audiences

Read the active audiences on an account, import audience membership and user attributes, attach additional identifiers, and pause an audience.

## Authentication

Every call below uses the AppsFlyer **API V2 token** (a JWT), issued by an account admin from the AppsFlyer
dashboard Security Center and shown exactly once:

```
Authorization: Bearer <API_V2_TOKEN>
```

Do not fall back to the legacy `api_token` query parameter — those V1 endpoints are superseded, and every
API V2 token issued before 2026-03-10 19:00 UTC was revoked. Full profile: `authentication/appsflyer-authentication.yml`.

## Steps

1. **See what exists** — `audience-external-active-audiences` →
   `GET https://hq1.appsflyer.com/api/audiences-external-api/active-audiences`.
   The connections and split-sync endpoints on the same base (`/audience/{audience_id}/connections`,
   `/audience/{audience_id}/split_syncs`, `/connections`, `/split_syncs`, `/audience/{audience_id}/upload_now`,
   `POST /audience`) carry **no `operationId`** in the published spec — address them by method + path.
2. **Import membership** — `audience-import-post` →
   `POST https://hq1.appsflyer.com/api/audiences-import-api/v2/{action}`.
3. **Import user attributes** — `audiences-user-attr-import-post` →
   `POST https://hq1.appsflyer.com/api/user-attributes-import-api/set-user-data`.
4. **Attach extra identifiers** — `audience-put-additional-identifier` →
   `PUT https://hq1.appsflyer.com/api/audience-bulk-api/v1/additional-identifiers/app/{app-id}`.
5. **Pause** — `audience-external-pause` → `POST .../audiences-external-api/pause`.

## Failure handling

Imports are bulk and **not idempotent**. Re-running the same import re-applies the membership rather than
being ignored, so record what you sent. `429` on import means the audience-import rate limit was hit.

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

---
name: Manage users in bulk and read the audit log
description: Create, list and delete AppsFlyer account users in bulk and pull the account audit log for compliance
  evidence.
api: openapi/appsflyer-user-management-openapi.yml
operations:
- bulk-users-get
- bulk-users-post
- bulk-users-delete
- audit-public-api-get
also:
- openapi/appsflyer-audit-public-api-openapi.yml
generated: '2026-07-31'
method: generated
---

# Manage users in bulk and read the audit log

Create, list and delete AppsFlyer account users in bulk and pull the account audit log for compliance evidence.

## Authentication

Every call below uses the AppsFlyer **API V2 token** (a JWT), issued by an account admin from the AppsFlyer
dashboard Security Center and shown exactly once:

```
Authorization: Bearer <API_V2_TOKEN>
```

Do not fall back to the legacy `api_token` query parameter — those V1 endpoints are superseded, and every
API V2 token issued before 2026-03-10 19:00 UTC was revoked. Full profile: `authentication/appsflyer-authentication.yml`.

## Steps

1. **Read before you write** — `bulk-users-get` →
   `GET https://hq1.appsflyer.com/api/user-management/v1.0/users`.
2. **Create in bulk** — `bulk-users-post` → `POST .../users`. There is no idempotency key: re-posting a batch
   after a timeout can produce duplicate invitations, so reconcile with step 1 first.
3. **Remove** — `bulk-users-delete` → `DELETE .../users/{userIds}`.
4. **Evidence** — `audit-public-api-get` → `GET https://hq1.appsflyer.com/api/audit/v1.0/logs`.
   Paginated with `limit`/`offset`. The Enterprise-Grade Security package extends audit-log retention to 180
   days and supports SIEM streaming: https://www.appsflyer.com/products/enterprise-grade-security/

This is not SCIM 2.0 — it is a proprietary bulk-users API, so an off-the-shelf SCIM provisioning connector
will not work against it.

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

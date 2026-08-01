---
name: Handle an OpenDSR privacy request end to end
description: Submit, poll, download and cancel an IAB OpenDSR data-subject request against AppsFlyer, rehearsing
  the whole flow against the stub endpoints first.
api: openapi/appsflyer-opendsr-api-openapi.yml
operations:
- discovery
- create-opendsr-request
- create-batch-opendsr-requests
- get-opendsr-request-status
- get-batch-status
- download-report
- get-certificate
- cancel-opendsr-request
- stub-discovery
- create-stub-opendsr-request
- get-stub-opendsr-request-status
- download-stub-report
- get-stub-certificate
- cancel-stub-opendsr-request
- create-stub-batch-opendsr-requests
- get-stub-batch-status
generated: '2026-07-31'
method: generated
---

# Handle an OpenDSR privacy request end to end

Submit, poll, download and cancel an IAB OpenDSR data-subject request against AppsFlyer, rehearsing the whole flow against the stub endpoints first.

## Authentication

Every call below uses the AppsFlyer **API V2 token** (a JWT), issued by an account admin from the AppsFlyer
dashboard Security Center and shown exactly once:

```
Authorization: Bearer <API_V2_TOKEN>
```

Do not fall back to the legacy `api_token` query parameter — those V1 endpoints are superseded, and every
API V2 token issued before 2026-03-10 19:00 UTC was revoked. Full profile: `authentication/appsflyer-authentication.yml`.

## Steps

Base: `https://hq1.appsflyer.com/api/gdpr/v1/`. This API implements the IAB **OpenDSR** specification and is
rate-limited to **350 requests per minute** (~504,000/day).

**Rehearse on the stubs.** Eight `stub` operations mirror the eight production operations one for one, so the
whole client can be exercised without touching a real subject's data:

| Production | Stub |
|---|---|
| `discovery` | `stub-discovery` |
| `create-opendsr-request` | `create-stub-opendsr-request` |
| `create-batch-opendsr-requests` | `create-stub-batch-opendsr-requests` |
| `get-opendsr-request-status` | `get-stub-opendsr-request-status` |
| `get-batch-status` | `get-stub-batch-status` |
| `download-report` | `download-stub-report` |
| `get-certificate` | `get-stub-certificate` |
| `cancel-opendsr-request` | `cancel-stub-opendsr-request` |

1. `discovery` → `GET /discovery` for the supported subject-request types and identity forms.
2. `create-opendsr-request` → `POST /opendsr_requests` (or `create-batch-opendsr-requests` →
   `POST /opendsr_requests/batch`). Keep the returned `subject_request_id`; there is no idempotency key, so a
   blind retry creates a **second** request against the same subject.
3. `get-opendsr-request-status` → `GET /opendsr_requests/{subject_request_id}` — poll, do not busy-loop.
   For batches use `get-batch-status` → `GET /opendsr_requests/batch/{batch_id}`.
4. `download-report` → `GET /download/{subject_request_id}` once the status is complete.
5. `get-certificate` → `GET /certificate` for the signed completion certificate you retain as evidence.
6. `cancel-opendsr-request` → `DELETE /opendsr_requests/{subject_request_id}` while it is still cancellable.

## Failure handling

`410` means the request has already completed and its artifacts have aged out. `429` means you crossed the
350/minute ceiling.

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

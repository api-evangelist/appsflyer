---
name: Pull raw and aggregate attribution data
description: Export AppsFlyer raw-data and aggregate reports (installs, in-app events, retargeting, ad revenue,
  Protect360 fraud) and the Master aggregate report, respecting the daily report quota.
api: openapi/appsflyer-raw-data-pull-api-v2-token-openapi.yml
operations:
- master-lastupdate
also:
- openapi/appsflyer-aggregate-pull-api-v2-token-openapi.yml
- openapi/appsflyer-master-api-openapi.yml
- openapi/appsflyer-master-freshness-api-openapi.yml
- openapi/appsflyer-cohort-api-openapi.yml
generated: '2026-07-31'
method: generated
---

# Pull raw and aggregate attribution data

Export AppsFlyer raw-data and aggregate reports (installs, in-app events, retargeting, ad revenue, Protect360 fraud) and the Master aggregate report, respecting the daily report quota.

## Authentication

Every call below uses the AppsFlyer **API V2 token** (a JWT), issued by an account admin from the AppsFlyer
dashboard Security Center and shown exactly once:

```
Authorization: Bearer <API_V2_TOKEN>
```

Do not fall back to the legacy `api_token` query parameter — those V1 endpoints are superseded, and every
API V2 token issued before 2026-03-10 19:00 UTC was revoked. Full profile: `authentication/appsflyer-authentication.yml`.

## Steps

1. **Check data freshness before you pull anything** — `master-lastupdate` →
   `GET https://hq1.appsflyer.com/api/master-agg-data/lastupdate`. Pulling before the last update simply burns
   a quota slot on stale data.
2. **Aggregate report** — `GET https://hq1.appsflyer.com/api/master-agg-data/v4/app/{app_id}` (Master API), or the
   narrower Aggregate Pull API reports at
   `https://hq1.appsflyer.com/api/agg-data/export/app/{app-id}/{daily_report|geo_report|geo_by_date_report|partners_report|partners_by_date_report}/v5`.
3. **Raw data** — `GET https://hq1.appsflyer.com/api/raw-data/export/app/{app-id}/{report}/v5` where `{report}` is one
   of `installs_report`, `in_app_events_report`, `organic_installs_report`, `organic_in_app_events_report`,
   `uninstall_events_report`, `ad_revenue_raw`, `ad_revenue_organic_raw`, `ad-revenue-raw-retarget`,
   `in-app-events-postbacks`, `blocked_installs_report`, `blocked_in_app_events_report`,
   `blocked_install_postbacks`, `blocked_clicks_report`, `fraud-post-inapps`, `detection`, and the retargeting
   variants. These operations carry **no `operationId`** in the published spec — address them by path.
4. **Cohorts** — `POST https://hq1.appsflyer.com/api/cohorts/v1/data/app/{app_id}`.

Every report takes a `from`/`to` date window and a `timezone`; raw-data reports return **CSV** by default and
cap at 1,000,000 rows per request. Split wide windows rather than asking for one enormous file.

## Failure handling

- **402** — the account's daily report quota for that report is exhausted. Do not retry today; see
  https://support.appsflyer.com/hc/en-us/articles/207034366-Report-generation-quotas-rate-limitations
- **429** — throttled. Back off; no `Retry-After` is sent.
- **410** — the report window has aged out of retention.
- **416** — the requested range is not satisfiable; narrow the date window.

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

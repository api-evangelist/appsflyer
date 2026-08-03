# AppsFlyer

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

AppsFlyer is a mobile marketing analytics and attribution platform used by app marketers to measure, attribute and optimize user acquisition across mobile, web, CTV, console and PC.

- Website: https://www.appsflyer.com/
- Developer hub: https://dev.appsflyer.com/hc
- API reference: https://dev.appsflyer.com/hc/reference
- Help Center: https://support.appsflyer.com/hc/en-us
- Status: https://status.appsflyer.com/
- GitHub: https://github.com/AppsFlyerSDK

## What is in this repo

| Artifact | Contents |
|---|---|
| `openapi/` | 39 OpenAPI 3.0.x documents, 141 operations, harvested verbatim from the AppsFlyer developer hub |
| `overlays/` | One OpenAPI Overlay 1.0.0 per spec carrying our cross-links and notes |
| `authentication/` | Auth profile — API V2 bearer JWT plus three legacy/product-specific key schemes |
| `conventions/` | Pagination, versioning, tracing, rate limiting, error envelope, and the recorded absence of idempotency |
| `errors/` | Error catalog derived from every 4xx/5xx response across the estate |
| `lifecycle/` | Versioning, status page, and the observed deprecations |
| `sandbox/` | Sandbox event hosts, the OpenDSR stub endpoints, test devices and debug apps |
| `asyncapi/` | Push API webhook (postback) catalog — no AsyncAPI is published |
| `mcp/` | Hosted AppsFlyer MCP server + the first-party `@appsflyer/sdk-mcp-server`, and the tool crosswalk |
| `skills/` | Nine packaged agent skills, every step grounded in a verified `operationId` |
| `packages/` | First-party SDKs across CocoaPods, Maven Central, npm, pub.dev, NuGet and GitHub |
| `well-known/` | `/.well-known/` probe index, including the RFC 9727 API catalog and the MCP OAuth metadata |
| `security/` | Domain security probe, trust center, and the recorded absence of a public VDP |
| `conformance/`, `data-model/`, `components/`, `changelog/`, `llms/` | Standards posture, entity graph, web components, release channels, llms.txt |

## Notable

- The OpenAPI estate is real and large but was not published as downloadable files — it lives inside the
  developer hub's ReadMe backend, one document per source spec, and was reconstructed by merging the
  per-operation OpenAPI payloads the hub serves.
- 63 of 141 operations carry an `operationId`; the rest are addressable only by method + path.
- No idempotency contract, no RFC 9457 problem details, no `Sunset`/`Deprecation` headers, no
  `security.txt`, and no A2A agent card.

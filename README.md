# AppsFlyer

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

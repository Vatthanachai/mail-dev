# PLAN: .NET 10 Platform

## Delivery Boundary

This package incorporates the superseding [Razor monolith intake](../../intake/razor-monolith.md) into the [original platform intake](../../intake/dotnet10-platform.md), retaining the user's **guidelines and planning first** scope. It does not authorize claiming application work complete. The empty solution shell is not an implemented host. [Tasks](Tasks.md) remain planned; [requirements](../../requirements/dotnet10-platform.md) preserve `REQ/AC-001`–`009` and add `REQ/AC-010`–`011`. Prior API-only `dotnet10-platform` QA/SEC/delivery reports are historical; current `razor-monolith` documentation QA → SEC → DocKeeper is pending.

## Proposed N-Tier Layout

```text
src/MailDev.Web/              One ASP.NET Core host, middleware, OpenAPI/Scalar, DI composition
  Pages/                     .cshtml Razor Pages and PageModels
  Controllers/               Attribute-routed /api/* API controllers and HTTP DTOs
  wwwroot/                   Static CSS, JavaScript and images
src/MailDev.Application/      Use cases and persistence/storage/cache interfaces
src/MailDev.Domain/           Entities, value objects, domain rules
src/MailDev.Infrastructure/   EF Core, provider adapters, external SDK integration
tests/MailDev.UnitTests/      Domain/application/PageModel/controller behavior and contracts
tests/MailDev.IntegrationTests/ Provider, UI forms, API, configuration and host verification
tests/MailDev.BrowserTests/   Planned browser navigation/form/cookie/security checks
```

Domain references no other project. Application references Domain. Infrastructure references Application/Domain. Web references Application and Infrastructure for startup registration; PageModels and Controllers use shared Application services rather than infrastructure SDKs. Keep composition references out of domain rules. Use small interfaces, constructor injection, encapsulated C# objects, and composition; add patterns only for actual provider variation. PageModels do not make loopback HTTP calls to their own API; browser JavaScript may use same-origin `/api/*` where needed.

Razor Pages uses `.cshtml` and PageModels, not Blazor. UI, controller API, static assets and framework documentation endpoints share one ASP.NET Core host/process/image/deployment. N-Tier describes logical project boundaries, not separately deployed services. Reserve `/api/*` for controllers and keep Razor/documentation routes separate.

## Sequence and Integration Approach

1. Populate the empty solution with `net10.0` Web, Application, Domain and Infrastructure projects; register/map Razor Pages and controllers, serve static assets, and establish route separation, test infrastructure, formatting/analyzers, package version policy, and coverage collection. Validate compatible .NET 10 dependencies before pinning them.
2. Implement configuration contracts from requirements and [architecture](../../architecture.md) using typed options and startup validation. Explicitly inject `.env` into Docker; choose/document the native-development mechanism and precedence. Add configuration Red/Green tests before registrations. Reconcile the existing [.env.example](../../../.env.example) with implemented options; its SQLite/Volume/InMemory selections and TTL 300 are illustrative local examples, not implemented defaults.
3. Define domain-facing persistence, streaming file storage, and cache contracts from actual use cases. Keep EF Core contexts, connection strings, SDKs and Redis details within Infrastructure.
4. Implement EF Core PostgreSQL and SQLite adapters with distinct migrations. Establish non-destructive explicit migration procedures and selected-provider contract tests.
5. Implement Volume, S3, and Azure Blob file adapters. Test sandboxed keys, stream handling, cleanup, not-found/error behavior and credential redaction. Resolve content/authorization rules before public upload/download endpoints.
6. Implement In-memory/Redis cache adapters with shared key, serialization, TTL and invalidation contracts. Document instance-local versus shared semantics; resolve outage behavior without a silent fallback.
7. Add thin PageModels and Controllers around shared Application use cases once business page/endpoint requirements exist. Plan model validation, authorization, antiforgery, output encoding, safe redirects, cookie behavior and functional browser/form tests. Generate OpenAPI for controller APIs and add Scalar, gated with an explicit setting and safe environment defaults; Razor UI is not an OpenAPI operation.
8. Package one multi-stage, non-root monolith image with `.dockerignore`, durable SQLite/Volume paths and required local services. Plan Docker/k3s/k8s deployment of the same image with whole-app replicas, probes and graceful shutdown. Define protected shared Data Protection keys/application discriminator for multi-pod cookies/antiforgery. Document PostgreSQL/shared storage recommendations and SQLite/node-local Volume constraints. Verify configuration injection and restart persistence.
9. QA verifies the full acceptance matrix and coverage; SEC reviews final code/configuration/dependencies after QA. Fixes loop Dev → QA → SEC. DocKeeper finalizes only after both pass.
10. PM refreshes/integrates latest `origin/develop`, resolves conflicts, repeats affected QA/SEC checks, records revisions, and opens the PR into `develop`. See [Git Flow](../../git-flow.md).

## Verification Strategy

- TDD for behavior: failing test with expected behavioral reason → minimal passing implementation → refactoring with passing tests. Link cycle evidence to IDs.
- Unit tests for domain/application, PageModel/controller contracts and error paths; real-host integration tests for controller APIs and Razor routing, model binding, validation and form antiforgery; planned browser tests for navigation, encoded content, redirects, cookies, UI/API authorization and cookie-authenticated API CSRF behavior. Use shared contract tests to prevent divergent UI/API business rules.
- Provider-specific integration tests cover both databases, three storage backends and both caches. Test a bounded representative cross-provider matrix rather than claiming every combination passed. Verify the same monolith artifact on Docker/k3s/k8s, including probes, graceful shutdown, persistence and multi-pod cookie/antiforgery continuity using shared protected keys.
- Test selected-only validation, unsupported providers, missing/invalid settings, secret redaction, persistence, cancellation, error translation, TTL expiry, and documented outage policies. Respect SQLite limitations rather than treating PostgreSQL-only behavior as portable.
- Plan xUnit using the VSTest runner with `coverlet.collector`, then merge all project reports through ReportGenerator and enforce **aggregate executable-line coverage of first-party production projects ≥80%**, including Web/PageModels/Controllers, Application, Domain and Infrastructure, following [development standards](../../development-standards.md). Explicitly select compatible VSTest/test SDK packages; do not assume a VSTest collector also works with Microsoft.Testing.Platform. Aggregate across the solution, include unexercised production assemblies, exclude test/generated/vendor code only with documented rationale, and never hide production behavior. Browser tests do not automatically produce server coverage; record collection limitations rather than claiming otherwise. QA re-runs on the final integrated revision. Below-threshold, empty, incomplete, unavailable or stale reports block completion.
- Introduce CI formatting/build/test/coverage gates when application scaffolding is implemented. Until then, this document-only change receives consistency/link/traceability and security-design review; runtime tests and coverage are not applicable.

## Risks and Decisions

No business page/API scope, identity/tenant rules, file retention/limits, production cloud account, or release date is inferred. Resolve material decisions through PM/BA before dependent implementation. Record provider defaults, options bounds, native `.env` loading, cache outage policy, migrations, documentation/UI routes, cookie/CSRF policy, key-store protection, ingress/proxy settings and topology before their implementation tasks. Changing backend does not copy existing data. SQLite/node-local files and InMemory cache do not become shared simply by increasing replicas. Cloud/emulator/cluster limitations must be reported accurately.

## Handoff Package

PM reviews scope/completeness before Dev starts. Dev provides task/requirement IDs, changed paths, TDD/check/coverage evidence and implementation revision. QA reports all required checks; SEC records findings and verified remediation; DocKeeper links both reports and operational instructions. PM keeps `ready-for-pr`, `pr-open` and `merged` distinct.

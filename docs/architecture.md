# Planned .NET 10 Razor Pages + API Monolith

## Status and Scope

This is the agreed development direction, not a running system. The current [monolith intake](intake/razor-monolith.md) revises the API-only plan; this change delivers guidelines and planning only. Implementation follows [requirements](requirements/dotnet10-platform.md), [PLAN](plans/dotnet10-platform/PLAN.md), and [Tasks](plans/dotnet10-platform/Tasks.md). Domain pages/endpoints and authentication rules require feature requirements. Prior API-only QA/SEC reports do not approve this revision.

## N-Tier Responsibilities and Dependencies

Use C# projects targeting `net10.0` as one monolith: a single ASP.NET Core host/process serves Razor Pages and API Controllers, publishes as one application image, and deploys as one workload. Application, Domain and Infrastructure are class libraries in that process. N-Tier describes logical boundaries, not separately deployed services.

| Project | Responsibility | Allowed project dependencies |
| --- | --- | --- |
| `src/MailDev.Web` | Razor Pages/PageModels, Controllers, static assets, HTTP mapping, validation, exception handling, OpenAPI/Scalar, composition root | Application; Infrastructure for registration only |
| `src/MailDev.Application` | Use cases, application DTOs, transaction orchestration, persistence/storage/cache interfaces | Domain |
| `src/MailDev.Domain` | Entities, value objects, invariants and business rules | None of the other tiers |
| `src/MailDev.Infrastructure` | EF Core, provider-specific migrations, storage/cache adapters, provider option validation | Application and Domain |

Place UI files in `MailDev.Web/Pages/` (`.cshtml` and PageModels), APIs in `Controllers/`, and public static assets in `wwwroot/`. Razor Pages means server-rendered pages, not Blazor. Register both `AddRazorPages`/`MapRazorPages` and `AddControllers`/`MapControllers` in the same host. Use `[ApiController]`, `ControllerBase` and explicit `/api/*` routes for APIs; keep UI routes outside that namespace. Framework OpenAPI/Scalar routes must not collide with UI/API routes.

PageModels and Controllers are thin presentation adapters that call the same Application services through DI. PageModels do not issue HTTP requests back to their own API for server-side work; browser JavaScript may call same-origin APIs for interactions. Keep business rules and authorization-relevant use-case rules in shared services to prevent drift; handle HTTP authorization at both presentation boundaries. No PageModel or Controller directly uses DbContext, cloud SDKs or Redis. EF Core/Npgsql and provider types stay outside Domain/Application contracts. Test reference boundaries.

## UI and API Security

Encode untrusted content in Razor; do not use raw HTML output for untrusted data. Keep antiforgery validation on state-changing Razor forms. Define an explicit CSRF strategy for unsafe cookie-authenticated API requests; `[ApiController]` does not itself provide antiforgery protection. A page's authorization does not automatically protect its corresponding API. Restrict redirects to approved local/allowlisted destinations, validate input server-side, and keep uploads outside public `wwwroot`.

Authentication/session policy remains a future feature decision. Where cookies are used, configure secure cookie attributes, consistent authorization, and shared durable Data Protection keys with the same application discriminator across replicas. Encrypt/restrict key storage and do not confuse it with cache/file-provider configuration. Do not enable broad CORS for the same-origin UI without a justified external-client requirement.

## Provider Contracts

| Concern | `.env` selector values | Planned implementation |
| --- | --- | --- |
| Database | `Database__Provider=PostgreSQL` or `SQLite` | EF Core with Npgsql or Microsoft SQLite provider aligned with EF Core 10 |
| Files | `Storage__Provider=Volume`, `S3`, or `AzureBlob` | Application `IFileStorage` contract with local filesystem, AWS SDK, or Azure Blob adapter |
| Cache | `Cache__Provider=InMemory` or `Redis` | Application `ICacheService` contract with IMemoryCache or Redis adapter |

**Azure Storage means Azure Blob Storage** for this plan. This is an explicit interpretation of object/file storage, not Azure Files. Revisit through PM/BA if Azure Files was intended.

Use `Database__ConnectionString` only for the active database. Keep separate PostgreSQL/SQLite migration sets and snapshots; switching providers neither moves nor synchronizes data. SQLite needs a writable persistent directory including journal/WAL side files, with concurrency/replica limitations documented. PostgreSQL needs its own database/user and controlled migration execution; do not auto-migrate concurrently from every monolith replica.

MailDev's planned persistence contract stores each received email in the selected database, including original raw MIME, message fields, envelopes, mailbox associations and attachment metadata/references. Extracted attachment contents use the selected `Storage__Provider` (Volume, S3 or Azure Blob). The user permits raw MIME in the database to include encoded attachment data also stored separately. POP3/API raw retrieval must preserve accepted message bytes after SMTP transport unstuffing; the attachment provider is not the authoritative store for the whole message. See [storage intake](intake/mail-storage.md).

Storage adapters share stream-based upload/read/delete contracts, opaque keys and consistent not-found/conflict behavior; finalize method signatures with use cases. Reject path traversal and unsafe keys, bound upload sizes, use cancellation, keep buckets/containers private, and grant least privilege. Volume writes stay within the configured mounted root; reads/deletes must also protect against symlink escape. S3 supports a configured region/bucket and optional compatible endpoint; validate endpoint/TLS and credential configuration. Azure Blob uses a configured service URI/container and standard credential chain/managed identity. Do not bake cloud credentials into images.

Before SMTP acceptance, persist required attachment objects and atomically commit the database email/raw MIME and references. The database and storage provider do not share a transaction: plan staging, recovery, safe orphan cleanup and failure injection. Coordinate backup, retention and deletion across both stores; measure database growth and limits caused by raw MIME duplication. These are planned behaviors, not implemented guarantees.

Cache is an optimization, never the source of truth. Define bounded TTL, namespaced keys, serialization, invalidation after successful mutation, and explicit outage behavior before coding. InMemory is process-local and lost at restart; Redis supports shared cache across replicas. Changing providers does not transfer cache entries. Never silently switch providers on configuration or connection failure.

## `.env` Configuration

Use the planned [.env.example](../.env.example) as the naming contract. Active provider options are validated at startup; reject unknown selector values and missing required active settings with redacted errors. Do not require credentials for inactive providers. Environment variables override JSON/defaults and use `__` to represent nested options.

.NET does **not** read `.env` automatically. For Docker explicitly pass `--env-file .env` or Compose `env_file: .env`; Compose's automatic `.env` interpolation alone does not inject all variables into the container. Native development will need a documented dotenv loader or wrapper that reads the project-root file before host creation while preserving already-set process environment variables. That mechanism remains a planned task, not an installed feature.

Keep the shared example in a simple literal `KEY=value` format: Docker `run --env-file` preserves surrounding quotes in values. Do not wrap the example connection string in quotes or rely on shell expansion/interpolation. Native and Compose parsing must be tested against the same intended literal values; document differences before accepting richer syntax.

Keep `.env` out of Git/build context. Permit only placeholder examples. Mount separate durable file/database directories for Volume/SQLite. Example future container paths are `/data/files` and `/data/db/maildev.db`; native runs must use appropriate writable paths. Exact port mappings, volumes, and startup commands will be verified when the Docker configuration exists.

## Scalar and OpenAPI

Use `Microsoft.AspNetCore.OpenApi` with `AddOpenApi`/`MapOpenApi` and `Scalar.AspNetCore` with `MapScalarApiReference`. Describe DTOs, response codes, validation/problem details, and security schemes once implemented. Gate **both** the OpenAPI JSON and Scalar UI with `ApiDocumentation__Enabled`; default off in Production. Production opt-in needs a defined access policy reviewed by SEC. Never embed live credentials in examples or Scalar configuration.

Scalar documents controller APIs, not Razor HTML pages. UI behavior and navigation belong in user documentation. Disabling Scalar/OpenAPI must not disable the normal Razor UI or controller APIs.

## Docker, k3s and k8s

Publish the Web host plus its class libraries and UI assets as one non-root Docker application. Use the same artifact for Docker and k3s/k8s; a replica runs the whole monolith. Route UI and `/api/*` to the same service/origin. Plan readiness/liveness checks, graceful shutdown and proxy/TLS handling with trusted forwarded headers; exact manifests and ports are implementation work.

For k3s/k8s, map the same configuration keys to environment variables using ConfigMaps for non-secret values and Secrets or an approved external secret provider for credentials. The project's local `.env` naming contract does not mean Kubernetes reads that file automatically. Do not commit populated Secrets manifests or bake `.env` into the image.

For multiple replicas, keep authoritative data in shared services and configure shared Data Protection keys when cookie/antiforgery features require them. InMemory is process-local, SQLite and local Volume have documented replica/access constraints, and a shared persistent volume does not automatically make SQLite safe for concurrent multi-pod writers. Select PostgreSQL, suitable shared object storage and Redis where shared semantics are required; Redis does not replace Data Protection configuration. Persist SQLite/Volume paths explicitly for supported single-instance deployments.

## Validation and Delivery

Use TDD, SOLID/OOP and the ≥80% aggregate production line-coverage gate in [standards](development-standards.md), including Web host/PageModel/Controller code. Test page rendering/forms/validation, shared-service behavior, route separation, API status/JSON and security boundaries alongside provider contracts. Use browser-level tests for critical UI journeys; line coverage alone does not verify Razor markup. QA must pass before SEC, then DocKeeper. Git Flow conflict integration and affected QA/SEC rechecks precede every PR into `develop`.

## Technical References

- [Controller-based ASP.NET Core APIs](https://learn.microsoft.com/en-us/aspnet/core/web-api/?view=aspnetcore-10.0)
- [Razor Pages registration and routing](https://learn.microsoft.com/en-us/aspnet/core/razor-pages/?view=aspnetcore-10.0)
- [Environment configuration providers](https://learn.microsoft.com/en-us/dotnet/core/extensions/configuration-providers)
- [EF Core database providers](https://learn.microsoft.com/en-us/ef/core/providers/)
- [SQLite limitations](https://learn.microsoft.com/en-us/ef/core/providers/sqlite/limitations)
- [Scalar ASP.NET Core integration](https://scalar.com/products/api-references/integrations/aspnetcore/integration)

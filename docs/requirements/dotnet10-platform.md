# Requirements: .NET 10 Platform

## Scope and Status

Sources: [original platform intake](../intake/dotnet10-platform.md) and the superseding [Razor monolith intake](../intake/razor-monolith.md). This delivery defines guidelines and a development plan only. Every application capability below is **planned**, not implemented or validated. Use one C#/.NET 10 ASP.NET Core host for Razor Pages and API Controllers with N-Tier libraries. Retain Docker, TDD, SOLID/OOP, minimum 80% aggregate production line coverage, and Git Flow. The existing empty solution shell does not implement this architecture.

Current documentation acceptance: the monolithic Razor Pages/API stack and provider/configuration contracts are consistent across guidance, agent roles and this BA package; every planned requirement maps to tasks; planned capabilities are clearly distinguished from verified implementation; QA then SEC review this revision before DocKeeper finalizes. `AC-001`–`AC-011` below apply to the future platform, not this document-only delivery. Prior `dotnet10-platform` QA/SEC/delivery reports concern the former API-only plan and are historical; they do not approve this revision. Current `razor-monolith` QA → SEC → DocKeeper review is pending.

Actors are browser users, API consumers, operators selecting backends, and contributors extending the platform. Business pages/endpoints and authentication/authorization policy require separate feature requirements; do not infer mail features from the repository name. Razor Pages means `.cshtml` with PageModels, not Blazor or a separate frontend deployable.

## Functional Requirements and Acceptance Criteria

| ID | Requirement | Given / When / Then acceptance criterion |
| --- | --- | --- |
| REQ-001 / AC-001 | One `net10.0` Razor Pages + controller API host | Given the completed `MailDev.Web` host, when UI or API requests arrive, then the same ASP.NET Core process serves `.cshtml`/PageModels, static assets and attribute-routed `/api/*` controllers using Razor Pages and controller registration/mapping. Razor page routes do not collide with `/api/*` or documentation routes; business APIs use controllers. |
| REQ-002 / AC-002 | N-Tier separation | Given Web, Application, Domain, and Infrastructure projects, when references are inspected, then Domain has no infrastructure/framework dependency; Application depends on Domain; Infrastructure implements Application contracts and depends on Application/Domain; Web depends on Application and references Infrastructure only for DI composition. PageModels and Controllers use shared Application services, never direct persistence/storage/cache SDKs or server loopback HTTP calls to their own API. |
| REQ-003 / AC-003 | Select PostgreSQL or SQLite using `.env` | Given either supported database selector and valid matching settings, when starting the application, then only that provider is registered and data operations persist through it. Invalid/missing required settings fail startup with redacted diagnostics. Provider-specific integration tests prove equivalent agreed behavior. |
| REQ-004 / AC-004 | Select Volume, S3, or Azure Blob file storage | Given one supported storage selector and configuration, when performing agreed stream write/read/delete operations, then the selected adapter provides the contract without exposing SDK types to business logic. Invalid keys cannot escape the Volume root; credentials and signed URLs are not logged. Missing required settings fail startup. |
| REQ-005 / AC-005 | Select In-memory or Redis caching | Given a supported cache selector, when an entry is set with TTL, read, expires, or is invalidated, then both implementations honor the agreed serialization/key/expiry contract. In-memory remains instance-local; Redis supports shared access. No switch silently substitutes a backend. |
| REQ-006 / AC-006 | OpenAPI and Scalar documentation | Given documentation is enabled for an allowed environment, when a consumer opens the documented Scalar endpoint, then Scalar renders the generated OpenAPI document describing controller routes, request/response models and error responses. Disabling documentation prevents both Scalar and OpenAPI exposure. No real credential is embedded. |
| REQ-007 / AC-007 | Explicit environment configuration | Given the project `.env`, when Docker starts using `--env-file` or explicit Compose `env_file`, then provider settings become environment variables. For native development, an explicit loading mechanism and precedence are documented/tested; `.env` is never claimed to load automatically. Only selected-provider settings are required. |
| REQ-008 / AC-008 | Durable, least-privilege monolith Docker runtime | Given one application image containing Razor Pages, API and logical tiers, when its non-root container is started or recreated, then both presentations remain in the same process and SQLite/Volume data persist through appropriate mounts. The image contains neither `.env` nor credentials; build and runtime stages are separate. |
| REQ-009 / AC-009 | Development and delivery quality | Given an implemented behavior, when Dev delivers it, then actual Red/Green/Refactor evidence exists and SOLID/OOP boundaries are reviewed; QA independently verifies aggregate production executable-line coverage ≥80%, then SEC passes before DocKeeper finalizes. Conflict checks against latest `origin/develop` precede every PR into `develop`. |
| REQ-010 / AC-010 | Shared UI/API behavior and browser security | Given an agreed use case and identity policy, when it is executed through a Razor Page or controller, then shared Application rules and authorization produce consistent outcomes. Razor forms reject missing/invalid antiforgery tokens; cookie-authenticated unsafe API requests have explicit CSRF protection. Planned functional form/browser tests verify binding, validation, encoding, redirects and cookie behavior without weakening controls for JSON endpoints. |
| REQ-011 / AC-011 | Same monolith artifact on Docker, k3s and Kubernetes | Given the same application image deployed to Docker, k3s or k8s, when it starts, scales or shuts down, then each replica runs the whole application, probes distinguish readiness/liveness, and graceful shutdown respects in-flight work. Multi-pod cookie/antiforgery tests use shared, persistent, access-controlled Data Protection keys and a consistent application discriminator. Storage/database choices respect replica constraints; no separate tier deployments are required. |

## Configuration Contract

These names and values define the planned configuration interface. The existing [.env.example](../../.env.example) contains illustrative local settings and placeholders, not a working configuration loader or implemented defaults. Its `SQLite`/`Volume`/`InMemory` selections and TTL `300` are examples; implementation defaults and permitted bounds remain decisions to record before coding. Validate selectors at startup and do not silently fall back. See [architecture](../architecture.md) for the shared configuration/layer contract.

| Key | Supported values / purpose |
| --- | --- |
| `Database__Provider` | `PostgreSQL` or `SQLite` |
| `Database__ConnectionString` | Connection for the selected database; proposed SQLite path `/data/db/maildev.db` must be durably mounted |
| `Storage__Provider` | `Volume`, `S3`, or `AzureBlob` |
| `Storage__Volume__RootPath` | Allowed mounted storage root when Volume is selected; proposed `/data/files` |
| `Storage__S3__Bucket`, `Storage__S3__Region` | Required S3 bucket and region; use the SDK credential chain for credentials |
| `Storage__S3__ServiceUrl` | Optional endpoint for explicitly supported compatible/local test services; validate URL and TLS policy |
| `Storage__AzureBlob__Container`, `Storage__AzureBlob__ServiceUri` | Azure Blob container/service endpoint; prefer workload identity/managed credentials; explicit local credentials stay out of Git |
| `Cache__Provider` | `InMemory` or `Redis` |
| `Cache__Redis__ConnectionString` | Required only for Redis; redact credentials |
| `Cache__DefaultTtlSeconds` | Positive default TTL; task implementation records the selected default and permitted bounds |
| `ApiDocumentation__Enabled` | Explicit Boolean; disabled by default outside development; a local example may explicitly enable it with the Development environment |

`.env` configures a selection, not data copying: changing a provider does not migrate or synchronize existing records/files/cache. Maintain distinct EF Core migrations for PostgreSQL and SQLite; migration execution must be explicit and documented rather than automatic destructive startup work. SQLite-specific concurrency/type constraints must be captured in integration tests.

## Assumptions and Decisions Before Implementation

- Azure Storage means **Azure Blob Storage** for file/object storage; Azure Files is not in scope.
- File adapters expose streaming operations; business upload limits, allowed content, retention, and download authorization await endpoint requirements. Resolve these before implementing exposed file endpoints.
- Default provider selections, cache TTL/bounds, outage/error behavior, native `.env` loading mechanism, database migration commands, and documentation endpoint paths are design decisions for the corresponding tasks. Record choices before coding their behavior.
- Authentication, authorization, tenancy, UI/API cookie and CSRF policy, Redis outage policy, cache invalidation ownership, and cloud credential deployment need feature/operations decisions. Do not silently add anonymous production pages/endpoints or cache sensitive responses. Preserve Razor form antiforgery and output encoding; a controller attribute alone does not provide API CSRF protection.
- Keep API routes under `/api/*` and define noncolliding Razor/documentation route spaces. PageModels invoke Application services directly; same-origin browser JavaScript may call `/api/*` when useful.
- Finalize Data Protection key storage/encryption, shared application discriminator, secret injection, probes, shutdown deadlines, ingress/cookie policy and replica counts before deployment implementation. SQLite and node-local Volume are single-instance/local options unless an explicitly tested topology supports otherwise; prefer PostgreSQL plus appropriate shared storage for multi-pod deployment. Redis is not a substitute for shared Data Protection keys unless a deliberate protected key-store integration is implemented and verified.
- Provider integration tests may require PostgreSQL/Redis services and S3/Azure emulators or controlled test accounts. Missing access is a recorded blocker, not a passing result.

## Traceability

See [PLAN](../plans/dotnet10-platform/PLAN.md) and [Tasks](../plans/dotnet10-platform/Tasks.md) for implementation sequence, owners, validation and stage gates.

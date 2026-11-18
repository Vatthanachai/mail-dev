# PM Intake: .NET 10 Platform

## Request and Confirmed Scope

The user replaces Rust with .NET 10, controller-based ASP.NET Core Web API, and N-Tier architecture. PostgreSQL/SQLite must be selectable through the project's `.env`. File storage must support Volume/S3/Azure Storage; caching must support In-memory/Redis. API documentation uses Scalar.

The user explicitly selected **guidelines and development planning first**, rather than scaffolding or implementing the application in this task. Update agent instructions and contributor/development documentation, and create a traceable BA development plan. No runnable platform, provider implementation, migration, cloud resources, or CI enforcement is delivered in this change.

## PM Assessment

- Priority: establish the technical baseline before feature development; no delivery date is committed.
- Direction: .NET 10/C#, Controllers, N-Tier, Docker; Azure Storage is interpreted as Azure Blob Storage for file/object storage. Record this interpretation explicitly.
- Retain TDD, SOLID/OOP, at least 80% aggregate application/library line coverage, Git Flow, conflict checks before PR into `develop`, and PM → BA → Dev → QA → SEC → DocKeeper.
- Proposed layer split: API, Application, Domain, Infrastructure. Provider adapters stay outside business logic; API is the composition root.
- Use `.env` selectors for database, storage, and cache. Plan startup validation and document that .NET does not automatically load `.env`; Docker must inject it explicitly. Native local loading needs an explicit development mechanism.
- Provider changes select a backend; they do not imply data migration or synchronization. Database migration histories must be provider-specific.
- Risks: provider-specific query semantics, SQLite persistence/concurrency, accidental secret disclosure, missing durable volumes, cloud permissions, cache invalidation, and cross-instance cache behavior.
- Dependencies: .NET 10-compatible provider packages and coverage tooling; later implementation needs test databases/emulators or controlled service access. Live credentials are not needed for this planning task.
- Branch: `feature/git-flow-guidelines` (existing uncommitted, related workflow changes preserved). PR target: `develop`. Remote conflict verification is a separate pre-PR gate, not yet executed.

## Completion Criteria and Handoff

BA creates requirements, PLAN and Tasks with stable IDs, provider/configuration contracts, acceptance criteria and implementation sequencing. PM coordinates updating existing guidance. QA checks documentation consistency and traceability; SEC reviews planned security/configuration controls; DocKeeper finalizes the documentation after both pass.

Application tests and coverage are not applicable to this document-only change; documentation review must never be reported as implementation validation. Implementation tasks remain planned. Material product behavior, authentication policy, and domain endpoints need feature requirements before coding.

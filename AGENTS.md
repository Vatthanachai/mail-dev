# Repository Guidelines

## Agent Roles

The primary agent is PM. Follow [workflow](docs/agent-workflow.md): PM intake → BA requirements/PLAN/Tasks → Dev → QA → SEC → DocKeeper. Resolve findings through Dev/QA/SEC. Definitions live in `.codex/agents/`.

## Project Structure & Module Organization

The planned stack is a C#/.NET 10 monolith: Razor Pages UI and API Controllers in one host/image, with N-Tier libraries. An empty solution shell exists; implementation is planned. See [architecture](docs/architecture.md).

Use `src/MailDev.Web/` for `Pages/`, `Controllers/`, and `wwwroot/`; keep Application, Domain, and Infrastructure as class libraries under `src/`. PageModels and Controllers share Application services through DI, without server-side loopback HTTP. Keep provider SDKs outside business logic and tests in `tests/`.

## Build, Test, and Development Commands

After the solution, locked dependencies, and tooling are introduced, run from the root:

- `dotnet restore --locked-mode` - restore committed dependency versions.
- `dotnet build -c Release --no-restore -warnaserror` - build with warnings treated as errors.
- `dotnet test -c Release --no-build` - run configured tests.
- `dotnet format --verify-no-changes` - check formatting.
- `docker build -t maildev .` - build the image once a Dockerfile exists.
- `docker run --rm --env-file .env maildev` - inject configuration; add documented ports and volumes.

Commands are planned, not runtime verification. See [standards](docs/development-standards.md) for coverage prerequisites and enforcement.

## Coding Style & Naming Conventions

Use four spaces, nullable reference types, `PascalCase` types/public members, `camelCase` locals/parameters, and `Async` method suffixes. Use `.editorconfig`, dotnet format, analyzers, DI and SOLID/OOP. Keep PageModels/Controllers thin; propagate cancellation through I/O.

## Testing Guidelines

Use TDD and xUnit unit/contract/integration tests, plus critical browser journeys. Test Razor forms/routes, APIs and every provider. Require aggregate production line coverage ≥80% using Coverlet and merged ReportGenerator reports, including Web/PageModels. QA then SEC precede DocKeeper/PR. Documentation-only runtime checks are not applicable.

## Commit & Pull Request Guidelines

Use Git Flow: `feature/<slug>` from `develop`; reserve `main` for releases. Before every PR into `develop`, fetch/integrate latest `origin/develop`, resolve conflicts, and repeat affected QA/SEC checks. Include requirement/task links, validation and conflict-check SHAs; see [Git Flow](docs/git-flow.md).

## Security & Configuration

Select PostgreSQL/SQLite, Volume/S3/Azure Blob Storage, and InMemory/Redis through validated `.env` options. Explicitly inject/load `.env`; never commit secrets or silently fall back between providers. Commit package lockfiles; exclude `bin/`, `obj/`, results, and secrets from images. Use non-root Docker, durable volumes, and Scalar/OpenAPI exposure controls.

Preserve Razor antiforgery and encoding; define CSRF protection for cookie-authenticated APIs. Plan shared Data Protection keys for replicas on k3s/k8s.

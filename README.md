# MailDev

## Current Status

This repository currently defines the development guidelines and platform plan. An empty solution shell exists, but application projects, provider implementations, Docker packaging, tests, and CI are not implemented. The examples below describe the planned contract, not a runnable service.

## Planned Platform

| Area | Direction |
| --- | --- |
| API | C#/.NET 10, ASP.NET Core Web API with Controllers |
| Architecture | N-Tier: API, Application, Domain, Infrastructure |
| Database | PostgreSQL or SQLite |
| File storage | Docker Volume/local filesystem, S3, or Azure Blob Storage |
| Cache | InMemory or Redis |
| API documentation | OpenAPI with Scalar |
| Deployment | Docker with explicit configuration injection and persistent mounts |

Provider selection uses the project's `.env`; see the planned [.env.example](.env.example) and [architecture/configuration guide](docs/architecture.md). .NET does not automatically load `.env`. Native loading and Docker packaging remain implementation tasks. Example selections and TTL values are illustrative, not installed runtime defaults.

## Development and Delivery

Use TDD (Red → Green → Refactor), SOLID/OOP, and at least 80% aggregate executable line coverage across production projects. Coverage collection and enforcement are planned; no coverage result has been measured. Follow PM → BA → Dev → QA → SEC → DocKeeper, with fixes returning through the relevant verification stages.

Use Git Flow and open development PRs into `develop` only after integrating the latest target, resolving conflicts, and repeating affected QA/SEC checks. No PR or remote conflict verification is claimed by this planning delivery.

## Documentation Index

- [Repository guidelines](AGENTS.md), [agent workflow](docs/agent-workflow.md), [development standards](docs/development-standards.md), and [Git Flow](docs/git-flow.md)
- [Architecture](docs/architecture.md) and [configuration example](.env.example)
- [PM intake](docs/intake/dotnet10-platform.md), [requirements](docs/requirements/dotnet10-platform.md), [PLAN](docs/plans/dotnet10-platform/PLAN.md), and [Tasks](docs/plans/dotnet10-platform/Tasks.md)
- [Dev handoff](docs/implementation/dotnet10-platform.md), [QA review](docs/qa/dotnet10-platform.md), and [SEC review](docs/security/dotnet10-platform.md)
- [Delivery notes](docs/delivery/dotnet10-platform.md) and [project status](docs/project-status.md)

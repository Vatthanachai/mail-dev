# MailDev

## Current Status

This repository currently defines the development guidelines and platform plan. An empty solution shell exists, but application projects, provider implementations, Docker packaging, tests, and CI are not implemented. The examples below describe the planned contract, not a runnable service.

## Planned Platform

| Area | Direction |
| --- | --- |
| Web | C#/.NET 10 Razor Pages and API Controllers in one `MailDev.Web` host |
| Architecture | N-Tier monolith: Web, Application, Domain, Infrastructure |
| Database | PostgreSQL or SQLite |
| File storage | Docker Volume/local filesystem, S3, or Azure Blob Storage |
| Cache | InMemory or Redis |
| API documentation | OpenAPI with Scalar for controller APIs |
| Deployment | One application image for Docker, k3s, and Kubernetes |

The Web host will contain Razor `Pages/` and PageModels, `Controllers/` for `/api/*`, and `wwwroot/` for public assets. PageModels and Controllers will use shared Application services through dependency injection. Application, Domain, and Infrastructure are logical tiers in the same deployed application. See [architecture](docs/architecture.md) for routing, security and replica requirements.

Provider selection uses the project's `.env`; see the planned [.env.example](.env.example) and [architecture/configuration guide](docs/architecture.md). .NET does not automatically load `.env`; Docker must inject it explicitly and k3s/k8s require environment configuration through ConfigMaps and Secrets or an approved secret provider. Native loading and deployment packaging remain implementation tasks. Example selections and TTL values are illustrative, not installed runtime defaults.

Multi-replica deployments need an appropriate shared database and file storage, and protected shared Data Protection keys when cookies or antiforgery are used. SQLite, node-local Volume storage, and InMemory cache have replica limitations; see the architecture guide before selecting them.

## Development and Delivery

Use TDD (Red → Green → Refactor), SOLID/OOP, and at least 80% aggregate executable line coverage across production projects, including the Web host and PageModels. Critical browser journeys require separate functional tests. Coverage collection and enforcement are planned; no coverage result has been measured. Follow PM → BA → Dev → QA → SEC → DocKeeper, with fixes returning through the relevant verification stages.

Use Git Flow and open development PRs into `develop` only after integrating the latest target, resolving conflicts, and repeating affected QA/SEC checks. No PR or remote conflict verification is claimed by this planning delivery.

## Documentation Index

- [Repository guidelines](AGENTS.md), [agent workflow](docs/agent-workflow.md), [development standards](docs/development-standards.md), and [Git Flow](docs/git-flow.md)
- [Architecture](docs/architecture.md) and [configuration example](.env.example)
- [Current PM intake](docs/intake/razor-monolith.md), [requirements](docs/requirements/dotnet10-platform.md), [PLAN](docs/plans/dotnet10-platform/PLAN.md), and [Tasks](docs/plans/dotnet10-platform/Tasks.md)
- [Dev handoff](docs/implementation/razor-monolith.md), [QA revision review](docs/qa/razor-monolith-revision.md), and [SEC review](docs/security/razor-monolith.md)
- [Current delivery notes](docs/delivery/razor-monolith.md) and [project status](docs/project-status.md)

The earlier [API-only delivery](docs/delivery/dotnet10-platform.md) and its QA/SEC reviews are historical and do not approve the current monolith revision. Current QA and SEC passes cover documentation and security design only, not a running implementation.

# Delivery: Razor Pages + API Monolith Plan

## Scope and Status

This documentation delivery updates the planned platform to C#/.NET 10 with Razor Pages and API Controllers in one `MailDev.Web` host. Application, Domain, and Infrastructure remain N-Tier class libraries in the same process and application image. PageModels and Controllers use shared Application services through DI; `/api/*` is reserved for controller APIs.

The same monolith artifact is planned for Docker, k3s, and Kubernetes. PostgreSQL/SQLite, Volume/S3/Azure Blob Storage, and InMemory/Redis will be selectable using the `.env` configuration contract. Scalar will document controller APIs; UI navigation belongs in user documentation. See [architecture](../architecture.md) and the planned [.env.example](../../.env.example).

The empty solution shell does not implement the application. No application projects, provider adapters, runtime Docker/cluster deployment, browser tests, CI gate, or coverage measurement are delivered by this documentation change. All 15 [implementation tasks](../plans/dotnet10-platform/Tasks.md) remain planned.

## Requirement and Task Traceability

The [requirements](../requirements/dotnet10-platform.md) and [PLAN](../plans/dotnet10-platform/PLAN.md) define future acceptance criteria. The table maps them to planned work and the current documentation review; it does not claim runtime acceptance.

| Requirement | Planned tasks | Review focus |
| --- | --- | --- |
| REQ-001 / AC-001 | TASK-001, 002, 010, 014 | Single Web host, Razor Pages and controller routes |
| REQ-002 / AC-002 | TASK-001, 002, 010, 014 | N-Tier dependencies and shared Application services |
| REQ-003 / AC-003 | TASK-001, 003, 004, 010, 011 | Database selection and provider-specific migrations |
| REQ-004 / AC-004 | TASK-001, 003, 005, 010, 011 | File storage contracts and isolation |
| REQ-005 / AC-005 | TASK-001, 003, 006, 010, 011 | Cache selection and expiry/invalidation |
| REQ-006 / AC-006 | TASK-001, 002, 003, 007, 010, 011 | Scalar/OpenAPI and exposure controls |
| REQ-007 / AC-007 | TASK-001, 003, 008, 010, 011 | Explicit configuration injection and validation |
| REQ-008 / AC-008 | TASK-001, 008, 010, 011, 015 | Non-root image and durable storage |
| REQ-009 / AC-009 | TASK-001, 002, 009–013 | TDD, coverage, review and Git Flow gates |
| REQ-010 / AC-010 | TASK-001, 010, 011, 014 | Shared UI/API behavior and browser security |
| REQ-011 / AC-011 | TASK-001, 010, 011, 015 | Docker/k3s/k8s replicas and protected keys |

## Evidence and Remaining Work

- [PM intake](../intake/razor-monolith.md) records the updated scope. [Dev handoff](../implementation/razor-monolith.md) records the technical documentation review.
- [QA revision check](../qa/razor-monolith-revision.md) passed for the documented architecture and plan at commit `12deed0d8a8a614da75518eb4303304d2b303c28`. It does not approve runtime behavior or this later DocKeeper edit.
- [SEC review](../security/razor-monolith.md) passed for security design and documentation after QA. It is not runtime security approval or a dependency audit.
- TDD, SOLID/OOP and aggregate production line coverage of at least 80% remain future implementation gates, including Web/PageModels. Browser behavior needs separate functional verification.
- Authentication policy, individual UI pages and endpoints, key-store deployment, provider operational settings, and other task decisions remain to be specified. Planned replica requirements include protected shared Data Protection keys and suitable shared persistence.

Prior API-only QA/SEC and [delivery evidence](dotnet10-platform.md) are historical, not approvals for this monolith revision. A PR has not been verified by DocKeeper; before opening one into `develop`, PM must complete the latest remote integration/conflict gate and affected checks according to [Git Flow](../git-flow.md).

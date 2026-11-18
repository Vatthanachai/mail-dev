# Implementation Handoff: .NET 10 Platform Guidance

## Scope and Revision

This is the Dev technical review of a **documentation/configuration-example change**, not application implementation. The user selected guidelines and development planning first. Future `dotnet10-platform/TASK-001`–`TASK-013` remain planned; this report does not mark them complete.

Branch: `feature/git-flow-guidelines`. At review, HEAD and the locally cached `origin/develop` both resolved to `35e47b6160290ac54b3a35806bec750a4f956aaa`. Reviewed content includes uncommitted working-tree changes; these SHAs do not identify a committed documentation delivery. No remote fetch, merge, push, or PR creation was performed by Dev.

An empty `MailDev.slnx` and directory shells appeared concurrently and were preserved. No implemented C# projects, provider adapters, Dockerfile, test configuration, or coverage enforcement were found. An empty solution shell is not evidence of implemented capability.

## Changed Guidance and Design Review

PM updated `AGENTS.md`, the five specialist definitions, workflow, development standards, Git Flow, and the architecture guide. BA supplied requirements, PLAN and Tasks. The planned `.env.example` exposes the agreed configuration keys with placeholders. Dev reviewed this package against `REQ/AC-001`–`009`:

- API/Application/Domain/Infrastructure references keep provider SDKs outside business rules; controllers consume application services and DI composition is isolated in API startup.
- PostgreSQL and SQLite require separate migration sets/snapshots and independent contract tests; provider selection does not migrate data. SQLite paths must persist journal/WAL files as well as the database.
- Docker must explicitly inject `.env`; native loading and precedence remain implementation tasks. Validate only active-provider options and reject unsupported selectors without silent fallback.
- Volume/S3/Azure Blob storage and InMemory/Redis cache have adapter boundaries and explicit durability, TTL, isolation and failure semantics to finalize before coding.
- Scalar and OpenAPI share an exposure gate. Production opt-in requires a defined access policy; neither endpoint is implemented yet.
- Planned xUnit/VSTest plus Coverlet collection and merged ReportGenerator reports preserve the ≥80% aggregate production line-coverage requirement. Pin compatible tool versions, validate all production assemblies appear, and test threshold failure during implementation.

## Checks Actually Executed

| Check | Outcome |
| --- | --- |
| `git status --short --branch` | Confirmed feature branch and uncommitted changes; unrelated concurrent solution shell preserved |
| `git diff --check` | Passed for tracked working-tree changes; does not validate untracked-file formatting |
| `git rev-parse HEAD origin/develop` | Recorded local source/target SHAs above; freshness not verified remotely |
| Relative Markdown-link check over AGENTS, architecture, standards, workflow and three BA files | Passed: non-HTTP concrete relative targets existed |
| `.env.example` active-key extraction and uniqueness check | Passed: eight active keys; selectors and names aligned with the BA contract |
| `rg --files` for C# projects/source, Dockerfile, runsettings and local tool manifest | No matching implementation or test/tool configuration found |
| `Get-Content MailDev.slnx` | Empty solution shell confirmed |

No TDD cycle, build, application tests, provider integration, Docker execution, security scan, or coverage measurement was performed. They are not applicable to this documentation-only delivery and must not be reported as passing runtime checks.

## Reconciliation Notes and Open Decisions

Dev notified PM of two documentation corrections before QA: acknowledge the empty solution shell instead of asserting no solution file exists; use an unquoted connection-string example compatible with the planned Docker `--env-file` path. PM applied both corrections, and Dev confirmed the sample uses `Database__ConnectionString=Data Source=/data/db/maildev.db` and guidance describes literal key/value parsing. Parser parity must still be verified when native loading is implemented. QA should independently confirm the final document wording.

Future decisions remain native `.env` loading/parser behavior, selected provider defaults and TTL bounds, cache outage/invalidation policy, public file limits/authorization/retention, database migration execution, documentation endpoint/access policy, and compatible pinned package/tool versions. Azure Storage is explicitly interpreted as Azure Blob Storage; changing that scope requires PM/BA reconciliation.

## QA Handoff

QA should validate documentation consistency, configuration names/examples, tier boundaries, traceability, planned-versus-implemented claims, and the corrections above. Passing this review authorizes the next **security-design/documentation review**, not runtime acceptance. SEC follows QA; DocKeeper follows both. Remote integration/conflict checks remain mandatory before any PR into `develop`.

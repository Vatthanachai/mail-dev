# Dev Handoff: Razor Pages + API Monolith Plan

## Scope and Review Baseline

This review covers the [monolith intake](../intake/razor-monolith.md), active contributor/agent guidance, [architecture](../architecture.md), [development standards](../development-standards.md), and revised [requirements](../requirements/dotnet10-platform.md), [PLAN](../plans/dotnet10-platform/PLAN.md), and [Tasks](../plans/dotnet10-platform/Tasks.md). The scope is documentation and planning only. All fifteen future implementation tasks remain planned; no application capability is accepted here.

Branch: `feature/git-flow-guidelines`. At review, HEAD and locally cached `origin/develop` both resolved to `35e47b6160290ac54b3a35806bec750a4f956aaa`. The reviewed documents are uncommitted working-tree content; this SHA does not identify a committed monolith delivery. Dev did not fetch, merge, push, create a PR, install dependencies, or scaffold code. The existing empty solution shell and other concurrent artifacts were preserved. Prior API-only implementation/QA/SEC/delivery reports are historical and do not approve this revision.

## Technical Review

- **REQ/AC-001–002:** `MailDev.Web` is the sole host/process/image. `Pages/` with PageModels, `Controllers/`, and `wwwroot/` share that host. Both Razor Pages and Controllers are registered/mapped. `/api/*` is reserved for controller APIs; Razor and Scalar/OpenAPI routes remain separate. PageModels and Controllers call shared Application services through DI without server-side loopback HTTP or direct provider SDK access. Application/Domain/Infrastructure remain logical class-library tiers with no dependency cycles in the planned reference rules.
- **REQ/AC-003–008:** Existing PostgreSQL/SQLite, Volume/S3/Azure Blob and InMemory/Redis contracts remain consistent. Explicit `.env` injection/loading, selected-provider validation, no silent fallback, per-provider migrations, durable SQLite/Volume directories, and controller-only Scalar documentation are preserved. Documentation gating must not disable the UI or controller APIs.
- **REQ/AC-009–010:** TDD/SOLID/OOP, UI/form/API/contract tests and browser journeys are planned. The ≥80% aggregate production line-coverage gate includes Web host, PageModels and Controllers. Browser tests do not automatically collect server coverage; functional markup/security verification remains separate. Razor encoding/antiforgery, safe redirects, authorization at both presentation boundaries, and explicit cookie-authenticated API CSRF protection are specified.
- **REQ/AC-011:** Docker/k3s/k8s use the same whole-app image. Shared protected Data Protection keys and a consistent application discriminator are required for relevant cookie/antiforgery features across replicas. PostgreSQL/shared storage/Redis do not automatically configure these keys. SQLite/local Volume/InMemory replica limitations, probes, shutdown and proxy trust are explicit implementation considerations.

No blocking technical inconsistency was found in the reviewed planning package. Exact domain pages/endpoints, identity policy, native dotenv mechanism, cache outage/default/bounds rules, deployment topology, key-store/encryption configuration and compatible package/tool versions remain future decisions before dependent implementation.

## Checks Actually Executed

| Check | Result |
| --- | --- |
| Read the updated Dev definition, intake and active technical/BA documents | Reviewed the current monolith direction and allowed scope |
| `git status --short --branch` and `git rev-parse HEAD origin/develop` | Feature branch and local SHAs recorded; no remote freshness claim |
| `git diff --check` | Passed for tracked working-tree changes; excludes untracked-file validation |
| Concrete relative Markdown-link check across eight active guidance/intake/BA files | Passed: referenced local targets existed |
| Requirement/task table extraction | Eleven unique requirement IDs; fifteen task rows, all explicitly planned |
| `.env.example` active-key uniqueness check | Passed; provider configuration contract remains unchanged |
| `rg --files` for project/C#/.cshtml/Dockerfile/YAML/runsettings artifacts | Zero matches; no implemented host or deployment was found |

Runtime build/tests, browser execution, provider integration, Docker/cluster execution, TDD cycle evidence and coverage measurement are **not applicable** to this document-only review. None is claimed as passing.

## QA Handoff

QA should independently verify the current monolith documents, role instructions, requirement/task traceability, route/service boundaries, UI/API controls, coverage scope and multi-replica constraints. Review README/delivery finalization after QA and SEC; prior API-only approval cannot substitute for this revision's review. SEC follows passing QA, then DocKeeper finalizes current documentation. Fresh remote integration/conflict checks and affected rechecks remain mandatory before any PR into `develop`.

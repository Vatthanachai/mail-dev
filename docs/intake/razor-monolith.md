# PM Intake: Razor Pages + API Monolith

## User Request and Scope

The user selects a monolith and changes the planned API-only host to .NET 10 Razor Pages plus API Controllers. Continue the established guidelines/planning-first scope: update architecture, contributor/agent instructions, requirements, PLAN and Tasks; do not scaffold or claim runtime implementation.

## Confirmed Direction

- One ASP.NET Core host, process, application image and deployment serves Razor Pages UI, static assets, controller APIs, and gated OpenAPI/Scalar.
- Keep logical N-Tier projects: `MailDev.Web` (presentation/composition), Application, Domain, Infrastructure. Replace the proposed `MailDev.Api` host name with `MailDev.Web`; no populated project currently requires renaming.
- Razor PageModels and API Controllers share Application services through DI. PageModels do not call their own HTTP API for server-side work. Browser JavaScript may call same-origin `/api/*` when needed.
- Razor Pages means `.cshtml` plus PageModel, not Blazor. No separate frontend runtime or deployable is required.
- Keep PostgreSQL/SQLite, Volume/S3/Azure Blob, InMemory/Redis, `.env` selection, TDD/SOLID/OOP, coverage ≥80%, QA → SEC → DocKeeper and Git Flow.
- Docker, k3s and k8s run the same monolith artifact. Multiple replicas are whole-app replicas, not separately deployed tiers. Plan probes, graceful shutdown, persistence and shared Data Protection keys for cookie/antiforgery compatibility across replicas; never put these keys or secrets into images/source.

## Risks and Decisions

Prevent UI/API route collisions, business-logic duplication, direct provider access from PageModels, CSRF/XSS/open-redirect risks, inconsistent authorization/error behavior, and cookie/key failures across pods. Authentication policy and domain pages/endpoints still require feature requirements. Razor Pages forms must preserve antiforgery; cookie-authenticated unsafe API requests need explicit CSRF protection. Do not infer that using API Controllers alone provides antiforgery.

Priority is to align the platform plan before implementation; no deadline is committed. Branch: `feature/git-flow-guidelines`, preserving related uncommitted work and the concurrent empty solution shell. PR target: `develop`; no fresh remote conflict verification or PR is claimed.

## Handoff and Acceptance

BA revises the active `dotnet10-platform` requirements/PLAN/Tasks, preserving IDs and adding requirements/tasks as needed. Dev updates/reviews technical guidance. QA verifies docs and traceability, then SEC reviews security design. DocKeeper finalizes README and `docs/delivery/razor-monolith.md` only after both pass. Earlier `dotnet10-platform` QA/SEC/delivery reports remain historical evidence for the prior API-only plan; they do not approve this revision.

Complete when active guidance consistently describes one Razor Pages + API host, shared Application services and N-Tier boundaries; all future work remains planned. Runtime tests and measured coverage are not applicable to this documentation-only change.

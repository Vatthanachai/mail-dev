# Project Status

## Current: Razor Pages + API Monolith Plan

- Intake: [razor-monolith](intake/razor-monolith.md); scope remains guidelines/planning only.
- Direction: one .NET 10 `MailDev.Web` host with Razor Pages and API Controllers; N-Tier class libraries, one Docker image/deployment, Docker/k3s/k8s.
- Branch: `feature/git-flow-guidelines`; PR target: `develop`. Existing uncommitted work and the empty solution shell are preserved.
- BA: active [requirements](requirements/dotnet10-platform.md), [PLAN](plans/dotnet10-platform/PLAN.md) and [Tasks](plans/dotnet10-platform/Tasks.md) revised; 11 requirements and 15 planned implementation tasks.
- Dev: [current technical review](implementation/razor-monolith.md) complete; no blocking planning inconsistencies found.
- QA: [current documentation review](qa/razor-monolith.md) passed; runtime validation/coverage not applicable.
- SEC: current security-design review in progress; DocKeeper follows after pass. Prior reports below cover the earlier API-only plan.
- Implementation: not started; runtime tests and coverage not applicable to this documentation-only task.
- PR: not opened; latest remote conflict/integration gate remains required before any PR.
- Next action: complete current SEC review and have DocKeeper finalize the monolith README/delivery documents.

## Previous: .NET 10 Platform Guidelines and Plan

- Branch: `feature/git-flow-guidelines`; PR target: `develop`.
- Scope: guidelines and planning only, as selected by the user. Application implementation has not started.
- PM: intake documented in [dotnet10-platform.md](intake/dotnet10-platform.md).
- BA: [requirements](requirements/dotnet10-platform.md), [PLAN](plans/dotnet10-platform/PLAN.md), and [Tasks](plans/dotnet10-platform/Tasks.md) completed for planning; implementation tasks remain planned.
- Dev: [technical guidance review](implementation/dotnet10-platform.md) completed; no application implementation.
- QA: [documentation review](qa/dotnet10-platform.md) passed; runtime validation and coverage are not applicable to this scope.
- SEC: [security-design review](security/dotnet10-platform.md) passed for documentation scope; no runtime security approval is implied.
- DocKeeper: [README](../README.md) and [delivery index](delivery/dotnet10-platform.md) completed after QA and SEC pass.
- Local documentation/planning delivery: complete; all 13 application implementation tasks remain planned.
- PR: not opened; remote integration/conflict gate not yet executed.
- Next action: resolve the remaining design decisions and begin implementation when requested. Before any PR, perform the remote integration/conflict gate and repeat affected checks.
- Risks/open decisions: see intake and BA requirements; no dates or implementation success claims are committed.

# Development Standards

## Technical Baseline

Use a C#/.NET 10 monolith combining Razor Pages and API Controllers in `MailDev.Web`, with N-Tier class libraries and one Docker application artifact for Docker/k3s/k8s. Follow [architecture](architecture.md) and the PM-reviewed BA requirements, PLAN, and Tasks. Populated projects, provider implementations, tests, and CI remain future deliverables.

## TDD: Red, Green, Refactor

1. **Red:** Write a focused xUnit behavior test for the next acceptance criterion or bug. Run it before implementation and confirm it fails for the intended reason; missing infrastructure or compilation failures are separate setup problems.
2. **Green:** Implement the smallest maintainable change that passes it. Run focused and relevant existing tests.
3. **Refactor:** Improve structure without changing behavior, keeping tests passing. Run formatting, analyzers, tests, and coverage before handoff.

Record test names, requirement/task IDs, commands, genuine initial failures, passing results, and refactoring in `docs/implementation/<feature>.md`. Do not invent Red evidence retrospectively or require a separate commit for each cycle.

## SOLID and OOP

- **Single Responsibility:** Keep Razor PageModels/API Controllers, application orchestration, domain rules, and provider I/O separate.
- **Open/Closed:** Extend provider behavior with cohesive interfaces and adapters at real variation points.
- **Liskov Substitution:** Implementations honor the same contracts, errors, ownership and cancellation semantics; use shared contract tests.
- **Interface Segregation:** Expose small interfaces aligned with consumer needs.
- **Dependency Inversion:** Domain/application rules depend on abstractions, with concrete adapters registered at the Web host composition root.

Use encapsulated classes/value objects, interfaces, constructor injection, and composition. Avoid unnecessary inheritance or generic repository layers. PageModels and Controllers share Application services through DI, not loopback HTTP; neither accesses DbContext or provider SDKs. Keep business behavior out of `.cshtml` and PageModels thin.

Use four-space indentation, nullable reference types, PascalCase public names, camelCase locals/parameters, Async suffixes, async I/O and cancellation tokens. Add `.editorconfig` and analyzers during scaffolding. Validate options on startup and do not log credentials.

## Coverage Gate: At Least 80%

Measure **aggregate executable line coverage across all first-party production projects** (Web, Application, Domain, Infrastructure), including PageModels, Controllers and host configuration. Exactly 80% passes. This is not per-file, changed-line, or branch coverage. Browser/markup behavior still needs functional tests; coverage does not replace acceptance tests or security review.

Plan xUnit with an explicitly selected VSTest-compatible runner, `Microsoft.NET.Test.Sdk`, and `coverlet.collector`. Select and pin compatible package/tool versions during scaffolding. Coverlet collector is a VSTest integration; do not mix it with Microsoft.Testing.Platform. If selecting MTP later, document and verify an appropriate collector before changing commands.

Merge all test-project Cobertura reports using a repository-local pinned ReportGenerator tool. An implementation must configure production assembly inclusion, `IncludeDirectory`/`IncludeAssembliesWithoutSources` or equivalent as supported by the selected Coverlet version, and verify that unreferenced/unexercised production assemblies are represented rather than silently omitted. The gate must fail on missing, empty, incomplete, or stale results.

Planned command sequence after test projects, packages, local tool manifest, and coverage configuration exist:

```sh
dotnet tool restore
dotnet restore --locked-mode
dotnet build -c Release --no-restore -warnaserror
dotnet test -c Release --no-build --collect:"XPlat Code Coverage" --results-directory artifacts/coverage/raw
dotnet reportgenerator "-reports:artifacts/coverage/raw/**/coverage.cobertura.xml" "-targetdir:artifacts/coverage/report" "-reporttypes:Html;TextSummary;Cobertura" "minimumCoverageThresholds:lineCoverage=80"
```

These examples are not executable against this documentation-only repository yet. Before each measurement, start with a fresh task-specific results directory or remove only the verified workspace-owned coverage results directory. Do not merge profiles from previous runs or revisions. CI must run a validated repository coverage script that checks report completeness and propagates nonzero test/report/threshold exits.

Record source revision, SDK/package/tool versions, command, build/feature configuration, included assemblies, covered/executable lines, total percentage, report path, and exclusions. Do not average project percentages; merge coverage for identical source lines across test assemblies. Exclude only documented generated/vendor/test sources, never first-party behavior to reach 80%. Review exclusions through PM. Keep business logic, startup/configuration, and adapter code in scope.

Dev supplies evidence; QA independently repeats the gate on the final integrated implementation. Below 80%, missing tooling or unavailable required measurements block SEC handoff, final documentation and PR creation. Add the gate to CI during platform implementation. For documentation-only changes, record coverage as not applicable without implying a runtime pass.

## Provider Verification and Security

Test Razor rendering, form GET/POST validation and antiforgery, API routing/JSON/status codes, UI/API authorization parity, and common Application-service behavior. Use ASP.NET Core integration tests for both surfaces and browser tests for critical journeys/static assets. Ensure `/api/*`, page routes and Scalar/OpenAPI do not collide; disabling API docs must leave UI/APIs working. Test encoding/untrusted output, redirects, cookie-authenticated API CSRF protection, and shared Data Protection key behavior across replicas when cookies/antiforgery are used.

Package one Web host with pages/static assets and class libraries. Verify Docker and planned k3s/k8s probes, graceful shutdown, forwarded-header trust, mounted permissions and whole-app replicas. Explicitly test/document SQLite/Volume/InMemory constraints; they do not automatically provide shared multi-replica state.

Test database contracts against PostgreSQL and SQLite independently, including separate migrations and restart persistence. Test file storage contracts against Volume/S3/Azure Blob and cache contracts against InMemory/Redis; verify failure, expiry/invalidation, cancellation, security boundaries, and no silent provider fallback. Use isolated services/emulators and clearly distinguish emulator evidence from real cloud integration.

After QA passes, SEC reviews code, NuGet advisories, Razor encoding/antiforgery, cookie/API CSRF and authorization, configuration, file paths/permissions, cloud access, cache isolation, Data Protection keys, Scalar/OpenAPI exposure, and container controls. Security fixes follow TDD → QA regression/coverage checks → SEC re-review before DocKeeper or PR.

## References

- [ASP.NET Core controller-based API](https://learn.microsoft.com/en-us/aspnet/core/web-api/?view=aspnetcore-10.0)
- [Razor Pages](https://learn.microsoft.com/en-us/aspnet/core/razor-pages/?view=aspnetcore-10.0)
- [Coverlet collector and test-platform compatibility](https://github.com/coverlet-coverage/coverlet)
- [ReportGenerator threshold settings](https://github.com/danielpalme/ReportGenerator/wiki/Settings)

# Development Workflow

## Required Sequence

User requirement → PM intake and documentation → BA requirements, PLAN and Tasks → Dev implementation → Tester/QA verification → SEC security review → DocKeeper documentation → PM delivery.

Follow this sequence without asking the user to authorize each routine handoff. Wait for each stage's deliverables before dependent work begins. QA defects loop through Dev/QA; SEC findings loop through Dev/QA/SEC. DocKeeper starts final documentation only after both QA and SEC pass all required checks.

All documents and implementation follow [Git Flow](git-flow.md). PM establishes a feature branch from `develop` before intake/document edits. After DocKeeper completes, PM coordinates integration with latest `origin/develop`, conflict resolution, and affected re-verification before opening every PR into `develop`.

The primary agent acts as PM. Specialist definitions live in `.codex/agents/`. If named agents are unavailable, pass the corresponding definition's instructions to an available subagent and disclose the fallback. Respect concurrency limits and assign one writer per file. Do not claim delegation or verification occurred without evidence.

## Documents and Traceability

Use one lowercase kebab-case `<feature>` slug across a feature's documents. Use stable IDs such as `REQ-001`, `AC-001`, `TASK-001`, and `BUG-001`; qualify IDs with the feature slug when cross-referencing features. Preserve IDs when revising documents.

| Owner | Required document | Contents |
| --- | --- | --- |
| PM | `docs/intake/<feature>.md` | User request, objective, scope/exclusions, priority, constraints, assumptions, questions, risks, dependencies, success criteria, decisions |
| PM | `docs/project-status.md` | Feature stage, task owners/status, blockers, next action, evidence and decision links |
| BA | `docs/requirements/<feature>.md` | Actors, stories, rules, flows, errors, boundaries, relevant nonfunctional requirements, acceptance criteria |
| BA | `docs/plans/<feature>/PLAN.md` | Approach, affected components/interfaces, data/configuration needs, dependencies, sequence, verification strategy, risks |
| BA | `docs/plans/<feature>/Tasks.md` | Task ID, requirement/acceptance links, description, owner, dependencies, completion criteria, status, evidence |
| Dev | `docs/implementation/<feature>.md` | Task IDs, changed paths, decisions, configuration changes, exact checks/results, limitations, QA handoff |
| QA | `docs/qa/<feature>.md` | Acceptance/check matrix, environment and implementation revision, commands/results, defects, re-test evidence, verdict |
| SEC | `docs/security/<feature>.md` | Reviewed source/target revisions, scope, threat boundaries, checks/tools/advisory freshness, findings/severity, remediation/re-check evidence, limitations, verdict |
| DocKeeper | `README.md`, relevant `docs/`, `docs/delivery/<feature>.md` | Verified usage/operations, documentation index, requirement-task-test links, delivered changes, limitations |

Create documents as their stage begins; do not invent requirements or results. BA owns plan and task definitions. Dev updates implementation task progress/evidence; QA records verification; PM reconciles overall status. Coordinate edits rather than overwriting another agent's work.

## 1. PM: Receive and Prepare

Record the user's request and PM assessment in the intake before handing off. Establish scope, priority, constraints, dependencies, risks, and expected outcomes. Keep a .NET 10 Razor Pages + API Controllers monolith, N-Tier class libraries and one Docker artifact for Docker/k3s/k8s as the direction; follow [architecture](architecture.md). Separate confirmed decisions from assumptions.

Update project status and assign BA the intake, expected artifacts, owned paths, and questions. Continue authorized work autonomously; ask the user for material missing business decisions or scope changes. Do not invent deadlines, commitments, or approval.

**Exit:** Intake is documented and sufficient for BA analysis. Material unknowns are explicit; dependent implementation waits for resolution.

## 2. BA: Analyze and Plan

Read the intake and relevant documents. Produce requirements, `PLAN.md`, and `Tasks.md`. Define testable Given/When/Then acceptance criteria, including error cases and relevant .NET/Docker operational needs. Map every requirement to tasks and planned QA checks.

Split work into actionable tasks with owners, dependencies, and completion criteria. Ground proposed approaches in the repository; Dev validates feasibility and reports necessary plan changes. Resolve material questions through PM. PM reviews completeness and scope before handing the package to Dev; no additional routine user approval is required.

Include TDD cycles, SOLID/C# OOP considerations, coverage setup and the minimum 80% line-coverage gate in PLAN/Tasks, following [development standards](development-standards.md).

**Exit:** Requirements, acceptance criteria, PLAN, and Tasks are consistent, traceable, and ready for implementation; material decisions are resolved.

## 3. Dev: Implement

Follow the BA package and assigned tasks. Update task progress and implementation evidence. Implement C# code, Docker configuration, and meaningful tests. Return scope/requirement changes to PM and BA before dependent implementation.

Use TDD (Red → Green → Refactor) for behavior changes and record actual cycle evidence. Apply SOLID and C# OOP using encapsulation, interfaces, and composition. Measure at least 80% aggregate solution application/library line coverage using Coverlet and merged ReportGenerator reports under [development standards](development-standards.md).

Run applicable formatting, lint, build, test, and container checks from `AGENTS.md`. Record exact outcomes, including missing prerequisites. Submit the implementation report and task links through PM to QA; developer checks do not replace independent QA.

**Exit:** Assigned implementation is complete, developer checks pass, and the handoff identifies the implementation revision and validation evidence. Unavailable required checks remain blocked.

## 4. Tester/QA: Verify and Re-test

Independently check every acceptance criterion and required verification item. Record pass/fail/blocked/not-run, commands, environment, evidence, and requirement/task links. Mark checks not applicable only with a recorded rationale consistent with scope.

Review TDD evidence and SOLID/OOP design against the development standards. Independently re-run the coverage gate on the final integrated implementation: line coverage must be at least 80%. Record metric, scope, versions, command, report, and source revision. Missing measurement or insufficient coverage blocks final documentation and PR creation.

Report defects with IDs, severity, reproduction steps, expected/actual behavior, and evidence. PM routes defects to Dev; Dev fixes and records results; QA re-tests fixes and affected regressions. Requirement defects return to BA through PM. Update plans and evidence when scope changes.

**Exit:** All required criteria and checks pass, all defects in scope are resolved, and no required check is blocked or unexecuted. PM hands the QA report and implementation revision to SEC. Otherwise remain in the Dev/QA loop or report the blocker.

## 5. SEC: Review Security

After QA passes, PM assigns `sec` the requirements, PLAN/Tasks, implementation, full diff against `develop`, QA evidence, and source/target revisions. Review relevant trust boundaries, input handling, injection/path traversal, authentication/authorization, secrets/logging, unsafe/native interop, resource limits, dependencies, and Docker privileges/build context. Scope checks to implemented behavior. Tools supplement manual code review; verify version-specific advisories against current authoritative sources.

Create `docs/security/<feature>.md` with scope, evidence, tool/advisory versions and freshness, commands, revisions, required-check results, limitations, and pass/fail/blocked verdict. Track findings as `SEC-001`, etc., with severity/rationale, affected paths, safe reproduction, impact, recommended remediation, status, and re-check evidence. Redact secrets. Explain false positives and not-applicable checks rather than silently dropping them.

SEC reports findings through PM to Dev; requirement changes return to BA. Dev fixes, QA re-runs affected acceptance/regression checks and the 80% coverage gate, then SEC re-reviews. SEC does not patch production code or override QA. Repeat this sequence for fixes and security-relevant integration/conflict changes.

**Exit:** QA is passing for the reviewed implementation, all confirmed in-scope security findings are resolved, required security checks pass, and SEC records a passing report. Unresolved findings or missing required checks block DocKeeper and PR creation. For documentation-only changes, review relevant instructions/configuration and record why code/dependency checks are not applicable. Pass applies only to the recorded review scope and revision.

## 6. DocKeeper: Finalize Documentation

After QA and SEC pass, use intake, BA artifacts, implementation notes, QA evidence, and the security report to update setup, usage, configuration, Docker operation, troubleshooting, relevant architecture decisions, and delivery notes. Check links, paths, and examples against actual files. Trace requirements through tasks to tests and delivered behavior; link the SEC report and document verified security-related operating constraints without exposing secrets.

Do not change requirements, source behavior, QA conclusions, or SEC findings. Return inconsistencies to PM and the responsible role. If a correction changes implementation, repeat affected Dev/QA/SEC work before finalizing documentation. SEC rechecks security-relevant documentation/configuration changes.

**Exit:** Documents describe the verified implementation, traceability is complete, and unresolved documentation gaps are reported.

## 7. PM: Close and Report

Review artifacts, acceptance matrix, passing SEC report, task completion, and documentation. Complete the Git Flow conflict gate and record source/target SHAs. Re-run affected Dev/QA/SEC checks and reconcile documentation after integration, then open the PR into `develop`. Track `ready-for-pr`, `pr-open`, and `merged` distinctly; mark development artifacts complete only after all gates pass. Report scope, QA/security results, conflict outcome, document links, PR URL/status, and limitations in Thai unless requested otherwise. This workflow does not itself authorize PR merge, deployment, or publication.

## Example Request

`รับ requirement นี้แล้วดำเนินการตาม workflow ให้ครบ PM → BA (Requirements/PLAN/Tasks) → Dev → QA → SEC → DocKeeper พร้อมบันทึกเอกสารแต่ละขั้น`

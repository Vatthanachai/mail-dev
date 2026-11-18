# Delivery: MailDev Server Analysis and Plan

## Scope and Status

The user confirmed a phased approach: phase 1 captures and inspects development email via SMTP and API, with a POP3 server that lets email clients retrieve messages. After phase 1 passes acceptance, the required phase 2 adds real external sending via SMTP and API. POP3 retrieves messages; it does not send them. External delivery remains disabled by default and requires an explicit activation policy. Upstream POP3 polling is outside confirmed scope; Razor UI screens remain conditional on agreed requirements.

The plan uses the existing .NET 10 Razor Pages/API N-Tier monolith, with SMTP/POP3 TCP listeners in the same process/image. Docker, k3s and Kubernetes need suitable TCP exposure alongside HTTP. Database, file storage and cache use the existing `.env` provider contract. [Platform tasks](../plans/dotnet10-platform/Tasks.md) are prerequisites.

This delivery comprises analysis and planning only: no SMTP/POP3 server, API implementation, provider integration, Docker deployment, runtime tests, security scans, or measured code coverage is claimed. All 16 implementation tasks remain planned. No PR or remote conflict verification is claimed by this planning delivery.

## Requirements and Task Traceability

The table maps future [requirements and acceptance criteria](../requirements/maildev-server.md) to their [planned tasks](../plans/maildev-server/Tasks.md). Evidence is documentation review only; runtime criteria remain unexecuted. Task 013 is independent QA, 014 is SEC, 015 is DocKeeper, and 016 is the PM Git Flow gate as applicable to each phase.

| Requirement / acceptance | Planned tasks | Focus |
| --- | --- | --- |
| REQ/AC-001 | TASK-001, 002, 005, 013–015 | Listener and mode selection |
| REQ/AC-002 | TASK-001, 003, 005, 013–015 | SMTP envelope and accepted recipients |
| REQ/AC-003 | TASK-001, 003–005, 013–015 | Durable capture and recovery |
| REQ/AC-004 | TASK-001, 003–005, 013–015 | Raw MIME and attachments |
| REQ/AC-005 | TASK-001, 003, 006, 013–015 | POP3 session and deletion semantics |
| REQ/AC-006 | TASK-001, 003, 004, 006, 013–015 | Mailbox isolation and concurrency |
| REQ/AC-007 | TASK-001, 007, 013–015 | Authorized API reads and management |
| REQ/AC-008 | TASK-001, 007, 013–015 | API capture/submission behavior |
| REQ/AC-009 | TASK-001–003, 005–007, 011, 013–015 | Security and operational controls |
| REQ/AC-010 | TASK-001, 002, 005, 006, 011, 013–015 | Listener configuration and deployment |
| REQ/AC-011 | TASK-001–004, 006, 011, 013–015 | Provider and replica safety |
| REQ/AC-012 | TASK-001, 002, 012–016 | Tests, coverage and delivery gates |
| REQ/AC-013 | TASK-001, 009, 013–015 | Required phase 2 external delivery |
| REQ/AC-014 | TASK-001, 010, 013–015 | Upstream POP3 only if separately requested |
| REQ/AC-015 | TASK-001, 008, 013–015 | Razor UI only after scope is agreed |

## Review and Open Decisions

- [Dev feasibility](../implementation/maildev-server.md) records planned protocol, persistence and deployment approaches; dependencies remain subject to evaluation.
- [QA](../qa/maildev-server.md) passed the planning review; [SEC](../security/maildev-server.md) passed the security-design review after QA. Neither result is runtime acceptance or a dependency audit.
- TDD and SOLID/OOP apply to implementation, with at least 80% aggregate production executable-line coverage. Coverage has not been measured.
- Before dependent implementation, agree mailbox provisioning/identity and authorization, recipient mapping, API and UI use cases, limits, retention, TLS/authentication, supported protocol profile and replica topology. Before phase 2, decide direct MX versus authenticated smarthost, delivery/retry/DSN policy and operational activation controls.

PM owns remaining decisions and the pre-PR conflict gate. Consult the [project status](../project-status.md) and [Git Flow guide](../git-flow.md) for current progress.

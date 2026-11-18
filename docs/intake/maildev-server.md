# PM Intake: MailDev Server

## User Request and Delivery Scope

Analyze and plan a MailDev Server supporting email reception and sending through POP3, SMTP, and API. This task delivers requirements, architecture, phased PLAN and Tasks, and reviews; it does not implement services or claim protocol conformance. The previous planning PR was reported complete by the user. Work starts from the fetched `origin/develop` on `feature/maildev-server-plan`.

The user confirmed two stages: first capture and inspect messages for development; after that works, support delivery to real external recipients. The user also confirmed a POP3 server for external mail clients to retrieve messages. POP3 is an email access/retrieval protocol, not a sending protocol; real sending uses SMTP submission/relay or an API that submits to the delivery service. An upstream POP3 client is outside the confirmed scope. Direct MX delivery versus authenticated SMTP relay remains an architectural decision, not a user decision.

## Established Constraints

- .NET 10/C#, Razor Pages and API Controllers in one N-Tier monolith/image; Docker, k3s and k8s.
- PostgreSQL or SQLite, Volume/S3/Azure Blob, InMemory/Redis selected through `.env` conventions; do not assume implementation exists.
- Scalar/OpenAPI for HTTP API. SMTP/POP3 are TCP protocols and need distinct listeners and service exposure; OpenAPI does not document their wire protocols.
- TDD, SOLID/OOP, ≥80% aggregate line coverage, QA then SEC then DocKeeper, and Git Flow/PR conflict checks.

## PM Analysis and Risks

Define explicit message lifecycle, raw RFC 5322/MIME preservation, envelope versus headers, recipients and mailbox access, attachments, persistence and cross-protocol visibility. SMTP acceptance must occur only after durable capture; document crash consistency between database and object storage. Handle message/body limits, TCP concurrency/timeouts, authentication/TLS policies and secrets. POP3 deletion needs transaction semantics and consistent mailbox locking. The required later outbound phase needs recipient-level outcomes, queue/retry semantics, failure reporting and abuse controls; do not assume direct-MX or promise delivery guarantees not yet designed.

Docker/k3s/k8s require independent SMTP/POP3 TCP ports, shared storage and coordination across replicas; HTTP ingress alone does not establish TCP exposure. Avoid claiming multi-replica safety with SQLite/local volumes or process-local locks. Define data protection, threat model, privacy and redacted logs before runtime design.

Use current standards and primary library documentation; distinguish protocol server libraries from client libraries. Validate library licensing, .NET 10 compatibility, capability fit and maintenance before choosing. No protocol endpoints, external relay, user identities, ports, or release dates have been approved merely by creating this intake.

## BA Handoff

Create `docs/requirements/maildev-server.md`, `docs/plans/maildev-server/PLAN.md`, and `docs/plans/maildev-server/Tasks.md` with feature-qualified stable IDs, requirements, acceptance criteria, alternatives, dependencies, traceable tasks, milestones, tests and security/deployment review. Identify material questions and mark conditional work pending user decisions. Keep the prior platform plan as prerequisites; avoid inventing current implementation or merging planning status with runtime delivery.

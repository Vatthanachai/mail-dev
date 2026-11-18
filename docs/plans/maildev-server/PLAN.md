# PLAN: MailDev Server

## Status and Decision Gate

This is a **development plan**, not an implemented server or a conformance claim. The preceding [.NET platform tasks](../dotnet10-platform/Tasks.md) supply the unimplemented host, dependencies, providers, Docker and coverage prerequisites. Requirements and ACs are in [maildev-server](../../requirements/maildev-server.md); transport choice, user identities, UI and protocol profile decisions remain open there. [Storage intake](../../intake/mail-storage.md) supersedes earlier full-message object-storage proposals; prior QA/SEC reports are historical and do not approve this revision.

The user confirms phase 1: capture/inspect mail for development, with a local POP3 server and HTTP API. Phase 2 is required: add real external SMTP/API sending only after phase 1 is working and passes acceptance. Keep real delivery disabled by default; decide direct MX versus smarthost and security/operations policies before implementing phase 2. The plan does not authorize sending real messages. Upstream POP3 polling is outside confirmed scope and requires a separate request. No schedule or effort estimate is implied.

## Conceptual Monolith

```text
Clients ──SMTP/TCP──┐
Mail clients ─POP3/TCP─┼─> MailDev.Web host (one process/image)
Browser / callers ─HTTP─┘      ├─ Razor Pages + API Controllers
                               ├─ SMTP listener / POP3 listener* (hosted services)
                               └─ Application services + protocol ports
                                      ├─ Domain: message, envelope, delivery, mailbox
                                      └─ Infrastructure: DB, blob storage, adapters
```

`*` Local POP3 listener is confirmed for phase 1. Listeners are lifecycle-managed .NET hosted services; they use adapters/protocol interfaces and Application use cases rather than calling HTTP endpoints. API Controllers and PageModels use the same Application services. Boundary placement between hosted service and Infrastructure is selected during design without allowing domain code to depend on protocol/network/EF/SDK details. Phase 2 external-delivery adapters, and any separately requested upstream clients, are outbound adapters in the same process; this is not a separate deployed microservice.

The selected PostgreSQL/SQLite database is authoritative for the original RFC 5322/MIME bytes after SMTP DATA unstuffing, message metadata/body and headers, envelope sender/accepted recipients, mailbox deliveries and attachment metadata/references. Do not infer Bcc recipients from headers. Extracted, decoded attachment bytes live in the Volume/S3/Azure Blob backend selected by `Storage__Provider`. The user accepts duplicated encoded attachment bytes inside database raw MIME. API raw reads and POP3 retrieval use the same DB bytes, with correct POP3 octet counts and dot-stuffing; attachment downloads resolve authorized metadata references to decoded content. Define inline, nested, malformed and unsupported MIME classification before implementation; do not silently omit required attachments.

For SMTP DATA, validate limits and use bounded staging while preserving the unstuffed bytes. Parse according to the approved MIME policy; durably stage and verify each required decoded attachment object (including size/checksum). Then commit raw MIME, envelope, metadata, attachment references and all accepted mailbox deliveries in one DB transaction, and only then return SMTP success. Partial parsing, write or DB failures must fail without visible partial deliveries. DB and Volume/S3/Azure are not a distributed transaction: assign stable object IDs and staged states; after crashes reconcile and remove only demonstrably unreferenced objects after a safe grace period and coordination with concurrent writers/readers. Reference ownership must protect multi-recipient/shared objects. An ambiguous lost response can still cause retry duplicates. Proposed ingestion is fail-closed when the configured attachment store is unhealthy even for attachment-free messages; validate availability/health timing and document before implementation.

Store complete raw MIME and extracted attachments deliberately twice. Plan provider-specific DB limits/streaming, attachment and total-message quotas, at-rest protection, retention, and coordinated database plus attachment backup/restore. Verify checksums and consistency on restore; decide lifecycle and authorization of inline/attached parts, missing objects, read-versus-delete races and retention before coding. PostgreSQL/SQLite and the selected storage must both meet the documented durability model; SQLite plus local Volume remains subject to single-instance deployment constraints.

For the confirmed local POP3 server, implement a session state machine (authorization, transaction, update), stable snapshot, mailbox-local persistent UIDLs and an exclusive mailbox lock valid across participating replicas. DELE marks session state, RSET clears marks and successful QUIT commits; abrupt disconnect must not commit. Enforce RFC CRLF, message octet counts and dot-stuffing, reject concurrent access safely, and define API deletion versus a POP3 session. Evaluate database transaction/lease ownership, expiration and fencing so a stale pod cannot commit after its lease expires; process-local InMemory cache is not a distributed lock. If these guarantees cannot be proven, deploy only one supported replica and document it.

## Phases and Deliverables

1. **Decisions and profile:** PM/BA record the confirmed phase 1 capture + POP3 server and required phase 2 real SMTP/API delivery. Agree persona/mailboxes, API use cases, security/retention, protocol subset/limits, TLS/auth, topology and acceptance criteria. Decide direct MX versus smarthost before dependent phase 2 work. Block only work affected by unresolved choices.
2. **Foundation and feasibility:** Complete applicable platform prerequisites. Benchmark/compare maintained .NET 10 libraries for SMTP server, POP3 server/client, MIME parsing and SMTP client as needed. Record license, compatibility, advisories, protocol profile, tests and decision; MailKit is a client, not an SMTP/POP3 server. Prototype only in assigned Dev work; no unverified framework/library selection here.
3. **Storage and application core:** Write TDD behavior tests first, define domain model, mailbox permissions, persistence/blob interfaces, configuration, durable ingestion/recovery and provider/migration strategies. Test PostgreSQL/SQLite and applicable object adapters independently with crash injection and isolation; API/message contracts are documented.
4. **Protocol and API:** Implement phase 1 SMTP ingress and local POP3 server, plus controller API and Scalar; provide distinct TCP listener configuration and status. Add UI only against explicit requirements. After phase 1 works and is accepted, implement required phase 2 SMTP/API external sending with a separate threat model and default-off activation; upstream POP3 requires a separate request.
5. **Operational deployment:** Publish a single monolith image; configure separate HTTP/SMTP/POP3 ports. For k3s/k8s plan TCP Services/load balancer/Ingress support appropriate to the actual controller, ConfigMap/Secret injection, probes, graceful shutdown, storage/locks, credentials/TLS and per-replica resource limits. Exercise both a supported local single-instance topology and explicitly chosen multi-pod topology; do not imply that an HTTP-only ingress handles SMTP/POP3.
6. **Delivery:** Dev runs TDD, build, format, unit/contract/integration/interoperability, crash/concurrency tests and solution-wide ≥80% executable production line coverage. QA independently tests approved ACs and provider/replica configurations; SEC reviews after QA; remediation loops Dev → QA → SEC. DocKeeper completes docs after QA+SEC pass. PM checks latest `origin/develop`, conflicts and affected verification before PR, following [Git Flow](../../git-flow.md).

## Verification Matrix

| Area | Required evidence before implementation acceptance |
| --- | --- |
| SMTP | Real SMTP client tests for supported HELO/EHLO, MAIL, multiple RCPT, DATA, RSET/NOOP/QUIT and only advertised extensions; invalid state, rejected recipient subset, limits, slow/disconnected clients, TLS/auth failures and storage failures. Verify byte content and acknowledgement only after durable commit. |
| POP3 server | Real POP3 client plus protocol transcript tests for CAPA, USER/PASS or chosen auth, STAT/LIST/UIDL/RETR/DELE/RSET/NOOP/QUIT, CRLF/dot-stuffing/octet count, UIDL stability, update on QUIT, disconnect rollback, isolation, simultaneous sessions and restart/cross-pod locking/fencing. |
| API/UI | `WebApplicationFactory` or equivalent real HTTP integration for auth, mailbox access, response contracts, pagination, raw/attachments, errors, upload limits and submission mode. Test Scalar + OpenAPI exposure policy and cookie CSRF. Test Razor encoding/forms/browser journeys only when UI scope is approved. |
| Persistence | Verify full unstuffed raw MIME, metadata and envelope in each selected database; decoded attachment bytes/checksums in each selected storage; nested/inline/malformed/attachment-free and large MIME fixtures; byte-identical raw API/POP3 before protocol framing; no attachments silently omitted. Test provider-specific migrations, quotas and supported combinations; failures/crashes before/after each attachment stage and DB commit, safe orphan cleanup, multi-recipient ownership, concurrent reads/deletes, backups/restore of both stores and no loss of acknowledged messages within the documented failure model. |
| Deployment/security | Docker restart/mount persistence; TCP exposure; k3s/k8s rollout/probes/drain; one- and multi-pod contention if supported; TLS/certificate validation, endpoint authentication, secrets/log redaction, rate/concurrency limits and open-relay tests. Never test against real third-party recipients without authorization. |
| Quality | Record Red/Green/Refactor evidence, source revision, SDK/tool versions, commands, reports, and all included production projects; clean merged Coverlet/ReportGenerator line coverage ≥80%, with required exclusions and runtime tests documented. QA and SEC explicitly record pass/fail/blocked/not-run and limitations. |

## Key Risks and Decisions

- SMTP accepts envelope recipients separately from MIME To/Cc/Bcc; mailbox mapping and accepted-recipient partial success need explicit policy.
- Provider-specific SQLite/volume behavior and POP3 lock lifetime/fencing can invalidate multi-pod guarantees. Establish a supported matrix, not a claim that all combinations work everywhere.
- Raw email and attachments can contain malicious HTML, tracking URLs, filenames or secrets; encode/sanitize previews, control attachment serving and avoid leakage through logs. Define auth/tenant policy rather than inventing it.
- Phase 2 real delivery can create an open relay, incorrect delivery claims and uncontrolled retries. It is required after phase 1 acceptance, with separate authorization, default-off activation and operational policy. This plan alone does not authorize sending actual messages.
- Shared DB/attachment storage offers no automatic distributed transaction; no SMTP success before DB raw MIME/metadata and required attachment objects are durable. Duplicate raw and decoded attachment bytes increase database/storage size, backup and retention costs. API read/delete and POP3 commit need consistent concurrency semantics.
- No release date, performance target, protocol-completeness claim or production-ready deployment is established. Record measurable limits and unsupported extensions when the user confirms requirements.

## Traceability

See [Tasks](Tasks.md) for owners, dependencies, requirement IDs and acceptance evidence. Planning artifacts and documentation reviews have their own status; do not mark future implementation tasks done based on those reviews.

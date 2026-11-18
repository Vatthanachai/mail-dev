# SEC Review: MailDev Server Plan

## Scope and Verdict

**PASS for security-design and documentation planning only.** SEC reviewed the PM intake, BA requirements/PLAN/Tasks, Dev feasibility review, existing architecture, and the [QA planning review](../qa/maildev-server.md), which passed before this review. The baseline is branch `feature/maildev-server-plan`, HEAD `c76dc05a4bb15eb01fb156d5c7352edeed51734e`; the reviewed plan is uncommitted working-tree content and is not identified by that commit.

The user confirmed phase 1 development capture and inspection with a POP3 server, followed by required real outbound delivery via SMTP/API in phase 2. POP3 retrieves mail; it does not send it. No confirmed security-documentation blockers remain. This verdict does not approve runtime code, production deployment, external mail transmission, any dependency version, or unexecuted tests.

## Threat Boundaries and Planned Controls

| Boundary | Planning review |
| --- | --- |
| SMTP ingest | Capture mode and external relay are distinct; no implicit relay. Plan authentication/TLS for exposed networks, correct STARTTLS capability negotiation, bounded connections, lines, recipients and message sizes, idle/total timeouts, command sequencing and safe response codes. Never log credentials or raw messages by default. Verify fail-closed behavior and open-relay rejection in implementation. |
| Real sending | Phase 2 is required after phase 1 passes; delivery remains disabled by default. Decide direct MX versus authenticated smarthost and define authorized senders, recipient restrictions, rate limits, queue/retries, per-recipient outcomes, audit and DSN/bounce-loop behavior before enabling it. SMTP transfer acceptance is not proof of inbox delivery; do not promise exactly once. Real external sends require separate operational authorization. |
| POP3 mailbox | Require authenticated access and mailbox isolation with protected credentials. Plan stable mailbox-local UIDLs and snapshot semantics; `DELE` marks, `RSET` clears, and successful `QUIT` from TRANSACTION commits. A disconnect must not commit deletion. Limit command sizes and sessions, and test lock loss/fencing so stale replicas cannot commit. See [RFC 1939](https://www.rfc-editor.org/rfc/rfc1939.html). |
| HTTP/API and UI | Define per-mailbox authorization for listing, raw messages, attachments, deletion and submissions. Prevent cross-mailbox disclosure, use non-leaking errors, preserve Razor output encoding and antiforgery, and explicitly protect unsafe cookie-authenticated API operations against CSRF. UI authorization alone does not secure API endpoints. Resolve UI scope before implementation. |
| Message content | Treat MIME, nested parts, filenames, HTML and URLs as untrusted. Bound streaming/parsing, message/attachment sizes and nesting/resource use; control download headers and content types. Design isolated previews with script and external tracking/network access blocked as appropriate; verify protection against stored XSS and malicious attachments before UI release. |
| Durable capture | Success after SMTP DATA requires recoverable durable message bytes and committed metadata/deliveries. The relational database and object store are not a distributed transaction: stage immutable objects, commit metadata, reconcile safely, and fault-test each crash boundary. Protect referenced content against cleanup races. A lost final response may cause duplicate delivery on retry. |
| Storage and replicas | Keep sensitive content and encryption/retention/backup policies explicit. Test database/object provider combinations and permissions; do not assume SQLite, local volumes or in-memory locks support multiple pods. Multi-pod POP3 locking needs a shared authority, expiry and fencing checked atomically at commit. Shared Data Protection keys are a separate requirement for web cookies/antiforgery. |
| Configuration/network | Inject `.env` explicitly for Docker, map its keys to Kubernetes ConfigMaps/Secrets or an approved secret provider, and never bake credentials into an image. Expose SMTP/POP3 using explicitly configured TCP services, not an HTTP-only ingress. Restrict network access, use least privilege, validate active provider options, fail startup safely on required invalid/missing security settings and redact diagnostics. |
| Dependencies | Evaluate server-side SMTP/POP3 implementations separately from MailKit's client role. Before adoption, review .NET 10 compatibility, license, maintained releases, transitive dependency advisories, exposed protocol features and test evidence. Security tooling availability and advisory freshness must be recorded in the later runtime review. |

TLS recommendations for mail submission/access and POP3 transport are documented in [RFC 8314](https://www.rfc-editor.org/rfc/rfc8314.html); SMTP semantics and extension negotiation are grounded in [RFC 5321](https://www.rfc-editor.org/rfc/rfc5321.html). These references do not establish that the future implementation conforms to any RFC.

## Evidence and Remaining Decisions

- Read the intake, requirements and ACs, PLAN/Tasks, Dev feasibility report, architecture, and QA report; traced security controls and phase gates across the plan.
- Ran `git status --short --branch`, `git rev-parse HEAD origin/develop`, `git diff --check`, and `git check-ignore .env .env.local .env.example`. The baseline was `c76dc05a4bb15eb01fb156d5c7352edeed51734e`; no remote fetch or conflict check was performed as part of this security review. `git diff --check` covers tracked differences, not all untracked plan documents.
- Existing ignore rules cover `.env` and `.env.local`; the example remains trackable. No populated secret values were read or included in this report.
- No server, manifests, cloud credentials, production test, runtime penetration test, package/advisory scan, TLS interoperability test, or coverage measurement was executed. Those are implementation/release gates, not passes here.

Before dependent implementation, PM/BA must agree mailbox identities and provisioning, authenticated and unauthenticated endpoint policies, permitted networks, concrete limits, content retention, browser preview/attachment policy, supported protocol profile and topology. Before phase 2, decide external transport, sender/recipient authorization, credential ownership, throttling, delivery status meaning, DSN handling and activation policy. If required network/service/security checks are unavailable or a confirmed vulnerability remains, mark that phase blocked rather than pass.

## Handoff

DocKeeper may finalize the planning documentation after this SEC planning pass and the existing QA planning pass. The future implementation follows Dev → QA → SEC with re-tests after fixes, the ≥80% aggregate line-coverage gate, and pre-PR conflict checks. Re-review security-relevant changes and runtime evidence for the actual implementation revision.

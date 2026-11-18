# PM Intake: Mail and Attachment Storage

## User Requirement

Email received by MailDev must be stored in the selected database (PostgreSQL or SQLite). Its attachments must be stored in the user-selected Volume, S3, or Azure Blob Storage, using the project's existing configuration contract. This revises the previous proposal to make full message blobs authoritative in object storage.

## Scope and Open Decision

Update the MailDev requirements, implementation plan and tasks; no runtime implementation is requested in this planning change. Preserve SMTP/POP3/API, phase-1 capture, phase-2 external delivery, N-Tier monolith, TDD, 80% coverage, and Git Flow.

The user confirmed that the database may retain complete original raw MIME, including encoded attachment bytes duplicated with separately stored attachments. Preserve the accepted MIME byte sequence in the database for POP3/API raw retrieval; define its boundaries after SMTP transport unstuffing. Store extracted attachment contents in the selected storage provider. Document the space, backup and retention impact of duplication.

## Analysis and Handoff

Database records hold email data, raw MIME, envelopes, mailbox associations and attachment metadata/references; selected storage holds extracted attachment content. Define how to classify MIME parts, handle inline attachments, malformed data, size/stream limits, retention, access, backup/restore and lifecycle of shared attachments.

Durably persist all required attachment objects and commit database mail/metadata/references before SMTP success. Define recoverable staged states, atomic database changes, orphan cleanup, retries, duplicate ambiguity, concurrent access and behavior during storage failure; no distributed transaction across DB and storage is assumed. Raw POP3/API retrieval, octet counts, UIDL, deletion and multi-recipient ownership must stay consistent.

BA revises the existing `maildev-server` requirements/PLAN/Tasks while preserving stable IDs. Dev examines alternatives and documents feasibility; QA reviews the updated plan, SEC reviews security, and DocKeeper updates current-facing documentation after both pass. Earlier reports remain historical and do not approve this change.

Branch: `feature/maildev-server-plan`; PR target: `develop`. No PR or remote conflict result is claimed for this revision.

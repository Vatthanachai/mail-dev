# Agent Workflow

## Roles

| Agent | Responsibility | Primary outputs |
| --- | --- | --- |
| `ba` | Define the problem, scope, business rules, and acceptance criteria | `docs/requirements/<feature>.md` |
| `dev` | Implement Rust code and Docker configuration; verify changes | Source, configuration, implementation tests |
| `qa` | Independently validate acceptance criteria and report defects | Integration tests, `docs/qa/<feature>.md` |
| `dockeeper` | Keep setup instructions and technical documentation accurate | `README.md`, `docs/`, relevant contributor guidance |

Definitions live in `.codex/agents/*.toml`. They inherit the parent session's model and permissions. This repository currently has no Rust or Docker implementation; agents must report missing prerequisites accurately.

## Handoff Sequence

1. BA records confirmed requirements, assumptions, stable IDs such as `REQ-001`, and testable acceptance criteria. Ask the user about unresolved decisions that affect implementation.
2. Dev implements the agreed scope and supplies changed paths, requirement IDs, validation results, and configuration changes.
3. QA checks each acceptance criterion, records evidence and defects, and returns pass, fail, or blocked. Failed checks return to Dev for correction and re-test.
4. DocKeeper updates documentation using the implementation and QA evidence. Planned behavior remains explicitly labeled.
5. The primary agent integrates results and reports remaining limitations to the user.

For small tasks, invoke only the relevant roles. Parallelize independent analysis or work on disjoint files; assign one writer per file. Respect the session's concurrency limit and queue other roles as needed. Include the task, inputs, owned paths, expected output, and dependencies in every delegation. Do not start dependent implementation before material requirements are resolved.

## Example Requests

- `ให้ ba วิเคราะห์ requirement ของฟีเจอร์นี้และเขียน acceptance criteria`
- `ให้ dev พัฒนาตาม docs/requirements/<feature>.md แล้วให้ qa ตรวจรับ`
- `ให้ dockeeper อัปเดตคู่มือ Rust/Docker จากโค้ดและผล QA ล่าสุด`

Use these definitions in a Codex client supporting project custom agents. If they are unavailable in the current session, start a new session in this repository; definitions alone do not launch agents or grant extra permissions. See the [official custom-agent documentation](https://learn.chatgpt.com/docs/agent-configuration/subagents#custom-agents).

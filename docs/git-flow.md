# Git Flow and Pull Requests

## Branch Policy

These rules apply to documentation, agent configuration, tests, and implementation.

- `main`: stable releases; do not develop directly here.
- `develop`: integration branch and target for completed development PRs; do not develop directly here.
- `feature/<slug>`: branch from latest `origin/develop` for each requirement or documentation change, for example `feature/docker-startup`. Keep PM, BA, Dev, QA, and DocKeeper artifacts for the same scope together on this branch.
- `release/<version>`: branch from `develop` when preparing an explicitly requested release; integrate through PRs into `main` and back into `develop`.
- `hotfix/<slug>`: branch from `main` for an explicitly requested production fix; integrate through PRs into `main` and `develop`. Check conflicts against each target separately.

PM records branch, scope, owners, and target in project status. Use focused commits with imperative subjects. Do not discard existing changes, rewrite shared history, or force-push to resolve conflicts. Coordinate agents on the same feature branch or isolated worktrees with explicit ownership.

## Starting Work

Inspect `git status --short --branch` first and preserve unrelated work. With a clean working tree and an available `origin/develop`:

```sh
git fetch origin --prune
git switch -c feature/<slug> origin/develop
```

Replace placeholders before running commands. If work already exists on `develop`, preserve it on a feature branch before continuing. Missing remotes or access are blockers to remote verification; do not describe local-only evidence as a check against the latest remote branch.

## Mandatory Check Before Every PR into develop

PM coordinates this gate after implementation, QA, SEC, and final documentation. It also applies to documentation-only PRs, with checks appropriate to that scope.

1. Commit the intended feature changes and ensure the working tree is clean. Confirm the source branch and PR target are correct.
2. Fetch the target and merge it into the feature branch:

   ```sh
   git fetch origin --prune
   git rev-parse HEAD origin/develop
   git merge --no-edit origin/develop
   ```

3. If the merge conflicts, stop PR creation. Resolve code conflicts through Dev, requirement/plan conflicts through BA, and final-document conflicts through DocKeeper; PM coordinates shared status files. Inspect both sides and preserve the intended behavior. Stage only resolved files and complete the merge commit.
4. Verify that no unmerged paths remain and review the full PR diff:

   ```sh
   git diff --name-only --diff-filter=U
   git diff --check origin/develop...HEAD
   git diff --stat origin/develop...HEAD
   git status --short --branch
   ```

   The unmerged-path output must be empty, diff checks must pass, and the working tree must be clean. A textual merge alone does not establish semantic compatibility.
5. Re-run affected validation after integration and every conflict fix. QA verifies acceptance criteria, regressions, and the minimum 80% line-coverage gate for .NET changes under [development standards](development-standards.md). SEC rechecks security-relevant changes after QA passes and records the reviewed revision. DocKeeper reconciles documentation afterward. Required failures, unresolved security findings, blocked checks, insufficient coverage, or unexecuted checks prevent opening the PR.
6. Record the fetched target SHA, final source SHA, conflict outcome/resolution, commands, and validation evidence in the QA report and delivery notes. For documentation-only changes, record link/path/format verification rather than claiming .NET or Docker tests ran.
7. Refresh `origin/develop` immediately before opening the PR. If its SHA changed, repeat integration, conflict verification, and affected checks. Push the feature branch and open a PR explicitly targeting `develop`. Confirm the remote PR source SHA matches the verified source SHA and inspect the hosting service's mergeability status. If it is pending or conflicts, report that status and resolve/recheck before declaring the PR ready.

## PR Contents and Completion

Include the problem and delivered behavior, intake/requirements/PLAN/Tasks links, configuration changes, QA evidence, passing SEC report, conflict-check SHAs/results, and relevant screenshots. List limitations accurately. Do not include secrets in commits or logs.

PM records the PR URL and status. Distinguish `ready-for-pr`, `pr-open`, and `merged`; opening a PR does not mean it is merged. Merge or release only when separately instructed or already authorized. If remote access or PR tooling is unavailable, retain the completed local artifacts and report the exact blocker instead of claiming a PR was opened.

# Agent Development Workflow

This repository uses an issue-driven, spec-first workflow. Every implementation must be associated with one GitHub Issue and one dedicated feature branch.

## Operating rules

- Work on one issue at a time.
- Do not modify `main` directly.
- Do not create a branch or write implementation code before the planning gate is approved.
- Do not merge pull requests.
- Do not close issues manually unless explicitly requested; use `Closes #<issue-number>` in the pull request body.
- Keep the implementation limited to the scope described in the issue.

## Phase 1: prepare the task and wait

When the user assigns an open GitHub Issue:

1. Read the complete issue, including its description and any existing comments.
2. Confirm that the issue is open and has not already been implemented.
3. Fill in these sections in the issue, in English:
   - `Acceptance Tests`
   - `Definition of Done`
   - `Implementation Plan`
   - `Changelog`
4. Make the acceptance tests observable and testable. Do not write vague criteria such as "works correctly".
5. Keep the `Changelog` as `TBD` until implementation changes are known, or describe the expected user-visible change if it is already clear.
6. Report the proposed plan to the user.
7. Stop and wait for explicit approval.

Do not create a branch, edit source files, run implementation commands, commit, push, or open a pull request during this phase.

An approval is valid only when the user explicitly authorizes implementation, for example:

```text
Approved. Implement the plan for issue #12.
```

## Phase 2: implement after approval

After explicit approval:

1. Update local `main` without rewriting history:

   ```bash
   git switch main
   git pull --ff-only origin main
   ```

2. Create a dedicated branch from the updated `main`.

   Branch format:

   ```text
   <type>/issue-<number>-<short-kebab-case-description>
   ```

   Allowed types include `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `ci`, and `hotfix`.

   Example:

   ```bash
   git switch -c feat/issue-12-hourly-forecast
   ```

3. Implement only the approved plan.
4. Add or update tests for the acceptance tests.
5. Update documentation when the behavior or public interface changes.
6. Replace the `Changelog` placeholder in the issue with a concise user-visible summary.
7. Run the repository verification commands. For Rust, use:

   ```bash
   cargo fmt --all -- --check
   cargo test --all
   cargo clippy --all-targets --all-features -- -D warnings
   ```

8. Verify every acceptance test and Definition of Done item.
9. Commit using Conventional Commits. Include the issue number when practical:

   ```text
   feat: add hourly forecast (#12)
   ```

10. Push the feature branch and open a pull request. Never merge it:

    ```bash
    git push --set-upstream origin <branch-name>
    gh pr create --base main --head <branch-name> \
      --title "feat: add hourly forecast" \
      --body-file <generated-pr-body-file>
    ```

## Pull request requirements

The pull request must include:

- a concise summary;
- the issue reference, using `Closes #<issue-number>`;
- the implementation plan actually followed;
- tests and verification commands executed;
- the acceptance tests and their results;
- the Definition of Done and its results;
- the changelog entry.

If a verification command fails, do not create the pull request until the failure is fixed or explicitly reported to the user.

## GitHub and permissions

- Use `gh issue view <number>` and `gh issue edit <number>` when GitHub CLI access is available.
- Use `gh pr create` to open the pull request only after implementation is complete.
- If authentication, network access, or push permission is unavailable, stop before the external operation and report the exact command that requires user action.
- Never bypass branch protection or merge a pull request.

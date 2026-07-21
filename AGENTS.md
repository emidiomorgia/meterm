# Agent Development Workflow

This repository uses an issue-driven, spec-first workflow. Every implementation must be associated with one GitHub Issue and one dedicated feature branch. Issues move through the following states: `Backlog`, `Ready`, `In Progress`, and `In Review`.

## Operating rules

- Work on one issue at a time.
- Do not modify `main` directly.
- Do not create a branch or write implementation code while refining a `Backlog` issue or before the issue is explicitly approved by moving it to `Ready`.
- Do not merge pull requests.
- Do not close issues manually unless explicitly requested; use `Closes #<issue-number>` in the pull request body.
- Keep the implementation limited to the scope described in the issue.

## Phase 1: refine a Backlog issue

An issue in `Backlog` may be selected for refinement. During this phase:

1. Read the complete issue, including its description and any existing comments.
2. Confirm that the issue is open and has not already been implemented.
3. Populate or update these sections in the issue, in English:
   - `Acceptance Tests`
   - `Definition of Done`
   - `Execution Plan`
   - `Changelog`
4. Make the acceptance tests observable and testable. Do not write vague criteria such as "works correctly".
5. Keep the `Changelog` as `TBD` until implementation changes are known, or describe the expected user-visible change if it is already clear.
6. Report the proposed execution plan to the user.
7. Leave the issue in `Backlog` and stop. The user approves the refinement by moving the issue to `Ready`.

Do not create a branch, edit source files, run implementation commands, commit, push, or open a pull request while refining a `Backlog` issue.

## Phase 2: start implementation from Ready

The agent may work on an issue only when both conditions are met:

1. The issue is in `Ready`.
2. The user explicitly asks the agent to implement that issue.

Moving an issue to `Ready` approves the refinement, but does not by itself authorize implementation. An implementation request for an issue in any other state is not valid.

After both conditions are met:

1. Move the issue to `In Progress`.
2. Update local `main` without rewriting history:

   ```bash
   git switch main
   git pull --ff-only origin main
   ```

3. Create a dedicated feature branch from the updated `main`.

Branch format:

```text
<type>/issue-<number>-<short-kebab-case-description>
```

Allowed types include `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `ci`, and `hotfix`.

Example:

```bash
git switch -c feat/issue-12-hourly-forecast
```

The agent must not implement work directly on `main` or on a branch shared with another issue.

An explicit implementation request may look like:

```text
Implement the plan for issue #12.
```

## Phase 3: implement and submit for review

During implementation:

1. Implement only the approved execution plan.
2. Add or update tests for the acceptance tests.
3. Update documentation when the behavior or public interface changes.
4. Run the repository verification commands. For Rust, use:

   ```bash
   cargo fmt --all -- --check
   cargo test --all
   cargo clippy --all-targets --all-features -- -D warnings
   ```

5. Verify every acceptance test and Definition of Done item.
6. Replace the `Changelog` placeholder in the issue with a concise user-visible summary.
7. Commit using Conventional Commits. Include the issue number when practical:

   ```text
   feat: add hourly forecast (#12)
   ```

8. Push the feature branch and open a pull request. Never merge it:

    ```bash
    git push --set-upstream origin <branch-name>
    gh pr create --base main --head <branch-name> \
      --title "feat: add hourly forecast" \
      --body-file <generated-pr-body-file>
    ```

9. Move the issue to `In Review` after the pull request has been created.

The user is responsible for reviewing and merging the pull request. The agent must never merge the pull request or close the issue manually.

## Pull request requirements

The pull request must include:

- a concise summary;
- the issue reference, using `Closes #<issue-number>`;
- the execution plan actually followed;
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

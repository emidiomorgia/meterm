# Agent Development Workflow

This repository uses an issue-driven, spec-first workflow. Every implementation must be associated with one GitHub Issue and one dedicated feature branch. Issues move through the following states: `Backlog`, `Ready`, `In Progress`, and `In Review`.

## Operating rules

- Work on one issue at a time.
- Do not modify `main` directly.
- Do not create a branch or write implementation code while refining a `Backlog` issue or before the issue is explicitly approved by moving it to `Ready`.
- Do not merge pull requests.
- Do not close issues manually unless explicitly requested; include `Closes #<issue-number>` in the changelog section of the pull request body.
- Keep the implementation limited to the scope described in the issue.

## Read the workflow state from GitHub Projects

The workflow state is the `Status` field on the issue's GitHub Project item. Do not infer it from the issue's open/closed state, labels, comments, or the issue body. In particular, `Ready` is the valid value; misspellings such as `Redy` are not valid states.

Before creating a branch or editing files for an issue:

1. Resolve the repository owner and name from the `origin` remote, then fetch the complete issue with `gh issue view <number> --json number,state,title,body,comments,url`.
2. Discover the projects owned by the repository owner and inspect their items. The following commands are the reference procedure; `jq` is required:

   ```bash
   OWNER="$(git remote get-url origin | sed -E 's#(git@|https://)github.com[:/]([^/]+)/.*#\2#')"
   REPOSITORY="${OWNER}/$(git remote get-url origin | sed -E 's#(\.git|/)$##; s#.*/##')"
   ISSUE_NUMBER=12

   MATCHES="$(
     for PROJECT_NUMBER in $(gh project list --owner "$OWNER" --format json --limit 100 \
       | jq -r '.projects[].number'); do
       gh project item-list "$PROJECT_NUMBER" --owner "$OWNER" \
         --format json --limit 1000 \
         | jq -c --arg repository "$REPOSITORY" --argjson issue "$ISSUE_NUMBER" \
           --arg project "$PROJECT_NUMBER" \
           '.items[]
            | select(.content.repository == $repository and .content.number == $issue)
            | {project_number: ($project | tonumber), item_id: .id, status: .status}'
     done
   )"
   MATCH_COUNT="$(printf '%s\n' "$MATCHES" | sed '/^$/d' | wc -l | tr -d ' ')"
   test "$MATCH_COUNT" -eq 1 || { echo "Expected exactly one matching project item, found $MATCH_COUNT" >&2; exit 1; }
   PROJECT_NUMBER="$(printf '%s\n' "$MATCHES" | jq -r '.project_number')"
   ITEM_ID="$(printf '%s\n' "$MATCHES" | jq -r '.item_id')"
   STATUS="$(printf '%s\n' "$MATCHES" | jq -r '.status')"
   test "$STATUS" = "Ready" || { echo "Expected status Ready, found $STATUS" >&2; exit 1; }
   ```

3. Continue only when exactly one matching project item is found and its `status` is exactly `Ready`. If no project item is found, more than one item is found, or the status is anything else, stop and report the observed project number, item ID, and status. Do not substitute an issue label or a user assertion for this check.
4. After the issue is verified as `Ready`, move the project item to `In Progress` before implementation. Resolve the project, field, and option IDs from the project API rather than guessing them:

   ```bash
   PROJECT_ID="$(gh project view "$PROJECT_NUMBER" --owner "$OWNER" --format json | jq -r '.id')"
   STATUS_FIELD="$(gh project field-list "$PROJECT_NUMBER" --owner "$OWNER" --format json --limit 100 \
     | jq -r '.fields[] | select(.name == "Status") | .id')"
   IN_PROGRESS_OPTION="$(gh project field-list "$PROJECT_NUMBER" --owner "$OWNER" --format json --limit 100 \
     | jq -r '.fields[] | select(.name == "Status") | .options[] | select(.name == "In Progress") | .id')"
   gh project item-edit --id "$ITEM_ID" --project-id "$PROJECT_ID" \
     --field-id "$STATUS_FIELD" --single-select-option-id "$IN_PROGRESS_OPTION"
   ```

5. After the pull request is created, use the same IDs and procedure to set the item to `In Review`. If the project API is unavailable or returns an authorization error, stop before claiming the transition succeeded and report the exact failing command.

The commands above intentionally inspect Project items directly. `gh issue view --json labels` and the issue REST payload do not contain the Project `Status` field and are insufficient for this workflow gate.

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

1. Verify the issue's Project item using [Read the workflow state from GitHub Projects](#read-the-workflow-state-from-github-projects), then move it to `In Progress`.
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

5. Verify every acceptance test and Definition of Done item against observable evidence. Update the issue body and replace `- [ ]` with `- [x]` only for criteria that are actually verified; leave any unverified criterion unchecked and report it. Do not check a parent section merely because its implementation was attempted.
6. Replace the `Changelog` placeholder in the issue with a concise user-visible summary.
7. Commit using Conventional Commits. Include the issue number when practical:

   ```text
   feat: add hourly forecast (#12)
   ```

8. Push the feature branch and open a pull request whose body contains only the changelog. Include `Closes #<issue-number>` in that changelog when the issue should be closed. Never merge it:

    ```bash
    git push --set-upstream origin <branch-name>
    gh pr create --base main --head <branch-name> \
      --title "feat: add hourly forecast" \
      --body-file <generated-pr-body-file>
    ```

9. Move the issue's Project item to `In Review` after the pull request has been created, using the project-item procedure above. Confirm the resulting status by fetching the item again before reporting completion.

The user is responsible for reviewing and merging the pull request. The agent must never merge the pull request or close the issue manually.

## Pull request requirements

The pull request body must contain only the `Changelog` section from `.github/pull_request_template.md`. It must provide a concise user-visible summary and include `Closes #<issue-number>` when applicable. Acceptance tests, Definition of Done results, execution details, and verification commands belong in the issue or the agent's completion report, not in the pull request body.

If a verification command fails, do not create the pull request until the failure is fixed or explicitly reported to the user.

## GitHub and permissions

- Use `gh issue view <number>` and `gh issue edit <number>` when GitHub CLI access is available.
- Use `gh pr create` to open the pull request only after implementation is complete.
- If authentication, network access, or push permission is unavailable, stop before the external operation and report the exact command that requires user action.
- Never bypass branch protection or merge a pull request.

<!-- BACKLOG.MD MCP GUIDELINES START -->

<CRITICAL_INSTRUCTION>

## BACKLOG WORKFLOW INSTRUCTIONS

This project uses Backlog.md MCP for all task and project management activities.

### Mandatory Workflow Rules

1. Tasks in status `To Do` are created and maintained by humans only.
   - The agent MUST NEVER start working on tasks in `To Do` status.
   - `To Do` tasks are considered not yet approved for implementation.

2. The agent MUST only work on tasks in status `In Progress`.
   - A task in `In Progress` means the human has explicitly approved it for implementation.
   - Tasks in `In Progress` may represent:
     - newly approved work
     - partially completed work
     - tasks returned after human review with requested modifications

3. Before starting implementation of an `In Progress` task, the agent MUST create and switch to a dedicated branch for that task.
   - The branch name MUST follow one of these formats: `feat/task-n`, `hotfix/task-n`, `bugfix/task-n`, or `chore/task-n`.
   - Replace `task-n` with the task identifier, preserving the numeric task number (for example, `feat/task-123`).
   - Choose `feat` for new functionality, `hotfix` for urgent production fixes, `bugfix` for non-urgent defect fixes, and `chore` for maintenance or documentation work.
   - The agent MUST NOT implement the task directly on `main` or another shared base branch.

4. The agent MUST mark work as completed ONLY when:
   - all Acceptance Criteria are fully satisfied
   - all Definition of Done requirements are fully verified

5. All implementation work MUST strictly follow the instructions provided in:
   - `Implementation Plan`
   - `Implementation Notes`

6. At the end of the work, the agent MUST complete the `Final Summary` field as if it were a Pull Request description intended for human review.

7. Once implementation is finished, the agent MUST move the task to status `In Review`.

8. The agent MUST NEVER modify tasks already in status `Done`.

9. The agent MUST NEVER merge a feature branch into the main branch.

10. If the human reviewer rejects or requests changes to the task execution:
   - the task MUST be moved back to `In Progress`
   - the agent MUST continue working on it according to the human review feedback

### Task Preparation and Approval

Before implementing any task in `In Progress`, the agent MUST:

1. Read the task, dependencies, referenced documentation, relevant code, and tests.
2. Draft task-specific `Acceptance Criteria` as measurable and verifiable outcomes.
   - Acceptance Criteria MUST describe what must be achieved, not the implementation steps.
   - Each criterion MUST have a clear pass/fail result.
3. Draft the applicable `Definition of Done` items as verifiable quality and completion requirements.
   - The Definition of Done MUST be adapted to the task type and project context.
   - Requirements that are not applicable to the task MUST NOT be added merely as boilerplate.
4. Draft an ordered `Implementation Plan` describing the intended approach and validation strategy.
5. Present the proposed `Acceptance Criteria`, `Definition of Done`, and `Implementation Plan` to the human for review.
6. Wait for explicit human approval before writing implementation code.
7. Record the approved fields using the Backlog.md CLI.
   - Backlog task Markdown files MUST NOT be edited directly.
   - Use `backlog task edit <task-id> --help` before changing unfamiliar fields.

### Progress Tracking During Implementation

During implementation, the agent MUST:

1. Work in focused implementation and verification loops.
2. Keep the Backlog task as the plan of record.
3. Update the `Implementation Plan` through the Backlog.md CLI before continuing whenever the approved approach materially changes.
4. Record relevant progress, decisions, blockers, and validation results in `Implementation Notes`.
5. Check each Acceptance Criterion as soon as it has been satisfied and verified, using:
   - `backlog task edit <task-id> --check-ac <index>`
6. Check each Definition of Done item only after its requirement has been concretely verified, using:
   - `backlog task edit <task-id> --check-dod <index>`
7. Never check incomplete, assumed, or unverified items.
8. Request human approval before changing the approved scope or Acceptance Criteria.
9. Stop and ask the human whether to extend the current task or create follow-up work if out-of-scope work is discovered.

### Required Field Format

The agent MUST use the following structure when proposing and recording the task fields. The example text is illustrative and MUST be replaced with content derived from the actual task, project, code, and tests.

~~~markdown
## Acceptance Criteria

- [ ] The user can complete the requested action through the intended interface.
- [ ] Valid input produces the expected observable result.
- [ ] Invalid input is rejected with a clear, actionable error message.
- [ ] Existing behavior outside the approved scope remains unchanged.

## Definition of Done

- [ ] All Acceptance Criteria have been satisfied and verified.
- [ ] Relevant automated tests have been added or updated and pass.
- [ ] Relevant project checks, such as build, lint, type checking, or formatting, pass.
- [ ] No known regressions have been introduced in the affected behavior.
- [ ] Documentation and configuration have been updated where required.
- [ ] The Implementation Plan reflects the solution actually implemented.
- [ ] Implementation Notes record relevant decisions and validation results.
- [ ] Final Summary explains what changed, why, and how it was verified.

## Implementation Plan

1. Inspect the existing implementation, related tests, dependencies, and project conventions.
2. Implement the smallest focused change that satisfies the approved Acceptance Criteria.
3. Add or update tests for the expected behavior, validation failures, and relevant edge cases.
4. Run the applicable automated checks and resolve failures caused by the change.
5. Verify every Acceptance Criterion, update Implementation Notes, and prepare the Final Summary.
~~~

Field rules:

- `Acceptance Criteria` MUST contain task-specific, externally observable outcomes. It MUST NOT contain implementation steps, file names, class names, or vague statements such as "works correctly".
- `Definition of Done` MUST contain verifiable quality and completion checks. The agent MUST remove non-applicable example items and add project-specific checks when required.
- `Implementation Plan` MUST be an ordered, task-specific sequence based on inspection of the current codebase. It MUST NOT merely copy the generic example above.
- Every proposed checklist item MUST initially use an unchecked checkbox (`- [ ]`).
- Checked checkboxes (`- [x]`) MUST appear only after the corresponding item has been completed and concretely verified.
- The proposal shown to the human MUST reproduce all three sections in full so their scope and verification approach can be approved together.

### Required Backlog Workflow

- If your client supports MCP resources, read `backlog://workflow/overview` to understand when and how to use Backlog for this project.
- If your client only supports tools or the above request fails, call `backlog.get_backlog_instructions()` to load the tool-oriented overview. Use the `instruction` selector when you need `task-creation`, `task-execution`, or `task-finalization`.

- **First time working here?** Read the overview resource IMMEDIATELY to learn the workflow.
- **Already familiar?** You should have the overview cached (`## Backlog.md Overview (MCP)`).
- **When to read it**: BEFORE creating tasks, or whenever you are unsure whether work should be tracked.

These guides cover:
- Decision framework for when to create tasks
- Search-first workflow to avoid duplicates
- Detailed guides for task creation, execution, and finalization
- MCP tools reference

You MUST read the overview resource to understand the complete workflow. The information is NOT summarized here.

</CRITICAL_INSTRUCTION>

<!-- BACKLOG.MD MCP GUIDELINES END -->

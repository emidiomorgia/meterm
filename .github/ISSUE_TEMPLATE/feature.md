---
name: Feature
about: Propose a feature using the spec-driven development workflow
title: "feat: "
labels: ["type:feature", "status:open"]
assignees: []
---

## Description

<!--
Describe the problem, the motivation, and the expected user or developer outcome.
Keep this section concise but specific enough for implementation planning.
-->

## Acceptance Tests

<!-- The agent must complete this section before implementation starts. -->

- [ ] Given ..., when ..., then ...
- [ ] Given ..., when ..., then ...

## Definition of Done

<!-- The agent must complete this section before implementation starts. -->

- [ ] The approved behavior is implemented.
- [ ] Acceptance tests pass.
- [ ] Automated tests are added or updated.
- [ ] Documentation is updated when required.
- [ ] `cargo fmt --all -- --check` passes.
- [ ] `cargo test --all` passes.
- [ ] `cargo clippy --all-targets --all-features -- -D warnings` passes.
- [ ] The pull request has been reviewed and approved.

## Implementation Plan

<!-- The agent must complete this section and wait for explicit approval. -->

1. TBD

## Changelog

<!-- Keep as TBD during planning. Complete after implementation. -->

TBD

## Approval Gate

- [ ] Implementation plan approved by the maintainer

<!--
The agent must not create a branch, modify source code, commit, push, or open a
pull request until the maintainer explicitly approves the plan.
-->

---
id: TASK-2
title: Create CICD Pipeline ad GitHub Action
status: In Progress
assignee: []
created_date: '2026-07-22 20:33'
updated_date: '2026-07-22 21:57'
labels: []
milestone: m-0
dependencies: []
priority: high
ordinal: 1000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
As a developer, I want the system builds binary artifacts compiling in rust when a TAG is created, in order to have binaries for current version for windows, macos, linux placed in the GitHub Artifacts page.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [x] #1 A GitHub Actions workflow is present under .github/workflows and starts automatically when a Git tag is pushed.
- [x] #2 The workflow builds release binaries for Linux x86_64, macOS x86_64, macOS Apple Silicon, and Windows x86_64.
- [ ] #3 Each target build completes successfully with Cargo and produces the expected executable artifact.
- [ ] #4 The workflow uploads one downloadable GitHub Actions artifact per target, with names identifying the project, tag/version, operating system, and architecture.
- [x] #5 Existing local Rust project behavior remains unchanged and cargo test continues to pass.
<!-- AC:END -->

## Implementation Plan

<!-- SECTION:PLAN:BEGIN -->
1. Inspect the existing tag-triggered matrix workflow and the reviewer comment about missing macOS Intel binaries and GitHub Release assets.
2. Update the target matrix to use currently supported GitHub-hosted runner labels while preserving the four required Rust target triples.
3. Keep one uniquely named Actions artifact per target, then add a single release-publishing job that gathers those artifacts and uploads the executables to the GitHub Release for the pushed tag.
4. Validate workflow syntax and run the applicable local Rust checks.
5. Record the correction and validation results, verify the acceptance criteria and Definition of Done items, and move TASK-2 to In Review only after all required checks pass.
<!-- SECTION:PLAN:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Implemented .github/workflows/release-binaries.yml. The workflow triggers on every pushed tag, uses checkout@v4, dtolnay/rust-toolchain@stable, and upload-artifact@v4, and builds the four requested targets: x86_64-unknown-linux-gnu on ubuntu-latest, x86_64-apple-darwin on macos-13, aarch64-apple-darwin on macos-14, and x86_64-pc-windows-msvc on windows-latest. Unix and Windows staging steps produce target-specific executable filenames, and each matrix job uploads a uniquely named meterm/<tag>/<os>/<architecture> artifact.

Validation: YAML parsed successfully with Ruby Psych; git diff --check passed; cargo fmt --check passed; cargo build --locked passed; cargo test --locked passed (0 tests); cargo build --locked --release --target aarch64-apple-darwin passed locally. The other three target builds and artifact downloads require a pushed branch/tag and a successful GitHub Actions run, which has not occurred in this local workspace.

Follow-up verification attempt: the required GitHub Actions run is still unavailable because feat/task-2 has not been pushed and no tag-triggered workflow run exists. Acceptance Criteria #3/#4 and Definition of Done #1/#4 remain intentionally unchecked until the branch is published and a tag run successfully builds all four targets and exposes the four downloadable artifacts.

Applied comment #2 correction: replaced the retired macos-13 runner with macos-15-intel for x86_64-apple-darwin. Added a publish-release job that waits for all matrix builds, downloads the four uniquely named Actions artifacts with actions/download-artifact@v4, and publishes their executable files to the tag GitHub Release using softprops/action-gh-release@v2 with contents:write permission. This preserves the Actions artifact page entries and adds Release page assets.

Correction validation: YAML parsed successfully with Ruby Psych; cargo fmt --check passed; cargo build --locked passed; cargo test --locked passed (0 tests); cargo build --locked --release --target aarch64-apple-darwin passed; git diff --check passed. A GitHub Actions run is still required to verify the Intel runner and downloadable Release assets remotely.
<!-- SECTION:NOTES:END -->

## Comments

<!-- COMMENTS:BEGIN -->
author: @codex
created: 2026-07-22 21:05
---
Implementation is ready, but final completion is blocked by the required remote GitHub Actions verification. Please push feat/task-2 and trigger a tag run; then verify all four target builds and downloadable artifacts before moving TASK-2 to In Review.
---

created: 2026-07-22 21:54
---
GitHub action did not create the macos x86 binaries anc did not produce artifacts in release page. fix that
---

author: @codex
created: 2026-07-22 21:57
---
Comment #2 addressed: macOS Intel now uses macos-15-intel, and release assets are published by a single post-build job to avoid matrix upload races.
---
<!-- COMMENTS:END -->

## Final Summary

<!-- SECTION:FINAL_SUMMARY:BEGIN -->
Corrected TASK-2 after review comment #2. The macOS Intel build now runs on the supported macos-15-intel runner, and a dedicated publish-release job gathers all four target artifacts and uploads their executables to the GitHub Release for the pushed tag. Local YAML and Rust checks pass; remote GitHub Actions verification remains pending.
<!-- SECTION:FINAL_SUMMARY:END -->

## Definition of Done
<!-- DOD:BEGIN -->
- [ ] #1 All Acceptance Criteria are verified against the workflow definition and a successful GitHub Actions run.
- [x] #2 The workflow uses pinned or explicitly versioned supported actions and a stable Rust toolchain.
- [x] #3 The workflow builds release binaries using the repository Cargo manifest for every required target.
- [ ] #4 Uploaded artifacts are uniquely named and can be downloaded from the GitHub Actions run.
- [x] #5 cargo fmt --check, cargo build, and cargo test pass locally.
- [x] #6 Implementation Notes record workflow decisions and validation results.
- [x] #7 Final Summary describes the change and verification performed.
<!-- DOD:END -->

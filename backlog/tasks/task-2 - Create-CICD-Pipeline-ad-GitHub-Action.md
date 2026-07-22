---
id: TASK-2
title: Create CICD Pipeline ad GitHub Action
status: In Progress
assignee: []
created_date: '2026-07-22 20:33'
updated_date: '2026-07-22 22:32'
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
1. Inspect the existing cross-platform release workflow and reviewer comment #4 specifying distribution formats and installer behavior.
2. Keep the four required Rust target builds and use supported GitHub-hosted runner labels for Intel and Apple Silicon macOS.
3. Package the Linux executable as an executable-bit-preserving tar.gz, package each macOS executable as an unsigned .pkg that installs under /usr/local/bin, and retain the unsigned Windows .exe.
4. Upload one target artifact per matrix job and publish all packaged files to the GitHub Release in a single dependent job.
5. Validate workflow syntax and run the applicable local Rust checks.
6. Record the packaging correction and validation results, verify the acceptance criteria and Definition of Done items, and move TASK-2 to In Review only after all required checks pass.
<!-- SECTION:PLAN:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Implemented .github/workflows/release-binaries.yml. The workflow triggers on every pushed tag, uses checkout@v4, dtolnay/rust-toolchain@stable, and upload-artifact@v4, and builds the four requested targets: x86_64-unknown-linux-gnu on ubuntu-latest, x86_64-apple-darwin on macos-13, aarch64-apple-darwin on macos-14, and x86_64-pc-windows-msvc on windows-latest. Unix and Windows staging steps produce target-specific executable filenames, and each matrix job uploads a uniquely named meterm/<tag>/<os>/<architecture> artifact.

Validation: YAML parsed successfully with Ruby Psych; git diff --check passed; cargo fmt --check passed; cargo build --locked passed; cargo test --locked passed (0 tests); cargo build --locked --release --target aarch64-apple-darwin passed locally. The other three target builds and artifact downloads require a pushed branch/tag and a successful GitHub Actions run, which has not occurred in this local workspace.

Follow-up verification attempt: the required GitHub Actions run is still unavailable because feat/task-2 has not been pushed and no tag-triggered workflow run exists. Acceptance Criteria #3/#4 and Definition of Done #1/#4 remain intentionally unchecked until the branch is published and a tag run successfully builds all four targets and exposes the four downloadable artifacts.

Applied comment #2 correction: replaced the retired macos-13 runner with macos-15-intel for x86_64-apple-darwin. Added a publish-release job that waits for all matrix builds, downloads the four uniquely named Actions artifacts with actions/download-artifact@v4, and publishes their executable files to the tag GitHub Release using softprops/action-gh-release@v2 with contents:write permission. This preserves the Actions artifact page entries and adds Release page assets.

Correction validation: YAML parsed successfully with Ruby Psych; cargo fmt --check passed; cargo build --locked passed; cargo test --locked passed (0 tests); cargo build --locked --release --target aarch64-apple-darwin passed; git diff --check passed. A GitHub Actions run is still required to verify the Intel runner and downloadable Release assets remotely.

Applied comment #4 correction: Linux now produces meterm-<tag>-linux-x86_64.tar.gz containing the executable named meterm with mode 0755. Both macOS targets now produce unsigned meterm-<tag>-macos-<arch>.pkg installers using pkgbuild; the package installs /usr/local/bin/meterm with mode 0755 and therefore requests administrator authorization during installation. Windows continues to publish the unsigned meterm-<tag>-windows-x86_64.exe; Windows security warnings remain the expected behavior for an unsigned Internet-downloaded executable.

Packaging validation: YAML parsed successfully with Ruby Psych; cargo fmt --check, cargo build --locked, cargo test --locked, and the Apple Silicon release build passed. On macOS, pkgbuild successfully created a test unsigned package and tar successfully preserved the executable mode and /usr/local/bin/meterm payload path. Remote GitHub Actions verification is still required.
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

author: @codex
created: 2026-07-22 22:32
---
Comment #4 addressed: Linux tar.gz, unsigned macOS .pkg installers with executable /usr/local/bin/meterm, and unsigned Windows .exe packaging are now defined in the workflow.
---
<!-- COMMENTS:END -->

## Final Summary

<!-- SECTION:FINAL_SUMMARY:BEGIN -->
Corrected TASK-2 for reviewer comment #4. The release workflow now packages Linux as an executable-bit-preserving tar.gz, both macOS architectures as unsigned .pkg installers targeting /usr/local/bin/meterm, and Windows as an unsigned .exe. Actions artifacts remain per-target, and the dependent release job publishes the packaged files to the GitHub Release. Local workflow, Rust, tar, and pkgbuild checks pass; remote CI verification remains pending.
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

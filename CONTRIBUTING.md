# Development workflow

## Branches and commits

- `main`: checked milestones. The initial milestone is documentation only.
- `feat/<feature>`: one independently reviewable feature.
- `fix/<issue>`: a focused repair.
- `docs/<topic>` and `chore/<task>`: documentation and project maintenance.
- `release/<version>`: optional stabilization branch when a release needs it.

Create branches as work starts. Do not create a permanent `develop` branch or several empty feature branches by default. Use Git worktrees when independent changes need separate working directories. Dependent work should branch from the accepted prerequisite or clearly declare a stacked dependency.

Use focused commits such as `feat(media): select and control a specific session` or `fix(lyrics): ignore results from the previous track`.

## Before synchronization

1. Review the diff for scope, local data, credentials, and unintended generated files.
2. Update [CHANGELOG.md](CHANGELOG.md) in the same change, explaining user-visible behavior or the project milestone.
3. Run checks appropriate to the change. Documentation-only changes need link/content/whitespace checks; do not claim a Rust build or real-player test for them.
4. For Rust changes, run the format, lint, test, and release-build commands in [TEST_PLAN.md](docs/TEST_PLAN.md). Commit Cargo.lock for the application.
5. Run live Windows/Apple Music checks for changes that depend on actual player or window behavior. Record skipped cases honestly; unfinished work stays local until its applicable gate passes.
6. Only then commit and push the feature branch, open a PR, inspect the checks on the exact remote commit, and merge passing work. Delete merged short-lived branches unless they are still needed.

The empty repository requires a first checked bootstrap commit before ordinary branch/PR work can begin. This exception does not remove the validation requirement. Local Git write/network restrictions may require synchronization through the connected GitHub API; report that limitation and do not call it a successful local push.

## Remotes

The intended remote configuration is:

```powershell
git remote add origin git@github.com:zhou6140919/dynamic_island_for_windows.git
git remote add upstream https://github.com/Renly1994/xiaotang-daozai.git
```

Run each command only if that remote does not already exist; otherwise inspect `git remote -v` before changing it. An `upstream` remote is a reference source. This independent repository does not share Daozai's history by default; do not merge unrelated histories as a routine update.

Local feature work, once Git metadata is writable:

```powershell
git switch main
git pull --ff-only origin main
git switch -c feat/media-session
```

Keep trust exceptions scoped to this exact repository when necessary. Do not disable Git ownership checks globally or force-push main to resolve environment problems.

## Source imports

Before copying or porting upstream code, record its repository, immutable commit, source file, local destination, license, and modifications in [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md). Retain required notices and full license texts. Check vendored dependencies and per-file exceptions separately.

## Project setup

M1 will create the Rust package, Cargo.lock, pinned toolchain, and Windows CI. It requires Rust for the MSVC target, Visual Studio C++ Build Tools, and a Windows SDK. Do not add arbitrary dependency versions or toolchain pins to this documentation-only bootstrap.

Keep media discovery, source selection, commands, lyric providers, matching, timing, and UI separable. Start with modules in one application crate; split into a workspace only when there is a demonstrated benefit. Add narrow tests for timing, wrong-song matching, stale async results, and platform failures.

## Pull requests and releases

PR descriptions should state the problem, resulting behavior, validation, and remaining limitations. Include source attribution when code is imported. The PR template is [here](.github/pull_request_template.md).

Create `v0.1.0` only after the music acceptance gates pass. Release notes must name supported platforms, Apple Music versions actually checked, known limitations, and the portable build's checksums. Use authored lyric fixtures; do not publish player caches or listening history.

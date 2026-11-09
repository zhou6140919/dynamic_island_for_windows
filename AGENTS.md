# Project instructions

This project is windows灵动岛: a high-quality Windows desktop island. The first accepted product is a music island with reliable Apple Music controls and lyrics. Read README.md and docs/DEVELOPMENT_PLAN.zh-CN.md before implementation.

## Scope and design

- Focus on the current music milestone; defer Pomodoro, calendars, email, and unrelated integrations.
- Use Rust, egui/eframe, and Windows media interfaces unless an evidence-based decision changes the documented plan.
- Keep UI, media session selection/control, lyric acquisition, matching, and timing separable.
- Prefer explicit unavailable states over wrong-song lyrics. Never fabricate timing for plain lyrics.
- Preserve the selected session when sending commands. Reject stale results after track/session changes.
- Perform disk/network work outside rendering frames and enforce bounded work and cancellation.

## Git and verification

- Use a short-lived branch per feature/fix and worktrees when independent work benefits from isolation.
- Update CHANGELOG.md with each meaningful change and record applicable verification.
- Pass relevant checks before synchronizing changes to GitHub. Follow CONTRIBUTING.md and docs/TEST_PLAN.md.
- Distinguish static/unit/mock checks from real Windows/Apple Music acceptance. Never claim checks that were skipped.
- Keep main's milestones checked. Tag releases only after the release gates pass.

## Reuse and data

- Record imported code's immutable source revision, file mapping, modifications, and license in THIRD_PARTY_NOTICES.md; preserve notices.
- Do not upload credentials, tokens, real player caches, lyric content, listening history, or local diagnostic data.
- Use authored fixtures. Keep compatibility reads scoped to required lyric files; do not access account secrets or alter other applications.

## Communication

Explain decisions and progress in plain Chinese for the user. Public-facing README and contribution material can use plain English. Keep the plan and actual implementation status consistent.

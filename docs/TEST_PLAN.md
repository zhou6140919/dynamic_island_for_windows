# Acceptance and test plan

## Current results

Bootstrap only: source inspection and documentation checks. No application source, Rust compilation, playback control, lyric integration, installer, or live Apple Music acceptance has passed in this project yet.

## Test levels

| Level | Environment | Purpose | Milestone |
| --- | --- | --- | --- |
| Pure logic | Rust unit tests with authored fixtures | Song matching, LRC/TTML parsing, timeline, state transitions | M1/M2 onward |
| Provider integration | Local mock responses and temporary directories | Cache indexing, invalid files, HTTP failure, cancellation, stale results | M2 onward |
| Windows build | Windows CI with pinned toolchain and lockfile | Format, lint, build, automatic tests | M1 onward |
| Live acceptance | Windows 11 x64 with Apple Music | Actual metadata, targeted controls, timing, window behavior | M1/M2/M3 |
| Soak and profiling | Local release build | Idle load, playback load, leaks, sleep/resume | M3/M4 |

Live tests are opt-in and must not run in normal CI. They may change playback; identify their target session explicitly. Keep read-only diagnostics separate from control tests. Unit-test success does not substitute for real-player acceptance.

## Required cases

| Area | Scenario | Expected result |
| --- | --- | --- |
| Sessions | No player; player opens, closes, or restarts | Stable idle state, then clean reconnection; no crash |
| Sessions | Apple Music and browser video both exist | Manual choice stays pinned; commands target that session |
| Sessions | Automatic mode, stale/paused sessions, changing system current session | Documented stable selection; playing music preferred; no rapid oscillation |
| Controls | Play/pause, previous, next | Actual player changes; unsupported/failed commands have a visible result |
| Controls | Session disappears during request | Safe failure; never redirect the command to a different player |
| Identity | Same title, other artist; Live/Remix/remaster; duet credits | Reject incompatible/ambiguous candidates; preserve recording qualifiers |
| Identity | Two cached lyrics with similar durations; newest cache belongs to another song | Neither recency nor duration alone permits selection |
| Identity | Corrected or incomplete metadata after track change | Retry boundedly; invalidate incompatible previous results |
| Lyrics | No cache, unreadable cache, partially written JSON, malformed TTML | Graceful fallback or unavailable state; UI stays responsive |
| Lyrics | Oversized files, excessive entries, links/reparse points | Enforced limits; no uncontrolled scan or parse |
| Lyrics | LRC multiple timestamps, offsets, Unicode, malformed timestamps | Correct supported parsing; bounded safe rejection |
| Lyrics | TTML spans, namespaces, empty lines, supplied offsets, unsupported timing | Accurate supported lines; unsupported timing handled explicitly |
| Timing | Pause for 15 seconds, resume | Lyrics stay still while paused and resume at actual playback position |
| Timing | Seek forward/back in the player; single-track repeat | Current line follows the corrected timeline without accumulated drift |
| Timing | Playback-rate change or stale timeline sample | Apply actual rate when usable; re-anchor or degrade safely |
| Races | Rapidly switch A �� B �� C; A's fetch finishes last | Only C's lyrics may be applied |
| Network | Offline, 404, 429, slow response, HTML error body | Correct state and bounded retry; no permanent negative cache for transient failure |
| Network | External lookup disabled | No external lyric or catalog requests; local controls and cache still work |
| Fallback | Plain-only lyrics, instrumental, no result | Distinct states; plain text never presented as synchronized |
| UI | Long/Chinese/English titles; 100%, 150%, 200% scaling | Readable text, correct clipping, usable controls |
| Window | Transparent regions, button clicks, expand/collapse | Correct hit testing; no keyboard-focus theft |
| Window | Mixed-DPI monitors, primary-display change, disconnect | Island remains reachable and correctly sized |
| Lifecycle | Full-screen app, lock/unlock, sleep/resume, tray exit | Documented hide/restore behavior; clean exit and recovery |
| Resources | 10-minute idle baseline, 60-minute playback, repeated track changes | Capture CPU, working set, handles, threads, I/O; investigate sustained growth |

## Release gates

1. `cargo fmt --all -- --check`, `cargo clippy --locked --all-targets -- -D warnings`, `cargo test --locked --all-targets`, and `cargo build --locked --release` pass once the Rust project exists. Add necessary feature coverage when features are introduced.
2. Required Windows 11 live scenarios pass on the release commit; document any intentionally unsupported scenarios as release limitations.
3. Use a small fixed acceptance set of at least 20 tracks across Chinese and English, available native lyrics, absent native lyrics, recording variants, and instrumental tracks. Report native-cache coverage and fallback coverage separately. Offline behavior depends on lyrics actually present locally.
4. Zero wrong-song acceptances in the fixed ambiguity test set. This is a test result, not a universal accuracy guarantee.
5. Proposed response targets: metadata/control feedback within 2 seconds when the player cooperates; supplied line timing generally within 500 ms after calibration on sampled points. Record actual measurements and player/API limitations; revise targets explicitly if evidence requires it.
6. Document resource measurements and fix sustained idle rendering, runaway polling, and growing handles/threads. Set numerical resource budgets after the first measured build, not before measuring.
7. Include change notes, imported-source notices, binary checksums, supported platforms, and known limitations. Test the produced portable package, not only `cargo run`.

## Recording evidence

For each implementation PR, record commands, result, OS/build, Apple Music version, test date, and any skipped cases with their reason. Keep personal metadata and lyric text local. Public test summaries should use anonymous case labels and authored fixtures. A release cannot say ��Apple Music tested�� when only mocks were used.

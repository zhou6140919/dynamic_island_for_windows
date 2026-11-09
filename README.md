# Dynamic Island for Windows

A Windows desktop island, starting with music playback controls and synchronized lyrics for Apple Music.

**Status: planning and repository setup. There is no application build or installer yet.**

## First release

The first usable version will focus on one complete experience:

- Detect Windows media sessions and let the user select a player.
- Show the current title, artist, artwork, and playback state.
- Control play/pause, previous track, and next track for the selected session.
- Show line-synchronized lyrics, with Apple Music for Windows as the first acceptance target.
- Provide a compact top-of-screen island, an expanded music panel, and a tray exit command.

Pomodoro timers, calendars, email, and other integrations are outside the first release.

## Approach

Use Rust, egui/eframe, and Windows media APIs. Selectively adapt relevant code from [Daozai](https://github.com/Renly1994/xiaotang-daozai), preserving attribution and recording the exact upstream revision of every import.

Windows media-session metadata does not contain lyrics. The proposed Apple Music integration will read the player's local timed-lyrics cache when available and confidently matched, with an optional external lookup fallback. That integration has not yet been implemented or tested in this project.

## Project documents

- [Development plan, in Chinese](docs/DEVELOPMENT_PLAN.zh-CN.md)
- [Source review and Apple Music investigation](docs/TECHNICAL_REVIEW.md)
- [Acceptance criteria and test matrix](docs/TEST_PLAN.md)
- [Development and Git workflow](CONTRIBUTING.md)
- [Change history](CHANGELOG.md)
- [Third-party source register](THIRD_PARTY_NOTICES.md)

## Development status

The next task is a small Windows media-session diagnostic, followed by a live Apple Music compatibility check. Rust/Cargo were not available in the inspected execution environment; the MSVC compiler has not been verified. See the plan for prerequisites and milestone gates. No Rust build or application test is claimed by this bootstrap.

## License

Original project material is available under the [MIT License](LICENSE). Imported code, dependencies, fonts, icons, and service content retain their applicable notices and terms. See [third-party notices](THIRD_PARTY_NOTICES.md).

# Source review and Apple Music investigation

Reviewed on 2026-11-09. This is a source review, not a successful application test.

## Fixed references

| Project | Revision inspected | Purpose |
| --- | --- | --- |
| [Renly1994/xiaotang-daozai](https://github.com/Renly1994/xiaotang-daozai) | `39413b3c65d0c91d598507bc6d1ab3be0c6d1105` | Rust island, media sessions, lyrics, native window handling |
| [BeteixZ/floating-lyrics-wam](https://github.com/BeteixZ/floating-lyrics-wam) | `c07b86b1f2fd1da1f510410519a18a510dbd6cd3` | Windows Apple Music cache discovery, TTML parsing, matching |

Reviewing a project is not importing its code. No upstream application source has been copied into this repository in this bootstrap.

## What Daozai actually provides

The inspected [Cargo.toml](https://github.com/Renly1994/xiaotang-daozai/blob/39413b3c65d0c91d598507bc6d1ab3be0c6d1105/Cargo.toml) declares version 0.6.63, Rust edition 2021, egui/eframe 0.31, and windows 0.58. It also patches eframe, egui, imap, and imap-proto through local vendor directories. This is the upstream configuration, not this project's selected dependency lockfile.

| Source | Confirmed behavior | Implication for this project |
| --- | --- | --- |
| [src/smtc.rs](https://github.com/Renly1994/xiaotang-daozai/blob/39413b3c65d0c91d598507bc6d1ab3be0c6d1105/src/smtc.rs) | Enumerates sessions; reads metadata and timeline; targets transport commands to a source identifier | Reuse concepts or isolated code; add deterministic player selection and command-result handling |
| [src/lyrics.rs](https://github.com/Renly1994/xiaotang-daozai/blob/39413b3c65d0c91d598507bc6d1ab3be0c6d1105/src/lyrics.rs) | LRC parsing; native cache handling for supported players; online NetEase, QQ, and LRCLIB matching | Apple Music is eligible for generic online lookup but has no dedicated native-cache branch in this file |
| [src/native.rs](https://github.com/Renly1994/xiaotang-daozai/blob/39413b3c65d0c91d598507bc6d1ab3be0c6d1105/src/native.rs) | Non-activating windows, native window regions, and hit testing | Window behavior requires Windows testing in addition to UI rendering |
| [src/app.rs](https://github.com/Renly1994/xiaotang-daozai/blob/39413b3c65d0c91d598507bc6d1ab3be0c6d1105/src/app.rs) | Large application module integrating several features | Extract music-related behavior rather than importing the entire application |
| [docs/DEPENDENCIES.md](https://github.com/Renly1994/xiaotang-daozai/blob/39413b3c65d0c91d598507bc6d1ab3be0c6d1105/docs/DEPENDENCIES.md) | Documents patched eframe window/context behavior and egui font textures | Evaluate relevant patches before claiming equivalent DPI/focus behavior |

In `lyrics.rs`, `read_native` handles Kugou and Soda Music. `fetch_for_source` has player-specific paths for QQ, Kugou, NetEase, and Soda, then falls back to generic lookup. The generic online path uses title, artist, and duration. Its `fetch_from` LRCLIB path performs search and candidate filtering. There is no Apple Music cache reader in the inspected routing.

This supports adding an Apple Music adapter. It does not establish the cause of any specific user's failure: the currently installed Daozai version, actual media metadata, network responses, and local cache have not been captured.

## Windows interface boundary

Microsoft's [GlobalSystemMediaTransportControlsSessionMediaProperties reference](https://learn.microsoft.com/en-us/uwp/api/windows.media.control.globalsystemmediatransportcontrolssessionmediaproperties?view=winrt-26100) lists title, artist, album, thumbnail, and related metadata, but no lyric content. Playback controls and timeline information belong to the media session and must be checked separately.

Apple's public [MusicKit Song documentation](https://developer.apple.com/documentation/musickit/song) describes `hasLyrics` as an availability flag. That flag is not lyric text. No public lyric-text retrieval endpoint was established in this review; the first implementation must not assume one exists or require an Apple account token to operate the overlay.

## Apple Music cache feasibility

The reference project's [AppleMusicCacheScanner.cs](https://github.com/BeteixZ/floating-lyrics-wam/blob/c07b86b1f2fd1da1f510410519a18a510dbd6cd3/src/AppleMusicLyrics.Infrastructure.Windows/Cache/AppleMusicCacheScanner.cs) locates package directories matching `AppleInc.AppleMusicWin_*`, then searches their `AC/INetCache` directories for `ttmlLyrics*.json`.

Its [TtmlLyricsParser.cs](https://github.com/BeteixZ/floating-lyrics-wam/blob/c07b86b1f2fd1da1f510410519a18a510dbd6cd3/src/AppleMusicLyrics.Core/Parsing/TtmlLyricsParser.cs) reads a JSON `ttml` field and extracts timed paragraphs. It also reads `lyricsId`, document duration, leading-silence metadata, and native lyric-offset metadata. These are observations of that implementation, not an Apple-supported cache contract.

Its [CatalogLyricsId.cs](https://github.com/BeteixZ/floating-lyrics-wam/blob/c07b86b1f2fd1da1f510410519a18a510dbd6cd3/src/AppleMusicLyrics.Core/Matching/CatalogLyricsId.cs) handles identifiers of the form `AP_<catalog song id>`, including some suffix variants. A separate catalog resolver verifies ambiguous candidates. A cache file's ID alone does not identify the currently playing session unless a trustworthy mapping to that song is also available.

Recommended first implementation:

- Read only lyric files in the known player cache roots, with file, depth, count, and parse-size limits; skip links/reparse points and unreadable files.
- Build a bounded index and update it in the background; avoid recursive disk scans and network I/O on rendering frames.
- Parse supported TTML timing into an independent lyric model. Treat unsupported timing expressions or malformed documents as unavailable, not as a crash.
- Preserve supplied timing and offsets; test their sign and interaction against actual playback rather than applying offsets twice.
- Prefer verified catalog identity. Treat duration and title evidence as candidate-discovery signals, not permission to select the latest file.
- Never infer full track duration solely from the last lyric end: instrumental outros can be long.
- If identity remains ambiguous, use a reliable alternative or show no lyrics.

Apple [documents viewing lyrics in its Windows app](https://support.apple.com/guide/music-windows/view-and-enter-lyrics-musf438ffc97/windows), where available. Whether opening that panel is necessary to populate a particular client's cache remains a live-test question.

## External fallback

[LRCLIB's API documentation](https://lrclib.net/docs) describes metadata-based retrieval with plain and synchronized lyric fields. Use title, artist, album, and duration when available; validate returned recording identity and preserve Live/Remix distinctions. Verify the current API contract during implementation, including missing metadata, 404, 429, timeout, and content-type handling.

External lookup must be cancellable and optional. Switching it off must also stop external catalog verification. Plain text must remain explicitly unsynchronized; do not manufacture timing or word highlighting.

## Scope and licensing

Daozai's [LICENSE](https://github.com/Renly1994/xiaotang-daozai/blob/39413b3c65d0c91d598507bc6d1ab3be0c6d1105/LICENSE) is MIT and names its original contributors. Preserve the license and notices when importing substantive code.

The second reference has MIT project-authored code and separately covered LyricsKit-derived provider files; see its [third-party notices](https://github.com/BeteixZ/floating-lyrics-wam/blob/c07b86b1f2fd1da1f510410519a18a510dbd6cd3/THIRD_PARTY_NOTICES.md). Do not treat its entire tree as MIT. The first release does not need to copy its QQ/NetEase provider implementations. Track any actual ports individually.

Software licenses do not grant ownership of returned lyrics or service branding. Keep real lyric caches, account data, and listening metadata out of repository fixtures and public diagnostics.

## What remains unverified

| Item | Current evidence | Next validation |
| --- | --- | --- |
| Apple Music metadata and controls on the user's client | Source/API feasibility only | M1 live diagnostic and targeted control test |
| Cache exists and is readable | Reference implementation only | M2 scoped, read-only cache discovery |
| Current TTML format and offsets | Reference parser only | M2 synthetic fixtures plus local playback comparison |
| Song identity and catalog mapping | Reference strategy only | M2 ambiguous-song and version tests |
| UI/DPI/idle resource behavior | Upstream implementation only | M3 Windows checks and profiling |
| Rust build environment | Rust/Cargo not found in inspected PATH or checked standard location | M1 install/locate toolchain and run build |
| Local Git writes | Initialization succeeded; writing `.git/config` was denied by execution permissions | Validate writable Git metadata before local feature work |

The connected GitHub repository is public and initially empty. Documentation synchronization may use the connected GitHub API when the execution shell cannot access GitHub; that is not evidence that local Git push or application compilation succeeded.

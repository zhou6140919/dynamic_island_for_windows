# Third-party source register

## Current state

This bootstrap includes original project documentation and configuration only. No third-party application code, player cache, font, icon, or lyric text has been imported. The projects below were inspected as references. Inspecting them does not make their licenses the license of this entire repository.

| Reference | Inspected revision | Material reviewed | Import status |
| --- | --- | --- | --- |
| [Renly1994/xiaotang-daozai](https://github.com/Renly1994/xiaotang-daozai) | `39413b3c65d0c91d598507bc6d1ab3be0c6d1105` | Cargo configuration, media, lyrics, windows, dependency notes, MIT license | Reference only |
| [BeteixZ/floating-lyrics-wam](https://github.com/BeteixZ/floating-lyrics-wam) | `c07b86b1f2fd1da1f510410519a18a510dbd6cd3` | Apple Music cache discovery, TTML, identity/matching, third-party notices | Reference only |

## Required import record

For every future substantive copy or port, add:

- Upstream project and immutable commit URL.
- Original file paths and local destination paths.
- Applicable license and copyright notices.
- Modifications and any vendored dependency patches needed.
- Location of the retained full license text and required distribution notices.

Daozai's original code is MIT-licensed with copyright attributed to С�Ƶ�Token and Daozai contributors. Preserve its actual license text when importing covered material.

The Apple Music reference includes separately MPL-2.0-covered LyricsKit-derived provider files. Check its pinned [THIRD_PARTY_NOTICES.md](https://github.com/BeteixZ/floating-lyrics-wam/blob/c07b86b1f2fd1da1f510410519a18a510dbd6cd3/THIRD_PARTY_NOTICES.md) before any port; do not relicense those files as MIT.

## Services and content

Apple Music, Windows, and other service names identify interoperability targets. No affiliation is claimed. Code licenses do not grant rights to service branding or lyric content. Actual lyric files, account data, and personal listening metadata are not repository test fixtures.

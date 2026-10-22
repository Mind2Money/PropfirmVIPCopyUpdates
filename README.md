# PropfirmVIPCopyUpdates

冒险岛VIP Copy（BatchTradovate 桌面版）的公开更新源。应用的「检查更新」只读取本仓库 `main` 分支的 `updates.json`；
每个正式版本在本仓库的 Releases 中附带可执行文件，更新日志见 [CHANGELOG.md](CHANGELOG.md)。

Public update feed for 冒险岛VIP Copy (the BatchTradovate desktop). The application's update check reads
only `updates.json` on the `main` branch; every release here carries the executable, and
[CHANGELOG.md](CHANGELOG.md) lists the notes.

## updates.json

| Field | Meaning |
| --- | --- |
| `product` | Display name of the product |
| `latest` | `null` before the first release, otherwise the newest stable release |
| `latest.version` | Release tag, `vX.Y.Z` |
| `latest.publishedAt` | ISO 8601 publication time |
| `latest.url` | Release page in this repository |
| `latest.notes` | Release notes (Markdown, at most 12000 characters) |
| `latest.executable` | `name`, `url` (release asset in this repository), `bytes`, `sha256` |
| `changelog` | Link to `CHANGELOG.md` |

The file is written by `scripts/release-updates.mjs` of the source repository on every release.

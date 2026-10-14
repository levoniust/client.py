# Fork notes (levoniust/client.py, branch `rumi-t90-pro`)

Personal fork of [DeebotUniverse/client.py](https://github.com/DeebotUniverse/client.py)
to get a **DEEBOT T90 PRO OMNI** (`twunby`, firmware 1.103.0) fully working in Home Assistant
while the fixes wait upstream. Not a general-purpose release; prefer upstream once merged.

## Base
Tag `18.6.0` (the version pinned by Home Assistant 2026.10.1).

## Changes (all taken from open upstream PRs, unmodified)
| Upstream | What | Why |
|---|---|---|
| PR #1860 (eros27sbt) | `twunby` uses `clean_V2`; STOP sends `stop_and_return`; rooms via `freeClean "1,a;1,b"` (`CleanRoomsV2`) | Firmware rejects V1 `clean` (`20003 rcp not support`), ignores `stop`, cleans only the first room of `"1,a,b"` |
| PR #1863 (hezide) | `OnMapInfoV2` accepts any `outlineVer` except `"0"`; Rust parser skips unknown layer types | Robot sends `outlineVer "2"` → no `MapInfoEvent` → `generate_svg()` returns `None` → HA map image HTTP 500 (issue #1762) |

Related upstream PRs doing the map part only: #1870, #1797.

## Verification (2026-10-14)
- Real `getMapInfo_V2` response from the robot (`outlineVer "2"`, zstd-compressed, layers 1/2/6)
  renders a full map (outline, 11 rooms, room borders). Stock 18.6.0 returns `None`.
  The capture is not committed (it is a private floor plan).
- Test suite: 703 passed (Docker-only MQTT tests skipped), run against the released 18.6.0
  native extension; the Rust change of #1863 was not rebuilt locally.
- Cleaning payloads match the ones PR #1860's author verified on firmware 1.103.0.
  Not yet tested on our own robot.

## How it is used
Home Assistant pins `deebot-client==18.6.0` and building this fork needs Rust, so HA does
not install the fork. A small custom integration (`deebot_fixes`) applies the same Python
changes at runtime and turns itself off once HA ships a fixed release. This fork is the
reference copy of the changes.

## Updating
```
git fetch upstream --tags
git checkout -b rumi-t90-pro-<ver> <new tag>
git merge upstream/pull/1860/head upstream/pull/1863/head   # skip any that were merged
```
Drop this branch when both PRs are in a release that HA uses.

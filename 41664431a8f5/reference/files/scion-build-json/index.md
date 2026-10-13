---
title: "scion-build.json"
description: "The description of a player build written next to it - build id, git commit, Unity, backend, defines, ABI, player path, slots and recipe names."
type: file
url: https://jand-s.github.io/scion/41664431a8f5/reference/files/scion-build-json/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/files/scion-build-json.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/grafts/build-modes.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/build.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# scion-build.json

`scion-build.json` sits next to every player Scion builds and says what the build is: its id (a hash of the main
game assembly), the git commit and whether the tree was dirty, the Unity version, the backend and defines, the
Scion version and ABI, the path of the executable, the slots it has and the recipes it contains.

## Location

`<build output folder>/scion-build.json`, beside `Orchard.app` or `Orchard.exe`.

## Written by and read by

Written by `ScionBuild` at the end of a successful build. Read by the CLI (build discovery, `build_stale`,
`build_wrong_mode`), the player (`run.json`), `graft build` (target `build_id`) and `anchor list`.

## Schema

`https://jand-s.github.io/scion/41664431a8f5/schemas/v1/scion-build.schema.json`

## Fields

| Field | Meaning |
|---|---|
| `build_id` | First 12 hex of the SHA-256 of `main_assembly` as built. |
| `main_assembly` | From `ScionSettings.json` (`Assembly-CSharp` by default). |
| `git_commit`, `git_dirty` | The commit and whether files were uncommitted. |
| `unity`, `target`, `backend`, `defines` | Build settings. |
| `scion`, `abi` | Package version and weaver ABI. |
| `player` | The executable, relative to this file. |
| `slots` | Slot ids (test builds). |
| `recipes` | Recipe class names (ids with variants come from `-scion-list`). |
| `built_at`, `seconds` | When and how long. |

## Example

<!-- validate: scion-build -->
```json
{"schema_version":1,"build_id":"4c49a1e0b2f3","main_assembly":"Orchard.Runtime","git_commit":"0ed3af7","git_dirty":false,"unity":"6000.3.25f1","target":"osx","backend":"mono","defines":["SCION_SLOTS","SCION_TRIALS"],"scion":"1.0.0","abi":1,"player":"Orchard.app/Contents/MacOS/Orchard","slots":["A__Orchard_Presentation_HitPresenter__Play__HitEvent","B__Orchard_Presentation_HitPresenter__Play__HitEvent"],"recipes":["CoopTurn","FireballArea","HudStates","KnightFalls","SlashHit","UnitGallery"],"built_at":"2026-10-11T14:30:00Z","seconds":35.2}
```

## Compatibility

Fields may be added in minor versions. The `build_id` rule never changes within 1.x.

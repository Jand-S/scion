---
title: "run.json"
description: "The per-process record a test player writes at start and rewrites at the end - build, machine, arguments, shard, ids, timing, exit code and error."
type: file
url: https://jand-s.github.io/scion/41664431a8f5/reference/files/run-json/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/files/run-json.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/run-folder.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/player-flags.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/exit-codes.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# run.json

`run.json` describes one process of the test player: which build and Scion version, which machine, which arguments,
which shard and role, which ids it selected, when it started and ended, and how it exited. It is written as soon as
the flags are parsed (proving the output folder is writable) and rewritten at the end.

## Location

`<scion-out>/run.json`, for a CLI run `<run>/q<i>/run.json`.

## Written by and read by

Written by the test player. Read by the CLI (exit codes, errors, build checks), agents (on exit 2 or 3) and CI.

## Schema

`https://jand-s.github.io/scion/41664431a8f5/schemas/v1/run.schema.json`

## Fields

| Field | Type | Meaning |
|---|---|---|
| `schema_version` | int | 1. |
| `scion` | string | Package version. |
| `abi` | int | Weaver ABI. |
| `build_id`, `git_commit`, `git_dirty` | string, string, bool | From `scion-build.json` beside the build; null when missing. |
| `unity`, `backend` | string | Unity version; `mono` or `il2cpp`. |
| `platform`, `size` | string, [w, h] | Platform id and capture size. |
| `machine` | object | `os`, `cpu`, `cores`, `ram_mb`. |
| `args` | string[] | The full command line. |
| `shard`, `role`, `rounds`, `run_seed`, `mode`, `mosaic` | | The process's share and settings. |
| `ids` | string[] | Selected ids after filters. |
| `started_at`, `ended_at`, `wall_s` | | Timing; `ended_at` null until the end. |
| `exit_code` | 0-3 or null | The process's exit code. |
| `error` | object or null | `code`, `message`, `hint`, `did_you_mean` on exit 2 or 3. |

## Example

<!-- validate: run -->
```json
{"schema_version":1,"scion":"1.0.0","abi":1,"build_id":"4c49a1e0b2f3","git_commit":"0ed3af7","git_dirty":false,"unity":"6000.3.25f1","backend":"mono","platform":"osx-metal-1280x720-offscreen","size":[1280,720],"machine":{"os":"macOS 26.5","cpu":"Apple M5","cores":10,"ram_mb":16384},"args":["-batchmode","-scion-trials","-scion-out","/Users/alex/orchard/.scion/runs/20261011-143200/q0","-scion-shard","0/6"],"shard":[0,6],"role":"host","rounds":2,"run_seed":1,"mode":null,"mosaic":null,"ids":["CoopTurn","FireballArea","KnightFalls","SlashHit"],"started_at":"2026-10-11T14:32:00Z","ended_at":"2026-10-11T14:32:27Z","wall_s":27.1,"exit_code":0,"error":null}
```

## Compatibility

Fields may be added in minor versions; `schema_version` changes only when a field changes meaning or is removed.

---
title: "control.json and live files"
description: "The connection file a copy writes when its control server listens - pid, port, token, build, ABI, output and role - in its folder and in the live folder."
type: file
url: https://jand-s.github.io/scion/41664431a8f5/reference/files/control-json/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/files/control-json.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/reference/control-protocol.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/warm-copies.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# control.json and live files

A copy started with `-scion-control` writes one small JSON object, twice: `control.json` in its output folder and
`~/.scion/live/<pid>.json` in the user's live folder. It has everything a client needs to connect: the port, the
token and which build it is. The CLI finds warm copies through the live folder.

## Location

| File | Path |
|---|---|
| `control.json` | `<scion-out>/control.json` |
| live file | `~/.scion/live/<pid>.json` (`%USERPROFILE%\.scion\live\` on Windows) |

## Written by and read by

Written by the copy after the control server listens, with mode 0600 on macOS and Linux. The live file is deleted on
`quit`; the CLI deletes live files of dead processes. Read by the CLI (`trial serve`, `trial run --warm`, `graft
inject`) and any client of the [control protocol](/scion/41664431a8f5/reference/control-protocol/).

## Schema

`https://jand-s.github.io/scion/41664431a8f5/schemas/v1/control.schema.json`

## Fields

| Field | Meaning |
|---|---|
| `pid` | The process id. |
| `port` | The local TCP port. |
| `token` | 32 hex digits (128 bits); every request must carry it. |
| `build_id`, `abi` | The build and the weaver ABI. |
| `out` | The copy's output folder. |
| `role` | `host` or `client:<i>`. |
| `started_at` | When it started listening. |

## Example

<!-- validate: control -->
```json
{"schema_version":1,"pid":4242,"port":51234,"token":"9c1f0e7a2b3c4d5e6f708192a3b4c5d6","build_id":"4c49a1e0b2f3","abi":1,"out":"/Users/alex/orchard/.scion/serve/q0","role":"host","started_at":"2026-10-11T14:32:00Z"}
```

## Compatibility

Fields may be added in minor versions. Treat the token as a secret: never log or commit it.

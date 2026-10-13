---
title: "trial serve"
description: "Start, list or stop warm copies of the test build that wait idle on a local control channel for recipes and injections."
type: cli
url: https://jand-s.github.io/scion/41664431a8f5/cli/trial-serve/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/cli/trial-serve.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/warm-copies.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/control-protocol.md
  - https://jand-s.github.io/scion/41664431a8f5/grafts/injection.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# trial serve

`dotnet scion trial serve -j N` starts N warm copies of the test build in the background: each prepares every
adapter and waits idle on a local control channel. `trial run --warm` and `graft inject` use them. Without `-j`, the
command lists the live copies; `--stop` ends them.

## Synopsis

```text
dotnet scion trial serve [-j N] [--stop] [--window] [--build <path>]
```

## Description

Each copy starts with `-scion-trials -scion-serve -scion-control 0 -scion-out .scion/serve/q<i>` (no `-scion-quit`),
writes `control.json` and `~/.scion/live/<pid>.json`, and reaches `idle`. The command returns when all are idle. If
copies of the same build are already live, `-j N` starts only the missing ones.

## Options

| Option | Default | Meaning |
|---|---|---|
| `-j N` | none (list only) | Copies to have running. |
| `--stop` | | Send `quit` to every live copy of this build. |
| `--window` | headless | Windows for the copies. |
| `--build <path>` | discovery | The test build. |

## Output

<!-- validate: cli/trial-serve -->
```json
{"schema_version":1,"command":"trial serve","ok":true,"copies":[{"q":0,"pid":4242,"port":51234,"state":"idle","out":"/Users/alex/orchard/.scion/serve/q0"},{"q":1,"pid":4243,"port":51240,"state":"running","out":"/Users/alex/orchard/.scion/serve/q1"}],"next_step":"dotnet scion trial run --warm"}
```

With `--stop`: `{"stopped": 2}`.

## Exit codes

| Code | When |
|---|---|
| 0 | Copies idle, listed or stopped. |
| 2 | Bad option. |
| 3 | A copy failed to start or prepare. |

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `prepare_failed` | 3 | An adapter's `Prepare` failed in a copy. | Read `.scion/serve/q<i>/player.log`. |
| `build_not_found` | 3 | No test build. | `dotnet scion build test`. |
| warning `build_changed` | 0 | Live copies run an older build. | `trial serve --stop`, then serve again. |
| warning `recycle` | 0 | A copy passed 50 injections and is being restarted. | Nothing; it comes back idle. |

## Examples

### Two warm copies

```sh
dotnet scion trial serve -j 2
```

### Stop them

```sh
dotnet scion trial serve --stop
```

## See also

- [Warm copies](/scion/41664431a8f5/trials/warm-copies/)
- [Control protocol](/scion/41664431a8f5/reference/control-protocol/)
- [Live injection](/scion/41664431a8f5/grafts/injection/)

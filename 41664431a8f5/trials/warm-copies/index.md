---
title: "Warm copies"
description: "Keep prepared copies of the test build running with trial serve, run recipes on them in seconds with trial run --warm, and inject grafts into them."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/trials/warm-copies/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/trials/warm-copies.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/grafts/injection.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/trial-serve.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/control-protocol.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/control-json.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Warm copies

A warm copy is a test build that started, prepared every adapter and waits, idle, for work on a local control
channel. `dotnet scion trial serve -j N` starts N of them; `dotnet scion trial run --warm` sends recipes to them
instead of starting new processes, so a run takes the time of the recipes alone. Warm copies also receive injected
grafts. This guide covers serving, running, stopping and the live files under `~/.scion/live/`.

## When to use

- Use warm copies while iterating on grafts or recipes: a run of a few recipes drops from seconds to under a second.
- Use cold runs (`trial run` without `--warm`) for baselines, CI and anything you approve: a warm run compares only
  hash and frames.

## Steps

### 1. Start them

```sh
dotnet scion trial serve -j 2
```

<!-- validate: cli/trial-serve -->
```json
{"schema_version":1,"command":"trial serve","ok":true,"copies":[{"q":0,"pid":4242,"port":51234,"state":"idle","out":"/Users/alex/orchard/.scion/serve/q0"},{"q":1,"pid":4243,"port":51240,"state":"idle","out":"/Users/alex/orchard/.scion/serve/q1"}],"next_step":"dotnet scion trial run --warm"}
```

The command returns when every copy is `idle`. Copies run in the background with
`-scion-serve -scion-control 0`, and without `-scion-quit`.

### 2. Run recipes on them

```sh
dotnet scion trial run --warm SlashHit FireballArea
```

The CLI splits the selection between idle copies and sends each an op `run`. The output is the same envelope as a
cold run.

### 3. Inject and run again

```sh
dotnet scion graft inject --to all
dotnet scion trial run --warm SlashHit
```

### 4. Stop them

```sh
dotnet scion trial serve --stop
```

## Reference

### What a warm run compares

| Compared | Cold run | Warm run |
|---|---|---|
| State hash | yes | yes |
| Frame count | yes | yes |
| Pixel hashes and images | yes | no (a warm process has rendered other recipes before, so caches and pools differ) |
| Can be approved | yes | no |

### Live files

Each copy writes `control.json` in its output folder and the same object in `~/.scion/live/<pid>.json`, mode 0600,
after the channel listens. The CLI finds copies through the live folder and deletes files of dead processes. The
token in the file authenticates every request ([control.json and live files](/scion/reference/files/control-json/)).

### Lifecycle of a copy

| State | Meaning |
|---|---|
| `preparing` | Startup: flags, discovery, every adapter's `Prepare`. |
| `idle` | Waiting for an op. |
| `running` | Playing a queue (`RunQueue`). An `inject` now answers `pending: true`. |

After 50 injections a copy reports `recycle: true` and `trial serve` replaces it at the next boundary (Mono never
unloads assemblies).

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `no_copies` | 3 | `--warm` found no live copy of this build. | `dotnet scion trial serve -j 2`. |
| `busy` | 3 | Every copy is running a queue. | Wait or start more copies. |
| `build_stale` | warning | The build changed after the copies started. | `trial serve --stop`, then serve again. |

## Examples

### Serve a different build

```sh
dotnet scion trial serve -j 2 --build /tmp/orchard-test/Orchard.app
```

### Watch status from a script

```sh
dotnet scion trial serve --format json | jq '.copies[] | {pid, state}'
```

`trial serve` without `-j` on a running set reports the copies and starts none.

## See also

- [trial serve](/scion/cli/trial-serve/)
- [Live injection](/scion/grafts/injection/)
- [Control protocol](/scion/reference/control-protocol/)
- [control.json and live files](/scion/reference/files/control-json/)

---
title: "From draft to final build"
description: "Take a graft from a draft in scion/grafts through check, injection, approval and promotion into the final build, or loop it with graft watch."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/grafts/lifecycle/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/grafts/lifecycle.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/grafts/injection.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/baselines.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/graft.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# From draft to final build

Every graft goes through the same path: a **draft** in `scion/grafts/` (`grafts.staging`), checked without Unity,
injected or run against the test build, approved through its trials, then **promoted** into the game's grafts
folder (`grafts.promoted`) and compiled into the final build in call mode. This guide walks the path with
`graft new`, `graft check`, `graft inject`, `trial run`, `trial approve`, `graft promote` and the save loop
`graft watch`.

## When to use

- Follow it for every new graft: the draft stage is where agents and people iterate without touching the game.
- Use `graft watch` while tuning: every save rebuilds, injects and replays the recipe.
- Skip the draft stage only for trivial grafts; the trials and the approval still apply.

## Before you begin

- `scion.json` names both folders:

```json
{"schema_version":1,"grafts":{"promoted":"Assets/Orchard/Grafts","staging":"scion/grafts"}}
```

- A test build and warm copies (`dotnet scion build test`, `dotnet scion trial serve -j 2`).

## Steps

### 1. Create the draft

```sh
dotnet scion graft new DamageNumberPop --after Orchard.Presentation.DamageNumbers.Spawn
```

The file `scion/grafts/DamageNumberPop.cs` has the right signature and a body to fill in.

### 2. Check it without Unity

```sh
dotnet scion graft check scion/grafts/DamageNumberPop.cs
```

<!-- validate: cli/graft-check -->
```json
{"schema_version":1,"command":"graft check","ok":true,"files":1,"diagnostics":[],"errors":0,"warnings_count":0,"next_step":"dotnet scion graft inject --to all"}
```

### 3. Try it live

```sh
dotnet scion graft inject --to all
dotnet scion trial run --warm SlashHit FireballArea
```

### 4. Run the cold trials with the draft

```sh
dotnet scion trial run --grafts .scion/grafts/ScionGrafts_51c2.dll
```

The state hash of every recipe must stay the same (grafts are visual); images may change.

### 5. Promote

```sh
dotnet scion graft promote DamageNumberPop
```

<!-- validate: cli/graft-promote -->
```json
{"schema_version":1,"command":"graft promote","ok":true,"name":"DamageNumberPop","from":"scion/grafts/DamageNumberPop.cs","to":"Assets/Orchard/Grafts/DamageNumberPop.cs","staged":true,"next_step":"dotnet scion build test && dotnet scion trial run --changed"}
```

`promote` moves the file (with `git mv` when it is tracked), keeps the namespace and stages it. In slot mode the
weaver records every promoted graft as `[assembly: ScionPromoted(id, graft, priority, wrapper)]`, which `anchor list`
reads.

### 6. Rebuild and approve

```sh
dotnet scion build test
dotnet scion trial run
dotnet scion trial approve --changed --reason "damage numbers pop on spawn"
```

Approving `changed` needs a person at an interactive terminal ([Baselines and approval](/scion/41664431a8f5/trials/baselines/)).

### 7. Ship

```sh
dotnet scion build final
```

The final build calls the promoted graft directly, with no slot and no lab code.

## Reference

### The stages

| Stage | Where | Compiled by | Runs in |
|---|---|---|---|
| Draft | `scion/grafts/` | `graft build` (Roslyn, outside Unity) | Test builds, through injection |
| Promoted | `Assets/Orchard/Grafts/` (with the `.asmref`) | Unity and the weaver | Test builds (slots) and final builds (direct calls) |

### graft watch

```sh
dotnet scion graft watch --recipe SlashHit
```

On every save under `scion/grafts/`: `graft build`, `graft inject --to all`, `trial run --warm SlashHit`. It prints one
JSON line per cycle and stops on Ctrl-C. With `trial loop` running, each injection applies on the next pass of the
loop ([The live loop](/scion/41664431a8f5/explore/live-loop/)).

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `SCN101`-`SCN112` | 1 | `graft check` found a weaver or analyzer error. | Fix the file at `file:line`. |
| `SCN113` | 1 | The draft's target has no slot in this build. | Add an `AnchorScope`, rebuild, inject again. |
| `no_match` | 2 | `graft promote` found no draft with that name. | Use `did_you_mean`. |
| `human_required` | 2 | An agent tried to approve `changed`. | Ask a person to approve at a terminal. |

## Examples

### Check a draft on every edit from an agent hook

```sh
dotnet scion graft check --hook
```

See [Hooks](/scion/41664431a8f5/agents/hooks/).

### Preview a promotion

```sh
dotnet scion graft promote DamageNumberPop --dry-run
```

## See also

- [graft (CLI)](/scion/41664431a8f5/cli/graft/)
- [Live injection](/scion/41664431a8f5/grafts/injection/)
- [Baselines and approval](/scion/41664431a8f5/trials/baselines/)
- [The live loop](/scion/41664431a8f5/explore/live-loop/)

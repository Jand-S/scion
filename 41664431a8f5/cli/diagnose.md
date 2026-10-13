---
title: "diagnose"
description: "Report the adoption level, determinism blockers with file and line, mosaic singletons and a plan, from the weaver's scan of each game assembly."
type: cli
url: https://jand-s.github.io/scion/41664431a8f5/cli/diagnose/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/cli/diagnose.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/determinism/diagnose.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/diagnostics/determinism.md
  - https://jand-s.github.io/scion/41664431a8f5/start/existing-project.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# diagnose

`dotnet scion diagnose` reads the determinism scans the weaver writes when Unity compiles
(`Library/Scion/determinism-<assembly>.json`) and reports the adoption level, counts per SCN301 to SCN307, the
blockers with file and line, the singletons that block the mosaic, an ordered plan and, with `--adapter`, an adapter
skeleton for the game.

## Synopsis

```text
dotnet scion diagnose [--assembly <name>]... [--adapter] [--dry-run]
```

## Description

Blockers are SCN303 to SCN307 findings in code that runs during recipes, and structural problems (no adapter, global
state). The plan orders them by how much they unlock. The full report goes to `Library/Scion/diagnose.md`.

With `--adapter`, `diagnose` writes `Assets/<Game>/Trials/GameAdapter.cs`: a `TrialAdapter<GameRecipe>` and a
`GameRecipe` base, with `Prepare`, `Run`, `Teardown` and `StateHash` stubs naming the game's scenes and the types it
found (`Battle`, `BattleState`).

## Options

| Option | Default | Meaning |
|---|---|---|
| `--assembly <name>` | every scanned assembly | Repeat to scan only some. |
| `--adapter` | off | Write the adapter skeleton. |
| `--dry-run` | off | With `--adapter`, print it instead. |

## Output

<!-- validate: cli/diagnose -->
```json
{"schema_version":1,"command":"diagnose","ok":true,"adoption_level":2,"assemblies":["Orchard.Core","Orchard.Runtime"],"counts":{"SCN301":14,"SCN302":2,"SCN303":1,"SCN304":0,"SCN305":0,"SCN306":0,"SCN307":0},"blockers":[{"file":"Assets/Orchard/Runtime/TurnTimer.cs","line":31,"code":"SCN303","message":"TurnTimer.Tick reads the wall clock (Stopwatch.GetTimestamp), which is not redirected."}],"singletons":[],"plan":["Use Time in TurnTimer.Tick or mark it [WallClock]."],"adapter_skeleton":null,"report":"Library/Scion/diagnose.md","next_step":"fix Assets/Orchard/Runtime/TurnTimer.cs:31, then: dotnet scion trial run --rounds 2 --check"}
```

## Exit codes

| Code | When |
|---|---|
| 0 | Reported (blockers do not fail). |
| 2 | Unknown assembly. |
| 3 | No scan files. |

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `no_match` | 2 | `--assembly` names an assembly without a scan. | `did_you_mean`. |
| `out_not_writable` | 3 | `Library/Scion/` cannot be written. | Check permissions. |
| warning `recipes_unchecked` | 0 | No scan yet. | Open the project in Unity once. |

## Examples

### Write the adapter skeleton

```sh
dotnet scion diagnose --adapter
```

### Only the core

```sh
dotnet scion diagnose --assembly Orchard.Core
```

## See also

- [Find non-determinism](/scion/41664431a8f5/determinism/diagnose/)
- [SCN301-SCN307](/scion/41664431a8f5/reference/diagnostics/determinism/)
- [Add Scion to an existing game](/scion/41664431a8f5/start/existing-project/)

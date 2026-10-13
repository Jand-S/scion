---
title: "Find non-determinism"
description: "Scan the game's assemblies for wall-clock reads, threads, unordered collections and unstable seeds with scion diagnose, and read the adoption level."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/determinism/diagnose/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/determinism/diagnose.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/cli/diagnose.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/diagnostics/determinism.md
  - https://jand-s.github.io/scion/41664431a8f5/start/existing-project.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Find non-determinism

`dotnet scion diagnose` reads the determinism scan the weaver writes for each game assembly
(`Library/Scion/determinism-<assembly>.json`) and turns it into a report: the project's **adoption level** (1 to 3),
the blockers with file and line (SCN303 to SCN307), what test builds already redirect (SCN301, SCN302), the
singletons that block the mosaic, a plan, and an adapter skeleton when there is no adapter yet.

## When to use

- First thing in an existing project: it says what works today and what to fix first.
- When `--check` points at `unscaled`, `rng` or `state` and the cause is not obvious.
- Before turning on the mosaic or the multiverse.

## Steps

### 1. Compile the project once

Open the project in Unity (or build). The weaver writes the scan for every assembly that references
`Scion.Runtime`.

### 2. Run diagnose

```sh
dotnet scion diagnose
```

<!-- validate: cli/diagnose -->
```json
{"schema_version":1,"command":"diagnose","ok":true,"adoption_level":2,"assemblies":["Orchard.Core","Orchard.Runtime"],"counts":{"SCN301":14,"SCN302":2,"SCN303":1,"SCN304":1,"SCN305":2,"SCN306":0,"SCN307":0},"blockers":[{"file":"Assets/Orchard/Runtime/TurnTimer.cs","line":31,"code":"SCN303","message":"TurnTimer.Tick reads the wall clock (Stopwatch.GetTimestamp), which is not redirected."},{"file":"Assets/Orchard/Core/AiPlanner.cs","line":88,"code":"SCN305","message":"AiPlanner.Pick depends on an unordered collection or query (HashSet<Unit>)."}],"singletons":[],"plan":["Use Time in TurnTimer.Tick or mark it [WallClock].","Sort candidates in AiPlanner.Pick before choosing."],"adapter_skeleton":null,"report":"Library/Scion/diagnose.md","next_step":"fix the 2 blockers, then: dotnet scion trial run --rounds 2 --check"}
```

### 3. Fix, rebuild, verify

Fix the blockers in the order of `plan`, then:

```sh
dotnet scion build test
dotnet scion trial run --rounds 2 --check
```

## Reference

### Adoption levels

| Level | Needs | Unlocks |
|---|---|---|
| 1. Grafts | The package. | Grafts, automatic anchors, `graft check`, the project index. |
| 2. Trials | An adapter; recipes give the same hash every round (`hash_stable`). | Parallel recipes, captures, checks, baselines, bisect, co-op comparison, the judge. |
| 3. Multiverse | The rules in an assembly without engine references, registered as an `ISimulation`, and no blockers. | Balance specs, the multiverse, metamorphic tests, the hunter, guided search; the mosaic once `singletons` is empty. |

`doctor` reports the same level with `adoption_next`, the next step.

### Findings

| Id | Severity | Finding |
|---|---|---|
| SCN301 | info | A read redirected in test builds (time, screen). |
| SCN302 | info | A creation replaced in test builds (`Random`, `WaitForSecondsRealtime`). |
| SCN303 | warning | A wall-clock read that is not redirected (`Stopwatch`, `Environment.TickCount`). |
| SCN304 | warning | Threads or tasks in game logic. |
| SCN305 | warning | An unordered collection or query decides an outcome. |
| SCN306 | warning | Randomness seeded from `GetHashCode`. |
| SCN307 | warning | A `Guid` created at run time. |

SCN3xx never appear in the Unity console: the scan can list hundreds of lines, which would bury real errors. They
appear only in the scan file and in `diagnose`. Details: [SCN301-SCN307](/scion/41664431a8f5/reference/diagnostics/determinism/).

### Options

| Option | Meaning |
|---|---|
| `--assembly <name>` | Only one assembly. |
| `--adapter` | Write an adapter skeleton to `Assets/Orchard/Trials/GameAdapter.cs` (`--dry-run` prints it). |

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `config_not_found` | 3 | No `scion.json` above the current folder. | `dotnet scion init`. |
| warning `recipes_unchecked` | 0 | No scan file yet. | Open the project in Unity once. |

## Examples

### Only the core

```sh
dotnet scion diagnose --assembly Orchard.Core
```

### Suppress a reviewed finding

```csharp
#pragma warning disable SCN305 // order does not matter: we sum the set
int total = units.Sum(u => u.Hp);
#pragma warning restore SCN305
```

## See also

- [diagnose](/scion/41664431a8f5/cli/diagnose/)
- [SCN301-SCN307](/scion/41664431a8f5/reference/diagnostics/determinism/)
- [Add Scion to an existing game](/scion/41664431a8f5/start/existing-project/)

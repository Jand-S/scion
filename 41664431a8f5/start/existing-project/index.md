---
title: "Add Scion to an existing game"
description: "Adopt Scion in a game that already has code - level 1 on day one, an adapter for level 2 and a deterministic core for level 3."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/start/existing-project/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/start/existing-project.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/start/install.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/diagnose.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/adapters.md
  - https://jand-s.github.io/scion/41664431a8f5/determinism/verify.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Add Scion to an existing game

This guide adopts Scion in a game that already exists. `dotnet scion init --existing` adds files and never moves
yours; `dotnet scion diagnose` scans your assemblies and reports the adoption level, the blockers with file and line,
and a plan. Level 1 (grafts) works the same day; level 2 costs one adapter; level 3 needs a deterministic rules core.
You prove determinism by running the same recipe twice and comparing hashes.

## When to use

- Use it when the game has code, scenes and its own folder layout that must stay as they are.
- Do not use it for a new, empty project: [`new --genre`](/scion/41664431a8f5/start/new-project/) gives you level 3 directly.

## Before you begin

- The package and the CLI are installed ([Install](/scion/41664431a8f5/start/install/), steps 1 and 2).
- Your game code is in assembly definitions. If it is all in `Assembly-CSharp`, create one `.asmdef` for the game
  code first: grafts compile into an assembly through an `.asmref`, and `Scion.Runtime` is not auto-referenced.

## Steps

### 1. Add Scion's files

```sh
dotnet scion init --existing --dry-run
dotnet scion init --existing
```

`init` asks which assembly is your game (it proposes the one with the most scripts) and writes `scion.json`,
`ProjectSettings/ScionSettings.json`, the grafts folder with its `.asmref`, the `scion/` folders and the ignore
rules. Nothing of yours moves.

### 2. Get the report

```sh
dotnet scion diagnose
```

<!-- validate: cli/diagnose -->
```json
{"schema_version":1,"command":"diagnose","ok":true,"adoption_level":1,"assemblies":["Orchard.Core","Orchard.Runtime"],"counts":{"SCN301":41,"SCN302":3,"SCN303":1,"SCN305":2},"blockers":[{"file":"Assets/Orchard/Runtime/Ai/Planner.cs","line":88,"code":"SCN305","message":"Planner.Pick depends on an unordered collection or query (Dictionary<int, Unit>)."},{"file":"Assets/Orchard/Runtime/Battle/Battlefield.cs","line":20,"code":"singleton","message":"Battlefield.Current is global state; one battle per process."}],"singletons":["Orchard.Runtime.Battlefield.Current","Orchard.Presentation.CameraRig.Instance"],"plan":["Write an adapter: dotnet scion diagnose --adapter writes a skeleton.","Sort the planner candidates by id (SCN305).","Move the battle rules to Orchard.Core with noEngineReferences for level 3."],"adapter_skeleton":null,"report":"/Users/alex/orchard/Library/Scion/diagnose.md","next_step":"write the adapter skeleton: dotnet scion diagnose --adapter"}
```

The full report is in `Library/Scion/diagnose.md`: the level, the blockers worst first, the singletons that would
block the [mosaic](/scion/41664431a8f5/trials/mosaic/), large files that mix rules and presentation, and the plan.

### 3. Level 1: write a graft today

Grafts need nothing but the package. Find an anchor and hang a graft on it:

```sh
dotnet scion anchor list --search hit
dotnet scion graft new HitFlash --after Orchard.Presentation.HitPresenter.Play
```

Then follow [Write a graft](/scion/41664431a8f5/grafts/write/). Without an adapter you verify grafts in the editor or a build;
with one (level 2) you verify them in recipes.

### 4. Level 2: write the adapter

```sh
dotnet scion diagnose --adapter
```

writes `Assets/<Game>/Trials/<Game>Adapter.cs`, a `TrialAdapter<TRecipe>` skeleton with a recipe base class and one
recipe. The real cost of an adapter is knowing three things about your game:

1. **When the scene is ready.** `Prepare` waits for the loaded level; `Run` waits for the setup to settle.
2. **How to reset global state.** `Teardown`, and a `TrialSession.Boundary` handler for statics.
3. **Every random source.** `UnityEngine.Random` and `new System.Random()` are seeded by Scion; any other generator
   (a custom PRNG, a seeded service) must take `TrialSession.RunSeed` or the recipe `Seed`.

A one-scene game without networking needs 30 to 50 lines; a networked tactics game about 300. See
[Write an adapter](/scion/41664431a8f5/trials/adapters/).

### 5. Prove determinism

Run the same recipe several times, in one copy and across copies:

```sh
dotnet scion trial run SlashHit --rounds 4 --check
```

`determinism.hash_stable: true` means every round gave the same state hash. When it is `false`, `determinism.tsv`
names the first frame and the first column that differs (`state`, `tick`, `rng`, `unscaled`, `time`, `px`,
`frames`). See [Verify determinism](/scion/41664431a8f5/determinism/verify/).

### 6. Level 3: register the simulation

Level 3 needs the rules in an assembly without engine references and a command log. Implement
`Scion.Core.ISimulation` over your rules and register it:

```csharp
[assembly: Scion.Core.Simulation(typeof(Orchard.Core.OrchardSimulation))]
```

`doctor` then reports `adoption_level: 3`, and the multiverse, balance specs, metamorphic tests and the rare-moment
hunter work. See [Core API](/scion/41664431a8f5/reference/api/core/).

## Reference

| Level | Proof | Time for a typical game |
|---|---|---|
| 1. Grafts | `graft check` passes and a graft fires in the editor | The same day |
| 2. Trials | A recipe passes 4 rounds with `hash_stable: true` | 1 to 3 sessions (the adapter) |
| 3. Multiverse | `doctor` reports level 3; a captured match replays to the same hash in plain .NET | Depends on how separate rules and presentation already are |

`init --existing` options: `--dry-run`, `--game-assembly <name>` (skip the question), `--grafts-folder <path>`
(default `Assets/<Game>/Grafts`). `diagnose` options: `--adapter` (write the skeleton), `--assembly <name>` (repeat
to scan only some). See [init](/scion/41664431a8f5/cli/init/) and [diagnose](/scion/41664431a8f5/cli/diagnose/).

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `usage` | 2 | `init` without `--existing` in a project that has scripts. | Add `--existing`. |
| `config_invalid` | 3 | An existing `scion.json` has a bad key. | Fix the key named in the message; `init` never overwrites it. |
| `no_simulation` | 3 | A level 3 command found no `[assembly: Simulation]`. | Register an `ISimulation` (step 6). |
| `SCN202`, `SCN203` | 3 | The adapter is not valid, or two adapters handle the same recipe type. | See [Trials diagnostics](/scion/41664431a8f5/reference/diagnostics/trials/). |

## Examples

### Graft-only adoption

A team that only wants grafts stops after step 3. `scion.json` still exists, so `graft check` and the hooks work;
`doctor` reports level 1 with `adoption_next` naming the adapter.

### Find the first non-deterministic system

```sh
dotnet scion trial run KnightFalls --rounds 6 --check --jitter 20
```

`--jitter 20` sleeps up to 20 ms of real time per frame from the second round on, so code that follows the wall clock
shows up as a `diff` line in `determinism.tsv`.

## See also

- [diagnose](/scion/41664431a8f5/cli/diagnose/)
- [Find non-determinism](/scion/41664431a8f5/determinism/diagnose/)
- [Write an adapter](/scion/41664431a8f5/trials/adapters/)
- [The Scion Standard](/scion/41664431a8f5/standard/overview/)

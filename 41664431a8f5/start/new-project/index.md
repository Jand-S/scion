---
title: "Start a new project"
description: "Create a Scion project at adoption level 3 from a genre template with dotnet scion new --genre."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/start/new-project/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/start/new-project.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/start/existing-project.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/init.md
  - https://jand-s.github.io/scion/41664431a8f5/standard/overview.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Start a new project

`dotnet scion new --genre <genre>` turns an empty Unity project into a Scion project that starts at adoption level 3:
assembly definitions in the Scion Standard layers, an adapter and a recipe from the genre sample, an example graft,
a rules core registered as an `ISimulation`, `scion.json`, `ProjectSettings/ScionSettings.json` and the agent block
in `CLAUDE.md` and `AGENTS.md`. The scaffold commands `dotnet scion new feature|effect|graft|trial <Name>` then add
files that compile and pass `doctor` from the first build.

## When to use

- Use it for a new game, or a prototype you want testable from day one.
- Do not use it on a game with existing code: `new` writes a full layout. Use
  [`init --existing`](/scion/41664431a8f5/start/existing-project/) instead; it only adds files.

## Before you begin

- An empty Unity 6000.3 project (created by the Unity Hub) in a git repository.
- The CLI installed in that repository (`dotnet tool install --local Scion.Cli --version 1.0.0`).
- The package added to `Packages/manifest.json` (step 1 of [Install](/scion/41664431a8f5/start/install/)). `new` adds it for you
  when it is missing.

## Steps

### 1. Preview the layout

```sh
dotnet scion new --genre tactics --name Orchard --dry-run
```

<!-- validate: cli/init -->
```json
{"schema_version":1,"command":"new","ok":true,"dry_run":true,"created":["Assets/Orchard/Core/Orchard.Core.asmdef","Assets/Orchard/Runtime/Orchard.Runtime.asmdef","Assets/Orchard/Grafts/Orchard.Grafts.asmref","Assets/Orchard/Trials/Orchard.Trials.asmdef","Assets/Orchard/Trials/OrchardAdapter.cs","Assets/Orchard/Trials/SlashHit.cs",{"more":14}],"changed":["Packages/manifest.json",".gitignore"],"skipped":[],"adoption_level":3,"next_step":"run without --dry-run: dotnet scion new --genre tactics --name Orchard"}
```

A list cut for length ends with one `{"more": N}` object, here 14 more files
([Behavior every command shares](/scion/41664431a8f5/cli/#behavior-every-command-shares)).

### 2. Create it

```sh
dotnet scion new --genre tactics --name Orchard
dotnet scion doctor
```

`doctor` reports `adoption_level: 3` once the first test build exists.

### 3. Build and run the sample recipe

```sh
dotnet scion build test
dotnet scion trial run
dotnet scion trial approve --new --reason "first baselines of the template"
```

### 4. Grow it with scaffolds

Each scaffold writes code that compiles, follows the [Scion Standard](/scion/41664431a8f5/standard/overview/) and comes with its
test:

```sh
dotnet scion new feature Poison        # a rule in Orchard.Core: a RuleModifier, its data row and a completeness test
dotnet scion new effect PoisonTick     # a presentation effect: a named method on the presenter, which becomes an anchor
dotnet scion new graft PoisonGlow --after Orchard.Presentation.UnitView.PlayStatus
dotnet scion new trial PoisonTicks     # a recipe for the effect, with checks
```

`new graft` is the same as [`graft new`](/scion/41664431a8f5/cli/graft/): the draft lands in `scion/grafts/` and comes with a
recipe stub that exercises its anchor.

## Reference

### Genres

| Genre | Sample game | Adapter | Recipes it starts with |
|---|---|---|---|
| `tactics` | Orchard Tactics: grid, turns, 1 to 4 players co-op | `OrchardAdapter : TrialAdapter<OrchardRecipe>` | `SlashHit`, `FireballArea`, `CoopTurn`, `HudStates`, `UnitGallery` ([Orchard sample](/scion/41664431a8f5/reference/orchard-sample/)) |
| `action` | A top-down arena with real-time combat | `ArenaAdapter : TrialAdapter<ArenaRecipe>` | `DashHit`, `WaveClear` |
| `platformer` | A side-scrolling level with a controllable character | `LevelAdapter : TrialAdapter<LevelRecipe>` | `JumpLand`, `CoinPickup` |
| `blank` | No game code; only the layout, an empty adapter and one recipe stub | `GameAdapter : TrialAdapter<GameRecipe>` | `FirstTrial` |

### Layout it creates

```text
Assets/<Name>/
  Core/          <Name>.Core.asmdef      rules: plain C#, integers, noEngineReferences; registers ISimulation
  Runtime/       <Name>.Runtime.asmdef   presentation and input; references Scion.Runtime and Core
  Grafts/        <Name>.Grafts.asmref    compiles into <Name>.Runtime; the promoted grafts
  Trials/        <Name>.Trials.asmdef    recipes and adapter; defineConstraints SCION_TRIALS
Trials/Specs/    <Name>.Specs.csproj     specs, built and run by the CLI in plain .NET
scion/grafts/    draft grafts           scion/recipes/  extra recipes and match files
scion/baselines/ approved baselines     scion.json, ProjectSettings/ScionSettings.json
CLAUDE.md, AGENTS.md                     the Scion block between <!-- scion:begin --> and <!-- scion:end -->
```

Dependencies only point down: Core depends on nothing; Runtime on Core; Grafts compile into Runtime; Trials use
Runtime as a black box and Core to compare hashes. Nothing depends on Grafts. See
[The Scion Standard](/scion/41664431a8f5/standard/overview/).

### Options

| Option | Default | Meaning |
|---|---|---|
| `--genre <genre>` | required | One of the genres above. |
| `--name <Name>` | the Unity product name | Prefix of folders, assemblies and namespaces. |
| `--players <n>` | `4` for `tactics`, `1` otherwise | Maximum players the co-op recipes cover. |
| `--dry-run` | off | List what would be written. |

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `usage` | 2 | `--genre` missing, or `new` in a project that already has `Assets/<Name>/`. | Pick another `--name`, or use `init --existing`. |
| `bad_value` | 2 | Unknown genre. | Use a genre from `did_you_mean`. |
| `usage` | 2 | Not at the root of a Unity project (`ProjectSettings/ProjectVersion.txt` missing). | Run it at the root of the Unity project. |
| `out_not_writable` | 3 | A target file cannot be written. | Fix permissions and run again; `new` is idempotent. |

## Examples

### A two-player platformer

```sh
dotnet scion new --genre platformer --name Hopper --players 2
dotnet scion build test && dotnet scion trial run
```

### Add a status effect end to end

```sh
dotnet scion new feature Burn
dotnet scion new effect BurnTick
dotnet scion new trial BurnTicks
dotnet scion build test && dotnet scion trial run BurnTicks
```

## See also

- [init, new, uninstall](/scion/41664431a8f5/cli/init/)
- [Add Scion to an existing game](/scion/41664431a8f5/start/existing-project/)
- [The Scion Standard](/scion/41664431a8f5/standard/overview/)
- [Enforcement](/scion/41664431a8f5/standard/enforcement/)

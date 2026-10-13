---
title: "Quickstart"
description: "Ten minutes on the Orchard sample - build the test player, run and approve a recipe, write a graft, inject it live and review the change."
type: tutorial
url: https://jand-s.github.io/scion/41664431a8f5/start/quickstart/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/start/quickstart.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/start/install.md
  - https://jand-s.github.io/scion/41664431a8f5/grafts/write.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/recipes.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/baselines.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Quickstart

This tutorial takes about ten minutes on the Orchard Tactics sample. You build the test player with
`dotnet scion build test`, run the `SlashHit` recipe with `dotnet scion trial run`, approve its first baseline,
write the `HitFlash` graft, inject it into a running copy with `graft inject`, promote it, and review the `changed`
verdict it causes. You finish with a second recipe of your own.

## Before you begin

- An empty Unity 6000.3 project (created by the Unity Hub) in a git repository.
- The package and the CLI in that project: steps 1 and 2 of [Install](/scion/41664431a8f5/start/install/). Skip step 3 of Install
  (`init --existing`): `new` below writes `scion.json` and every other file `init` would, and the two are never
  mixed in one project.

Create the Orchard sample and check the setup:

```sh
dotnet scion new --genre tactics --name Orchard
dotnet scion doctor
git add -A && git commit -m "Orchard sample"
```

`new --genre tactics` writes the [Orchard sample](/scion/41664431a8f5/reference/orchard-sample/) (`Assets/Orchard/Core`,
`Runtime`, `Grafts`, `Trials`), a battle scene, the `OrchardAdapter`, its recipes, `scion.json` and the agent block
in `CLAUDE.md` ([Start a new project](/scion/41664431a8f5/start/new-project/)). `doctor` exits 0; its `build` check warns until
step 1.

## Steps

### 1. Build the test player

```sh
dotnet scion build test
```

<!-- validate: cli/build -->
```json
{"schema_version":1,"command":"build test","ok":true,"channel":"batchmode","seconds":31.8,"log":"/Users/alex/orchard/.scion/builds/test/build.log","build_id":"4c49a1e0b2f3","output":"/Users/alex/orchard/.scion/builds/test/Orchard.app","errors":[],"next_step":"run: dotnet scion trial run"}
```

The test build is a Mono player with the `SCION_SLOTS` and `SCION_TRIALS` defines. With the editor open the CLI
builds through it when the project has `com.unity.pipeline`; otherwise close the editor first (`editor_open`).

### 2. Run the sample recipe

```sh
dotnet scion trial run SlashHit --rounds 2
```

<!-- validate: cli/trial-run -->
```json
{"schema_version":1,"command":"trial run","ok":true,"run":"/Users/alex/orchard/.scion/runs/20261011-143200","build_id":"4c49a1e0b2f3","copies":2,"wall_s":6.2,"runs":{"recipes":1,"rounds":2,"total":2,"ok":2,"failed":0,"changed":0,"within":0,"px_changed":0,"new":1,"stale":0},"failed":[],"changed":[],"determinism":{"hash_stable":true,"frame_jitter":[]},"next_step":"check /Users/alex/orchard/.scion/runs/20261011-143200/q0/SlashHit/r1/report.md, then: dotnet scion trial approve --new --reason \"...\""}
```

Both rounds passed and agree on the state hash (`hash_stable: true`). `new: 1` means `SlashHit` has no baseline yet.

### 3. Read the result

```sh
dotnet scion trial show SlashHit
```

The output names `report.md` and the images. Read `report.md` first: the verdict, the checks, the timeline of marks
and the probes. Open `window.png` (12 frames around the impact) only to look at the effect itself.

### 4. Approve the first baseline

```sh
dotnet scion trial approve --new --reason "first baseline of SlashHit"
git commit -m "Baseline for SlashHit"
```

`trial approve` writes `scion/baselines/SlashHit/` and stages it with `git add`; it never commits. Anyone,
including an AI agent, may approve a `new` recipe. The next run of `SlashHit` reports `same`.

### 5. Write a graft

Create a draft graft hung after `HitPresenter.Play`, the method that presents one hit:

```sh
dotnet scion graft new HitFlash --after Orchard.Presentation.HitPresenter.Play
```

Edit `scion/grafts/HitFlash.cs`:

```csharp
using Orchard.Core;
using Orchard.Presentation;
using Scion;
using UnityEngine;

namespace Orchard.Grafts
{
    public static class HitFlash
    {
        // Runs after every HitPresenter.Play, without editing HitPresenter.
        [After(typeof(HitPresenter), nameof(HitPresenter.Play))]
        public static void After(HitPresenter presenter, HitEvent e)
        {
            Fx.Flash(presenter.UnitView(e.Target), Color.white, 0.08f);
        }
    }
}
```

Check it against the build, without Unity:

```sh
dotnet scion graft check scion/grafts/HitFlash.cs
```

A target name with a typo, such as `[After(typeof(HitPresenter), "Ply")]`, comes back as `SCN103` with
`did_you_mean: ["Play"]`. (With `nameof(HitPresenter.Ply)` the C# compiler stops first, with CS0117.)

### 6. Try it live in a running copy

Start one warm copy, compile the graft against the build, inject it and replay the recipe:

```sh
dotnet scion trial serve -j 1 --window
dotnet scion graft build
dotnet scion graft inject
dotnet scion trial run SlashHit --warm
```

The copy keeps running between commands, so each edit-build-inject-run cycle takes about a second. The flash is
visual only: the state hash stays the same as the baseline.

### 7. Promote the graft

```sh
dotnet scion graft promote HitFlash
dotnet scion build test
dotnet scion trial run SlashHit
```

`graft promote` moves the file to `Assets/Orchard/Grafts/HitFlash.cs`, where it compiles into the game. The run now
compares the images with the baseline and finds the flash:

<!-- validate: cli/trial-run -->
```json
{"schema_version":1,"command":"trial run","ok":true,"run":"/Users/alex/orchard/.scion/runs/20261011-150410","build_id":"8d02f1c3a7b9","copies":1,"wall_s":4.1,"runs":{"recipes":1,"rounds":1,"total":1,"ok":1,"failed":0,"changed":1,"within":0,"px_changed":0,"new":0,"stale":0},"failed":[],"changed":[{"id":"SlashHit","vs_approved":"changed","hash":"same","image":"q0/SlashHit/r1/window.png","diff":"diff/SlashHit/r1/window.diff.png"}],"determinism":{"hash_stable":true,"frame_jitter":[]},"next_step":"review: dotnet scion trial show SlashHit --grid; a person approves at a terminal"}
```

Exit code 0 with `ok: true` and `changed[]` filled: `SlashHit` is `changed`. A `changed` verdict is a review signal, not
a failure ([`ok` and `changed`](/scion/41664431a8f5/cli/#ok-and-changed)), so always read `changed[]`. The rules did not change
(`hash: same`); the image did, outside the tolerance.

### 8. Review and approve the change

```sh
dotnet scion trial show SlashHit --grid
```

`--grid` writes `diff/SlashHit/r1/window.grid.png` with three columns: the baseline, the current frames and the
difference. If the flash is what you wanted, a person approves it at an interactive terminal:

```sh
dotnet scion trial approve SlashHit --reason "white hit flash on every hit"
```

An AI agent cannot do this step: without a terminal the CLI stops with `human_required`. That is the point: a
result a person approved is only replaced by a person.

### 9. Write a second recipe

The template ships `SlashHit`, `FireballArea`, `CoopTurn`, `HudStates` and `UnitGallery`. Add the
[Orchard sample](/scion/41664431a8f5/reference/orchard-sample/)'s last one, `Assets/Orchard/Trials/KnightFalls.cs`:

```csharp
using Scion.Trials;

namespace Orchard.Trials
{
    public sealed class KnightFalls : OrchardRecipe
    {
        public KnightFalls()
        {
            Attacker = "skeleton"; Defender = "knight"; DefenderHp = 3;
            Description = "A skeleton finishes a wounded knight.";
            Tags = new[] { "combat", "death" };
        }

        protected override void Checks(TrialChecks c) =>
            c.Order("IMPACT", "unit_down").ProbeIs("defender_hp", hp => hp == 0, "the knight falls");
    }
}
```

`Attacker` and `Defender` are unit ids (`UnitKind.Id`); the other fields keep the defaults of `OrchardRecipe`.

Build, run and approve it as a `new` recipe:

```sh
dotnet scion build test
dotnet scion trial run KnightFalls --rounds 2
dotnet scion trial approve --new --reason "first baseline of KnightFalls"
```

## Check the result

- `dotnet scion trial run` exits 0 and reports `same` for `SlashHit` and `KnightFalls`.
- `scion/baselines/` holds both recipes, committed.
- `Assets/Orchard/Grafts/HitFlash.cs` is in the game; `dotnet scion build final` builds the shipping player with the
  flash as a direct call.

## Next steps

- [Write a graft](/scion/41664431a8f5/grafts/write/): every rule of `[Before]`, `[After]`, priorities and observers.
- [Write a recipe](/scion/41664431a8f5/trials/recipes/) and [Write an adapter](/scion/41664431a8f5/trials/adapters/).
- [Baselines and approval](/scion/41664431a8f5/trials/baselines/): verdicts and who may approve what.
- [Live injection](/scion/41664431a8f5/grafts/injection/): the warm-copy loop in depth.

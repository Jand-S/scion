---
title: "Specs"
description: "Write a TrialSpec that states what a change must achieve - balance numbers, visual gates and judge questions - and run it against variants for one verdict."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/trials/specs/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/trials/specs.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/specs.md
  - https://jand-s.github.io/scion/41664431a8f5/evaluation/gates.md
  - https://jand-s.github.io/scion/41664431a8f5/explore/balance.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/trial-run.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Specs

A spec states, before the work starts, what a change must achieve: balance numbers from the simulator, visual
expectations and gates on recipes, and questions for the judge. It is a `TrialSpec` in the `Scion.Specs` project,
run with `dotnet scion trial run --spec <Name> --variants a,b,c`, and it returns one JSON verdict per variant. Specs
turn "make crits feel stronger" into something an agent can iterate on and a person can trust.

## When to use

- Write a spec before asking an agent for a feel or balance change, so success is defined up front.
- Use it to compare variants (presets, grafts, tunable sets) on the same criteria.
- Do not change a spec after running it to make a variant pass; write a new spec.

## Before you begin

- A specs project referencing `Scion.Specs` (`specs.project` in `scion.json`):

```sh
dotnet new classlib -o Trials/Specs/Orchard.Specs
dotnet add Trials/Specs/Orchard.Specs package Scion.Specs --version 1.0.0
```

## Steps

### 1. Write the spec

```csharp
using Scion.Specs;

namespace Orchard.Specs
{
    public sealed class StrongerCrits : TrialSpec
    {
        public override string Name => "StrongerCrits";
        public override string Target => "Critical hits read clearly as stronger without slowing the turn.";

        public override void Define(SpecBuilder s)
        {
            s.Balance("Duel", players: 1, matches: 500)
             .Expect("win_rate", min: 0.45, max: 0.55)
             .Expect("turns_mean", min: 6, max: 9);

            s.Visual("SlashHit")
             .Expect("hit_stop_ms", min: 50, max: 120)
             .Expect("shake_peak_px", min: 4, max: 12)
             .Gate(Gates.Photosensitivity)
             .Gate(Gates.TextContrast);

            s.Judge("SlashHit", head: "hit_weight").Expect("score", min: 0.6);
        }
    }
}
```

### 2. Run it on variants

```sh
dotnet scion trial run --spec StrongerCrits --variants base,heavy,snappy
```

<!-- validate: cli/trial-run -->
```json
{"schema_version":1,"command":"trial run","ok":true,"run":"/Users/alex/orchard/.scion/runs/20261012-091500","build_id":"4c49a1e0b2f3","copies":6,"wall_s":48.3,"runs":{"recipes":1,"rounds":1,"total":3,"ok":3,"failed":0,"changed":0,"px_changed":0,"new":0,"stale":0},"failed":[],"changed":[],"determinism":{"hash_stable":true,"frame_jitter":[]},"spec":{"name":"StrongerCrits","ok":true,"variants":[{"name":"base","ok":false,"failed":["hit_stop_ms 40 < 50"]},{"name":"heavy","ok":true,"failed":[]},{"name":"snappy","ok":true,"failed":[]}],"verdict":"heavy and snappy pass; base fails hit_stop_ms"},"next_step":"compare heavy and snappy: dotnet scion trial show SlashHit --grid"}
```

### 3. Let a person pick

The spec cuts the variants that fail; it does not choose between those that pass. Look at the grid or ask the judge
for a duel, then approve.

## Reference

### SpecBuilder

| Call | Runs | Measures |
|---|---|---|
| `Balance(name, players, matches)` | The rules simulator in .NET, with bots, `matches` times | `ISimulation.Metrics`: `win_rate`, `turns_mean` and the game's own |
| `Visual(recipe)` | The recipe in the test build, with `--motion` | The motion metrics (`hit_stop_ms`, `shake_peak_px`, `flash_ms`...) and probes |
| `Judge(recipe, head)` | The judge head on the recipe's images | `score`, `probabilities` |
| `Metamorphic(property)` | A property on the core ([Metamorphic tests](/scion/explore/metamorphic/)) | pass or fail |
| `Curve(name, levels)` | A balance curve over levels ([Balance with the simulator](/scion/explore/balance/)) | per-level metrics |

`Expect(metric, min, max)` adds a bound (either may be omitted). `Gate(id)` enables a gate on the recipe.

### Variants

`--variants a,b,c` names presets in `scion/presets/`, applied one per variant to every part of the spec. The
`base` preset is always the initial values.

### Guards

- A spec is frozen once it ran: the CLI records its source hash in the run, and a later run of a changed spec with
  the same name warns.
- Every spec run includes the `base` variant as a control.
- When a variant fails, `next_step` asks for a person to look at a sample of the failures, not just the numbers.

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `no_match` | 2 | No spec with that name in `specs.project`. | Use `did_you_mean`. |
| `no_simulation` | 3 | `Balance` needs an `ISimulation` and the core declares none. | Implement `ISimulation` ([Core API](/scion/reference/api/core/)). |
| `model_missing` | 3 | `Judge` needs a head that is not trained. | `dotnet scion judge list`; train the head. |
| spec `ok: false` | 1 | No variant passes. | Read `spec.variants[].failed`. |

## Examples

### A balance-only spec

```csharp
public override void Define(SpecBuilder s) =>
    s.Balance("CoopFloor3", players: 4, matches: 1000).Expect("win_rate", min: 0.6, max: 0.75);
```

### Run a spec in CI

```sh
dotnet scion trial run --spec StrongerCrits --variants base
```

## See also

- [Specs API](/scion/reference/api/specs/)
- [Gates](/scion/evaluation/gates/)
- [Balance with the simulator](/scion/explore/balance/)
- [trial run](/scion/cli/trial-run/)

---
title: "Tune a hit effect"
description: "Tune a critical hit's feel end to end - live loop, draft graft, motion metrics, presets, a judge duel, promotion and approval."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/cookbook/tune-hit-feel/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/cookbook/tune-hit-feel.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/explore/live-loop.md
  - https://jand-s.github.io/scion/41664431a8f5/evaluation/motion-pack.md
  - https://jand-s.github.io/scion/41664431a8f5/judge/presets.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Tune a hit effect

This recipe tunes Orchard's critical hit from "it works" to "it feels right" in one sitting. You loop the moment in
the test build, edit a draft graft with `graft watch` injecting every save, read the motion metrics, save the result
as presets, compare them blind with a judge duel, then promote the graft and approve the baselines.

## When to use

- Any feel work on one moment: a hit, a death, a pickup, a turn change.

## Before you begin

- A captured match of the moment (`knight-dies-to-crit`), or a recipe (`SlashHit`).
- Warm copies are not needed: the loop runs its own copy.

## Steps

### 1. Loop the moment

```sh
dotnet scion trial loop "CapturedMatch[match=knight-dies-to-crit]" --span 3..3 --window
```

### 2. Draft the graft and watch

```sh
dotnet scion graft new CritShake --after Orchard.Presentation.HitPresenter.Play
dotnet scion graft watch
```

Edit `scion/grafts/CritShake.cs`; every save shows up on the next pass.

### 3. Read the numbers

```sh
dotnet scion trial run SlashHit --grafts .scion/grafts/ScionGrafts_51c2.dll --motion
```

`metrics.json` gives hit-stop, shake peak and decay, flash length. Aim for the bounds of your spec (for example
hit-stop 50 to 120 ms, shake peak 4 to 12 px).

### 4. Save two feels as presets

`scion/presets/heavy.json` and `scion/presets/snappy.json` with the tunables you settled on.

### 5. Duel them

```sh
dotnet scion trial run SlashHit --grafts .scion/grafts/ScionGrafts_51c2.dll --variants heavy,snappy --motion
dotnet scion judge run <run> --heads hit_weight
dotnet scion judge vote --run <run> --head hit_weight
```

### 6. Promote and approve

```sh
dotnet scion graft promote CritShake
dotnet scion build test
dotnet scion trial run --rounds 2
dotnet scion trial approve --changed --reason "crit shake and hit-stop"
```

Approving `changed` needs a person at a terminal.

## Reference

| Step | Tool | Time |
|---|---|---|
| Loop and edit | `trial loop`, `graft watch` | Seconds per change |
| Measure | `--motion`, `metrics.json` | One run |
| Compare | presets, `judge run`, `judge vote` | Minutes |
| Ship | `graft promote`, `build test`, `trial approve` | One build |

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| loop stopped by `error` | 1 | The graft changed the hash. | It touches rules; keep it visual. |
| `SCN113` | 1 | No slot for `HitPresenter.Play`. | Add `AnchorScope(typeof(HitPresenter))`, rebuild. |

## Examples

### A blend between the two feels

```sh
dotnet scion trial run SlashHit --preset "base + 0.5 heavy + 0.5 snappy" --motion
```

### Check photosensitivity before approving

```csharp
protected override void Checks(TrialChecks c) => c.Gate(Gates.Photosensitivity);
```

## See also

- [The live loop](/scion/41664431a8f5/explore/live-loop/)
- [The motion pack](/scion/41664431a8f5/evaluation/motion-pack/)
- [Taste heads and feel presets](/scion/41664431a8f5/judge/presets/)
- [From draft to final build](/scion/41664431a8f5/grafts/lifecycle/)

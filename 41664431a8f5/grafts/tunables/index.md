---
title: "Tunables and presets"
description: "Declare tunable fields with ranges, override them per recipe and per run, save feel presets, blend them, and keep rule tunables in the lobby hash."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/grafts/tunables/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/grafts/tunables.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/core.md
  - https://jand-s.github.io/scion/41664431a8f5/judge/presets.md
  - https://jand-s.github.io/scion/41664431a8f5/explore/guided-search.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/gen.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Tunables and presets

A tunable is a static numeric field marked `[Tunable(min, max)]`: a number someone will want to tune, such as a shake
strength, a hit-stop length or a damage multiplier. Scion lists every tunable (`dotnet scion gen --list tunables`),
lets recipes and runs override them (`tunable:CritShake.Strength`), saves sets of values as **feel presets** in
`scion/presets/`, blends presets, and adds the values of rule tunables to the lobby hash so co-op peers agree.
SCN005 reports a numeric tunable without a range.

## When to use

- Use `[Tunable]` on every number in a graft or the core that someone will tune by eye or by search.
- Use presets to keep named feels (`heavy`, `snappy`) and compare them in trials and judge duels.
- Use guided search when the space is too big to tune by hand ([Guided search](/scion/explore/guided-search/)).

## Steps

### 1. Declare the tunable

```csharp
public static class CritShake
{
    [Tunable(0f, 1f, Doc = "Camera shake on a critical hit.")]
    public static float Strength = 0.3f;

    [Tunable(0f, 0.2f, Doc = "Hit-stop on a critical hit, in seconds.")]
    public static float Stop = 0.06f;
}
```

A tunable is a `static` field of type `int`, `float`, `double` or `bool`. Numeric tunables need a range; `bool`
tunables take none.

### 2. Override it in a recipe

```csharp
public sealed class SlashHitFeel : SlashHit
{
    protected override IEnumerable<TrialArgs> Variants()
    {
        yield return new TrialArgs { { "tunable:CritShake.Strength", 0.3f } };
        yield return new TrialArgs { { "tunable:CritShake.Strength", 0.8f }, { "tunable:CritShake.Stop", 0.1f } };
    }
}
```

### 3. Override it for one run

```sh
dotnet scion trial run SlashHit --set tunable:CritShake.Strength=0.8
```

### 4. Save and use a preset

`scion/presets/heavy.json`:

<!-- validate: preset -->
```json
{"schema_version":1,"name":"heavy","description":"Slow, weighty hits.","tunables":{"CritShake.Strength":0.8,"CritShake.Stop":0.1,"HitFlash.Seconds":0.12}}
```

```sh
dotnet scion trial run SlashHit --preset heavy
dotnet scion trial run SlashHit --preset "base + 0.7 heavy + 0.3 snappy"
```

## Reference

### Names

A tunable's name is `<Type>.<Field>` with the type's simple name (`CritShake.Strength`). When two types share a
simple name, use the full name (`Orchard.Grafts.CritShake.Strength`). In recipe arguments and `--set`, prefix it with
`tunable:`.

### Order of overrides

From weakest to strongest: the field's initial value, the preset, the recipe's arguments, `--set`. A value outside
the range is clamped and reported as a warning in `result.json`.

### Blends

`base + 0.7 heavy + 0.3 snappy` computes, for each tunable, `base + 0.7 × (heavy − base) + 0.3 × (snappy − base)`,
clamped to the range. `base` is the initial values. `bool` tunables take the value of the heaviest preset that sets
them. Weights are decimals with a dot.

### Rule tunables

Tunables declared in a rule assembly (`analyzers.rule_assemblies`) change rules, so every peer must have the same
values. Their values enter `Tunables.LobbyHash`, which the game compares when a client joins; a host and a client
with different rule tunables refuse to play. Presentation tunables (in grafts) stay local to each machine.

### Listing

```sh
dotnet scion gen --list tunables
```

prints every tunable with its type, range, default and doc, and `scion gen` writes them into
`scion.manifest.json` ([gen](/scion/cli/gen/)).

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `SCN005` | compile warning | A numeric `[Tunable]` without a range. | `[Tunable(min, max)]`. |
| `bad_value` | 2 | `--set` names an unknown tunable or a value of the wrong type. | Use `gen --list tunables`; read `did_you_mean`. |
| `no_match` | 2 | `--preset` names a file that does not exist in `scion/presets/`. | List `scion/presets/`. |

## Examples

### A rule tunable in the core

```csharp
namespace Orchard.Core
{
    public static class Balance
    {
        [Tunable(500, 2000, Doc = "Critical damage multiplier, in permille.")]
        public static int CritPermille = 1500;
    }
}
```

### Compare three feels side by side

```sh
dotnet scion trial run SlashHit --variants base,heavy,snappy --mosaic 3
```

## See also

- [Core API](/scion/reference/api/core/)
- [Taste heads and feel presets](/scion/judge/presets/)
- [Guided search](/scion/explore/guided-search/)
- [gen](/scion/cli/gen/)

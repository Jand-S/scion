---
title: "Variants, modes and seeds"
description: "Expand one recipe into many ids with Vary fields, Variants, capture modes and seed ranges, and select them with patterns, tags and globs."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/trials/variants/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/trials/variants.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/recipes.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/trial-list.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/recipes.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Variants, modes and seeds

One recipe class can stand for many situations. `[Vary]` fields and `Variants()` expand it into **variants**
(`SlashHit[Attacker=mage,Distance=3]`); `[assembly: TrialModes]` adds **modes** such as resolutions (`@360p`); a seed
range repeats the selection per seed (`#7`). Each combination is an **id** with its own folder, baseline and verdict.
This guide covers the id grammar, `TrialArgs`, `Setup()`, modes, tags, patterns and `trial list --ids`.

## When to use

- Use `Variants()` to vary members the recipe inherits (the fields of `OrchardRecipe`), when combinations are not a
  full grid, or when values are computed.
- Use `[Vary]` for a small grid over fields the recipe class declares itself.
- Use modes for capture settings that apply to every recipe (resolutions, a color-blind filter).
- Use seed ranges to search for rare failures, not for baselines: each seed is its own id.

## Steps

### 1. Yield the variants

This page turns the [Orchard sample](/scion/41664431a8f5/reference/orchard-sample/)'s `SlashHit` into a grid of three attackers
and two distances. `Attacker` and `Distance` are fields of the base class `OrchardRecipe`, which is what the adapter
reads, so the recipe sets them by name with `TrialArgs`:

```csharp
public sealed class SlashHit : OrchardRecipe
{
    protected override IEnumerable<TrialArgs> Variants()
    {
        foreach (var attacker in new[] { "knight", "mage", "rogue" })
            foreach (var distance in new[] { 1, 3 })
                yield return new TrialArgs { { "Attacker", attacker }, { "Distance", distance } };
    }

    protected override void Setup()
    {
        Ability = Attacker == "mage" ? "fireball" : "slash";   // after the args are applied
    }
}
```

This gives 6 ids: `SlashHit[Attacker=knight,Distance=1]` through `SlashHit[Attacker=rogue,Distance=3]`. Keys are
field or property names, looked up on the recipe class and then on its base classes, so an inherited field is set
where the adapter reads it. A key can also be `tunable:<Type>.<Field>`
([Tunables and presets](/scion/41664431a8f5/grafts/tunables/)):

```csharp
yield return new TrialArgs { { "Attacker", "mage" }, { "Distance", 4 }, { "tunable:CritShake.Strength", 0.8f } };
```

### 2. Or vary the recipe's own fields

`[Vary]` marks a field or property that the recipe class declares itself. Map it onto the base fields in `Setup()`:

```csharp
public sealed class SlashRange : OrchardRecipe
{
    [Vary("near", "far")] public string Range = "near";

    protected override void Setup() => Distance = Range == "far" ? 3 : 1;
}
```

This gives `SlashRange[Range=near]` and `SlashRange[Range=far]`. `Variants()` replaces the `[Vary]` grid when it
returns non-null.

:::caution[Never hide a base field]
`[Vary] public new string Attacker` declares a second field. The adapter reads `OrchardRecipe.Attacker`, so every
variant would play the same fight under a different id. The session fails such a recipe with
`[Vary] hides OrchardRecipe.Attacker: use Variants()`.
:::

### 3. Declare modes

```csharp
[assembly: TrialModes("360p=640x360", "720p=1280x720", "deuter")]
```

A mode `name=WxH` sets the capture size. Any other mode, declared in `TrialModes` or not, keeps the default size and
is passed to every adapter that implements `ITrialModeAware`:

```csharp
public sealed class OrchardAdapter : TrialAdapter<OrchardRecipe>, ITrialModeAware
{
    public void ApplyMode(string mode) => ColorFilter.Set(mode == "deuter" ? Filter.Deuteranopia : Filter.None);
    // ...
}
```

### 4. Select

```sh
dotnet scion trial list --ids
dotnet scion trial run "SlashHit[Attacker=mage,*]" --mode 360p
dotnet scion trial run --tag combat --seeds 1-20
```

## Reference

### Id grammar

```text
<Name>[<key>=<value>,...]@<mode>#<seed>
```

| Part | Present when | Example |
|---|---|---|
| `Name` | always | `SlashHit` |
| `[k=v,...]` | the recipe has variants; keys in declaration order | `[Attacker=mage,Distance=3]` |
| `@mode` | a mode is selected | `@360p` |
| `#seed` | a seed range is selected | `#7` |

Values print with invariant culture (`0.8`, `true`). The folder name replaces any character outside
`[A-Za-z0-9_.,=@#[]-]` with `_`.

### Patterns

| Pattern | Matches |
|---|---|
| `SlashHit` | The `Name`: every variant of the recipe. |
| `SlashHit[Attacker=mage,Distance=3]` | One exact id. |
| `SlashHit[Attacker=mage,*]` | A glob over ids: `*` any text, `?` one character. |
| `--tag combat,ui` | Recipes with any of the tags. |
| `--players 4` | Recipes with `Players == 4`. |

Several patterns are separated by commas or given as separate arguments. A pattern that matches nothing exits 2
with `no_match` and `did_you_mean`. `Explicit` recipes run only when a pattern names their exact `Name` or `Id`.

### Player flags

| Flag | CLI | Meaning |
|---|---|---|
| `-scion-only <patterns>` | positional patterns | Select ids. |
| `-scion-tag <tags>` | `--tag` | Select by tag. |
| `-scion-mode <name>` | `--mode` | One mode for the whole run. |
| `-scion-seeds <a-b>` | `--seeds` | Repeat the selection per seed; ids get `#<seed>`. |
| `-scion-list` | `trial list --ids` | Write `recipes.json` with the expanded ids and exit 0. |

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `no_match` | 2 | No id matches. | Use `did_you_mean` or `trial list --ids`. |
| `bad_value` | 2 | A `[Vary]` or `TrialArgs` value does not convert to the member's type. | Fix the value. |
| recipe error `unknown variant key <k>` | 1 | `TrialArgs` names a member neither the recipe nor its base classes have. | Use the field name. |
| recipe error `[Vary] hides <Base>.<member>` | 1 | A `[Vary]` field hides an inherited one with `new`. | Remove the field; set the base member from `Variants()`. |

## Examples

### Every attacker at 360p and 720p

```sh
dotnet scion trial run "SlashHit[*]" --mode 360p
dotnet scion trial run "SlashHit[*]" --mode 720p
```

### Hunt a rare failure over 200 seeds

```sh
dotnet scion trial run KnightFalls --seeds 1-200 -j 8
```

## See also

- [Write a recipe](/scion/41664431a8f5/trials/recipes/)
- [trial list](/scion/41664431a8f5/cli/trial-list/)
- [recipes.json](/scion/41664431a8f5/reference/files/recipes-json/)
- [Recipes API](/scion/41664431a8f5/reference/api/recipes/)

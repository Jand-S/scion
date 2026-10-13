---
title: "Recipes API"
description: "TrialRecipe with its data, overrides and masks, the Vary attribute, TrialArgs, TrialModes, ITrialModeAware and the CapturedMatch recipe."
type: api
url: https://jand-s.github.io/scion/41664431a8f5/reference/api/recipes/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/api/recipes.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/recipes.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/variants.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/captured-matches.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Recipes API

A recipe is a `TrialRecipe` subclass created through its public parameterless constructor. Variants come from
`[Vary]` fields or `Variants()`; modes from `[assembly: TrialModes]`. All types are in `Scion.Trials` under
`SCION_TRIALS`.

## TrialRecipe

**Namespace:** `Scion.Trials` · **Assembly:** `Scion.Trials` · **Define:** `SCION_TRIALS`

```csharp
public abstract class TrialRecipe
{
    public virtual string Name { get; }                       // class name
    public string Id { get; }
    public string Variant { get; }
    public string Mode { get; }
    public IReadOnlyList<string> Tags { get; protected set; }
    public string Description { get; protected set; }
    public int Seed { get; protected set; } = 1;
    public int Fps { get; protected set; } = 30;
    public float TailSeconds { get; protected set; } = 0.5f;
    public float TimeoutSeconds { get; protected set; } = 30f;
    public int Players { get; protected set; } = 1;
    public bool Explicit { get; protected set; }
    public DeterminismControl Control { get; protected set; } = DeterminismControl.All;
    public ImageTolerance Tolerance { get; protected set; } = ImageTolerance.Default;
    protected virtual void Checks(TrialChecks c) { }
    protected virtual void Setup() { }
    protected virtual IEnumerable<TrialArgs> Variants() => null;
    protected void Mask(string artifact, Rect normalized, string why);
}
```

| Member | Meaning | Default |
|---|---|---|
| `Id`, `Variant`, `Mode` | Set by the session when it expands the recipe. | |
| `Players` | 1 to 4. | 1 |
| `Explicit` | Runs only when named. | false |
| `Tolerance` | `null` compares no images. | `ImageTolerance.Default` |
| `Mask` | `artifact` is a file name or `*`; `normalized` has the origin at the bottom left. | |

## VaryAttribute

**Namespace:** `Scion.Trials` · **Assembly:** `Scion.Trials` · **Define:** `SCION_TRIALS`

```csharp
[AttributeUsage(AttributeTargets.Field | AttributeTargets.Property)]
public sealed class VaryAttribute : Attribute
{
    public VaryAttribute(params object[] values);
}
```

Every combination of `[Vary]` values is a variant, in declaration order. `[Vary]` applies to members the recipe class
declares itself; a `[Vary]` field that hides an inherited member with `new` fails the recipe. Vary inherited members
from `Variants()` ([Variants, modes and seeds](/scion/41664431a8f5/trials/variants/)).

## TrialArgs

**Namespace:** `Scion.Trials` · **Assembly:** `Scion.Trials` · **Define:** `SCION_TRIALS`

```csharp
public sealed class TrialArgs : IEnumerable<KeyValuePair<string, object>>
{
    public void Add(string name, object value);
}
```

Keys are names of recipe members, looked up on the recipe class and then on its base classes, or
`tunable:<Type>.<Field>`.

## TrialModesAttribute

**Namespace:** `Scion.Trials` · **Assembly:** `Scion.Trials` · **Define:** `SCION_TRIALS`

```csharp
[AttributeUsage(AttributeTargets.Assembly)]
public sealed class TrialModesAttribute : Attribute
{
    public TrialModesAttribute(params string[] modes);   // "360p=640x360", "deuter"
}
```

## ITrialModeAware

**Namespace:** `Scion.Trials` · **Assembly:** `Scion.Trials` · **Define:** `SCION_TRIALS`

```csharp
public interface ITrialModeAware
{
    void ApplyMode(string mode);
}
```

Adapters implement it for modes that are not resolutions. A `-scion-mode` that `TrialModes` does not declare also
goes to `ApplyMode`, with the default capture size.

## CapturedMatch

**Namespace:** `Scion.Trials` · **Assembly:** `Scion.Trials` · **Define:** `SCION_TRIALS`

```csharp
public sealed class CapturedMatch : TrialRecipe
{
    public string Match { get; }          // the variant key "match"
    public MatchData Data { get; }        // the parsed .match.json
}
```

One variant per file in `scion/recipes/captured/` and `found/`. Played by `IMatchCapture.Replay`.

## See also

- [Write a recipe](/scion/41664431a8f5/trials/recipes/)
- [Variants, modes and seeds](/scion/41664431a8f5/trials/variants/)
- [Turn any match into a recipe](/scion/41664431a8f5/trials/captured-matches/)

---
title: "Write a recipe"
description: "Declare a TrialRecipe with its seed, fps, timeout, players, tags and checks, cut the capture around the action, and ship recipes in a DLL."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/trials/recipes/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/trials/recipes.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/adapters.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/session.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/checks.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/recipes.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Write a recipe

A recipe is data: a `TrialRecipe` subclass whose constructor sets what to play and how to capture it. The session
creates one instance of every concrete recipe class through its public parameterless constructor (SCN201). This
guide covers the members (`Name`, `Id`, `Variant`, `Tags`, `Description`, `Seed`, `Fps`, `TailSeconds`,
`TimeoutSeconds`, `Players`, `Explicit`, `Control`, `Tolerance`), the overrides (`Checks`, `Setup`, `Variants`,
`Mask`), cutting the capture by event, and recipes compiled outside Unity with `--recipes`.

## When to use

- Write one recipe per situation a person would want to look at after a change: one hit, one ability, one turn,
  one screen.
- Keep recipes short (a few seconds of game time): many short recipes parallelize better than one long one.
- Put the fields every recipe of the game shares in one base class, played by one adapter.

## Before you begin

- An assembly for trials code with `defineConstraints: ["SCION_TRIALS"]`, so it only compiles into test builds
  (`Orchard.Trials`), or `#if SCION_TRIALS` around the code.
- An adapter for the recipe's base type ([Write an adapter](/scion/41664431a8f5/trials/adapters/)).

## Steps

### 1. A base type with the game's fields

```csharp
using Scion.Trials;

namespace Orchard.Trials
{
    public abstract class OrchardRecipe : TrialRecipe
    {
        public string Attacker = "knight";
        public string Defender = "skeleton";
        public int DefenderHp = 20;
        public int Distance = 1;
        public string Ability = "slash";
    }
}
```

### 2. One recipe

```csharp
namespace Orchard.Trials
{
    public sealed class SlashHit : OrchardRecipe
    {
        public SlashHit()
        {
            Description = "The knight slashes a skeleton next to it.";
            Tags = new[] { "combat", "melee" };
            Seed = 7;
            TailSeconds = 1f;
        }

        protected override void Checks(TrialChecks c) =>
            c.Marked("IMPACT").Frames(20, 90).ProbeIs("damage", d => d > 0, "the slash deals damage");
    }
}
```

### 3. Cut the capture around the action

The adapter decides when the interesting part starts and ends:

```csharp
session.BeginCut();                 // frames from here on are captured
battle.Issue(Command.Attack(recipe.Ability, target));
yield return session.WaitUntil(() => battle.Idle, "attack finished");
yield return session.WaitSeconds(recipe.TailSeconds);
session.EndCut();
```

The moment that matters is marked from a trials-only observer graft, so `window.png` centers on it:

```csharp
[Before(typeof(HitPresenter), nameof(HitPresenter.Play), Observe = true)]
public static void Before(HitPresenter presenter, HitEvent e) => TrialSession.Current?.MarkImpact();
```

### 4. List and run it

```sh
dotnet scion trial list
dotnet scion trial run SlashHit --rounds 2
```

## Reference

### Members

| Member | Type | Default | Meaning |
|---|---|---|---|
| `Name` | `string`, virtual get | class name | What `--only` patterns match; the base of `Id`. |
| `Id` | `string`, set by the session | `Name` | The instance id, with variant, mode and seed: `SlashHit[Distance=3]@360p#7`. |
| `Variant` | `string`, set by the session | `null` | The variant arguments, `Distance=3`. |
| `Mode` | `string`, set by the session | `null` | The mode, `360p`. |
| `Tags` | `IReadOnlyList<string>`, protected set | empty | Labels for `--tag` filters. |
| `Description` | `string`, protected set | `null` | One line: what the recipe shows. |
| `Seed` | `int`, protected set | 1 | Seeds `UnityEngine.Random` and `SeededRandom` at the start of the recipe. |
| `Fps` | `int`, protected set | 30 | Capture rate; game time advances exactly `1/Fps` per frame. |
| `TailSeconds` | `float`, protected set | 0.5 | Game seconds to keep capturing after the action; the adapter uses it. |
| `TimeoutSeconds` | `float`, protected set | 30 | Limit for `Run`, in game seconds. |
| `Players` | `int`, protected set | 1 | 1 to 4; co-op recipes run in groups ([Co-op trials](/scion/41664431a8f5/trials/coop/)). |
| `Explicit` | `bool`, protected set | `false` | Runs only when a pattern names its exact `Id` or `Name`. |
| `Control` | `DeterminismControl`, protected set | `All` | What the session makes deterministic ([Particles, animators, UI and physics](/scion/41664431a8f5/determinism/control/)). |
| `Tolerance` | `ImageTolerance`, protected set | `ImageTolerance.Default` | Image comparison; `null` compares no images ([Image comparison](/scion/41664431a8f5/trials/image-comparison/)). |
| `Checks(TrialChecks c)` | protected virtual | none | Declares checks ([Checks](/scion/41664431a8f5/trials/checks/)). |
| `Setup()` | protected virtual | none | Runs after variant arguments are applied. |
| `Variants()` | protected virtual | `null` | Yields `TrialArgs`, one per variant ([Variants, modes and seeds](/scion/41664431a8f5/trials/variants/)). |
| `Mask(artifact, rect, why)` | protected | | A region image comparison ignores, in normalized coordinates. |

`Seed` is per recipe; `TrialSession.RunSeed` (`--seed`) is per run, and the adapter maps it to the game's world
seed, so another run seed tries another world with the same recipes.

### Recipes from a DLL

Recipes compiled outside Unity load at startup:

```sh
dotnet scion trial run --recipes scion/recipes/bin/Orchard.Recipes.dll
```

The DLL references the game's assemblies from the build's `Managed/` folder. A recipe whose `Name` equals a
built-in one replaces it, so a new or changed recipe never needs a game rebuild. `trial list` also lists the
sources in `scion/recipes/` (`trials.recipes_staging`).

### Recipes that must not run by default

Set `Explicit = true` for recipes that fail on purpose (to test a check), take minutes, or need a special machine.
They run only when named: `dotnet scion trial run KnightFallsSlow`.

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `SCN201` | 1 (selected) or log warning | The recipe has no public parameterless constructor. | Add `public SlashHit() { ... }`. |
| `no_match` | 2 | No recipe matches the pattern. | Use `did_you_mean`, or `trial list`. |
| recipe error `timeout (30s): <what>` | 1 | `Run` passed `TimeoutSeconds`. | Read the `what`; raise the timeout only if the game really needs it. |

## Examples

### Three attackers

```csharp
public sealed class FireballArea : OrchardRecipe
{
    public FireballArea()
    {
        Attacker = "mage"; Ability = "fireball"; Distance = 4;
        Description = "A fireball hits three skeletons in an area.";
        Tags = new[] { "combat", "area" };
    }
    protected override void Checks(TrialChecks c) => c.Marked("IMPACT", atLeast: 3);
}
```

### A unit going down

```csharp
public sealed class KnightFalls : OrchardRecipe
{
    public KnightFalls()
    {
        Attacker = "skeleton"; Defender = "knight"; DefenderHp = 3;
        Description = "A skeleton finishes a wounded knight.";
    }
    protected override void Checks(TrialChecks c) => c.Order("IMPACT", "unit_down").ProbeIs("defender_hp", hp => hp == 0, "the knight falls");
}
```

## See also

- [Write an adapter](/scion/41664431a8f5/trials/adapters/)
- [TrialSession](/scion/41664431a8f5/trials/session/)
- [Checks](/scion/41664431a8f5/trials/checks/)
- [Recipes API](/scion/41664431a8f5/reference/api/recipes/)

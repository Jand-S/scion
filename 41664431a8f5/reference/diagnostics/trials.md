---
title: "SCN201-SCN203"
description: "Trials diagnostics - recipes that cannot be created, invalid adapters, and two adapters claiming the same recipe type."
type: diagnostics
url: https://jand-s.github.io/scion/41664431a8f5/reference/diagnostics/trials/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/diagnostics/trials.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/recipes.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/adapters.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# SCN201-SCN203

The test player checks recipes and adapters when the session starts. A selected recipe that cannot be created fails
alone; an invalid adapter or an ambiguous choice stops the run with exit 3 before any recipe plays.

## Diagnostics

### SCN201

**Severity:** error · **Source:** runtime

**Message:** `Recipe {0} has no public parameterless constructor.` **Hint:** `Add: public {0}() { ... }`

**Cause.** The session creates recipes through their public parameterless constructor.

**Fix.**

```csharp
public sealed class SlashHit : OrchardRecipe
{
    public SlashHit() { Seed = 7; }
}
```

When the recipe is selected, it fails with `error: "SCN201: ..."`; when it is not, the log has a warning and the
CLI reports `recipe_unbuildable`.

**Suppress.** Not allowed.

### SCN202

**Severity:** error · **Source:** runtime

**Message:** `Adapter {0} is not valid: {1}.` **Hint:** `Derive from TrialAdapter<TRecipe> (or implement
ITrialAdapter) with a public parameterless constructor.`

**Cause.** A type in `[assembly: TrialAdapter(typeof(...))]` does not implement `ITrialAdapter`, is abstract, or a
non-`MonoBehaviour` adapter has no public parameterless constructor. Exit 3.

**Fix.** Derive from `TrialAdapter<TRecipe>`.

**Suppress.** Not allowed.

### SCN203

**Severity:** error · **Source:** runtime

**Message:** `Adapters {0} and {1} both handle {2}.` **Hint:** `Keep one adapter per recipe type; a more specific type
wins.`

**Cause.** Two registered adapters have the same recipe type and both accept a selected recipe. Exit 3.

**Fix.** Remove one, or make one handle a more specific recipe type.

**Suppress.** Not allowed.

## See also

- [Write a recipe](/scion/41664431a8f5/trials/recipes/)
- [Write an adapter](/scion/41664431a8f5/trials/adapters/)
- [Diagnostics](/scion/41664431a8f5/reference/diagnostics/)

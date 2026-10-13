---
title: "SCN001-SCN005"
description: "The Scion Standard analyzers - sealed rule classes, shallow inheritance, grafts that write rule state, float in rule code and tunables without a range."
type: diagnostics
url: https://jand-s.github.io/scion/41664431a8f5/reference/diagnostics/standard/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/diagnostics/standard.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/standard/rules.md
  - https://jand-s.github.io/scion/41664431a8f5/standard/enforcement.md
  - https://jand-s.github.io/scion/41664431a8f5/grafts/visual-only.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# SCN001-SCN005

These analyzers enforce the [Scion Standard](/scion/standard/overview/). SCN001, SCN002, SCN004 and SCN005 apply only
to rule code (the assemblies in `analyzers.rule_assemblies` of `ScionSettings.json`); SCN003 applies to graft code.
With no `analyzers` section they report nothing.

## Diagnostics

### SCN001

**Severity:** warning · **Source:** analyzer

**Message:** `{0} is not sealed.` **Hint:** `Seal the class, or suppress SCN001 with a justification.`

**Cause.** A class in rule code is not `sealed` (rule R1). Abstract classes and static classes are exempt.

**Fix.** Add `sealed`.

```csharp
// Before
public class BackstabBonus : RuleModifier<DamageCalc> { }
// After
public sealed class BackstabBonus : RuleModifier<DamageCalc> { }
```

**Suppress.** `[SuppressMessage("Scion", "SCN001", Justification = "...")]`.

### SCN002

**Severity:** warning · **Source:** analyzer

**Message:** `{0} inherits more than one level ({1}).` **Hint:** `Prefer composition or a single abstract base.`

**Cause.** A class in rule code derives from a class that itself derives from something other than `object` or a
Scion base (rule R1).

**Fix.** Flatten the hierarchy: derive from the Scion base directly and move shared code into a helper.

**Suppress.** With a justification, for example when a library requires its own base.

### SCN003

**Severity:** error · **Source:** analyzer

**Message:** `Graft {0} writes rule state ({1}); grafts are visual only.` **Hint:** `Move the rule change to the
core; rule code is listed in ScionSettings.json analyzers.`

**Cause.** Inside graft code, a write to a field, property, indexer, event or array element of a rule type, or any use
of an internal member of a rule type.

**Fix.** Move the change into the core as a `RuleModifier<T>` or a command.

```csharp
// Before
[After(typeof(HitPresenter), nameof(HitPresenter.Play))]
public static void After(HitPresenter presenter, HitEvent e) { presenter.Battle.State.Turn = 0; }
// After: no write; the rule change lives in Orchard.Core
```

**Suppress.** Not allowed.

### SCN004

**Severity:** warning · **Source:** analyzer

**Message:** `float used in rule code ({0}).` **Hint:** `Use int or fixed point in the core so every machine gets
the same result.`

**Cause.** A `float` or `double` field, local, parameter or operation in rule code. Floating point can differ between
machines and compilers, which breaks co-op and replays.

**Fix.** Use `int` with a fixed scale (permille).

```csharp
// Before
c.Multiplier *= 1.5f;
// After
c.MulPermille += 500;
```

**Suppress.** For values that never feed the state (debug output), with a justification.

### SCN005

**Severity:** warning · **Source:** analyzer

**Message:** `{0} is [Tunable] without a range.` **Hint:** `Declare it as [Tunable(min, max)]; numeric tunables need
a range.`

**Cause.** A numeric field marked `[Tunable]` without `min` and `max`. Search, presets and clamping need the range.

**Fix.**

```csharp
// Before
[Tunable] public static float Strength = 0.3f;
// After
[Tunable(0f, 1f)] public static float Strength = 0.3f;
```

**Suppress.** Not recommended.

## See also

- [Rules R1-R8](/scion/standard/rules/)
- [Enforcement](/scion/standard/enforcement/)
- [Grafts are visual only](/scion/grafts/visual-only/)
- [Diagnostics](/scion/reference/diagnostics/)

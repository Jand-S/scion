---
title: "Write a graft"
description: "Write Before and After grafts - method shape, the asmref folder, parameter binding by exact signature or prefix, Priority and Observe."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/grafts/write/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/grafts/write.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/grafts/call-context.md
  - https://jand-s.github.io/scion/41664431a8f5/grafts/anchors.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/grafts.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/diagnostics/grafts.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Write a graft

This guide writes grafts with `[Before(typeof(T), "Method")]` and `[After(typeof(T), "Method")]`: the method shape the
weaver accepts (`static void`, public or internal, not generic, SCN101), the folder with an `.asmref` that compiles it
into the target's assembly (SCN102), how parameters bind to the target (exact signature, then a unique prefix, SCN112),
and the ordering controls `Priority` and `Observe`.

## When to use

- Use a graft to add presentation to a game method without editing it: an effect, a sound, a camera move, a HUD
  change, telemetry, a trials mark.
- Do not use a graft to change a rule (damage, range, turn order, anything in the state hash). Change the core with a
  `RuleModifier<T>` ([Grafts are visual only](/scion/41664431a8f5/grafts/visual-only/)).

## Before you begin

- The game assembly references `Scion.Runtime` (`dotnet scion init` adds it).
- A promoted grafts folder with an `.asmref` to the game assembly, for example
  `Assets/Orchard/Grafts/Orchard.Grafts.asmref`:

```json
{"reference":"Orchard.Runtime"}
```

## Steps

### 1. Find the anchor

```sh
dotnet scion anchor list --search hit
dotnet scion anchor show presentation.hit
```

`anchor show` prints the target's signature and a declaration that binds to it.

### 2. Create the draft

```sh
dotnet scion graft new CritShake --after Orchard.Presentation.HitPresenter.Play
```

<!-- validate: cli/graft-new -->
```json
{"schema_version":1,"command":"graft new","ok":true,"file":"scion/grafts/CritShake.cs","target":"Orchard.Presentation.HitPresenter.Play","kind":"after","signature":"public static void After(HitPresenter presenter, HitEvent e, ref Call<float> c)","next_step":"edit scion/grafts/CritShake.cs, then: dotnet scion graft check scion/grafts/CritShake.cs"}
```

### 3. Write the body

```csharp
using Orchard.Core;
using Orchard.Presentation;
using Scion;
using Scion.Core;

namespace Orchard.Grafts
{
    public static class CritShake
    {
        [Tunable(0f, 1f, Doc = "Camera shake on a critical hit.")]
        public static float Strength = 0.3f;

        // After every hit: critical hits shake the camera, pause briefly and last a bit longer.
        [After(typeof(HitPresenter), nameof(HitPresenter.Play))]
        public static void After(HitPresenter presenter, HitEvent e, ref Call<float> c)
        {
            if (!e.Crit) return;
            Fx.Shake(Strength);
            Fx.HitStop(0.06f);
            c.Result += 0.06f;   // Play returns how long the hit lasts
        }
    }
}
```

### 4. Check it

```sh
dotnet scion graft check scion/grafts/CritShake.cs
```

The check runs Roslyn and the weaver rules against the test build, without Unity. Fix every diagnostic it lists.

### 5. Try and promote it

Inject it into a running copy ([Live injection](/scion/41664431a8f5/grafts/injection/)), or promote it and rebuild
([From draft to final build](/scion/41664431a8f5/grafts/lifecycle/)).

## Reference

### Before and After

| Attribute | Runs | Typical use |
|---|---|---|
| `[Before(typeof(T), "Method")]` | before the target's body | watch a call, guard it, skip it, return early |
| `[After(typeof(T), "Method")]` | at every `return` of the target | react to a call, read or change its result |

Name the target with `nameof` when it is visible from the graft, and with a string when it is private:

```csharp
[After(typeof(HitPresenter), "PlayCritical")]   // private method: name as a string
public static void After(HitPresenter presenter, HitEvent e) { }
```

Both attributes allow multiple use, so one method can hang on several targets. Targets must be declared on the type
itself: a method inherited from a base class is not found ([SCN103](/scion/41664431a8f5/reference/diagnostics/grafts/#scn103)).

### Method shape

| Rule | Diagnostic |
|---|---|
| `static void` | [SCN101](/scion/41664431a8f5/reference/diagnostics/grafts/#scn101) |
| `public` or `internal` (injected drafts: `public`, in a `public` type) | SCN101 |
| Not generic, not in a generic type, not in a private or protected nested type | SCN101 |
| Compiled into the target's assembly (folder with an `.asmref`) | [SCN102](/scion/41664431a8f5/reference/diagnostics/grafts/#scn102) |
| Never called by game code | [SCN110](/scion/41664431a8f5/reference/diagnostics/grafts/#scn110) |

Name the class after the graft (`CritShake`) and the method after its side (`Before` or `After`). The graft's
identity is its full name, `Orchard.Grafts.CritShake.After`: it orders ties, names the graft in `Fx` telemetry and
decides which promoted graft an injected one replaces.

### Parameter binding

A graft's parameters mirror the target's:

1. the target instance, for instance methods (omit it for static targets);
2. the target's parameters, in order, with the same `in` and `ref` modifiers;
3. optionally, last, `ref Call` (void target) or `ref Call<T>` (target returns `T`).

**Prefixes.** A graft may stop early and declare only the first parameters:

```csharp
[After(typeof(HitPresenter), nameof(HitPresenter.Play))]
public static void After(HitPresenter presenter) => hits++;   // only the instance
```

**Overloads.** The overload whose full signature matches wins; otherwise the single overload the prefix matches.
When a prefix matches two or more overloads, it is [SCN112](/scion/41664431a8f5/reference/diagnostics/grafts/#scn112): declare all
parameters. A signature that matches nothing is [SCN104](/scion/41664431a8f5/reference/diagnostics/grafts/#scn104), whose hint
gives the expected signature.

**Values.** A graft sees the current parameter values: if the body reassigns a parameter, an `[After]` sees the new
value.

**Overrides.** On a virtual method, a graft fires only for the method it targets. A graft on `UnitView.Play` does not
fire when an override in `KnightView` runs without calling `base.Play`: target the override.

### Priority

`Priority` (an `int`, default 0) orders the grafts of one anchor and side: lower runs first. Ties break on the graft's
full name (ordinal), so order never depends on file or compile order.

```csharp
[After(typeof(HitPresenter), nameof(HitPresenter.Play), Priority = -10)]
public static void After(HitPresenter presenter, HitEvent e) { }
```

### Observe

`Observe = true` (only on `[Before]`) marks an observer: a graft that only watches. Observers always run, before
every `[Before]` that may skip the target, ordered by `Priority` among themselves, and may not call `Skip()` or
`Return()` ([SCN108](/scion/41664431a8f5/reference/diagnostics/grafts/#scn108)). Use one when the graft must see every call,
including calls another graft skips, such as a trials mark. `TrialSession` exists only in test builds, so a graft
that uses it sits inside `#if SCION_TRIALS`; a promoted graft compiles into the game, and without the define the
final build fails ([Build modes](/scion/41664431a8f5/grafts/build-modes/)):

```csharp
#if SCION_TRIALS
[Before(typeof(HitPresenter), nameof(HitPresenter.Play), Observe = true)]
public static void Before(HitPresenter presenter, HitEvent e) => TrialSession.Current?.MarkImpact();
#endif
```

### Order within one call

1. observers, by `Priority`;
2. the other `[Before]` grafts, by `Priority`; the first that skips stops the remaining befores and the body;
3. the target's body;
4. the `[After]` grafts, by `Priority`; they also run after a skip, and can read `Skipped`.

If the target throws, the `[After]` grafts do not run.

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `SCN101` | compile error | Not `static void`, private, generic, or in a generic or hidden type. | `public static void After(...)` in a non-generic public class. |
| `SCN102` | compile error | The file compiles into another assembly than the target. | Move it to the folder with the `.asmref`. |
| `SCN103` | compile error | The target method does not exist on that type. | Use the name from the hint. |
| `SCN104` | compile error | Parameters match no overload. | Copy the expected signature from the hint. |
| `SCN105` | compile error | The target cannot be an anchor. | Hang the graft on the plain method it calls. |
| `SCN108` | compile error | `Skip` or `Return` in an `[After]` or an observer. | Move it to a non-observer `[Before]`. |
| `SCN112` | compile error | A prefix matches several overloads. | Declare all parameters. |

In `graft check` and `build test` these come back in `diagnostics[]` or `errors[]` with exit code 1. The full list is
in [Graft diagnostics](/scion/41664431a8f5/reference/diagnostics/grafts/).

## Examples

### A sound on every turn banner

```csharp
[Before(typeof(Hud), nameof(Hud.ShowTurn))]
public static void Before(Hud hud, int turn)
{
    Fx.Sound(turn == 1 ? "turn_first" : "turn");
}
```

### One graft on two targets

```csharp
[After(typeof(DamageNumbers), nameof(DamageNumbers.Spawn))]
[After(typeof(DamageNumbers), nameof(DamageNumbers.SpawnHeal))]
public static void After(DamageNumbers numbers, int amount, Vector3 at) => Fx.Sound("pop", at, 0.6f);
```

Both targets share the parameter list `(int amount, Vector3 at)`, so one method fits both.

## See also

- [Call and Call\<T\>](/scion/41664431a8f5/grafts/call-context/)
- [Anchors](/scion/41664431a8f5/grafts/anchors/)
- [graft (CLI)](/scion/41664431a8f5/cli/graft/)
- [Grafts API](/scion/41664431a8f5/reference/api/grafts/)

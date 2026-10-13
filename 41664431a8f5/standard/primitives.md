---
title: "Primitives"
description: "The building blocks of the Scion Standard - anchors, grafts, rule modifiers, strategies, type objects, tunables, probes, cues, adapters and exemplars."
type: reference
url: https://jand-s.github.io/scion/41664431a8f5/standard/primitives/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/standard/primitives.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/standard/rules.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/core.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/grafts.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Primitives

A primitive is a building block with one job. Choosing the right one is most of following the standard: a number
that changes a rule is a `RuleModifier<T>`, a sound on a hit is a graft, a number someone will tune is a
`[Tunable]`. This page lists each primitive, when to use it and where it is documented.

## Table

| Primitive | Use it for | Never for | Docs |
|---|---|---|---|
| `[Anchor(Id, Doc)]` | Stable, documented presentation moments agents should find first. | Rule methods. | [Anchors](/scion/41664431a8f5/grafts/anchors/) |
| Graft (`[Before]`/`[After]`) | Juice, sound, camera, HUD, telemetry, tutorial hints. | Changing a rule. | [Write a graft](/scion/41664431a8f5/grafts/write/) |
| `RuleModifier<T>` + `Stage` | Any rule that changes a number (damage, range, cost). | Presentation. | [Core API](/scion/41664431a8f5/reference/api/core/) |
| Strategy by enum | Effects, AI choices, presentation per effect. | Open-ended plugins. | [Rules R1-R8](/scion/41664431a8f5/standard/rules/) |
| Type Object | Immutable definitions: abilities, units, rooms (data rows, or a ScriptableObject compiled to a table). | State that changes during a match. | [Rules R1-R8](/scion/41664431a8f5/standard/rules/) |
| `[Tunable(min, max)]` | A number someone tunes; on rule fields, it enters the lobby hash. | Constants nobody tunes. | [Tunables and presets](/scion/41664431a8f5/grafts/tunables/) |
| `[Probe("key")]` | A rule value recorded in trials. | Per-frame values (use a sensor). | [Telemetry, probes and sensors](/scion/41664431a8f5/evaluation/telemetry/) |
| Cue | A visual triggered by an event, one-shot or added and removed (a burning aura). | Logic. | below |
| Adapter | The game's port: trials, network, input. | Game rules. | [Write an adapter](/scion/41664431a8f5/trials/adapters/) |
| Exemplar | A small compiled, tested example per primitive for agents to copy. | Documentation prose. | below |

## Stages of a rule modifier

| Stage | Order | Typical use |
|---|---|---|
| `Stage.Base` | 1 | Set the base value from data. |
| `Stage.Add` | 2 | Flat bonuses. |
| `Stage.Mul` | 3 | Multipliers in permille (`MulPermille += 500` is +50 %). |
| `Stage.Clamp` | 4 | Minimums, maximums, immunities. |
| `Stage.Final` | 5 | Last-word effects (shields that absorb). |

Within a stage, `Priority` orders modifiers (lower first), then the type's full name.

## Cues

A cue is a pattern, not a type: a presentation method that reacts to one core event, named after it, and therefore
an anchor.

```csharp
public sealed class StatusCues : MonoBehaviour
{
    public void OnBurnAdded(int unit) => Auras.Add(unit, "burn");
    public void OnBurnRemoved(int unit) => Auras.Remove(unit, "burn");
}
```

Grafts on `OnBurnAdded` add sound or shake without touching the status system.

## Exemplars

`Assets/Orchard/Exemplars/` holds one file per primitive (`ExemplarGraft.cs`, `ExemplarModifier.cs`,
`ExemplarRecipe.cs`...), compiled with the project and covered by a test. `scion new` copies from them, and the
project index points agents to them.

## See also

- [Rules R1-R8](/scion/41664431a8f5/standard/rules/)
- [Core API](/scion/41664431a8f5/reference/api/core/)
- [Grafts API](/scion/41664431a8f5/reference/api/grafts/)
- [Enforcement](/scion/41664431a8f5/standard/enforcement/)

---
title: "Particles, animators, UI and physics"
description: "Make particle systems, animators, UI Toolkit and physics deterministic per recipe with DeterminismControl and Determinism.SeedFor."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/determinism/control/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/determinism/control.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/determinism/overview.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/determinism.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/image-comparison.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Particles, animators, UI and physics

Time and randomness are not the only sources of variation: particle systems pick their own seeds, animators and UI
Toolkit update on their own schedules, and physics steps depend on frame timing. `DeterminismControl` turns Scion's
control of each on or off per recipe (`Particles`, `Animators`, `UiToolkit`, `Physics`; all on by default), and
`Determinism.SeedFor` gives your own systems stable seeds by path and creation order.

## When to use

- Leave `Control = DeterminismControl.All` unless a recipe tests the uncontrolled behavior.
- Turn one control off when it conflicts with how the game drives that system, and mask or skip the affected pixels.
- Use `SeedFor` for procedural systems of your own (foliage sway, ambient creatures).

## Steps

### 1. Keep the default, or narrow it

```csharp
public sealed class FireballArea : OrchardRecipe
{
    public FireballArea()
    {
        Control = DeterminismControl.All & ~DeterminismControl.Physics;   // the game steps physics itself
    }
}
```

### 2. Seed your own systems

```csharp
void Awake()
{
#if SCION_TRIALS
    seed = Determinism.SeedFor(transform.GetHierarchyPath(), creationOrder: Sway.Count++);
#else
    seed = Environment.TickCount;
#endif
}
```

### 3. Verify

```sh
dotnet scion trial run FireballArea --rounds 3 --check
```

## Reference

### Controls

| Flag | What Scion does while a recipe runs |
|---|---|
| `Particles` | Every `ParticleSystem` gets `useAutoRandomSeed = false` and `randomSeed = SeedFor(path, order)` when it starts; clones of one prefab differ by creation order. |
| `Animators` | Animators update in the session's frame step (`AnimatorUpdateMode.Normal` driven by virtual time), never with `UnscaledTime` from the real clock. |
| `UiToolkit` | UI Toolkit panels tick on the virtual clock, so transitions and caret blinks land on the same frames. |
| `Physics` | `Physics.simulationMode = Script` and one `Physics.Simulate(1/Fps)` per frame; `Physics2D` likewise. |

`DeterminismControl.None` turns all off; `All` turns all on.

### Determinism members

| Member | Meaning |
|---|---|
| `Determinism.SeedFor(string path, int creationOrder)` | FNV hash of the recipe seed, the path and the order. |
| `Determinism.Census()` | What this frame still depends on: real-time reads, auto-seeded systems, dynamic bodies. Logged in `report.md` when a round differs. |
| `Determinism.Begin`, `End` | Called by the session. |

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| check `px` fails | 1 | Pixels differ between rounds. | Read `determinism.tsv` with `--check`; look at `Census` in `report.md`. |
| warning `px_unstable` | 0 | Pixels differ between rounds of the run. | Find the uncontrolled system; mask it as a last resort. |

## Examples

### A recipe that tests physics as the game runs it

```csharp
Control = DeterminismControl.Particles | DeterminismControl.Animators | DeterminismControl.UiToolkit;
```

### Print the census from the adapter

```csharp
foreach (var line in Determinism.Census()) s.Note(line);
```

## See also

- [Deterministic runs](/scion/41664431a8f5/determinism/overview/)
- [Verify determinism](/scion/41664431a8f5/determinism/verify/)
- [Determinism API](/scion/41664431a8f5/reference/api/determinism/)

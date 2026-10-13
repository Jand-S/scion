---
title: "Clock, screen and random"
description: "Reference for Clock, VirtualScreen, VirtualWaitForSecondsRealtime, SeededRandom, the WallClock attribute and the redirect_assemblies setting."
type: reference
url: https://jand-s.github.io/scion/41664431a8f5/determinism/clock-and-screen/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/determinism/clock-and-screen.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/determinism/overview.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/determinism.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/configuration.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Clock, screen and random

These types are the targets of the weaver's redirects in test builds (`SCION_TRIALS`) and live in `Scion.Runtime`,
namespace `Scion`. Game code rarely calls them directly: it keeps reading `Time`, `Screen` and `DateTime`, and the
weaver points the reads here. This page lists what each one does, when it is active, and how to keep real time with
`[WallClock]`.

## Clock

| Member | Meaning |
|---|---|
| `const double RecipeStart = 1000.0` | `Time` and `UnscaledTime` at the start of every recipe. |
| `static readonly DateTime DefaultEpoch` | 2000-01-01T00:00:00Z. |
| `static bool Active` | True between `Begin` and `End`. Inactive, every property returns the real value. |
| `static DateTime Epoch` | The virtual wall-clock start (`-scion-epoch`). |
| `static void Begin(double start = RecipeStart)` | Called by the session at each recipe. |
| `static void End()` | Called at the end of the queue. |
| `static void Advance(float delta)` | Tests outside Unity: move every clock by `delta`. |
| `Time`, `TimeAsDouble` | Virtual `Time.time`. |
| `TimeSinceLevelLoad`, `TimeSinceLevelLoadAsDouble` | Virtual time since the recipe start. |
| `UnscaledTime`, `UnscaledTimeAsDouble`, `UnscaledDeltaTime` | Virtual unscaled time. |
| `RealtimeSinceStartup`, `RealtimeSinceStartupAsDouble` | Equal to virtual unscaled time. |
| `Now`, `UtcNow`, `Today` | `Epoch` plus elapsed virtual unscaled time. `Now` and `UtcNow` have the same value. |

## VirtualScreen

| Member | Meaning |
|---|---|
| `static void Set(int width, int height)` | The capture size; the session sets it from `--size`. |
| `static void Clear()` | Back to the real screen. |
| `static int Width`, `Height` | Targets of `Screen.width` and `Screen.height`. |

`-batchmode` players report 640x480 from `Screen`; with `VirtualScreen` the game lays out its UI at the capture size.

## VirtualWaitForSecondsRealtime

A `WaitForSecondsRealtime` whose wait reads `Clock.RealtimeSinceStartupAsDouble`, so a real-time wait lasts the
same number of frames in every run. The weaver replaces `new WaitForSecondsRealtime(t)` with it.

## SeededRandom

A `System.Random` whose seed is a hash of the recipe seed and the order of creation within the recipe.
`SeededRandom.Reset(seed)` restarts the count; the session calls it at each recipe. The weaver replaces
`new System.Random()` with `new SeededRandom()`. Create instances on the main thread so the order is stable.

## WallClock

```csharp
[WallClock]
static double ElapsedRealSeconds() => Time.realtimeSinceStartupAsDouble - startedAt;
```

`[WallClock]` on a method, class or struct stops every redirect inside it, including compiler-generated code
(iterators, async bodies, lambdas, local functions) and nested types. Use it for profilers, timeouts and log
timestamps. It changes nothing in final builds.

## Redirect assemblies

Assemblies that do not reference `Scion.Runtime` (Unity UI, URP) can have their reads redirected too:

```json
{"schema_version":1,"trials":{"redirect_assemblies":["UnityEngine.UI","Unity.RenderPipelines.Universal.Runtime"]}}
```

in `ProjectSettings/ScionSettings.json`. Only reads are redirected there; `new` expressions are not replaced. Scion's
own assemblies are never redirected. Recompile after editing the file.

## See also

- [Deterministic runs](/scion/determinism/overview/)
- [Determinism API](/scion/reference/api/determinism/)
- [Configuration](/scion/reference/configuration/)
- [SCN301-SCN307](/scion/reference/diagnostics/determinism/)

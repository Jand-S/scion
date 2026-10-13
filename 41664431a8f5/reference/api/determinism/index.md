---
title: "Determinism API"
description: "Clock, VirtualScreen, VirtualWaitForSecondsRealtime, SeededRandom, Determinism and NetClock - the virtual time, screen and random of test builds."
type: api
url: https://jand-s.github.io/scion/41664431a8f5/reference/api/determinism/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/api/determinism.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/determinism/overview.md
  - https://jand-s.github.io/scion/41664431a8f5/determinism/clock-and-screen.md
  - https://jand-s.github.io/scion/41664431a8f5/determinism/control.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Determinism API

These types are the targets of the weaver's redirects and the session's per-recipe resets in test builds. They live
in `Scion.Runtime` under `SCION_TRIALS` (`NetClock` in `Scion.Trials`). Inactive, they return real values.

## Clock

**Namespace:** `Scion` · **Assembly:** `Scion.Runtime` · **Define:** `SCION_TRIALS`

```csharp
public static class Clock
{
    public const double RecipeStart = 1000.0;
    public static readonly DateTime DefaultEpoch;      // 2000-01-01T00:00:00Z
    public static bool Active { get; }
    public static DateTime Epoch { get; set; }
    public static void Begin(double start = RecipeStart);
    public static void End();
    public static void Advance(float delta);
    public static float Time { get; }
    public static double TimeAsDouble { get; }
    public static float TimeSinceLevelLoad { get; }
    public static double TimeSinceLevelLoadAsDouble { get; }
    public static float UnscaledTime { get; }
    public static double UnscaledTimeAsDouble { get; }
    public static float UnscaledDeltaTime { get; }
    public static float RealtimeSinceStartup { get; }
    public static double RealtimeSinceStartupAsDouble { get; }
    public static DateTime Now { get; }
    public static DateTime UtcNow { get; }
    public static DateTime Today { get; }
}
```

`Begin` and `End` are called by the session. `Advance` is for tests outside Unity.

## VirtualScreen

**Namespace:** `Scion` · **Assembly:** `Scion.Runtime` · **Define:** `SCION_TRIALS`

```csharp
public static class VirtualScreen
{
    public static void Set(int width, int height);
    public static void Clear();
    public static int Width { get; }
    public static int Height { get; }
}
```

## VirtualWaitForSecondsRealtime

**Namespace:** `Scion` · **Assembly:** `Scion.Runtime` · **Define:** `SCION_TRIALS`

```csharp
public sealed class VirtualWaitForSecondsRealtime : UnityEngine.WaitForSecondsRealtime
{
    public VirtualWaitForSecondsRealtime(float time);
}
```

Waits on `Clock.RealtimeSinceStartupAsDouble`.

## SeededRandom

**Namespace:** `Scion` · **Assembly:** `Scion.Runtime` · **Define:** `SCION_TRIALS`

```csharp
public sealed class SeededRandom : System.Random
{
    public SeededRandom();
    public static void Reset(int seed);
}
```

The seed is a hash of the recipe seed and the order of creation since the last `Reset`.

## DeterminismControl

**Namespace:** `Scion` · **Assembly:** `Scion.Runtime` · **Define:** `SCION_TRIALS`

```csharp
[Flags]
public enum DeterminismControl { None = 0, Particles = 1, Animators = 2, UiToolkit = 4, Physics = 8, All = ~0 }
```

## Determinism

**Namespace:** `Scion` · **Assembly:** `Scion.Runtime` · **Define:** `SCION_TRIALS`

```csharp
public static class Determinism
{
    public static void Begin(int seed, DeterminismControl control);
    public static void End();
    public static int SeedFor(string path, int creationOrder);
    public static IReadOnlyList<string> Census();
}
```

`Begin` and `End` are called by the session. `SeedFor` is FNV of the seed, the path and the order. `Census` lists
what still depends on real time or automatic seeds.

## NetClock

**Namespace:** `Scion.Trials` · **Assembly:** `Scion.Trials` · **Define:** `SCION_TRIALS`

```csharp
public static class NetClock
{
    public static double Time { get; }
    public static float DeltaTime { get; }
}
```

Continuous virtual time for libraries that keep their own clock; it never jumps back at `Clock.Begin`.

## See also

- [Deterministic runs](/scion/41664431a8f5/determinism/overview/)
- [Clock, screen and random](/scion/41664431a8f5/determinism/clock-and-screen/)
- [Particles, animators, UI and physics](/scion/41664431a8f5/determinism/control/)
- [Netcode clock](/scion/41664431a8f5/determinism/netclock/)

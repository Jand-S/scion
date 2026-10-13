---
title: "SCN301-SCN307"
description: "Determinism scan findings - redirected reads and creations, wall-clock reads, threads, unordered collections, hash-code seeds and Guids."
type: diagnostics
url: https://jand-s.github.io/scion/41664431a8f5/reference/diagnostics/determinism/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/diagnostics/determinism.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/determinism/diagnose.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/diagnose.md
  - https://jand-s.github.io/scion/41664431a8f5/determinism/clock-and-screen.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# SCN301-SCN307

The weaver scans every assembly that references `Scion.Runtime` when Unity compiles it and writes the findings to
`Library/Scion/determinism-<assembly>.json`. They never appear in the Unity console; `dotnet scion diagnose` reports
them with file and line, counts and a plan.

## Diagnostics

### SCN301

**Severity:** info · **Source:** scan

**Message:** `{0} reads {1}; redirected to {2} in test builds.`

**Cause.** A read of `Time.*`, `DateTime.Now`, `Screen.width` or similar that the test build already redirects.
Nothing to fix; it shows what the virtual clock covers.

### SCN302

**Severity:** info · **Source:** scan

**Message:** `{0} creates {1}; replaced by {2} in test builds.`

**Cause.** `new System.Random()` or `new WaitForSecondsRealtime(t)` that the test build replaces. Nothing to fix.

### SCN303

**Severity:** warning · **Source:** scan

**Message:** `{0} reads the wall clock ({1}), which is not redirected.` **Hint:** `Use Time or Clock, or mark the
method [WallClock] if it must be real time.`

**Cause.** `Stopwatch`, `Environment.TickCount`, `DateTime.Now` in a `[WallClock]`-free path not covered, or a native
time call.

**Fix.**

```csharp
// Before
turnStarted = Stopwatch.GetTimestamp();
// After
turnStarted = Time.timeAsDouble;
```

**Suppress.** `[WallClock]` on code that must measure real time.

### SCN304

**Severity:** warning · **Source:** scan

**Message:** `{0} uses threads or tasks ({1}); results may change between runs.` **Hint:** `Keep game logic on the
main thread.`

**Cause.** `Task.Run`, `Thread`, `Parallel`, or `async` continuations on the thread pool in game logic.

**Fix.** Run game logic on the main thread; keep threads for I/O and pure computations whose result does not depend on
timing.

**Suppress.** `#pragma warning disable SCN304` with a reason when the result is order-independent.

### SCN305

**Severity:** warning · **Source:** scan

**Message:** `{0} depends on an unordered collection or query ({1}).` **Hint:** `Sort first, or use a list or array.`

**Cause.** Iterating a `HashSet`, `Dictionary` or an unordered query and letting the order decide an outcome (the
first target, a tie-break).

**Fix.**

```csharp
// Before
var target = candidates.First();          // HashSet<Unit>
// After
var target = candidates.OrderBy(u => u.Id).First();
```

**Suppress.** When the result does not depend on order (a sum).

### SCN306

**Severity:** warning · **Source:** scan

**Message:** `{0} seeds randomness from GetHashCode ({1}).` **Hint:** `Use a fixed seed (recipe Seed or
Determinism.SeedFor).`

**Cause.** `new Random(name.GetHashCode())`: string hash codes change between processes in .NET.

**Fix.** Use a stable hash or `Determinism.SeedFor(path, order)`.

**Suppress.** Not recommended.

### SCN307

**Severity:** warning · **Source:** scan

**Message:** `{0} creates a Guid ({1}); it changes every run.` **Hint:** `Use a counter or a seeded id.`

**Cause.** `Guid.NewGuid()` in game logic or state.

**Fix.** Use a counter or an id derived from the seed.

**Suppress.** For ids that never enter state or output (telemetry session ids).

## See also

- [Find non-determinism](/scion/determinism/diagnose/)
- [diagnose](/scion/cli/diagnose/)
- [Clock, screen and random](/scion/determinism/clock-and-screen/)
- [Diagnostics](/scion/reference/diagnostics/)

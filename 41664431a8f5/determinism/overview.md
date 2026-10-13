---
title: "Deterministic runs"
description: "How the test build makes time, screen size and randomness identical across rounds and machines, so the same recipe gives the same hash and frames."
type: concept
url: https://jand-s.github.io/scion/41664431a8f5/determinism/overview/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/determinism/overview.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/determinism/clock-and-screen.md
  - https://jand-s.github.io/scion/41664431a8f5/determinism/verify.md
  - https://jand-s.github.io/scion/41664431a8f5/determinism/diagnose.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Deterministic runs

A trial is only useful if the same recipe gives the same result every time. In test builds the weaver redirects
every source of variation the game reads (time, wall clock, screen size, unseeded randomness) to Scion's virtual
versions, and the session resets them at the start of every recipe. The game's rules then produce the same state
hash on every round and machine, and the same frames, so a difference always means a change.

## How it works

### What is redirected

With `SCION_TRIALS`, the weaver rewrites reads in every assembly that references `Scion.Runtime`, and the reads only
in the assemblies listed in `trials.redirect_assemblies` of `ScionSettings.json`:

| The game reads | In a test build it reads |
|---|---|
| `Time.time`, `timeAsDouble`, `timeSinceLevelLoad(AsDouble)` | `Clock.Time`, ... (virtual, from 1000 s at each recipe) |
| `Time.unscaledTime(AsDouble)`, `unscaledDeltaTime` | `Clock.UnscaledTime`, ... |
| `Time.realtimeSinceStartup(AsDouble)` | `Clock.RealtimeSinceStartup` (= virtual unscaled time) |
| `DateTime.Now`, `UtcNow`, `Today` | `Clock.Now`, ... (from `Clock.Epoch`, 2000-01-01 by default; no time zone) |
| `Screen.width`, `Screen.height` | `VirtualScreen.Width`, `Height` (the capture size, also in `-batchmode`) |
| `new WaitForSecondsRealtime(t)` | `new VirtualWaitForSecondsRealtime(t)` |
| `new System.Random()` (no seed) | `new SeededRandom()` (seed from the recipe and creation order) |

`Time.deltaTime` needs no redirect: `Time.captureFramerate = recipe.Fps` makes every frame advance exactly `1/Fps`
of game time, however long it took to render.

### What the session resets per recipe

1. `UnityEngine.Random.InitState(recipe.Seed)`;
2. `Time.captureFramerate = recipe.Fps`;
3. `Clock.Begin(Clock.RecipeStart)`: virtual time restarts at 1000 s, so the position in the queue does not matter;
4. `SeededRandom.Reset(recipe.Seed)`: the n-th `new System.Random()` gets the same seed every round;
5. the determinism controls of `recipe.Control`: particle seeds, animator and UI Toolkit updates, physics stepping
   ([Particles, animators, UI and physics](/scion/41664431a8f5/determinism/control/)).

### What the game must do

- Seed its own generators from the recipe (`Seed`) and the run (`RunSeed`).
- Keep rules on the main thread, with ordered collections and integer math in the core (rules R4 and R5 of the
  [Scion Standard](/scion/41664431a8f5/standard/rules/)).
- Mark real-time code (profilers, timeouts) with `[WallClock]` so it keeps the real clock.

`dotnet scion diagnose` finds the rest: wall-clock reads, threads, unordered collections, `GetHashCode` seeds and
`Guid`s, with file and line ([Find non-determinism](/scion/41664431a8f5/determinism/diagnose/)).

### The state hash

The adapter's `StateHash` is a hash of the rules state after the recipe. It is the main determinism signal: rounds
must agree on it, baselines store it, the multiverse compares it per turn and co-op peers compare it per turn.

## Key terms

| Term | Meaning |
|---|---|
| Virtual clock | `Clock`: time that advances one capture step per frame, from 1000 s at each recipe. |
| Epoch | The virtual wall-clock start, `Clock.Epoch` (`-scion-epoch`). |
| Recipe seed | `TrialRecipe.Seed`: seeds Unity's random and `SeededRandom` per recipe. |
| Run seed | `TrialSession.RunSeed` (`--seed`): the game maps it to its world seed. |
| State hash | The adapter's hash of rules state; equal across rounds when deterministic. |
| Adoption level | 1 (grafts), 2 (trials), 3 (multiverse): what the project can use today ([Find non-determinism](/scion/41664431a8f5/determinism/diagnose/)). |

## Guarantees and limits

- Redirects exist only in test builds; final builds read the real clock and screen.
- Only `new` expressions in woven code are replaced: a class derived from `System.Random`, or one created by
  reflection, keeps its behavior; pass it a seed.
- Inside `FixedUpdate`, the virtual `Time.time` keeps the frame's value; fixed-step code should add up
  `Time.fixedDeltaTime`.
- Netcode keeps its own tick unless `SCION_NETCLOCK` is on ([Netcode clock](/scion/41664431a8f5/determinism/netclock/)).
- Pixels can still vary with GPU drivers across machines; images compare per platform.

## See also

- [Clock, screen and random](/scion/41664431a8f5/determinism/clock-and-screen/)
- [Verify determinism](/scion/41664431a8f5/determinism/verify/)
- [Find non-determinism](/scion/41664431a8f5/determinism/diagnose/)
- [Determinism API](/scion/41664431a8f5/reference/api/determinism/)

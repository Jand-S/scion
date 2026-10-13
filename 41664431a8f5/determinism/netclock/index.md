---
title: "Netcode clock"
description: "Put the Netcode for GameObjects tick on the virtual clock with SCION_NETCLOCK and netclock_assemblies, so co-op recipes give the same frames every round."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/determinism/netclock/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/determinism/netclock.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/coop.md
  - https://jand-s.github.io/scion/41664431a8f5/determinism/clock-and-screen.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/configuration.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Netcode clock

Netcode for GameObjects keeps its own time from the real clock, so a networked recipe can tick one frame earlier or
later between rounds: the state hash still agrees, but the frame count jitters by one. With the define
`SCION_NETCLOCK` and `netclock_assemblies` in `ScionSettings.json`, the weaver redirects the time reads of Netcode
to the virtual clock on the host, and frame counts become exact.

## When to use

- Turn it on when co-op or networked solo recipes report `frame_jitter` and you want frames to fail on difference
  (`Deterministic(frames: true)`).
- Leave it off for games without Netcode.

## Before you begin

- Netcode for GameObjects in the project.
- A test build workflow (`dotnet scion build test`).

## Steps

### 1. List Netcode's assembly

`ProjectSettings/ScionSettings.json`:

```json
{"schema_version":1,"trials":{"netclock_assemblies":["Unity.Netcode.Runtime"]}}
```

### 2. Add the define to test builds

Add `SCION_NETCLOCK` to the scripting defines of the test build profile, or pass it in your build script:

```csharp
ScionBuild.BuildTestPlayer(options with { extraScriptingDefines = new[] { "SCION_NETCLOCK" } });
```

### 3. Verify

```sh
dotnet scion build test
dotnet scion trial run CoopTurn --rounds 3 --check
```

`frame_jitter` in the output should be empty.

## Reference

| Item | Detail |
|---|---|
| What is redirected | Netcode's reads of `Time.realtimeSinceStartup`, `Time.unscaledTime`, `Time.unscaledDeltaTime` in the listed assemblies. |
| Where | Only on the host of a group, and in solo copies. Clients stay on the real clock and follow the host's ticks. |
| Continuity | The netclock never jumps back at `Clock.Begin`: it advances continuously across recipes, so connections survive the queue. |
| `NetClock` type | `Scion.Trials.NetClock.Time` and `DeltaTime`: the same continuous time, for libraries you cannot weave (a custom time provider). |
| Without the define | `netclock_assemblies` is ignored. |

### A time provider for libraries you cannot weave

```csharp
sealed class TrialsTimeProvider : IRealTimeProvider   // Netcode for GameObjects, host of a trials run
{
    public float RealTimeSinceStartup => (float)NetClock.Time;
    public float UnscaledTime => (float)NetClock.Time;
    public float UnscaledDeltaTime => NetClock.DeltaTime;
    public float DeltaTime => UnityEngine.Time.deltaTime;
    public float FixedDeltaTime => UnityEngine.Time.fixedDeltaTime;
}
```

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `SCN114` | compile error | A redirect target is missing in `Scion.Runtime` (a mismatched package). | Reinstall the package; CLI and package versions must match. |
| warning `frame_jitter` | 0 | Frame counts differ by one between rounds. | Turn on `SCION_NETCLOCK`. |

## Examples

### Fail co-op recipes on any frame difference

```csharp
protected override void Checks(TrialChecks c) => c.Deterministic(frames: true);
```

### Check the tick column

```sh
dotnet scion trial run CoopTurn --rounds 2 --check
grep tick .scion/runs/20261011-143200/q0/determinism.tsv
```

## See also

- [Co-op trials (1-4 players)](/scion/41664431a8f5/trials/coop/)
- [Clock, screen and random](/scion/41664431a8f5/determinism/clock-and-screen/)
- [Configuration](/scion/41664431a8f5/reference/configuration/)

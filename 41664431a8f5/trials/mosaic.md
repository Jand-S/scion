---
title: "Mosaic"
description: "Run K arenas in one process, each with its own camera and render texture, to compare variants side by side when memory limits parallel copies."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/trials/mosaic/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/trials/mosaic.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/parallel.md
  - https://jand-s.github.io/scion/41664431a8f5/determinism/diagnose.md
  - https://jand-s.github.io/scion/41664431a8f5/explore/multiverse.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Mosaic

The mosaic runs K **arenas** in one test-build process: K copies of the scene side by side, each with its own camera
rendering into its own texture, each playing one id. One process instead of K saves memory and startup, and the
output is a grid that shows the variants of one moment next to each other. This guide covers the requirements,
when to choose the mosaic over copies, `trial run --mosaic K` and the `IMosaicAdapter` interface.

## When to use

| Situation | Choose |
|---|---|
| Many different recipes, CPU-bound | Copies (`-j`) |
| A few variants of one moment, to look at side by side | Mosaic |
| Memory is the limit (big scenes, small machine) | Mosaic |
| The game has singletons or global state per scene | Copies, until `diagnose` shows no blockers |

## Before you begin

- The game can run several battles in one scene: no singletons for battle state, no global `Time.timeScale`, no
  static caches shared between battles.
- `dotnet scion diagnose` lists what blocks the mosaic under `singletons`.

## Steps

### 1. Check the blockers

```sh
dotnet scion diagnose
```

<!-- validate: cli/diagnose -->
```json
{"schema_version":1,"command":"diagnose","ok":true,"adoption_level":3,"assemblies":["Orchard.Core","Orchard.Runtime"],"counts":{"SCN301":12,"SCN302":3,"SCN303":0,"SCN304":0,"SCN305":1,"SCN306":0,"SCN307":0},"blockers":[],"singletons":["Orchard.Runtime.CameraRig.Current (Assets/Orchard/Runtime/CameraRig.cs:14)"],"plan":["Make CameraRig.Current per arena to enable the mosaic."],"adapter_skeleton":null,"report":"Library/Scion/diagnose.md","next_step":"fix the singletons listed, then: dotnet scion trial run --mosaic 4"}
```

### 2. Make the adapter arena-aware

```csharp
public sealed class OrchardAdapter : TrialAdapter<OrchardRecipe>, IMosaicAdapter
{
    public IEnumerator PrepareArena(int arena, TrialSession s)
    {
        yield return SceneLoader.LoadAdditive("Battle", offset: new Vector3(arena * 100, 0, 0));
    }

    protected override IEnumerator Run(OrchardRecipe r, TrialSession s)
    {
        var arena = s.Arena;                     // 0..K-1
        var battle = Battle.Start(setup, root: Arenas.Root(arena));
        // ... the same as a single run, through battle and arena-local objects ...
    }
}
```

`TrialSession.Arena` is the arena of the current recipe; `TrialSession.ForArena(i)` returns the session of
arena `i` from shared code.

### 3. Run

```sh
dotnet scion trial run SlashHit --variants base,heavy,snappy --mosaic 3
```

The run writes the usual per-id folders, plus `mosaic.png`: the `window.png` of every arena in one row each,
aligned on `IMPACT`.

## Reference

| Item | Detail |
|---|---|
| Arenas | `K` from 2 to 16; each plays one id at a time from the process's shard. |
| Cameras | One per arena, rendering to a texture at the capture size; nothing renders to the screen. |
| Clock | Shared: all arenas advance the same virtual frame together. |
| Hash | Per arena, from the adapter's `StateHash` while `Arena` is set. |
| Result | `result.json` has `arena`; `run.json` has `mosaic: K`. |
| Player flag | `-scion-mosaic K`. |

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `mosaic_unsupported` | 3 | The adapter does not implement `IMosaicAdapter`. | Implement it, or use copies. |
| recipe error `arena state leaked` | 1 | An arena's hash depends on another arena (a shared singleton). | Fix what `diagnose` lists under `singletons`. |

## Examples

### Four seeds of one moment

```sh
dotnet scion trial run KnightFalls --seeds 1-4 --mosaic 4
```

### The multiverse in a mosaic

```sh
dotnet scion multiverse run knight-dies-to-crit --at-turn 2 --variants base,heavy --mosaic
```

## See also

- [Parallel copies](/scion/trials/parallel/)
- [Find non-determinism](/scion/determinism/diagnose/)
- [The multiverse](/scion/explore/multiverse/)

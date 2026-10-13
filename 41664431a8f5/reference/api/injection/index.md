---
title: "Injection API"
description: "Injection and InjectReport - load draft graft DLLs into a running test build, restore promoted grafts, disable slots and read the result."
type: api
url: https://jand-s.github.io/scion/41664431a8f5/reference/api/injection/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/api/injection.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/grafts/injection.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/inject-report.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Injection API

`Injection` swaps the slots of a running test build (`SCION_SLOTS`). The CLI and the control server call it; use it
directly only in your own tools. Swaps requested during a recipe wait for the next boundary.

## Injection

**Namespace:** `Scion` · **Assembly:** `Scion.Injection` · **Define:** `SCION_SLOTS`

```csharp
public static class Injection
{
    public static int Version { get; }
    public static event Action<InjectReport> Applied;
    public static InjectReport Inject(string dllPath, bool isolate = false);
    public static void Restore();
    public static bool Disable(string slotId);
}
```

| Member | Meaning |
|---|---|
| `Version` | Loads applied so far; the `grafts` column of `results.tsv`. |
| `Applied` | Raised on the main thread after each applied load. |
| `Inject` | Loads the DLL and binds its grafts; merges by graft identity, or replaces with `isolate`. Refuses another `build_id` (SCN130) or ABI (SCN131). |
| `Restore` | Every slot back to the promoted grafts. |
| `Disable` | Empties one slot; returns false when the slot does not exist. |

Main thread only. Mono never unloads assemblies: each load stays in memory.

```csharp
#if SCION_SLOTS
var r = Injection.Inject(path);
Debug.Log(r.ToJson());
#endif
```

## InjectReport

**Namespace:** `Scion` · **Assembly:** `Scion.Injection` · **Define:** `SCION_SLOTS`

```csharp
public readonly struct InjectReport
{
    public int Version { get; }
    public bool Ok { get; }
    public bool Pending { get; }
    public string[] Bound { get; }
    public string[] Errors { get; }
    public bool Recycle { get; }
    public string ToJson();
}
```

`Errors` lines are `"SCNxxx: message"`. `Recycle` is true after 50 loads. JSON shape:
[InjectReport JSON](/scion/41664431a8f5/reference/files/inject-report/).

## See also

- [Live injection](/scion/41664431a8f5/grafts/injection/)
- [InjectReport JSON](/scion/41664431a8f5/reference/files/inject-report/)
- [Control protocol](/scion/41664431a8f5/reference/control-protocol/)

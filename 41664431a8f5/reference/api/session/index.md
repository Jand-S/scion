---
title: "Session API"
description: "The TrialSession class with every property and method signature, and the TrialRole struct for co-op hosts and clients."
type: api
url: https://jand-s.github.io/scion/41664431a8f5/reference/api/session/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/api/session.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/session.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/coop.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Session API

`TrialSession` is the MonoBehaviour that runs the queue; `TrialRole` names a process's role in a co-op group.
Behavior and examples are on [TrialSession](/scion/41664431a8f5/trials/session/); this page lists the signatures.

## TrialSession

**Namespace:** `Scion.Trials` · **Assembly:** `Scion.Trials` · **Define:** `SCION_TRIALS`

```csharp
public sealed partial class TrialSession : MonoBehaviour
{
    public static TrialSession Current { get; }
    public static event Action Boundary;

    public string OutputRoot { get; }
    public TrialRecipe Recipe { get; }
    public string RecipeDir { get; }
    public int Round { get; }
    public bool InRecipe { get; }
    public int RunSeed { get; }
    public IReadOnlyList<TrialRecipe> Recipes { get; }
    public TrialRole Role { get; }
    public int Arena { get; }
    public static TrialSession ForArena(int arena);

    public void BeginCut();
    public void EndCut();
    public void Mark(string label);
    public void MarkImpact();
    public void Still(string name);
    public void MaskRect(Rect screenPixels, string why);

    public void Fail(string reason);
    public void Check(bool ok, string what);
    public void Note(string line);

    public void Probe(string key, double value);
    public void Sensor(string name, Func<long> read);
    public void Event(string name, params (string Key, object Value)[] fields);

    public IEnumerator WaitUntil(Func<bool> cond, string what);
    public IEnumerator WaitSeconds(float gameSeconds);
    public IEnumerator RunQueue(string only, int rounds, string outDir, Action<int> done = null);
}
```

Main thread only; calls from other threads are ignored and logged as errors. `Arena` is 0 outside the mosaic.

## TrialRole

**Namespace:** `Scion.Trials` · **Assembly:** `Scion.Trials` · **Define:** `SCION_TRIALS`

```csharp
public readonly struct TrialRole : IEquatable<TrialRole>
{
    public static TrialRole Host { get; }
    public static TrialRole Client(int index);   // 1 to 3
    public bool IsHost { get; }
    public int Index { get; }                    // 0 = host
    public override string ToString();           // "host" or "client:1"
    public static bool TryParse(string text, out TrialRole role);
}
```

## See also

- [TrialSession](/scion/41664431a8f5/trials/session/)
- [Co-op trials (1-4 players)](/scion/41664431a8f5/trials/coop/)
- [Checks API](/scion/41664431a8f5/reference/api/checks/)

---
title: "Adapters API"
description: "ITrialAdapter, TrialAdapter of TRecipe, the TrialAdapter attribute, static trials and views, match capture and the mosaic adapter interface."
type: api
url: https://jand-s.github.io/scion/41664431a8f5/reference/api/adapters/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/api/adapters.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/adapters.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/static-trials.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/mosaic.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Adapters API

Adapters are the game's port into Trials. `TrialAdapter<TRecipe>` handles recipes of a type; `StaticTrial<TItem>`
needs no game adapter; `IMatchCapture` turns matches into recipes; `IMosaicAdapter` runs several arenas. All in
`Scion.Trials` under `SCION_TRIALS`.

## ITrialAdapter

**Namespace:** `Scion.Trials` · **Assembly:** `Scion.Trials` · **Define:** `SCION_TRIALS`

```csharp
public interface ITrialAdapter
{
    IEnumerator Prepare(TrialSession session);
    bool Accepts(TrialRecipe recipe);
    IEnumerator Run(TrialRecipe recipe, TrialSession session);
    IEnumerator Teardown(TrialRecipe recipe, TrialSession session);
    string StateHash { get; }
}
```

A plain implementation handles `TrialRecipe`, the least specific type. `Accepts` must not throw.

## TrialAdapter\<TRecipe\>

**Namespace:** `Scion.Trials` · **Assembly:** `Scion.Trials` · **Define:** `SCION_TRIALS`

```csharp
public abstract class TrialAdapter<TRecipe> : MonoBehaviour, ITrialAdapter where TRecipe : TrialRecipe
{
    public virtual IEnumerator Prepare(TrialSession s);
    protected abstract IEnumerator Run(TRecipe r, TrialSession s);
    protected virtual IEnumerator Teardown(TRecipe r, TrialSession s);
    public virtual string StateHash => null;
}
```

`Accepts` is `r is TRecipe`. The most derived `TRecipe` among accepting adapters wins (SCN203 on a tie).

## TrialAdapterAttribute

**Namespace:** `Scion.Trials` · **Assembly:** `Scion.Trials` · **Define:** `SCION_TRIALS`

```csharp
[AttributeUsage(AttributeTargets.Assembly, AllowMultiple = true)]
public sealed class TrialAdapterAttribute : Attribute
{
    public TrialAdapterAttribute(Type adapter);
    public Type Adapter { get; }
}
```

## StaticTrial

**Namespace:** `Scion.Trials` · **Assembly:** `Scion.Trials` · **Define:** `SCION_TRIALS`

```csharp
public abstract class StaticTrial : TrialRecipe
{
    public int SettleFrames { get; protected set; } = 3;
    public abstract IEnumerator Play(TrialSession s);
}
```

The non-generic base, so adapters can be `TrialAdapter<StaticTrial>`. The package registers `StaticTrialAdapter`,
which only calls `Play`.

## StaticTrial\<TItem\>

**Namespace:** `Scion.Trials` · **Assembly:** `Scion.Trials` · **Define:** `SCION_TRIALS`

```csharp
public abstract class StaticTrial<TItem> : StaticTrial
{
    protected abstract IEnumerable<TItem> Enumerate();
    protected virtual string ItemName(TItem item);
    protected abstract IEnumerator Show(TItem item, TrialSession s);
    protected virtual IEnumerator Clear(TItem item, TrialSession s);
    protected virtual IEnumerable<TrialView> Views(TItem item);
    protected virtual void Measure(TItem item, TrialSession s);
}
```

## TrialView

**Namespace:** `Scion.Trials` · **Assembly:** `Scion.Trials` · **Define:** `SCION_TRIALS`

```csharp
public readonly struct TrialView
{
    public static TrialView Screen { get; }
    public static TrialView Ui { get; }
    public static TrialView Top(Bounds bounds, float pxPerUnit, bool grid = false);
    public string Name { get; }
}
```

## IMatchCapture

**Namespace:** `Scion.Trials` · **Assembly:** `Scion.Trials` · **Define:** `SCION_TRIALS`

```csharp
public interface IMatchCapture
{
    bool CanCapture { get; }
    int Seed { get; }
    int Players { get; }
    string SaveSetup();
    IEnumerable<(int Turn, int Player, string Data)> Commands();
    IEnumerator Replay(MatchData match, TrialSession s);
    string StateHash { get; }
}
```

## MatchCaptureAttribute

**Namespace:** `Scion.Trials` · **Assembly:** `Scion.Trials` · **Define:** `SCION_TRIALS`

```csharp
[AttributeUsage(AttributeTargets.Assembly)]
public sealed class MatchCaptureAttribute : Attribute
{
    public MatchCaptureAttribute(Type capture);
    public KeyCode Key { get; set; } = KeyCode.F9;
}
```

## IMosaicAdapter

**Namespace:** `Scion.Trials` · **Assembly:** `Scion.Trials` · **Define:** `SCION_TRIALS`

```csharp
public interface IMosaicAdapter
{
    IEnumerator PrepareArena(int arena, TrialSession s);
}
```

Implemented by an adapter that can run arenas side by side; `TrialSession.Arena` tells `Run` which one it plays.

## See also

- [Write an adapter](/scion/41664431a8f5/trials/adapters/)
- [Static trials](/scion/41664431a8f5/trials/static-trials/)
- [Turn any match into a recipe](/scion/41664431a8f5/trials/captured-matches/)
- [Mosaic](/scion/41664431a8f5/trials/mosaic/)

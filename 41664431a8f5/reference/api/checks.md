---
title: "Checks API"
description: "TrialChecks for declared checks and gates, ITrialCheck and its attribute for custom checks, CheckResult, CheckStatus and FrameView."
type: api
url: https://jand-s.github.io/scion/41664431a8f5/reference/api/checks/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/api/checks.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/checks.md
  - https://jand-s.github.io/scion/41664431a8f5/evaluation/gates.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Checks API

Recipes declare checks with `TrialChecks`; games add checks for every recipe with `ITrialCheck`. Results are
`CheckResult` values. All in `Scion.Trials` under `SCION_TRIALS`.

## TrialChecks

**Namespace:** `Scion.Trials` · **Assembly:** `Scion.Trials` · **Define:** `SCION_TRIALS`

```csharp
public sealed class TrialChecks
{
    public TrialChecks NoErrors(params string[] allowRegex);
    public TrialChecks Deterministic(bool frames = false, bool pixels = true);
    public TrialChecks Marked(string label, int atLeast = 1);
    public TrialChecks Order(params string[] labels);
    public TrialChecks Frames(int min, int max);
    public TrialChecks ProbeIs(string key, Func<double, bool> ok, string why);
    public TrialChecks Gate(string id, params (string Name, object Value)[] options);
    public TrialChecks Skip(string checkId, string why);
}
```

Check ids: `log`, `assert`, `blank`, `magenta`, `px`, `determinism`, `marked:<label>`, `order`, `frames`,
`probe:<key>`, `gate:<id>`, plus custom ids. A wrong argument throws `ArgumentException` and fails the recipe.

## ITrialCheck

**Namespace:** `Scion.Trials` · **Assembly:** `Scion.Trials` · **Define:** `SCION_TRIALS`

```csharp
public interface ITrialCheck
{
    string Id { get; }
    void Begin(TrialRecipe recipe, TrialSession session);
    void OnFrame(in FrameView frame);
    void OnLog(string message, string stack, LogType type);
    CheckResult End();
}
```

One instance per process; main thread; `OnFrame` must not allocate.

## TrialCheckAttribute

**Namespace:** `Scion.Trials` · **Assembly:** `Scion.Trials` · **Define:** `SCION_TRIALS`

```csharp
[AttributeUsage(AttributeTargets.Assembly, AllowMultiple = true)]
public sealed class TrialCheckAttribute : Attribute
{
    public TrialCheckAttribute(Type check);
    public Type Check { get; }
}
```

## FrameView

**Namespace:** `Scion.Trials` · **Assembly:** `Scion.Trials` · **Define:** `SCION_TRIALS`

```csharp
public readonly ref struct FrameView
{
    public int Index { get; }
    public int Width { get; }
    public int Height { get; }
    public ReadOnlySpan<byte> Pixels { get; }   // bottom row first, 4 bytes per pixel
    public string Layout { get; }               // RGBA8, ARGB8 or BGRA8
    public ulong Px { get; }
    public double GameMs { get; }
}
```

## CheckStatus

**Namespace:** `Scion.Trials` · **Assembly:** `Scion.Trials` · **Define:** `SCION_TRIALS`

```csharp
public enum CheckStatus { Pass, Warn, Fail, Skipped }
```

## CheckResult

**Namespace:** `Scion.Trials` · **Assembly:** `Scion.Trials` · **Define:** `SCION_TRIALS`

```csharp
public readonly struct CheckResult
{
    public CheckStatus Status { get; }
    public string Reason { get; }
    public int Frame { get; }            // -1 = no frame
    public RectInt? Region { get; }      // pixels, origin bottom left
    public static CheckResult Pass();
    public static CheckResult Warn(string reason, int frame = -1, RectInt? region = null);
    public static CheckResult Fail(string reason, int frame = -1, RectInt? region = null);
}
```

`default(CheckResult)` fails the check. `Warn` and `Fail` need a non-blank reason.

## See also

- [Checks](/scion/41664431a8f5/trials/checks/)
- [Gates](/scion/41664431a8f5/evaluation/gates/)
- [Session API](/scion/41664431a8f5/reference/api/session/)

---
title: "Specs API"
description: "TrialSpec, SpecBuilder with balance, visual, judge, metamorphic and curve parts, the Gates ids and the Metamorphic properties."
type: api
url: https://jand-s.github.io/scion/41664431a8f5/reference/api/specs/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/api/specs.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/specs.md
  - https://jand-s.github.io/scion/41664431a8f5/explore/balance.md
  - https://jand-s.github.io/scion/41664431a8f5/explore/metamorphic.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Specs API

`Scion.Specs` is a NuGet package for specs projects (`specs.project` in `scion.json`). A spec is a `TrialSpec` that
defines its parts on a `SpecBuilder`; `trial run --spec` runs it. It references `Scion.Core` and runs balance and
metamorphic parts on the simulator in plain .NET.

## TrialSpec

**Namespace:** `Scion.Specs` · **Assembly:** `Scion.Specs` · **Define:** none

```csharp
public abstract class TrialSpec
{
    public abstract string Name { get; }
    public abstract string Target { get; }       // one sentence: what must be true
    public abstract void Define(SpecBuilder s);
}
```

## SpecBuilder

**Namespace:** `Scion.Specs` · **Assembly:** `Scion.Specs` · **Define:** none

```csharp
public sealed class SpecBuilder
{
    public BalancePart Balance(string name, int players, int matches);
    public VisualPart Visual(string recipe);
    public JudgePart Judge(string recipe, string head);
    public void Metamorphic(Metamorphic property, int matches, int players = 1);
    public void Metamorphic(string name, Func<ISimulation, int, bool> property, int matches);
    public CurvePart Curve(string name, int levels, int players, int matches);
}

public sealed class BalancePart { public BalancePart Expect(string metric, double? min = null, double? max = null); }
public sealed class VisualPart
{
    public VisualPart Expect(string metric, double? min = null, double? max = null);
    public VisualPart Gate(string id, params (string Name, object Value)[] options);
}
public sealed class JudgePart { public JudgePart Expect(string answer, double? min = null, double? max = null); }
public sealed class CurvePart
{
    public CurvePart ExpectAt(int level, string metric, double? min = null, double? max = null);
    public CurvePart Monotonic(string metric, bool decreasing);
    public CurvePart MaxStep(string metric, double step);
}
```

| Part | Metrics |
|---|---|
| `Balance`, `Curve` | `ISimulation.Metrics`: `win_rate`, `turns_mean` and the game's own. |
| `Visual` | Motion metrics (`hit_stop_ms`, `shake_peak_px`, `flash_ms`...) and the recipe's probes. |
| `Judge` | `score`, `noul`, or a choice's probability. |

## Gates

**Namespace:** `Scion.Specs` (also `Scion.Trials`) · **Assembly:** `Scion.Specs` · **Define:** none

```csharp
public static class Gates
{
    public const string TextOverflow = "text_overflow";
    public const string UiGrid = "ui_grid";
    public const string TextContrast = "text_contrast";
    public const string Photosensitivity = "photosensitivity";
    public const string ImageRegression = "image_regression";
    public const string TelemetryVsPixel = "telemetry_vs_pixel";
    public const string NothingChanged = "nothing_changed";
    public const string Palette = "palette";
}
```

## Metamorphic

**Namespace:** `Scion.Specs` · **Assembly:** `Scion.Specs` · **Define:** none

```csharp
public enum Metamorphic { Mirror, Permute, VisualToggle, SaveLoad }
```

## See also

- [Specs](/scion/41664431a8f5/trials/specs/)
- [Balance with the simulator](/scion/41664431a8f5/explore/balance/)
- [Metamorphic tests](/scion/41664431a8f5/explore/metamorphic/)
- [Gates](/scion/41664431a8f5/evaluation/gates/)

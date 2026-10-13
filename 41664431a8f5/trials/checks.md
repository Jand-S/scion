---
title: "Checks"
description: "Fail recipes with built-in checks, declared TrialChecks, soft Check calls and your own ITrialCheck, and allow known log errors."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/trials/checks/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/trials/checks.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/session.md
  - https://jand-s.github.io/scion/41664431a8f5/evaluation/gates.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/checks.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Checks

A check is a rule a recipe must pass. Built-in checks run on every recipe (no errors in the log, no blank or magenta
frames, the same hash every round); a recipe declares more in `Checks(TrialChecks c)`; the adapter adds soft checks
with `session.Check`; the game adds checks for every recipe with `ITrialCheck`. A failed check fails the recipe at
its end, without stopping it, and its id and reason go to `result.json` and `results.tsv`.

## When to use

- Use `Fail` when going on makes no sense (the battle never started); use `Check` or declared checks for everything
  else, so one run reports every problem.
- Use `ProbeIs` for numbers the rules produce (damage, HP, turns).
- Use an `ITrialCheck` for a rule about every frame of every recipe (no black frames, a HUD element always visible).

## Steps

### 1. Declare checks in the recipe

```csharp
protected override void Checks(TrialChecks c) =>
    c.Marked("IMPACT")
     .Order("IMPACT", "unit_down")
     .Frames(20, 90)
     .ProbeIs("damage", d => d >= 10, "a critical slash deals at least 10")
     .NoErrors("^Shader warning in 'Orchard/Water'")
     .Skip("px", "leaf particles drift between rounds");
```

### 2. Add soft checks in the adapter

```csharp
s.Check(battle.State.Unit(1).Hp < hp, "the slash lowers the defender's HP");
```

### 3. Allow known errors for the whole project

`scion.json`:

```json
{"schema_version":1,"log_allow":["^The referenced script \\(Unknown\\) on this Behaviour is missing"]}
```

The CLI passes them to every copy with `-scion-log-allow <file>`.

## Reference

### Built-in checks

| Id | Fails when |
|---|---|
| `log` | An Error, Exception or Assert is logged between the start of `Run` and the end of the recipe. `[Scion]` lines never count. |
| `assert` | `TrialSession.Check(false, ...)` was called. |
| `blank` | Every frame of the cut is one color (99 % or more). Some blank frames only warn. |
| `magenta` | A frame has 0.2 % or more of Unity's missing-shader magenta. |
| `determinism` | With 2 or more rounds in the process: the state hash differs (fails), or the frame count differs by one (warns) or more (fails). |
| `px` | With 2 or more rounds: the pixel hash differs (fails; set `Deterministic(pixels: false)` to warn only). |
| `gate:<id>` | An enabled gate fails ([Gates](/scion/41664431a8f5/evaluation/gates/)). |

### TrialChecks

| Method | Check id | Meaning |
|---|---|---|
| `NoErrors(params string[] allowRegex)` | `log` | Each .NET regex allows matching errors. Calls add up. |
| `Deterministic(bool frames = false, bool pixels = true)` | `determinism`, `px` | Round comparison with any number of rounds; `frames: true` fails on one frame of difference. |
| `Marked(string label, int atLeast = 1)` | `marked:<label>` | The label was marked at least `atLeast` times in the cut. |
| `Order(params string[] labels)` | `order` | The labels were marked in this order (same frame or later). |
| `Frames(int min, int max)` | `frames` | The cut has `min` to `max` frames. |
| `ProbeIs(string key, Func<double, bool> ok, string why)` | `probe:<key>` | The last value of the probe passes `ok`. A key never probed fails. |
| `Gate(string id, params (string, object)[] options)` | `gate:<id>` | Enables a gate for this recipe. |
| `Skip(string checkId, string why)` | | Turns a check off for this recipe; shown as `skipped` with `why`. |

When a base recipe declares checks and a subclass adds more, call `base.Checks(c)`.

### ITrialCheck

```csharp
[assembly: TrialCheck(typeof(Orchard.Trials.HudAlwaysVisible))]

public sealed class HudAlwaysVisible : ITrialCheck
{
    int missing;
    public string Id => "hud-visible";
    public void Begin(TrialRecipe recipe, TrialSession session) => missing = -1;
    public void OnFrame(in FrameView frame)
    {
        if (missing < 0 && !Hud.Current.IsVisible) missing = frame.Index;
    }
    public void OnLog(string message, string stack, LogType type) { }
    public CheckResult End() => missing < 0 ? CheckResult.Pass() : CheckResult.Fail("HUD hidden", missing);
}
```

- One instance per process; `Begin` for each (recipe, round), then `OnFrame` per captured frame and `OnLog` per log
  line, then `End`. All on the main thread.
- `Id`: unique, no spaces, commas or `=`, not a built-in id.
- `FrameView`: `Index`, `Width`, `Height`, `Pixels` (bottom row first), `Layout`, `Px`, `GameMs`. Valid only during
  the call; keep `OnFrame` allocation-free.
- `CheckResult.Pass()`, `Warn(reason, frame, region)`, `Fail(reason, frame, region)`; `Region` in pixels, origin at
  the bottom left. `CheckStatus` is `Pass`, `Warn`, `Fail` or `Skipped`.

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| check `fail` | 1 | A check failed. | Read `checks[]` in `result.json`: `id`, `reason`, `frame`, `region`. |
| invalid check | 3 | An `ITrialCheck` type or id is invalid. | Fix the id or the constructor. |

## Examples

### Every recipe ends with no unit stuck mid-animation

```csharp
public CheckResult End() => UnitView.AnyAnimating ? CheckResult.Warn("a unit is still animating") : CheckResult.Pass();
```

### A recipe that must fail, for testing the checks

```csharp
public sealed class SlashMisses : OrchardRecipe
{
    public SlashMisses() { Explicit = true; Distance = 5; }
    protected override void Checks(TrialChecks c) => c.ProbeIs("damage", d => d > 0, "expected to fail: out of range");
}
```

## See also

- [TrialSession](/scion/41664431a8f5/trials/session/)
- [Gates](/scion/41664431a8f5/evaluation/gates/)
- [Checks API](/scion/41664431a8f5/reference/api/checks/)

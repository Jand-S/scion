---
title: "Call and Call<T>"
description: "Use ref Call and ref Call<T> to skip a target, return early, change its result and pass state from Before to After with Tag."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/grafts/call-context/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/grafts/call-context.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/grafts/write.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/grafts.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/diagnostics/grafts.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Call and Call<T>

A graft may take one extra parameter, always the last, that gives it the per-call context: `ref Call` for targets
that return `void`, `ref Call<T>` for targets that return `T`. With it a graft reads `Skipped`, skips the target with
`Skip()`, returns early with `Return(value)`, reads or replaces the result through `Result`, and hands state from a
`[Before]` to an `[After]` of the same call with `Tag`. The weaver checks the rules as SCN108, SCN109 and SCN111.

## When to use

- Use `Skip()` or `Return()` to suppress or replace presentation: skip a banner during a replay, cut an animation
  short, return a shorter duration.
- Use `Result` in an `[After]` to adjust what the target returns (a duration, a color, a position).
- Use `Tag` to measure or correlate something between the start and the end of one call.
- Do not use any of these to change rules. Skipping a rule method in a graft desynchronizes co-op peers; the analyzer
  reports writes to rule state as SCN003.

## Steps

### 1. Declare the context as the last parameter

| Target returns | Last parameter | Members |
|---|---|---|
| `void` | `ref Call c` | `Skipped`, `Skip()`, `Tag` |
| `T` | `ref Call<T> c`, with `T` equal to the return type | `Skipped`, `Result`, `Return(T)`, `Tag` |

It must be passed by `ref`; a `Call` by value, `in` or `out` is a signature error
([SCN104](/scion/41664431a8f5/reference/diagnostics/grafts/#scn104)). A `T` that differs from the return type, or `Call` on a
target that returns a value, is [SCN109](/scion/41664431a8f5/reference/diagnostics/grafts/#scn109).

### 2. Skip a void target

Only a `[Before]` that is not an observer may skip:

```csharp
[Before(typeof(Hud), nameof(Hud.ShowTurn))]
public static void Before(Hud hud, int turn, ref Call c)
{
    if (ReplayMode.Active) c.Skip();   // the body and later befores do not run; afters do
}
```

### 3. Return early with a value

`Return(value)` sets `Result` and skips:

```csharp
[Before(typeof(HitPresenter), nameof(HitPresenter.Play))]
public static void Before(HitPresenter presenter, HitEvent e, ref Call<float> c)
{
    if (e.Amount == 0) c.Return(0f);   // a miss presents nothing and takes no time
}
```

### 4. Change the result in an After

```csharp
[After(typeof(HitPresenter), nameof(HitPresenter.Play))]
public static void After(HitPresenter presenter, HitEvent e, ref Call<float> c)
{
    if (e.Crit) c.Result += 0.06f;
}
```

In an `[After]`, `Result` is what the target returns now, and the graft may change it; the next after sees the
changed value. In a `[Before]`, `Result` is the value returned if the target is skipped.

### 5. Hand state from Before to After

`Tag` is a `long`, 0 at the start of every call, shared by every graft of that call:

```csharp
public static class HitTiming
{
    [Before(typeof(HitPresenter), nameof(HitPresenter.Play))]
    public static void Before(HitPresenter presenter, ref Call<float> c) => c.Tag = Stopwatch.GetTimestamp();

    [After(typeof(HitPresenter), nameof(HitPresenter.Play))]
    public static void After(HitPresenter presenter, ref Call<float> c) =>
        TrialSession.Current?.Probe("hit_present_ticks", Stopwatch.GetTimestamp() - c.Tag);
}
```

When two or more befores, or two or more afters, of one anchor use `Tag`, the weaver warns
([SCN111](/scion/41664431a8f5/reference/diagnostics/grafts/#scn111)): let one pair own it, or keep other state in your own fields.

## Reference

### Order in one call

| Step | Who runs | Can skip |
|---|---|---|
| 1 | Observers (`[Before(..., Observe = true)]`), by `Priority` | no |
| 2 | Other befores, by `Priority`; the first that skips stops the rest of this step and step 3 | yes |
| 3 | The target's body | |
| 4 | Afters, by `Priority`, with `Skipped` set if step 2 skipped | no |

If the body throws, step 4 does not run. The context lives on the anchor's stack: nothing allocates.

### Members

| Member | `Call` | `Call<T>` | Meaning |
|---|---|---|---|
| `Skipped` | yes | yes | True after a before skipped the call. |
| `Skip()` | yes | | Skip the rest of the befores and the body. Non-observer befores only (SCN108). |
| `Result` | | yes | After: the current return value, settable. Before: the value returned on skip. |
| `Return(T value)` | | yes | `Result = value` and skip. Non-observer befores only (SCN108). |
| `Tag` | yes | yes | A `long` shared by every graft of one call; 0 at the start. |
| `Call(T result)` constructor | | yes | For tests outside Unity: a context that holds a result. |

### Cost

Measured on an Apple M5: in IL2CPP call mode, a graft without context adds nothing measurable, a `[Before]` with
`ref Call` adds about 0.5 ns and an `[After]` with `ref Call<float>` about 0.2 ns. In Mono slot mode (test builds),
the same cases cost about 2 to 7 ns; prefer `int` over `float` results in hot paths of test builds.

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `SCN104` | compile error | `Call` not passed by `ref`, or not the last parameter. | `ref Call c` or `ref Call<T> c`, last. |
| `SCN108` | compile error | `Skip()` or `Return()` in an `[After]` or an observer. | Use a non-observer `[Before]`, or only read `c.Skipped`. |
| `SCN109` | compile error | `Call<T>` with the wrong `T`, or `Call` on a non-void target. | Match the target's return type. |
| `SCN111` | compile warning | Several befores or afters of one anchor use `Tag`. | Let one graft own `Tag`. |

## Examples

### Skip a banner only in test runs

```csharp
[Before(typeof(Hud), nameof(Hud.ShowTurn))]
public static void Before(Hud hud, int turn, ref Call c)
{
#if SCION_TRIALS
    if (TrialSession.Current?.Recipe?.Tags.Contains("no-banner") == true) c.Skip();
#endif
}
```

### Test the result change without Unity

```csharp
var c = new Call<float>(0.5f);
CritShake.After(null, new HitEvent(attacker: 0, target: 1, amount: 9, crit: true), ref c);
Assert.Equal(0.56f, c.Result, 3);
```

See [Test grafts without Unity](/scion/41664431a8f5/grafts/testing/).

## See also

- [Write a graft](/scion/41664431a8f5/grafts/write/)
- [Grafts API](/scion/41664431a8f5/reference/api/grafts/)
- [Graft diagnostics](/scion/41664431a8f5/reference/diagnostics/grafts/)

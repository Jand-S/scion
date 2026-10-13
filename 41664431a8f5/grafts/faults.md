---
title: "When a graft throws"
description: "How slot mode catches a throwing graft, disables only that graft and fails the recipe, and how call mode lets the exception propagate."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/grafts/faults/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/grafts/faults.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/grafts/build-modes.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/grafts.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/checks.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# When a graft throws

A graft is code, and code throws. In slot mode (test builds) the dispatcher wraps each graft in its own `try/catch`:
the throwing graft is disabled alone, the exception goes to `Faults.Report`, the game keeps running and the current
recipe fails. In call mode (final builds) nothing is caught: the exception propagates like any other. This guide
covers both, the `Faults.Reported` event and the `fault` line in `telemetry.jsonl`.

## When to use

- Read this when a recipe fails with `graft <source> threw <Type>: <message>`.
- Subscribe to `Faults.Reported` when your own tool (a debug overlay, a crash reporter in test builds) must know.

## Steps

### 1. Read the failure

```text
SlashHit r1: graft Orchard.Grafts.CritShake.After threw NullReferenceException: Object reference not set to an instance of an object.
```

The recipe's `telemetry.jsonl` has the line:

```json
{"f":14,"t":1.467,"kind":"fault","source":"Orchard.Grafts.CritShake.After","type":"NullReferenceException","msg":"Object reference not set to an instance of an object."}
```

and `log.txt` has the stack.

### 2. Fix and inject again

A disabled graft comes back only when its slot is injected or restored:

```sh
dotnet scion graft inject --to all
```

### 3. Listen from your own code

```csharp
Faults.Reported += (source, e) => DebugOverlay.Show($"{source}: {e.Message}");
```

## Reference

| | Slot mode (test build) | Call mode (final build) |
|---|---|---|
| A graft throws | Caught per graft | Propagates to the caller |
| Other grafts of the anchor | Keep running | Do not run (the exception unwinds) |
| The anchor's body | Runs | Does not run if a before threw |
| The graft afterwards | Disabled for the rest of the process | Runs again on the next call |
| Report | `Faults.Report(source, e)`: a `[Scion]` warning and `Faults.Reported` | None |
| Trials | The recipe fails; `fault` in telemetry | Not applicable |

`source` is the graft's full name as `Namespace.Type.Method` (`+` between nested types). Handlers run on the thread
that reported; an exception thrown by a handler is caught and logged. `Faults.Report` is public so your own
dispatchers can use the same channel.

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| recipe error `graft <source> threw <Type>` | 1 | A graft threw during the recipe. | Fix the graft; read `log.txt` for the stack. |

## Examples

### Count faults in a soak test

```csharp
int faults = 0;
Faults.Reported += (_, _) => faults++;
// ... play ...
TrialSession.Current?.Check(faults == 0, "no graft faults");
```

### Report a fault from your own code

```csharp
try { RunCustomEffect(); }
catch (Exception e) { Faults.Report("Orchard.Presentation.CustomEffect", e); }
```

## See also

- [Grafts API](/scion/reference/api/grafts/)
- [Build modes](/scion/grafts/build-modes/)
- [telemetry.jsonl](/scion/reference/files/telemetry-jsonl/)

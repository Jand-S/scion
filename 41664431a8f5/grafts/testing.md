---
title: "Test grafts without Unity"
description: "Unit-test grafts with dotnet test using the Scion.Testing package, FxRecorder and a Call constructed with a result."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/grafts/testing/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/grafts/testing.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/testing.md
  - https://jand-s.github.io/scion/41664431a8f5/grafts/fx.md
  - https://jand-s.github.io/scion/41664431a8f5/grafts/call-context.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Test grafts without Unity

A graft is a static method, so it can be tested like one: call it with arguments and a `Call<T>`, and record its
effects with `FxRecorder`. The `Scion.Testing` NuGet package brings `Scion.Runtime` compiled for .NET, `FxRecorder`
and `FxCall`, so a plain `dotnet test` project tests grafts in milliseconds, without Unity and without a license.

## When to use

- Use unit tests for the graft's logic: which effects it calls, with which arguments, and how it changes the result.
- Use trials for what the player sees: timing, pixels, the state hash.

## Before you begin

- .NET 10 SDK.
- The grafts and the types they use compile outside Unity (only `UnityEngine` value types such as `Vector3` and
  `Color` are provided by `Scion.Testing`). Grafts that read a `MonoBehaviour` take `null` as the instance in tests.

## Steps

### 1. Create the test project

```sh
dotnet new xunit -o Tests/Orchard.Grafts.Tests
dotnet add Tests/Orchard.Grafts.Tests package Scion.Testing --version 1.0.0
```

Link the graft sources into the project:

```xml
<ItemGroup>
  <Compile Include="../../Assets/Orchard/Grafts/*.cs" />
  <Compile Include="../../Assets/Orchard/Core/HitEvent.cs" />
</ItemGroup>
```

### 2. Write the test

```csharp
using Orchard.Core;
using Orchard.Grafts;
using Scion;
using Scion.Testing;
using Xunit;

public class CritShakeTests
{
    [Fact]
    public void Crit_shakes_and_lengthens_the_hit()
    {
        var fx = new FxRecorder();
        Fx.Sink = fx;
        var c = new Call<float>(0.5f);

        CritShake.After(null, new HitEvent(attacker: 0, target: 1, amount: 9, crit: true), ref c);

        Assert.Equal(0.56f, c.Result, 3);
        Assert.Contains(fx.Calls, x => x.Verb == "Shake" && (float)x.Args[0] == CritShake.Strength);
    }
}
```

### 3. Run it

```sh
dotnet test Tests/Orchard.Grafts.Tests
```

## Reference

| Member | Use |
|---|---|
| `new Call<T>(T result)` | A context whose `Result` starts at `result`, as the target returned it. |
| `new Call()` | A context for void targets. `Skipped` reads false until a graft calls `Skip()`. |
| `FxRecorder.Calls` | Every `Fx` call in order: `Verb`, `Args`, `Source`. |
| `FxRecorder.Clear()` | Forget the recorded calls. |
| `Clock.Advance(delta)` | Move the virtual clock in tests that read `Time`. |

The weaver does not run in `dotnet test`: the test calls the graft method directly, so the order between grafts and
`Priority` are not exercised. Trials cover that.

Test assemblies that reference `Scion.Testing` are exempt from SCN110, so tests may call grafts.

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `SCN110` | compile error | A non-test assembly calls a graft. | Reference `Scion.Testing` in the test project, or move the call into a test. |

## Examples

### A skip in a Before

```csharp
var c = new Call();
ReplayMode.Active = true;
TurnBanner.Before(null, 3, ref c);
Assert.True(c.Skipped);
```

### No effect on a normal hit

```csharp
var fx = new FxRecorder(); Fx.Sink = fx;
var c = new Call<float>(0.5f);
CritShake.After(null, new HitEvent(0, 1, 4, crit: false), ref c);
Assert.Empty(fx.Calls);
Assert.Equal(0.5f, c.Result);
```

## See also

- [Testing API](/scion/41664431a8f5/reference/api/testing/)
- [Effects with Fx](/scion/41664431a8f5/grafts/fx/)
- [Call and Call\<T\>](/scion/41664431a8f5/grafts/call-context/)

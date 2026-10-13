---
title: "Testing API"
description: "The Scion.Testing NuGet package - Scion.Runtime for plain .NET, FxRecorder and FxCall - for unit tests of grafts without Unity."
type: api
url: https://jand-s.github.io/scion/41664431a8f5/reference/api/testing/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/api/testing.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/grafts/testing.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/fx.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Testing API

`Scion.Testing` is a NuGet package for `dotnet test` projects. It brings `Scion.Runtime` compiled for .NET (so
`Call<T>`, `Fx` and the attributes work outside Unity), Unity value types used by grafts (`Vector3`, `Color`), the
Scion analyzers, and a recorder for `Fx` calls.

## FxRecorder

**Namespace:** `Scion.Testing` · **Assembly:** `Scion.Testing` · **Define:** none

```csharp
public sealed class FxRecorder : IFxSink
{
    public IReadOnlyList<FxCall> Calls { get; }
    public void Clear();
}
```

Set `Fx.Sink = new FxRecorder()` before calling a graft; every verb is appended to `Calls` with its arguments and
`Fx.Source`.

## FxCall

**Namespace:** `Scion.Testing` · **Assembly:** `Scion.Testing` · **Define:** none

```csharp
public readonly struct FxCall
{
    public string Verb { get; }       // "Sound", "Shake", ...
    public object[] Args { get; }
    public string Source { get; }
}
```

## The package

```sh
dotnet add Tests/Orchard.Grafts.Tests package Scion.Testing --version 1.0.0
```

Projects that reference `Scion.Testing` are test assemblies: SCN110 does not apply to them.

## See also

- [Test grafts without Unity](/scion/41664431a8f5/grafts/testing/)
- [Fx API](/scion/41664431a8f5/reference/api/fx/)

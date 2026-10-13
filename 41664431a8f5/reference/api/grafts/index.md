---
title: "Grafts API"
description: "The attributes and types grafts use - Before, After, Anchor, AnchorScope, Call, Call of T, Faults, Anchors, WallClock and ScionPromoted."
type: api
url: https://jand-s.github.io/scion/41664431a8f5/reference/api/grafts/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/api/grafts.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/grafts/write.md
  - https://jand-s.github.io/scion/41664431a8f5/grafts/call-context.md
  - https://jand-s.github.io/scion/41664431a8f5/grafts/anchors.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Grafts API

These types exist in every build (namespace `Scion`, assembly `Scion.Runtime`), except `ScionPromotedAttribute`
(`Scion.Injection`, `SCION_SLOTS`). The weaver reads the attributes at compile time; `Call` and `Call<T>` live on the
anchor's stack at run time.

## BeforeAttribute

**Namespace:** `Scion` · **Assembly:** `Scion.Runtime` · **Define:** none

```csharp
[AttributeUsage(AttributeTargets.Method, AllowMultiple = true)]
public sealed class BeforeAttribute : Attribute
{
    public BeforeAttribute(Type target, string method);
    public Type Target { get; }
    public string Method { get; }
    public int Priority { get; set; }
    public bool Observe { get; set; }
}
```

| Member | Meaning | Default |
|---|---|---|
| `Target`, `Method` | The anchor: a method declared on `Target`. | |
| `Priority` | Lower runs first; ties by the graft's full name. | 0 |
| `Observe` | Observer: runs first, always, cannot skip. | false |

## AfterAttribute

**Namespace:** `Scion` · **Assembly:** `Scion.Runtime` · **Define:** none

```csharp
[AttributeUsage(AttributeTargets.Method, AllowMultiple = true)]
public sealed class AfterAttribute : Attribute
{
    public AfterAttribute(Type target, string method);
    public Type Target { get; }
    public string Method { get; }
    public int Priority { get; set; }
}
```

Runs at every return of the target, also after a skip; not after an exception.

## AnchorAttribute

**Namespace:** `Scion` · **Assembly:** `Scion.Runtime` · **Define:** none

```csharp
[AttributeUsage(AttributeTargets.Method)]
public sealed class AnchorAttribute : Attribute
{
    public string Id { get; set; }
    public string Doc { get; set; }
}
```

Metadata for tools (`anchor list`, the manifest); the weaver ignores it.

## AnchorScopeAttribute

**Namespace:** `Scion` · **Assembly:** `Scion.Runtime` · **Define:** none

```csharp
[AttributeUsage(AttributeTargets.Assembly, AllowMultiple = true)]
public sealed class AnchorScopeAttribute : Attribute
{
    public AnchorScopeAttribute(Type target);
    public Type Target { get; }
    public string[] Members { get; set; }   // null = every eligible method
}
```

Gives slots to eligible methods in test builds; ignored in final builds. Must be in the assembly that defines
`Target` (SCN107).

## Call

**Namespace:** `Scion` · **Assembly:** `Scion.Runtime` · **Define:** none

```csharp
public struct Call
{
    public bool Skipped { get; }
    public void Skip();
    public long Tag { get; set; }
}
```

The last parameter of a graft on a void target, as `ref Call`. `Skip` only in a non-observer `[Before]` (SCN108).

## Call\<T\>

**Namespace:** `Scion` · **Assembly:** `Scion.Runtime` · **Define:** none

```csharp
public struct Call<T>
{
    public Call(T result);
    public bool Skipped { get; }
    public T Result { get; set; }
    public void Return(T value);
    public long Tag { get; set; }
}
```

`T` must be the target's return type (SCN109). `Result` in an `[After]` is the current return value; in a `[Before]`,
the value returned if skipped. The constructor is for tests.

## Faults

**Namespace:** `Scion` · **Assembly:** `Scion.Runtime` · **Define:** none

```csharp
public static class Faults
{
    public static event Action<string, Exception> Reported;
    public static void Report(string source, Exception e);
}
```

The slot dispatcher reports a throwing graft here with `source = "Namespace.Type.Method"` and disables it. Handlers
run on the reporting thread; exceptions in handlers are caught and logged.

## Anchors

**Namespace:** `Scion` · **Assembly:** `Scion.Runtime` · **Define:** none

```csharp
public static class Anchors
{
    public const int AbiVersion = 1;
    public const string GeneratedType = "__ScionGenerated";
    public const string RestoreMethod = "__Restore";
    public static string Id(bool after, string typeFullName, string method, string[] parameterTypeNames);
}
```

`Id` computes the anchor id the weaver and injection use: `A__` or `B__`, then the type, method and parameter simple
names with every character outside `[A-Za-z0-9_]` replaced by `_`.

## WallClockAttribute

**Namespace:** `Scion` · **Assembly:** `Scion.Runtime` · **Define:** none

```csharp
[AttributeUsage(AttributeTargets.Method | AttributeTargets.Class | AttributeTargets.Struct)]
public sealed class WallClockAttribute : Attribute { }
```

No time, screen or random redirect inside the marked code (and its generated code and nested types).

## ScionPromotedAttribute

**Namespace:** `Scion` · **Assembly:** `Scion.Injection` · **Define:** `SCION_SLOTS`

```csharp
[AttributeUsage(AttributeTargets.Assembly, AllowMultiple = true)]
public sealed class ScionPromotedAttribute : Attribute
{
    public ScionPromotedAttribute(string id, string graft, int priority, string wrapper);
}
```

Emitted by the weaver in slot mode, one per promoted graft, so injection can merge by identity and `anchor list` can
show promoted grafts. Never written by hand.

## See also

- [Write a graft](/scion/41664431a8f5/grafts/write/)
- [Call and Call\<T\>](/scion/41664431a8f5/grafts/call-context/)
- [Anchors](/scion/41664431a8f5/grafts/anchors/)
- [When a graft throws](/scion/41664431a8f5/grafts/faults/)

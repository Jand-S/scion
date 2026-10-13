---
title: "Core API"
description: "The engine-free rules API - RuleModifier and Stage, Tunable and Probe attributes, ISimulation with bots and capabilities, and Tunables."
type: api
url: https://jand-s.github.io/scion/41664431a8f5/reference/api/core/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/api/core.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/grafts/visual-only.md
  - https://jand-s.github.io/scion/41664431a8f5/standard/rules.md
  - https://jand-s.github.io/scion/41664431a8f5/explore/multiverse.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Core API

`Scion.Core` is a small assembly without engine references, for the game's rules core. It gives rules an explicit
extension point (`RuleModifier<T>`), declares tunables and probes, and lets Scion run the rules outside Unity
(`ISimulation`) for balance specs, the multiverse, metamorphic tests and the hunter. It ships in the package and on
NuGet with the same version.

## RuleModifier\<T\>

**Namespace:** `Scion.Core` · **Assembly:** `Scion.Core` · **Define:** none

```csharp
public abstract class RuleModifier<T> where T : struct
{
    public abstract Stage Stage { get; }
    public virtual int Priority => 0;
    public abstract void Apply(ref T context);
}

public sealed class ModifierSet<T> where T : struct
{
    public void Add(RuleModifier<T> modifier);
    public bool Remove(RuleModifier<T> modifier);
    public void Apply(ref T context);           // by Stage, then Priority, then type full name
    public IReadOnlyList<RuleModifier<T>> All { get; }
}
```

## Stage

**Namespace:** `Scion.Core` · **Assembly:** `Scion.Core` · **Define:** none

```csharp
public enum Stage { Base = 1, Add = 2, Mul = 3, Clamp = 4, Final = 5 }
```

## TunableAttribute

**Namespace:** `Scion.Core` · **Assembly:** `Scion.Core` · **Define:** none

```csharp
[AttributeUsage(AttributeTargets.Field)]
public sealed class TunableAttribute : Attribute
{
    public TunableAttribute();                        // bool tunables
    public TunableAttribute(double min, double max);
    public double Min { get; }
    public double Max { get; }
    public string Doc { get; set; }
}
```

On `static` fields of type `int`, `float`, `double` or `bool`. Numeric tunables need a range (SCN005).

## ProbeAttribute

**Namespace:** `Scion.Core` · **Assembly:** `Scion.Core` · **Define:** none

```csharp
[AttributeUsage(AttributeTargets.Field | AttributeTargets.Property)]
public sealed class ProbeAttribute : Attribute
{
    public ProbeAttribute(string key);
    public string Key { get; }
}
```

Values recorded at the end of each recipe as probes.

## Tunables

**Namespace:** `Scion.Core` · **Assembly:** `Scion.Core` · **Define:** none

```csharp
public static class Tunables
{
    public static bool Set(string key, object value);       // clamps to the range; false if unknown
    public static object Get(string key);
    public static string LobbyHash { get; }                 // hash of rule-layer tunable values
}
```

## ISimulation

**Namespace:** `Scion.Core` · **Assembly:** `Scion.Core` · **Define:** none

```csharp
public interface ISimulation
{
    void Reset(int seed, string setup);
    IReadOnlyList<string> LegalCommands(int player);
    void Apply(int player, string command);
    bool Done { get; }
    string Hash();
    IReadOnlyDictionary<string, double> Metrics();
    IEnumerable<string> Features();
}
```

Commands are serialized strings (the same `Data` as captured matches).

## SimulationAttribute

**Namespace:** `Scion.Core` · **Assembly:** `Scion.Core` · **Define:** none

```csharp
[AttributeUsage(AttributeTargets.Assembly)]
public sealed class SimulationAttribute : Attribute
{
    public SimulationAttribute(Type simulation);
}
```

Registering it raises the project to adoption level 3.

## IBot

**Namespace:** `Scion.Core` · **Assembly:** `Scion.Core` · **Define:** none

```csharp
public interface IBot
{
    string Choose(ISimulation sim, int player, IReadOnlyList<string> legal, Random random);
}

[AttributeUsage(AttributeTargets.Class)]
public sealed class BotAttribute : Attribute { public BotAttribute(string name); }
```

## Capabilities

**Namespace:** `Scion.Core` · **Assembly:** `Scion.Core` · **Define:** none

```csharp
public interface ISaveable { byte[] Save(); void Load(byte[] state); }
public interface IInspectable { string Diff(ISimulation other); }   // first differing field, e.g. "units[0].hp"
public interface IMirrorable { void Mirror(); }
public interface IPermutable { void Permute(int[] order); }
```

Implemented by the simulation for forking (`ISaveable`), divergence reports (`IInspectable`) and metamorphic
properties (`IMirrorable`, `IPermutable`).

## See also

- [Grafts are visual only](/scion/41664431a8f5/grafts/visual-only/)
- [Rules R1-R8](/scion/41664431a8f5/standard/rules/)
- [The multiverse](/scion/41664431a8f5/explore/multiverse/)
- [Balance with the simulator](/scion/41664431a8f5/explore/balance/)

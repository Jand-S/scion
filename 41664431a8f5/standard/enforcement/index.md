---
title: "Enforcement"
description: "How the standard is enforced - analyzers SCN001 to SCN005 configured in ScionSettings.json, doctor, completeness tests, scaffolds and suppressions."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/standard/enforcement/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/standard/enforcement.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/reference/diagnostics/standard.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/init.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/doctor.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Enforcement

The standard is enforced by tools, not by long text: Roslyn analyzers (SCN001 to SCN005, plus SCN003 and SCN110 for
grafts) report in the IDE and in Unity, scoped to the assemblies you list in `ScionSettings.json`; `doctor` reports
the adoption level and what is missing; `scion new` scaffolds code that already passes; completeness tests catch
missing handlers. This guide sets them up.

## When to use

- When adopting the standard in a project: set the scope first, then fix warnings gradually.
- When reviewing agent changes: the analyzers and tests catch most deviations before review.

## Steps

### 1. Scope the analyzers

`ProjectSettings/ScionSettings.json`:

```json
{"schema_version":1,"analyzers":{"rule_assemblies":["Orchard.Core"],"rule_types":["Unity.Netcode.*"],"graft_paths":["Assets/Orchard/Grafts/","../scion/grafts/"]}}
```

Only the assemblies in `rule_assemblies` get SCN001, SCN002, SCN004 and SCN005. Recompile after editing.

### 2. Scaffold new code

```sh
dotnet scion new feature Burn
dotnet scion new effect Push
dotnet scion new graft BurnAuraSound --after Orchard.Presentation.StatusCues.OnBurnAdded
dotnet scion new trial BurnTicks
```

Each command writes code that compiles and passes `doctor`, copied from the project's exemplars: a feature gets its
modifier, its data row and a recipe; an effect gets its enum case, its strategy and a completeness test entry.

### 3. Check the project

```sh
dotnet scion doctor --level 3
```

## Reference

### Analyzers

| Id | Severity | Rule |
|---|---|---|
| SCN001 | warning | A class in rule code is not sealed (R1). |
| SCN002 | warning | A class in rule code inherits more than one level (R1). |
| SCN003 | error | A graft writes rule state (R3). |
| SCN004 | warning | `float` in rule code (R3). |
| SCN005 | warning | A numeric `[Tunable]` without a range. |
| SCN110 | error | Game code calls a graft. |

The analyzers ship in the package (`Scion.Analyzers`, labeled `RoslynAnalyzer`), so Unity and IDEs that read Unity's
generated projects run them. For `dotnet build` outside Unity, add the `Scion.Testing` package, which carries them.

### Suppressing a deviation

```csharp
[SuppressMessage("Scion", "SCN002", Justification = "Netcode requires NetworkBehaviour as the base")]
public sealed class TurnSync : TurnSyncBase { }
```

or `#pragma warning disable SCN004` around the lines. `doctor` lists every suppression with its justification.

### Completeness tests

`scion new effect` adds the effect to `Tests/Orchard.Core.Tests/Completeness.cs`:

```csharp
[Fact]
public void Every_effect_has_a_handler()
{
    foreach (AbilityEffect e in Enum.GetValues(typeof(AbilityEffect)))
        Assert.True(Effects.Handles(e), $"{e} has no handler");
}
```

Other completeness tests in the template: every unit kind has a presentation, every status has cues, every rule
tunable has a range.

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `SCN001`-`SCN005` | compile warning or error | See the table above. | Follow the hint, or suppress with a justification. |
| `bad_value` | 2 | `scion new` with an unknown kind. | Use `feature`, `effect`, `graft` or `trial`. |

## Examples

### Fail CI on standard warnings in the core

```sh
dotnet build Trials/Specs/Orchard.Specs.csproj -warnaserror:SCN001,SCN002,SCN004,SCN005
```

### Preview a scaffold

```sh
dotnet scion new feature Burn --dry-run
```

## See also

- [SCN001-SCN005](/scion/41664431a8f5/reference/diagnostics/standard/)
- [init, new, uninstall](/scion/41664431a8f5/cli/init/)
- [doctor](/scion/41664431a8f5/cli/doctor/)
- [Rules R1-R8](/scion/41664431a8f5/standard/rules/)

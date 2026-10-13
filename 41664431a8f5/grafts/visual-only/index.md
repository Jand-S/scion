---
title: "Grafts are visual only"
description: "Why grafts never change rule state, how SCN003 and SCN110 enforce it, and how rules change through RuleModifier in the core."
type: concept
url: https://jand-s.github.io/scion/41664431a8f5/grafts/visual-only/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/grafts/visual-only.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/grafts/write.md
  - https://jand-s.github.io/scion/41664431a8f5/standard/rules.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/core.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/diagnostics/standard.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Grafts are visual only

Every graft is presentation: effects, sound, camera, HUD, telemetry, trials marks. Rules (damage, range, turn order,
anything in the state hash) change only in the game's core, through `RuleModifier<T>` registered explicitly. This page
explains why, how the analyzers SCN003 and SCN110 enforce it with the `analyzers` section of
`ProjectSettings/ScionSettings.json`, and how a state-hash A/B run proves a graft harmless before it is promoted.

## How it works

### Why rules stay out of grafts

The weaver runs only inside Unity's compilation. It does not run in `dotnet test`, in the rules simulator that balance
specs and the multiverse use, or on a dedicated server built from the core. A graft that changed a rule would work in
the game and vanish everywhere else: tests would pass on rules the game does not use, and co-op peers or replays would
disagree. So the split is fixed:

| Change | How it extends | Where it runs |
|---|---|---|
| **Visual**: juice, sound, camera, HUD, telemetry | A graft on a presentation anchor | Locally on each machine; only in Unity |
| **Rule**: damage, range, turns, AI decisions | A `RuleModifier<T>` registered in the core, with a stage, a priority and integers | Identically on every peer, in Unity and in plain .NET |

### How it is enforced

Two analyzers read `ProjectSettings/ScionSettings.json`:

```json
{"schema_version":1,"analyzers":{"rule_assemblies":["Orchard.Core"],"rule_types":["Unity.Netcode.*"],"graft_paths":["Assets/Orchard/Grafts/","../scion/grafts/"]}}
```

- [SCN003](/scion/41664431a8f5/reference/diagnostics/standard/#scn003) (error): inside graft code, any write to a field, property,
  indexer, event or array element of a rule type, and any use of an internal member of a rule type. A rule type is
  any type in `rule_assemblies`, any type matched by `rule_types`, and any class derived from one.
- [SCN110](/scion/41664431a8f5/reference/diagnostics/grafts/#scn110) (error): game code that refers to graft code. Nothing may
  depend on a graft, so deleting the grafts folder leaves a game that compiles and plays the same.

```csharp
[After(typeof(HitPresenter), nameof(HitPresenter.Play))]
public static void After(HitPresenter presenter, HitEvent e)
{
    presenter.Battle.State.Turn = 0;   // SCN003: Graft CritShake.After writes rule state (State.Turn); grafts are visual only.
}
```

### The proof: hash A/B

SCN003 is a heuristic: it does not see a public method that changes state (`state.Apply(cmd)`) or a write through an
alias. The safety net is the state hash. Run the recipes with and without the graft; the hash must not move:

```sh
dotnet scion trial run --rounds 2
dotnet scion trial run --rounds 2 --grafts .scion/grafts/ScionGrafts_51c2.dll
```

Any `hash: changed` in the second run means the graft touched a rule. The multiverse does the same comparison per turn
and names the first field that changed ([The multiverse](/scion/41664431a8f5/explore/multiverse/)).

### Rules change in the core

```csharp
namespace Orchard.Core
{
    public sealed class BackstabBonus : RuleModifier<DamageCalc>
    {
        public override Stage Stage => Stage.Mul;
        public override int Priority => 10;
        public override void Apply(ref DamageCalc c)
        {
            if (c.FromBehind) c.MulPermille += 500;   // +50 %, integers only
        }
    }
}
```

The core registers it explicitly (`damage.Add(new BackstabBonus())`), so the rule set is visible in code, enters the
lobby hash and runs the same in Unity, in the simulator and on every peer. See
[Core API](/scion/41664431a8f5/reference/api/core/).

## Key terms

| Term | Meaning |
|---|---|
| Graft code | Every `[Before]`/`[After]` method with its lambdas and local functions, and everything declared under `graft_paths`. |
| Rule type | A type in `rule_assemblies`, matched by `rule_types`, or derived from one. |
| Rule code | The code of the `rule_assemblies`; it also follows SCN001, SCN002 and SCN004. |
| State hash | The adapter's `StateHash`: a deterministic hash of the rules state after a recipe. |

## Guarantees and limits

- With an empty or missing `analyzers` section, SCN003 reports nothing; SCN110 still runs.
- SCN003 does not see state changes through public methods or aliases: the hash A/B run and the multiverse do.
- Test assemblies are exempt from SCN110 (those that reference `UnityEngine.TestRunner` in Unity, or `Scion.Testing`
  outside it).
- After editing `ScionSettings.json`, recompile the affected assembly; Unity does not track the file.
- To reach audio, camera or HUD from a graft, call the `Fx` facade instead of the game's systems
  ([Effects with Fx](/scion/41664431a8f5/grafts/fx/)).

## See also

- [Rules R1-R8](/scion/41664431a8f5/standard/rules/)
- [Core API](/scion/41664431a8f5/reference/api/core/)
- [Standard diagnostics](/scion/41664431a8f5/reference/diagnostics/standard/)
- [Effects with Fx](/scion/41664431a8f5/grafts/fx/)

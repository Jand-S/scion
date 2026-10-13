---
title: "The Scion Standard"
description: "Five principles and six layers that keep a game easy to extend, test and hand to agents, with the dependency rule that holds them together."
type: concept
url: https://jand-s.github.io/scion/41664431a8f5/standard/overview/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/standard/overview.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/standard/rules.md
  - https://jand-s.github.io/scion/41664431a8f5/standard/primitives.md
  - https://jand-s.github.io/scion/41664431a8f5/standard/enforcement.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# The Scion Standard

The Scion Standard is a small set of rules for how a Unity game is organized so that Scion, people and agents can
extend it safely. Its five principles say where rules live and how they change; its six layers (Core, Presentation,
Grafts, Adapters, Trials, Tests) say what may depend on what. Analyzers SCN001 to SCN005, `doctor` and the scaffolds
of `scion new` enforce it; nothing in it is required to use grafts or trials, but every rule you follow makes the
adapter thinner and unlocks more of Scion.

## How it works

### Five principles

1. **Rules are explicit in the core.** Plain C#, deterministic, integer math, extended by registration
   (`RuleModifier<T>`), never by IL weaving.
2. **Weaving happens only in presentation.** Grafts are visual; deleting the grafts folder still compiles and plays
   the same.
3. **Shallow inheritance, content as data.** A new enemy or ability is a data row plus a registered strategy, not a
   subclass tree.
4. **Commands go down, events come up.** Presentation sends commands to the core and reacts to its events; the
   methods that react are the anchors.
5. **Types, analyzers and `doctor` enforce the standard**, not long text files. Context files alone help little and
   cost tokens on every request.

### Six layers

| Layer | Contains | May depend on |
|---|---|---|
| Core | Rules, state, commands, events, rule tunables, the `ISimulation` | nothing of Unity |
| Presentation | Views, HUD, camera, audio, the `Fx` sink | Core |
| Grafts | `[Before]`/`[After]` methods, presentation tunables | Presentation, Core (read only) |
| Adapters | `TrialAdapter<T>`, network and input ports | Core, Presentation |
| Trials | Recipes, checks, specs | Adapters, Core |
| Tests | `dotnet test` projects, completeness tests | anything |

The dependency rule: arrows point down the table and never back up. Nothing depends on Grafts (SCN110); the Core
depends on nothing of Unity (an assembly with `noEngineReferences: true`).

### In Orchard

| Layer | Orchard |
|---|---|
| Core | `Orchard.Core`: `BattleState`, `Command`, `DamageCalc`, `BackstabBonus : RuleModifier<DamageCalc>` |
| Presentation | `Orchard.Runtime`: `HitPresenter`, `Hud`, `CameraRig`, `DamageNumbers`, `OrchardFxSink` |
| Grafts | `Assets/Orchard/Grafts`: `HitFlash`, `CritShake`, `DamageNumberPop`, `TurnBanner` |
| Adapters | `OrchardAdapter`, `OrchardMatchCapture` |
| Trials | `Orchard.Trials`: `SlashHit`, `CoopTurn`, `HudStates`; `Trials/Specs` |
| Tests | `Tests/Orchard.Core.Tests`, `Tests/Orchard.Grafts.Tests` |

## Key terms

| Term | Meaning |
|---|---|
| Core | The rules assembly, listed in `analyzers.rule_assemblies`. |
| Rule code | Code in the core; follows SCN001, SCN002 and SCN004. |
| Primitive | One of the standard's building blocks ([Primitives](/scion/standard/primitives/)). |
| Exemplar | A small, compiled, tested example in the project that agents copy. |
| Completeness test | A test that every enum value has a handler, every enemy a presentation. |

## Guarantees and limits

- Following the standard is gradual: each rule stands alone, and `doctor` reports the adoption level.
- The analyzers only report on assemblies you list; third-party code is never checked.
- Deviations are allowed with a justification, through the normal Roslyn suppression.

## See also

- [Rules R1-R8](/scion/standard/rules/)
- [Primitives](/scion/standard/primitives/)
- [Enforcement](/scion/standard/enforcement/)
- [Grafts are visual only](/scion/grafts/visual-only/)

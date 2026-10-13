---
title: "Orchard sample"
description: "The frozen Orchard Tactics sample every snippet uses - its assemblies, rules types, presentation types, recipe base class, adapter and recipes."
type: reference
url: https://jand-s.github.io/scion/41664431a8f5/reference/orchard-sample/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/orchard-sample.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/start/quickstart.md
  - https://jand-s.github.io/scion/41664431a8f5/start/new-project.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/recipes.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/adapters.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Orchard sample

Every example on this site plays the same small game: Orchard Tactics, the sample that
`dotnet scion new --genre tactics --name Orchard` writes. This page fixes its types once. A snippet on any other page
uses only the names below, with the signatures below; when a page needs one more line (a variant grid, a mode, a
tunable), the snippet shows it and says so. Units are named by their `UnitKind.Id` string everywhere, so recipes,
variants and `--set` all set them from text.

## Assemblies

| Assembly | Namespace | Holds |
|---|---|---|
| `Orchard.Core` | `Orchard.Core` | The rules: plain C#, integers, no `UnityEngine`. Registers the `ISimulation`. |
| `Orchard.Runtime` | `Orchard.Runtime`, `Orchard.Presentation` | The battle in the scene, input, the HUD and the presenters. References `Scion.Runtime`. |
| `Orchard.Grafts` (`.asmref`) | `Orchard.Grafts` | Promoted grafts; compiles into `Orchard.Runtime`. |
| `Orchard.Trials` | `Orchard.Trials` | `OrchardRecipe`, `OrchardAdapter`, the recipes. `defineConstraints: ["SCION_TRIALS"]`. |

## Rules types

```csharp
namespace Orchard.Core
{
    // "knight", "skeleton", "mage", "rogue".
    public sealed class UnitKind { public string Id { get; } public int MaxHp { get; } }
    public static class UnitKinds { public static IReadOnlyList<UnitKind> All { get; } public static UnitKind Get(string id); }

    public readonly struct HitEvent
    {
        public HitEvent(int attacker, int target, int amount, bool crit);
        public int Attacker { get; } public int Target { get; } public int Amount { get; } public bool Crit { get; }
    }

    // Units 0 .. players-1 attack; units players .. players+defenders-1 defend.
    public sealed class BattleSetup
    {
        public BattleSetup(string attacker, string defender, int defenderHp, int distance,
                           int defenders = 1, int players = 1, int seed = 1);
        public string Attacker { get; } public string Defender { get; } public int DefenderHp { get; }
        public int Distance { get; } public int Defenders { get; } public int Players { get; } public int Seed { get; }
    }

    public sealed class Command
    {
        public static Command Ability(string ability, int attacker, int target);   // "slash", "fireball"
        public static Command FromJson(string json);
        public int Turn { get; } public int Player { get; }
        public string ToJson();
    }

    public sealed class BattleState
    {
        public long Hash64 { get; } public long Tick { get; } public int Turn { get; }
        public UnitState Unit(int index);          // UnitState: Kind, Hp, Down
    }

    public static class Balance
    {
        [Tunable(500, 2000, Doc = "Critical damage multiplier, in permille.")] public static int CritPermille = 1500;
        [Tunable(500, 2000, Doc = "Enemy HP, in permille of the unit data.")] public static int EnemyHpPermille = 1000;
    }
}
```

## Runtime and presentation types

```csharp
namespace Orchard.Runtime
{
    public sealed class Battle
    {
        public static Battle Current { get; }                           // null between battles
        public static Battle Start(BattleSetup setup, Transform root = null);
        public static void End();
        public BattleSetup Setup { get; } public BattleState State { get; }
        public bool Ready { get; } public bool Idle { get; }
        public IReadOnlyList<Command> Log { get; }
        public void Issue(Command command);
    }

    public static class Boot { public static bool Ready { get; } }
    public static class Lobby { public static int Connected { get; } public static int LocalPlayer { get; } }
    public static class Gallery { public static void Place(UnitKind k); public static void Clear(); public static bool Ready { get; } public static int HeightPx { get; } }
    public static class Arenas { public static Transform Root(int arena); }
}

namespace Orchard.Presentation
{
    public sealed class HitPresenter : MonoBehaviour
    {
        [Anchor(Id = "presentation.hit", Doc = "Plays one hit and returns how long it lasts.")]
        public float Play(HitEvent e);
        public UnitView UnitView(int unit);
        public Vector3 TargetPosition { get; }
    }

    public sealed class UnitView : MonoBehaviour { public void PlayStatus(string status); public static bool AnyAnimating { get; } }
    public sealed class Hud : MonoBehaviour { public static Hud Current { get; } public void ShowTurn(int turn); }
}
```

## Recipe base class

```csharp
namespace Orchard.Trials
{
    public abstract class OrchardRecipe : TrialRecipe
    {
        public string Attacker = "knight";     // a UnitKind.Id
        public string Defender = "skeleton";   // a UnitKind.Id
        public int DefenderHp = 20;
        public int Defenders = 1;
        public int Distance = 1;
        public string Ability = "slash";
    }
}
```

`OrchardAdapter : TrialAdapter<OrchardRecipe>` plays every recipe: it builds the battle with
`SetupFor(r, s)`, which is `new BattleSetup(r.Attacker, r.Defender, r.DefenderHp, r.Distance, defenders: r.Defenders,
players: r.Players, seed: s.RunSeed)`, issues `Command.Ability(r.Ability, attacker: 0, target: r.Players)`, and
probes `damage` and `defender_hp`. Its full code is on [Write an adapter](/scion/41664431a8f5/trials/adapters/).

## Recipes

| Recipe | Players | What it shows | Where it comes from |
|---|---|---|---|
| `SlashHit` | 1 | The knight slashes a skeleton next to it. | The template |
| `FireballArea` | 1 | A mage's fireball hits three skeletons (`Defenders = 3`). | The template |
| `CoopTurn` | 4 | Four knights each act once; every peer agrees on the state. | The template |
| `HudStates` | 1 | The HUD in each of its states, as a static trial. | The template |
| `UnitGallery` | 1 | Every `UnitKind` in its idle pose, as a static trial. | The template |
| `KnightFalls` | 1 | A skeleton finishes a wounded knight (`DefenderHp = 3`). | Step 9 of the [Quickstart](/scion/41664431a8f5/start/quickstart/) |
| `CapturedMatch[match=<name>]` | 1 | A match captured with F9 and replayed. | [Captured matches](/scion/41664431a8f5/trials/captured-matches/) |

The graft examples hang on `HitPresenter.Play` (anchor `presentation.hit`): `HitFlash` (the quickstart) and
`CritShake` (the agent loop and the graft guides). The rule tunables are `Balance.CritPermille` and
`Balance.EnemyHpPermille`; the graft tunable is `CritShake.Strength`.

## See also

- [Quickstart](/scion/41664431a8f5/start/quickstart/)
- [Start a new project](/scion/41664431a8f5/start/new-project/)
- [Write a recipe](/scion/41664431a8f5/trials/recipes/)
- [Write an adapter](/scion/41664431a8f5/trials/adapters/)
- [The Scion Standard](/scion/41664431a8f5/standard/overview/)

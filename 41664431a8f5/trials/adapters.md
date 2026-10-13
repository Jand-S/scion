---
title: "Write an adapter"
description: "Write the TrialAdapter that plays your recipes - Prepare once, Run per recipe, Teardown, StateHash - and register it so each recipe finds its adapter."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/trials/adapters/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/trials/adapters.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/recipes.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/session.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/adapters.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/diagnostics/trials.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Write an adapter

The adapter is the game's only Trials-specific code: it prepares the game once, plays each recipe (load, set up,
act, wait), cleans up and reports a hash of the rules state. Derive from `TrialAdapter<TRecipe>` and register it with
`[assembly: TrialAdapter(typeof(...))]`. This guide covers `Prepare`, `Run`, `Teardown`, `StateHash`, how a recipe
picks its adapter, the coroutine rules, and the reset and seeding every adapter must do. Errors: SCN202, SCN203.

## When to use

- Write one adapter per family of recipes that start the same way: battles, HUD screens, menus.
- Use `ITrialAdapter` directly only when the choice depends on something other than the recipe's type.
- For screens and items that need no action, derive recipes from `StaticTrial<TItem>` and skip the adapter
  entirely ([Static trials](/scion/41664431a8f5/trials/static-trials/)).

## Steps

### 1. Write the adapter

```csharp
using System.Collections;
using Orchard.Core;
using Orchard.Runtime;
using Scion.Trials;

[assembly: TrialAdapter(typeof(Orchard.Trials.OrchardAdapter))]

namespace Orchard.Trials
{
    public sealed class OrchardAdapter : TrialAdapter<OrchardRecipe>
    {
        string hash;
        public override string StateHash => hash;

        // One battle setup for every recipe; co-op and mosaic runs use it too.
        public static BattleSetup SetupFor(OrchardRecipe r, TrialSession s) =>
            new BattleSetup(r.Attacker, r.Defender, r.DefenderHp, r.Distance,
                            defenders: r.Defenders, players: r.Players, seed: s.RunSeed);

        public override IEnumerator Prepare(TrialSession s)
        {
            s.Sensor("state", () => Battle.Current?.State.Hash64 ?? 0L);
            s.Sensor("tick", () => Battle.Current?.State.Tick ?? 0L);
            yield return s.WaitUntil(() => Boot.Ready, "boot scene loaded");
        }

        protected override IEnumerator Run(OrchardRecipe r, TrialSession s)
        {
            if (r.Players > 1) { yield return RunCoop(r, s); yield break; }   // see Co-op trials
            hash = null;
            var battle = Battle.Start(SetupFor(r, s));
            yield return s.WaitUntil(() => battle.Ready, "battle ready");
            s.Still("before");
            yield return null;

            int target = r.Players;          // the first defender
            int hp = battle.State.Unit(target).Hp;
            s.BeginCut();
            battle.Issue(Command.Ability(r.Ability, attacker: 0, target: target));
            yield return s.WaitUntil(() => battle.Idle, "ability resolved");
            yield return s.WaitSeconds(r.TailSeconds);
            s.EndCut();

            s.Probe("damage", hp - battle.State.Unit(target).Hp);
            s.Probe("defender_hp", battle.State.Unit(target).Hp);
            hash = battle.State.Hash64.ToString("X16");
        }

        protected override IEnumerator Teardown(OrchardRecipe r, TrialSession s)
        {
            Battle.End();
            yield return s.WaitUntil(() => Battle.Current == null, "battle cleared");
        }
    }
}
```

### 2. Reset global state between recipes

Every recipe must start from the same state, whatever ran before it in the queue:

- end and unload what `Run` created in `Teardown`;
- reset statics and singletons the game keeps (`Time.timeScale`, caches, pools) in `Teardown` or on
  `TrialSession.Boundary`;
- seed every random source from the recipe: `UnityEngine.Random` and `new System.Random()` are seeded by the
  session; seed the game's own generators from `s.RunSeed` and the recipe's `Seed`.

```csharp
TrialSession.Boundary += () => { UnityEngine.Time.timeScale = 1f; DamageNumbers.Pool.Clear(); };
```

### 3. Run it twice

```sh
dotnet scion trial run --rounds 2
```

With two rounds, the `determinism` check fails any recipe whose `StateHash` differs between rounds.

## Reference

### Members

| Member | Default | Called |
|---|---|---|
| `public virtual IEnumerator Prepare(TrialSession s)` | nothing | Once per process, before the first recipe this adapter gets. With warm copies, at startup. |
| `protected abstract IEnumerator Run(TRecipe r, TrialSession s)` | | Per (recipe, round): set up, act, cut, probe. |
| `protected virtual IEnumerator Teardown(TRecipe r, TrialSession s)` | nothing | After `Run`, even when it failed. |
| `public virtual string StateHash { get; }` | `null` | Read after `Run`, before `Teardown`. A deterministic hash of the rules state, or null. |

`TrialAdapter<TRecipe>` is a `MonoBehaviour`: the session gives it its own GameObject that survives scene loads.
A plain `ITrialAdapter` (with `Accepts`) needs a public parameterless constructor.

### Which adapter plays a recipe

1. Only adapters whose recipe type is the recipe's class or a base of it, and whose `Accepts(recipe)` is true, count.
   A plain `ITrialAdapter` handles `TrialRecipe`, the least specific type.
2. The adapter with the most derived recipe type wins. `TrialAdapter<HudStates>` beats the package's
   `StaticTrialAdapter : TrialAdapter<StaticTrial>` for `HudStates`.
3. Two adapters for the same type are SCN203. Registration order decides nothing.

A selected recipe no adapter accepts fails alone, naming the registered adapters.

### Coroutine rules

- Everything is a coroutine: wait with `yield return s.WaitUntil(...)`, `s.WaitSeconds(...)` or `null`, never with
  sleeps.
- A throw, a timeout or `s.Fail` in `Run` fails the current recipe and stops `Run` at its next `yield`; `Teardown`
  still runs (limit 15 game seconds).
- In `Prepare`, any of them stops the queue with exit 3 (`prepare_failed`). `Prepare` has a limit of 60 seconds of
  real time.
- Split steps into methods and `yield return Step();` so the guards reach them. A coroutine started with
  `StartCoroutine` runs outside the guards.

### What makes a good StateHash

- Only rules state: unit positions, HP, turn, RNG state of the core. Never presentation (animations, particles).
- Stable across machines: integers, ordered collections, no `GetHashCode` of strings.
- The same value the multiverse and co-op telemetry use, so one function serves all three.

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `SCN202` | 3 | A registered type is not a valid adapter. | Derive from `TrialAdapter<TRecipe>` with a public parameterless constructor. |
| `SCN203` | 3 | Two adapters handle the same recipe type. | Keep one per type; make one more specific. |
| `no_adapter` | 3 | No `[assembly: TrialAdapter]` in the build. | Register the adapter; `dotnet scion diagnose --adapter` writes a skeleton. |
| `prepare_failed` | 3 | `Prepare` threw, timed out or called `Fail`. | Read `player.log` of the copy. |

## Examples

### A HUD adapter for static trials

```csharp
[assembly: TrialAdapter(typeof(Orchard.Trials.HudAdapter))]

public sealed class HudAdapter : TrialAdapter<HudStates>
{
    protected override IEnumerator Run(HudStates r, TrialSession s)
    {
        yield return SceneLoader.Load("HudPreview");
        yield return r.Play(s);
    }
}
```

### Map the run seed to the world

```csharp
var battle = Battle.Start(setup with { MapSeed = s.RunSeed * 7919 + r.Seed });
```

## See also

- [Write a recipe](/scion/41664431a8f5/trials/recipes/)
- [TrialSession](/scion/41664431a8f5/trials/session/)
- [Adapters API](/scion/41664431a8f5/reference/api/adapters/)
- [SCN201-SCN203](/scion/41664431a8f5/reference/diagnostics/trials/)

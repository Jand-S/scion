---
title: "Balance with the simulator"
description: "Run balance specs on the core simulator without Unity for 1 to 4 players, check a difficulty curve, and calibrate it with real telemetry."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/explore/balance/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/explore/balance.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/specs.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/specs.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/core.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Balance with the simulator

Balance questions ("is floor 1 easy enough?", "does four-player co-op get too easy?") need thousands of matches,
not one recipe. Balance specs run the rules as an `ISimulation` in plain .NET with bots, for 1 to 4 players, in
seconds, and check bounds on metrics such as win rate and match length. A difficulty curve spec checks a whole
progression at once. Bots shape the curve; telemetry from real players sets its level.

## When to use

- After every change to rules data (damage tables, enemy stats, floor depth).
- To compare balance presets before playtesting.

## Before you begin

- Adoption level 3: the rules in an assembly without engine references, with `ISimulation` and at least one `IBot`.

## Steps

### 1. Write a curve spec

```csharp
public sealed class DifficultyCurve : TrialSpec
{
    public override string Name => "DifficultyCurve";
    public override string Target => "Starts easy, ends hard, for 1 to 4 players.";

    public override void Define(SpecBuilder s)
    {
        for (int players = 1; players <= 4; players++)
            s.Curve("floors", levels: 6, players: players, matches: 500)
             .ExpectAt(1, "win_rate", min: 0.95)
             .ExpectAt(6, "win_rate", min: 0.35, max: 0.55)
             .Monotonic("win_rate", decreasing: true)
             .MaxStep("win_rate", 0.15);
    }
}
```

### 2. Run it

```sh
dotnet scion trial run --spec DifficultyCurve
```

Each player count runs 6 x 500 matches with the registered bots. On an Apple M5, Orchard's 12,000 matches take
about 40 seconds.

### 3. Read the curve

The spec's report lists win rate per floor and player count, and the bounds that failed:

```text
players=4  floor 1 0.99  floor 2 0.93  floor 3 0.90  floor 4 0.71  floor 5 0.52  floor 6 0.41
           failed: MaxStep win_rate 0.19 between floors 3 and 4
```

## Reference

### What a balance run needs from the core

| Member | Use |
|---|---|
| `ISimulation.Reset(seed, setup)` | A new match. |
| `ISimulation.LegalCommands`, `Apply`, `Done` | The bots play through them. |
| `ISimulation.Metrics` | `win_rate`, `turns`, and the game's own metrics (`damage_taken`, `heals_used`). |
| `IBot` with `[Bot("name")]` | Players with a style (`aggressive`, `cautious`, `random`). |

### Bots are not people

Bots of different styles shape the curve: if every style finds floor 6 equally hard, the curve's shape holds. The
level of the curve (how hard is hard) comes from telemetry of real matches: compare the spec's win rate with the win
rate players had, and set the bounds from that.

### Searching balance

With bounds in a spec, guided search can tune the rules data automatically
([Guided search](/scion/41664431a8f5/explore/guided-search/)).

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `no_simulation` | 3 | No `ISimulation`. | Register one. |
| spec `ok: false` | 1 | A bound failed. | Read the report; change data or the bounds (in a new spec). |

## Examples

### One floor, four players

```csharp
s.Balance("Floor3", players: 4, matches: 1000).Expect("win_rate", min: 0.6, max: 0.75);
```

### Compare two balance presets

```sh
dotnet scion trial run --spec DifficultyCurve --variants base,enemy-hp-90
```

## See also

- [Specs](/scion/41664431a8f5/trials/specs/)
- [Specs API](/scion/41664431a8f5/reference/api/specs/)
- [Core API](/scion/41664431a8f5/reference/api/core/)

---
title: "The multiverse"
description: "Fork a deterministic match at a turn, run variants of code or numbers from that point, and see the first turn and field where the rules diverge."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/explore/multiverse/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/explore/multiverse.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/cli/multiverse.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/captured-matches.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/mosaic.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/core.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# The multiverse

The multiverse replays a captured match up to turn N, forks it, and plays K variants from there: other grafts,
other tunables, other presets, other bots. Because the rules are deterministic, any difference is caused by the
variant. Scion reports, per variant, whether the rules diverged, the first turn and the first field that changed,
and shows the visuals side by side in a mosaic. It is how you prove a graft is only visual, and how you see what a
balance change does to a real match.

## When to use

- Before promoting a graft: run the match with and without it; nothing may diverge.
- When tuning rules: see the first turn where a new damage value changes the outcome.
- When a bug appears late in a match: fork just before it and try fixes.

## Before you begin

- Adoption level 3: the rules registered as an `ISimulation`, deterministic in plain .NET
  ([Find non-determinism](/scion/41664431a8f5/determinism/diagnose/)).
- A captured match ([Turn any match into a recipe](/scion/41664431a8f5/trials/captured-matches/)).
- For `grafts:` variants and presentation tunables: a test build (`dotnet scion build test`), and an `IMatchCapture`
  whose `Replay` writes a `turn` event with the state hash after every turn.

## Steps

### 1. Fork and run

```sh
dotnet scion multiverse run knight-dies-to-crit --at-turn 2 --variants base,heavy,crit-1200
```

`crit-1200` is a preset that sets the rule tunable `Balance.CritPermille` to 1200, so it runs on the core simulator.
`heavy` is a feel preset of graft tunables (`CritShake.Strength` and others), so it runs in the test build.

<!-- validate: cli/multiverse -->
```json
{"schema_version":1,"command":"multiverse run","ok":true,"run":"/Users/alex/orchard/.scion/multiverse/20261012-101500","match":"knight-dies-to-crit","at_turn":2,"variants":[{"name":"base","diverged":false,"first_turn":null,"field":null},{"name":"heavy","diverged":false,"first_turn":null,"field":null},{"name":"crit-1200","diverged":true,"first_turn":3,"field":"units[0].hp"}],"report":"multiverse.json","mosaic":null,"next_step":"heavy is visual only; crit-1200 changes units[0].hp at turn 3: dotnet scion multiverse show 20261012-101500"}
```

### 2. Read the report

```sh
dotnet scion multiverse show 20261012-101500
```

`multiverse.json` has the hash of every turn for the base and each variant, and for diverged variants the field's
value before and after.

### 3. See it

```sh
dotnet scion multiverse run knight-dies-to-crit --at-turn 2 --variants base,heavy --mosaic
```

With `--mosaic`, the test build replays the variants side by side from the fork turn and writes `mosaic.png`.

## Reference

### Variants

| Kind | Given as | Changes | Runs on |
|---|---|---|---|
| `preset` | a preset name or blend | tunables | the core simulator when every tunable is in the core assembly; otherwise the test build |
| `tunables` | `tunable:Balance.CritPermille=1200` | one or more tunables | the same rule as `preset` |
| `grafts` | `grafts:<dll>` | presentation code (the rules must not diverge) | the test build, always |
| `bot` | `bot:<name>` | who plays the remaining turns | the core simulator |

`--variants` takes a comma-separated list, or a JSON file with one object per variant.

### How it runs

Each variant runs where the code it changes runs: rule variants on the core simulator, presentation variants in the
test build.

#### On the core simulator

Rule variants (rule tunables, presets of rule tunables, bots) change only what the `ISimulation` reads:

1. The `ISimulation` replays the match's commands up to `--at-turn` and saves the state (`ISaveable`).
2. For each variant, it loads the state, applies the variant and plays the remaining commands (or the bot).
3. After every turn it records `Hash()`; the first turn whose hash differs from the base is `first_turn`.
4. At that turn, `IInspectable.Diff` names the first field that differs (`units[0].hp`).

These steps run in plain .NET, in parallel, in milliseconds per turn.

#### In the test build

A graft hangs on a game method such as `HitPresenter.Play`, which the plain .NET `ISimulation` never calls; the same
holds for tunables read by presentation code. On the core simulator such a variant could never diverge, so it would
prove nothing. These variants run in the test build instead:

1. One copy replays the match without changes (`-scion-match <file>`): the base. After every turn, `Replay` writes
   a `turn` event with the state hash.
2. Each variant gets its own copy. It replays the same commands to `--at-turn`, then applies the variant at that turn
   boundary (the DLL is injected through the copy's control channel; tunables are set) and plays the remaining turns,
   writing the same `turn` events.
3. The CLI compares the per-turn hashes of each variant with the base, turn by turn; the first that differs is
   `first_turn`.
4. At that turn, both copies save the state (`ISaveable`) and `IInspectable.Diff` names the first field that differs.

A `grafts:` variant whose hashes match the base on every turn after the fork really leaves the rules alone, because
the graft ran on every presentation call of the replay. This takes seconds per variant (one copy each), and the copies
run in parallel. `--mosaic` uses the same copies to write `mosaic.png`.

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `no_simulation` | 3 | No `[assembly: Simulation]`. | Register an `ISimulation`. |
| `not_forkable` | 3 | The simulation does not implement `ISaveable`. | Implement `Save` and `Load` on the state. |
| `no_match` | 2 | The match file or a variant does not exist. | `trial list` shows captured matches. |
| variant `diverged` with `grafts` | 1 | A graft changed rules. | Move the change to the core ([Grafts are visual only](/scion/41664431a8f5/grafts/visual-only/)). |

## Examples

### Prove a draft graft is visual

```sh
dotnet scion build test
dotnet scion graft build
dotnet scion multiverse run knight-dies-to-crit --at-turn 0 --variants base,grafts:.scion/grafts/ScionGrafts_51c2.dll
```

Take the DLL path from the `dll` field of `graft build`. Both variants replay in the test build, and the graft runs on
every hit of the match. Exit 0 with `diverged: false` means the state hash matched the base after every turn. For a
recipe instead of a match, `trial run <Id> --grafts <dll>` gives the same proof on one recipe: `hash: same` against
the baseline.

### Let bots finish the match

```sh
dotnet scion multiverse run knight-dies-to-crit --at-turn 2 --variants bot:aggressive,bot:cautious
```

## See also

- [multiverse (CLI)](/scion/41664431a8f5/cli/multiverse/)
- [Turn any match into a recipe](/scion/41664431a8f5/trials/captured-matches/)
- [Mosaic](/scion/41664431a8f5/trials/mosaic/)
- [Core API](/scion/41664431a8f5/reference/api/core/)

---
title: "The live loop"
description: "Loop a real window of play from a captured match or telemetry, and see each injected graft on the next pass without restarting."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/explore/live-loop/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/explore/live-loop.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/cli/trial-loop.md
  - https://jand-s.github.io/scion/41664431a8f5/grafts/injection.md
  - https://jand-s.github.io/scion/41664431a8f5/cookbook/tune-hit-feel.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# The live loop

The live loop replays a short window of real play over and over in a test build: say turns 3 to 4 of a captured
match. Each graft injection applies at the start of the next pass, so you tune an effect while watching the exact
moment it belongs to. `dotnet scion trial loop` drives it; `graft watch` injects on every save.

## When to use

- Tuning feel on one moment: a crit, a unit falling, a turn banner.
- Reproducing a visual bug that needs a specific game state.

## Before you begin

- A captured match or a recipe's `telemetry.jsonl` with turn events.
- A test build (`dotnet scion build test`).

## Steps

### 1. Start the loop

```sh
dotnet scion trial loop "CapturedMatch[match=knight-dies-to-crit]" --span 3..4 --window
```

The test build opens in a window, replays the match to the start of turn 3 once, saves that state and then plays
turns 3 to 4 in a loop.

### 2. Edit and inject

In another terminal:

```sh
dotnet scion graft watch
```

Every save rebuilds the drafts and injects them; the next pass shows the change.

### 3. Stop

Press Ctrl-C in the loop's terminal, or pass `--passes N` to stop after N passes.

<!-- validate: cli/trial-loop -->
```json
{"schema_version":1,"command":"trial loop","ok":true,"run":"/Users/alex/orchard/.scion/loops/20261012-103000","id":"CapturedMatch[match=knight-dies-to-crit]","span":"3..4","passes":14,"injects":5,"last":{"ok":true,"hash":"9F3A0C11D2E4B5A6","frames":96,"window":"pass14/window.png"},"stopped_by":"interrupt","next_step":"dotnet scion graft promote CritShake"}
```

## Reference

| Option | Default | Meaning |
|---|---|---|
| `<Id>` | | A recipe id, usually a captured match. |
| `--from <file>` | the recipe's match | A `.match.json` or a `telemetry.jsonl` to take the window from. |
| `--span a..b` | the whole recipe | Turns to loop (inclusive). |
| `--passes N` | until stopped | Stop after N passes. |
| `--window` | headless | Show the window (usual for the loop). |

Every pass writes `pass<N>/` with `result.json`, `window.png` and telemetry, so you can compare passes with
`trial diff`. The hash of the loop must stay the same across passes; when an injected graft changes it, the loop
stops with `stopped_by: error`.

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `no_simulation` | 3 | Restoring the state needs `ISaveable` on the rules. | Implement it, or loop a whole recipe (no `--span`). |
| `bad_value` | 2 | `--span` outside the match's turns. | Check `turns` in the match file. |
| stopped by `error` | 1 | A pass changed the hash. | The injected graft touches rules. |

## Examples

### Loop a whole short recipe

```sh
dotnet scion trial loop SlashHit --window
```

### Loop from a recipe's telemetry

```sh
dotnet scion trial loop KnightFalls --from .scion/runs/20261011-143200/q1/KnightFalls/r1/telemetry.jsonl --span 1..1
```

## See also

- [trial loop](/scion/cli/trial-loop/)
- [Live injection](/scion/grafts/injection/)
- [Tune a hit effect](/scion/cookbook/tune-hit-feel/)

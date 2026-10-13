---
title: "trial loop"
description: "Replay a window of play in a loop in the test build, applying each graft injection on the next pass, until stopped or a pass count."
type: cli
url: https://jand-s.github.io/scion/41664431a8f5/cli/trial-loop/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/cli/trial-loop.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/explore/live-loop.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/graft.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/captured-matches.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# trial loop

`dotnet scion trial loop` plays one id over and over in a test build: the whole recipe, or a span of turns of a
captured match. Each pass restores the saved start state and replays; injections made between passes apply on the
next one. It is the engine of the live loop.

## Synopsis

```text
dotnet scion trial loop <Id> [--from <file>] [--span a..b] [--passes N] [--window] [--build <path>]
```

## Description

1. Starts one copy with a control channel (`-scion-control 0`), and `-scion-match <file>` when the id is a match.
2. Plays to the start of `--span` once and saves the state (`ISaveable`); without `--span`, the recipe start.
3. Loops: restore, play the span, write `pass<N>/`, apply pending injections, repeat.
4. Stops on Ctrl-C, after `--passes`, or when a pass changes the hash (an injected graft touched rules).

## Options

| Option | Default | Meaning |
|---|---|---|
| `<Id>` | required | A recipe id, usually `CapturedMatch[match=<name>]`. |
| `--from <file>` | | A `.match.json` or `telemetry.jsonl` to take the window from. |
| `--span a..b` | whole recipe | Turns to loop, inclusive. |
| `--passes N` | until stopped | Number of passes. |
| `--window` | headless | Show the window. |
| `--build <path>` | discovery | The test build. |

## Output

<!-- validate: cli/trial-loop -->
```json
{"schema_version":1,"command":"trial loop","ok":true,"run":"/Users/alex/orchard/.scion/loops/20261012-103000","id":"SlashHit","span":null,"passes":3,"injects":1,"last":{"ok":true,"hash":"9F3A0C11D2E4B5A6","frames":46,"window":"pass3/window.png"},"stopped_by":"passes","next_step":"compare passes: dotnet scion trial diff 20261012-103000/pass1 20261012-103000/pass3"}
```

`stopped_by` is `passes`, `stop`, `interrupt` or `error`.

## Exit codes

| Code | When |
|---|---|
| 0 | Stopped by passes, stop or interrupt. |
| 1 | A pass failed or changed the hash. |
| 2 | Bad option. |
| 3 | The copy failed, or the state cannot be saved. |

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `not_forkable` | 3 | `--span` needs `ISaveable`. | Implement it, or loop the whole recipe. |
| `bad_value` | 2 | `--span` outside the match. | Check `turns`. |
| `interrupted` | 0 | Ctrl-C. | Normal stop. |

## Examples

### Two turns of a match, in a window

```sh
dotnet scion trial loop "CapturedMatch[match=knight-dies-to-crit]" --span 3..4 --window
```

### Five passes headless

```sh
dotnet scion trial loop SlashHit --passes 5
```

## See also

- [The live loop](/scion/explore/live-loop/)
- [graft](/scion/cli/graft/)
- [Turn any match into a recipe](/scion/trials/captured-matches/)

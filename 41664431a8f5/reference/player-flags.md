---
title: "Player command line"
description: "Every -scion flag of the test player, with its value, default, meaning and the CLI option that sets it; unknown flags exit 2."
type: reference
url: https://jand-s.github.io/scion/41664431a8f5/reference/player-flags/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/player-flags.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/cli/trial-run.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/exit-codes.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/overview.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Player command line

The test player reads `-scion-*` flags from its command line. The CLI sets them for you; you need them to start the
player yourself, in CI scripts or for debugging. Every flag starts with `-scion-`; an unknown one exits 2 with
`unknown_flag` and `did_you_mean`. Unity's flags (`-batchmode`, `-screen-width`, `-logFile`) and the game's own flags
pass through unchecked.

## Example

```sh
.scion/builds/test/Orchard.app/Contents/MacOS/Orchard -batchmode -scion-trials -scion-out /tmp/scion-out -scion-only SlashHit -scion-rounds 2 -scion-quit
```

## Flags

### -scion-trials

No value. Turns the lab on: without it, the test build runs as the normal game. Set by every CLI command that starts
players.

### -scion-out

A folder; required with `-scion-trials`. This process's output (`run.json`, `results.tsv`, recipe folders). The CLI
uses `<run>/q<i>/`. Missing: exit 2 `missing_out`.

### -scion-recipes

A DLL; repeatable. Recipes compiled outside Unity; a recipe with the same `Name` replaces the built-in one. CLI:
`--recipes`.

### -scion-only

Comma-separated patterns. Default: every non-`Explicit` recipe. Each pattern matches an exact `Id`, a `Name` (all its
variants) or a glob (`*`, `?`) over ids. Nothing matched: exit 2 `no_match`. CLI: positional patterns.

### -scion-rounds

An integer, at least 1; default 1. Total rounds of each id (shards split them). CLI: `--rounds`.

### -scion-quit

No value. Exit when the queue ends, with the exit code. Without it the process stays open.

### -scion-sound

No value. Keep audio on (default muted). CLI: `--sound`.

### -scion-offscreen

No value. Capture from textures. Automatic with `-batchmode`; use it with a window to capture offscreen anyway.

### -scion-frames

No value. Save every frame of the cut as `frames/NNNN.png`. CLI: `--frames`.

### -scion-inject

A DLL. Inject draft grafts at startup (needs `SCION_SLOTS`). CLI: `--grafts`.

### -scion-shard

`i/n`, default `0/1`. Play shard `i` of `n` (0 to n-1); each (id, round) pair runs in exactly one shard, round-robin
over (round, id). Set by the CLI from `-j`.

### -scion-check

No value. Compare each later round with the first frame by frame and write `determinism.tsv`. CLI: `--check`.

### -scion-jitter

Milliseconds, 0 to 1000; default 0. Sleep a random real time up to this per frame from the second occurrence of each
id. CLI: `--jitter`.

### -scion-epoch

ISO 8601; default `2000-01-01T00:00:00Z`. The start of the virtual `DateTime.Now` and `UtcNow`.

### -scion-seed

An integer; default 1. `TrialSession.RunSeed`; the adapter maps it to the game's world seed. CLI: `--seed`.

### -scion-raw

No value. Keep `frames/NNNN.raw` and `frames/raw.json`. CLI: `--raw`.

### -scion-log-allow

A file with one .NET regex per line. Errors whose message matches do not fail the `log` check. The CLI writes it from
`log_allow`.

### -scion-list

No value. Write `recipes.json` with the selected, expanded ids and exit 0 without playing. CLI: `trial list --ids`.

### -scion-players

1 to 4; default 1. Play only recipes whose `Players` equals this. Set by the CLI per group.

### -scion-role

`host` or `client:<i>` (1 to 3); default `host`. The process's role in a co-op group. Set by the CLI.

### -scion-control

A port, or 0 for any free port. Open the control server on the loopback interface and write `control.json`.

### -scion-serve

No value; needs `-scion-control`. Warm copy: prepare every adapter and wait idle for `run` requests instead of playing
the queue. Without `-scion-control`: exit 2 `bad_value`. CLI: `trial serve`.

### -scion-tag

Comma-separated tags. Only recipes with any of them. CLI: `--tag`.

### -scion-mode

A mode name from `[assembly: TrialModes]`. `name=WxH` modes set the capture size; others go to `ITrialModeAware`.
CLI: `--mode`.

### -scion-seeds

`a-b`. Repeat the selection for each seed in the range; ids get `#<seed>`. CLI: `--seeds`.

### -scion-match

A `.match.json` file. Play that captured match as a `CapturedMatch` recipe, also in a window for the live loop.

### -scion-set

`key=value`; repeatable. Override a tunable for every recipe (`tunable:CritShake.Strength=0.8`). CLI: `--set`,
`--preset` (expanded to one `-scion-set` per value).

### -scion-motion

No value. Write the motion pack for every recipe. CLI: `--motion`.

### -scion-mosaic

An integer K, 2 to 16. Run K arenas in this process. CLI: `--mosaic`.

## Rules

- Flags are case-sensitive and take their value as the next argument.
- A flag repeated (except `-scion-recipes` and `-scion-set`) keeps the first value; the CLI places overrides first.
- Errors in flags write `run.json` with `error` and exit 2 before anything plays.

## See also

- [trial run](/scion/41664431a8f5/cli/trial-run/)
- [Exit codes](/scion/41664431a8f5/reference/exit-codes/)
- [How trials work](/scion/41664431a8f5/trials/overview/)
- [run.json](/scion/41664431a8f5/reference/files/run-json/)

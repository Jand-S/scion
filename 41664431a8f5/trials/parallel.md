---
title: "Parallel copies"
description: "Run N copies of the test build at once with shards, choose N from cores and free memory, and keep runs, timeouts and the watchdog under control."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/trials/parallel/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/trials/parallel.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/warm-copies.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/mosaic.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/trial-run.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Parallel copies

`trial run -j N` starts N copies of the test build, each with a shard `i/N` of the (id, round) pairs, and merges
their folders into one run. This guide covers how shards split the work, how to choose N (`doctor` suggests it from
cores and free memory), timeouts, the watchdog, `keep_runs`, and measured costs on a laptop.

## When to use

- Always: even two copies halve the time of a long queue.
- Raise `-j` until the machine runs out of cores or memory; past that, use warm copies or the mosaic.

## Steps

### 1. Ask doctor

```sh
dotnet scion doctor
```

The output has `suggested_copies`: the smaller of (cores - 1) and (free memory / memory per copy measured on the
last run, or 600 MB before the first).

### 2. Run with that number

```sh
dotnet scion trial run -j 6 --rounds 4
```

### 3. Set it as the default

```json
{"schema_version":1,"trials":{"defaults":{"copies":6,"rounds":1,"timeout_s":300}}}
```

## Reference

### Shards

Every (round, id) pair, sorted by round and then by id (ordinal), is assigned round-robin: pair `k` goes to shard
`k mod N`. Each pair runs in exactly one copy. A copy with no pairs exits 0 at once. Co-op groups take the host's
shard ([Co-op trials](/scion/41664431a8f5/trials/coop/)).

### Options that affect copies

| Option | Default | Meaning |
|---|---|---|
| `-j N` | `trials.defaults.copies` | Copies (co-op groups count `Players` each). |
| `--rounds N` | 1 | Rounds of every id; shards split them. |
| `--timeout <s>` | `trials.defaults.timeout_s` (300) | Real seconds for one copy; past it, the copy is killed (`copy_timeout`, exit 3). |
| `--window` | off | Show the copies' windows instead of headless offscreen capture. |
| `--size WxH` | `trials.defaults.size` (1280x720) | Capture size. |

### Watchdog

Each copy rewrites `run.json` as it progresses. A copy that makes no progress for 120 seconds of real time ends with
exit 3 and `watchdog`. The CLI kills the whole process tree of a copy that times out.

### Keeping runs

Runs live in `trials.out` (default `.scion/runs/`), one folder per run named `yyyyMMdd-HHmmss`. At the start of each
run, the CLI deletes the oldest runs beyond `trials.keep_runs` (default 10). There is no separate clean-up command.

### Measured costs

On an Apple M5 (10 cores, 16 GB), Orchard's test build at 1280x720 offscreen:

| Measure | Value |
|---|---|
| Startup to first recipe | 2.1 s |
| Memory per copy | about 550 MB |
| One combat recipe (2 s of game time at 30 fps) | 1.4 s of real time |
| 120 recipe-rounds with `-j 1` | 172 s |
| 120 recipe-rounds with `-j 6` | 31 s |

Startup dominates short runs; warm copies remove it ([Warm copies](/scion/41664431a8f5/trials/warm-copies/)).

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `copy_timeout` | 3 | A copy passed `--timeout`, or was interrupted. | Read `q<i>/player.log`; raise `--timeout` for long queues. |
| `watchdog` | 3 | A copy stopped making progress for 120 s. | Look for a `WaitUntil` that never ends or a modal dialog. |
| `player_crashed` | 3 | A copy exited without writing its outputs. | Read `q<i>/player.log` and the crash report. |
| warning `low_memory` | 0 | Free memory is below what `-j` copies need. | Lower `-j`. |

## Examples

### Spread a long queue on a big machine

```sh
dotnet scion trial run -j 14 --rounds 2
```

### One copy for debugging, with a window

```sh
dotnet scion trial run SlashHit -j 1 --window
```

## See also

- [trial run](/scion/41664431a8f5/cli/trial-run/)
- [Warm copies](/scion/41664431a8f5/trials/warm-copies/)
- [Mosaic](/scion/41664431a8f5/trials/mosaic/)
- [doctor](/scion/41664431a8f5/cli/doctor/)

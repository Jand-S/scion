---
title: "trial run"
description: "Run recipes on N parallel copies of the test build, compare with baselines and print counts, failures, changes and the next step."
type: cli
url: https://jand-s.github.io/scion/41664431a8f5/cli/trial-run/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/cli/trial-run.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/overview.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/parallel.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/summary.md
  - https://jand-s.github.io/scion/41664431a8f5/agents/reading-results.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# trial run

`dotnet scion trial run` is the command you use most. It selects recipes, starts copies of the test build (or uses
warm copies), splits the work into shards and co-op groups, waits, compares every result with `scion/baselines/`,
writes `summary.json`, `summary.tsv` and `compare.json` in a new run folder, and prints a short envelope with counts,
failures, changes, determinism and the `next_step`. It also runs specs, variants, the mosaic and the motion pack.

## Synopsis

```text
dotnet scion trial run [patterns] [-j N] [--rounds N] [selection options] [capture options]
                       [--grafts <dll>] [--recipes <dll>] [--warm] [--spec <Name>] [--variants a,b]
                       [--build <path>] [-- player-args]
```

## Description

1. Selects ids: patterns, `--tag`, `--players`, `--mode`, `--seeds`, `--changed`, `--failed`; `Explicit` recipes only
   when named.
2. Creates `<trials.out>/<yyyyMMdd-HHmmss>/` and deletes runs beyond `keep_runs`.
3. Starts copies (cold) or sends `run` to warm copies (`--warm`). Co-op recipes run in groups of `Players` copies.
4. Waits, with `--timeout` per copy and the 120 s watchdog.
5. Compares each (id, round) with its baseline: hash, frames, images (cold runs only), rule digest.
6. Writes `summary.json` (the full envelope plus `copies[]`), `summary.tsv`, `compare.json`, `diff/` heatmaps.
7. Prints the envelope, with lists cut to stay short.

## Options

### Selection

| Option | Default | Meaning |
|---|---|---|
| `patterns` | every non-`Explicit` recipe | Names, ids or globs, comma- or space-separated. |
| `--tag <tags>` | | Recipes with any of the tags. |
| `--players <n>` | all | Recipes with that `Players`. |
| `--mode <name>` | none | One mode of `[assembly: TrialModes]`. |
| `--seeds <a-b>` | none | Repeat each id per seed; ids get `#<seed>`. |
| `--changed` | | The ids that were `changed` or `stale` in the last run. |
| `--failed` | | The ids that failed in the last run. |

### Execution

| Option | Default | Meaning |
|---|---|---|
| `-j N` | `trials.defaults.copies` | Parallel copies. |
| `--rounds N` | 1 | Rounds per id. |
| `--seed N` | 1 | Run seed (`RunSeed`). |
| `--timeout <s>` | 300 | Real seconds per copy. |
| `--check` | off | Frame-by-frame determinism report (`determinism.tsv`). |
| `--jitter <ms>` | 0 | Random real-time sleep per frame from the second round. |
| `--warm` | off | Use warm copies (`trial serve`); compares hash and frames only. |
| `--mosaic <K>` | off | K arenas per process. |
| `--build <path>` | discovery | The test build. |

### Capture

| Option | Default | Meaning |
|---|---|---|
| `--window` | headless | Render to a window. |
| `--size WxH` | `trials.defaults.size` | Capture size. |
| `--frames` | off | Every frame as PNG. |
| `--raw` | off | Keep raw frames. |
| `--sound` | muted | Audio on. |
| `--motion` | off | The motion pack. |
| `--learn-unstable <n>` | off | Run n extra times and learn the unstable mask (cap 2 %). |

### Code and values

| Option | Default | Meaning |
|---|---|---|
| `--grafts <dll>` | none | Inject a draft graft DLL at startup (`-scion-inject`); the run cannot be approved. |
| `--recipes <dll>` | none | Load recipes from a DLL; same `Name` replaces. Repeatable. |
| `--set <key>=<value>` | none | Override a tunable (`tunable:CritShake.Strength=0.8`). Repeatable. |
| `--preset <name\|blend>` | none | Apply a preset or blend to every id. |
| `--variants a,b,c` | none | One variant per preset or blend; ids get `[preset=<name>]`. |
| `--spec <Name>` | none | Run a spec; the envelope gets `spec`. |
| `-- args` | | Passed to every player; replaces the same flag in `player_args`. |

## Output

<!-- validate: cli/trial-run -->
```json
{"schema_version":1,"command":"trial run","ok":false,"run":"/Users/alex/orchard/.scion/runs/20261011-143200","build_id":"4c49a1e0b2f3","copies":6,"wall_s":27.1,"runs":{"recipes":6,"rounds":2,"total":12,"ok":11,"failed":1,"changed":1,"within":0,"px_changed":0,"new":0,"stale":0},"failed":[{"id":"KnightFalls","round":1,"copy":3,"error":"timeout (30s): unit down animation","report":"q3/KnightFalls/r1/report.md"}],"changed":[{"id":"SlashHit","vs_approved":"changed","hash":"same","image":"q0/SlashHit/r1/window.png","diff":"diff/SlashHit/r1/window.diff.png"}],"determinism":{"hash_stable":true,"frame_jitter":[]},"next_step":"read /Users/alex/orchard/.scion/runs/20261011-143200/q3/KnightFalls/r1/report.md; SlashHit changed only in images: see /Users/alex/orchard/.scion/runs/20261011-143200/diff/SlashHit/r1/window.diff.png"}
```

| Field | Meaning |
|---|---|
| `run` | The run folder; paths in the envelope are relative to it. |
| `copies` | Processes started (or warm copies used). |
| `runs` | Counts: ids, rounds, (id, round) pairs, ok, failed, and per verdict. |
| `failed[]` | `id`, `round`, `copy`, `error`, `report`. |
| `changed[]` | `id`, `vs_approved`, `hash` (`same`/`changed`), `image`, `diff`. |
| `determinism` | `hash_stable`, `frame_jitter`. |
| `mosaic` | K, when used. |
| `spec` | With `--spec`: `name`, `ok`, `variants[]`, `verdict`. |

Files: [summary.json, summary.tsv, compare.json](/scion/41664431a8f5/reference/files/summary/).

## Exit codes

| Code | When |
|---|---|
| 0 | Every recipe passed. Verdicts such as `changed` do not fail a run: `ok` stays `true` and `changed[]` lists them ([`ok` and `changed`](/scion/41664431a8f5/cli/#ok-and-changed)). |
| 1 | A recipe or check failed, or a spec failed. |
| 2 | Bad option or a pattern matched nothing. |
| 3 | No build, a copy crashed or timed out, environment errors. |

Copies are combined with precedence 2 > 3 > 1 > 0.

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `no_match` | 2 | No id matches. | `did_you_mean`; `trial list --ids`. |
| `build_not_found` | 3 | No test build. | `dotnet scion build test`. |
| `build_wrong_mode` | 3 | The build lacks `SCION_TRIALS`. | `dotnet scion build test`. |
| `player_crashed` | 3 | A copy exited without outputs. | Read `q<i>/player.log`. |
| `copy_timeout` | 3 | A copy passed `--timeout` or was interrupted. | Raise `--timeout`, or find the stuck `WaitUntil`. |
| `no_free_port` | 3 | No free port for a group. | Close other servers. |
| `no_copies` | 3 | `--warm` with no warm copies. | `dotnet scion trial serve -j 2`. |
| `mosaic_unsupported` | 3 | `--mosaic` and no `IMosaicAdapter`. | Implement it or drop `--mosaic`. |
| warning `build_stale` | 0 | Sources changed after the build. | Rebuild before trusting the result. |
| warning `px_unstable` | 0 | Pixels vary between rounds. | See [Verify determinism](/scion/41664431a8f5/determinism/verify/). |
| `shard_ignored` | 3 | Two copies ran the same (recipe, round): the build ignores `-scion-shard`. | `dotnet scion build test`, or run with `-j 1`. |

## Examples

### The everyday run

```sh
dotnet scion trial run --rounds 2
```

### Combat only, with the motion pack, at 720p

```sh
dotnet scion trial run --tag combat --motion --size 1280x720
```

### A draft graft against every recipe

```sh
dotnet scion trial run --grafts .scion/grafts/ScionGrafts_51c2.dll
```

## See also

- [How trials work](/scion/41664431a8f5/trials/overview/)
- [Parallel copies](/scion/41664431a8f5/trials/parallel/)
- [summary.json, summary.tsv, compare.json](/scion/41664431a8f5/reference/files/summary/)
- [Reading results](/scion/41664431a8f5/agents/reading-results/)
- [Player command line](/scion/41664431a8f5/reference/player-flags/)

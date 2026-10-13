---
title: "How trials work"
description: "One test build launched as N copies, each playing a share of a recipe queue, with captures and verdicts against approved baselines."
type: concept
url: https://jand-s.github.io/scion/41664431a8f5/trials/overview/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/trials/overview.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/recipes.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/adapters.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/baselines.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/trial-run.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# How trials work

Trials are Scion's automated test lab. A **recipe** describes one situation to play (a slash at 3 tiles, a co-op
turn, a HUD state); the game's **adapter** knows how to play it; the **test build** of the game plays a queue of
recipes headless and writes captures, telemetry and a verdict for each. The CLI launches N copies of that one build
in parallel, splits the queue between them and compares every result with the approved **baseline**.

## How it works

![Diagram: dotnet scion trial run starts three copies of the test build; each plays its shard of the queue and writes its folder; the CLI compares the results with scion/baselines and writes summary.json.](../../../assets/diagrams/run-flow.svg)

### One build, many copies

There is no special test scene and no test runner: the test build is the game, compiled with `SCION_SLOTS` and
`SCION_TRIALS`. Started with `-scion-trials`, it creates a `TrialSession`, discovers every `TrialRecipe` class,
picks an adapter for each, plays the queue and, with `-scion-quit`, exits with a code (0 ok, 1 a recipe failed,
2 usage, 3 environment).

`dotnet scion trial run` does the orchestration:

1. reads `scion.json`, finds the test build and checks it was built in the right mode;
2. starts `-j N` copies (default: the `doctor` suggestion from cores and free memory), each with `-scion-shard i/N`
   and its own `q<i>/` folder;
3. waits for them, with a watchdog for copies that stop making progress;
4. compares each (recipe, round) with `scion/baselines/` and writes `summary.json`, `summary.tsv` and `compare.json`;
5. prints a short JSON with the counts, the failures, the changes and the `next_step`.

### Recipes, rounds and shards

Each recipe expands into one or more **ids** (variants, modes and seeds). The queue is every (id, round) pair; a
shard `i/n` plays exactly the pairs assigned to it, round-robin over (round, id). Rounds of the same id in one
process are compared with each other: the same state hash every round is the first determinism check.

### Captures

During a recipe the adapter marks a **cut** (`BeginCut`/`EndCut`): every frame rendered in the cut is hashed and
can be saved. The session writes `window.png` (up to 12 frames around the impact), `context.png` (6 frames across
the cut), named stills and, for static trials, `sheet.png`. Capture is offscreen and works in `-batchmode`; time is
virtual, so the same recipe gives the same frames on any machine.

### Verdicts

| Verdict | Meaning | Exit |
|---|---|---|
| `same` | Hash, frame count and images match the baseline. | 0 |
| `within` | Images differ but inside the tolerance. | 0 |
| `px_changed` | Hash and frames match; pixel hashes differ and the recipe compares no images. Informative. | 0 |
| `changed` | Hash, frame count (beyond ±1) or images outside the tolerance differ. | 0 with `ok: true` and `changed[]` filled: a review signal, not a failure ([`ok` and `changed`](/scion/41664431a8f5/cli/#ok-and-changed)) |
| `new` | No `baseline.json` for this id. | 0 |
| `stale` | The baseline is for another platform, fps, Unity or player args; images are suspended, the hash still counts. | 0 |

A recipe that fails (timeout, `Fail`, a check, an error in the log, a graft that threw) exits 1 whatever its verdict.

### The adapter is the only game-specific code

Everything above is generic. The game writes recipes (data) and one or more adapters (how to load a scene, start a
battle, issue a command, read a state hash). The more the game follows the [Scion Standard](/scion/41664431a8f5/standard/overview/),
the thinner the adapter.

## Key terms

| Term | Meaning |
|---|---|
| Recipe | A `TrialRecipe` subclass: the data of one situation. |
| Id | One concrete instance of a recipe: `SlashHit`, `SlashHit[Distance=3]@360p#7`. |
| Adapter | The game's `TrialAdapter<TRecipe>` that plays recipes of a type. |
| Round | One play of an id; `--rounds 2` plays every id twice. |
| Copy | One process of the test build. |
| Shard | The share `i/n` of the queue one copy plays. |
| Group | The copies that play one co-op recipe together (a host and its clients). |
| Run | One `trial run`: a folder under `trials.out` named by its start time. |
| Cut | The frames between `BeginCut` and `EndCut`. |
| Baseline | The approved hash, frames and images of an id, in `scion/baselines/`. |

## Guarantees and limits

- The same build, recipe and seed give the same state hash on every machine and every round; frame counts may differ
  by one until the Netcode clock is virtual ([Netcode clock](/scion/41664431a8f5/determinism/netclock/)).
- Copies never write into each other's folders; the CLI never writes inside a copy's folder.
- Images are compared per platform (OS, graphics API, size, capture); the hash is compared everywhere.
- A copy that makes no progress for 120 seconds of real time is ended (`watchdog`, exit 3).
- Runs beyond `trials.keep_runs` (default 10) are deleted at the start of the next run.

## See also

- [Quickstart](/scion/41664431a8f5/start/quickstart/)
- [Write a recipe](/scion/41664431a8f5/trials/recipes/)
- [Write an adapter](/scion/41664431a8f5/trials/adapters/)
- [Baselines and approval](/scion/41664431a8f5/trials/baselines/)
- [trial run](/scion/41664431a8f5/cli/trial-run/)

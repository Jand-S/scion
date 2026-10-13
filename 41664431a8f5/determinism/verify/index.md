---
title: "Verify determinism"
description: "Prove recipes are deterministic with rounds, frame-by-frame comparison, timing jitter and the determinism.tsv report, and run it nightly."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/determinism/verify/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/determinism/verify.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/determinism/diagnose.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/tables.md
  - https://jand-s.github.io/scion/41664431a8f5/cookbook/ci.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Verify determinism

Determinism is verified, not assumed. Two or more rounds of a recipe in one process must give the same state hash
(the `determinism` check) and the same pixels (`px`). `--check` compares every later round frame by frame with the
first and writes `determinism.tsv` with the first frame that differs and the first cause; `--jitter` adds random
real-time delays to expose code that follows the wall clock.

## When to use

- Run 2 rounds on every change (`--rounds 2`): it costs little and catches most problems.
- Run `--check --jitter 20` nightly, on many rounds, to catch the rare ones.
- Run `--check` when a round differs and you need to know where.

## Steps

### 1. Two rounds

```sh
dotnet scion trial run --rounds 2
```

The output's `determinism` block says whether every hash was stable and which recipes jittered by a frame:

```json
{"hash_stable":true,"frame_jitter":["CoopTurn 212/213"]}
```

### 2. Find where rounds diverge

```sh
dotnet scion trial run FireballArea --rounds 3 --check
```

`q0/determinism.tsv`:

```tsv
id	round	vs_round	status	frame	cause	detail
FireballArea	2	1	same	-1	none	
FireballArea	3	1	diff	17	rng	Random.state differs from frame 17
```

### 3. Shake the timing

```sh
dotnet scion trial run --rounds 4 --check --jitter 20
```

From the second round of each id, every frame sleeps a random 0 to 20 ms of real time. Deterministic code does not
notice; code that reads the real clock does.

## Reference

### Causes, in order

`cause` is the first column of `frames.tsv` that differs, in this order:

| Cause | Column | Usually means |
|---|---|---|
| `state` | the adapter's `state` sensor | A rule diverged. Look at the frame's events. |
| `tick` | the adapter's `tick` sensor | Network or fixed-step timing ([Netcode clock](/scion/41664431a8f5/determinism/netclock/)). |
| `rng` | `UnityEngine.Random` state | Something consumed random numbers in a different amount. |
| `unscaled` | virtual unscaled time | A real-time read leaked. |
| `time` | virtual time | `Time.timeScale` changed differently. |
| `px` | pixel hash | Only presentation differs ([Particles, animators, UI and physics](/scion/41664431a8f5/determinism/control/)). |
| `frames` | frame count | The cut ended on a different frame. |

Register the `state` and `tick` sensors in the adapter's `Prepare` so the first two causes can be found.

### Options

| CLI | Player flag | Meaning |
|---|---|---|
| `--rounds N` | `-scion-rounds N` | Rounds per id. |
| `--check` | `-scion-check` | Frame-by-frame comparison; writes `determinism.tsv`. Never changes a verdict. |
| `--jitter <ms>` | `-scion-jitter <ms>` | Random real-time sleep per frame from the second round; 0 to 1000. |

### In the CLI output

| Field | Meaning |
|---|---|
| `determinism.hash_stable` | Every id had the same hash in every round. |
| `determinism.frame_jitter` | Ids whose frame counts differed by one, as `Id a/b`. |
| warning `px_unstable` | Pixel hashes varied between rounds. |

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| check `determinism` fails | 1 | Hashes differ between rounds. | `--check`; fix the cause at the frame. |
| check `px` fails | 1 | Pixels differ between rounds. | See `cause` `px`. |
| warning `frame_jitter` | 0 | Frame counts off by one. | Usually Netcode; turn on `SCION_NETCLOCK`. |

## Examples

### A nightly batch

```sh
dotnet scion trial run --rounds 10 --check --jitter 20 -j 8
```

### One recipe, many seeds

```sh
dotnet scion trial run KnightFalls --seeds 1-50 --rounds 2
```

## See also

- [Find non-determinism](/scion/41664431a8f5/determinism/diagnose/)
- [results.tsv, frames.tsv, determinism.tsv](/scion/41664431a8f5/reference/files/tables/)
- [Run Scion in CI](/scion/41664431a8f5/cookbook/ci/)

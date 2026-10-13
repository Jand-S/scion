---
title: "TrialSession"
description: "Every member of TrialSession - capture, verdict, data, waiting, queues and properties - with what each call writes to the run folder."
type: reference
url: https://jand-s.github.io/scion/41664431a8f5/trials/session/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/trials/session.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/adapters.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/checks.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/session.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# TrialSession

`TrialSession` is the object adapters, grafts and checks talk to during a run. The player creates it when started
with `-scion-trials` (`TrialSession.Current`); adapters receive it in every call. Call its methods on the main thread:
a call from another thread does nothing and logs an error, which fails the recipe through the `log` check.

## Properties

| Member | Meaning |
|---|---|
| `static TrialSession Current` | The running session, or null without `-scion-trials`. |
| `static event Action Boundary` | Raised on the main thread after each recipe and once after the queue. Reset leaking state here. |
| `string OutputRoot` | The queue's output folder (`-scion-out`, or the folder of the last `RunQueue`). |
| `TrialRecipe Recipe` | The recipe running now, or null between recipes. |
| `string RecipeDir` | `<OutputRoot>/<IdDir>/r<Round>`. |
| `int Round` | The round being played, from 1. |
| `bool InRecipe` | True from the start of a recipe until its outputs are written. |
| `int RunSeed` | `-scion-seed` (`--seed`), 1 by default. |
| `IReadOnlyList<TrialRecipe> Recipes` | Every recipe found and expanded into ids, before filters. |
| `TrialRole Role` | `host` or `client:i` in co-op groups; `host` alone. |

## Capture

| Method | What it does |
|---|---|
| `BeginCut()` | Starts capturing every rendered frame. Call it right before the action. |
| `EndCut()` | Stops capturing. The session also calls it when `Run` ends. |
| `Mark(string label)` | Labels the current frame (`frames.tsv`, `report.md`, `Marked` and `Order` checks, `context.png`). Ignored outside a cut. Labels may not contain a comma, `=`, tab or line break. |
| `MarkImpact()` | Marks the moment that matters: `window.png` centers on the first call; each call adds the label `IMPACT`. |
| `Still(string name)` | Saves the next rendered frame as `<name>.png`, inside or outside a cut. Names: up to 64 of `A-Za-z0-9_-.`; not `window`, `context`, `sheet`, `unstable`. Let a frame render after it. |
| `MaskRect(Rect screenPixels, string why)` | A region image comparison ignores in this recipe, in screen pixels, origin at the bottom left. |

## Verdict

| Method | What it does |
|---|---|
| `Fail(string reason)` | Fails the recipe and stops `Run` (or `Prepare`) at its next `yield`. `Teardown`, outputs and checks still run. The first reason is kept. Between recipes it fails the next one; after the last, the queue. |
| `Check(bool ok, string what)` | A soft check (id `assert`): when false, the recipe keeps running and fails at its end with `what`. |
| `Note(string line)` | A line in the recipe's `report.md`, stamped with game time, and in the log with `[Scion]`. |

## Data

| Method | What it does |
|---|---|
| `Probe(string key, double value)` | A number. The last value per key goes to `result.json` (`probes`) and is what `ProbeIs` tests; every call goes to `telemetry.jsonl`. |
| `Sensor(string name, Func<long> read)` | A `frames.tsv` column read once per captured frame. Lasts for the process: register in `Prepare`. Names `[a-z0-9_]+`. Keep `read` cheap and allocation-free. |
| `Event(string name, params (string Key, object Value)[] fields)` | A `"kind":"event"` line in `telemetry.jsonl`. Co-op uses `Event("turn", ("turn", n), ("hash", h))`. |

## Waiting

| Method | What it does |
|---|---|
| `IEnumerator WaitUntil(Func<bool> cond, string what)` | Checks now, then every frame. On timeout: `timeout (30s): <what>`. |
| `IEnumerator WaitSeconds(float gameSeconds)` | Waits game time on the recipe's clock: `0.5f` is 15 frames at 30 fps on any machine. |

## Queues

```csharp
IEnumerator RunQueue(string only, int rounds, string outDir, Action<int> done = null)
```

Plays another queue in the same process without restarting the game; adapters already prepared skip `Prepare`.
`only` is a comma-separated pattern list (null = every non-`Explicit` recipe); `outDir` gets its own `run.json`,
`results.tsv` and recipe folders. `done` receives the exit code. Warm copies run every queue this way (op `run` of
the [control protocol](/scion/41664431a8f5/reference/control-protocol/)). Throws `InvalidOperationException` when a queue is
already playing.

## Order of one recipe

| Moment | What happens |
|---|---|
| Start | `Recipe`, `Round`, `RecipeDir` set; `Random.InitState(Seed)`; `captureFramerate = Fps`; `Clock.Begin(1000)`; `SeededRandom.Reset(Seed)`; determinism controls on; checks `Begin`. |
| Run | `adapter.Run`, guarded; timeout = `TimeoutSeconds` of game time. |
| End | `EndCut`; `StateHash` read; PNG encoding starts; `Teardown`; PNGs awaited; checks `End`; then `frames.tsv`, `telemetry.jsonl`, `log.txt`, `result.json`, `report.md`; a line in `results.tsv`; `Boundary`. |

## See also

- [Write an adapter](/scion/41664431a8f5/trials/adapters/)
- [Checks](/scion/41664431a8f5/trials/checks/)
- [Session API](/scion/41664431a8f5/reference/api/session/)
- [Telemetry, probes and sensors](/scion/41664431a8f5/evaluation/telemetry/)

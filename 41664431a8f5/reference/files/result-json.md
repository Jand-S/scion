---
title: "result.json"
description: "The verdict of one recipe round - ok, error, state and pixel hashes, frames, checks, probes, images, masks and timing."
type: file
url: https://jand-s.github.io/scion/41664431a8f5/reference/files/result-json/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/files/result-json.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/checks.md
  - https://jand-s.github.io/scion/41664431a8f5/agents/reading-results.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/run-folder.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# result.json

`result.json` is the verdict of one id in one round: whether it passed and why not, the state hash and pixel hash,
the frame count, every check with its status, the last value of every probe, the images with their frames and marks,
the masks and the timing. It is the contract between the test player, the CLI and the judge.

## Location

`<scion-out>/<IdDir>/r<round>/result.json`.

## Written by and read by

Written by the test player after the recipe's teardown, with the final verdict. Read by the CLI (summaries,
baselines), `trial show`, agents and the judge.

## Schema

`https://jand-s.github.io/scion/41664431a8f5/schemas/v1/result.schema.json`

## Fields

| Field | Meaning |
|---|---|
| `id`, `name`, `variant`, `mode`, `round`, `role`, `arena` | Which instance this is. |
| `ok` | The verdict; equals the `ok` column of `results.tsv`. |
| `error` | The first failure reason, or null. |
| `hash` | The adapter's state hash, or null. |
| `px_hash` | xxHash64 of the sequence of frame hashes (16 hex). |
| `frames`, `fps` | Frames in the cut; capture rate. |
| `seed`, `run_seed`, `size` | Seeds and capture size. |
| `grafts` | `Injection.Version` (0 without injection). |
| `checks[]` | `id`, `status` (`pass`, `warn`, `fail`, `skipped`), `reason`, `frame`, `region` ([x, y, w, h], top-left origin). |
| `probes` | Last value per key. |
| `images[]` | `file`, `role` (`window`, `context`, `still`, `sheet`, `motion-diff`, `tracks`, `preview`), `frames`, `marks`. |
| `masks[]` | `artifact`, `rect` (pixels, top-left origin), `why`. |
| `timing` | `seconds` and parts: `setup_s`, `cut_s`, `read_s`, `wait_s`, `outputs_s`, `switch_seconds`. |

## Example

<!-- validate: result -->
```json
{"schema_version":1,"id":"KnightFalls","name":"KnightFalls","variant":null,"mode":null,"round":1,"role":"host","arena":null,"ok":false,"error":"timeout (30s): unit down animation","hash":"1B44E07AC9D20F13","px_hash":"0c1d2e3f4a5b6c7d","frames":52,"fps":30,"seed":1,"run_seed":1,"size":[1280,720],"grafts":0,"checks":[{"id":"log","status":"pass"},{"id":"order","status":"fail","reason":"unit_down was never marked","frame":-1,"region":null}],"probes":{"damage":3,"defender_hp":0},"images":[{"file":"window.png","role":"window","frames":[20,21,22,23,24,25,26,27,28,29,30,31],"marks":["IMPACT"]},{"file":"before.png","role":"still","frames":[],"marks":[]}],"masks":[{"artifact":"*","rect":[0,0,1280,40],"why":"turn timer"}],"timing":{"seconds":2.1,"setup_s":0.4,"cut_s":1.5,"read_s":0.1,"wait_s":0,"outputs_s":0.05,"switch_seconds":0.3}}
```

## Compatibility

Fields may be added in minor versions. Check ids are stable; new built-in checks get new ids.

---
title: "baseline.json and platform.json"
description: "The approved state of a recipe id - hash, frames and approval in baseline.json, and per platform the frame hashes, images and tolerance."
type: file
url: https://jand-s.github.io/scion/41664431a8f5/reference/files/baselines/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/files/baselines.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/baselines.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/trial-approve.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/image-comparison.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# baseline.json and platform.json

A baseline has two parts. `baseline.json` holds what does not depend on the platform: the state hash, the frame
count, fps, player arguments, the rule digest and who approved it, when and why. Each platform folder holds what does:
`platform.json` (size, Unity, tolerance, images), `frames.tsv` with the frame hashes, the approved images and an
optional learned mask.

## Location

```text
scion/baselines/<IdDir>/baseline.json
scion/baselines/<IdDir>/<platform>/platform.json
scion/baselines/<IdDir>/<platform>/frames.tsv
scion/baselines/<IdDir>/<platform>/window.png  context.png  <still>.png  unstable.png
```

## Written by and read by

Written only by `trial approve` (the `guard` hook denies other edits). Read by every `trial run` for verdicts and by
`trial show --grid`.

## Schema

- `https://jand-s.github.io/scion/41664431a8f5/schemas/v1/baseline.schema.json`
- `https://jand-s.github.io/scion/41664431a8f5/schemas/v1/platform.schema.json`

## Fields

### baseline.json

| Field | Meaning |
|---|---|
| `id`, `hash`, `frames`, `fps` | The approved id, state hash, frame count and capture rate. |
| `player_args` | Normalized player arguments (without `-scion-out`, ports, shards, control, `-logFile`). |
| `rule_digest` | 12 hex over the files in `rule_paths` at approval. |
| `build_id` | The build the run used. |
| `approved_by`, `approved_via`, `reason`, `approved_at` | Who (`tty` or `script`), why, when. |

### platform.json

| Field | Meaning |
|---|---|
| `platform`, `size`, `unity`, `build_id` | Where these images come from. |
| `approved_by`, `approved_via`, `approved_at` | Approval on this platform. |
| `tolerance` | `pixel`, `max_ratio`, `max_pixels`, or null when the recipe compares no images. |
| `images[]` | `file`, `role`. |

## Example

<!-- validate: baseline -->
```json
{"schema_version":1,"id":"SlashHit","hash":"9F3A0C11D2E4B5A6","frames":46,"fps":30,"player_args":[],"rule_digest":"a1b2c3d4e5f6","build_id":"4c49a1e0b2f3","approved_by":"alex","approved_via":"tty","reason":"crit shake is stronger","approved_at":"2026-10-11T15:00:00Z"}
```

<!-- validate: platform -->
```json
{"schema_version":1,"platform":"osx-metal-1280x720-offscreen","size":[1280,720],"unity":"6000.3.25f1","build_id":"4c49a1e0b2f3","approved_by":"alex","approved_via":"tty","approved_at":"2026-10-11T15:00:00Z","tolerance":{"pixel":0.1,"max_ratio":0.001,"max_pixels":-1},"images":[{"file":"window.png","role":"window"},{"file":"context.png","role":"context"},{"file":"before.png","role":"still"}]}
```

## Compatibility

A baseline written by any 1.x version is read by every later 1.x version. Store PNGs with Git LFS.

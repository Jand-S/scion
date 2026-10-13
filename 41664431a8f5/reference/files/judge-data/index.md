---
title: "Judge data files"
description: "Where the judge keeps heads, questions, votes and gold sets, and the shape of a head file and a vote line."
type: file
url: https://jand-s.github.io/scion/41664431a8f5/reference/files/judge-data/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/files/judge-data.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/judge/training.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/judge.md
  - https://jand-s.github.io/scion/41664431a8f5/judge/models.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Judge data files

The judge's data is small and versioned: heads (what each one asks and how well it agrees with people), questions,
votes (one JSON line each) and gold sets. Embeddings and model weights are large and stay out of git, in the cache.

## Location

```text
scion/judge/                        judge.dir
  questions/<head>.json             question files
  heads/<head>.json                 current version of each head
  heads/<head>@<version>.bin        trained weights per version
  votes/<head>.jsonl                one vote per line
  gold/<set>.jsonl                  gold votes, never used for training
  references/                       optional reference screens and notes
.scion/judge/                       judge.cache: embeddings per eyes and view
```

## Written by and read by

Written by `judge new`, `judge vote` and `judge train`. Read by `judge run`, `judge show`, `trial eval` (default
questions) and `search run`.

## Schema

- `https://jand-s.github.io/scion/41664431a8f5/schemas/v1/head.schema.json`
- `https://jand-s.github.io/scion/41664431a8f5/schemas/v1/vote.schema.json` (each line)

## Fields

### Head

| Field | Meaning |
|---|---|
| `name`, `version` | Head name and current version. |
| `eyes`, `view` | The encoder and the view (`full`, `hud`, `corners`, `actor`). |
| `question` | The question it answers. |
| `trained_at`, `votes` | Last training and how many votes it used. |
| `calibration` | `gold_set`, `agreement`, `ceiling` (people's agreement with themselves). |

### Vote

| Field | Meaning |
|---|---|
| `head`, `at`, `by`, `source` | Which head, when, who, `person` or `agent`. |
| `a`, `b` | Image or sequence paths (`b` null for single-item labels). |
| `winner`, `strength` | For duels: `a`, `b` or `tie`; 1 slight, 2 clear. |
| `value` | For scores and choices. |
| `tags` | Free tags (`synthetic:contrast`). |

## Example

<!-- validate: head -->
```json
{"schema_version":1,"name":"readability","version":4,"eyes":"siglip2-so400m","view":"hud","question":{"type":"score","instructions":"How easy is the HUD to read at 1x on a laptop screen?"},"trained_at":"2026-10-12T08:10:00Z","votes":412,"calibration":{"gold_set":"ui_v1","agreement":0.81,"ceiling":0.86}}
```

<!-- validate: vote -->
```json
{"schema_version":1,"head":"hit_weight","at":"2026-10-12T09:01:12Z","by":"alex","source":"person","a":".scion/runs/20261012-091500/q0/SlashHit_preset=heavy_/r1/preview.gif","b":".scion/runs/20261012-091500/q1/SlashHit_preset=snappy_/r1/preview.gif","winner":"a","strength":2,"tags":[]}
```

## Compatibility

Votes are append-only. A head file keeps working across 1.x; changing `judge.eyes` requires retraining.

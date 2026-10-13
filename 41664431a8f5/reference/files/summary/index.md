---
title: "summary.json, summary.tsv, compare.json"
description: "The CLI's run summaries - the full trial run envelope with copies, one line per id round copy, and the per-id comparison with baselines."
type: file
url: https://jand-s.github.io/scion/41664431a8f5/reference/files/summary/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/files/summary.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/cli/trial-run.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/baselines.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/run-folder.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# summary.json, summary.tsv, compare.json

After every `trial run`, the CLI writes three files at the top of the run folder: `summary.json` (the full envelope,
uncut, plus the copies), `summary.tsv` (one line per id, round and copy) and `compare.json` (per id, the verdict
against the baseline with its reasons, hashes, frames and image results).

## Location

`<run>/summary.json`, `<run>/summary.tsv`, `<run>/compare.json`.

## Written by and read by

Written by the CLI. Read by `trial show`, `trial approve`, `trial eval`, agents and CI.

## Schema

- `https://jand-s.github.io/scion/41664431a8f5/schemas/v1/summary.schema.json`
- `https://jand-s.github.io/scion/41664431a8f5/schemas/v1/compare.schema.json`
- `summary.tsv` follows the [TSV rules](/scion/41664431a8f5/reference/files/tables/); its version is `summary.json`'s.

## Fields

### summary.json

The `trial run` envelope without cuts, plus `copies[]`: `q`, `pid`, `role`, `group`, `exit`, `wall_s`, `out`, `log`.

### summary.tsv columns

`id round copy role ok hash frames vs_approved px_diff img_diff eval`

| Column | Meaning |
|---|---|
| `px_diff` | Fraction of frames whose `px` differs from the baseline. |
| `img_diff` | `Ratio` of the worst artifact in the image comparison. |
| `eval` | Evaluator score, after `trial eval`. |

Empty means not applicable.

### compare.json

Per id: `vs_approved`, `reasons` (`baseline`, `hash`, `frames`, `px`, `image`, `rule_hash`), `stale` (`platform`,
`fps`, `player_args`, `unity`), `hash` (`approved`, `current[]`, `stable`), `frames` (`approved`, `current[]`),
`px_diff`, `px_unstable`, `images[]` (`file`, `status`, `ratio`, `bounds`, `diff`).

## Example

<!-- validate: compare -->
```json
{"schema_version":1,"recipes":{"SlashHit":{"vs_approved":"changed","reasons":["image"],"stale":[],"hash":{"approved":"9F3A0C11D2E4B5A6","current":["9F3A0C11D2E4B5A6"],"stable":true},"frames":{"approved":46,"current":[46]},"px_diff":0.12,"px_unstable":0,"images":[{"file":"window.png","status":"changed","ratio":0.003,"bounds":[400,180,64,64],"diff":"diff/SlashHit/r1/window.diff.png"}]}}}
```

## Compatibility

Fields may be added in minor versions. Verdict and reason values are stable within 1.x.

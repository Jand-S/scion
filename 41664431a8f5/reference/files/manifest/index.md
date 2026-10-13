---
title: "scion.manifest.json and GRAFTS.md"
description: "The generated inventory of anchors, grafts, tunables, recipes and probes, as JSON for tools and as Markdown tables for people."
type: file
url: https://jand-s.github.io/scion/41664431a8f5/reference/files/manifest/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/files/manifest.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/cli/gen.md
  - https://jand-s.github.io/scion/41664431a8f5/agents/project-index.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# scion.manifest.json and GRAFTS.md

`dotnet scion gen` writes two files at the project root from the code: `scion.manifest.json`, the inventory tools and
agents read (every anchor, graft, tunable, recipe and probe), and `GRAFTS.md`, the same data as tables for people and
pull requests. Both are generated: edit the code, then run `gen`.

## Location

`scion.manifest.json` and `GRAFTS.md` at the project root (next to `scion.json`), versioned.

## Written by and read by

Written only by `dotnet scion gen` (the `guard` hook denies hand edits). Read by `index`, the MCP server, agents,
reviewers and CI (`gen --check`).

## Schema

`https://jand-s.github.io/scion/41664431a8f5/schemas/v1/manifest.schema.json`. `GRAFTS.md` has no schema: one table per anchor.

## Fields

| Field | Items |
|---|---|
| `anchors[]` | `id`, `target`, `signature`, `doc`, `slot`, `used_by` (graft names) |
| `grafts[]` | `name`, `file`, `line`, `kind`, `target`, `priority`, `observe`, `promoted` |
| `tunables[]` | `key`, `type`, `min`, `max`, `value`, `layer` (`visual` or `rule`), `doc` |
| `recipes[]` | `name`, `type`, `players`, `tags`, `description` |
| `probes[]` | `key`, `member` |
| `scion`, `generated_from` | Version and source (`build` or `source`). |

## Example

<!-- validate: manifest -->
```json
{"schema_version":1,"scion":"1.0.0","generated_from":"build","anchors":[{"id":"presentation.hit","target":"Orchard.Presentation.HitPresenter.Play","signature":"float Play(HitEvent e)","doc":"Plays one hit and returns how long it lasts.","slot":true,"used_by":["Orchard.Grafts.CritShake.After","Orchard.Grafts.HitFlash.After"]}],"grafts":[{"name":"Orchard.Grafts.CritShake.After","file":"Assets/Orchard/Grafts/CritShake.cs","line":12,"kind":"after","target":"Orchard.Presentation.HitPresenter.Play","priority":0,"observe":false,"promoted":true}],"tunables":[{"key":"CritShake.Strength","type":"float","min":0,"max":1,"value":0.3,"layer":"visual","doc":"Camera shake on a critical hit."},{"key":"Balance.CritPermille","type":"int","min":500,"max":2000,"value":1500,"layer":"rule","doc":"Critical damage multiplier, in permille."}],"recipes":[{"name":"SlashHit","type":"Orchard.Trials.SlashHit","players":1,"tags":["combat","melee"],"description":"The knight slashes a skeleton next to it."}],"probes":[{"key":"damage","member":"Orchard.Core.DamageCalc.Final"}]}
```

`GRAFTS.md`:

```md
## presentation.hit: HitPresenter.Play(HitEvent e)

Plays one hit and returns how long it lasts.

| Graft | Side | Priority | File |
|---|---|---|---|
| Orchard.Grafts.HitFlash.After | after | 0 | Assets/Orchard/Grafts/HitFlash.cs:9 |
| Orchard.Grafts.CritShake.After | after | 0 | Assets/Orchard/Grafts/CritShake.cs:12 |
```

## Compatibility

Sorted and stable: the same code always gives the same bytes, so diffs show only real changes.

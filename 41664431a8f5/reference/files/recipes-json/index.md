---
title: "recipes.json"
description: "The list of selected and expanded recipe ids a test player writes with -scion-list, with variants, modes, tags, players and settings."
type: file
url: https://jand-s.github.io/scion/41664431a8f5/reference/files/recipes-json/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/files/recipes-json.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/cli/trial-list.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/variants.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# recipes.json

`recipes.json` lists every id the current selection would play, after variants, modes and seeds are expanded, with
the settings the CLI needs to plan a run (players for co-op groups, explicit, fps, seed, timeout). The player writes
it and exits 0 without playing.

## Location

`<scion-out>/recipes.json`, only with `-scion-list`.

## Written by and read by

Written by the test player. Read by `trial list --ids`, `trial run` (to plan groups) and agents.

## Schema

`https://jand-s.github.io/scion/41664431a8f5/schemas/v1/recipes.schema.json`

## Fields

| Field | Meaning |
|---|---|
| `recipes[].id`, `name`, `variant`, `mode` | The instance. |
| `tags`, `description` | From the recipe. |
| `players`, `explicit` | Co-op size; whether it runs only when named. |
| `fps`, `seed`, `timeout_s` | Capture settings. |
| `type` | The recipe's C# type. |

## Example

<!-- validate: recipes -->
```json
{"schema_version":1,"recipes":[{"id":"SlashHit[Attacker=mage,Distance=3]","name":"SlashHit","variant":"Attacker=mage,Distance=3","mode":null,"tags":["combat","melee"],"description":"The knight slashes a skeleton next to it.","players":1,"explicit":false,"fps":30,"seed":7,"timeout_s":30,"type":"Orchard.Trials.SlashHit"},{"id":"CoopTurn","name":"CoopTurn","variant":null,"mode":null,"tags":[],"description":"Four players each act once; every peer agrees on the state.","players":4,"explicit":false,"fps":30,"seed":1,"timeout_s":60,"type":"Orchard.Trials.CoopTurn"}]}
```

## Compatibility

Fields may be added in minor versions.

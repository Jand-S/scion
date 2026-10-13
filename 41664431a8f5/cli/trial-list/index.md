---
title: "trial list"
description: "List the recipes of the test build and the recipe sources in scion/recipes, or every expanded id with variants, modes and seeds."
type: cli
url: https://jand-s.github.io/scion/41664431a8f5/cli/trial-list/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/cli/trial-list.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/recipes.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/variants.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/recipes-json.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# trial list

`dotnet scion trial list` lists the recipes a run can play. By default it reads recipe names from the test build's
assemblies (without starting it) and adds the sources in `scion/recipes/` and the captured and found matches. With
`--ids` it starts the build with `-scion-list` and prints every expanded id (variants, modes, seeds) with players,
tags and flags.

## Synopsis

```text
dotnet scion trial list [patterns] [--ids] [--tag <tags>] [--players <n>] [--build <path>]
```

## Description

| Source | `source` | Read how |
|---|---|---|
| The test build | `build` | Recipe classes found in `Managed/*.dll` without running the player. |
| `scion/recipes/*.cs` | `staging` | Recipe sources compiled with `--recipes`. |
| `scion/recipes/captured/*.match.json` | `captured` | One `CapturedMatch` id per file. |
| `scion/recipes/found/*.match.json` | `found` | Finds of the hunter. |

A recipe that cannot be built (no public parameterless constructor) is listed with `problem`.

## Options

| Option | Default | Meaning |
|---|---|---|
| `patterns` | all | Names, ids or globs. |
| `--ids` | off | Expand ids by starting the player with `-scion-list`. |
| `--tag <tags>` | | Only recipes with one of the tags (needs `--ids`). |
| `--players <n>` | | Only recipes with that `Players` (needs `--ids`). |
| `--build <path>` | discovery | The test build. |

## Output

<!-- validate: cli/trial-list -->
```json
{"schema_version":1,"command":"trial list","ok":true,"build":"/Users/alex/orchard/.scion/builds/test/Orchard.app","build_id":"4c49a1e0b2f3","count":7,"recipes":[{"name":"SlashHit","source":"build","players":1},{"name":"FireballArea","source":"build","players":1},{"name":"KnightFalls","source":"build","players":1},{"name":"CoopTurn","source":"build","players":4},{"name":"HudStates","source":"build","players":1},{"name":"UnitGallery","source":"build","players":1},{"name":"CapturedMatch","id":"CapturedMatch[match=knight-dies-to-crit]","source":"captured","players":1}],"next_step":"dotnet scion trial run"}
```

With `--ids`, each entry has `id`, `type`, `tags` and `explicit`, and the full list is in `recipes.json` of a
temporary folder.

## Exit codes

| Code | When |
|---|---|
| 0 | Listed. |
| 2 | A pattern matched nothing. |
| 3 | No build, or the player failed with `--ids`. |

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `no_match` | 2 | No recipe matches. | Use `did_you_mean`. |
| `build_not_found` | 3 | No test build. | `dotnet scion build test`. |
| warning `recipe_unbuildable` | 0 | A recipe has no public parameterless constructor (SCN201). | Add one. |
| warning `unreadable_dll` | 0 | A DLL of the build could not be read. | Rebuild. |

## Examples

### Every id of one recipe

```sh
dotnet scion trial list SlashHit --ids
```

### Only co-op recipes

```sh
dotnet scion trial list --ids --players 4
```

## See also

- [Write a recipe](/scion/trials/recipes/)
- [Variants, modes and seeds](/scion/trials/variants/)
- [recipes.json](/scion/reference/files/recipes-json/)

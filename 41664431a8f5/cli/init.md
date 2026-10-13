---
title: "init, new, uninstall"
description: "Set up Scion in a project with init, create a project or scaffold a feature, effect, graft or trial with new, and remove Scion with uninstall."
type: cli
url: https://jand-s.github.io/scion/41664431a8f5/cli/init/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/cli/init.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/start/new-project.md
  - https://jand-s.github.io/scion/41664431a8f5/start/existing-project.md
  - https://jand-s.github.io/scion/41664431a8f5/standard/enforcement.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# init, new, uninstall

Three commands manage Scion in a project. `dotnet scion init` adds Scion to an existing Unity project (`scion.json`,
`ScionSettings.json`, the grafts folder and `.asmref`, `.gitignore` entries, the agent integration). `dotnet scion
new` creates a whole project from a genre template, or scaffolds a feature, effect, graft or trial inside one.
`dotnet scion uninstall` removes what `init` added. All three accept `--dry-run` and report `created`, `changed`,
`removed` and `skipped` files.

## Synopsis

```text
dotnet scion init [--existing] [--agent claude|codex|cursor|none] [--game-assembly <name>] [--grafts-folder <path>] [--dry-run]
dotnet scion new --genre tactics|action|platformer|blank --name <Name> [--players 1-4] [--dry-run]
dotnet scion new feature|effect|graft|trial <Name> [--after|--before <Type.Method>] [--dry-run]
dotnet scion uninstall [--purge] [--dry-run]
```

## Description

`init` never overwrites: an existing file is listed in `skipped`. `uninstall` reverses `init` and keeps your grafts,
recipes and baselines unless `--purge`.

## init

| Creates or changes | Content |
|---|---|
| `scion.json` | Paths, defaults and builds for this project. |
| `ProjectSettings/ScionSettings.json` | `build.main_assembly`, analyzer scope. |
| `Assets/<Game>/Grafts/<Game>.Grafts.asmref` | Points the grafts folder at the game assembly. |
| `Packages/manifest.json` | Adds `me.jandson.scion` if missing. |
| `.config/dotnet-tools.json` | Adds `Scion.Cli`. |
| `.gitignore` | `.scion/`, `scion.local.json`, `Library/Scion/`. |
| `.gitattributes` | LFS for `scion/baselines/**/*.png`. |
| agent files | With `--agent claude`: the plugin's hooks in `.claude/settings.json` and `CLAUDE.md` pointing to the index. |

`--existing` is required when the project already has scripts: `init` then asks (or takes `--game-assembly`) which
assembly is the game.

## new

| Form | Creates |
|---|---|
| `new --genre tactics --name Orchard --players 4` | A Unity project at level 3: core with `ISimulation`, presentation, grafts, adapter, recipes, specs, tests, exemplars. |
| `new feature <Name>` | A `RuleModifier<T>`, its data row, a recipe and a completeness entry. |
| `new effect <Name>` | An enum case, its strategy, a cue, a completeness entry. |
| `new graft <Name> --after <Type.Method>` | A promoted graft with the right signature (drafts use `graft new`). |
| `new trial <Name>` | A recipe for the project's adapter, with checks. |

Genres: `tactics` (turn-based, grid), `action` (real time), `platformer`, `blank` (only the layers).

## uninstall

Removes `scion.json`, `ScionSettings.json`, the `.asmref`, the package and tool entries and the agent files. With
`--purge`, also `scion/` (drafts, recipes, presets, baselines, judge data) and `.scion/`.

## Options

| Option | Commands | Meaning |
|---|---|---|
| `--existing` | init | Adopt a project with scripts. |
| `--agent <name>` | init | Agent integration: `claude` (default), `codex`, `cursor`, `none`. |
| `--game-assembly <name>` | init | The game assembly, without asking. |
| `--grafts-folder <path>` | init | Default `Assets/<Game>/Grafts`. |
| `--genre`, `--name`, `--players` | new | Project template. |
| `--after`, `--before` | new graft | The target. |
| `--purge` | uninstall | Also delete Scion data. |
| `--dry-run` | all | Change nothing. |

## Output

<!-- validate: cli/init -->
```json
{"schema_version":1,"command":"init","ok":true,"dry_run":false,"created":["scion.json","ProjectSettings/ScionSettings.json","Assets/Orchard/Grafts/Orchard.Grafts.asmref"],"changed":["Packages/manifest.json",".gitignore",".gitattributes",".config/dotnet-tools.json"],"skipped":[],"adoption_level":1,"next_step":"dotnet scion doctor"}
```

## Exit codes

| Code | When |
|---|---|
| 0 | Done (or nothing to do). |
| 2 | Bad option, or `init` without `--existing` in a project with scripts. |
| 3 | A file could not be written. |

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `usage` | 2 | `init` in a project with scripts and no `--existing`. | Add `--existing`. |
| `config_invalid` | 3 | An existing `scion.json` is invalid. | Fix it; `init` never overwrites. |
| `bad_value` | 2 | Unknown genre or kind. | Use one of the listed values. |

## Examples

### Preview init in an existing project

```sh
dotnet scion init --existing --game-assembly Orchard.Runtime --dry-run
```

### A four-player tactics project

```sh
dotnet scion new --genre tactics --name Orchard --players 4
```

## See also

- [Start a new project](/scion/41664431a8f5/start/new-project/)
- [Add Scion to an existing game](/scion/41664431a8f5/start/existing-project/)
- [Enforcement](/scion/41664431a8f5/standard/enforcement/)

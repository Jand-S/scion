---
title: "graft"
description: "Create, check, build, inject, restore, disable, promote and watch grafts from the command line, without opening Unity."
type: cli
url: https://jand-s.github.io/scion/41664431a8f5/cli/graft/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/cli/graft.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/grafts/lifecycle.md
  - https://jand-s.github.io/scion/41664431a8f5/grafts/injection.md
  - https://jand-s.github.io/scion/41664431a8f5/agents/hooks.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# graft

The `graft` commands cover a graft's whole life outside Unity: `graft new` writes a draft with the right signature,
`graft check` runs Roslyn and the weaver rules against the test build, `graft build` compiles drafts into a DLL,
`graft inject`, `graft restore` and `graft disable` change running copies, `graft promote` moves a draft into the
game, and `graft watch` repeats build, inject and run on every save.

## Synopsis

```text
dotnet scion graft new <Name> --after|--before <Type.Method> [--dry-run]
dotnet scion graft check [files] [--strict] [--hook]
dotnet scion graft build [files]
dotnet scion graft inject [--to all|<pid>] [--isolate] [--dll <file>]
dotnet scion graft restore [--to all|<pid>]
dotnet scion graft disable <slot> [--to all|<pid>]
dotnet scion graft promote <Name> [--dry-run]
dotnet scion graft watch [--recipe <Id>]
```

## Description

Drafts live in `grafts.staging` (`scion/grafts/`); promoted grafts in `grafts.promoted`. All commands read the test
build for types and signatures; none starts Unity.

## graft new

Writes `scion/grafts/<Name>.cs`: namespace from the project, `using` lines, a `public static class <Name>` and a
method `Before` or `After` whose signature binds to the target, with `ref Call<T>` when it returns a value.

## graft check

Compiles the given files (default: every draft, and promoted grafts changed since the build) with the build's
assemblies, runs the analyzers and the weaver's rules (SCN101 to SCN113), and returns `diagnostics[]`. `--strict`
treats warnings as errors. `--hook` prints Claude Code hook JSON and only checks files under the grafts folders.

## graft build

Compiles drafts with the test defines against `Managed/*.dll`, generates the dispatchers the weaver would, and writes
`.scion/grafts/ScionGrafts_<hash>.dll` with the target `build_id` and ABI.

## graft inject

Sends the DLL (building first if drafts changed) to live copies of this build through the control channel. Default
merges by graft identity; `--isolate` runs only the DLL's grafts on the anchors it touches.

## graft restore

Every slot of the target copies back to the promoted grafts.

## graft disable

Empties one slot (promoted grafts included) in the target copies.

## graft promote

Moves `scion/grafts/<Name>.cs` to the promoted folder (`git mv` when tracked) and stages it. Rebuild to compile it
into the game.

## graft watch

On every save under the drafts folder: build, inject into all live copies, and, with `--recipe`, run it warm. Prints
one JSON line per cycle until Ctrl-C, then the summary.

## Options

| Option | Commands | Meaning |
|---|---|---|
| `--after`, `--before <Type.Method>` | new | The target and side. |
| `--strict` | check | Warnings fail. |
| `--hook` | check | Hook output for agents. |
| `--to all\|<pid>` | inject, restore, disable | Target copies (default all). |
| `--isolate` | inject | Replace instead of merge. |
| `--dll <file>` | inject | A DLL built elsewhere. |
| `--recipe <Id>` | watch | Run after each injection. |
| `--dry-run` | new, promote | Change nothing. |

## Output

<!-- validate: cli/graft-check -->
```json
{"schema_version":1,"command":"graft check","ok":false,"files":1,"diagnostics":[{"file":"scion/grafts/CritShake.cs","line":7,"code":"SCN103","message":"HitPresenter.Ply does not exist.","hint":"Did you mean 'Play'?","did_you_mean":["Play"]}],"errors":1,"warnings_count":0,"next_step":"fix scion/grafts/CritShake.cs:7, then: dotnet scion graft check scion/grafts/CritShake.cs"}
```

Each command's fields are in its schema: `cli/graft-new`, `cli/graft-check`, `cli/graft-build`, `cli/graft-inject`,
`cli/graft-restore`, `cli/graft-disable`, `cli/graft-promote`, `cli/graft-watch` ([JSON Schemas](/scion/41664431a8f5/reference/schemas/)).

## Exit codes

| Code | When |
|---|---|
| 0 | Done; `check` found no errors. |
| 1 | Diagnostics with errors; an injection with errors in some copy. |
| 2 | Bad option, unknown target or name. |
| 3 | No build, no live copies, a copy without `SCION_SLOTS`. |

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `SCN101`-`SCN113` | 1 | Graft rules. | [Graft diagnostics](/scion/41664431a8f5/reference/diagnostics/grafts/). |
| `SCN130`, `SCN131` | 1 | The DLL does not match the copy's build or ABI. | `graft build` again; same Scion version. |
| `no_copies` | 3 | No live copy. | `dotnet scion trial serve -j 2`. |
| `not_supported` | 3 | A copy built without `SCION_SLOTS`. | Use the test build. |
| `no_match` | 2 | Unknown target, draft or slot. | `did_you_mean`. |
| warning `recycle` | 0 | A copy passed 50 injections. | `trial serve` restarts it at the next boundary. |

## Examples

### The inner loop

```sh
dotnet scion graft new CritShake --after Orchard.Presentation.HitPresenter.Play
dotnet scion graft check scion/grafts/CritShake.cs
dotnet scion graft inject
dotnet scion trial run --warm SlashHit
```

### Strict check in CI

```sh
dotnet scion graft check --strict
```

## See also

- [From draft to final build](/scion/41664431a8f5/grafts/lifecycle/)
- [Live injection](/scion/41664431a8f5/grafts/injection/)
- [Hooks](/scion/41664431a8f5/agents/hooks/)

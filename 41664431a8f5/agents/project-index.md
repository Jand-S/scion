---
title: "Project index and manifest"
description: "The project index agents get at session start, the scion block in CLAUDE.md or AGENTS.md, and the generated manifest and GRAFTS.md."
type: reference
url: https://jand-s.github.io/scion/41664431a8f5/agents/project-index/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/agents/project-index.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/agents/hooks.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/index-and-guard.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/gen.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/manifest.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Project index and manifest

Agents learn a Scion project in three layers: the **index** (under 2 KB, always loaded), the **commands** they query on
demand (`anchor list`, `trial list`), and the **rules** the hooks enforce. This page covers the index that
`dotnet scion index` prints, the `<!-- scion:begin -->` block it keeps in `CLAUDE.md` or `AGENTS.md`, and the
manifest that `dotnet scion gen` generates from the code: `scion.manifest.json` for tools and `GRAFTS.md` for people.

## The index

`dotnet scion index` writes a short description of the project as facts, not instructions, so it never reads like a
prompt injection. It lists categories and counts, never every anchor:

```text
This project uses Scion 1.0.0 (adoption level 3).
Grafts: new presentation code in Assets/Orchard/Grafts/*.cs (promoted) and scion/grafts/*.cs (drafts), hung on anchors with [Before]/[After].
Anchor categories: presentation (14), hud (9), camera (3), audio (4).
Rules change in Orchard.Core with RuleModifier<T>; tunables: 7 visual, 3 rule.
Recipes: SlashHit, FireballArea, KnightFalls, CoopTurn (4 players), HudStates, UnitGallery. Specs: SlimeIsThreatening, DifficultyCurve.
Commands: dotnet scion anchor list|show, graft new|check|build|inject|promote, trial run|show|approve, doctor.
The manifest is queried through commands, not read whole.
```

Anchor categories come from the first segment of `[Anchor]` ids (`presentation.hit` is in `presentation`); anchors
without an id count under their type's namespace. With `--hook` the same text is wrapped as Claude Code
`SessionStart` context ([Hooks](/scion/41664431a8f5/agents/hooks/)).

## The block in CLAUDE.md or AGENTS.md

`dotnet scion index --write <file>` keeps the index plus the agent rules between two markers. Everything outside the
markers is yours and never touched:

```md
<!-- scion:begin -->
## Scion
This project uses Scion 1.0.0 (adoption level 3). Docs: https://jand-s.github.io/scion/41664431a8f5/llms.txt
...
<!-- scion:end -->
```

If the project keeps its instructions in `AGENTS.md`, `CLAUDE.md` should import it with the line `@AGENTS.md`, so
Claude Code and the other agents read the same block.

## The manifest

`dotnet scion gen` reads the code (attributes, recipes, tunables, probes) and writes two files at the project root:

| File | For | Content |
|---|---|---|
| `scion.manifest.json` | tools and agents, through commands | Every anchor with its id, target, signature and doc; every graft with file, line, kind, priority and target; every tunable with type, range, value and layer; every recipe; every probe. Schema: [`manifest.schema.json`](/scion/41664431a8f5/schemas/v1/manifest.schema.json) |
| `GRAFTS.md` | people and pull requests | A readable table of the same data, grouped by anchor |

Both are generated: nobody edits them by hand, and the baseline guard denies agent edits to them
([Hooks](/scion/41664431a8f5/agents/hooks/)). The source of truth is the code: `[Anchor(Id, Doc)]`, `[Before]`/`[After]`,
`[Tunable]`, `[Probe]` and the recipe classes.

`gen` reads source with Roslyn when the Unity project has not compiled, and the build's DLLs with Mono.Cecil when a
test build exists; `generated_from` in the manifest says which.

## Keep them current in CI

```sh
dotnet scion gen && git diff --exit-code scion.manifest.json GRAFTS.md
dotnet scion graft check --strict
```

The first line fails the job when someone changed an anchor or graft without regenerating. The second fails on
warnings too. See [Run Scion in CI](/scion/41664431a8f5/cookbook/ci/).

## Querying instead of reading

Agents never read the manifest whole. They ask:

| Question | Command |
|---|---|
| Where can I hang a hit effect? | `dotnet scion anchor list --search hit` |
| What is the exact signature? | `dotnet scion anchor show presentation.hit` |
| Which grafts run on this anchor, in what order? | `dotnet scion anchor show presentation.hit` (`grafts`) |
| Which recipes exist? | `dotnet scion trial list` |
| Which tunables can I change? | `dotnet scion gen --list tunables` |

## See also

- [index, guard](/scion/41664431a8f5/cli/index-and-guard/)
- [gen](/scion/41664431a8f5/cli/gen/)
- [scion.manifest.json and GRAFTS.md](/scion/41664431a8f5/reference/files/manifest/)
- [Hooks](/scion/41664431a8f5/agents/hooks/)

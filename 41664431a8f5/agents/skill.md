---
title: "The Scion skill"
description: "What the scion skill of the Claude Code plugin contains, when it loads, the scion-triage subagent and the scoped rules."
type: reference
url: https://jand-s.github.io/scion/41664431a8f5/agents/skill/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/agents/skill.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/agents/setup.md
  - https://jand-s.github.io/scion/41664431a8f5/agents/hooks.md
  - https://jand-s.github.io/scion/41664431a8f5/agents/reading-results.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# The Scion skill

The Claude Code plugin ships one skill, `scion`, one subagent, `scion-triage`, and path-scoped rules. This page lists
what each contains and when it loads, so you can predict what the agent knows. Other agents get the same content from
`AGENTS.md` and the MCP `docs_read` tool.

## The scion skill

| Field | Value |
|---|---|
| Name | `scion` |
| Loads when | The task mentions Scion, the test build, trials, recipes, baselines, grafts, anchors, visual regression, a state hash or frame count that changed, or checking a gameplay change in the real build |
| Files | `SKILL.md` (the loop and the rules, about 120 lines) and `reference/cli.md` (every command, option, output field and code) |
| Version | Same as the plugin; the plugin and the CLI must match the package version |

What `SKILL.md` covers, in order:

1. What Scion is in four lines (grafts, trials, baselines, the CLI).
2. Setup once per clone: `dotnet tool restore`, `dotnet scion doctor`.
3. The loop: edit, `graft check`, `build test` or `graft inject`, `trial run`, read the JSON, `trial show`, approve.
4. The fields of the `trial run` JSON and the verdicts.
5. The approval rules: agents approve only `new`; `changed`, `stale` and `px_changed` need a person.
6. The live loop with warm copies (`trial serve`, `graft inject`, `trial run --warm`).
7. Exit codes and the common error codes with their fix.
8. Reading order and the image budget.

The skill is facts and commands, not a long tutorial: every detail it does not hold is one `docs_read` call away.

## The scion-triage subagent

`scion-triage` reads images so the main agent does not have to. Use it when more than about 5 images need a look.

| Field | Value |
|---|---|
| Input | A run folder and a list of recipe ids (usually `changed[]` of a `trial run`) |
| Reads | `compare.json`, each recipe's `report.md`, then `window.png` or `<artifact>.grid.png` |
| Returns | One line per recipe: id, verdict, what changed in the frames, whether it looks intended, and the file it looked at |
| Never does | Approve, edit files or run commands other than `trial show` |

Example return:

```text
SlashHit      changed   white flash on the skeleton, frames 12-14; looks intended (HitFlash) diff/SlashHit/r1/window.grid.png
FireballArea  within    no visible change                                                     q1/FireballArea/r1/window.png
KnightFalls   changed   knight falls one frame later; hash same; check HitStop                diff/KnightFalls/r1/window.grid.png
```

## Scoped rules

The plugin installs rules in its own folder that load only when the agent reads or edits matching paths:

| Rule | Paths | Says |
|---|---|---|
| `grafts` | `**/Grafts/**`, `scion/grafts/**` | Grafts are visual; use `Fx`; run `graft check`; no rule state |
| `recipes` | `**/Trials/**`, `scion/recipes/**` | Recipes are data; set properties in the constructor; seed every RNG |
| `baselines` | `scion/baselines/**` | Never edit; only `trial approve` writes here |
| `core` | the folders of `rule_paths` | Integers only, no `float` (SCN004); changes go through `RuleModifier<T>` |

Projects add their own in `.claude/rules/*.md` with a `paths:` frontmatter ([Set up your agent](/scion/41664431a8f5/agents/setup/)).

## Keeping it current

- `dotnet scion doctor` warns `version_mismatch` when the plugin, the CLI and the package differ.
- `/plugin update scion` updates the plugin; the CLI and the package update as in [Install](/scion/41664431a8f5/start/install/).
- For agents without the plugin, `dotnet scion index --write AGENTS.md` refreshes the `AGENTS.md` block.

## See also

- [Set up your agent](/scion/41664431a8f5/agents/setup/)
- [Hooks](/scion/41664431a8f5/agents/hooks/)
- [Reading results](/scion/41664431a8f5/agents/reading-results/)
- [MCP server](/scion/41664431a8f5/agents/mcp/)

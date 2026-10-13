---
title: "For AI agents"
description: "Start page for AI coding agents - the Scion loop, the rules, what to read in which order and where the machine-readable files are."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/agents/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/agents.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/agents/setup.md
  - https://jand-s.github.io/scion/41664431a8f5/agents/next-action.md
  - https://jand-s.github.io/scion/41664431a8f5/agents/reading-results.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# For AI agents

This is the start page for AI coding agents (Claude Code, Codex, Cursor, GitHub Copilot and others) that work on a
game with Scion. It gives the loop (`graft check`, `graft inject`, `trial run`, `trial show`, `trial approve --new`),
the rules every agent must follow, the reading order that keeps token use low, and the machine-readable files:
`llms.txt`, the Markdown twin of every page, the JSON Schemas and the docs inside the installed package.

## When to use

- Read this page before your first Scion command in a project, and whenever a command's result surprises you.
- For one exit code or error code, go straight to [What to do next](/scion/41664431a8f5/agents/next-action/).

## Rules

These rules are also at the top of [/scion/llms.txt](/scion/41664431a8f5/llms.txt). Follow them in every session:

- Run `dotnet scion doctor` first. Exit 0 means usable; exit 3 means fix what `error.hint` says.
- Every command prints one JSON object on stdout (`schema_version`, `ok`, `next_step`, `error` on exits 2 and 3). Parse it; do not scrape text. Follow `next_step`.
- Exit codes: 0 ok, 1 a check failed, 2 usage error (see `did_you_mean`), 3 environment. Next actions: https://jand-s.github.io/scion/41664431a8f5/agents/next-action.md
- You may approve only `new` baselines (`trial approve --new --reason "..."`). Never approve `changed`, `stale` or `px_changed`, and never edit `scion/baselines/` by hand: ask a person.
- Read `report.md` only for failed or changed recipes; open images only when pixels changed or the task is visual.

Two more that follow from the design:

- Grafts are visual only. A change to damage, range, turn order or any other rule goes into the game's core as a
  `RuleModifier<T>`, never into a graft ([Grafts are visual only](/scion/41664431a8f5/grafts/visual-only/)).
- Never fake a terminal (`script`, `expect`, a pseudo-TTY) to get past `human_required`. Tell the person what changed
  and give them the command from `next_step`.

## Steps

### 1. Orient yourself

```sh
dotnet scion doctor
dotnet scion index
```

`index` prints the project index (under 2 KB): Scion version, adoption level, graft folders, anchor categories,
recipe names and the commands. With the Claude Code plugin it arrives on its own at session start.

### 2. Find where to hang the change

```sh
dotnet scion anchor list --search hit
dotnet scion anchor show presentation.hit
```

`anchor show` returns the signature and a ready graft declaration (`example`). Never guess a signature.

### 3. Write and check the graft

```sh
dotnet scion graft new CritShake --after Orchard.Presentation.HitPresenter.Play
dotnet scion graft check scion/grafts/CritShake.cs
```

Apply every `hint` and `did_you_mean` until `graft check` exits 0. With the plugin, the `PostToolUse` hook runs the
check after each edit and returns the errors to you.

### 4. Try it in the real build

```sh
dotnet scion trial serve -j 2
dotnet scion graft build && dotnet scion graft inject
dotnet scion trial run SlashHit KnightFalls --warm
```

Warm runs compare state hashes and frame counts. For images, run a cold run with the graft:
`dotnet scion trial run SlashHit --grafts <dll from graft build>`.

### 5. Read the results

Follow [Reading results](/scion/41664431a8f5/agents/reading-results/): the JSON, then `report.md` of failed or changed recipes,
then images only when pixels changed or the task is visual.

### 6. Hand over

- New recipes: `dotnet scion trial approve --new --reason "..."`, then commit the staged baselines.
- Changed recipes: stop and report. Give the person `trial show --changed` output, the reason and the exact approve
  command from `next_step`.
- Promote a graft only when the person agrees: `dotnet scion graft promote CritShake`.

## Reference

### Machine-readable files

| File | URL | Use |
|---|---|---|
| Page index | [/scion/llms.txt](/scion/41664431a8f5/llms.txt) | Every page as a link to its Markdown, with one line each; the rules first |
| Full docs | [/scion/llms-full.txt](/scion/41664431a8f5/llms-full.txt) | Every page in one file, in sidebar order |
| Section files | `/scion/<section>/llms-full.txt` | One section in one file, for example [/scion/grafts/llms-full.txt](/scion/41664431a8f5/grafts/llms-full.txt) |
| Markdown twin | `<page URL>.md` or `<page URL>/index.md` | The page as Markdown with frontmatter (`title`, `type`, `url`, `related`) |
| JSON Schemas | [JSON Schemas](/scion/41664431a8f5/reference/schemas/) | Every file format and every CLI output, draft 2020-12 |

### Version-matched local docs

The installed package carries the Markdown of its own version. Prefer it over the website when the project may not
run the latest version:

```text
Packages/me.jandson.scion/Documentation~/site/src/content/docs/                        embedded or local package
Library/PackageCache/me.jandson.scion@<hash>/Documentation~/site/src/content/docs/    Git URL or OpenUPM package
```

`dotnet scion mcp` serves the same files through its `docs_search` and `docs_read` tools.

### Token budget

| Read | Cost |
|---|---|
| A command's JSON | under 1.5 KB, always |
| `report.md` of one recipe | about 1,200 tokens |
| `window.png` (12 frames) | about 1,000 tokens |
| `context.png` (6 frames) | about 1,900 tokens |
| 120 captures opened blindly | about 1 million tokens: never do this |

With more than about 5 images to look at, hand them to the `scion-triage` subagent, which returns one line per
recipe ([The Scion skill](/scion/41664431a8f5/agents/skill/)).

## Errors

| Signal | Next action |
|---|---|
| Exit 1, `failed[]` | Read `failed[].report`, fix, run again. |
| Exit 1, `changed[]` | Explain the change with `trial show --changed`; ask a person to approve or fix it. |
| Exit 2 | Fix the call with `error.hint` and `did_you_mean`; never retry unchanged. |
| Exit 3 | Fix the environment with `error.hint`; `doctor` names the failing check. |

Every signal, with its exact next action: [What to do next](/scion/41664431a8f5/agents/next-action/).

## Examples

### A complete session

```sh
dotnet scion doctor
dotnet scion anchor list --search shake
dotnet scion graft new CritShake --after Orchard.Presentation.HitPresenter.Play
dotnet scion graft check scion/grafts/CritShake.cs
dotnet scion build test
dotnet scion trial run --grafts .scion/grafts/ScionGrafts_51c2.dll
dotnet scion trial show --changed
```

The run reports `SlashHit` as `changed` with `hash: same`: only pixels moved. The agent reports to the person:
"CritShake adds a 0.3 shake on critical hits; SlashHit changed in pixels only (window.png). To accept, promote the
graft, rebuild, run and approve at your terminal: `dotnet scion trial approve SlashHit --reason "crit shake"`."

## See also

- [Set up your agent](/scion/41664431a8f5/agents/setup/)
- [What to do next](/scion/41664431a8f5/agents/next-action/)
- [Reading results](/scion/41664431a8f5/agents/reading-results/)
- [MCP server](/scion/41664431a8f5/agents/mcp/)

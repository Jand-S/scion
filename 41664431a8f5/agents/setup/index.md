---
title: "Set up your agent"
description: "Connect Claude Code, Codex, Cursor, GitHub Copilot or any other AI agent to Scion with the plugin, MCP and an AGENTS.md block."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/agents/setup/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/agents/setup.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/agents/mcp.md
  - https://jand-s.github.io/scion/41664431a8f5/agents/skill.md
  - https://jand-s.github.io/scion/41664431a8f5/agents/hooks.md
  - https://jand-s.github.io/scion/41664431a8f5/agents/project-index.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Set up your agent

This guide connects an AI coding agent to Scion. Claude Code gets everything from the `scion` plugin (skill, hooks,
MCP server, `scion-triage` subagent). Codex, Cursor and GitHub Copilot get the `dotnet scion mcp` server in their MCP
configuration and the Scion block in `AGENTS.md`. Any other agent reads `/scion/llms.txt`, uses GitMCP or Context7, or
calls the CLI directly. `dotnet scion init --agent <name>` writes the files for you.

## When to use

- Use it once per project and agent, after [Install](/scion/start/install/).
- Agents that only run shell commands need nothing more than the CLI and the `AGENTS.md` block; MCP is optional.

## Before you begin

- `dotnet scion doctor` passes in the project.
- Run the commands below from the project root.

## Steps

### Claude Code

Install the plugin from the Scion marketplace:

```sh
/plugin marketplace add Jand-S/scion
/plugin install scion@scion
```

The plugin brings the [skill](/scion/agents/skill/), the four [hooks](/scion/agents/hooks/), the
[MCP server](/scion/agents/mcp/) and the `scion-triage` subagent. It does not edit your repository. Optionally, keep
the project index in `CLAUDE.md` too:

```sh
dotnet scion index --write CLAUDE.md
```

If the project uses `AGENTS.md`, make `CLAUDE.md` import it with a line `@AGENTS.md`, and write the block there
instead.

### Codex

```sh
dotnet scion init --agent codex
```

writes the Scion block into `AGENTS.md` and prints the MCP entry to add to `~/.codex/config.toml`:

```toml
[mcp_servers.scion]
command = "dotnet"
args = ["scion", "mcp"]
```

Codex reads `AGENTS.md` at the start of every task, and the MCP server gives it one tool per command.

### Cursor

```sh
dotnet scion init --agent cursor
```

writes `AGENTS.md` and `.cursor/mcp.json`:

```json
{"mcpServers":{"scion":{"command":"dotnet","args":["scion","mcp"]}}}
```

### GitHub Copilot

```sh
dotnet scion init --agent copilot
```

writes `AGENTS.md` and `.vscode/mcp.json` for agent mode in VS Code:

```json
{"servers":{"scion":{"type":"stdio","command":"dotnet","args":["scion","mcp"]}}}
```

### Any other agent

- Give it the shell: every command prints one JSON object, and the [rules](/scion/agents/) cover the rest.
- Point it at [/scion/llms.txt](/scion/llms.txt), or at the version-matched Markdown inside the package
  (`Library/PackageCache/me.jandson.scion@<hash>/Documentation~/site/src/content/docs/`).
- For agents that speak MCP but cannot run local tools, use GitMCP: add the remote server
  `https://jand-s.gitmcp.io/scion`, which reads `llms.txt` and serves the docs.
- Context7 indexes the Scion repository; ask for `me.jandson.scion` docs by name.

## Reference

### The AGENTS.md block

`init --agent` and `index --write AGENTS.md` keep this block between markers, regenerated on each run. Text outside
the markers is yours.

```md
<!-- scion:begin -->
## Scion

This project uses Scion 1.0.0 (adoption level 3). Docs: https://jand-s.github.io/scion/llms.txt
- Run `dotnet scion doctor` first. Every command prints one JSON object; follow `next_step`.
- New presentation code goes in grafts (`scion/grafts/*.cs`, `[Before]`/`[After]`); rules change in Orchard.Core with `RuleModifier<T>`.
- Find anchors with `dotnet scion anchor list --search <word>`; check grafts with `dotnet scion graft check <file>`.
- Run recipes with `dotnet scion trial run`; read `report.md` of failed or changed recipes only.
- Approve only `new` baselines (`dotnet scion trial approve --new --reason "..."`); never edit `scion/baselines/`.
<!-- scion:end -->
```

### Files per agent

| Agent | Instructions | MCP configuration | Written by |
|---|---|---|---|
| Claude Code | plugin skill; optional `CLAUDE.md` block | plugin | `/plugin install scion@scion` |
| Codex | `AGENTS.md` | `~/.codex/config.toml` (printed, not written) | `init --agent codex` |
| Cursor | `AGENTS.md` | `.cursor/mcp.json` | `init --agent cursor` |
| GitHub Copilot | `AGENTS.md` | `.vscode/mcp.json` | `init --agent copilot` |

### Scoped rules for Claude Code

The plugin also installs path-scoped rules that load only when the agent touches matching files. You can add your
own in `.claude/rules/` with a `paths:` frontmatter:

```md
---
paths:
  - "Assets/Orchard/Grafts/**"
  - "scion/grafts/**"
---
Grafts here are visual only. Use the Fx facade (Fx.Shake, Fx.Flash, Fx.Sound) instead of calling CameraRig or
AudioHub directly, so trials record every effect.
```

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `bad_value` | 2 | Unknown agent name in `--agent`. | Use `claude`, `codex`, `cursor` or `copilot`. |
| `out_not_writable` | 3 | `AGENTS.md` or the MCP file cannot be written. | Fix permissions; the command is idempotent. |
| `version_mismatch` (warning) | 0 | The plugin, the CLI and the package are not the same version. | Update the one `doctor` names. |

## Examples

### Two agents on one project

```sh
dotnet scion init --agent codex --agent cursor
git add AGENTS.md .cursor/mcp.json && git commit -m "Scion agent setup"
```

Both agents read the same `AGENTS.md` block; each has its own MCP file.

## See also

- [MCP server](/scion/agents/mcp/)
- [The Scion skill](/scion/agents/skill/)
- [Hooks](/scion/agents/hooks/)
- [Project index and manifest](/scion/agents/project-index/)

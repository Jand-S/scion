---
title: "MCP server"
description: "The dotnet scion mcp server - one tool per CLI command plus docs_search and docs_read - and GitMCP for the website."
type: reference
url: https://jand-s.github.io/scion/41664431a8f5/agents/mcp/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/agents/mcp.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/cli/mcp.md
  - https://jand-s.github.io/scion/41664431a8f5/agents/setup.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# MCP server

`dotnet scion mcp` is a Model Context Protocol server over stdio, built into the CLI. It exposes one tool per CLI
command (`scion_trial_run`, `scion_graft_check` and so on) that returns the same JSON envelope as the command, plus
`docs_search` and `docs_read` over the Markdown docs of the installed package version. For agents that cannot run
local processes, GitMCP serves the website docs at `https://jand-s.gitmcp.io/scion`.

## Configuration

The server runs from the project root, like every command, and needs nothing else. Per client:

**Claude Code** (the plugin registers it; by hand):

```sh
claude mcp add scion -- dotnet scion mcp
```

**Codex** (`~/.codex/config.toml`):

```toml
[mcp_servers.scion]
command = "dotnet"
args = ["scion", "mcp"]
```

**Cursor** (`.cursor/mcp.json`) and any client that uses the `mcpServers` shape:

```json
{"mcpServers":{"scion":{"command":"dotnet","args":["scion","mcp"]}}}
```

**VS Code with GitHub Copilot** (`.vscode/mcp.json`):

```json
{"servers":{"scion":{"type":"stdio","command":"dotnet","args":["scion","mcp"]}}}
```

## Tools

Every command tool takes the command's options as a JSON object (snake_case names, positional arguments as
`patterns` or `ids`) and returns the command's envelope as structured content and as text. The exit code is in
`exit_code`. Tool descriptions repeat the first line of the command's help.

| Tool | Command | Notes |
|---|---|---|
| `scion_doctor` | `doctor` | |
| `scion_build_test`, `scion_build_final` | `build test`, `build final` | Long: progress goes to MCP progress notifications |
| `scion_trial_list` | `trial list` | |
| `scion_trial_run` | `trial run` | Progress notifications per finished pair |
| `scion_trial_show` | `trial show` | |
| `scion_trial_approve` | `trial approve` | No terminal: only `new` recipes can be approved (`human_required` otherwise) |
| `scion_trial_serve` | `trial serve` | |
| `scion_trial_diff` | `trial diff` | |
| `scion_trial_eval` | `trial eval` | `export` and `import` modes for agent evaluators |
| `scion_anchor_list`, `scion_anchor_show` | `anchor list`, `anchor show` | |
| `scion_graft_new`, `scion_graft_check`, `scion_graft_build`, `scion_graft_inject`, `scion_graft_restore`, `scion_graft_disable`, `scion_graft_promote` | `graft ...` | |
| `scion_bisect` | `bisect` | Long |
| `scion_diagnose` | `diagnose` | |
| `scion_index`, `scion_gen` | `index`, `gen` | |
| `scion_judge_list`, `scion_judge_show`, `scion_judge_train`, `scion_judge_run` | `judge ...` | `judge vote` is for people and has no tool |
| `scion_multiverse_run`, `scion_multiverse_show` | `multiverse ...` | |
| `scion_hunt_run`, `scion_search_run` | `hunt run`, `search run` | Long |

Commands that need a person or never end have no tool: `trial loop`, `graft watch`, `judge vote`, `uninstall`.

Example call and result:

```json
{"name":"scion_trial_run","arguments":{"patterns":["SlashHit"],"rounds":2}}
```

```json
{"structuredContent":{"schema_version":1,"command":"trial run","ok":true,"exit_code":0,"run":"/Users/alex/orchard/.scion/runs/20261011-143200","copies":2,"runs":{"recipes":1,"rounds":2,"total":2,"ok":2,"failed":0,"changed":0,"within":0,"px_changed":0,"new":0,"stale":0},"failed":[],"changed":[],"determinism":{"hash_stable":true,"frame_jitter":[]},"next_step":"nothing changed"},"isError":false}
```

`isError` is true for exit codes 2 and 3; exit code 1 is a normal result with `ok: false`.

## Docs tools

| Tool | Arguments | Returns |
|---|---|---|
| `docs_search` | `query` (text), `limit` (default 8) | `[{title, path, url, snippet}]`, best first, from the docs of the installed package version |
| `docs_read` | `path` (a page id such as `grafts/anchors`, or a URL of this site) | The page as Markdown, with its frontmatter |

The docs come from `Documentation~/site/src/content/docs/` inside the package, so they always match the version the
project runs. `docs_read` with an anchor (`grafts/anchors#automatic-anchors`) returns only that section.

## GitMCP

GitMCP turns the public site into a remote MCP server with no setup: `https://jand-s.gitmcp.io/scion`. It reads
`/scion/llms.txt` first and serves pages on request. Use it when the agent runs in the cloud and cannot start
`dotnet scion mcp`; it documents the latest release, not necessarily the version installed in your project.

## Limits

- One server per project and agent session; it runs commands one at a time and queues the rest.
- Long commands report MCP progress; a client that cancels a call stops the command as Ctrl-C would.
- The server never calls an AI model and never approves anything a terminal rule protects.

## See also

- [mcp](/scion/41664431a8f5/cli/mcp/)
- [Set up your agent](/scion/41664431a8f5/agents/setup/)
- [For AI agents](/scion/41664431a8f5/agents/)

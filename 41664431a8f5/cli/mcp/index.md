---
title: "mcp"
description: "Run Scion as a Model Context Protocol server over stdio, exposing CLI commands as scion tools and the installed docs as docs_search and docs_read."
type: cli
url: https://jand-s.github.io/scion/41664431a8f5/cli/mcp/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/cli/mcp.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/agents/mcp.md
  - https://jand-s.github.io/scion/41664431a8f5/agents/setup.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# mcp

`dotnet scion mcp` starts a Model Context Protocol server on stdin and stdout. Agents that speak MCP call Scion's
commands as tools (`scion_<noun>_<verb>`, with the same arguments and the same JSON envelope as structured content)
and read the documentation of the installed version with `docs_search` and `docs_read`. The server is started by the
agent's client, not by you.

## Synopsis

```text
dotnet scion mcp [--root <path>] [--log <file>]
```

## Description

The server implements MCP over stdio: `initialize`, `tools/list`, `tools/call`, progress notifications and
cancellation. Every tool runs the corresponding command in-process from the project root and returns its envelope as
`structuredContent`, with `isError` true for exit codes 2 and 3. Commands that need a person or never end have no
tool. The full tool list is on [MCP server](/scion/agents/mcp/).

## Options

| Option | Default | Meaning |
|---|---|---|
| `--root <path>` | the nearest `scion.json` from the working directory | The project root. |
| `--log <file>` | none | Write a log of requests (never to stdout). |

## Output

The server writes only MCP messages to stdout. A `tools/call` result:

```json
{"structuredContent":{"schema_version":1,"command":"anchor list","ok":true,"total":26,"anchors":[],"next_step":"dotnet scion anchor show presentation.hit"},"isError":false}
```

## Exit codes

| Code | When |
|---|---|
| 0 | The client closed stdin. |
| 2 | Bad option. |
| 3 | No project root. |

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `config_not_found` | 3 | No `scion.json` from the working directory or `--root`. | Set `--root` in the client config. |
| `busy` | tool error | Another long command is running. | The server queues calls; wait for the first. |

## Examples

### Claude Code project config

```json
{"mcpServers":{"scion":{"command":"dotnet","args":["scion","mcp"]}}}
```

### Debug with a log

```sh
dotnet scion mcp --log /tmp/scion-mcp.log
```

## See also

- [MCP server](/scion/agents/mcp/)
- [Set up your agent](/scion/agents/setup/)

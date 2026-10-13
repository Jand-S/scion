---
title: "CLI overview"
description: "How the dotnet scion CLI is installed and called, how it finds the project, build and Unity, its global options, the JSON envelope and exit codes."
type: cli
url: https://jand-s.github.io/scion/41664431a8f5/cli/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/cli.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/start/install.md
  - https://jand-s.github.io/scion/41664431a8f5/agents/reading-results.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/exit-codes.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/errors.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# CLI overview

`dotnet scion` is Scion's command line, a local .NET tool (`Scion.Cli`). Every command has the form
`dotnet scion <noun> <verb>`, prints one JSON envelope outside a terminal (text inside one), always says the
`next_step`, and exits 0, 1, 2 or 3. This page covers installation, the global options (`--format`, `--dry-run`,
`--help`, `--version`), how the CLI finds the project root (`scion.json`), the test build (`--build`, `SCION_BUILD`),
Unity and the editor, the envelope and the exit-code precedence.

## Synopsis

```text
dotnet scion <command> [<verb>] [arguments] [options] [-- player-args]
dotnet scion --help | --version
```

| Command | Verbs | Page |
|---|---|---|
| `doctor` | | [doctor](/scion/41664431a8f5/cli/doctor/) |
| `init`, `new`, `uninstall` | | [init, new, uninstall](/scion/41664431a8f5/cli/init/) |
| `build` | `test`, `final` | [build test, build final](/scion/41664431a8f5/cli/build/) |
| `trial` (alias `trials`) | `list`, `run`, `show`, `approve`, `serve`, `diff`, `eval`, `loop` | [trial run](/scion/41664431a8f5/cli/trial-run/) and siblings |
| `anchor` | `list`, `show` | [anchor list, anchor show](/scion/41664431a8f5/cli/anchor/) |
| `graft` | `new`, `check`, `build`, `inject`, `restore`, `disable`, `promote`, `watch` | [graft](/scion/41664431a8f5/cli/graft/) |
| `bisect` | | [bisect](/scion/41664431a8f5/cli/bisect/) |
| `diagnose` | | [diagnose](/scion/41664431a8f5/cli/diagnose/) |
| `index`, `guard` | | [index, guard](/scion/41664431a8f5/cli/index-and-guard/) |
| `gen` | | [gen](/scion/41664431a8f5/cli/gen/) |
| `judge` | `list`, `show`, `new`, `vote`, `train`, `run` | [judge](/scion/41664431a8f5/cli/judge/) |
| `multiverse` | `run`, `show` | [multiverse](/scion/41664431a8f5/cli/multiverse/) |
| `hunt`, `search` | `run` | [hunt, search](/scion/41664431a8f5/cli/hunt-and-search/) |
| `mcp` | | [mcp](/scion/41664431a8f5/cli/mcp/) |

## Description

### Install

```sh
dotnet new tool-manifest            # once per repository, if there is no .config/dotnet-tools.json
dotnet tool install --local Scion.Cli --version 1.0.0
dotnet tool restore                  # on another machine
```

The CLI and the package must have the same version: a DLL injected into a build of another weaver ABI is refused
(SCN131), and `doctor` warns `version_mismatch`.

### How it finds things

| What | Order |
|---|---|
| Project root | The nearest folder upward with `scion.json`. |
| Configuration | `scion.json`, deep-merged with `scion.local.json` (local wins). |
| Test build | `--build <app\|exe\|folder>`, then `SCION_BUILD`, then `scion.local.json`, then `build.test.output.<os>` in `scion.json`. `scion-build.json` must sit next to it. |
| Unity | The version in `ProjectSettings/ProjectVersion.txt`, at the Hub's default path for the OS; `unity.editor` in `scion.local.json` overrides. |
| Editor open | `Temp/UnityLockfile` locked. Builds then go through the open editor (needs `com.unity.pipeline`) or stop with `editor_open`. |
| Ports | A free port per co-op group and per control channel (bind to 0, release). |

### Behavior every command shares

- Commands that write accept `--dry-run` and are idempotent: running one twice changes nothing the second time.
- Output is compact JSON (about 1.5 KB at most): long lists are cut with `"more": N`, and the full data goes to a
  file (`summary.json`) whose path is in the envelope.
- Arguments after `--` go to every player process; a flag also present in `player_args` replaces it.

## Options

| Option | Default | Meaning |
|---|---|---|
| `--format json\|text` | `json` outside a TTY, `text` in one | Output format. |
| `--dry-run` | off | Print what would change; change nothing. |
| `--build <path>` | discovery | The test build to use (commands that run players). |
| `--help`, `-h` | | Help for the command. |
| `--version` | | `1.0.0`. |

## Output

Every command prints one envelope:

<!-- validate: envelope -->
```json
{"schema_version":1,"command":"trial run","ok":false,"next_step":"run: dotnet scion build test","error":{"code":"build_not_found","message":"no test build at /Users/alex/orchard/.scion/builds/test/Orchard.app","hint":"run: dotnet scion build test","did_you_mean":[]}}
```

| Field | Meaning |
|---|---|
| `schema_version` | 1. |
| `command` | The command as typed, without options (`trial run`). |
| `ok` | True when the exit code is 0. |
| `next_step` | One line: what to do next. Always present. |
| `warnings[]` | `{code, message}`; never change the exit code. |
| `error` | Only on exit 2 and 3: `code`, `message`, `hint`, `did_you_mean`. |
| command fields | Documented on each command's page and in its JSON Schema. |

## Exit codes

| Code | Meaning |
|---|---|
| 0 | Everything ok (warnings do not count). |
| 1 | A recipe, check, graft check or spec failed. |
| 2 | Usage: unknown command, option or value; nothing matched. |
| 3 | Environment: no build, no Unity, a player crashed, a copy timed out. |

When several copies or steps end differently, the command exits with the highest-priority code: 2, then 3, then 1,
then 0. See [Exit codes](/scion/41664431a8f5/reference/exit-codes/).

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `config_not_found` | 2 | No `scion.json` above the current folder. | `dotnet scion init`, or `cd` into the project. |
| `config_invalid` | 3 | `scion.json` does not match its schema. | Fix the key in the message. |
| `unknown_command`, `unknown_option` | 2 | A typo. | Use `did_you_mean`. |
| `build_not_found` | 3 | No test build found. | `dotnet scion build test`. |
| `unity_not_found` | 3 | No Unity of the project's version. | Set `unity.editor` in `scion.local.json`. |

Every code: [Error and warning codes](/scion/41664431a8f5/reference/errors/).

## Examples

### Text output in a script

```sh
dotnet scion trial run --format text
```

### Pick a build for one command

```sh
SCION_BUILD=/tmp/orchard-test/Orchard.app dotnet scion trial list
```

## See also

- [Install](/scion/41664431a8f5/start/install/)
- [Reading results](/scion/41664431a8f5/agents/reading-results/)
- [Exit codes](/scion/41664431a8f5/reference/exit-codes/)
- [Error and warning codes](/scion/41664431a8f5/reference/errors/)
- [Configuration](/scion/41664431a8f5/reference/configuration/)

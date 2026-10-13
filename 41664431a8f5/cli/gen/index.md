---
title: "gen"
description: "Generate scion.manifest.json and GRAFTS.md from the code - anchors, grafts, tunables, recipes and probes - or list one kind of item."
type: cli
url: https://jand-s.github.io/scion/41664431a8f5/cli/gen/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/cli/gen.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/agents/project-index.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/manifest.md
  - https://jand-s.github.io/scion/41664431a8f5/grafts/tunables.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# gen

`dotnet scion gen` reads the project's code and writes two generated files at the project root:
`scion.manifest.json` (every anchor, graft, tunable, recipe and probe, for tools and agents) and `GRAFTS.md` (the
same data as tables, for people and pull requests). `--list` prints one kind of item without writing. Run it after
changing attributes; CI checks the files are current.

## Synopsis

```text
dotnet scion gen [--check] [--dry-run]
dotnet scion gen --list anchors|grafts|tunables|recipes|probes [--search <text>]
```

## Description

Sources, in order: the test build's DLLs (Mono.Cecil) when they are newer than the sources; otherwise the C# sources
(Roslyn). Output is sorted and stable, so the same code always gives the same bytes.

## Options

| Option | Default | Meaning |
|---|---|---|
| `--check` | off | Write nothing; exit 1 if the files are not current. |
| `--list <kind>` | | Print items of one kind. |
| `--search <text>` | | Filter `--list`. |
| `--dry-run` | off | Print what would change. |

## Output

<!-- validate: cli/gen -->
```json
{"schema_version":1,"command":"gen","ok":true,"manifest":"scion.manifest.json","grafts_md":"GRAFTS.md","anchors":26,"grafts":4,"tunables":9,"changed":true,"next_step":"git add scion.manifest.json GRAFTS.md"}
```

With `--list tunables`, `items[]` holds `key`, `type`, `min`, `max`, `value`, `layer` (`visual` or `rule`) and `doc`.

## Exit codes

| Code | When |
|---|---|
| 0 | Written, listed, or current. |
| 1 | `--check` and the files are out of date. |
| 2 | Unknown `--list` kind. |

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `bad_value` | 2 | Unknown kind. | `anchors`, `grafts`, `tunables`, `recipes`, `probes`. |
| warning `name_at_run_time` | 0 | A target named by a computed string. | Use `nameof` or a literal. |

## Examples

### Check in CI

```sh
dotnet scion gen --check
```

### Every rule tunable

```sh
dotnet scion gen --list tunables --search Balance
```

## See also

- [Project index and manifest](/scion/agents/project-index/)
- [scion.manifest.json and GRAFTS.md](/scion/reference/files/manifest/)
- [Tunables and presets](/scion/grafts/tunables/)

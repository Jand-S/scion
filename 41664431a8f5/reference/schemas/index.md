---
title: "JSON Schemas"
description: "The JSON Schema of every Scion file and CLI output, with their URLs, and how to validate files with ajv, check-jsonschema or an editor."
type: reference
url: https://jand-s.github.io/scion/41664431a8f5/reference/schemas/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/schemas.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/reference/configuration.md
  - https://jand-s.github.io/scion/41664431a8f5/cli.md
  - https://jand-s.github.io/scion/41664431a8f5/agents/reading-results.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# JSON Schemas

Every JSON file Scion reads or writes, and the output of every CLI command, has a JSON Schema (draft 2020-12)
published under `https://jand-s.github.io/scion/41664431a8f5/schemas/v1/`. Use them to validate configuration in your editor, to
check files in CI, and to generate types for tools. The `v1` in the path matches `schema_version: 1`.

## Validate

In `scion.json`, point editors at the schema with `$schema` (the CLI ignores the key):

```json
{"$schema":"https://jand-s.github.io/scion/41664431a8f5/schemas/v1/scion-config.schema.json","schema_version":1}
```

From the command line:

```sh
npx ajv-cli validate --spec=draft2020 -s https://jand-s.github.io/scion/41664431a8f5/schemas/v1/result.schema.json -d ".scion/runs/*/q*/*/r*/result.json"
check-jsonschema --schemafile https://jand-s.github.io/scion/41664431a8f5/schemas/v1/scion-config.schema.json scion.json
```

CLI outputs reference `envelope.schema.json` and add their fields.

## Files

| Schema | Validates |
|---|---|
| `envelope.schema.json` | The envelope every CLI command prints. |
| `run.schema.json` | `run.json` |
| `result.schema.json` | `result.json` |
| `recipes.schema.json` | `recipes.json` |
| `summary.schema.json` | `summary.json` |
| `compare.schema.json` | `compare.json` |
| `baseline.schema.json` | `baseline.json` |
| `platform.schema.json` | `platform.json` |
| `scion-build.schema.json` | `scion-build.json` |
| `scion-config.schema.json` | `scion.json` and `scion.local.json` |
| `scion-settings.schema.json` | `ProjectSettings/ScionSettings.json` |
| `control.schema.json` | `control.json` and live files |
| `control-request.schema.json` | Control protocol requests |
| `control-response.schema.json` | Control protocol responses |
| `inject-report.schema.json` | `InjectReport.ToJson()` |
| `raw-frames.schema.json` | `frames/raw.json` |
| `telemetry-line.schema.json` | Each line of `telemetry.jsonl` |
| `judge-question.schema.json` | Question files and `eval-in.json` |
| `judge-answer.schema.json` | `eval-out.json` and judge answers |
| `manifest.schema.json` | `scion.manifest.json` |
| `multiverse.schema.json` | `multiverse.json` |
| `match.schema.json` | `*.match.json` captured and found matches |
| `preset.schema.json` | `scion/presets/*.json` |
| `metrics.schema.json` | `metrics.json` of the motion pack |
| `search.schema.json` | `search.json` of guided search |
| `head.schema.json` | `scion/judge/heads/*.json` |
| `vote.schema.json` | Each line of `scion/judge/votes/*.jsonl` and gold sets |

## CLI outputs

Under `https://jand-s.github.io/scion/41664431a8f5/schemas/v1/cli/`:

| Schema | Command |
|---|---|
| `cli/doctor.schema.json` | `doctor` |
| `cli/init.schema.json` | `init`, `new`, `uninstall` |
| `cli/build.schema.json` | `build test`, `build final` |
| `cli/trial-list.schema.json` | `trial list` |
| `cli/trial-run.schema.json` | `trial run` |
| `cli/trial-show.schema.json` | `trial show` |
| `cli/trial-approve.schema.json` | `trial approve` |
| `cli/trial-serve.schema.json` | `trial serve` |
| `cli/trial-diff.schema.json` | `trial diff` |
| `cli/trial-eval.schema.json` | `trial eval` |
| `cli/trial-loop.schema.json` | `trial loop` |
| `cli/anchor-list.schema.json` | `anchor list` |
| `cli/anchor-show.schema.json` | `anchor show` |
| `cli/graft-new.schema.json` | `graft new` |
| `cli/graft-check.schema.json` | `graft check` |
| `cli/graft-build.schema.json` | `graft build` |
| `cli/graft-inject.schema.json` | `graft inject` |
| `cli/graft-restore.schema.json` | `graft restore` |
| `cli/graft-disable.schema.json` | `graft disable` |
| `cli/graft-promote.schema.json` | `graft promote` |
| `cli/graft-watch.schema.json` | `graft watch` |
| `cli/bisect.schema.json` | `bisect` |
| `cli/diagnose.schema.json` | `diagnose` |
| `cli/index.schema.json` | `index` |
| `cli/guard.schema.json` | `guard` |
| `cli/gen.schema.json` | `gen` |
| `cli/judge.schema.json` | `judge list`, `show`, `new`, `vote`, `train`, `run` |
| `cli/multiverse.schema.json` | `multiverse run`, `multiverse show` |
| `cli/hunt-run.schema.json` | `hunt run` |
| `cli/search-run.schema.json` | `search run` |

## Versioning

A schema URL never changes meaning. New optional fields are added to the same `v1` schema; a breaking change gets
`schema_version: 2` and a `v2/` folder, with `v1/` kept.

## See also

- [Configuration](/scion/41664431a8f5/reference/configuration/)
- [CLI overview](/scion/41664431a8f5/cli/)
- [Reading results](/scion/41664431a8f5/agents/reading-results/)
- [Versioning and compatibility](/scion/41664431a8f5/contribute/compatibility/)

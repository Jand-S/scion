---
title: "InjectReport JSON"
description: "The JSON form of an injection result - version, ok, pending, bound slots, errors and the recycle flag - returned per copy by graft inject."
type: file
url: https://jand-s.github.io/scion/41664431a8f5/reference/files/inject-report/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/files/inject-report.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/grafts/injection.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/injection.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/control-protocol.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# InjectReport JSON

Every injection produces an `InjectReport`: the load counter after it, whether it succeeded, whether it waits for the
next recipe boundary, which slots it bound, which errors it hit and whether the copy should be recycled.
`InjectReport.ToJson()` gives this shape; the control protocol's `inject` op answers with it, and `graft inject`
lists one per copy.

## Location

Not a file on disk by default: it travels in control responses and in the `copies[]` of `graft inject`. The `inject`
line of `telemetry.jsonl` repeats `version` and `bound`.

## Written by and read by

Produced by `Injection.Inject` in the copy (`SCION_SLOTS`). Read by the CLI, agents and your own tools through
`Injection.Applied`.

## Schema

`https://jand-s.github.io/scion/41664431a8f5/schemas/v1/inject-report.schema.json`

## Fields

| Field | Meaning |
|---|---|
| `version` | `Injection.Version` after this load. |
| `ok` | No errors. |
| `pending` | A recipe is running; the swap applies at the next boundary. |
| `bound` | Slot ids that now run injected grafts. |
| `errors` | `"SCNxxx: message"` lines (SCN113, SCN130, SCN131). |
| `recycle` | True after 50 loads: restart the copy (Mono keeps every loaded assembly). |

## Example

<!-- validate: inject-report -->
```json
{"schema_version":1,"version":3,"ok":true,"pending":false,"bound":["A__Orchard_Presentation_HitPresenter__Play__HitEvent"],"errors":[],"recycle":false}
```

## Compatibility

Fields may be added in minor versions.

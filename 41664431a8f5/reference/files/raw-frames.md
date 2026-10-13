---
title: "Raw frames"
description: "The raw frame files kept with -scion-raw - the exact bytes each frame hash was computed from, and raw.json with size, layout and row order."
type: file
url: https://jand-s.github.io/scion/41664431a8f5/reference/files/raw-frames/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/files/raw-frames.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/capture.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/compare.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Raw frames

With `-scion-raw` (`--raw`), the player keeps every captured frame of the cut as raw bytes: exactly the bytes the
frame's `px` hash was computed from, in the format the GPU readback produced, without conversion. `raw.json` beside
them says how to read them. `Scion.Compare` loads them directly.

## Location

`<scion-out>/<IdDir>/r<n>/frames/NNNN.raw` and `frames/raw.json`.

## Written by and read by

Written by the test player. Read by `Scion.Compare` (`Image.Load`), heatmaps and your own tools.

## Schema

`https://jand-s.github.io/scion/41664431a8f5/schemas/v1/raw-frames.schema.json` for `raw.json`. Each `.raw` file has no header:
`width x height x 4` bytes.

## Fields

| Field | Meaning |
|---|---|
| `width`, `height` | Frame size in pixels. |
| `layout` | `RGBA8`, `ARGB8` or `BGRA8`: the byte order captured. |
| `rows` | Always `bottom-up`: the first row in the file is the bottom of the image. |

Alpha is always 255.

## Example

<!-- validate: raw-frames -->
```json
{"schema_version":1,"width":1280,"height":720,"layout":"BGRA8","rows":"bottom-up"}
```

```csharp
var img = Scion.Compare.Image.Load(".scion/runs/20261011-143200/q0/SlashHit/r1/frames/0014.raw");
img.SavePng("/tmp/frame14.png");
```

## Compatibility

The layout list may grow in minor versions; `rows` stays `bottom-up`.

---
title: "Capture and outputs"
description: "What a recipe captures - window, context, stills, sheets, frames and raw frames - in headless offscreen mode, with sizes, platforms and sound."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/trials/capture/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/trials/capture.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/run-folder.md
  - https://jand-s.github.io/scion/41664431a8f5/evaluation/motion-pack.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/raw-frames.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Capture and outputs

Every recipe writes a folder of outputs: a verdict (`result.json`), a human report (`report.md`), per-frame data
(`frames.tsv`), telemetry, the log, and images built from the frames of the cut. This guide covers headless and
offscreen capture, the contact sheets `window.png` and `context.png`, stills, `sheet.png`, the options `--frames`,
`--sound`, `--size` and `-scion-raw`, the 2000 px rule and the platform id that names baselines.

## When to use

- Read `window.png` first: it shows the moment that matters, frame by frame.
- Use `--frames` only to make a video or inspect one frame; it writes every frame as a PNG and is slow.
- Use `-scion-raw` when a tool needs the exact bytes that were hashed.

## Steps

### 1. Run headless (the default)

```sh
dotnet scion trial run SlashHit
```

Copies start with `-batchmode` and `-scion-offscreen`: the camera renders into textures at the capture size, with
no window. `Screen.width` and `Screen.height` read the capture size through `VirtualScreen`.

### 2. Find the images

```sh
dotnet scion trial show SlashHit
```

The output lists the paths of `window.png`, `context.png`, the stills and `report.md`.

### 3. Capture more when needed

```sh
dotnet scion trial run SlashHit --frames --size 1920x1080 --sound
```

## Reference

### Images per recipe

| File | Content | Size rule |
|---|---|---|
| `window.png` | Up to 12 consecutive frames centered on the first `MarkImpact` (or the cut's middle), 3 columns. | Longer side at most 2000 px |
| `context.png` | 6 frames spread over the whole cut (start, marks, end), 3x2. | Same |
| `<still>.png` | One frame per `Still(name)`, full size. | Capture size |
| `sheet.png` | Every item of a static trial, labeled. | Longer side at most 2000 px |
| `frames/NNNN.png` | Every frame of the cut (`--frames`). | Capture size |
| `frames/NNNN.raw` + `frames/raw.json` | The exact bytes hashed per frame (`-scion-raw`). | Capture size |
| motion pack | `motion-diff.png`, `tracks.png`, `idmask/`, `metrics.json`, `preview.gif` (`--motion`) | See [The motion pack](/scion/41664431a8f5/evaluation/motion-pack/) |

Sheets downscale by the smallest integer that keeps the longer side at or under 2000 px: 12 frames of 1280x720 in
3 columns (3840x2880) downscale by 2. Labels name the frame number and its marks.

### Platform id

Images depend on the platform, so baselines keep one folder per platform:

```text
<os>-<gfx>-<W>x<H>-<capture>        osx-metal-1280x720-offscreen
```

`os` is `osx`, `win` or `linux`; `gfx` is the graphics API in lowercase (`metal`, `direct3d11`, `vulkan`); capture
is `offscreen` or `screen` (`--window`), because UI composition differs.

### Options

| CLI | Player flag | Default | Effect |
|---|---|---|---|
| `--window` | (no `-batchmode`) | headless | Render to a window; the platform id ends in `screen`. |
| `--size WxH` | `-screen-width`, `-screen-height` | `trials.defaults.size` | Capture size. |
| `--frames` | `-scion-frames` | off | Every frame as PNG. |
| `--sound` | `-scion-sound` | muted | Keep audio on. |
| `--raw` | `-scion-raw` | off | Keep raw frames. |
| `--motion` | `-scion-motion` | off | Write the motion pack. |

### Coordinates

In the C# API (`MaskRect`, `CheckResult.Region`, `FrameView`) the origin is at the bottom left, like `Screen`. In
JSON, PNG and `Scion.Compare`, the origin is at the top left. The player converts when writing.

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `capture_broken` | 3 | Captures are empty, usually `-batchmode` without offscreen capture. | Let the CLI start copies, or add `-scion-offscreen`. |
| recipe error `PNG encode failed` | 1 | Writing an image failed. | Check free disk space in `trials.out`. |
| `out_not_writable` | 3 | The output folder cannot be written. | Fix permissions or `trials.out`. |

## Examples

### A 4K still for a store page

```sh
dotnet scion trial run UnitGallery --size 3840x2160
```

### Frames for a GIF

```sh
dotnet scion trial run SlashHit --frames
ffmpeg -framerate 30 -i .scion/runs/20261011-143200/q0/SlashHit/r1/frames/%04d.png slash.gif
```

## See also

- [The run folder](/scion/41664431a8f5/reference/files/run-folder/)
- [The motion pack](/scion/41664431a8f5/evaluation/motion-pack/)
- [Raw frames](/scion/41664431a8f5/reference/files/raw-frames/)
- [Baselines and approval](/scion/41664431a8f5/trials/baselines/)

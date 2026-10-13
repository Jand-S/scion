---
title: "Models and licenses"
description: "The vision models the judge supports as eyes, the hardware they need, where embeddings and heads live, and the licenses to check."
type: reference
url: https://jand-s.github.io/scion/41664431a8f5/judge/models/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/judge/models.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/judge/overview.md
  - https://jand-s.github.io/scion/41664431a8f5/help/license.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/configuration.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Models and licenses

The judge's eyes are pre-trained vision encoders that run locally. Scion does not ship model weights: `dotnet scion
doctor` checks the configured model and prints the command that downloads it into the local model cache. This page
lists the supported models, their cost, where the judge keeps its data and what to check in each license.

## Supported eyes

| `judge.eyes` | Model | Good at | Embedding | Time per image (Apple M5) |
|---|---|---|---|---|
| `siglip2-so400m` (default) | SigLIP 2 So400m, patch 14, 384 px | General images, UI with text | 1152 | about 25 ms |
| `siglip2-base` | SigLIP 2 Base, patch 16, 256 px | Fast first passes | 768 | about 8 ms |
| `dinov2-base` | DINOv2 Base, patch 14 | Shapes, layout, art style without text | 768 | about 15 ms |
| `uiclip` | UIClip | Web and app UI conventions | 512 | about 10 ms |

Switching eyes recomputes embeddings (10,000 images take about 5 to 10 minutes) and retrains heads (seconds each).

## Hardware

The judge runs the eyes with ONNX Runtime, on the GPU when one is available:

| Machine | Runs on |
|---|---|
| macOS on Apple silicon | GPU and Neural Engine through CoreML. |
| Windows with a DirectX 12 GPU | GPU through DirectML. |
| Linux with an NVIDIA GPU | GPU through CUDA. |
| Any machine without a supported GPU | CPU, about 5 to 10 times slower. |

## Where the data lives

| Data | Path | In git |
|---|---|---|
| Questions, heads, votes, gold sets | `judge.dir` (`scion/judge/`) | Yes (small JSON and JSONL) |
| Embeddings cache | `judge.cache` (`.scion/judge/`) | No |
| Model weights | the local model cache (`~/.cache/scion/models/`) | No |

Nothing leaves the machine: the judge makes no network calls after the model is downloaded.

## Licenses

- Check the model card of the eyes you use; SigLIP 2 and DINOv2 are published under Apache-2.0 at the time of
  writing, UIClip under its own terms.
- Heads trained on your votes are yours. Heads trained on third-party images (store screenshots, glitch datasets)
  are for internal use: do not ship their weights.
- If you use an agent to label pairs, follow the model provider's terms on using outputs for training.

## See also

- [The Judge](/scion/41664431a8f5/judge/overview/)
- [Judge data files](/scion/41664431a8f5/reference/files/judge-data/)
- [License](/scion/41664431a8f5/help/license/)

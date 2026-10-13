---
title: "The Judge"
description: "A fast trainable judge - frozen vision eyes turn captures into embeddings once, small heads trained in seconds answer one question each, and it only filters."
type: concept
url: https://jand-s.github.io/scion/41664431a8f5/judge/overview/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/judge/overview.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/judge/questions.md
  - https://jand-s.github.io/scion/41664431a8f5/judge/training.md
  - https://jand-s.github.io/scion/41664431a8f5/judge/models.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/judge.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# The Judge

The Scion Judge answers taste questions about captures in milliseconds: is this HUD readable, is there a visual
glitch, does this hit feel heavy, does this screen look like our game. It has two parts: frozen **eyes** (a
pre-trained vision encoder) that turn each image into an embedding once, and small **heads** trained on your own
votes in seconds, one per question. The judge filters and orders; it never approves anything.

## How it works

```text
capture -> eyes (frozen, e.g. SigLIP 2) -> embedding, computed once and cached
                                              |-> head "has_glitch"   (noul)
                                              |-> head "readability"  (score)
                                              |-> head "main_issue"   (choice)
                                              '-> head "hit_weight"   (score)
```

### Eyes once, heads many

Computing an embedding is the expensive part, and it does not depend on the question. Scion computes it once per
image (and per view: full frame, HUD crop, corners, actor crop) and caches it under `judge.cache`. A new question is
a new head: a small classifier over the cached embeddings, trained in seconds. The cost of a question is in its
labels, not in its training.

### Why not ask a large model every time

A general vision-language model can review a screen, but it re-reads the whole context for every image, takes
seconds per item and does not know your taste. The judge answers in 10 to 50 ms per image, 100 captures in seconds,
a 30-frame sequence in under a second, on a laptop, and its answers are calibrated against your own gold set.
Large models are still useful: as agent evaluators on the few items the judge escalates.

### Where it sits

The judge is stage 3 of the [evaluation funnel](/scion/evaluation/overview/): after the state oracle and the gates,
before a person. It sorts variants, picks which pairs a person should see, and escalates items it is unsure about.

### Pixel art

Downscaling erases one-pixel detail. For pixel art, views crop instead of scaling, or upscale 2x with nearest
neighbor, before the eyes see the image.

## Key terms

| Term | Meaning |
|---|---|
| Eyes | The frozen vision encoder (`judge.eyes` in `scion.json`). |
| Embedding | The vector the eyes produce for one image and view; cached. |
| Head | A small model trained on embeddings and votes; answers one question. |
| View | Which part of the image the eyes see: `full`, `hud`, `corners`, `actor`. |
| Vote | One person's (or agent's) answer: a duel winner, a score or a choice. |
| Gold set | Votes by the project's people, never used for training, used to measure a head. |
| Escalate | A question whose answer had low confidence; sent to a person or an agent evaluator. |

## Guarantees and limits

- The judge never changes an exit code or approves a baseline.
- A head predicts the taste of the people who voted; with few votes it is noisy, and `judge show` says so.
- Confidence is not accuracy: accuracy is measured on the gold set and reported as `calibration.agreement`.
- Studio data (captures, votes, heads) stays on your machines; nothing is sent anywhere.
- Model licenses vary by model card; check them before shipping heads ([Models and licenses](/scion/judge/models/)).

## See also

- [Questions and answers](/scion/judge/questions/)
- [Train a head](/scion/judge/training/)
- [Review a UI with the judge](/scion/judge/ui-review/)
- [judge (CLI)](/scion/cli/judge/)

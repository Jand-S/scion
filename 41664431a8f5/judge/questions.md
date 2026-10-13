---
title: "Questions and answers"
description: "The judge's question types noul, score and choice, the answer fields, and Scion's additions - evidence, escalate, head version and calibration."
type: reference
url: https://jand-s.github.io/scion/41664431a8f5/judge/questions/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/judge/questions.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/judge/overview.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/evals.md
  - https://jand-s.github.io/scion/41664431a8f5/evaluation/evaluators.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Questions and answers

Every head answers one question, and every evaluator receives and returns the same shapes. A question has a type
(`noul`, `score` or `choice`), instructions and criteria; an answer has the value, the probabilities, a confidence
and a short legend, plus Scion's fields: evidence (frame and region), whether to escalate, the head version and its
calibration against the gold set.

## Question types

| Type | Asks | Answer value |
|---|---|---|
| `noul` | Yes or no (`has_glitch`). | `noul`: probability of yes, 0 to 1. |
| `score` | A level from 2 to 10 levels, each with what it means and examples. | `score`: the probability-weighted mean of the levels. |
| `choice` | One of up to 255 options, each with a description (`main_issue`). | `choice`: the option id. |

## Question file

`scion/judge/questions/readability.json`, the input of `judge new`:

<!-- validate: judge-question -->
```json
{"schema_version":1,"run":null,"blind":true,"orders":2,"questions":{"readability":{"type":"score","instructions":"How easy is the HUD to read at 1x on a laptop screen?","criteria":["text size","contrast","hierarchy"],"levels":[{"what":"Hard to read","examples":["scion/judge/examples/read-0.png"]},{"what":"Readable with effort","examples":[]},{"what":"Effortless","examples":["scion/judge/examples/read-2.png"]}]}},"items":[]}
```

The same shape, with `items` filled, is `eval-in.json` for evaluators.

## Answer

<!-- validate: judge-answer -->
```json
{"schema_version":1,"model":"siglip2-so400m","head_version":"readability@3","items":{"X":{"answers":{"readability":{"type":"score","score":1.2,"confidence":0.41,"probabilities":{"0":0.12,"1":0.56,"2":0.32},"legend":"small turn counter","evidence":{"frame":3,"region":[1100,20,160,40]}},"has_glitch":{"type":"noul","noul":0.04,"confidence":0.92}},"escalate":["readability"]}},"preferences":[],"calibration":{"gold_set":"ui_v1","agreement":0.78}}
```

## Answer fields

| Field | Meaning |
|---|---|
| `model` | The eyes (or the evaluator's model). |
| `head_version` | `<head>@<version>`. |
| `items.<label>.answers.<question>` | One answer per question. |
| `type` | The question type. |
| `noul`, `score`, `choice` | The value, by type. |
| `probabilities` | Per level or option. |
| `confidence` | 0 to 1: how concentrated the probabilities are. Not accuracy. |
| `legend` | A few words on why (from the nearest labeled examples). |
| `evidence` | `frame` and `region` (pixels, top-left origin) the answer is about. |
| `escalate` | Questions whose confidence fell under the head's threshold. |
| `preferences` | For duels: `pair`, `winner`, `strength` (1 slight, 2 clear). |
| `calibration` | `gold_set` and `agreement` with the people's gold votes. |

## Writing good questions

- Ask something concrete. "Is it beautiful?" is too vague to learn; "Does the palette stay cohesive?", "Does it look
  like these five approved screens?", "Is the selected state obvious?" work.
- Give every score level a description and examples.
- Prefer duels for feel ("which hit is heavier?"): people agree more on comparisons than on absolute scores.

## See also

- [The Judge](/scion/41664431a8f5/judge/overview/)
- [eval-in.json, eval-out.json, evals.tsv](/scion/41664431a8f5/reference/files/evals/)
- [Judge data files](/scion/41664431a8f5/reference/files/judge-data/)
- [Evaluators](/scion/41664431a8f5/evaluation/evaluators/)

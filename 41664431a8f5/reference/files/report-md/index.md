---
title: "report.md"
description: "The human and agent readable report of a process and of each recipe - results, failed checks, images, timeline, notes and log errors."
type: file
url: https://jand-s.github.io/scion/41664431a8f5/reference/files/report-md/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/files/report-md.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/agents/reading-results.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/result-json.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# report.md

`report.md` is the first file to read when a recipe fails. The process-level report indexes every recipe, failures
first; the recipe-level report says what happened, which checks did not pass, which images exist with their frames,
the timeline of marks, the notes and the log errors. It is short, in English and links only relative paths.

## Location

| Level | Path |
|---|---|
| Process | `<scion-out>/report.md` |
| Recipe round | `<scion-out>/<IdDir>/r<n>/report.md` |

## Written by and read by

Written by the test player. Read by agents (the `next_step` of a failed run points to it) and people.

## Schema

Markdown. The first line is always `<!-- scion report v1 -->`; the version changes if the section order changes.

## Fields

Recipe report sections, in order:

| Section | Content |
|---|---|
| Title | `# <Id> r<n>: passed` or `failed: <error>` |
| Checks | Checks that did not pass, with reason, frame and region. |
| Images | Each image with its frames and marks. |
| Timeline | Marks by frame and game time. |
| Notes | `TrialSession.Note` lines. |
| Log | Errors and exceptions from `log.txt`, first lines of each stack. |
| Motion | With `--motion`: the metrics table. |

## Example

```md
<!-- scion report v1 -->
# KnightFalls r1: failed: timeout (30s): unit down animation

## Checks
- order: fail: unit_down was never marked

## Images
- window.png: frames 20-31, marks IMPACT
- before.png: still

## Timeline
- frame 0 (0.000 s): cut begins
- frame 24 (0.800 s): IMPACT

## Log
- (none)
```

## Compatibility

Section titles and order are stable within `scion report v1`; new sections may be added at the end.

---
title: "Exit codes"
description: "The four exit codes shared by the test player and the CLI, the bisect step code 125, and the precedence used when copies disagree."
type: reference
url: https://jand-s.github.io/scion/41664431a8f5/reference/exit-codes/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/exit-codes.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/reference/errors.md
  - https://jand-s.github.io/scion/41664431a8f5/cli.md
  - https://jand-s.github.io/scion/41664431a8f5/agents/next-action.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Exit codes

The test player and every CLI command exit with one of four codes: 0 (ok), 1 (a recipe or check failed), 2 (usage)
or 3 (environment). Agents and scripts decide the next step from the code first, then from `error.code`. A bisect
step uses 125 to tell `git bisect` to skip a commit.

## Codes

| Code | Meaning | Player examples | CLI examples |
|---|---|---|---|
| 0 | Everything ok. Warnings do not count. | Five recipes passed; `-scion-list`; a shard with no pairs. | `trial run` with all passing (even with `changed` verdicts); `doctor` without failed checks. |
| 1 | A recipe, check, graft check or spec failed. | Timeout; `Fail`; `Check(false)`; an error in the log; hash differs between rounds; a graft threw. | `graft check` with errors; `trial approve` refused; `gen --check` stale. |
| 2 | Usage: the call is wrong. | No `-scion-out`; `-scion-only Slsh` (`did_you_mean: ["SlashHit"]`); `-scion-rond`; `-scion-shard 6/6`. | Unknown command or option; no match; `human_required`. |
| 3 | Environment: something outside the call is wrong. | No adapter; `Prepare` failed; watchdog; empty captures; output not writable. | No build; no Unity; editor open; a copy crashed or timed out. |
| 125 | Bisect step only: skip this commit. | | The build failed or rounds disagree at that commit. |

Codes 2 and 3 always come with `error` (`code`, `message`, `hint`, `did_you_mean`): in the CLI envelope, and in the
player's `run.json`.

## Precedence

When one command combines several results (copies of a run, steps), it exits with the most important one:

```text
2 > 3 > 1 > 0
```

Usage errors explain everything after them; environment errors explain failed recipes. A copy killed by a timeout or
Ctrl-C counts as 3 (`copy_timeout`).

## Without -scion-quit

A player started without `-scion-quit` does not exit; its code is in `run.json` (`exit_code`) when the queue ends.

## See also

- [Error and warning codes](/scion/41664431a8f5/reference/errors/)
- [CLI overview](/scion/41664431a8f5/cli/)
- [What to do next](/scion/41664431a8f5/agents/next-action/)

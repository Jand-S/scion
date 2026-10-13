---
title: "trial approve"
description: "Turn results of a run into baselines - agents approve only new ids, people approve changes at an interactive terminal, all or nothing."
type: cli
url: https://jand-s.github.io/scion/41664431a8f5/cli/trial-approve/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/cli/trial-approve.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/baselines.md
  - https://jand-s.github.io/scion/41664431a8f5/agents/hooks.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/baselines.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# trial approve

`dotnet scion trial approve` writes the results of a run into `scion/baselines/` as the new approved state and stages
the files with `git add` (it never commits). A reason is required. Approving ids that already have a baseline needs a
person at an interactive terminal; agents get `human_required`. Approval is all or nothing.

## Synopsis

```text
dotnet scion trial approve <Id...> | --changed | --new --reason "<text>" [--run <run>] [--dry-run]
```

## Description

For each selected id, from the given run (default: last):

1. Refuses if any round failed, the run used `--grafts` or `--warm`, or rounds disagree on the hash.
2. Checks who is approving: without a TTY, only `new` ids pass; anything else is `human_required`.
3. With a TTY and non-`new` ids, shows the list and asks for confirmation.
4. Writes `baseline.json` (hash, frames, fps, player args, rule digest, approval) and this platform's folder
   (`platform.json`, `frames.tsv`, images, learned mask).
5. Runs `git add` on the written files.

## Options

| Option | Default | Meaning |
|---|---|---|
| `<Id...>` | | Ids or patterns. |
| `--changed` | | Every id whose verdict is not `same` or `new`. |
| `--new` | | Every `new` id. |
| `--reason "<text>"` | required | Why; stored in the baseline. |
| `--run <run>` | last run | The run to approve from. |
| `--dry-run` | off | List what would be approved. |

## Output

<!-- validate: cli/trial-approve -->
```json
{"schema_version":1,"command":"trial approve","ok":false,"run":"/Users/alex/orchard/.scion/runs/20261011-143200","approved":0,"unchanged":0,"baselines":[],"staged":false,"refused":[{"id":"SlashHit","why":"changed needs a person at a terminal"}],"needs_person":true,"next_step":"ask a person to run: dotnet scion trial approve SlashHit --reason \"crit shake is stronger\"","error":{"code":"human_required","message":"approving changed baselines needs an interactive terminal","hint":"show the person trial show SlashHit --grid and the command in next_step","did_you_mean":[]}}
```

| Field | Meaning |
|---|---|
| `approved`, `unchanged` | Ids written; ids already equal to the baseline. |
| `baselines[]` | `id`, `was` (the verdict), `round`, `copy`, `dir`. |
| `staged` | `git add` succeeded. |
| `refused[]` | `id`, `why`. |
| `needs_person` | True when a person must run the command. |
| `would_approve` | With `--dry-run`. |

## Exit codes

| Code | When |
|---|---|
| 0 | Approved (or nothing to approve). |
| 1 | Refused: a failed round, `--grafts`, `--warm`, unstable hash. |
| 2 | `human_required`, missing `--reason`, no match. |

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `human_required` | 2 | Non-`new` ids without a TTY. | Stop; give the person the command. |
| `refused` | 1 | An id cannot be approved. | Read `refused[].why`. |
| `usage` | 2 | No `--reason`. | Add one. |
| warning `not_staged` | 0 | `git add` failed. | Stage by hand; check Git LFS. |
| warning `baseline_unreadable` | 0 | An old baseline could not be read; it is replaced. | Review the diff in git. |

## Examples

### Agents: approve the first baselines

```sh
dotnet scion trial approve --new --reason "baselines for the new gallery"
```

### People: approve a reviewed change

```sh
dotnet scion trial approve SlashHit --reason "crit shake is stronger"
```

## See also

- [Baselines and approval](/scion/trials/baselines/)
- [Hooks](/scion/agents/hooks/)
- [baseline.json and platform.json](/scion/reference/files/baselines/)

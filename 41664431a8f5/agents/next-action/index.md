---
title: "What to do next"
description: "Decision table from exit code, error code, verdict, warning and SCN id to the next action an agent should take."
type: reference
url: https://jand-s.github.io/scion/41664431a8f5/agents/next-action/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/agents/next-action.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/reference/errors.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/exit-codes.md
  - https://jand-s.github.io/scion/41664431a8f5/agents/reading-results.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# What to do next

Look up the exit code and the signal a Scion command returned, and do the action in the last column. The signals are
the envelope fields (`error.code`, `warnings[].code`, `failed[]`, `changed[]`, `vs_approved`, `determinism`), the
`SCNxxx` ids of diagnostics and the control-protocol errors. Every code links to its full entry in
[Error and warning codes](/scion/41664431a8f5/reference/errors/). When two rows apply, handle the higher exit code first.

## Exit 0

| Signal | Meaning | Next action |
|---|---|---|
| `warnings` empty | Done. | Follow `next_step`. |
| `vs_approved: same` | Hash, frames and images match the baseline. | Nothing to do. |
| `vs_approved: new` | The recipe has no baseline yet. | Read its `report.md`; if it shows what the recipe should, `dotnet scion trial approve --new --reason "..."` and commit the staged files. |
| `vs_approved: within` | Pixels differ, inside the recipe's tolerance. | Nothing to approve. Mention it if the task was visual. |
| `vs_approved: px_changed` | Pixel hashes changed for a recipe that keeps no images. Informative. | Open `window.png` only if the task is visual; if intended, a person may approve it. |
| `vs_approved: stale` | The baseline was approved on another platform, Unity version, fps or player arguments; images were not compared. | Tell the person; a person approves on this platform when the result looks right. |
| warning [`build_stale`](/scion/41664431a8f5/reference/errors/#build_stale) | Sources or rule files changed after the build. | `dotnet scion build test`, then run again before trusting the result. |
| warning [`build_info_missing`](/scion/41664431a8f5/reference/errors/#build_info_missing) | No `scion-build.json` next to the build. | `dotnet scion build test`. |
| warning [`frame_jitter`](/scion/41664431a8f5/reference/errors/#frame_jitter) | A frame count is one off its baseline (allowed). | Nothing; if it repeats, see [Verify determinism](/scion/41664431a8f5/determinism/verify/). |
| warning [`px_unstable`](/scion/41664431a8f5/reference/errors/#px_unstable) | Pixels vary between rounds of this run. | Treat `px_changed` as noise; see [Particles, animators, UI and physics](/scion/41664431a8f5/determinism/control/). |
| warning [`low_memory`](/scion/41664431a8f5/reference/errors/#low_memory) | Fewer copies fit in free RAM than requested. | Use `suggested_copies` from `doctor` for `-j`. |
| warning [`version_mismatch`](/scion/41664431a8f5/reference/errors/#version_mismatch) | CLI, package and plugin versions differ. | Update the one the message names. |
| warning [`recycle`](/scion/41664431a8f5/reference/errors/#recycle) | A warm copy loaded 50 graft DLLs. | `dotnet scion trial serve --stop`, then `trial serve` again. |
| other warnings | Something did not stop the command. | Read its entry in [Error and warning codes](/scion/41664431a8f5/reference/errors/#warnings). |

## Exit 1

| Signal | Meaning | Next action |
|---|---|---|
| `failed[]` | A (recipe, round) failed: timeout, `Fail`, a check, a logged error, a graft that threw. | Read `failed[].report`; fix the cause; run again. |
| `vs_approved: changed` | Hash, frames (off by more than one), images outside tolerance or the rule digest changed. | If intended: explain it with `trial show --changed` (and `--grid`) and ask a person to approve. If not: fix the code. |
| `determinism.hash_stable: false` | Rounds of one recipe disagree, or no round passed. | Check `failed[]` first. Then `dotnet scion trial run <Id> --rounds 4 --check`; read `determinism.tsv`; see [Verify determinism](/scion/41664431a8f5/determinism/verify/). |
| [`refused`](/scion/41664431a8f5/reference/errors/#refused) in `trial approve` | A selected recipe has a failed round, an unstable hash or was run with `--grafts`. Nothing was written. | Fix that recipe; `next_step` gives the command that approves the others. |
| `diagnostics[]` in `graft check` or `graft build` | Graft errors with SCN ids. | Apply each `hint` and `did_you_mean`; check again. |
| `errors[]` in `build test` | Compiler (`CSxxxx`) or Scion (`SCNxxx`) errors. | Fix the file and line; build again. |
| `spec.ok: false` in `trial run --spec` | An expectation of the spec failed. | Read `spec.json`; never edit the spec to make it pass. |

## Exit 2

| Signal | Meaning | Next action |
|---|---|---|
| [`usage`](/scion/41664431a8f5/reference/errors/#usage), [`unknown_command`](/scion/41664431a8f5/reference/errors/#unknown_command), [`unknown_option`](/scion/41664431a8f5/reference/errors/#unknown_option), [`bad_value`](/scion/41664431a8f5/reference/errors/#bad_value) | Wrong call. | Use `did_you_mean` and `error.hint`, or `<command> --help`. Never retry unchanged. |
| [`no_match`](/scion/41664431a8f5/reference/errors/#no_match) | No recipe, run or anchor with that name. | `dotnet scion trial list` (or `anchor list`); use `did_you_mean`. |
| [`human_required`](/scion/41664431a8f5/reference/errors/#human_required) | The approval replaces a baseline a person approved, and there is no terminal. | Stop. Tell the person what changed and the exact command from `next_step`. Never fake a terminal. |
| [`not_supported`](/scion/41664431a8f5/reference/errors/#not_supported) | The build lacks the define the command needs (control op without `SCION_SLOTS` or `SCION_TRIALS`). | `dotnet scion build test`. |
| [`missing_out`](/scion/41664431a8f5/reference/errors/#missing_out), [`unknown_flag`](/scion/41664431a8f5/reference/errors/#unknown_flag) | A player started by hand got a bad command line. | Fix the `-scion-*` flags; see [Player command line](/scion/41664431a8f5/reference/player-flags/). |

## Exit 3

| Signal | Meaning | Next action |
|---|---|---|
| [`config_not_found`](/scion/41664431a8f5/reference/errors/#config_not_found) | No `scion.json` above the current folder. | `cd` into the project, or `dotnet scion init`. |
| [`config_invalid`](/scion/41664431a8f5/reference/errors/#config_invalid) | A bad key, type or JSON line. | Fix the key the message names; validate with [`scion-config.schema.json`](/scion/41664431a8f5/schemas/v1/scion-config.schema.json). |
| [`build_not_found`](/scion/41664431a8f5/reference/errors/#build_not_found), [`build_wrong_mode`](/scion/41664431a8f5/reference/errors/#build_wrong_mode) | No usable test build. | `dotnet scion build test`. |
| [`unity_not_found`](/scion/41664431a8f5/reference/errors/#unity_not_found) | The project's Unity version is not installed. | Install it, or set `unity.editor` in `scion.local.json`. |
| [`editor_open`](/scion/41664431a8f5/reference/errors/#editor_open) | The editor holds the project and has no `com.unity.pipeline`. | Close the editor, or add `com.unity.pipeline` to the project. |
| [`unity_failed`](/scion/41664431a8f5/reference/errors/#unity_failed) | Unity ended without building and without a compiler error. | Read the log named in the message. |
| [`player_crashed`](/scion/41664431a8f5/reference/errors/#player_crashed), [`copy_timeout`](/scion/41664431a8f5/reference/errors/#copy_timeout), [`watchdog`](/scion/41664431a8f5/reference/errors/#watchdog) | A copy died or stalled. | Read `q<i>/player.log`; rebuild if the build is old; raise `--timeout` only for slow recipes. |
| [`no_adapter`](/scion/41664431a8f5/reference/errors/#no_adapter), [`prepare_failed`](/scion/41664431a8f5/reference/errors/#prepare_failed) | No valid adapter, or an adapter's `Prepare` failed. | Read `error.message` (it names SCN202 or SCN203 when relevant) and the player log. |
| [`capture_broken`](/scion/41664431a8f5/reference/errors/#capture_broken) | Frame capture failed. | Read the player log; run once with `--window` to compare. |
| [`out_not_writable`](/scion/41664431a8f5/reference/errors/#out_not_writable), [`no_free_port`](/scion/41664431a8f5/reference/errors/#no_free_port) | The machine blocks the run. | Fix permissions or `trials.out`; close programs that hold ports, or use fewer copies. |
| [`shard_ignored`](/scion/41664431a8f5/reference/errors/#shard_ignored) | The build predates sharding. | `dotnet scion build test`. |
| [`no_copies`](/scion/41664431a8f5/reference/errors/#no_copies) | `graft inject`, `--warm` or `trial loop` found no warm copy. | `dotnet scion trial serve -j 2`. |
| [`no_simulation`](/scion/41664431a8f5/reference/errors/#no_simulation), [`not_forkable`](/scion/41664431a8f5/reference/errors/#not_forkable) | A level 3 command without an `ISimulation`. | Register one; see [Add Scion to an existing game](/scion/41664431a8f5/start/existing-project/). |
| [`mosaic_unsupported`](/scion/41664431a8f5/reference/errors/#mosaic_unsupported) | `--mosaic` with an adapter that is not an `IMosaicAdapter`. | Run without `--mosaic`, or see [Mosaic](/scion/41664431a8f5/trials/mosaic/). |
| [`model_missing`](/scion/41664431a8f5/reference/errors/#model_missing) | A Judge eye model is not downloaded and the machine is offline. | Connect once, or copy the model into `~/.scion/models/`. |
| [`SCN130`](/scion/41664431a8f5/reference/diagnostics/builds-and-injection/#scn130) | The graft DLL was built for another build. | `dotnet scion graft build`, then inject again. |
| [`SCN131`](/scion/41664431a8f5/reference/diagnostics/builds-and-injection/#scn131) | The graft DLL or the build has another Scion ABI. | Use the same Scion version for the CLI and the build. |
| [`SCN202`](/scion/41664431a8f5/reference/diagnostics/trials/#scn202), [`SCN203`](/scion/41664431a8f5/reference/diagnostics/trials/#scn203) | Invalid adapter, or two adapters for one recipe type. | Fix the adapter as the hint says. |
| [`interrupted`](/scion/41664431a8f5/reference/errors/#interrupted) | Ctrl-C stopped the command. | Run it again. |
| [`internal_error`](/scion/41664431a8f5/reference/errors/#internal_error) | A bug in Scion. | Report it with the stack trace; do not loop. |

## Exit 125

| Signal | Meaning | Next action |
|---|---|---|
| A `bisect` step | The step cannot be judged: the build, the recipe or its rounds broke at that commit. | Nothing: `bisect` skips the commit and goes on. |

## Control protocol

| Signal | Meaning | Next action |
|---|---|---|
| [`busy`](/scion/41664431a8f5/reference/errors/#busy) | A queue is already running in that copy. | Wait for it, or send `quit`. |
| [`bad_token`](/scion/41664431a8f5/reference/errors/#bad_token) | Wrong or stale token. | Re-read `control.json` of that copy. |
| [`not_supported`](/scion/41664431a8f5/reference/errors/#not_supported) | The copy lacks `SCION_SLOTS` or `SCION_TRIALS`. | Rebuild with `dotnet scion build test`. |
| [`bad_json`](/scion/41664431a8f5/reference/errors/#bad_json), [`line_too_long`](/scion/41664431a8f5/reference/errors/#line_too_long), [`unknown_op`](/scion/41664431a8f5/reference/errors/#unknown_op), [`bad_args`](/scion/41664431a8f5/reference/errors/#bad_args), [`shutting_down`](/scion/41664431a8f5/reference/errors/#shutting_down) | A malformed request, or the copy is quitting. | Fix the request; see [Control protocol](/scion/41664431a8f5/reference/control-protocol/). |

## Diagnostics

| Signal | Next action |
|---|---|
| `SCN001`-`SCN005` | Standard warnings and SCN003 (error). [Standard diagnostics](/scion/41664431a8f5/reference/diagnostics/standard/). |
| `SCN101`-`SCN114` | Graft errors; apply the hint. [Graft diagnostics](/scion/41664431a8f5/reference/diagnostics/grafts/). |
| `SCN120`-`SCN131` | Build mode and injection. [Build and injection diagnostics](/scion/41664431a8f5/reference/diagnostics/builds-and-injection/). |
| `SCN201`-`SCN203` | Recipes and adapters. [Trials diagnostics](/scion/41664431a8f5/reference/diagnostics/trials/). |
| `SCN301`-`SCN307` | Determinism scan, in `diagnose` only. [Determinism diagnostics](/scion/41664431a8f5/reference/diagnostics/determinism/). |

## See also

- [Error and warning codes](/scion/41664431a8f5/reference/errors/)
- [Exit codes](/scion/41664431a8f5/reference/exit-codes/)
- [Reading results](/scion/41664431a8f5/agents/reading-results/)
- [Diagnostics](/scion/41664431a8f5/reference/diagnostics/)

---
title: "Error and warning codes"
description: "Every error code of the CLI, the test player and the control protocol, and every warning, with its exit code, cause and next action."
type: reference
url: https://jand-s.github.io/scion/41664431a8f5/reference/errors/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/errors.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/reference/exit-codes.md
  - https://jand-s.github.io/scion/41664431a8f5/agents/next-action.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/diagnostics.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Error and warning codes

Errors appear in `error.code` of the CLI envelope and of the player's `run.json` (exit codes 2 and 3), or in the
`error.code` of a control-protocol response. Warnings appear in `warnings[].code` and never change the exit code.
Diagnostics (`SCNxxx`) are listed separately in [Diagnostics](/scion/reference/diagnostics/). Each entry gives the
exit code, where it comes from, the cause and what to do.

## CLI errors

### usage

Exit 2, CLI. The call is malformed: a missing required argument (`--reason`), conflicting options, or a dirty tree
for `bisect`. Read `error.message` and `<command> --help`; do not retry unchanged.

### unknown_command

Exit 2, CLI. No such command. Use `did_you_mean`.

### unknown_option

Exit 2, CLI. No such option for this command. Use `did_you_mean`.

### bad_value

Exit 2, CLI and player. An option or flag has an invalid value (`--size 1280`, `-scion-shard 6/6`, an unknown tunable
in `--set`). `error.message` names the value and the expected form.

### no_match

Exit 2, CLI and player. A pattern, id, name, head, preset or run matched nothing. Use `did_you_mean`, or list with
`trial list --ids`, `anchor list`, `judge list`.

### human_required

Exit 2, CLI. `trial approve` would replace a baseline a person approved, and there is no interactive terminal. Agents:
stop and give the person the command from `next_step`.

### not_supported

Exit 2 or 3, CLI and control protocol. The operation needs something this build or command does not have (`inject`
on a build without `SCION_SLOTS`, `judge vote` without a browser, an evaluator with `gate: true`).

### config_not_found

Exit 2, CLI. No `scion.json` in the current folder or above. Run `dotnet scion init` or change folder.

### config_invalid

Exit 3, CLI. `scion.json`, `scion.local.json` or `ScionSettings.json` does not match its schema. The message names the
key; validate with the [JSON Schemas](/scion/reference/schemas/).

### build_not_found

Exit 3, CLI. No test build at the discovered path, or no `scion-build.json` beside it. Run `dotnet scion build test`
or pass `--build`.

### build_wrong_mode

Exit 3, CLI. The build lacks `SCION_TRIALS` (a final build). Run `dotnet scion build test`.

### unity_not_found

Exit 3, CLI. No Unity editor of the project's version in the Hub's default path. Install it, or set `unity.editor` in
`scion.local.json`.

### editor_open

Exit 3, CLI. The editor has the project open and `com.unity.pipeline` is not installed, so the CLI cannot build
through it. Close the editor or install the pipeline package.

### unity_failed

Exit 3, CLI. Unity exited without a `BUILD_RESULT` line (crash, license, import error). Read the log in `log`.

### player_crashed

Exit 3, CLI. A copy exited without writing its outputs. Read `q<i>/player.log` and the OS crash report.

### shard_ignored

Exit 0 with a warning, CLI. `-scion-shard` was passed after `--`; the CLI shards the run itself and ignored it.

### copy_timeout

Exit 3, CLI. A copy passed `--timeout`, or the run was interrupted, and the CLI killed its process tree. Raise
`--timeout` for long queues, or find the `WaitUntil` that never ends.

### no_free_port

Exit 3, CLI. No free local port for a co-op group or a control channel. Close other servers.

### out_not_writable

Exit 3, CLI and player. The output folder cannot be created or written. Fix permissions or `trials.out`.

### interrupted

Exit 3, CLI. Ctrl-C or a cancelled MCP call. Partial outputs stay in the run folder.

### internal_error

Exit 3, CLI and player. A bug in Scion (or in a session hook): the stack is in the log. Report it with the log.

### refused

Exit 1, CLI. `trial approve` refused ids: a failed round, a run with `--grafts` or `--warm`, or an unstable hash.
`refused[].why` says which.

### no_copies

Exit 3, CLI. `--warm`, `graft inject` or `restore` found no live copy of this build. Run `dotnet scion trial serve`.

### no_simulation

Exit 3, CLI. A command needs the rules as an `ISimulation` (multiverse, balance specs, hunt, metamorphic) and none is
registered. See [Core API](/scion/reference/api/core/).

### not_forkable

Exit 3, CLI. Forking or looping a span needs `ISaveable` on the simulation state. Implement `Save` and `Load`.

### mosaic_unsupported

Exit 3, CLI. `--mosaic` and no adapter implements `IMosaicAdapter`. Implement it or run copies.

### model_missing

Exit 3, CLI. The judge's eye model is not downloaded. Run the command in `error.hint`.

## Player errors

### missing_out

Exit 2, player. `-scion-trials` without `-scion-out`.

### unknown_flag

Exit 2, player. A `-scion-*` flag that does not exist. Use `did_you_mean`.

### no_adapter

Exit 3, player. No `[assembly: TrialAdapter]` in the build. Register one; `dotnet scion diagnose --adapter` writes a
skeleton.

### prepare_failed

Exit 3, player. An adapter's `Prepare` threw, timed out (60 s real time) or called `Fail`. Read the copy's
`player.log`.

### watchdog

Exit 3, player. No progress for 120 seconds of real time. Look for a wait that never ends or a modal dialog.

### capture_broken

Exit 3, player. Captures are empty, usually `-batchmode` without offscreen capture. Let the CLI start players, or add
`-scion-offscreen`.

## Control protocol errors

### bad_json

The request line is not valid JSON. The connection stays open.

### line_too_long

The request line is longer than 64 KiB. The connection is closed.

### bad_token

The `token` does not match `control.json`. Read the token from the live file again; the copy may have restarted.

### unknown_op

The `op` is not one of `hello`, `status`, `inject`, `restore`, `disable`, `run`, `quit`.

### bad_args

The arguments of the op are missing or of the wrong type (`run` without `out`).

### busy

A `run` while a queue is running; also returned by the CLI when every warm copy is busy. Wait, or start more copies.

### shutting_down

The copy received `quit` and accepts no more requests.

## Warnings

### build_stale

Sources or rule files changed after the test build was made. Rebuild before trusting a result.

### build_info_missing

The build has no `scion-build.json`; `build_id` and git fields are null. Build with `dotnet scion build test`.

### unity_exit

Unity exited with a non-zero code after a successful build. Usually harmless; read the log.

### unreadable_dll

A DLL of the build could not be read while listing recipes or anchors. Rebuild.

### name_at_run_time

A graft names its target with a string computed at run time; tools cannot list it. Use `nameof` or a literal.

### recipe_unbuildable

A recipe has no public parameterless constructor (SCN201). It fails when selected.

### recipes_unchecked

No determinism scan or recipe list yet, so some checks were skipped. Open the project in Unity once.

### low_memory

Free memory is below what the requested copies need. Lower `-j`.

### pairs_missing

Some (id, round) pairs are missing from the outputs (a crashed copy, or ids present in only one side of a diff).

### result_unreadable

A `result.json` is missing or invalid. The copy probably crashed; read its log.

### frame_jitter

Frame counts differ by one between rounds. Usually the Netcode tick; see [Netcode clock](/scion/determinism/netclock/).

### px_unstable

Pixel hashes vary between rounds of this run, so `px_changed` is noise. See
[Particles, animators, UI and physics](/scion/determinism/control/).

### baseline_unreadable

A baseline file could not be read (often Git LFS pointers not pulled). Run `git lfs pull`.

### summary_not_written

The run was interrupted before `summary.json` was written. Run again.

### build_changed

Live warm copies run an older build than the current one. Stop and serve again.

### not_staged

`trial approve` or `graft promote` wrote files but `git add` failed. Stage them yourself.

### recycle

A warm copy passed 50 injections; it is restarted at the next boundary because Mono never unloads assemblies.

### mask_cap

A learned unstable mask would cover more than 2 % of an image; it was not saved. Make the recipe deterministic.

### version_mismatch

The CLI and the package have different versions. Install the same version of both.

## See also

- [Exit codes](/scion/reference/exit-codes/)
- [What to do next](/scion/agents/next-action/)
- [Diagnostics](/scion/reference/diagnostics/)
- [Control protocol](/scion/reference/control-protocol/)

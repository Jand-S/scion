---
title: "Run Scion in CI"
description: "GitHub Actions with a dotnet job for analyzers, graft checks and generated files, a Unity job for trials, a baseline guard and a nightly batch."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/cookbook/ci/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/cookbook/ci.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/determinism/verify.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/baselines.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/trial-run.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Run Scion in CI

This recipe runs Scion in GitHub Actions with three jobs. A fast `dotnet` job needs no Unity license: analyzers,
strict graft checks, `gen --check`, unit tests and specs on the simulator. A Unity job builds the test player with
GameCI, runs the trials and uploads the run folder. A nightly job runs many rounds with `--check --jitter` to catch
rare non-determinism. Pull requests that change baselines must come with the approved files.

## When to use

- Every repository with Scion: the dotnet job costs under two minutes and catches most mistakes.
- Add the Unity job when you have a license for CI (GameCI documents the activation).

## Before you begin

- `scion.json`, the tool manifest and baselines committed (PNGs in Git LFS).
- A Unity license secret for the Unity job (`UNITY_LICENSE`, `UNITY_EMAIL`, `UNITY_PASSWORD`) set in the repository
  settings, never in files.

## Steps

### 1. The dotnet job

`.github/workflows/scion.yml`:

```yaml
name: scion
on: [pull_request, push]
jobs:
  dotnet:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with: { lfs: true }
      - uses: actions/setup-dotnet@v4
        with: { dotnet-version: '10.0.x' }
      - run: dotnet tool restore
      - run: dotnet test Tests/Orchard.Core.Tests
      - run: dotnet test Tests/Orchard.Grafts.Tests
      - run: dotnet scion graft check --strict
      - run: dotnet scion gen --check
      - run: dotnet scion trial run --spec RulesHold
```

### 2. The Unity job

```yaml
  unity:
    runs-on: ubuntu-latest
    needs: dotnet
    steps:
      - uses: actions/checkout@v4
        with: { lfs: true }
      - uses: actions/setup-dotnet@v4
        with: { dotnet-version: '10.0.x' }
      - run: dotnet tool restore
      - uses: game-ci/unity-builder@v4
        env:
          UNITY_LICENSE: ${{ secrets.UNITY_LICENSE }}
          UNITY_EMAIL: ${{ secrets.UNITY_EMAIL }}
          UNITY_PASSWORD: ${{ secrets.UNITY_PASSWORD }}
        with:
          targetPlatform: StandaloneLinux64
          buildMethod: Scion.Editor.ScionBuild.TestFromCommandLine
          customParameters: -scion-build-out .scion/builds/test
      - run: dotnet scion trial run --rounds 2 -j 2 --build .scion/builds/test
      - uses: actions/upload-artifact@v4
        if: always()
        with: { name: scion-run, path: .scion/runs }
```

### 3. Fail on unapproved changes

A run exits 0 with `ok: true` when recipes pass, even with `changed` verdicts
([`ok` and `changed`](/scion/41664431a8f5/cli/#ok-and-changed)). To require that visual changes come with approved
baselines, fail the job when any id is `changed` or `new`:

```yaml
      - run: |
          dotnet scion trial show --changed --format json > changed.json
          test "$(jq .count changed.json)" = "0"
```

Linux baselines have their own platform folder; approve them once from a Linux machine or a CI artifact.

### 4. The nightly batch

```yaml
on:
  schedule: [{ cron: '0 3 * * *' }]
# same Unity job, with:
      - run: dotnet scion trial run --rounds 10 --check --jitter 20 -j 4 --build .scion/builds/test
```

## Reference

| Job | Needs Unity | Catches |
|---|---|---|
| dotnet | no | Analyzer errors, graft signatures, stale manifest, rule regressions, broken specs |
| Unity | yes | Failing recipes, determinism, visual changes |
| Nightly | yes | Rare non-determinism, timing-dependent bugs |

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `unity_failed` | 3 | The license step failed. | Check the GameCI activation secrets. |
| `baseline_unreadable` | warning | LFS files not checked out. | `lfs: true` in checkout. |
| `capture_broken` | 3 | No GPU and no offscreen capture. | Keep the default headless mode; do not pass `--window`. |

## Examples

### Only the dotnet job on documentation changes

```yaml
on:
  pull_request:
    paths-ignore: ['**/*.md']
```

### Upload only failures

```yaml
      - run: dotnet scion trial show --failed --format json > failed.json
```

## See also

- [Verify determinism](/scion/41664431a8f5/determinism/verify/)
- [Baselines and approval](/scion/41664431a8f5/trials/baselines/)
- [trial run](/scion/41664431a8f5/cli/trial-run/)

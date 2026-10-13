---
title: "Install"
description: "Install the Scion package, the dotnet scion CLI and the agent integration, then verify the setup with dotnet scion doctor."
type: tutorial
url: https://jand-s.github.io/scion/41664431a8f5/start/install/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/start/install.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/start/requirements.md
  - https://jand-s.github.io/scion/41664431a8f5/agents/setup.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/init.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/doctor.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Install

This tutorial installs Scion 1.0.0 into a Unity project: the `me.jandson.scion` package (Git URL or OpenUPM), the
`Scion.Cli` .NET tool (`dotnet scion`), the project files that `dotnet scion init` writes (`scion.json`, folders,
`.gitignore` lines, the `.asmref` for grafts) and the integration with your AI agent. Every path ends with
`dotnet scion doctor`, which proves the setup works.

## Before you begin

- Unity 6000.3 or later, with the Mono and IL2CPP modules for your platform ([Requirements](/scion/start/requirements/)).
- The .NET 10 SDK: `dotnet --version` prints 10 or later.
- git, and git LFS if you will keep image baselines.
- The project root is a git repository. Run every command below from it.

There are three ways in, and they converge:

| Path | You do | Scion does |
|---|---|---|
| A. Claude Code | `/plugin install scion@scion`, then ask "install Scion in this project" | Runs steps 1 to 5 below, shows the `--dry-run` preview first and asks you to confirm |
| B. Codex, Cursor, Copilot or another agent | Steps 1 to 3 by hand, then step 4 for your agent | Gives your agent the MCP server and the `AGENTS.md` block |
| C. By hand or in CI | Steps 1 to 3 and 5 | Nothing else |

## Steps

### 1. Add the package

Pick one source. Both pin the version with a tag.

**Git URL.** Add this line to the `dependencies` of `Packages/manifest.json`:

```json
{
  "dependencies": {
    "me.jandson.scion": "https://github.com/Jand-S/scion.git?path=/Packages/me.jandson.scion#v1.0.0"
  }
}
```

**OpenUPM.** With the [OpenUPM CLI](https://openupm.com/docs/getting-started-cli.html) installed:

```sh
openupm add me.jandson.scion@1.0.0
```

Unity downloads the package and its dependency `com.unity.nuget.mono-cecil`. From the next compilation on, the
weaver processes every assembly that references `Scion.Runtime`.

### 2. Install the CLI

The CLI is a local .NET tool, pinned in the repository's tool manifest so every clone and CI job uses the same
version:

```sh
dotnet new tool-manifest            # only if .config/dotnet-tools.json does not exist yet
dotnet tool install --local Scion.Cli --version 1.0.0
dotnet scion --version
```

```text
1.0.0
```

On another clone, `dotnet tool restore` installs the version the manifest names. Keep the CLI version equal to the
package version: a graft DLL built by one version is refused by a player of another
([SCN131](/scion/reference/diagnostics/builds-and-injection/#scn131)), and `doctor` warns with `version_mismatch`.

### 3. Set up the project

`dotnet scion init` adds Scion's files without moving anything. Preview first:

```sh
dotnet scion init --existing --dry-run
```

<!-- validate: cli/init -->
```json
{"schema_version":1,"command":"init","ok":true,"dry_run":true,"created":["scion.json","scion/grafts/","scion/recipes/","scion/baselines/","Assets/Orchard/Grafts/Orchard.Grafts.asmref",".config/dotnet-tools.json"],"changed":[".gitignore",".gitattributes","Packages/manifest.json"],"skipped":["ProjectSettings/ScionSettings.json"],"adoption_level":1,"next_step":"run without --dry-run: dotnet scion init --existing"}
```

Then apply it:

```sh
dotnet scion init --existing
```

What `init` writes:

| File or folder | Content |
|---|---|
| `scion.json` | CLI configuration with defaults for your project ([Configuration](/scion/reference/configuration/)) |
| `ProjectSettings/ScionSettings.json` | Weaver, build and analyzer settings; `main_assembly` set to your game assembly |
| `scion/grafts/`, `scion/recipes/`, `scion/baselines/` | Draft grafts, extra recipes and match files, approved baselines |
| `<grafts folder>/<Game>.Grafts.asmref` | Makes the promoted grafts folder compile into your game assembly |
| `.gitignore` | `.scion/` (runs, builds, caches) and `scion.local.json` |
| `.gitattributes` | `scion/baselines/**/*.png filter=lfs diff=lfs merge=lfs -text` |
| Your game `.asmdef` | A reference to `Scion.Runtime` (and `Scion.Trials`, `Scion.Injection` for test-only code) |

`init` is idempotent: run it again and it reports everything as `skipped`. To start a project from a genre
template instead, see [Start a new project](/scion/start/new-project/).

### 4. Connect your agent

**Claude Code.** Install the plugin. It brings the `scion` skill, the hooks, the MCP server and the
`scion-triage` subagent:

```sh
/plugin marketplace add Jand-S/scion
/plugin install scion@scion
```

**Codex, Cursor, GitHub Copilot and others.** Add the MCP server to your agent's configuration and the Scion block
to `AGENTS.md`. The exact files per agent are on [Set up your agent](/scion/agents/setup/). The server is the CLI
itself:

```json
{"mcpServers":{"scion":{"command":"dotnet","args":["scion","mcp"]}}}
```

### 5. Run doctor

```sh
dotnet scion doctor
```

<!-- validate: cli/doctor -->
```json
{"schema_version":1,"command":"doctor","ok":true,"level":2,"version":"1.0.0","adoption_level":1,"adoption_next":"level 2: write an adapter and a recipe (dotnet scion new trial MyFirstTrial)","checks":[{"id":"build","status":"warn","detail":"no test build at /Users/alex/orchard/.scion/builds/test/Orchard.app","hint":"run: dotnet scion build test"},{"id":"config","status":"ok","detail":"/Users/alex/orchard/scion.json"},{"id":"unity","status":"ok","detail":"6000.3.25f1"},{"id":"machine","status":"ok","detail":"10 cores, 9.1 GB free of 16.0 GB"}],"suggested_copies":4,"next_step":"run: dotnet scion build test"}
```

Exit 0 means the setup is usable; read the `hint` of each `warn`. On a fresh project, the `build` check warns until
the first `dotnet scion build test`. Exit 3 means a check failed: fix what `error.hint` says and run `doctor` again.

## Check the result

- `dotnet scion doctor` exits 0.
- `dotnet scion --version` prints `1.0.0`, and the Unity Package Manager shows Scion 1.0.0.
- `git status` shows the files from step 3, ready to commit.
- In Unity, the console has no `Scion SCNxxx` errors after the next compilation.

Commit the new files: `scion.json`, `ProjectSettings/ScionSettings.json`, `.config/dotnet-tools.json`, the
`.asmref`, `.gitignore`, `.gitattributes` and `Packages/manifest.json`.

## Update and uninstall

- **Update:** change the tag in `Packages/manifest.json` (or `openupm add me.jandson.scion@<version>`), then
  `dotnet tool update --local Scion.Cli --version <version>`. Read the [changelog](/scion/help/changelog/) first.
- **Uninstall:** `dotnet scion uninstall --dry-run` lists what it would remove; `dotnet scion uninstall` removes the
  files `init` created, the package line and the tool. It keeps `scion/baselines/` and your grafts, recipes and specs
  unless you pass `--purge`. See [init, new, uninstall](/scion/cli/init/).

## Next steps

- [Quickstart](/scion/start/quickstart/): a graft, a recipe and a first approved baseline in ten minutes.
- [Add Scion to an existing game](/scion/start/existing-project/): what to do on day one and how to reach level 2.
- [For AI agents](/scion/agents/): the loop, the rules and the machine-readable files.

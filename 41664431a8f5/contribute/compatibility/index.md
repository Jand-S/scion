---
title: "Versioning and compatibility"
description: "What Scion's version numbers promise for the C# API, the weaver ABI, player flags, file formats, CLI output, SCN ids and the CLI and package pairing."
type: reference
url: https://jand-s.github.io/scion/41664431a8f5/contribute/compatibility/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/contribute/compatibility.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/help/changelog.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/schemas.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/diagnostics/builds-and-injection.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Versioning and compatibility

Scion follows Semantic Versioning: within 1.x nothing documented breaks; breaking changes wait for 2.0. This page says
exactly what "documented" covers, how each surface evolves, and which pieces must have the same version.

## Surfaces

| Surface | Within 1.x | Breaking change means |
|---|---|---|
| C# API (public types and members) | Members may be added; none removed or changed. | Removing or changing a signature. |
| Weaver ABI (`Anchors.AbiVersion`) | Stays 1. | A new ABI number; DLLs of another ABI are refused (SCN131). |
| Player flags (`-scion-*`) | Flags may be added; none removed or changed in meaning. | Removing or redefining a flag. |
| File formats (`schema_version`) | Optional fields may be added; readers ignore unknown fields. | `schema_version: 2` and a `v2/` schema folder. |
| CLI output (envelope and command fields) | Fields may be added. | Removing or renaming a field. |
| Exit codes and error codes | Stable; new codes may be added. | Changing what a code means. |
| SCN ids | Never reused; new ids may be added; severities may be lowered, not raised. | Raising a warning to an error. |
| Verdicts (`same`, `within`, `px_changed`, `changed`, `new`, `stale`) | Stable. | Adding or redefining a verdict. |

## Versions that must match

| Pair | Rule | When they differ |
|---|---|---|
| CLI and package | Same version. | `doctor` warns `version_mismatch`; injection may fail with SCN131. |
| Graft DLL and test build | Built against the same `build_id`. | SCN130. |
| `Scion.Testing`, `Scion.Compare`, `Scion.Specs`, `Scion.Core` and the package | Same version. | Tests may compile against members the game does not have. |

The CLI's NuGet version is the package version; local development builds use a `-dev` suffix.

## Deprecation

A member to be removed in 2.0 is marked `[Obsolete]` with the replacement in the message for at least one minor
version, and listed in the changelog under "Deprecated".

## Supported Unity versions

Each release states its minimum Unity version in `package.json` (`unity: 6000.3`). A minor release may raise it only
when the previous Unity is out of Unity's support.

## See also

- [Changelog](/scion/41664431a8f5/help/changelog/)
- [JSON Schemas](/scion/41664431a8f5/reference/schemas/)
- [SCN120-SCN131](/scion/41664431a8f5/reference/diagnostics/builds-and-injection/)

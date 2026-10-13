---
title: "Diagnostics"
description: "Where Scion's SCN diagnostics appear, their message format, how to suppress warnings, and a summary of every id from SCN001 to SCN307."
type: diagnostics
url: https://jand-s.github.io/scion/41664431a8f5/reference/diagnostics/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/diagnostics.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/reference/errors.md
  - https://jand-s.github.io/scion/41664431a8f5/standard/enforcement.md
  - https://jand-s.github.io/scion/41664431a8f5/determinism/diagnose.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Diagnostics

Scion reports problems in code as diagnostics with stable ids `SCNxxx`. They come from four sources: the Roslyn
analyzers (in the IDE and in Unity), the weaver (as compile errors when Unity compiles), the build and the injection
(as CLI errors), and the determinism scan (only through `dotnet scion diagnose`). Ids are never reused; each has a
message and a hint with the fix.

## Format

```text
Scion SCN104: The signature of CritShake.After does not match HitPresenter.Play. Hint: Expected: public static void After(HitPresenter presenter, HitEvent e, ref Call<float> c)
```

In Unity the message has the file and line of the graft; in the CLI it comes as
`{file, line, code, message, hint, did_you_mean}` inside `diagnostics[]` or `errors[]`.

## Where they appear

| Source | Ids | Shown in |
|---|---|---|
| Analyzers | SCN001-SCN005, SCN101-SCN104, SCN108-SCN110, SCN112 | IDE, Unity console, `graft check` |
| Weaver | SCN101-SCN109, SCN111, SCN112, SCN114 | Unity console as compile errors, `build test`, `graft check` |
| Draft compiler | SCN103-SCN105, SCN113 | `graft build`, `graft inject` |
| Build | SCN120, SCN121 | `build test`, `build final` |
| Runtime and CLI | SCN130, SCN131, SCN201-SCN203 | `InjectReport.Errors`, `result.json`, `run.json`, envelope |
| Determinism scan | SCN301-SCN307 | `Library/Scion/determinism-<assembly>.json`, `diagnose` |

SCN3xx never reach the Unity console, so hundreds of findings never bury a real compile error.

## Summary

| Id | Severity | Short |
|---|---|---|
| SCN001 | warning | Rule class not sealed |
| SCN002 | warning | Rule class inherits more than one level |
| SCN003 | error | Graft writes rule state |
| SCN004 | warning | `float` in rule code |
| SCN005 | warning | `[Tunable]` without a range |
| SCN101 | error | Graft is not `static void`, public or internal, non-generic |
| SCN102 | error | Target outside the graft's assembly |
| SCN103 | error | Target method does not exist |
| SCN104 | error | Signature does not match |
| SCN105 | error | Target cannot be an anchor |
| SCN106 | error | Overloads collide on one anchor id |
| SCN107 | error | `AnchorScope` outside its assembly |
| SCN108 | error | Skip or Return outside a non-observer `[Before]` |
| SCN109 | error | Wrong `Call` type |
| SCN110 | error | Game code calls a graft |
| SCN111 | warning | Several grafts share `Call.Tag` |
| SCN112 | error | Prefix matches several overloads |
| SCN113 | error | No slot for the target in this build |
| SCN114 | error | Redirect target missing in `Scion.Runtime` |
| SCN120 | error | Slots need Mono |
| SCN121 | error | Final build with test defines |
| SCN130 | error | Graft DLL built for another build |
| SCN131 | error | Graft DLL of another ABI |
| SCN201 | error | Recipe without public parameterless constructor |
| SCN202 | error | Invalid adapter |
| SCN203 | error | Two adapters for one recipe type |
| SCN301 | info | Read redirected in test builds |
| SCN302 | info | Creation replaced in test builds |
| SCN303 | warning | Wall-clock read not redirected |
| SCN304 | warning | Threads or tasks |
| SCN305 | warning | Unordered collection or query |
| SCN306 | warning | Seed from `GetHashCode` |
| SCN307 | warning | `Guid` created at run time |

## Suppression

Errors cannot be suppressed. Warnings use the standard Roslyn mechanisms:

```csharp
#pragma warning disable SCN111
// ...
#pragma warning restore SCN111

[SuppressMessage("Scion", "SCN002", Justification = "Netcode requires NetworkBehaviour")]
```

or a `.globalconfig` for a whole project (`dotnet_diagnostic.SCN004.severity = none`). `doctor --level 3` lists
every suppression with its justification.

## See also

- [SCN001-SCN005](/scion/41664431a8f5/reference/diagnostics/standard/)
- [SCN101-SCN114](/scion/41664431a8f5/reference/diagnostics/grafts/)
- [SCN120-SCN131](/scion/41664431a8f5/reference/diagnostics/builds-and-injection/)
- [SCN201-SCN203](/scion/41664431a8f5/reference/diagnostics/trials/)
- [SCN301-SCN307](/scion/41664431a8f5/reference/diagnostics/determinism/)

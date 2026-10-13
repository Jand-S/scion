---
title: "License"
description: "Scion is licensed under Apache-2.0, with a NOTICE file, third-party notices for Mono.Cecil, and license notes for the judge's models."
type: reference
url: https://jand-s.github.io/scion/41664431a8f5/help/license/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/help/license.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/judge/models.md
  - https://jand-s.github.io/scion/41664431a8f5/help/faq.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# License

Scion is open source under the Apache License, Version 2.0. The package, the CLI and the NuGet packages carry the same
license and the same `NOTICE`. Third-party code and models keep their own licenses.

## Scion

- License: Apache-2.0 (`LICENSE` in the package).
- Notice: `NOTICE` in the package, to keep with redistributions.
- You may use Scion in commercial games. Final builds contain only Scion's runtime attributes and generated
  dispatchers, under the same license.

## Third-party software

| Component | Used by | License |
|---|---|---|
| Mono.Cecil (`com.unity.nuget.mono-cecil`) | The weaver, the CLI | MIT |
| Roslyn (`Microsoft.CodeAnalysis`) | Analyzers, `graft check`, `graft build` | MIT |

## Models

The judge's eye models are downloaded separately and keep their licenses; check each model card before
redistributing heads trained on them ([Models and licenses](/scion/41664431a8f5/judge/models/)). Heads trained on your own votes and
captures are yours.

## Documentation

This documentation is part of the package and is licensed under Apache-2.0 too.

## See also

- [Models and licenses](/scion/41664431a8f5/judge/models/)
- [FAQ](/scion/41664431a8f5/help/faq/)

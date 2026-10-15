# llamarc42 CLI

This repository is the planned user-facing command surface for the llamarc42
documentation framework. CLI v0.1 is not yet implemented.

The accepted architecture assigns the CLI command invocation, argument
handling, and human and machine-readable presentation. Reusable documentation
discovery, validation, template selection, and initialization behavior belongs
to [`llamarc42/core`](https://github.com/llamarc42/core); the CLI must invoke
that behavior without duplicating its rules.

The existing [`cli-spec.md`](cli-spec.md) describes the earlier local-copilot
direction. It is retained for traceability but is not current implementation
authority. [CLI-01](https://github.com/llamarc42/cli/issues/20) owns its
reviewed replacement before CLI implementation begins.

Current governing sources are maintained in
[`llamarc42/docs`](https://github.com/llamarc42/docs):

- [project definition](https://github.com/llamarc42/docs/blob/main/projects/llamarc42/project.md);
- [Standalone Initialization Contract v0.1](https://github.com/llamarc42/docs/blob/main/projects/llamarc42/specifications/standalone-initialization-contract-v0.1.md);
- [Deterministic Validation Contract v0.1](https://github.com/llamarc42/docs/blob/main/projects/llamarc42/specifications/deterministic-validation-contract-v0.1.md); and
- [ADR-006 repository responsibilities](https://github.com/llamarc42/docs/blob/main/projects/llamarc42/decisions/ADR-006-framework-repository-responsibilities.md).

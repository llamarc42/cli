# CLI Specification — llamarc42

## Status

Draft — in progress (Issue 1–3 complete, review deferred)

---

## Purpose

Define the CLI surface for llamarc42 before implementation so behavior is explicit, testable, and aligned with the system architecture.

This specification defines:

* command hierarchy
* command behavior
* argument and input handling
* session interaction rules

This document is the **source of truth** for CLI behavior.

---

## Design Principles

1. **Workflow-first design**
   Commands are organized around developer workflows, not internal system structure.

2. **Local-first**
   All behavior assumes local files, sessions, and context.

3. **Deterministic behavior**
   No hidden behavior, no implicit ambiguity.

4. **Automation-safe**
   CLI must behave consistently in scripts and pipelines.

5. **CLI is a surface only**
   All logic remains in Core.

---

# 1. Command Hierarchy

## Canonical Executable

```text
llamarc42
```

### Alias

* Deferred until CLI surface stabilizes
* Not part of current milestone

---

## Command Grouping Philosophy

Top-level commands are organized around **user workflows**, not internal system domains.

Commands answer:

> “What is the developer trying to do?”

Not:

> “What subsystem is being used?”

---

## Top-Level Command Tree

```text
llamarc42 chat
llamarc42 sessions
llamarc42 docs
llamarc42 project
llamarc42 help
llamarc42 version
```

---

## Command Responsibilities

### `chat`

Primary conversational workflow.

### `sessions`

Session lifecycle and continuity.

### `docs`

Documentation inspection and validation.

### `project`

Project inspection and validation.

### `help`

Command discovery.

### `version`

CLI version output.

---

## Command Design Rules

### A command is top-level only if:

* represents a primary user workflow
* has a clear responsibility boundary
* is understandable without internal knowledge
* cannot fit under an existing command

---

### A command is NOT top-level if:

* it exposes internal concepts (retrieval, prompt, etc.)
* it supports another workflow
* it overlaps another command
* it is too narrow

---

## Explicitly Excluded Top-Level Commands

| Command     | Reason                     |
| ----------- | -------------------------- |
| `context`   | internal retrieval concept |
| `retrieval` | implementation detail      |
| `prompt`    | internal process           |
| `config`    | not yet a primary workflow |
| `debug`     | handled via flags          |

---

## Deferred Commands

### `review`

**Reason:**

* not currently a first-class workflow in PowerShell or documented system behavior
* can be performed via `chat` using prompt-based interaction
* introducing it now would create new behavior at the CLI layer

**Future consideration:**

* may be introduced once review workflows are formalized in Core and PowerShell

---

## Naming Rules

* lowercase only
* no abbreviations
* user-task language
* no PowerShell verb-noun
* no aliases in spec phase

---

## Examples

```text
llamarc42 chat
llamarc42 chat "summarize the retrieval design"

llamarc42 sessions list
llamarc42 docs validate
llamarc42 project show
```

---

# 2. Chat Command

## Purpose

`llamarc42 chat` is the primary conversational entry point.

It supports:

* **one-shot mode** (single prompt, immediate response)
* **interactive mode** (REPL-style session)

---

## Command Forms

```text
llamarc42 chat "<prompt>"
llamarc42 chat --name <session-name> "<prompt>"
llamarc42 chat --session <session-id> "<prompt>"

llamarc42 chat
llamarc42 chat --name <session-name>
llamarc42 chat --session <session-id>
```

---

## Mode Selection

### One-shot mode

Triggered when:

* positional prompt is provided
* stdin is piped

Behavior:

* process input once
* return response
* exit

---

### Interactive mode

Triggered when:

* no prompt
* no piped stdin
* interactive terminal
* no `--non-interactive`

Behavior:

* enter REPL session
* remain active until exit

---

### Invalid condition

If:

* no prompt
* no stdin
* non-interactive environment

→ fail with explicit error

---

## Input Handling

### Sources

1. positional argument
2. piped stdin

### Precedence

```
positional argument > stdin
```

If both are present:

* positional argument is used
* stdin is ignored

---

## Session Targeting

### `--session <id>`

* must match exactly one existing session
* must not create new session
* failure = error

---

### `--name <name>`

* resolve existing session if unique
* create if not found
* ambiguity possible

---

### Invalid combination

```
--session + --name → error
```

---

## Default Session Behavior

### One-shot

* uses new isolated session context
* does NOT reuse previous session

---

### Interactive

* resumes last active session if available
* otherwise creates new session

---

## Ambiguity Handling

### Interactive

* may display matches and allow selection

### Non-interactive

* must fail
* must not prompt
* must display matches

---

## Piped Input Behavior

When stdin is piped:

* always one-shot
* never interactive
* never prompt
* must be automation-safe

Examples:

```text
git diff | llamarc42 chat
Get-Content file.md | llamarc42 chat
echo "summarize" | llamarc42 chat
```

---

## `--non-interactive`

Forces automation-safe mode.

* no prompts
* no REPL
* ambiguity = error
* missing input = error

---

## Behavior Summary

### One-shot

* prompt or stdin
* isolated session
* single response
* exit

---

### Interactive

* terminal session
* resumes or creates session
* continuous interaction

---

# 3. Sessions Command

## Purpose

`llamarc42 sessions` provides lifecycle management for persisted conversation sessions.

---

## Command Forms

```text
llamarc42 sessions list
llamarc42 sessions new [--name <name>]

llamarc42 sessions resume <session-id|name>

llamarc42 sessions show <session-id|name>
llamarc42 sessions summarize <session-id|name>
```

---

## Session Identity Model

Each session has:

* unique system ID
* optional name (not guaranteed unique)

---

## Identity Resolution

| Input           | Behavior    |
| --------------- | ----------- |
| ID              | exact match |
| name (1 match)  | success     |
| name (0)        | error       |
| name (multiple) | ambiguous   |

---

## Ambiguity Handling

### Interactive

* display matches
* allow selection

### Non-interactive

* display matches
* fail
* instruct to use ID

---

## Subcommands

### `sessions list`

* lists sessions
* default output: table

---

### `sessions new`

* creates session
* optional name
* becomes active session

---

### `sessions resume`

* resolves session
* enters interactive chat
* equivalent to `chat --session`

---

### `sessions show`

* displays metadata and summary

---

### `sessions summarize`

* generates or returns summary

---

## Persistence

* project-scoped
* stored locally
* append-only history
* summaries separate from raw data

---

## Examples

```text
llamarc42 sessions list
llamarc42 sessions new --name architecture
llamarc42 sessions resume architecture
llamarc42 sessions show architecture
```

Perfect — here is the **clean, updated spec section** for Issue 6 with:

* `validate` removed
* `new` deferred
* clear boundaries
* explicit deferral notes

This drops directly into your `cli-spec.md`.

---

# 4. Docs Command

## Purpose

`llamarc42 docs` provides inspection workflows for project documentation.

This command group is responsible for helping a developer understand:

* what documentation is available
* what a specific document contains

`docs` operates on documentation artifacts as the source of truth.

It does **not** perform validation, scaffolding, conversational analysis, or modification of documentation.

---

## Command Forms

```text
llamarc42 docs list
llamarc42 docs show <path|document-id>
```

---

## Responsibility Boundary

### `docs` owns

* listing known documentation artifacts
* showing a specific documentation artifact

### `docs` does not own

* documentation validation
* documentation scaffolding
* project configuration reporting
* session operations
* conversational workflows
* modification of documentation files

---

## `docs list`

Lists documentation artifacts known to the current project.

### Behavior

* returns documentation files discoverable within the project scope
* helps the developer understand available source-of-truth artifacts

### Minimum reported fields

* document identifier or relative path
* existence status

### Output

* default: table
* supports: plain, json

---

## `docs show`

Displays the contents of a specific document.

```text
llamarc42 docs show <path|document-id>
```

### Accepted target forms

* relative path
* document identifier (if defined by project)

### Behavior

* resolves the target document
* prints document contents in a readable form
* must fail explicitly if the target cannot be resolved

### Resolution rules

1. exact path match
2. exact document identifier match
3. otherwise fail

---

## Docs Examples

### List docs

```text
llamarc42 docs list
```

### Show a document

```text
llamarc42 docs show docs/projects/llamarc42/architecture/boundaries.md
```

---

# 5. Project Command

## Purpose

`llamarc42 project` provides inspection workflows for the current project.

This command group is responsible for helping a developer understand:

* what project is currently active
* how the CLI has resolved the project context

`project` operates at the project boundary, not the document boundary.

---

## Command Forms

```text
llamarc42 project show
```

---

## Responsibility Boundary

### `project` owns

* displaying current project metadata
* reporting resolved project paths and context

### `project` does not own

* documentation inspection (beyond reporting paths)
* session operations
* conversational workflows
* validation of project structure (in this milestone)
* project scaffolding

---

## `project show`

Displays the currently resolved project and its key metadata.

```text
llamarc42 project show
```

### Behavior

* reports the current project context
* shows key paths and detected structure used by the CLI
* helps the developer confirm that the CLI is operating against the intended project

### Minimum reported fields

* project root path
* documentation root path (if detected)
* project identification (if available)

### Output

* default: plain
* supports: json

---

## Project Examples

### Show current project

```text
llamarc42 project show
```

---

# Deferred Workflows

The following commands are intentionally deferred until the canonical project model is defined in Core and aligned across all surfaces:

### Documentation workflows

* `llamarc42 docs validate`
* `llamarc42 docs new`

### Project workflows

* `llamarc42 project validate`
* `llamarc42 project new`

---

## Reason for Deferral

These commands depend on a formal definition of:

* canonical project structure
* required vs optional documentation artifacts
* validation rules
* scaffolding/templates

These behaviors belong to Core and must be defined there before being exposed in the CLI.

---

## Future Consideration

These workflows may be introduced in a future milestone once:

* the project model is explicitly defined
* PowerShell and CLI behavior are aligned
* validation and scaffolding rules are stable

---

## Boundary Between `docs` and `project`

### Use `docs` when:

* inspecting documentation artifacts
* viewing document contents

### Use `project` when:

* inspecting current project context
* understanding how the CLI resolved the project

# 6. Argument Conventions

## Purpose

CLI arguments must behave consistently across all commands so the surface is predictable for both interactive use and automation.

These conventions apply across the entire CLI unless a command explicitly documents an exception.

---

## General Rules

### Long flags are the default

Use long-form flags for all public CLI options.

Examples:

```text
--session
--name
--non-interactive
```

Short flags are not part of the current spec milestone unless a future case is both standard and unambiguous.

---

### Use kebab-case for flags

All public flag names must use lowercase kebab-case.

Examples:

```text
--non-interactive
--working-directory
--output-format
```

Do not use:

* camelCase
* PascalCase
* snake_case

---

### Positional arguments are allowed only for the primary target

A positional argument may be used only when it represents the primary subject of the command.

Examples:

* chat prompt
* session identifier or name for `sessions resume`
* document target for `docs show`

Do not use positional arguments for secondary or optional values when a flag would be clearer.

---

## Required vs Optional Arguments

### Required arguments

A required argument must be either:

* a required positional target, or
* a required flag when positional use would be unclear or ambiguous

Examples:

```text
llamarc42 docs show <path|document-id>
llamarc42 sessions resume <session-id|name>
```

If a required argument is missing, the command must fail explicitly with a usage error.

---

### Optional arguments

Optional arguments must use flags unless they represent the omission of a required positional target that changes mode intentionally.

Example:

* `llamarc42 chat` with no prompt enters interactive mode
* `llamarc42 sessions new --name architecture` uses an optional flag

---

## Positional Argument Rules

### Rule 1: one primary positional target per command

Commands should use at most one primary positional argument unless there is a very strong reason otherwise.

This keeps parsing simple and avoids ambiguity.

---

### Rule 2: positional values must have a single meaning

A positional argument may allow multiple value types only when they represent the same conceptual target.

Examples:

* `<session-id|name>` = session selector
* `<path|document-id>` = document selector

This is acceptable because the user is still identifying a single target.

---

### Rule 3: do not overload positional order

Do not make argument meaning depend on position among multiple free-form values.

Avoid patterns like:

```text
llamarc42 something <name> <path> <mode>
```

unless the meaning is extremely obvious and stable.

---

### Rule 4: prompts remain positional for `chat`

For `chat`, the prompt is the primary subject of one-shot use and should remain positional.

Example:

```text
llamarc42 chat "summarize the current session model"
```

This is more natural than requiring a flag like `--prompt`.

---

## Target Addressing Conventions

The CLI currently works with these target types:

* session IDs
* session names
* document paths
* document identifiers

These must be passed consistently.

---

### Session targets

When a command targets a session, it should accept:

```text
<session-id|name>
```

Examples:

```text
llamarc42 sessions resume architecture
llamarc42 sessions resume session-20260401-001
```

If a session must be specified by flag, use:

```text
--session <session-id>
--name <session-name>
```

Rules:

* `--session` means exact session identity
* `--name` means named session resolution or creation, depending on command semantics
* these two flags must not be combined unless a future command explicitly defines valid behavior

---

### Document targets

When a command targets a document, it should accept:

```text
<path|document-id>
```

Examples:

```text
llamarc42 docs show docs/projects/llamarc42/architecture/boundaries.md
llamarc42 docs show architecture-boundaries
```

Rules:

* path resolution should be explicit and deterministic
* document ID use is allowed only if the project defines stable identifiers

---

### Path arguments

Paths must be passed as plain positional or flagged string values.

Examples:

```text
llamarc42 docs show docs/global/principles.md
llamarc42 --working-directory ./examples/project-a
```

Rules:

* prefer relative paths where practical
* path handling must respect the current working directory or explicitly provided working directory
* path interpretation must be deterministic

---

## Flag Value Conventions

### Flags with values

Flags that require values must be followed by a value.

Examples:

```text
--name architecture
--session session-20260401-001
```

This spec does not require `=` syntax, though parsers may support it if implemented consistently.

Canonical examples should use space-separated values.

---

### Boolean flags

Boolean flags should be presence-based.

Examples:

```text
--non-interactive
```

Avoid requiring explicit boolean values like:

```text
--non-interactive true
```

unless a future command specifically requires tri-state behavior.

---

## Naming Conventions for Flags

Flag names should:

* be descriptive
* use stable terminology already present in the CLI
* avoid abbreviations
* avoid PowerShell-style parameter casing
* avoid synonyms when one canonical term exists

Prefer:

```text
--session
--name
--non-interactive
```

Avoid:

```text
--sess
--session-name
--nointeractive
```

when a shorter consistent canonical term already exists.

---

## Consistency Rules

### Same concept, same argument form

If two commands refer to the same concept, they should use the same name and shape.

Examples:

* session targeting should consistently use `--session` and `--name`
* interactive suppression should consistently use `--non-interactive`

---

### Do not rename concepts across commands

Do not vary between equivalent names such as:

* `--session-id` vs `--session`
* `--doc` vs `--document`
* `--cwd` vs `--working-directory`

Choose one canonical form and use it everywhere.

---

## Error Expectations

Argument-related errors must be:

* explicit
* actionable
* deterministic

Examples:

```text
Error: missing required argument '<session-id|name>'.
```

```text
Error: '--session' and '--name' cannot be used together.
```

```text
Error: unknown option '--session-name'.
Did you mean '--session'?
```

---

## Examples

### Positional prompt

```text
llamarc42 chat "summarize current docs"
```

### Optional flag

```text
llamarc42 sessions new --name architecture
```

### Session target

```text
llamarc42 sessions resume architecture
```

### Document target

```text
llamarc42 docs show docs/projects/llamarc42/architecture/boundaries.md
```

### Boolean flag

```text
llamarc42 chat --non-interactive "summarize the project"
```

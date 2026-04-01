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

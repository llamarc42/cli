# CLI Specification — llamarc42

## Status

Draft — in progress

---

## Scope and Deferred Workflows

This specification defines the initial CLI surface for llamarc42 based on current, well-understood workflows.

The CLI is intentionally **narrow in scope** for this milestone.

The following capabilities are **deferred or absorbed** and are not exposed as first-class commands:

* review workflows → currently handled via `chat`
* retrieval/context inspection → surfaced via verbosity (`--verbosity detailed|diagnostic`)
* validation workflows → deferred until the canonical project model is defined in Core
* scaffolding/bootstrap workflows → deferred until project structure and templates are formalized

These areas will be revisited in a future milestone once:

* Core behavior is defined and stable
* PowerShell and CLI surfaces are aligned
* architectural boundaries are validated

This approach ensures the CLI evolves existing capabilities rather than introducing new behavior prematurely.

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

# 7. Shared Flags

## Purpose

Shared flags provide consistent behavior across all CLI commands.

These flags must:

* have **stable meaning across all commands**
* behave consistently in interactive and non-interactive contexts
* be safe for automation and scripting

No command may redefine or alter the meaning of a shared flag.

---

## Supported Shared Flags

```text
--output <plain|json|table>
--verbosity <quiet|normal|detailed|diagnostic>
--working-directory <path>
--non-interactive
--no-color
```

---

## `--output`

Controls the format of command output.

### Supported values

```text
plain
json
table
```

---

### Behavior

#### `plain`

* default human-readable output
* concise and structured
* suitable for terminal usage

---

#### `json`

* strictly machine-readable
* must be valid JSON
* must contain **no extra prose or formatting**
* errors must also be returned as JSON

---

#### `table`

* human-readable tabular output
* used for list-style commands (e.g., `sessions list`, `docs list`)
* must not be used for deeply nested or complex data

---

### Rules

* commands that cannot logically produce tabular output must fail if `table` is requested
* `json` output must be stable for scripting and automation
* default output is `plain` unless otherwise specified

---

## `--verbosity`

Controls the level of detail included in output.

### Supported values

```text
quiet
normal
detailed
diagnostic
```

---

### Behavior

#### `quiet`

* minimal output
* only final result or critical errors

---

#### `normal`

* default behavior
* user-friendly output

---

#### `detailed`

* includes additional context
* may include:

  * selected session info
  * resolved targets
  * high-level processing steps

---

#### `diagnostic`

* includes deep troubleshooting information
* may include:

  * context selection details
  * resolution decisions
  * internal workflow boundaries

---

### Rules

* verbosity must not alter the **meaning** of output, only its **detail level**
* verbosity must not break structured output (`json`)
* diagnostic output must not expose hidden reasoning or internal chain-of-thought

---

## `--working-directory`

Overrides the current working directory used by the CLI.

```text
--working-directory <path>
```

---

### Behavior

* all relative paths are resolved from this directory
* project detection is performed relative to this directory
* must behave as if the CLI was executed from the specified path

---

### Rules

* path must exist
* path must be accessible
* failure to resolve path must produce an error

---

### Example

```text
llamarc42 --working-directory ./examples/project-a docs list
```

---

## `--non-interactive`

Forces automation-safe behavior.

---

### Behavior

* disables all interactive prompts
* disables REPL entry
* requires all required inputs to be provided explicitly
* ambiguity must result in failure

---

### Rules

* must not prompt under any condition
* must not fallback to interactive behavior
* must produce deterministic results

---

### Example

```text
llamarc42 chat --non-interactive "summarize docs"
```

---

## `--no-color`

Disables colored output.

---

### Behavior

* all output must be plain text with no ANSI color codes
* applies to all output modes except `json` (which must already be color-free)

---

### Rules

* must not affect output content, only presentation
* must be respected across all commands

---

## Invalid Combinations

### `--output json` with interactive behavior

If a command would enter interactive mode:

```text
llamarc42 chat --output json
```

Behavior:

* must fail
* must not enter REPL

Reason:

* JSON output must be single-response and machine-readable

---

### `--non-interactive` with missing required input

Example:

```text
llamarc42 chat --non-interactive
```

Behavior:

* must fail
* must not prompt

---

### `--output table` on non-tabular commands

Example:

```text
llamarc42 chat --output table "summarize docs"
```

Behavior:

* must fail with error
* must explain unsupported format

---

### Unknown flag combinations

Any unsupported combination must:

* fail explicitly
* provide actionable error message

---

## Command-Specific Exceptions

Commands may restrict output formats where appropriate.

Examples:

* `chat`:

  * supports: `plain`, `json`
  * does not support: `table`

* `sessions list`:

  * supports: `plain`, `json`, `table`

* `docs show`:

  * supports: `plain`, `json`
  * does not support: `table`

These restrictions must be documented in the respective command sections.

---

## Error Expectations

Errors related to flags must be:

* explicit
* actionable
* consistent

Examples:

```text
Error: '--output table' is not supported for command 'chat'.
```

```text
Error: interactive mode is not allowed when '--output json' is specified.
```

```text
Error: '--working-directory' path './missing' does not exist.
```

---

## Examples

### JSON output

```text
llamarc42 sessions list --output json
```

---

### Diagnostic verbosity

```text
llamarc42 chat --verbosity diagnostic "explain session resolution"
```

---

### Working directory override

```text
llamarc42 --working-directory ./project-a docs list
```

---

### Non-interactive execution

```text
llamarc42 chat --non-interactive "summarize docs"
```

---

### Disable color

```text
llamarc42 docs list --no-color
```

# 8. Output Contracts

## Purpose

Output contracts define how the CLI communicates results in both human-readable and machine-readable forms.

These contracts must remain stable enough for:

* interactive terminal use
* automation and scripting
* tests and golden examples
* future compatibility across surfaces

Output mode changes presentation, not command meaning.

---

## Supported Output Modes

```text
plain
table
json
```

These are selected using:

```text
--output <plain|table|json>
```

If `--output` is omitted, the command uses its default output mode.

---

## Core Rules

### Rule 1: output mode changes format, not behavior

The selected output mode must not change what the command does.

It may change:

* formatting
* structure
* level of presentation detail

It must not change:

* target resolution
* validation result
* session selection behavior
* success vs failure outcome

---

### Rule 2: `json` is authoritative for automation

`json` output is the machine-safe format.

When `--output json` is used:

* output must be valid JSON
* output must contain no prose outside the JSON document
* logs or diagnostics must not corrupt stdout JSON output
* errors must also be returned as structured JSON

---

### Rule 3: `table` is for list-oriented human output

`table` is only valid when the command returns row-like data.

It should be used for:

* lists of sessions
* lists of documents

It should not be used for:

* free-form chat responses
* deeply nested objects
* single-artifact content display

---

### Rule 4: `plain` is the default human-readable mode

`plain` is the default mode unless a command explicitly defines a different default.

`plain` should be:

* readable in a terminal
* concise
* structured enough to scan quickly
* suitable for copy/paste into notes or docs

---

## `plain` Output

## Purpose

Human-readable terminal output for everyday use.

## Characteristics

* concise
* readable without requiring parsing
* structured using headings, labels, spacing, or bullets where helpful
* may include explanatory text appropriate for direct human consumption

## Use cases

* `chat`
* `sessions show`
* `docs show`
* `project show`

## Rules

* should avoid excessive decoration
* should not rely on color alone to convey meaning
* should remain understandable when redirected to a file

---

## `table` Output

## Purpose

Human-readable tabular output for list-style commands.

## Characteristics

* column-based
* easy to scan
* optimized for comparing multiple items

## Use cases

* `sessions list`
* `docs list`

## Rules

* must use stable column meanings for a given command
* columns may evolve only deliberately and with care
* should not be used when the command returns a single object or free-form text
* if the command does not support tabular output, requesting `table` must fail explicitly

## Minimum expectation

Each tabular command should define:

* its core columns
* the meaning of those columns
* whether optional columns may appear under higher verbosity

---

## `json` Output

## Purpose

Machine-readable output for scripting, testing, and integration.

## Characteristics

* valid JSON only
* deterministic structure
* no extra prose
* no ANSI color
* safe for piping to other tools

## Rules

* output must be a single JSON document
* field names should be stable once published
* object structure should be intentional and versionable
* `json` mode must remain usable even when verbosity changes
* if diagnostic information is emitted, it must not corrupt stdout JSON output

## Recommended shape

Command output in JSON mode should generally be shaped as:

```json
{
  "command": "sessions list",
  "status": "success",
  "data": {}
}
```

For list commands:

```json
{
  "command": "docs list",
  "status": "success",
  "data": {
    "items": []
  }
}
```

For single-target commands:

```json
{
  "command": "project show",
  "status": "success",
  "data": {
    "project": {}
  }
}
```

This is a recommended consistency pattern for the CLI as a whole.

---

## Default Output by Command Type

### Conversational commands

Examples:

* `chat`

Default output:

* `plain`

Supported output:

* `plain`
* `json`

Not supported:

* `table`

---

### List commands

Examples:

* `sessions list`
* `docs list`

Default output:

* `table`

Supported output:

* `plain`
* `table`
* `json`

---

### Single-artifact inspection commands

Examples:

* `sessions show`
* `docs show`
* `project show`

Default output:

* `plain`

Supported output:

* `plain`
* `json`

Not supported:

* `table`

---

## Structured Error Output in `json` Mode

When `--output json` is used and the command fails, the CLI must return a structured JSON error object.

### Rules

* no extra prose outside JSON
* error output must be valid JSON
* error object must be stable enough for automation
* command failures must still use non-zero exit codes

### Recommended shape

```json
{
  "command": "sessions resume",
  "status": "error",
  "error": {
    "code": "session_ambiguous",
    "message": "Session name 'architecture' is ambiguous.",
    "details": {
      "matches": [
        {
          "id": "session-20260401-001",
          "name": "architecture"
        },
        {
          "id": "session-20260329-004",
          "name": "architecture"
        }
      ]
    },
    "suggestion": "Re-run with a session ID."
  }
}
```

### Error fields

Recommended common fields:

* `code` — stable machine-readable error code
* `message` — human-readable summary
* `details` — structured context where appropriate
* `suggestion` — actionable remediation guidance where appropriate

---

## Output Stability Guidance

### Human-readable modes

`plain` and `table` should be stable enough for users, but may evolve for clarity over time.

They are not the preferred parsing surface for automation.

### Machine-readable mode

`json` is the preferred automation contract and should be treated as the stable integration surface.

Any breaking change to JSON structure should be deliberate and documented.

---

## Command Support Matrix

| Command              | Default | Supported                |
| -------------------- | ------- | ------------------------ |
| `chat`               | `plain` | `plain`, `json`          |
| `sessions list`      | `table` | `plain`, `table`, `json` |
| `sessions new`       | `plain` | `plain`, `json`          |
| `sessions resume`    | `plain` | `plain`, `json`          |
| `sessions show`      | `plain` | `plain`, `json`          |
| `sessions summarize` | `plain` | `plain`, `json`          |
| `docs list`          | `table` | `plain`, `table`, `json` |
| `docs show`          | `plain` | `plain`, `json`          |
| `project show`       | `plain` | `plain`, `json`          |

---

## Canonical Examples

### Plain output example

```text
Session: architecture
Id: session-20260401-001
Last Active: 2026-04-01T09:12:00
Summary: Working through CLI spec issues.
```

---

### Table output example

```text
ID                    NAME           LAST ACTIVE
session-20260401-001  architecture   2026-04-01T09:12:00
session-20260329-004  docs           2026-03-29T16:48:00
```

---

### JSON success example

```json
{
  "command": "docs list",
  "status": "success",
  "data": {
    "items": [
      {
        "path": "global/principles.md",
        "exists": true
      },
      {
        "path": "projects/llamarc42/architecture/boundaries.md",
        "exists": true
      }
    ]
  }
}
```

---

### JSON error example

```json
{
  "command": "docs show",
  "status": "error",
  "error": {
    "code": "document_not_found",
    "message": "Document 'architecture.md' was not found.",
    "details": {},
    "suggestion": "Use 'llamarc42 docs list' to inspect available documents."
  }
}
```

---

## Invalid Output Requests

If a user requests an unsupported output mode for a command, the command must fail explicitly.

Example:

```text
Error: '--output table' is not supported for command 'chat'.
```

JSON mode equivalent:

```json
{
  "command": "chat",
  "status": "error",
  "error": {
    "code": "unsupported_output_mode",
    "message": "'table' output is not supported for command 'chat'.",
    "details": {
      "requested": "table",
      "supported": ["plain", "json"]
    },
    "suggestion": "Use '--output plain' or '--output json'."
  }
}
```

# 9. Interactivity Model

## Purpose

The CLI must behave predictably in both:

* **interactive terminal usage** (human-driven workflows)
* **non-interactive execution** (automation, scripts, pipelines)

This section defines how the CLI detects and responds to these contexts.

---

## Mode Definitions

### Interactive Mode

A command is considered **interactive** when:

* stdin is attached to a terminal
* stdout is attached to a terminal
* `--non-interactive` is **not** specified

---

### Non-Interactive Mode

A command is considered **non-interactive** when any of the following are true:

* stdin is redirected or piped
* stdout is redirected or piped
* `--non-interactive` is specified

---

## Mode Detection Rules

### Rule 1: Explicit override takes precedence

```text
--non-interactive
```

Always forces non-interactive behavior regardless of terminal state.

---

### Rule 2: I/O redirection implies non-interactive

If either:

* stdin is piped
* stdout is redirected

The CLI must treat the command as non-interactive.

---

### Rule 3: Terminal presence enables interactive mode

Interactive mode is only allowed when:

* stdin is a TTY
* stdout is a TTY
* no override flag is present

---

## Prompting Rules

### Interactive Mode

Prompting is allowed only in interactive mode.

The CLI may prompt for:

* ambiguity resolution (e.g., multiple session matches)
* selection from known options
* confirmation where explicitly defined (none currently in scope)

Prompting must:

* be deterministic
* present clear options
* allow user cancellation

---

### Non-Interactive Mode

Prompting is **never allowed**.

If user input is required:

* the command must fail
* the error must explain what input is missing

---

## Behavior When Input is Missing

### Interactive Mode

If required input is missing and the command supports interactive fallback:

* the CLI may prompt the user
* or enter an interactive workflow (e.g., `chat`)

Example:

```text
llamarc42 chat
```

→ enters interactive chat session

---

### Non-Interactive Mode

If required input is missing:

* the command must fail
* the command must not attempt fallback
* the error must be actionable

Example:

```text
llamarc42 chat --non-interactive
```

Result:

```text
Error: no prompt provided and interactive mode is disabled.
```

---

## Behavior with Piped Input

If stdin is piped:

* the command must operate in **one-shot mode**
* the command must not enter interactive mode
* the command must not prompt

Examples:

```text
git diff | llamarc42 chat
Get-Content file.md | llamarc42 chat
```

Behavior:

* input is consumed once
* output is produced once
* command exits

---

## Behavior with Redirected Output

If stdout is redirected:

* the command must operate in non-interactive mode
* the command must not prompt
* output must remain valid for the selected format

Example:

```text
llamarc42 sessions list > sessions.txt
```

---

## Ambiguity Handling

### Interactive Mode

* display matching results
* allow user selection

---

### Non-Interactive Mode

* display matching results
* fail explicitly
* do not prompt

---

## Relationship to Output Modes

### JSON Output

When:

```text
--output json
```

is specified:

* the command must behave as non-interactive
* the command must not prompt
* output must be a single valid JSON document

---

## Command Behavior Summary

| Condition           | Mode            | Behavior          |
| ------------------- | --------------- | ----------------- |
| TTY + no flags      | interactive     | prompts allowed   |
| stdin piped         | non-interactive | one-shot          |
| stdout redirected   | non-interactive | no prompts        |
| `--non-interactive` | non-interactive | no prompts        |
| `--output json`     | non-interactive | structured output |

---

## Error Handling Expectations

Errors in non-interactive mode must:

* not require user input
* include clear remediation steps
* be deterministic

Example:

```text
Error: session name 'architecture' is ambiguous.
Matches:
- session-001
- session-002

Re-run with a session ID.
```

---

## Examples

### Interactive usage

```text
llamarc42 chat
```

* enters interactive session
* allows prompts and selections

---

### One-shot usage

```text
llamarc42 chat "summarize docs"
```

* processes once
* exits

---

### Piped usage

```text
git diff | llamarc42 chat
```

* one-shot
* no prompts

---

### Non-interactive flag

```text
llamarc42 chat --non-interactive "summarize docs"
```

* no prompts
* deterministic output

---

### Redirected output

```text
llamarc42 sessions list > sessions.txt
```

* no prompts
* plain output written to file

---

### JSON automation

```text
llamarc42 sessions list --output json
```

* machine-readable
* no prompts

# 10. Ambiguity Handling

## Purpose

Ambiguity is expected in a documentation-driven and session-driven CLI.

The CLI must handle ambiguity consistently across commands so that:

* interactive use remains efficient
* automation remains deterministic
* users receive actionable guidance

Ambiguity handling applies to any command target that may resolve to multiple matches, including:

* session names
* document identifiers
* document paths or shorthand references
* future named targets introduced by the CLI

---

## Core Rule

If a user-supplied target resolves to more than one valid match, the CLI must treat the result as ambiguous.

The CLI must never silently choose one match unless a command explicitly documents deterministic precedence.

---

## Interactive Behavior

In interactive mode, the CLI may support disambiguation by:

* displaying the matching results
* allowing the user to select one match interactively

Interactive disambiguation must:

* show enough information for the user to distinguish matches
* use a deterministic presentation order
* allow the user to cancel

If the user cancels, the command must exit without applying a selection.

---

## Non-Interactive Behavior

In non-interactive mode, the CLI must not prompt.

If ambiguity occurs, the command must:

* display the matching results
* fail explicitly
* explain how to disambiguate
* return a non-zero exit code

The CLI must never guess in automation contexts.

---

## Ambiguous Session Handling

When a session name matches multiple sessions:

### Interactive mode

* display matching sessions
* may allow the user to select one

### Non-interactive mode

* display matching sessions
* fail explicitly
* instruct the user to re-run with a session ID

Example:

```text
Error: session name 'architecture' is ambiguous.

Matches:
- session-20260401-001  architecture  last active: 2026-04-01T09:12:00
- session-20260329-004  architecture  last active: 2026-03-29T16:48:00

Re-run with a session ID.
```

---

## Ambiguous Document Handling

When a document target matches multiple documents:

### Interactive mode

* display matching documents
* may allow the user to select one

### Non-interactive mode

* display matching documents
* fail explicitly
* instruct the user to re-run with a full path or stable document identifier

Example:

```text
Error: document target 'architecture' is ambiguous.

Matches:
- docs/projects/llamarc42/architecture/overview.md
- docs/projects/llamarc42/architecture/boundaries.md

Re-run with a full path or document identifier.
```

---

## Display Requirements for Matches

When ambiguity occurs, the CLI should display enough metadata to help the user make a correct choice.

### Session matches should include

* session ID
* session name
* last activity timestamp

### Document matches should include

* relative path
* document identifier, if available
* scope/category, if available

---

## Ordering Rules

When multiple matches are displayed, ordering should be deterministic.

Recommended ordering:

### Sessions

* most recently active first

### Documents

* lexical path order unless a stronger command-specific rule is defined

---

## Error Messaging Requirements

Ambiguity errors must be:

* explicit
* actionable
* deterministic

They must include:

* the target the user provided
* that the result was ambiguous
* the matching candidates
* how to re-run unambiguously

---

## JSON Error Shape

In `--output json` mode, ambiguity must be returned as structured JSON.

Example:

```json
{
  "command": "sessions resume",
  "status": "error",
  "error": {
    "code": "ambiguous_target",
    "message": "Session name 'architecture' is ambiguous.",
    "details": {
      "target": "architecture",
      "target_type": "session",
      "matches": [
        {
          "id": "session-20260401-001",
          "name": "architecture"
        },
        {
          "id": "session-20260329-004",
          "name": "architecture"
        }
      ]
    },
    "suggestion": "Re-run with a session ID."
  }
}
```

# 11. Error Model and Exit Codes

## Purpose

The CLI must report failures in a way that is:

* explicit for humans
* actionable for developers
* stable for automation
* consistent across commands

Errors must help the user understand:

* what failed
* why it failed
* what to do next

The CLI surface must not hide or silently swallow failures originating in Core.

---

## Error Design Principles

### Explicit

Errors must clearly state that the command failed.

---

### Actionable

Errors should include remediation guidance when reasonable.

---

### Deterministic

The same failure condition should produce the same error category and exit code.

---

### Surface-safe

The CLI may translate internal failures into user-facing errors, but must not invent misleading causes or discard important context.

---

## Error Categories

### 1. Usage Error (`usage`)

The command was invoked incorrectly.

Examples:

* missing required argument
* unsupported flag combination
* unknown option
* unsupported output mode

---

### 2. Not Found Error (`not_found`)

A required target could not be resolved.

Examples:

* session not found
* document not found
* working directory not found

---

### 3. Ambiguity Error (`ambiguity`)

A target resolved to multiple valid matches.

(See **Section 10 — Ambiguity Handling**)

---

### 4. Validation Error (`validation`)

The operation completed, but the subject is not valid.

Defined now for future use (validate commands are currently deferred).

---

### 5. Execution Error (`execution`)

The command failed during operation.

Examples:

* unreadable file
* storage failure
* serialization failure

---

### 6. Internal Error (`internal`)

Unexpected failure not attributable to user input.

Examples:

* unhandled Core exception
* invariant violation

---

## Human-Readable Error Style

### Format

```text
Error: <summary>

<optional details>

<optional suggestion>
```

---

## Examples

### Usage error

```text
Error: '--session' and '--name' cannot be used together.

Use '--session' or '--name', not both.
```

---

### Not found

```text
Error: session 'architecture' was not found.

Use 'llamarc42 sessions list' to inspect available sessions.
```

---

### Ambiguity

```text
Error: session name 'architecture' is ambiguous.

Matches:
- session-20260401-001  architecture
- session-20260329-004  architecture

Re-run with a session ID.
```

---

### Execution error

```text
Error: could not read document 'docs/global/principles.md'.

Check file permissions and try again.
```

---

## JSON Error Output

When `--output json` is used:

* output must be valid JSON
* no prose outside JSON
* errors must use structured format
* exit codes still apply

---

## JSON Error Shape

```json
{
  "command": "sessions resume",
  "status": "error",
  "error": {
    "code": "ambiguous_target",
    "category": "ambiguity",
    "message": "Session name 'architecture' is ambiguous.",
    "details": {},
    "suggestion": "Re-run with a session ID."
  }
}
```

---

## JSON Error Fields

| Field        | Description                        |
| ------------ | ---------------------------------- |
| `code`       | stable machine-readable identifier |
| `category`   | error category                     |
| `message`    | human-readable explanation         |
| `details`    | structured context                 |
| `suggestion` | remediation guidance               |

---

## Exit Codes

| Code | Category   | Meaning                        |
| ---- | ---------- | ------------------------------ |
| 0    | success    | command completed successfully |
| 2    | usage      | invalid invocation             |
| 3    | not_found  | target not found               |
| 4    | ambiguity  | multiple matches               |
| 5    | validation | validation failed              |
| 6    | execution  | runtime failure                |
| 7    | internal   | unexpected failure             |

---

## Exit Code Rules

### Success

* return `0`

---

### Usage

* return `2`

---

### Not found

* return `3`

---

### Ambiguity

* return `4`

---

### Validation

* return `5`

---

### Execution

* return `6`

---

### Internal

* return `7`

---

## Core Error Translation

The CLI must map Core errors to CLI categories without losing meaning.

### Examples

| Core Condition       | CLI Category | Exit |
| -------------------- | ------------ | ---- |
| session not found    | not_found    | 3    |
| multiple matches     | ambiguity    | 4    |
| file read failure    | execution    | 6    |
| unexpected exception | internal     | 7    |

---

## Output Rules

### Human-readable modes

* readable
* actionable
* may include suggestions

---

### JSON mode

* machine-safe
* no extra text
* stable structure

# 12. Verbosity and Diagnostic Behavior

## Purpose

Verbosity controls how much supporting information the CLI surfaces alongside the primary command result.

It exists to support:

* concise everyday usage
* deeper troubleshooting
* explainability of command behavior
* inspectability of retrieval-driven workflows

Verbosity must not change command meaning or result. It changes only how much supporting information is surfaced.

---

## Supported Verbosity Levels

```text
quiet
normal
detailed
diagnostic
```

These are selected using:

```text
--verbosity <quiet|normal|detailed|diagnostic>
```

If `--verbosity` is omitted, the command uses `normal`.

---

## Core Rules

### Rule 1: verbosity changes detail, not behavior

Verbosity must not change:

* target resolution
* session selection behavior
* success vs failure outcome
* output mode semantics
* validation result
* retrieval result selection logic

It may change:

* how much supporting context is shown
* whether intermediate decisions are surfaced
* how much operational detail is displayed

---

### Rule 2: structured output must remain clean

When `--output json` is used:

* stdout must remain valid JSON
* verbosity must not inject prose or logs into stdout
* any auxiliary diagnostic information must not corrupt the structured result

---

### Rule 3: verbosity must respect command scope

Diagnostic output may explain how a command behaved, but it must not expose:

* hidden chain-of-thought
* private reasoning traces
* internal speculative analysis
* implementation details that violate architectural boundaries

The CLI may explain **what it did** and **why a visible decision occurred**, but not expose hidden reasoning internals.

---

## Verbosity Levels

## `quiet`

### Purpose

Minimal output for scripts or users who only want the final result.

### Behavior

* suppress non-essential informational output
* show only:

  * final result, or
  * fatal error output

### Use cases

* automation-friendly human usage
* reduced-noise terminal workflows

### Notes

`quiet` must not suppress structured error output in `json` mode.

---

## `normal`

### Purpose

Default user-facing behavior.

### Behavior

* show the primary result in a readable way
* include only the level of context needed to understand the result

### Use cases

* standard daily CLI usage
* default terminal interaction

---

## `detailed`

### Purpose

Show additional supporting information useful for understanding command behavior.

### Behavior

May include:

* resolved target information
* selected session identity
* relevant file or path resolution details
* high-level command steps
* additional metadata associated with the result

### Use cases

* troubleshooting command usage
* understanding what the CLI resolved or selected

### Examples

For `sessions resume`, detailed output may include:

* whether resolution matched by ID or by name
* which session was resumed

For `project show`, detailed output may include:

* resolved project root
* documentation root
* discovered markers used to identify the project

---

## `diagnostic`

### Purpose

Expose maximum safe troubleshooting detail for advanced inspection and explainability.

### Behavior

May include:

* command resolution steps
* target matching details
* ambiguity candidate sets
* project/document discovery details
* retrieval-context selection summary
* why visible artifacts were included in the effective context
* high-level explanation of surfaced command decisions

### Use cases

* debugging CLI behavior
* retrieval explainability
* inspecting why a command selected certain visible inputs

### Important boundary

`diagnostic` is **not** permission to expose hidden reasoning.

It may include:

* “selected documents A and B because they matched the project scope and command target”
* “session name matched two sessions; command entered ambiguity flow”

It must not include:

* hidden internal reasoning text
* raw chain-of-thought
* speculative private deliberation
* internal-only prompts or hidden model scaffolding

---

## Safe Diagnostic Explainability

Because llamarc42 is retrieval-based and documentation-driven, diagnostic output may provide bounded explainability.

This may include:

* what visible artifacts were selected
* what visible rules were applied
* what command-path decisions were taken
* why an ambiguity or failure occurred
* whether context was built fresh or reused, if the command exposes such a concept later

This must remain:

* deterministic
* inspectable
* artifact-grounded
* free of hidden reasoning traces

---

## Stdout vs Stderr Expectations

## Stdout

Stdout is the primary result channel.

It should contain:

* normal command results
* human-readable output in `plain` or `table` mode
* structured result objects in `json` mode

---

## Stderr

Stderr is the auxiliary diagnostics and error channel.

It may contain:

* human-readable errors
* warnings
* diagnostic notes
* verbose operational messages, where appropriate

### Rule for JSON mode

When `--output json` is used:

* stdout must contain only the JSON result or JSON error
* any non-JSON diagnostics must not be written to stdout
* if diagnostics are emitted separately, they must go to stderr

This preserves machine-safe parsing.

---

## Relationship Between Verbosity and Output Modes

### `plain`

All verbosity levels are allowed.

* `quiet` → minimal human-readable output
* `normal` → standard readable output
* `detailed` → extra supporting context
* `diagnostic` → max safe detail

---

### `table`

Supported only for commands that support table output.

Verbosity may affect:

* whether optional columns appear
* whether supplemental notes are shown outside the table

But it must not make tabular output confusing or unstable.

---

### `json`

All verbosity levels may be accepted, but:

* stdout JSON must remain valid
* verbosity must not alter the command result schema unpredictably
* diagnostic detail, if included, must be deliberately structured

Recommended approach:

* primary result remains under `data`
* verbosity-driven supporting metadata, if surfaced in JSON, should be placed under a predictable field such as `meta`

Example:

```json
{
  "command": "project show",
  "status": "success",
  "data": {
    "project": {
      "root": "/repo/llamarc42"
    }
  },
  "meta": {
    "verbosity": "diagnostic",
    "resolution": {
      "project_root_source": "working-directory"
    }
  }
}
```

If `meta` is omitted at lower verbosity levels, that omission must be intentional and documented as allowed.

---

## Allowed Diagnostic Content by Command Type

## Conversational commands (`chat`)

May include:

* selected session identity
* whether execution was one-shot or interactive
* whether stdin or positional prompt was used
* visible context artifact summary
* bounded explanation of visible retrieval selection

Must not include:

* hidden prompt internals
* hidden reasoning traces

---

## Session commands (`sessions`)

May include:

* name vs ID resolution details
* ambiguity candidate sets
* session metadata used in resolution

---

## Documentation commands (`docs`)

May include:

* document resolution path
* identifier match details
* discovery source for listed documents

---

## Project commands (`project`)

May include:

* project root discovery details
* detected project markers
* resolved documentation roots or related paths

---

## Warnings

Warnings are non-fatal signals and may be surfaced in `normal`, `detailed`, or `diagnostic` output where appropriate.

Examples:

* deprecated command form in a future version
* partial project discovery
* missing optional metadata

Warnings must not be used to hide real failures.

In `json` mode, warnings should be structured and included predictably, for example:

```json
{
  "command": "project show",
  "status": "success",
  "data": {},
  "warnings": [
    {
      "code": "partial_project_metadata",
      "message": "Project root was resolved, but no explicit project identifier was found."
    }
  ]
}
```

---

## Examples

## Normal output example

```text
$ llamarc42 sessions resume architecture

Resumed session: architecture
Id: session-20260401-001
```

---

## Diagnostic output example

```text
$ llamarc42 sessions resume architecture --verbosity diagnostic

Resolved target type: session
Resolution strategy: exact id, then name
Name match count: 2

Ambiguous session name 'architecture'

Matches:
- session-20260401-001  architecture  last active: 2026-04-01T09:12:00
- session-20260329-004  architecture  last active: 2026-03-29T16:48:00
```

---

## JSON diagnostic example

```json
{
  "command": "docs show",
  "status": "success",
  "data": {
    "document": {
      "path": "docs/projects/llamarc42/architecture/boundaries.md",
      "content": "..."
    }
  },
  "meta": {
    "verbosity": "diagnostic",
    "resolution": {
      "input": "boundaries",
      "match_strategy": "document_id"
    }
  }
}
```

# 13. Help System

## Purpose

The CLI help system provides built-in guidance for discovering commands, understanding usage, and learning expected argument and output patterns.

Help is part of the public CLI contract and must be:

* consistent across command groups
* readable in a terminal
* useful for both first-time and experienced users
* aligned with the documented command surface

Help must describe the CLI as specified, not expose internal implementation details.

---

## Help Entry Points

The CLI supports help through both explicit `help` commands and `--help`.

### Root help

```text
llamarc42 help
llamarc42 --help
```

These are equivalent.

---

### Command-group help

```text
llamarc42 <command> help
llamarc42 <command> --help
```

These are equivalent.

Examples:

```text
llamarc42 chat help
llamarc42 chat --help

llamarc42 sessions help
llamarc42 sessions --help
```

---

### Subcommand help

```text
llamarc42 <command> <subcommand> --help
```

Example:

```text
llamarc42 sessions resume --help
```

`<command> <subcommand> help` is optional for future support, but `--help` is the required canonical form at the subcommand level.

---

## Help Behavior

### Root help behavior

Root help must display:

* CLI name and purpose
* current top-level commands
* short description of each top-level command
* canonical usage patterns
* how to get more help for specific commands

---

### Command-group help behavior

Command-group help must display:

* command purpose
* available subcommands, if any
* supported arguments and flags
* shared flags relevant to the command
* examples for common workflows

---

### Subcommand help behavior

Subcommand help must display:

* exact usage syntax
* required and optional arguments
* supported shared flags and command-specific constraints
* behavior notes for interactive vs non-interactive use, if relevant
* examples

---

## Help Content Structure

Help output should follow a consistent structure.

### Recommended structure

```text
NAME
  <command name> - <short purpose>

USAGE
  <usage forms>

DESCRIPTION
  <short explanation>

COMMANDS
  <subcommands, if applicable>

ARGUMENTS
  <required positional arguments>

FLAGS
  <supported flags>

EXAMPLES
  <canonical examples>
```

Not every section is required for every command, but the ordering should remain consistent where sections are present.

---

## Required Help Content by Level

## Root level

Must include:

* CLI name
* short overall description
* top-level command list
* one-line description for each top-level command
* example of command-specific help usage

Example topics:

* `chat`
* `sessions`
* `docs`
* `project`
* `help`
* `version`

---

## Command-group level

Must include:

* command purpose
* subcommand list, where applicable
* usage summary
* examples
* supported shared flags relevant to that command

Examples:

* `chat` help should explain one-shot vs interactive use
* `sessions` help should list `list`, `new`, `resume`, `show`, `summarize`
* `docs` help should list `list`, `show`
* `project` help should list `show`

---

## Subcommand level

Must include:

* exact usage line
* description of target arguments
* output mode support, where constrained
* ambiguity behavior, where relevant
* examples

Examples:

* `sessions resume --help` should explain `<session-id|name>`
* `docs show --help` should explain `<path|document-id>`

---

## `help` Command Scope

The `help` command is an informational surface only.

It does not:

* validate the project
* inspect sessions
* resolve command targets
* perform command execution

It only explains the CLI surface.

---

## `--help` Behavior

When `--help` is present:

* the CLI must display help for the relevant command scope
* the CLI must not execute the normal command action
* the CLI should return success exit code unless help resolution itself fails

Examples:

```text
llamarc42 chat --help
```

Must show help for `chat`, not enter chat mode.

```text
llamarc42 sessions resume --help
```

Must show help for `sessions resume`, not attempt to resolve a session.

---

## Examples in Help Output

Examples are required in help output because the CLI is workflow-oriented.

Examples should be:

* short
* canonical
* realistic
* aligned with the current spec

Examples must not:

* use undocumented commands
* imply deferred workflows are active
* rely on implementation details not in the spec

---

## Example Help Expectations by Command

### `chat`

Should include examples for:

* one-shot prompt
* interactive mode
* named session
* explicit session
* piped input

Example commands:

```text
llamarc42 chat "summarize the current docs"
llamarc42 chat
llamarc42 chat --name architecture
git diff | llamarc42 chat
```

---

### `sessions`

Should include examples for:

* listing sessions
* creating a named session
* resuming by name
* showing a session

Example commands:

```text
llamarc42 sessions list
llamarc42 sessions new --name architecture
llamarc42 sessions resume architecture
llamarc42 sessions show session-20260401-001
```

---

### `docs`

Should include examples for:

* listing docs
* showing a document

Example commands:

```text
llamarc42 docs list
llamarc42 docs show docs/projects/llamarc42/architecture/boundaries.md
```

---

### `project`

Should include examples for:

* showing project context

Example commands:

```text
llamarc42 project show
```

---

## Help Output and Formatting Rules

### Default format

Help output is human-readable plain text.

### Output modes

Help does not currently require `table` or `json` output.

For this milestone, help is specified as a human-oriented terminal surface.

If structured help is introduced later, it should be treated as a separate capability.

### Color

Help output may use color in interactive terminals if color is supported, but it must remain readable without color and respect `--no-color`.

---

## Error Handling for Help

Help lookup failures should be rare, but must still be explicit.

Examples:

```text
Error: unknown command 'sesions'.

Did you mean 'sessions'?
```

```text
Error: no help is available for subcommand 'project validate'.
This command is deferred and not part of the current CLI surface.
```

---

## Example Root Help Structure

```text
NAME
  llamarc42 - local-first documentation-driven AI CLI

USAGE
  llamarc42 <command> [options]
  llamarc42 help
  llamarc42 --help

COMMANDS
  chat       Start one-shot or interactive conversation workflows
  sessions   Manage persisted conversation sessions
  docs       Inspect project documentation
  project    Inspect current project context
  help       Show CLI help
  version    Show CLI version

EXAMPLES
  llamarc42 chat "summarize the current docs"
  llamarc42 sessions list
  llamarc42 docs list
  llamarc42 project show
  llamarc42 chat --help
```

---

## Example Command Help Structure

### `llamarc42 sessions --help`

```text
NAME
  sessions - manage persisted conversation sessions

USAGE
  llamarc42 sessions list
  llamarc42 sessions new [--name <name>]
  llamarc42 sessions resume <session-id|name>
  llamarc42 sessions show <session-id|name>
  llamarc42 sessions summarize <session-id|name>

DESCRIPTION
  Manage session lifecycle and continuity for local project conversations.

COMMANDS
  list        List sessions
  new         Create a new session
  resume      Resume a session in interactive chat
  show        Show session metadata and summary
  summarize   Generate or display a session summary

EXAMPLES
  llamarc42 sessions list
  llamarc42 sessions new --name architecture
  llamarc42 sessions resume architecture
```

---

## Example Subcommand Help Structure

### `llamarc42 docs show --help`

```text
NAME
  docs show - display a documentation artifact

USAGE
  llamarc42 docs show <path|document-id>

ARGUMENTS
  <path|document-id>   Document target to display

DESCRIPTION
  Show the contents of a project documentation artifact by path or stable identifier.

FLAGS
  --output <plain|json>
  --verbosity <quiet|normal|detailed|diagnostic>
  --working-directory <path>
  --no-color

EXAMPLES
  llamarc42 docs show docs/projects/llamarc42/architecture/boundaries.md
```

# 14. PowerShell-to-CLI Mapping

## Purpose

This section maps current PowerShell-oriented workflows to the future CLI surface.

The goal is to ensure the CLI:

* evolves existing capabilities rather than inventing a competing workflow model
* preserves architectural boundaries
* avoids leaking PowerShell verb-noun naming into the public CLI surface

This mapping is workflow-oriented, not cmdlet-oriented.

---

## Mapping Principles

### Workflow parity over cmdlet parity

The CLI should map to user workflows that already exist in practice, not mirror PowerShell command names directly.

### Core owns logic

If a PowerShell surface currently performs logic that belongs in Core, the CLI must not duplicate that logic. Both surfaces should converge on shared Core behavior over time.

### Defer where behavior is not yet formalized

If a PowerShell workflow is implied, ad hoc, or not yet formalized in Core, the CLI should defer or absorb it into an existing workflow rather than invent a new top-level command.

### No PowerShell naming leakage

The CLI must not expose public command names like verb-noun cmdlets. It should use workflow language such as `chat`, `sessions`, `docs`, and `project`.

---

## Current Workflow Mapping

| Current workflow / concept                          | CLI surface                                  | Status           | Notes                                                              |
| --------------------------------------------------- | -------------------------------------------- | ---------------- | ------------------------------------------------------------------ |
| Architectural planning in a chat-style workflow     | `llamarc42 chat`                             | mapped           | Primary one-shot or interactive conversation workflow              |
| Conversation continuity / prolonged working session | `llamarc42 sessions` + `llamarc42 chat`      | mapped           | Session lifecycle stays distinct from chat execution               |
| Inspect documentation artifacts                     | `llamarc42 docs list`, `llamarc42 docs show` | mapped           | Inspection only in current milestone                               |
| Inspect current project context                     | `llamarc42 project show`                     | mapped           | Inspection only in current milestone                               |
| Review / drift detection workflow                   | `llamarc42 chat`                             | absorbed for now | Deliberately not a first-class top-level command in this milestone |
| Validation of project/doc model                     | deferred                                     | deferred         | Depends on canonical Core project model                            |
| Scaffolding / project bootstrap                     | deferred                                     | deferred         | Depends on canonical Core project model                            |
| Retrieval/context explainability                    | `--verbosity diagnostic`                     | absorbed for now | No top-level `context` command in this milestone                   |

---

## Mapped Workflows

### 1. Architectural planning

The current documented planning workflow is a chat-style interaction grounded in global and project artifacts, including architecture and decisions. That maps directly to:

```text
llamarc42 chat
llamarc42 chat "<prompt>"
```

This is consistent with the documented “Architectural Planning (Chat UI)” scenario and its retrieval-first constraints.

---

### 2. Conversation continuity

The documented notes emphasize that continuity should come from retrieval rather than tool-specific memory, but the CLI still needs explicit session lifecycle behavior for local developer use. That maps to:

```text
llamarc42 sessions list
llamarc42 sessions new
llamarc42 sessions resume
llamarc42 sessions show
llamarc42 sessions summarize
```

and interactive execution remains under:

```text
llamarc42 chat
```

This preserves the boundary:

* `sessions` manages continuity
* `chat` performs execution

---

### 3. Documentation inspection

The project is documentation-driven and treats documentation artifacts as source of truth. That maps to:

```text
llamarc42 docs list
llamarc42 docs show <path|document-id>
```

This keeps the current milestone at inspection-only scope and avoids prematurely inventing validation or scaffolding behavior.

---

### 4. Project inspection

The CLI needs a lightweight way to show the currently resolved project context. That maps to:

```text
llamarc42 project show
```

This is intentionally narrow in the current milestone.

---

## Absorbed / Deferred Workflows

### Review / drift detection

The notes describe review/drift detection as a real workflow, but in the current CLI spec it should remain under `chat`, not a separate `review` top-level command. The documented scenario shows review as a prompt-driven workflow grounded in retrieved constraints, boundaries, ADRs, tradeoffs, and risks.

Current CLI treatment:

```text
llamarc42 chat "review this change ..."
```

Reason for deferral as a first-class command:

* not yet formalized as a stable Core workflow
* would risk creating new product behavior in the CLI surface
* should be revisited when Core and PowerShell are aligned

---

### Context / explainability

The notes make explainability important, especially around retrieval selection and bounded behavior. They also explicitly distinguish what artifacts are included or excluded by intent.

Current CLI treatment:

* no top-level `context` command
* explainability is surfaced through:

  * `--verbosity detailed`
  * `--verbosity diagnostic`

Reason:

* keeps retrieval inspectable without elevating internal concepts into the top-level command tree

---

### Validation and scaffolding

Commands such as:

* `llamarc42 docs validate`
* `llamarc42 docs new`
* `llamarc42 project validate`
* `llamarc42 project new`

remain deferred.

Reason:

* they depend on a canonical Core definition of project shape, required artifacts, and bootstrap rules
* specifying them now would force CLI-led architecture instead of Core-led architecture

---

## No Direct One-to-One Requirement

There is intentionally **no requirement** that every PowerShell function or script workflow map to a CLI command.

Some PowerShell behaviors may remain:

* internal implementation details
* transitional wrappers
* Core-facing helpers
* deferred workflows not yet ready for public CLI exposure

This is acceptable as long as:

* the public CLI surface stays coherent
* behavior is not duplicated across layers
* PowerShell and CLI converge on shared Core logic over time

---

## Surface vs Core Responsibilities

### CLI surface owns

* parsing commands and flags
* shaping user-facing input/output
* interactive vs non-interactive behavior
* presenting errors, ambiguity, help, and examples

### Core owns

* retrieval behavior
* session persistence logic
* project recognition/model rules
* future validation/scaffolding behavior
* any reusable workflow semantics shared across surfaces

### PowerShell surface

PowerShell should ultimately align with the same Core behaviors as the CLI, even if the current PowerShell shape differs.

---

## Intentionally Hidden or Deferred Implementation-Specific Concepts

The following are intentionally not exposed as first-class CLI workflows in this milestone:

* internal retrieval pipeline stages
* prompt construction internals
* storage implementation details
* model/provider selection mechanics
* PowerShell-specific helper names
* review as a top-level command
* context as a top-level command
* validation/scaffolding commands before Core formalization

---

## Examples

### Mapped planning workflow

```text
llamarc42 chat "Should we split the ingestion pipeline into a separate service?"
```

### Mapped continuity workflow

```text
llamarc42 sessions new --name architecture
llamarc42 chat --name architecture
```

### Mapped docs inspection workflow

```text
llamarc42 docs list
llamarc42 docs show docs/projects/llamarc42/architecture/boundaries.md
```

### Mapped project inspection workflow

```text
llamarc42 project show
```

### Review workflow absorbed into chat

```text
llamarc42 chat "Review this change for architectural drift."
```

# 15. Canonical Examples

## Purpose

Canonical examples demonstrate the intended CLI workflows end-to-end.

They serve as:

* guidance for users
* validation targets for implementers
* reference scenarios for testing

Examples must reflect the **current CLI contract only** and must not include deferred or undefined behavior.

---

## Chat Workflows

### One-shot usage

```text
llamarc42 chat "summarize the current documentation structure"
```

---

### Interactive session

```text
llamarc42 chat
```

---

### Named session workflow

```text
llamarc42 chat --name architecture
```

---

### Explicit session continuation

```text
llamarc42 chat --session session-20260401-001 "continue previous discussion"
```

---

### Piped input

```text
git diff | llamarc42 chat
```

---

### Non-interactive usage

```text
llamarc42 chat --non-interactive "summarize the project"
```

---

### JSON output

```text
llamarc42 chat --output json "summarize the project"
```

---

## Session Workflows

### List sessions

```text
llamarc42 sessions list
```

---

### Create a new session

```text
llamarc42 sessions new
llamarc42 sessions new --name architecture
```

---

### Resume a session

```text
llamarc42 sessions resume architecture
```

---

### Resume by ID

```text
llamarc42 sessions resume session-20260401-001
```

---

### Show session details

```text
llamarc42 sessions show architecture
```

---

### Summarize a session

```text
llamarc42 sessions summarize architecture
```

---

## Documentation Workflows

### List documentation

```text
llamarc42 docs list
```

---

### Show a document by path

```text
llamarc42 docs show docs/projects/llamarc42/architecture/boundaries.md
```

---

### Show a document by identifier

```text
llamarc42 docs show architecture-boundaries
```

---

## Project Workflows

### Show current project

```text
llamarc42 project show
```

---

### Use alternate working directory

```text
llamarc42 --working-directory ./examples/project-a project show
```

---

## Output and Automation Workflows

### JSON output for sessions

```text
llamarc42 sessions list --output json
```

---

### JSON output for docs

```text
llamarc42 docs list --output json
```

---

### Redirect output to file

```text
llamarc42 sessions list > sessions.txt
```

---

### Disable color

```text
llamarc42 docs list --no-color
```

---

## Verbosity Examples

### Detailed output

```text
llamarc42 project show --verbosity detailed
```

---

### Diagnostic output

```text
llamarc42 sessions resume architecture --verbosity diagnostic
```

---

## Ambiguity Handling Example

```text
llamarc42 sessions resume architecture
```

Result:

```text
Error: session name 'architecture' is ambiguous.

Matches:
- session-20260401-001  architecture
- session-20260329-004  architecture

Re-run with a session ID.
```

---

## Non-Interactive Failure Example

```text
llamarc42 chat --non-interactive
```

Result:

```text
Error: no prompt provided and interactive mode is disabled.
```

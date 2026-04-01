# CLI Specification — llamarc42

## Status

Draft — in progress (Issue 1–2 complete)

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
llamarc42 review
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

### `review`

Artifact-focused review workflows.

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

### A command is top-level only if

* represents a primary user workflow
* has a clear responsibility boundary
* is understandable without internal knowledge
* cannot fit under an existing command

---

### A command is NOT top-level if

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
llamarc42 review adr 001
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

* positional prompt is provided, OR
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

→ fail with error

---

## Input Handling

### Sources

1. positional argument
2. piped stdin

### Precedence

```text
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

```text
--session + --name → error
```

---

## Default Session Behavior

### One-shot (prompt or piped input)

* use **new isolated session context**
* do NOT reuse previous session

---

### Interactive mode

* resume last active session if exists
* otherwise create new session

---

## Ambiguity Handling

### Interactive

* may prompt user to select

### Non-interactive

* must fail
* must not prompt

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

Rules:

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

* no input
* terminal session
* resumes last session or creates new
* continuous interaction

---

## Errors

Must be:

* explicit
* actionable
* deterministic

---

## Examples

```text
llamarc42 chat "summarize session model"

llamarc42 chat --name architecture "compare ADR patterns"

llamarc42 chat --session session-001 "continue discussion"

llamarc42 chat

git diff | llamarc42 chat
```

---

## Current Coverage

✔ Command hierarchy defined
✔ Chat command fully specified

---

Here is your **updated `Sessions` section** with the refined ambiguity behavior integrated cleanly into the spec.

---

# 3. Sessions Command

## Purpose

`llamarc42 sessions` provides lifecycle management for persisted conversation sessions.

Sessions enable:

* conversation continuity
* resumable workflows
* structured history and summarization

This command group is responsible for **managing sessions**, not executing chat behavior.

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

* a **unique system-generated ID** (primary identifier)
* an optional **user-defined name** (not guaranteed unique)

---

## Identity Resolution Rules

### Session ID

* always unique
* resolves exactly one session
* preferred for automation and scripting

---

### Session Name

* may not be unique
* resolves:

  * exactly one → success
  * none → not found error
  * multiple → ambiguous

---

## Target Resolution Behavior

Commands accepting `<session-id|name>` must resolve targets using:

1. exact ID match
2. name match

---

### Resolution outcomes

| Condition             | Result    |
| --------------------- | --------- |
| exact ID match        | success   |
| single name match     | success   |
| no match              | error     |
| multiple name matches | ambiguous |

---

## Ambiguity Handling

If a session name matches multiple sessions, the CLI must surface the matching sessions.

### Interactive mode

* display matching sessions
* may allow the user to choose one interactively
* selection should be simple and deterministic (e.g., numbered list)

Example:

```text
Session name 'architecture' matched multiple sessions:

1. session-20260401-001  architecture  last active: 2026-04-01 09:12
2. session-20260329-004  architecture  last active: 2026-03-29 16:48

Select a session by number:
```

---

### Non-interactive mode

* display matching sessions
* must not prompt
* must fail with an ambiguity error
* must instruct the user to re-run with a session ID

Example:

```text
Error: session name 'architecture' is ambiguous.

Matches:
- session-20260401-001  architecture  last active: 2026-04-01 09:12
- session-20260329-004  architecture  last active: 2026-03-29 16:48

Re-run with a session ID:
llamarc42 sessions resume session-20260401-001
```

---

## Subcommands

---

### `sessions list`

Lists all known sessions for the current project.

#### Behavior

* returns all sessions
* includes:

  * session ID
  * name (if present)
  * created timestamp
  * last activity timestamp
  * summary (if available)

#### Output

* default: table
* supports: json, plain

---

### `sessions new`

Creates a new session.

```text
llamarc42 sessions new
llamarc42 sessions new --name architecture
```

#### Behavior

* creates a new session
* optionally assigns a name
* becomes the current session for subsequent interactive use

#### Rules

* name is optional
* name is not required to be unique

---

### `sessions resume`

Resumes a session and enters interactive chat mode.

```text
llamarc42 sessions resume <session-id|name>
```

#### Behavior

* resolves target session
* enters interactive chat mode
* behaves as if:

```text
llamarc42 chat --session <resolved-id>
```

#### Rules

* must resolve exactly one session
* ambiguity handled per standard rules

---

### `sessions show`

Displays session metadata and summary.

```text
llamarc42 sessions show <session-id|name>
```

#### Behavior

* resolves session
* displays:

  * ID
  * name
  * timestamps
  * summary (if available)
  * basic metadata

#### Does NOT:

* display full message history (future consideration)

---

### `sessions summarize`

Generates or displays a structured summary for a session.

```text
llamarc42 sessions summarize <session-id|name>
```

#### Behavior

* resolves session
* triggers summary generation if needed
* returns structured summary

#### Notes

* summary is:

  * lossy
  * structured
  * separate from raw message history

---

## Relationship to `chat`

* `sessions` manages session lifecycle
* `chat` performs conversational execution

### Equivalent behavior

```text
llamarc42 sessions resume <id>
```

is equivalent to:

```text
llamarc42 chat --session <id>
```

(with interactive mode)

---

## Interaction with Persistence

Sessions are:

* project-scoped
* stored locally
* persisted as append-only history (`messages.jsonl`)
* optionally accompanied by summary artifacts

The CLI must:

* not expose storage internals directly
* operate through defined session interfaces
* avoid modifying history outside defined workflows

---

## Error Conditions

### Not found

```text
Error: session 'xyz' was not found.
```

---

### Ambiguous

(see Ambiguity Handling section above)

---

### Invalid input

* missing required argument
* invalid combinations

---

## Examples

### List sessions

```text
llamarc42 sessions list
```

---

### Create session

```text
llamarc42 sessions new
llamarc42 sessions new --name architecture
```

---

### Resume session

```text
llamarc42 sessions resume architecture
llamarc42 sessions resume session-20260401-001
```

---

### Show session

```text
llamarc42 sessions show architecture
```

---

### Summarize session

```text
llamarc42 sessions summarize architecture
```

---

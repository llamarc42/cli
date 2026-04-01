# CLI Specification — llamarc42

## Status

Draft for milestone planning and implementation alignment.

## Purpose

Define the CLI surface for llamarc42 before implementation so behavior is explicit, testable, and aligned with the existing PowerShell and documentation-first architecture.

## Scope

This specification defines:

* command hierarchy
* command naming and argument conventions
* mapping from existing PowerShell capabilities to CLI commands
* output and formatting rules
* interactive vs non-interactive behavior
* error handling and exit code expectations
* verbosity and logging behavior

This specification does **not** define:

* internal implementation details beyond what is necessary to preserve architecture
* API transport details
* UI behavior
* future plugin systems unless explicitly required by current milestone

---

## Design Principles

1. **Documentation-first**

   * CLI behavior must reflect documented system concepts.
   * Commands should expose the documented architecture, not invent new abstractions.

2. **Local-first**

   * Default behavior should operate against local project files and local session state.
   * Network dependence must never be assumed.

3. **Deterministic and inspectable**

   * Commands that select context, sessions, or documents must provide explainable results.
   * Hidden behavior should be avoided.

4. **PowerShell parity without PowerShell leakage**

   * The CLI should map cleanly to existing PowerShell workflows where possible.
   * The CLI must not expose PowerShell-specific naming patterns such as verb-noun cmdlet names.

5. **Stable human UX, scriptable machine UX**

   * Human-readable output should be concise and helpful.
   * Structured output must be available for automation.

---

## Top-Level Command

Canonical executable:

```text
llamarc42
```

Optional short alias for later consideration:

```text
llx
```

For this specification, `llamarc42` is the canonical command name.

---

## Command Hierarchy

## Primary command groups

```text
llamarc42 chat
llamarc42 sessions
llamarc42 context
llamarc42 review
llamarc42 docs
llamarc42 project
llamarc42 version
llamarc42 help
```

## Proposed command tree

```text
llamarc42 chat [prompt]
llamarc42 chat --name <session-name> [prompt]
llamarc42 chat --session <session-id> [prompt]

llamarc42 sessions list
llamarc42 sessions new [--name <name>]
llamarc42 sessions resume <session-id|name>
llamarc42 sessions show <session-id|name>
llamarc42 sessions summarize <session-id|name>

llamarc42 context show
llamarc42 context list
llamarc42 context inspect <document-id|path>
llamarc42 context explain

llamarc42 review adr <adr-id|path>
llamarc42 review doc <path>
llamarc42 review changes

llamarc42 docs list
llamarc42 docs show <path>
llamarc42 docs validate

llamarc42 project show
llamarc42 project validate

llamarc42 version
llamarc42 help
```

---

## Command Semantics

### `chat`

Purpose: start or continue a conversational workflow using project context and session history.

Examples:

```text
llamarc42 chat "summarize the current retrieval design"
llamarc42 chat --name architecture "compare session approaches"
llamarc42 chat --session session-20260329-001 "continue"
```

Rules:

* If no session is specified, the CLI may create or select a default session according to documented session policy.
* If no prompt is provided and the terminal is interactive, enter interactive chat mode.
* If no prompt is provided and the terminal is non-interactive, return an error.
* `--name` requests a named session.
* `--session` targets an existing session explicitly.

### `sessions`

Purpose: inspect and manage persisted session state.

#### `sessions list`

List known sessions for the current project.

#### `sessions new`

Create a new session, optionally named.

#### `sessions resume`

Resume an existing session by identifier or unique name.

#### `sessions show`

Display session metadata and summary.

#### `sessions summarize`

Trigger or display structured summary output for a session, depending on final architecture.

### `context`

Purpose: inspect retrieval inputs and explain selected documentation.

#### `context show`

Show the context package that would be or was supplied to the model.

#### `context list`

List candidate documents and sources for the current project.

#### `context inspect`

Display a specific document included in retrieval.

#### `context explain`

Explain why specific documents were selected.

### `review`

Purpose: run targeted review workflows grounded in project docs.

#### `review adr`

Review an ADR for clarity, completeness, and architectural alignment.

#### `review doc`

Review a documentation file.

#### `review changes`

Review local changes against project guidance, if enabled by current architecture.

### `docs`

Purpose: inspect and validate project documentation.

#### `docs list`

List known project and global docs.

#### `docs show`

Print a specific document.

#### `docs validate`

Validate document structure and required files.

### `project`

Purpose: inspect current project configuration and health.

#### `project show`

Display current project metadata, paths, and detected configuration.

#### `project validate`

Validate that the project is correctly structured for llamarc42.

---

## Naming Rules

### Command naming

* Use lowercase command names.
* Use noun-based command groups and verb-like subcommands only where needed.
* Avoid aliases until the primary surface is stable.
* Avoid abbreviations in the primary command tree.

### Argument naming

* Use kebab-case for long flags.
* Prefer long flags over short flags unless the short flag is industry-standard and unambiguous.

Examples:

```text
--session
--output
--non-interactive
--working-directory
```

### Positional arguments

* Use positional arguments only for the primary target of a command.
* Do not overload positional arguments with multiple meanings.

Example:

```text
llamarc42 review adr 001
```

---

## PowerShell to CLI Mapping

This section defines conceptual mapping, not a requirement to expose one-to-one internal commands.

| Existing PowerShell concept | CLI surface                                             | Notes                           |
| --------------------------- | ------------------------------------------------------- | ------------------------------- |
| Chat/invoke workflow        | `llamarc42 chat`                                        | Main conversational entry point |
| Session creation            | `llamarc42 sessions new`                                | Explicit session creation       |
| Session resume              | `llamarc42 sessions resume`                             | Continue prior conversation     |
| Context inspection          | `llamarc42 context show`                                | Explainable retrieval           |
| Document review             | `llamarc42 review doc`                                  | Review a specific file          |
| ADR review                  | `llamarc42 review adr`                                  | Architecture-focused workflow   |
| Project/doc validation      | `llamarc42 docs validate`, `llamarc42 project validate` | Structural validation           |

### Mapping rules

* CLI commands should map to core behaviors, not directly to PowerShell cmdlet names.
* Shared logic must remain in Core.
* CLI is a surface only and must not duplicate retrieval, session, or review logic.
* If a PowerShell function is too implementation-specific, do not expose it directly in the CLI.

---

## Argument Conventions

### General rules

* Prefer explicit flags over implicit behavior.
* Flags must have stable meaning across command groups.
* Reuse common flags consistently.

### Common flags

```text
--output <plain|json|table>
--verbosity <quiet|normal|detailed|diagnostic>
--working-directory <path>
--non-interactive
--no-color
```

### Command-specific examples

```text
llamarc42 sessions list --output table
llamarc42 context show --output json
llamarc42 review adr 001 --verbosity detailed
llamarc42 chat --non-interactive "summarize docs"
```

### Flag behavior rules

* `--output json` must emit machine-friendly structured output only.
* `--output plain` should be stable and readable in terminals and logs.
* `--output table` is only valid for list-like results.
* Invalid combinations must fail explicitly.

---

## Output Formats

### `plain`

Default for human usage.

Characteristics:

* readable
* concise
* no unnecessary framing text
* suitable for copy/paste into docs or terminal logs

### `table`

Default for list-oriented interactive commands when terminal output supports it.

Use for:

* `sessions list`
* `docs list`
* `context list`

Must not be used for:

* deeply nested data
* large structured review output

### `json`

Required for automation.

Rules:

* Must be valid JSON.
* Must not include extra prose.
* Field names must remain stable once published.
* Errors in JSON mode must also be structured JSON.

---

## Interactive vs Non-Interactive Behavior

## Interactive mode

A command is interactive when:

* stdin is a terminal, and
* stdout is a terminal, and
* `--non-interactive` is not specified.

Interactive behavior may include:

* entering chat REPL mode
* prompting for session selection when ambiguity exists
* displaying formatted tables
* using color when supported

## Non-interactive mode

A command is non-interactive when:

* input is piped, or
* output is redirected, or
* `--non-interactive` is specified.

Non-interactive rules:

* never prompt
* never require human confirmation
* fail explicitly on ambiguity
* prefer plain or json-safe output
* exit with meaningful exit code

### Ambiguity handling

Interactive:

* may prompt the user to choose among multiple sessions or docs.

Non-interactive:

* must return an ambiguity error and suggest how to disambiguate.

---

## Error Handling Standards

### Principles

* Errors must be explicit.
* Errors must be actionable.
* Errors must not hide local filesystem causes.
* Surfaces must not swallow core exceptions without translation.

### Error categories

1. User input error
2. Project configuration error
3. Document validation error
4. Retrieval error
5. Session error
6. Execution/runtime error

### Error message format

Human-readable errors should contain:

* what failed
* why it failed if known
* what the user can do next

Example:

```text
Error: session 'architecture' is ambiguous.
Matched 2 sessions.
Use `llamarc42 sessions list` to inspect ids, then re-run with `--session <id>`.
```

### JSON error format

Example shape:

```json
{
  "error": {
    "code": "session_ambiguous",
    "message": "Session 'architecture' is ambiguous.",
    "details": {
      "matches": 2
    },
    "suggestion": "Use 'llamarc42 sessions list' and re-run with '--session <id>'."
  }
}
```

### Exit code guidance

| Exit code | Meaning                       |
| --------- | ----------------------------- |
| 0         | Success                       |
| 1         | General failure               |
| 2         | Invalid arguments or usage    |
| 3         | Not found                     |
| 4         | Ambiguous target              |
| 5         | Validation failure            |
| 6         | Runtime or dependency failure |

These codes should remain small and stable.

---

## Verbosity and Logging

### Verbosity levels

```text
quiet
normal
detailed
diagnostic
```

### Rules

* `quiet`: only final output or fatal errors
* `normal`: default human-facing behavior
* `detailed`: includes additional context about selected docs, session, and steps
* `diagnostic`: includes inspectable internals suitable for troubleshooting

### Logging principles

* Logging must not contaminate structured output.
* In `json` mode, logs should go to stderr where applicable.
* Diagnostic output should help explain retrieval, prompt construction boundaries, and session selection without leaking hidden chain-of-thought.

---

## Help and Discoverability

### Built-in help

Each command group should support:

```text
llamarc42 help
llamarc42 <group> help
llamarc42 <group> <command> --help
```

### Help content expectations

Help should include:

* purpose
* syntax
* examples
* key flags
* output behavior

---

## Architecture Constraints

1. CLI is a surface, not the home of business logic.
2. Retrieval selection remains policy-driven and implemented in Core.
3. Session persistence remains file-based and project-scoped.
4. Prompt construction must continue to be artifact-driven.
5. CLI commands must call shared services rather than implementing parallel workflows.

---

## Open Decisions

These items should be resolved before implementation starts:

1. Whether `chat` should be both a one-shot and REPL command.
2. Whether `sessions resume` should also imply entering interactive chat mode.
3. Whether `context show` displays the last built context or computes a fresh one.
4. Whether `review changes` is in scope for the upcoming milestone.
5. Whether a short executable alias should be introduced now or deferred.

---

## Recommended GitHub Issues

### Command Design

1. Define canonical top-level CLI command tree
2. Define `chat` command behavior and session targeting rules
3. Define `sessions` subcommands and identity rules
4. Define `context` inspection commands and explainability behavior
5. Define `review` command scope for ADRs, docs, and changes
6. Define `docs` and `project` validation commands

### Command Mapping

7. Map current PowerShell capabilities to CLI surface
8. Define cross-command argument conventions
9. Define common flag set and invalid flag combinations
10. Define output contract for plain, table, and json

### UX Rules

11. Define interactive vs non-interactive behavior rules
12. Define error taxonomy, messages, and exit codes
13. Define verbosity and diagnostic output rules
14. Define help and command discoverability rules

### Documentation

15. Finalize `cli-spec.md`
16. Add examples section with canonical workflows
17. Add architecture-boundary notes for CLI implementers

---

## Definition of Done

CLI specification is complete when:

* all command groups and subcommands are named and scoped
* argument behavior is defined without ambiguity
* output modes are defined and consistent
* interactive and non-interactive behavior is specified
* error handling and exit codes are specified
* mapping to current PowerShell capabilities is documented
* architecture constraints are explicitly captured
* `cli-spec.md` is approved as the implementation source of truth

---

## Recommended Implementation Posture

When implementation begins:

* start with command parsing and surface contracts
* route behavior into shared core services
* avoid adding commands not present in this spec without updating the spec first
* add golden examples for CLI outputs once command contracts stabilize

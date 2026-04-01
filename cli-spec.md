# CLI Spec — Issue 1: Top-Level Command Design

## Top-Level Command

Canonical executable:

```text
llamarc42
```

### Alias

* Deferred until after CLI surface stabilization
* Not part of the current spec milestone

### Rationale

* Explicit and aligned with project identity
* Avoids premature shorthand decisions that could create drift
* Ensures consistency across docs, scripts, and examples

---

## Command Grouping Philosophy

Top-level commands are organized around **user workflows**, not internal system domains.

The CLI is intended for **local developer usage**, so commands should reflect *what the user is trying to do*, not how the system is implemented.

### This means

* Commands answer: *“what task am I performing?”*
* Internal concepts (retrieval, prompt construction, etc.) are **not top-level**
* Supporting or inspectable behavior belongs in subcommands or flags

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

## Top-Level Command Responsibilities

### `chat`

Primary conversational workflow.

**Owns:**

* one-shot interactions
* interactive chat mode
* session targeting for conversation

**Does NOT own:**

* session lifecycle management
* project validation
* documentation validation

---

### `sessions`

Session lifecycle and continuity.

**Owns:**

* list, create, resume sessions
* inspect session metadata
* summarize sessions

**Does NOT own:**

* chat execution itself
* retrieval inspection
* project validation

---

### `review`

Artifact-focused review workflows.

**Owns:**

* ADR review
* documentation review
* future structured review workflows

**Does NOT own:**

* general doc browsing
* session management
* project inspection

---

### `docs`

Documentation inspection and validation.

**Owns:**

* list docs
* show docs
* validate documentation structure

**Does NOT own:**

* conversational workflows
* review logic (analysis belongs in `review`)
* project configuration

---

### `project`

Project-level inspection and validation.

**Owns:**

* project metadata
* project structure validation
* readiness checks

**Does NOT own:**

* document content validation
* session operations
* chat workflows

---

### `help`

Command discovery and usage guidance.

---

### `version`

CLI version output.

---

## Command Design Rules

### A command is top-level only if

* it represents a **primary user workflow**
* it has a **clear responsibility boundary**
* it is understandable without internal system knowledge
* it cannot logically fit under an existing command

---

### A command should NOT be top-level if

* it represents an internal concept (e.g., retrieval, prompts)
* it supports another workflow
* it would overlap with an existing command
* it is too narrow in scope

---

## Commands Explicitly NOT Top-Level

These are intentionally excluded to prevent architectural leakage:

### ❌ `context`

* Internal retrieval concept
* Better as subcommand or diagnostic feature
* Not a primary user workflow

---

### ❌ `retrieval`

* Implementation detail
* Not user-facing workflow

---

### ❌ `prompt`

* Internal system process
* Not user-driven action

---

### ❌ `config`

* Not yet defined as a core workflow
* Can be introduced later if needed

---

### ❌ `debug`

* Diagnostics handled via flags (`--verbosity`, etc.)
* Not a top-level concern

---

## Naming Rules

* lowercase only
* no abbreviations
* use **user-task language**, not implementation terms
* avoid PowerShell verb-noun patterns
* no aliases during spec phase

---

## Examples

```text
llamarc42 chat
llamarc42 chat "summarize the retrieval design"

llamarc42 sessions list
llamarc42 sessions resume architecture

llamarc42 review adr 001

llamarc42 docs validate

llamarc42 project show
```

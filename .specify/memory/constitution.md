<!--
SYNC IMPACT REPORT
==================
Version change: [unversioned template] → 1.0.0
Source: prompts/constitution.md (initial fill)
Modified principles: all (initial ratification — no prior version)
Added sections: Core Principles (I–V), Packaging & Distribution, Development Workflow, Governance
Removed sections: none
Templates requiring updates:
  ✅ .specify/memory/constitution.md — this file
  ✅ .specify/templates/plan-template.md — Constitution Check gate is generic placeholder; compatible, no structural change required
  ✅ .specify/templates/spec-template.md — template is generic; compatible with constitution constraints
  ✅ .specify/templates/tasks-template.md — task phases compatible; no new principle-driven task types required
  ℹ  .specify/templates/commands/ — no command files found; nothing to update
Deferred TODOs:
  - None. RATIFICATION_DATE set to 2026-04-29 (today, date of initial adoption).
-->

# task-cli Constitution

## Core Principles

### I. Language & Runtime

The project MUST be implemented in TypeScript, targeting the current Node.js LTS release.
All source files MUST execute via `tsx` or an equivalent direct TypeScript runner — no
separate compile step is required for development. Code style MUST be functional over
class-based: prefer pure functions, use `camelCase` for variables and functions, and
`PascalCase` exclusively for types and interfaces. Commander.js is the only permitted
CLI argument-parsing framework.

**Rationale**: TypeScript provides appropriate type safety for a CLI of this scope without
a compiled-binary workflow. Functional style keeps logic independently testable and
predictable.

### II. Minimal Architecture

The project structure MUST remain flat and organized by functionality:

- `commands/` — CLI command handlers
- `db/` — database access layer
- `utils/` — shared helpers

Clean Architecture layers, repository patterns, use-case classes, and similar abstractions
are PROHIBITED. The structure MUST NOT grow beyond these directories unless a concrete,
documented need is approved via a constitution amendment.

**Rationale**: A CLI tool of this scale does not warrant enterprise patterns. Premature
abstraction increases cognitive overhead and maintenance cost without measurable benefit.

### III. Data & Dependency Discipline

SQLite MUST be accessed via `better-sqlite3`. The database file MUST be stored at
`~/.task-cli/tasks.db`. ORMs and query builders are PROHIBITED. A dependency MUST NOT
be added if the same result can be achieved in a few lines of code. The only permitted
dependencies for data display are `cli-table3` or equivalent (tabular output only).

**Rationale**: `better-sqlite3` is synchronous, fast, and zero-config — ideal for a local
CLI. A lean dependency footprint reduces install size and maintenance surface.

### IV. Error Handling & Output

All user-facing error handling MUST follow these rules:

- Stack traces MUST NOT be shown to users; expose only clear, actionable messages
- Exit code MUST be `0` for success and `1` for any error condition
- Terminal output MUST be human-readable at all times
- List output MUST use a tabular format (`cli-table3` or equivalent)
- Color MUST be used exclusively for status indicators and priority levels; not decorative

**Rationale**: A CLI is a user-facing product. Leaking implementation internals degrades
the experience and provides no value to end users. Consistent exit codes enable scripting.

### V. Testing Discipline

All business logic MUST have unit tests written with Vitest. No minimum coverage percentage
is enforced, but any business-logic code without a corresponding test is a constitution
violation. Test files MUST live alongside or adjacent to the code they cover.

**Rationale**: Business logic correctness is non-negotiable. Coverage percentages are a
proxy metric; deliberate coverage of all meaningful code paths is the real requirement.

## Packaging & Distribution

The CLI MUST be installable globally via `npm install -g`. The `package.json` MUST declare
a `bin` entry pointing to the compiled entry point. The tool MUST function correctly when
invoked from any directory after global installation.

## Development Workflow

- Run `vitest` before any commit that touches business logic
- Do not introduce new top-level directories without documenting the rationale in the PR
- Keep `README.md` current with install instructions and command reference

## Governance

This constitution supersedes all other conventions in this repository. Any deviation from
these principles MUST be justified in writing and merged via a pull request that updates
this document. Version increments follow semantic versioning:

- **MAJOR**: removal or incompatible redefinition of a principle
- **MINOR**: new principle or section added, or materially expanded guidance
- **PATCH**: clarifications, wording fixes, non-semantic refinements

All implementation plans MUST include a Constitution Check gate that verifies compliance
with the five Core Principles before implementation begins.

**Version**: 1.0.0 | **Ratified**: 2026-04-29 | **Last Amended**: 2026-04-29

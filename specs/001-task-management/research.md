# Research: task-cli Core Task Management

**Phase 0 Output** | **Date**: 2026-04-29 | **Plan**: [plan.md](plan.md)

All technical decisions for this project are pre-determined by the project constitution.
Research confirms the choices are sound for the requirements.

---

## Decision 1: TypeScript + tsx

**Decision**: Use TypeScript with the `tsx` runner for development and `ts-node`-style
direct execution. Compile to CommonJS for the published npm package.

**Rationale**: TypeScript provides type safety for the Task entity and status transition
logic without a heavy build pipeline. `tsx` enables `ts-node`-compatible execution during
development. Mandated by Constitution Principle I.

**Alternatives considered**: Plain JavaScript (rejected — no type safety), Bun
(rejected — not universally available as a global runtime).

---

## Decision 2: better-sqlite3

**Decision**: Use `better-sqlite3` for all database operations. Synchronous API only.

**Rationale**: Synchronous SQLite is ideal for a CLI tool — no async overhead, no
connection pools, no event loop complexity. The library is well-maintained, zero-config,
and battle-tested for local-file use cases. Mandated by Constitution Principle III.

**Alternatives considered**: `node-sqlite3` (rejected — callback/async API adds complexity
for no benefit in a CLI context), Prisma/Drizzle ORM (rejected by constitution).

---

## Decision 3: Commander.js

**Decision**: Use Commander.js for all CLI argument parsing, subcommand registration, and
`--help` generation.

**Rationale**: Commander.js is the standard Node.js CLI framework. It handles subcommand
dispatch, option parsing, and help text with minimal boilerplate. Mandated by Constitution
Principle I.

**Alternatives considered**: yargs (rejected — constitution specifies Commander.js), meow
(rejected — too minimal for multi-command CLIs).

---

## Decision 4: cli-table3 for list output

**Decision**: Use `cli-table3` for rendering the `task list` output as a table.

**Rationale**: `cli-table3` produces well-formatted ASCII tables with column alignment,
color support via ANSI, and good cross-platform compatibility. Mandated by Constitution
Principle III.

**Alternatives considered**: `console.table` (rejected — no color support, inconsistent
formatting), hand-rolled padding (rejected — adds maintenance burden for no benefit over
a permitted dependency).

---

## Decision 5: ANSI color via inline helpers (no chalk)

**Decision**: Implement color output using a small inline ANSI helper in `utils/output.ts`
rather than adding `chalk` as a dependency.

**Rationale**: Color is needed only for status and priority values — a handful of ANSI
escape codes. Adding chalk for this would violate Constitution Principle III
("a dependency MUST NOT be added if the same result can be achieved in a few lines of code").

**Alternatives considered**: chalk (rejected — unnecessary dependency for 6 color values),
kleur (same rejection reason).

---

## Decision 6: Status transition enforcement at application layer

**Decision**: Validate status transitions in `utils/validation.ts` as a pure function,
not as a database constraint.

**Rationale**: SQLite CHECK constraints can enforce valid enum values but cannot express
"from-to" transition rules. A pure function in the application layer is testable, readable,
and keeps the logic where it belongs. The database still enforces the valid status enum.

**Valid transitions**:
- `todo` → `in-progress`
- `in-progress` → `done`
- `in-progress` → `todo`
- `done` → `todo`

**Invalid** (explicitly rejected):
- `todo` → `done` (must pass through `in-progress`)
- `done` → `in-progress` (must revert to `todo` first)

---

## Decision 7: Database path at ~/.task-cli/tasks.db

**Decision**: Store the SQLite file at `~/.task-cli/tasks.db`, created on first run if
it does not exist.

**Rationale**: Mandated by Constitution Principle III. Using the home directory avoids
permission issues and ensures the database persists across working directory changes.

**Implementation note**: Use `os.homedir()` to resolve the path cross-platform. Create
`~/.task-cli/` with `fs.mkdirSync({ recursive: true })` on startup.

---

## Decision 8: Vitest for unit tests

**Decision**: Use Vitest for all unit tests. Test files live in `tests/` adjacent to source.

**Rationale**: Mandated by Constitution Principle V. Vitest has native TypeScript support,
fast cold start, and a Jest-compatible API that requires no additional config.

**Test priority**: `utils/validation.ts` (transition logic) and `db/tasks.ts` (CRUD) are
the highest-priority test targets — they contain all business logic.

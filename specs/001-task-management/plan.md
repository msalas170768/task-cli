# Implementation Plan: task-cli Core Task Management

**Branch**: `001-task-management` | **Date**: 2026-04-29 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/001-task-management/spec.md`

## Summary

Build `task-cli`, a personal task management CLI installable as a global npm package.
The tool provides create, list, update, delete, and status-transition commands backed by
a local SQLite database at `~/.task-cli/tasks.db`. All design decisions are constrained
by the project constitution: TypeScript + Commander.js + better-sqlite3 + cli-table3 +
Vitest, flat commands/db/utils directory layout, no ORMs, no stack traces exposed.

## Technical Context

**Language/Version**: TypeScript / Node.js LTS (v20+)
**Primary Dependencies**: Commander.js (CLI), better-sqlite3 (SQLite), cli-table3 (tables)
**Storage**: SQLite at `~/.task-cli/tasks.db`
**Testing**: Vitest
**Target Platform**: Cross-platform (Windows/macOS/Linux), global npm install
**Project Type**: CLI tool
**Performance Goals**: All operations complete in <100ms (local synchronous SQLite)
**Constraints**: No stack traces to users; exit 0/1; tabular list; color for status/priority only
**Scale/Scope**: Single user, single machine, personal task list

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Requirement | Status |
|-----------|-------------|--------|
| I. Language & Runtime | TypeScript + Node.js LTS + tsx + Commander.js | ✅ Pass |
| II. Minimal Architecture | Flat: commands/, db/, utils/, index.ts only | ✅ Pass |
| III. Data & Dependency Discipline | better-sqlite3, no ORM, cli-table3 for display | ✅ Pass |
| IV. Error Handling & Output | No stack traces, exit 0/1, tables, color for status/priority | ✅ Pass |
| V. Testing Discipline | Vitest unit tests for all business logic | ✅ Pass |

All gates pass. No complexity tracking required.

*Post-design re-check (Phase 1): All artifacts conform — no violations introduced.*

## Project Structure

### Documentation (this feature)

```text
specs/001-task-management/
├── plan.md              # This file (/speckit-plan command output)
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
├── contracts/
│   └── cli-schema.md    # Phase 1 output
└── tasks.md             # Phase 2 output (/speckit-tasks — not created here)
```

### Source Code (repository root)

```text
commands/
├── add.ts          # `task add <title>` handler
├── list.ts         # `task list` handler
├── status.ts       # `task status <id> <status>` handler
├── update.ts       # `task update <id>` handler
└── delete.ts       # `task delete <id>` handler

db/
├── database.ts     # DB init, connection singleton, schema creation
└── tasks.ts        # Task CRUD: insert, selectAll, getById, update, delete

utils/
├── output.ts       # Table rendering and inline ANSI color helpers
└── validation.ts   # Status transition validation, input validators

index.ts            # Commander root — registers all commands, sets bin name

tests/
├── db/
│   └── tasks.test.ts
└── utils/
    └── validation.test.ts
```

**Structure Decision**: Single-project flat layout matching constitution Principle II.
No web/mobile layers. All logic fits within the three prescribed directories.

## Complexity Tracking

> No violations. No complexity justification required.

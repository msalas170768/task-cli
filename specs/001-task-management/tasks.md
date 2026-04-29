---
description: "Task list for task-cli core task management"
---

# Tasks: task-cli Core Task Management

**Input**: Design documents from `specs/001-task-management/`
**Prerequisites**: plan.md ✅, spec.md ✅, data-model.md ✅, contracts/cli-schema.md ✅, research.md ✅

**Tests**: Included — required by Constitution Principle V (all business logic must have unit tests).

**Organization**: Tasks are grouped by user story to enable independent implementation
and testing of each story.

## Format: `[ID] [P?] [Story?] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (US1, US2, US3)
- Exact file paths are included in each task description

---

## Phase 1: Setup

**Purpose**: Initialize the npm project, install dependencies, and configure tooling.

- [ ] T001 Initialize npm project — create `package.json` with name `task-cli`, version `0.1.0`, `"type": "module"`, and `"bin": { "task": "./index.js" }`
- [ ] T002 Install runtime dependencies — `npm install commander better-sqlite3 cli-table3`
- [ ] T003 [P] Install dev dependencies — `npm install -D typescript tsx vitest @types/node @types/better-sqlite3`
- [ ] T004 [P] Create `tsconfig.json` — strict mode, target ES2022, module NodeNext, outDir `./dist`
- [ ] T005 [P] Create `vitest.config.ts` — configure test include pattern `tests/**/*.test.ts`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core shared infrastructure that every user story depends on.

**⚠️ CRITICAL**: No user story work can begin until this phase is complete.

- [ ] T006 Implement `db/database.ts` — resolve `~/.task-cli/tasks.db` via `os.homedir()`, create directory with `fs.mkdirSync({ recursive: true })`, open `better-sqlite3` connection, run schema DDL from data-model.md, export a `getDb()` function returning the singleton connection
- [ ] T007 [P] Implement `utils/validation.ts` — export `isValidTransition(from, to)` (see transition table in data-model.md), `validateStatus(s)`, `validatePriority(p)`, `validateId(id)`, each returning `{ valid: boolean; error?: string }`
- [ ] T008 [P] Implement `utils/output.ts` — export `renderTable(headers, rows)` using `cli-table3`, export `colorStatus(s)` and `colorPriority(p)` using inline ANSI escape codes (no chalk): todo=white, in-progress=yellow, done=green; high=red, medium=yellow, low=white
- [ ] T009 Create `index.ts` — initialize Commander program, set name `task`, version from package.json, call `getDb()` on startup; placeholder to register commands (filled in per user story)

**Checkpoint**: DB connection works, validation functions are tested, table rendering works. User story work can now begin.

---

## Phase 3: User Story 1 — Task Lifecycle: Create and Complete Work (Priority: P1) 🎯 MVP

**Goal**: Users can create tasks and advance them through the status state machine.

**Independent Test**: `task add "Fix bug"` → `task list` (see it) → `task status 1 in-progress` → `task status 1 done`. Also verify `task status 1 done` from todo state is rejected.

### Tests for User Story 1 ⚠️ (write first, must fail before implementation)

- [ ] T010 [P] [US1] Write `tests/utils/validation.test.ts` — test all valid transitions pass, `todo→done` fails, `done→in-progress` fails, invalid status strings fail
- [ ] T011 [P] [US1] Write `tests/db/tasks.test.ts` (setup) — in-memory DB helper using `new Database(':memory:')` that applies the same schema DDL

### Implementation for User Story 1

- [ ] T012 [US1] Implement Task type and insert/get in `db/tasks.ts` — export `Task` interface (matching data-model.md), `insertTask(db, input)` returning new task id, `getTaskById(db, id)` returning `Task | null`
- [ ] T013 [US1] Implement `updateTaskStatus(db, id, newStatus)` in `db/tasks.ts` — validates task exists, calls `isValidTransition`, updates status and `updated_at`, throws on invalid transition
- [ ] T014 [US1] Implement `commands/add.ts` — Commander subcommand `add <title>`, options `--priority` and `--description`, calls `insertTask`, prints `Task #N created.`, exits 1 on validation error
- [ ] T015 [US1] Implement `commands/status.ts` — Commander subcommand `status <id> <new-status>`, calls `getTaskById` (error if not found), calls `updateTaskStatus`, prints `Task #N status updated: old → new.`, exits 1 on error
- [ ] T016 [US1] Register `add` and `status` commands in `index.ts`
- [ ] T017 [US1] Add `insertTask` and `updateTaskStatus` unit tests to `tests/db/tasks.test.ts` — verify insert returns correct id, verify valid/invalid transitions, verify task-not-found error

**Checkpoint**: US1 fully functional. A user can create a task, list it, advance and complete it, and get an error for invalid transitions.

---

## Phase 4: User Story 2 — Browse and Filter Tasks (Priority: P2)

**Goal**: Users can list all tasks in a table and filter by status or priority.

**Independent Test**: Create 3 tasks with different status/priority values. Run `task list`, `task list --status todo`, `task list --priority high`, and `task list --status done` (empty). Verify each shows correct subset.

### Tests for User Story 2 ⚠️ (write first, must fail before implementation)

- [ ] T018 [P] [US2] Add `selectAllTasks` tests to `tests/db/tasks.test.ts` — verify unfiltered returns all, status filter works, priority filter works, combined filter works, empty result returns `[]`

### Implementation for User Story 2

- [ ] T019 [US2] Implement `selectAllTasks(db, filters?)` in `db/tasks.ts` — accepts optional `{ status?, priority? }`, returns `Task[]` ordered by `created_at DESC`
- [ ] T020 [US2] Implement `commands/list.ts` — Commander subcommand `list`, options `--status` and `--priority`, calls `selectAllTasks`, renders with `renderTable` (colorized status/priority), prints `No tasks found.` if empty, exits 1 on invalid filter value
- [ ] T021 [US2] Register `list` command in `index.ts`

**Checkpoint**: US1 and US2 both work independently. Users can create tasks and browse/filter them.

---

## Phase 5: User Story 3 — Update and Delete Tasks (Priority: P3)

**Goal**: Users can modify task fields and remove tasks with a confirmation prompt.

**Independent Test**: Create a task → `task update 1 --title "New title"` → verify in `task list`. Then `task delete 1` (confirm y) → verify task gone. Test `task delete 1 --force` skips prompt. Test `task update 1` with no options shows error.

### Tests for User Story 3 ⚠️ (write first, must fail before implementation)

- [ ] T022 [P] [US3] Add `updateTask` and `deleteTask` tests to `tests/db/tasks.test.ts` — update changes only specified fields, delete removes the row, delete returns 0 rowcount for missing id

### Implementation for User Story 3

- [ ] T023 [P] [US3] Implement `updateTask(db, id, fields)` in `db/tasks.ts` — dynamically builds UPDATE for only provided fields (title, description, priority), updates `updated_at`, throws if task not found or no fields provided
- [ ] T024 [P] [US3] Implement `deleteTask(db, id)` in `db/tasks.ts` — deletes row by id, returns boolean (true if row existed)
- [ ] T025 [US3] Implement `commands/update.ts` — Commander subcommand `update <id>`, options `--title`, `--description`, `--priority`, validates at least one option provided, calls `getTaskById` (error if not found), calls `updateTask`, prints `Task #N updated.`
- [ ] T026 [US3] Implement `commands/delete.ts` — Commander subcommand `delete <id>`, option `--force`, calls `getTaskById` (error if not found), if no `--force` prompt `Delete task #N "title"? (y/N):` (use `readline`), calls `deleteTask` on confirm, prints `Task #N deleted.` or `Cancelled.`
- [ ] T027 [US3] Register `update` and `delete` commands in `index.ts`

**Checkpoint**: All user stories functional. Full CRUD + status management works end-to-end.

---

## Phase N: Polish & Cross-Cutting Concerns

**Purpose**: Final packaging, validation, and documentation.

- [ ] T028 [P] Create `.gitignore` — include `node_modules/`, `dist/`, `*.db`
- [ ] T029 Add `build` and `start` scripts to `package.json` — `"build": "tsc"`, `"start": "tsx index.ts"`, `"test": "vitest run"`
- [ ] T030 [P] Verify global install — `npm run build`, `npm install -g .`, run `task --help` and confirm binary works as `task`
- [ ] T031 Run all quickstart.md scenarios manually end-to-end and confirm output matches the contract in `contracts/cli-schema.md`
- [ ] T032 [P] Create/update `README.md` with install instructions and all five command examples

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies — start immediately
- **Foundational (Phase 2)**: Depends on Setup completion — BLOCKS all user stories
- **User Stories (Phase 3–5)**: All depend on Foundational phase
  - US1 (P1) first, then US2 and US3 can proceed in any order once US1 is done
  - US2 and US3 are independent of each other
- **Polish (Phase N)**: Depends on all desired user stories being complete

### User Story Dependencies

- **US1 (P1)**: Must complete first — creates the `db/tasks.ts` module that US2 and US3 extend
- **US2 (P2)**: Requires US1 complete (`db/tasks.ts` exists); adds `selectAllTasks`
- **US3 (P3)**: Requires US1 complete (`db/tasks.ts` exists); adds `updateTask`/`deleteTask`
- US2 and US3 have no dependency on each other

### Within Each User Story

1. Tests written first (must fail before implementation)
2. DB layer before command handler
3. Command handler before registration in `index.ts`

### Parallel Opportunities

- T003, T004, T005 (Setup) — all parallel
- T007, T008 (Foundational) — parallel with each other
- T010, T011 (US1 tests) — parallel setup
- T018 (US2 tests), T022 (US3 tests) — parallel with each other after US1 completes
- T023, T024 (US3 DB layer) — parallel with each other
- T028, T029, T030, T031, T032 (Polish) — mostly parallel

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational
3. Complete Phase 3: US1 (T010–T017)
4. **STOP and VALIDATE**: `task add`, `task status`, `task list` (basic) all work
5. Ship or demo MVP

### Incremental Delivery

1. Setup + Foundational → infrastructure ready
2. US1 → create + status management → MVP
3. US2 → list + filter → enhanced visibility
4. US3 → update + delete → full management
5. Polish → global install verified, docs complete

---

## Notes

- `[P]` = different files, no inter-task dependencies within the same phase
- `[USN]` maps each task to a specific user story for traceability
- Tests must be written first and confirmed failing before implementation
- Use `new Database(':memory:')` in tests — never touch `~/.task-cli/tasks.db` in tests
- Each user story phase must be independently testable before moving to the next
- Commit after each logical group or checkpoint

# Feature Specification: task-cli Core Task Management

**Feature Branch**: `001-task-management`
**Created**: 2026-04-29
**Status**: Draft
**Input**: User description from prompts/specify.md

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Task Lifecycle: Create and Complete Work (Priority: P1)

A user wants to capture a new task, start working on it, and mark it done when finished.

**Why this priority**: This is the fundamental value of the tool — without create and
complete, nothing else matters.

**Independent Test**: Create a task with `task add "Fix bug"`, advance it with
`task status 1 in-progress`, complete it with `task status 1 done`. Verify status at each
step via `task list`.

**Acceptance Scenarios**:

1. **Given** no tasks exist, **When** `task add "Fix login bug"` runs, **Then** a task is
   created with ID 1, status "todo", priority "medium", and a current timestamp.
2. **Given** task 1 has status "todo", **When** `task status 1 in-progress` runs, **Then**
   task 1 status changes to "in-progress".
3. **Given** task 1 has status "in-progress", **When** `task status 1 done` runs, **Then**
   task 1 status changes to "done".
4. **Given** task 1 has status "todo", **When** `task status 1 done` runs, **Then** the
   system rejects the transition with a clear error message and exits with code 1.

---

### User Story 2 - Browse and Filter Tasks (Priority: P2)

A user wants to see all their tasks at a glance and filter by status or priority.

**Why this priority**: Visibility into the task list is essential for managing work but
depends on tasks existing first (US1).

**Independent Test**: Create several tasks with varying status and priority, then run
`task list`, `task list --status todo`, and `task list --priority high` independently.

**Acceptance Scenarios**:

1. **Given** multiple tasks exist, **When** `task list` runs, **Then** all tasks are shown
   in a table with columns: ID, Title, Status, Priority, Created. Sorted newest-first.
2. **Given** tasks with various statuses exist, **When** `task list --status todo` runs,
   **Then** only tasks with status "todo" are shown.
3. **Given** tasks with various priorities exist, **When** `task list --priority high` runs,
   **Then** only high-priority tasks are shown.
4. **Given** no tasks match the filter, **When** `task list --status done` runs, **Then**
   an empty-state message is displayed (not an error), exit code 0.

---

### User Story 3 - Update and Delete Tasks (Priority: P3)

A user wants to modify task details or remove a task that is no longer relevant.

**Why this priority**: Management operations are useful but the core workflow (US1) remains
functional without them.

**Independent Test**: Create a task, update its title, verify change in `task list`. Then
delete the task and confirm it no longer appears.

**Acceptance Scenarios**:

1. **Given** task 1 exists, **When** `task update 1 --title "New title"` runs, **Then**
   task 1's title is updated and reflected in `task list`.
2. **Given** task 1 exists, **When** `task delete 1` runs without `--force`, **Then** the
   system prompts for confirmation before deleting.
3. **Given** task 1 exists, **When** `task delete 1 --force` runs, **Then** task 1 is
   deleted immediately without prompting.
4. **Given** a non-existent ID is provided, **When** any command uses that ID, **Then** a
   clear error message is shown and the process exits with code 1.

---

### Edge Cases

- What happens when the user passes an empty string as the task title?
- What happens when the database directory cannot be created (permission error)?
- What happens when an invalid status string is passed to `task status`?
- What happens when an invalid priority string is passed to any command?

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST create a task with a required title, optional priority
  (high/medium/low, default: medium), and optional description
- **FR-002**: System MUST assign each task a unique auto-incrementing integer ID
- **FR-003**: System MUST create every new task with initial status "todo"
- **FR-004**: System MUST display all tasks in a table: ID, Title, Status, Priority, Created
- **FR-005**: System MUST support filtering the task list by status (todo/in-progress/done)
- **FR-006**: System MUST support filtering the task list by priority (high/medium/low)
- **FR-007**: System MUST sort the task list by creation date, newest first, by default
- **FR-008**: System MUST change task status by ID, enforcing valid transitions only
- **FR-009**: System MUST reject the transition todo → done with a clear error message
- **FR-010**: System MUST allow modifying title, description, and/or priority of an existing
  task by ID
- **FR-011**: System MUST require user confirmation before deleting a task unless
  `--force` is provided
- **FR-012**: System MUST persist tasks in a local SQLite database at `~/.task-cli/tasks.db`
- **FR-013**: System MUST exit with code 0 on success and code 1 on any error
- **FR-014**: System MUST show clear, actionable error messages — no stack traces

### Key Entities

- **Task**: Has a unique ID, title, optional description, status (todo/in-progress/done),
  priority (high/medium/low), creation timestamp, and last-updated timestamp. Status
  transitions are enforced by the application layer.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A user can add, view, update, and delete tasks in under 5 seconds per
  operation on a modern machine
- **SC-002**: All valid status transitions complete without errors; all invalid transitions
  are rejected with a clear, actionable message
- **SC-003**: The task list remains accurate and consistent across multiple terminal sessions
- **SC-004**: A new user can install the tool and complete their first task workflow in
  under 2 minutes following the quickstart guide

## Assumptions

- Users run Node.js LTS (v20+)
- The user's home directory is writable (for `~/.task-cli/tasks.db`)
- Single user, single machine — no concurrent database access
- Due dates, tags, subtasks, external sync, authentication, and a web/REST interface are
  all explicitly out of scope for this version

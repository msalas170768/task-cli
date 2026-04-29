# CLI Contract: task-cli

**Phase 1 Output** | **Date**: 2026-04-29 | **Plan**: [../plan.md](../plan.md)

The public interface of `task-cli` is its command-line schema. This document defines
every command, argument, option, and output format that the tool exposes to users.

---

## Global

```
task [command] [options]
task --help
task --version
```

The binary is invocable as `task` after global npm install.

---

## Commands

### `task add`

Create a new task.

```
task add <title> [options]

Arguments:
  title               Task title (required, non-empty string)

Options:
  -p, --priority <level>      Priority: high | medium | low  (default: medium)
  -d, --description <text>    Optional description
  -h, --help                  Display help for command

Exit codes:
  0   Task created successfully
  1   Validation error (empty title, invalid priority)
```

**Output on success**:
```
Task #1 created.
```

**Output on error**:
```
Error: Title cannot be empty.
```

---

### `task list`

Display all tasks in a table.

```
task list [options]

Options:
  -s, --status <status>       Filter by status: todo | in-progress | done
  -p, --priority <level>      Filter by priority: high | medium | low
  -h, --help                  Display help for command

Exit codes:
  0   Always (empty result is not an error)
  1   Invalid filter value
```

**Output (with tasks)**:
```
┌────┬──────────────────┬─────────────┬──────────┬─────────────────────┐
│ ID │ Title            │ Status      │ Priority │ Created             │
├────┼──────────────────┼─────────────┼──────────┼─────────────────────┤
│  1 │ Fix login bug    │ in-progress │ high     │ 2026-04-29 10:00    │
│  2 │ Write tests      │ todo        │ medium   │ 2026-04-29 09:00    │
└────┴──────────────────┴─────────────┴──────────┴─────────────────────┘
```

Status and priority values are colorized:
- Status: `todo` (white), `in-progress` (yellow), `done` (green)
- Priority: `high` (red), `medium` (yellow), `low` (white)

**Output (no tasks)**:
```
No tasks found.
```

---

### `task status`

Change a task's status.

```
task status <id> <new-status>

Arguments:
  id            Task ID (positive integer)
  new-status    Target status: todo | in-progress | done

Exit codes:
  0   Status updated successfully
  1   Task not found, invalid status, or invalid transition
```

**Output on success**:
```
Task #1 status updated: in-progress → done.
```

**Output on invalid transition**:
```
Error: Cannot transition from 'todo' to 'done'. Move to 'in-progress' first.
```

**Output on task not found**:
```
Error: Task #99 not found.
```

---

### `task update`

Modify one or more fields of an existing task.

```
task update <id> [options]

Arguments:
  id                    Task ID (positive integer)

Options:
  -t, --title <text>          New title (non-empty string)
  -d, --description <text>    New description (pass "" to clear)
  -p, --priority <level>      New priority: high | medium | low
  -h, --help                  Display help for command

Exit codes:
  0   Task updated successfully
  1   Task not found, no options provided, invalid value
```

**Output on success**:
```
Task #1 updated.
```

**Output when no options provided**:
```
Error: Provide at least one field to update (--title, --description, --priority).
```

---

### `task delete`

Delete a task by ID.

```
task delete <id> [options]

Arguments:
  id            Task ID (positive integer)

Options:
  -f, --force   Skip confirmation prompt
  -h, --help    Display help for command

Exit codes:
  0   Task deleted successfully (or user cancelled — cancellation is not an error)
  1   Task not found
```

**Output (interactive prompt)**:
```
Delete task #1 "Fix login bug"? (y/N): _
```

- Answering `y` or `Y`: deletes and prints `Task #1 deleted.`
- Any other input: prints `Cancelled.` and exits 0

**Output with --force**:
```
Task #1 deleted.
```

---

## Validation Rules (cross-command)

| Input       | Rule                                     | Error message                                   |
|-------------|------------------------------------------|-------------------------------------------------|
| `title`     | Non-empty string after trim              | `Error: Title cannot be empty.`                 |
| `id`        | Positive integer                         | `Error: ID must be a positive integer.`         |
| `status`    | One of: todo, in-progress, done          | `Error: Invalid status '<value>'.`              |
| `priority`  | One of: high, medium, low                | `Error: Invalid priority '<value>'.`            |
| Transition  | Must follow state machine rules          | `Error: Cannot transition from 'X' to 'Y'. ...` |
| Not found   | Task with given ID must exist            | `Error: Task #N not found.`                     |

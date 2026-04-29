# Quickstart: task-cli

**Phase 1 Output** | **Date**: 2026-04-29

---

## Install

```bash
npm install -g task-cli
```

Verify:

```bash
task --version
task --help
```

---

## Typical Workflow

### 1. Create tasks

```bash
task add "Buy groceries"
# Task #1 created.

task add "Prepare presentation" --priority high
# Task #2 created.

task add "Read Atomic Habits" --priority low --description "Chapter 1-5 this week"
# Task #3 created.
```

### 2. View all tasks

```bash
task list
```

```
┌────┬──────────────────────┬────────┬──────────┬─────────────────────┐
│ ID │ Title                │ Status │ Priority │ Created             │
├────┼──────────────────────┼────────┼──────────┼─────────────────────┤
│  3 │ Read Atomic Habits   │ todo   │ low      │ 2026-04-29 10:02    │
│  2 │ Prepare presentation │ todo   │ high     │ 2026-04-29 10:01    │
│  1 │ Buy groceries        │ todo   │ medium   │ 2026-04-29 10:00    │
└────┴──────────────────────┴────────┴──────────┴─────────────────────┘
```

### 3. Start working on a task

```bash
task status 2 in-progress
# Task #2 status updated: todo → in-progress.
```

### 4. Filter to see what's in progress

```bash
task list --status in-progress
```

### 5. Complete the task

```bash
task status 2 done
# Task #2 status updated: in-progress → done.
```

### 6. Update task details

```bash
task update 1 --title "Buy groceries and cook dinner"
# Task #1 updated.

task update 3 --priority medium
# Task #3 updated.
```

### 7. Delete a task

```bash
# Interactive (prompts for confirmation):
task delete 1
# Delete task #1 "Buy groceries and cook dinner"? (y/N): y
# Task #1 deleted.

# Skip confirmation:
task delete 3 --force
# Task #3 deleted.
```

---

## Filter Examples

```bash
# All high-priority tasks
task list --priority high

# All tasks not yet started
task list --status todo

# Combine filters
task list --status todo --priority high
```

---

## Error Examples

```bash
# Invalid transition — must go through in-progress first
task status 1 done
# Error: Cannot transition from 'todo' to 'done'. Move to 'in-progress' first.

# Task not found
task status 99 done
# Error: Task #99 not found.

# No update options provided
task update 1
# Error: Provide at least one field to update (--title, --description, --priority).
```

---

## Data Location

Tasks are stored at `~/.task-cli/tasks.db`. The directory and file are created
automatically on first use. To reset all tasks, delete this file.

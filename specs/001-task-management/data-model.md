# Data Model: task-cli Core Task Management

**Phase 1 Output** | **Date**: 2026-04-29 | **Plan**: [plan.md](plan.md)

---

## Entity: Task

The sole persistent entity. Represents a unit of work a user wants to track.

### Fields

| Field        | SQLite Type  | Constraints                                      | Description                        |
|--------------|--------------|--------------------------------------------------|------------------------------------|
| `id`         | INTEGER      | PRIMARY KEY AUTOINCREMENT                        | Unique auto-incrementing identifier |
| `title`      | TEXT         | NOT NULL                                         | Short description of the task      |
| `description`| TEXT         |                                                  | Optional longer description        |
| `status`     | TEXT         | NOT NULL DEFAULT 'todo' CHECK(status IN (...))   | Current lifecycle state            |
| `priority`   | TEXT         | NOT NULL DEFAULT 'medium' CHECK(priority IN (...))| Urgency level                     |
| `created_at` | TEXT         | NOT NULL                                         | ISO 8601 creation timestamp        |
| `updated_at` | TEXT         | NOT NULL                                         | ISO 8601 last-update timestamp     |

### Valid Values

**status**: `'todo'` | `'in-progress'` | `'done'`

**priority**: `'high'` | `'medium'` | `'low'`

### DDL

```sql
CREATE TABLE IF NOT EXISTS tasks (
  id          INTEGER PRIMARY KEY AUTOINCREMENT,
  title       TEXT    NOT NULL,
  description TEXT,
  status      TEXT    NOT NULL DEFAULT 'todo'
                CHECK(status IN ('todo', 'in-progress', 'done')),
  priority    TEXT    NOT NULL DEFAULT 'medium'
                CHECK(priority IN ('high', 'medium', 'low')),
  created_at  TEXT    NOT NULL,
  updated_at  TEXT    NOT NULL
);
```

---

## Status State Machine

```
         ┌──────────────────────────────┐
         │                              │
         ▼                              │
      [ todo ] ──────────────► [ in-progress ] ──────────────► [ done ]
         ▲                              │                          │
         │                              │                          │
         └──────────────────────────────┘                          │
         ▲                                                         │
         └─────────────────────────────────────────────────────────┘
```

### Transition Table

| From         | To           | Valid |
|--------------|--------------|-------|
| `todo`       | `in-progress`| ✅    |
| `in-progress`| `done`       | ✅    |
| `in-progress`| `todo`       | ✅    |
| `done`       | `todo`       | ✅    |
| `todo`       | `done`       | ❌    |
| `done`       | `in-progress`| ❌    |

---

## TypeScript Interface

```typescript
export interface Task {
  id: number;
  title: string;
  description: string | null;
  status: 'todo' | 'in-progress' | 'done';
  priority: 'high' | 'medium' | 'low';
  created_at: string;  // ISO 8601
  updated_at: string;  // ISO 8601
}

export type TaskStatus = Task['status'];
export type TaskPriority = Task['priority'];
```

---

## Database Location

`~/.task-cli/tasks.db` — resolved via `os.homedir()` at runtime. The directory
`~/.task-cli/` is created with `fs.mkdirSync({ recursive: true })` on first run.

---

## Indexes

No explicit indexes needed. The dataset is single-user and small; SQLite's rowid-based
primary key scan is sufficient for all operations.

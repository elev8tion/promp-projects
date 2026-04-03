# t0ggles IDs & Tool Reference

## Board: Prompt Projects

| Key | Value |
|-----|-------|
| Board ID | `GvBfmcjk7DXvvaIHurlX` |
| Board URL | `https://t0ggles.com/prompt-projects` |

## Statuses

| Name | ID | Type | Use For |
|------|----|------|---------|
| To-Do | `gWDIdGO9CWJ2EO25VE8L` | initial | All new tasks |
| In Progress | `AGFTWabh2UtnUJdEvhUr` | progress | Active work |
| Done | `E0UUGCLWpN60twU4AtrU` | done | Completed steps |

> Always use the To-Do status ID when creating tasks. The user manually advances status as they execute each prompt.

---

## Tool Cheatsheet for This Workflow

### Find the project
```
mcp__t0ggles__list-projects
  boardId: GvBfmcjk7DXvvaIHurlX
→ match project by title (case-insensitive), save projectId
```

### Create a milestone (one per phase)
```
mcp__t0ggles__create-milestone
  boardId:     GvBfmcjk7DXvvaIHurlX
  projectId:   <projectId>
  title:       "Phase N: [Phase Title]"
  description: "<one-line scope>"
→ save milestoneId
```

### Bulk-create tasks (call once per phase)
```
mcp__t0ggles__bulk-create-tasks
  boardId:   GvBfmcjk7DXvvaIHurlX
  projectId: <projectId>
  tasks: [
    {
      title:       "...",
      description: "<full Claude Code prompt>",
      statusId:    "gWDIdGO9CWJ2EO25VE8L",
      priority:    "high" | "medium" | "low",
      tags:        ["Phase N"]
    }
  ]
→ save all returned task IDs in order
```

### Wire a dependency
```
mcp__t0ggles__create-dependency
  taskId:          <successor task ID>
  dependsOnTaskId: <predecessor task ID>
```
> Wire after ALL tasks are created — you need IDs for both ends.

### Create master note
```
mcp__t0ggles__create-note
  boardId:   GvBfmcjk7DXvvaIHurlX
  projectId: <projectId>
  title:     "[project_name] — Rebuild Master Plan"
  content:   "<markdown content>"
```

### Set project AI context
```
mcp__t0ggles__update-project
  boardId:   GvBfmcjk7DXvvaIHurlX
  projectId: <projectId>
  aiContext: "<summary string>"
```

---

## Dependency Wiring Pattern

Given tasks grouped by phase with IDs tracked as arrays:

```
phase1_tasks = [t1, t2, t3, t4]        → last = t4
phase2_tasks = [t5, t6, t7]            → last = t7
phase3_tasks = [t8, t9]                → last = t9
phase4_users = [t10, t11]              → last = t11
phase4_orders = [t12, t13]             → last = t13
phase5_tasks = [t14, t15, t16]         → last = t16
phase6_tasks = [t17, t18, t19]         → last = t19
phase7_tasks = [t20, t21]

Dependencies to create:
  t5→t4, t6→t4, t7→t4          (all phase 2 → last phase 1)
  t8→t7, t9→t7                  (all phase 3 → last phase 2)
  t10→t9, t11→t9                (phase4 users → last phase 3)
  t11→t10                       (sequential within users tag)
  t12→t9, t13→t9                (phase4 orders → last phase 3)
  t13→t12                       (sequential within orders tag)
  t14→t11, t14→t13              (phase 5 → last of ALL phase 4 groups)
  t15→t14, t16→t14              (sequential within phase 5)
  t17→t16, t18→t16, t19→t16    (all phase 6 → last phase 5)
  t20→t19, t21→t19              (phase 7 → last phase 6)
```

> Phase 5 tasks depend on the last task of ALL Phase 4 tag groups (not just one). Create a dependency from the first Phase 5 task to every last-Phase-4 task.

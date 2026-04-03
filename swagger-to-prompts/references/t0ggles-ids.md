# t0ggles IDs & Tool Reference

## Board: Prompt Projects

| Key | Value |
|-----|-------|
| Board ID | `GvBfmcjk7DXvvaIHurlX` |
| Board URL | `https://t0ggles.com/prompt-projects` |

## Statuses

| Name | ID | Type |
|------|----|------|
| To-Do | `gWDIdGO9CWJ2EO25VE8L` | initial |
| In Progress | `AGFTWabh2UtnUJdEvhUr` | progress |
| Done | `E0UUGCLWpN60twU4AtrU` | done |

> Always use the To-Do status ID when creating tasks. The user advances status manually.

---

## Tool Cheatsheet

### Find the project
```
mcp__t0ggles__list-projects
  boardId: GvBfmcjk7DXvvaIHurlX
→ match by title (case-insensitive), save projectId
```

### Create spec notes folder
```
mcp__t0ggles__create-note
  boardId: GvBfmcjk7DXvvaIHurlX
  title:   "[project_name] — Build Spec"
  type:    "folder"
→ save id as spec_folder_id
```

### Create a note inside the folder
```
mcp__t0ggles__create-note
  boardId:     GvBfmcjk7DXvvaIHurlX
  title:       "00 · Project Context"   (or "Phase N · [Title]")
  parentId:    <spec_folder_id>
  description: "<markdown content>"
→ save url for use in task descriptions
```

### Create a milestone
```
mcp__t0ggles__create-milestone
  boardId:     GvBfmcjk7DXvvaIHurlX
  projectId:   <projectId>
  title:       "Phase N: [Phase Title]"
  description: "<one-line scope> · Spec: <phase_note_url>"
→ save milestoneId
```

### Create a task with subtasks
```
mcp__t0ggles__create-task
  boardId:   GvBfmcjk7DXvvaIHurlX
  projectId: <projectId>
  title:     "<action-oriented title>"
  description: "<short prompt — max 20 lines, links to spec notes>"
  statusId:  "gWDIdGO9CWJ2EO25VE8L"
  priority:  "high" | "medium" | "low"
  tags:      ["Phase N", "<semantic tag>"]
  subtasks:  [
    { title: "<verifiable acceptance criterion>" },
    { title: "<verifiable acceptance criterion>" }
  ]
→ save returned task id

NOTE: Use create-task (not bulk-create-tasks) for all tasks — only
      create-task supports the subtasks array.
```

### Wire a dependency
```
mcp__t0ggles__create-dependency
  boardId:          GvBfmcjk7DXvvaIHurlX
  predecessorTaskId: <task that must complete first>
  successorTaskId:   <task that depends on it>
```
> Wire after ALL tasks are created — you need both IDs.

### Set project AI context
```
mcp__t0ggles__update-project
  boardId:   GvBfmcjk7DXvvaIHurlX
  projectId: <projectId>
  aiContext: "<summary + context_note_url + stack>"
```

---

## Dependency Wiring Pattern

```
phase1_tasks = [t1, t2, t3, t4]           → last = t4
phase2_tasks = [t5, t6, t7]               → last = t7
phase3_tasks = [t8, t9]                   → last = t9
phase4_tagA  = [t10, t11]                 → last = t11
phase4_tagB  = [t12, t13]                 → last = t13
phase5_tasks = [t14, t15]                 → last = t15
phase6_tasks = [t16, t17, t18, t19]       → last = t19
phase7_tasks = [t20, t21]

Dependencies to create:
  t2→t1, t3→t2, t4→t3               (Phase 1 sequential)
  t5→t4, t6→t4, t7→t4               (all Phase 2 → last Phase 1)
  t6→t5, t7→t6                       (Phase 2 sequential)
  t8→t7, t9→t7                       (all Phase 3 → last Phase 2)
  t9→t8                               (Phase 3 sequential)
  t10→t9, t11→t9                     (Phase 4 tagA → last Phase 3)
  t11→t10                             (sequential within tagA)
  t12→t9, t13→t9                     (Phase 4 tagB → last Phase 3)
  t13→t12                             (sequential within tagB)
  t14→t11, t14→t13                   (Phase 5 → last of ALL Phase 4 groups)
  t15→t14                             (Phase 5 sequential)
  t16→t15, t17→t15, t18→t15, t19→t15 (all Phase 6 → last Phase 5)
  t17→t16, t18→t17, t19→t18          (Phase 6 sequential)
  t20→t19, t21→t20                   (Phase 7 → last Phase 6, sequential)
```

> Phase 5 depends on the last task of EVERY Phase 4 tag group — not just one.
> Create a dependency from Phase 5's first task to each group's last task.

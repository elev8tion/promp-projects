---
name: swagger-to-prompts
description: >
  Consumes a swagger.yaml (OpenAPI 3.0) file and a t0ggles project name, then populates that
  t0ggles project with ordered, self-contained Claude Code prompts to systematically rebuild the
  original project under a new name/brand identity. Use when the user says "prompt this file,
  here [project name]" and provides a path to a .yaml file, or says "use swagger-to-prompts on
  [path] for project [name]". The new project name is BOTH the t0ggles project to populate AND
  the new identity that replaces all original branding in every generated prompt.
---

# swagger-to-prompts

Transform a `swagger.yaml` into a fully structured t0ggles rebuild plan — ordered Claude Code prompts, milestones, dependencies, and a master note — all under a new project identity.

## Inputs

- **yaml_path**: Absolute path to the `swagger.yaml` file
- **project_name**: The t0ggles project name (must already exist on the board) — also becomes the new identity replacing the original project name in every prompt

If either is missing, ask before proceeding.

---

## Step 1 — Validate

1. Read the yaml file at `yaml_path`. If it doesn't exist or isn't valid YAML, stop and report.
2. Call `mcp__t0ggles__list-projects` (boardId: `GvBfmcjk7DXvvaIHurlX`). Find the project whose title matches `project_name` (case-insensitive). Save its `projectId`.
   - If not found: stop — "Create a project named '[project_name]' on the Prompt Projects board first, then re-run."
3. Call `mcp__t0ggles__list-statuses` (boardId: `GvBfmcjk7DXvvaIHurlX`). Save the ID for the status with `type: "initial"` (To-Do).

---

## Step 2 — Parse the Swagger

Extract and hold in memory:

| Field | Source |
|-------|--------|
| `original_name` | `info.title` |
| `description` | `info.description` |
| `base_path` | `servers[0].url` |
| `tags` | `tags[].name` (or infer from first path segment if absent) |
| `schemas` | `components.schemas` — full property definitions |
| `security_schemes` | `components.securitySchemes` — names and types |
| `paths_by_tag` | Group all `paths` entries by their first `tags[]` value |
| `tech_stack` | `info.x-framework`, `info.x-generator`, or infer from schema style |

If `tags` is empty, derive tag groups from the first path segment of each endpoint (e.g. `/users/...` → `Users`).

---

## Step 3 — Plan Phases & Create Milestones

Read `references/phase-structure.md` for full phase definitions and task-splitting rules.

Build the phase list dynamically:
- Always include: Phase 1 (Foundation), Phase 5 (Middleware), Phase 6 (Testing), Phase 7 (Docs & Deploy)
- Include Phase 2 (Models) only if `schemas` is non-empty
- Include Phase 3 (Auth) only if `security_schemes` is non-empty
- Include one Phase 4 milestone **per tag group** — title: `"Phase 4: Core API — [Tag]"`

Renumber phases sequentially after any skipped phases.

For each phase, call `mcp__t0ggles__create-milestone`:
```
boardId:     GvBfmcjk7DXvvaIHurlX
projectId:   <found project ID>
title:       "Phase N: [phase title]"
description: <one-line summary of what this phase covers>
```

Save each returned milestone ID mapped to its phase number.

---

## Step 4 — Generate Tasks & Prompts

Read `references/prompt-template.md` for the exact prompt format each task description must follow.
Read `references/phase-structure.md` for per-phase task breakdown and granularity rules.

For each phase, construct the task array then call `mcp__t0ggles__bulk-create-tasks`:
```
boardId:   GvBfmcjk7DXvvaIHurlX
projectId: <found project ID>
tasks: [
  {
    title:       "<action-oriented title>",
    description: "<full self-contained Claude Code prompt — see prompt-template.md>",
    statusId:    "<To-Do status ID>",
    priority:    "high"   (Phase 1–3)
                 "medium" (Phase 4–5)
                 "low"    (Phase 6–7),
    tags:        ["Phase N"]
  }
]
```

**Non-negotiables for every prompt:**
- Uses `project_name` everywhere — `original_name` never appears
- Is fully self-contained (no references to prior steps for context)
- Pulls exact specs from swagger (schema properties with types, HTTP methods, paths, request/response shapes)
- Ends with the stop phrase: `Reply "Step N complete — [summary]" and await the next prompt.`

Track all created task IDs in phase order for Step 5.

---

## Step 5 — Wire Dependencies

After all tasks are created, call `mcp__t0ggles__create-dependency` for each link:

```
mcp__t0ggles__create-dependency
  taskId:          <successor>
  dependsOnTaskId: <predecessor>
```

Wiring rules:
- Every Phase 2 task → last Phase 1 task
- Every Phase 3 task → last Phase 2 task (or last Phase 1 if Phase 2 skipped)
- Every Phase 4 task → last Phase 3 task (or last Phase 2 if Phase 3 skipped)
- Phase 4 tasks within the same tag group → sequential (each depends on prior)
- Every Phase 5 task → last Phase 4 task across all tag groups
- Every Phase 6 task → last Phase 5 task
- Phase 7 tasks → last Phase 6 task

---

## Step 6 — Create Master Note

Call `mcp__t0ggles__create-note`:
```
boardId:   GvBfmcjk7DXvvaIHurlX
projectId: <found project ID>
title:     "[project_name] — Rebuild Master Plan"
```

Note content (markdown) must include:
- **Identity Mapping** table: original_name → project_name, where to apply substitution
- **Tech Stack** section: inferred language, framework, auth, DB
- **Build Phases** table: phase number, title, task count
- **Schema Reference**: bullet list of all schema names
- **Endpoint Groups**: bullet list of tag groups with endpoint count each
- **How to Execute** instructions (copy task description → paste into fresh Claude Code session → mark Done → next task)

---

## Step 7 — Set Project AI Context

Call `mcp__t0ggles__update-project`:
```
boardId:   GvBfmcjk7DXvvaIHurlX
projectId: <found project ID>
aiContext: |
  Project: [project_name]
  Original source: [original_name] — [N] endpoints, [N] tag groups
  Tech stack: [inferred]
  New identity: [project_name] (replaces all [original_name] references)
  Build: [N] phases, [N] total tasks — execute in order
```

---

## Step 8 — Report to User

```
✓ Project found:       [project_name]
✓ Swagger parsed:      [original_name] — [N] endpoints across [N] tag groups
✓ New identity:        [project_name]
✓ Milestones created:  [N] phases
✓ Tasks created:       [N] total prompts
✓ Dependencies wired:  [N] links
✓ Master note created: "[project_name] — Rebuild Master Plan"

All prompts are in t0ggles → Prompt Projects → [project_name].
Execute them in order. Each task description is your Claude Code prompt.
```

---

## Reference Files

- `references/phase-structure.md` — phase definitions, task-splitting rules, per-phase task inventory
- `references/prompt-template.md` — exact Claude Code prompt format with all field explanations
- `references/t0ggles-ids.md` — board ID, status IDs, tool quick-reference for this workflow

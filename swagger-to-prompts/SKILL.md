---
name: swagger-to-prompts
description: >
  Consumes a swagger.yaml (OpenAPI 3.0) file and a t0ggles project name, then populates that
  t0ggles project with a three-layer build plan: spec notes (source of truth), short Claude
  Code prompt tasks (max 20 lines) that link to the notes, and subtask checklists for
  verification. Use when the user says "prompt this file, here [project name]" and provides a
  path to a .yaml file, or says "use swagger-to-prompts on [path] for project [name]". The new
  project name is BOTH the t0ggles project to populate AND the new identity replacing all
  original branding in every generated note and prompt.
---

# swagger-to-prompts

Transform a `swagger.yaml` into a t0ggles build plan structured in three layers:

- **Notes** — the spec (source of truth, editable, linked from tasks)
- **Tasks** — short Claude Code prompts (max 20 lines) that reference spec notes
- **Subtasks** — acceptance criteria checklist (verifiable in t0ggles)

To change the tech stack or any spec detail later, edit the relevant note — all tasks
automatically reflect the change on next execution because they read the notes at runtime.

## Autonomous Execution

All t0ggles MCP tools are pre-approved — no permission prompts will interrupt this skill. When triggered with both inputs, proceed through all 8 steps from start to finish without pausing between steps. Only stop if a hard error occurs that prevents continuation (project not found, invalid YAML, MCP failure) — report the error and wait for the user to resolve it before continuing.

## Inputs

- **yaml_path**: Absolute path to the `swagger.yaml` file
- **project_name**: The t0ggles project name (must already exist) — also the new identity replacing all original branding

If either input is missing, ask for it before starting. Once both are confirmed, run to completion.

---

## Step 1 — Validate

1. Read the yaml file at `yaml_path`. If missing or invalid, stop and report.
2. Call `mcp__t0ggles__list-projects` (boardId: `GvBfmcjk7DXvvaIHurlX`). Match `project_name` case-insensitively. Save `projectId`.
   - If not found: stop — "Create a project named '[project_name]' on the Prompt Projects board first."
3. Call `mcp__t0ggles__list-statuses` (boardId: `GvBfmcjk7DXvvaIHurlX`). Save the ID for `type: "initial"` (To-Do).

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
| `paths_by_tag` | Group all `paths` by their first `tags[]` value |
| `tech_stack` | `info.x-framework`, `info.x-generator`, or infer from schema/path style |

---

## Step 3 — Create Spec Notes

Read `references/note-template.md` for exact note content formats.

**3a. Create the root spec folder**
```
mcp__t0ggles__create-note
  boardId: GvBfmcjk7DXvvaIHurlX
  title:   "[project_name] — Build Spec"
  type:    "folder"
```
Save returned `id` as `spec_folder_id`.

**3b. Create the Project Context note**
```
mcp__t0ggles__create-note
  boardId:     GvBfmcjk7DXvvaIHurlX
  title:       "00 · Project Context"
  parentId:    <spec_folder_id>
  description: <see note-template.md — Project Context section>
```
Save returned `url` as `context_note_url`.

**3c. Create one Phase spec note per phase**
For each phase in the build plan:
```
mcp__t0ggles__create-note
  boardId:     GvBfmcjk7DXvvaIHurlX
  title:       "Phase N · [Phase Title]"
  parentId:    <spec_folder_id>
  description: <see note-template.md — Phase Spec section>
```
Save each returned `url` as `phase_note_url[N]`.

---

## Step 4 — Plan Phases & Create Milestones

Read `references/phase-structure.md` for full phase definitions.

Build phases dynamically:
- Always include: Phase 1 (Foundation), Phase N (Middleware), Phase N (Testing), Phase N (Docs & Deploy)
- Include Phase 2 (Models) only if `schemas` non-empty
- Include Phase 3 (Auth) only if `security_schemes` non-empty
- One Phase 4 milestone **per tag group** — title: `"Phase N: Core API — [Tag]"`
- Renumber sequentially after any skipped phases

For each phase:
```
mcp__t0ggles__create-milestone
  boardId:     GvBfmcjk7DXvvaIHurlX
  projectId:   <projectId>
  title:       "Phase N: [phase title]"
  description: "[one-line scope] · Spec: [phase_note_url[N]]"
```
Save each returned milestone ID.

---

## Step 5 — Create Tasks

Read `references/prompt-template.md` for the short task description format.
Read `references/phase-structure.md` for per-phase task breakdown and subtask rules.

For each task, call `mcp__t0ggles__create-task` (supports subtasks natively):
```
mcp__t0ggles__create-task
  boardId:     GvBfmcjk7DXvvaIHurlX
  projectId:   <projectId>
  title:       "<action-oriented title>"
  description: "<short prompt — see prompt-template.md, max 20 lines>"
  statusId:    <To-Do status ID>
  priority:    "high"   (Phase 1–3)
               "medium" (Phase 4–5)
               "low"    (Phase 6–7)
  tags:        ["Phase N", "<semantic tag: auth|database|api|testing|deploy>"]
  subtasks:    [
    { title: "<verifiable acceptance criterion>" },
    { title: "<verifiable acceptance criterion>" },
    ...
  ]
```

Track all created task IDs in phase order for Step 6.

**Non-negotiables for every task description:**
- Max 20 lines total
- Opens with links to Project Context note and the Phase spec note
- Uses `project_name` everywhere — `original_name` never appears
- Ends with: `Reply "Step N complete — [summary]" and await the next prompt.`

**Subtasks = acceptance criteria (3–6 per task)**
Each subtask is a single verifiable statement — see `references/prompt-template.md` for examples.

---

## Step 6 — Wire Dependencies

After ALL tasks are created, call `mcp__t0ggles__create-dependency` for each link.

See `references/t0ggles-ids.md` for the full wiring pattern.

Rules:
- Every Phase 2 task → last Phase 1 task
- Every Phase 3 task → last Phase 2 task (or last Phase 1 if Phase 2 skipped)
- Every Phase 4 task → last Phase 3 task
- Phase 4 tasks within same tag group → sequential
- Every Phase 5 task → last Phase 4 task across ALL tag groups
- Every Phase 6 task → last Phase 5 task
- Phase 7 tasks → last Phase 6 task

---

## Step 7 — Set AI Context

```
mcp__t0ggles__update-project
  boardId:   GvBfmcjk7DXvvaIHurlX
  projectId: <projectId>
  aiContext: |
    Project: [project_name] (rebuilt from [original_name])
    Spec note: [context_note_url]
    Stack: [inferred tech stack — one line]
    [N] phases · [N] tasks — execute in strict order
    To work on any step: read the project context note first,
    then read the phase note linked in the task description.
```

---

## Step 8 — Report

```
✓ Project found:       [project_name]
✓ Swagger parsed:      [original_name] — [N] endpoints · [N] tag groups
✓ Spec notes created:  1 folder · 1 context note · [N] phase notes
✓ Milestones created:  [N] phases
✓ Tasks created:       [N] tasks · [N] total subtasks
✓ Dependencies wired:  [N] links
✓ AI context set

All prompts are in t0ggles → Prompt Projects → [project_name].
Tasks are short — Claude reads the spec notes for full context.
Execute in order. Mark each task Done before moving to the next.
```

---

## Reference Files

- `references/note-template.md` — formats for the Project Context and Phase spec notes
- `references/prompt-template.md` — short task description format and subtask rules
- `references/phase-structure.md` — phase definitions, task breakdown, subtask titles per phase
- `references/t0ggles-ids.md` — board ID, status IDs, tool quick-reference

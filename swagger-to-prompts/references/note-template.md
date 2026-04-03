# Note Templates

Notes are the spec — the single source of truth. Tasks link to notes. To change the plan, edit the note. Tasks automatically reflect it at runtime.

---

## Project Context Note

**Title:** `00 · Project Context`
**Purpose:** Everything Claude needs before starting any step — identity, stack, env vars, schemas.

### Template

```
# [project_name] — Project Context

## Identity Mapping
| Original | Replacement | Apply To |
|----------|-------------|----------|
| [original_name] | [project_name] | package names, class/model names, variable names, string literals, comments, API title/description, folder names |

## Description
[info.description with original_name replaced by project_name.
If empty, write a one-sentence summary inferred from the endpoints and tags.]

## Tech Stack
| Layer | Technology |
|-------|-----------|
| Runtime | [e.g. Node.js 18+, Next.js 15 App Router] |
| Database | [e.g. Convex / MongoDB / PostgreSQL] |
| Auth | [e.g. Clerk / JWT / NextAuth] |
| [add rows for: Background Jobs, Payments, Push, OAuth, etc. if present] |

## Base Path
All API routes prefixed: [servers[0].url — e.g. /api/v1]

## Authentication Schemes
[For each entry in components.securitySchemes:]
- **[name]**: [type] — [how it's used: e.g. "Bearer JWT in Authorization header", "x-api-key header"]

## Environment Variables
[Infer all required env vars from servers, securitySchemes, x- extensions, and description text]

| Variable | Description | Required | Example |
|----------|-------------|----------|---------|
| [VAR_NAME] | [what it's for] | Yes/No | [placeholder value] |

## Schema Reference
[For EACH schema in components.schemas — copy ALL properties verbatim:]

### [SchemaName]
| Field | Type | Format | Required | Constraints |
|-------|------|--------|----------|-------------|
| [fieldName] | [string/integer/boolean/array/object] | [date-time/uuid/email/etc] | [yes/no] | [minLength, pattern, enum, minimum, etc] |

## Endpoint Overview
[One line per tag group:]
- **[Tag]** ([N] endpoints): [one-line summary of what this group does]
```

---

## Phase Spec Note

**Title:** `Phase N · [Phase Title]`
**Purpose:** Full technical details for every step in this phase. Claude reads this at runtime.

### Template

```
# Phase N · [Phase Title]

## What This Phase Builds
[2–3 sentences: goal, outputs, and how it fits into the overall project.]

---

[Repeat the following block for each step in this phase:]

## Step N — [Step Title]

### Files
[List every file this step creates or modifies:]
- `[path/to/file]` — [what it does / its role]

### Specifications
[Pull exact details from swagger. Be exhaustive — the executor reads this instead of the swagger.]

For schemas:
- Field name, type, format, required/optional, all constraints (minLength, pattern, enum, etc.)

For endpoints:
- HTTP method + full path (e.g. POST /api/v1/users/{id}/profile)
- Path params: name, type, required
- Query params: name, type, required, default
- Request body: every field with type, required, constraints
- Response: every field per status code (200, 201, 400, 401, 403, 404, etc.)
- Auth required: yes/no — which scheme

For auth/middleware:
- Library and function names (e.g. clerkMiddleware from @clerk/nextjs)
- Configuration options
- Which routes are protected vs public

For background jobs:
- Function name, trigger event, step names, retry config

### Acceptance Criteria
[Mirror the subtasks on the task exactly — one verifiable statement per line:]
- `[path/to/file]` exists and exports [specific named export]
- `[HTTP METHOD] [path]` returns [status] with [response shape]
- `npm run [script]` completes without errors
```

---

## Content Rules

### What goes in the Project Context note (global)
- Identity mapping
- Tech stack
- ALL env vars with descriptions
- ALL schema definitions with complete field lists
- Auth scheme details
- Base path

### What goes in a Phase spec note (phase-scoped)
- Only the files, endpoints, and schemas relevant to THIS phase
- Complete endpoint contracts (not summaries — full request/response shapes)
- Acceptance criteria that match the task subtasks exactly

### Keep notes current
If the user changes something (e.g. swaps a library, adds a field), edit the relevant note. Tasks don't embed specs — they read notes at runtime, so the change propagates automatically.

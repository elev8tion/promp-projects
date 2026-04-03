# Task Description Template

Every task description is a short Claude Code prompt — max 20 lines. Full specs live in the linked notes. Claude reads them at runtime.

---

## Template

```
## Context
- Project spec: [context_note_url]
- Phase spec:   [phase_note_url]

## Step [N] — [Task Title]

[3–5 sentences: WHAT to build. Name the files. State the outcome.
Do NOT repeat specs from the notes — just say what this step accomplishes
and tell Claude to read the phase spec note for full contracts.]

## Files
  [path/to/file]    — [one-line role]
  [path/to/file]    — [one-line role]

---
Reply "Step [N] complete — [one-line summary of what was built]" and await the next prompt.
```

---

## Field Rules

### Context links
Always two links — Project Context note (identity, stack, schemas) and the Phase spec note (file specs, endpoint contracts). Claude reads both before building.

### Step description (3–5 sentences)
State WHAT gets built, not HOW. Name files. Point to the phase spec for details.

Good:
> Create `src/middleware.ts` — Clerk auth middleware protecting all `/api/*` routes. See the phase spec note for the exact clerkMiddleware configuration, matcher array, and public route list.

Bad (too much inline spec — belongs in the note):
> Set up Clerk middleware using clerkMiddleware from @clerk/nextjs. Configure it with a matcher array of ['/api/(.*)', '/((?!_next|favicon.ico).*)'] and mark /api/health and /api/webhooks/(.*) as public routes...

### Files section
List every file this step creates. The executor uses this as a completion checklist.

### Stop phrase
Always the last line: `Reply "Step N complete — [summary]" and await the next prompt.`
The summary names what was actually built (files, routes, features).

---

## Subtask Format (Acceptance Criteria)

Subtasks are created alongside the task via the `subtasks` array in `create-task`. Each is one verifiable statement. The executor checks these off in t0ggles as they verify.

**3–6 subtasks per task.**

Good (specific and verifiable):
- `` `src/middleware.ts` exists and exports `clerkMiddleware` ``
- `` `GET /api/health` returns `{ status: "ok" }` without an auth token ``
- `` `GET /api/sessions` returns `401` with no Authorization header ``
- `` `npm run dev` starts without errors on port 3000 ``
- `` No references to "[original_name]" appear in any created file ``

Bad (too vague to verify):
- "Middleware is configured"
- "Auth works correctly"
- "Tests pass"

---

## Example

**Task description:**
```
## Context
- Project spec: https://t0ggles.com/prompt-projects/notes/00-project-context-abc123
- Phase spec:   https://t0ggles.com/prompt-projects/notes/phase-1-foundation-def456

## Step 1 — Initialize project and install dependencies

Create `package.json` and `.gitignore` at the project root. Install all dependencies
listed in the Phase 1 spec note. The package name must be `nexus-commerce` — no
references to the original name anywhere.

## Files
  package.json    — project manifest with name, version, scripts (start, dev, test), and all deps
  .gitignore      — excludes node_modules, .env, dist, coverage

---
Reply "Step 1 complete — package.json created, all dependencies installed" and await the next prompt.
```

**Subtasks for this task:**
- `` `npm install` runs without errors ``
- `` `package.json` `name` field is `"nexus-commerce"` ``
- `` `.gitignore` excludes `node_modules` and `.env*` ``
- `` No references to the original project name in any file ``

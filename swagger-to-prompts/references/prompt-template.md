# Claude Code Prompt Template

Every task description created by this skill IS the Claude Code prompt. Use this exact structure for every task.

---

## Template

```
## Context
You are building **[PROJECT_NAME]** — [info.description with original name replaced by PROJECT_NAME].

This is Step [N] of a systematic rebuild. Complete ONLY the tasks listed below, then stop and confirm completion. Do not implement anything beyond what is specified here.

---

## Task: [Task Title]

## What to Build
[Precise description of what to create. List every file by path. Describe what each file contains and its role in the project.]

## Specifications
[Pull exact details from swagger. Be exhaustive. Include:]
[- Schema: property name, type, format, required/optional, constraints]
[- Endpoints: HTTP method, full path, path params, query params, request body fields, response fields, HTTP status codes, auth requirement]
[- Any x- extension fields or description text from the swagger that clarifies intent]

## Identity Mapping
- Original project name: "[ORIGINAL_NAME]" → Replace with: "[PROJECT_NAME]"
- Apply to: package names, class/model names, variable names, string literals, comments, README text, API title/description fields, folder names if project-named

## File Structure
[List every file this task creates or modifies:]
[  src/models/user.js      — User mongoose schema]
[  src/models/index.js     — Model exports]

## Acceptance Criteria
- [ ] [Specific verifiable deliverable — one per file or feature]
- [ ] All files exist at the correct paths
- [ ] No references to "[ORIGINAL_NAME]" remain in any created file
- [ ] [Any framework/library-specific check, e.g. "schema passes validation", "server starts without errors"]

---

Reply "Step [N] complete — [one-line summary of what was built]" and await the next prompt.
```

---

## Field Rules

### `[PROJECT_NAME]`
The new identity provided by the user. Used **everywhere** — in the Context block, in every file reference, in class/function names, in string literals. The original name must never appear.

### `[ORIGINAL_NAME]`
From `info.title` in the swagger. Appears **only** in the Identity Mapping block as the source to replace. Never used anywhere else in the prompt.

### `[N]` — Step Number
Sequential integer across all phases and tasks. Step 1 is the first task of Phase 1. Continues unbroken across phases (Step 7 might be the first task of Phase 2, for example).

### Context block
One sentence describing what the project does, sourced from `info.description`. Replace any occurrence of the original name. If description is empty, write a one-sentence summary inferred from the swagger's endpoints and tags.

### What to Build
Be precise. List file paths. Don't say "create the user model" — say "create `src/models/user.js` which defines a Mongoose schema with the following fields:". The person executing this prompt has no prior context.

### Specifications
This is where the swagger data goes. Copy field names, types, formats, and constraints exactly. For endpoints, include the full path (e.g. `POST /api/v1/users/{id}/profile`), every parameter, the request body shape, and every response shape. Never summarize — be complete.

### File Structure
List every file the task creates. This lets the executor verify completeness without reading the whole prompt twice.

### Acceptance Criteria
Write one checkbox per meaningful deliverable. These should be copy-paste verifiable. Bad: "✓ User model created". Good: "✓ `src/models/user.js` exports a Mongoose model named `User` with fields: id (ObjectId), email (String, required, unique), passwordHash (String, required), createdAt (Date, default: Date.now)".

### Stop Phrase
Always the last line: `Reply "Step N complete — [summary]" and await the next prompt.`

The summary in the stop phrase should name what was built (e.g. "Step 3 complete — JWT middleware, auth routes /login /register /refresh /logout, and authentication guard created").

---

## Example (Phase 1, Task 1)

```
## Context
You are building **Nexus Commerce** — a RESTful e-commerce API providing product catalog, cart, order management, and user authentication.

This is Step 1 of a systematic rebuild. Complete ONLY the tasks listed below, then stop and confirm completion. Do not implement anything beyond what is specified here.

---

## Task: Initialize project structure and install dependencies

## What to Build
Create the following files and directories:
- `package.json` — Node.js project manifest with all dependencies
- `.gitignore` — Standard Node.js gitignore
- `src/` — Source root directory
- `src/index.js` — Entry point (imports app, starts server)
- `src/app.js` — Express app factory (no server.listen here)

## Specifications
Framework: Express.js
Runtime: Node.js
Required dependencies:
  - express (web framework)
  - mongoose (MongoDB ODM)
  - dotenv (environment config)
  - bcryptjs (password hashing)
  - jsonwebtoken (JWT auth)
  - express-validator (input validation)
  - cors (CORS middleware)
  - helmet (security headers)
  - morgan (HTTP logging)
Dev dependencies:
  - nodemon
  - jest
  - supertest

Base path: /api/v1 (all routes prefixed here)
Server port: from process.env.PORT, default 3000

## Identity Mapping
- Original project name: "ShopFlow API" → Replace with: "Nexus Commerce"
- Apply to: package.json name field, app title in comments, any string that says "ShopFlow"

## File Structure
  package.json          — project manifest, scripts: start, dev, test
  .gitignore            — node_modules, .env, dist
  src/index.js          — loads dotenv, imports app, calls app.listen
  src/app.js            — creates express instance, mounts /api/v1 router stub, exports app

## Acceptance Criteria
- [ ] `npm install` runs without errors
- [ ] `npm run dev` starts the server on port 3000
- [ ] `GET /health` returns `{ status: "ok", project: "Nexus Commerce" }`
- [ ] No references to "ShopFlow" appear in any file
- [ ] package.json `name` field is `"nexus-commerce"`

---

Reply "Step 1 complete — project initialized with Express, dependencies installed, base server running" and await the next prompt.
```

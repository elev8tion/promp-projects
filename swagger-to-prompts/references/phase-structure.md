# Phase Structure & Task Rules

## Phase Overview

| # | Phase Title | Included When | Typical Tasks |
|---|-------------|---------------|---------------|
| 1 | Project Foundation | Always | 4–5 |
| 2 | Data Models & Schemas | schemas non-empty | 2–4 |
| 3 | Authentication & Security | securitySchemes non-empty | 2–3 |
| 4 | Core API — [Tag] | One per tag group | 1–3 per tag |
| 5 | Middleware & Utilities | Always | 2–3 |
| 6 | Testing | Always | 3–5 |
| 7 | Documentation & Deployment | Always | 2 |

Renumber sequentially if phases are skipped (e.g. no auth → Phase 3 becomes "Phase 3: Core API — Users").

**Priority:** Phase 1–3 = `high` · Phase 4–5 = `medium` · Phase 6–7 = `low`

**Semantic tags** (add alongside "Phase N"):
- Phase 1: `setup`
- Phase 2: `database`
- Phase 3: `auth`
- Phase 4: `api`
- Phase 5: `middleware`
- Phase 6: `testing`
- Phase 7: `deploy`

---

## Phase 1: Project Foundation

### Phase Spec Note content
Include in the Phase 1 note:
- Framework + runtime version (e.g. Next.js 15 App Router, Node 18+)
- Full dependency list: name + version + purpose (separate dev deps)
- Complete folder structure diagram (every directory and its role)
- Entry point and app factory pattern
- Health check route spec: path, method, response shape
- Port config (env var name + default)
- Any setup commands (e.g. `npx convex dev`, `npx prisma generate`)

### Tasks

1. **Initialize project and install dependencies**
   Files: `package.json`, `.gitignore`
   Subtasks:
   - `npm install` completes without errors
   - `package.json` `name` field matches `[project_name]` in kebab-case
   - No references to `[original_name]` in any file

2. **Create folder structure and scaffold entry points**
   Files: `src/` tree (all directories), `src/index.js`, `src/app.js` (or framework equivalent)
   Subtasks:
   - All directories from the spec note exist
   - Entry point file imports app and calls server start
   - `GET /health` returns `{ status: "ok", project: "[project_name]" }`

3. **Configure environment variables**
   Files: `.env.example`, config loader module
   Subtasks:
   - `.env.example` contains every variable from the Project Context note
   - Config loader reads from `process.env` with defaults
   - App starts with only `.env.example` values (no hard-coded secrets)

4. **Connect database** *(only if swagger implies a DB)*
   Files: DB connection module
   Subtasks:
   - Connection module exports a connect function and graceful shutdown
   - Server logs "Database connected" on startup
   - Connection error exits with code 1

---

## Phase 2: Data Models & Schemas

### Phase Spec Note content
For each schema group covered in this phase:
- Full schema definition (all fields from Project Context note, verbatim)
- ORM/validation library to use (inferred from stack)
- Input validation rules derived from OpenAPI constraints
- Relationships to other schemas (foreign keys, refs)

### Task-splitting rules
- Group related schemas: `User` + `UserProfile` + `UserPreferences` = one task
- Max 5 schemas per task
- Keep DTOs (`CreateUserInput`, `UpdateUserInput`) with their parent entity
- Order: core entities first → join/relational models → pure DTOs

### Tasks (one per schema group)
Each task title: `Define [Entity] schema and model`
Files: `src/models/[entity].js` (or ORM equivalent)
Subtasks:
- Model file exists at the correct path
- All required fields from the spec note are present with correct types
- Optional fields are nullable/optional
- Validation rejects missing required fields
- Model exports match the import pattern used in routes

---

## Phase 3: Authentication & Security

### Phase Spec Note content
For each security scheme:
- Library name + install command
- Token format, expiry, signing secret env var name
- Middleware function signature
- Which routes are protected vs public (list both explicitly)
- 401 and 403 response shapes
- Register/login/refresh/logout endpoint contracts (full request + response)

### Tasks

1. **JWT/session setup** *(if bearerAuth)*
   Files: `src/lib/auth.js`, `src/middleware/auth.js`
   Subtasks:
   - Token sign function accepts `{ userId }` and returns signed JWT
   - Token verify function returns payload or throws on invalid/expired
   - Auth middleware attaches user to `req.user` on success
   - Auth middleware returns 401 JSON on missing/invalid token

2. **Auth routes** *(register, login, refresh, logout)*
   Files: `src/routes/auth.js`
   Subtasks:
   - `POST /[base]/register` returns 201 with user + token
   - `POST /[base]/login` returns 200 with token or 401 on bad credentials
   - `POST /[base]/refresh` returns new token or 401
   - `POST /[base]/logout` returns 200

3. **API key middleware** *(if apiKey scheme)*
   Files: `src/middleware/apiKey.js`
   Subtasks:
   - Reads key from header specified in swagger
   - Returns 401 if missing, 403 if invalid
   - Passes to next() if valid

---

## Phase 4: Core API — [Tag]

### Phase Spec Note content
For EACH endpoint in this tag group:
- HTTP method + full path (e.g. `GET /api/v1/users/{id}`)
- Path params: name, type, required
- Query params: name, type, required, default value
- Request body: every field, type, required/optional, constraints
- Response per status code: every field with type (200, 201, 400, 401, 403, 404, 422, etc.)
- Auth required: yes/no — which scheme applies

### Task-splitting rules
- One task per tag group by default
- Split into Task A + Task B if the group has **more than 8 endpoints**:
  - Task A: CRUD (GET list, GET by ID, POST, PUT/PATCH, DELETE)
  - Task B: Specialized (search, bulk, nested resources, actions)

### Each task includes
Files: `src/routes/[tag].js` + `src/controllers/[tag].js` (or framework equivalent)
Subtasks (3–5):
- Route file registered in the app router
- Each endpoint returns the correct status code on the happy path
- Auth-protected endpoints return 401 with no token
- Input validation returns 400 on missing required fields
- Response shape matches the spec note exactly

---

## Phase 5: Middleware & Utilities

### Phase Spec Note content
- Error response format (`{ error, message, statusCode }` or project-specific)
- CORS origins (from env var, list any swagger-specified origins)
- Security headers library and configuration
- Log format (timestamp, method, path, status, duration)
- Rate limit config if applicable (window, max requests, env var names)

### Tasks

1. **Global error handling and 404 handler**
   Files: `src/middleware/errorHandler.js`
   Subtasks:
   - All unhandled errors return `{ error, message, statusCode }` shape
   - 404 handler returns 404 JSON for unknown routes
   - Stack traces suppressed in production (`NODE_ENV=production`)

2. **CORS and security headers**
   Files: `src/middleware/security.js`
   Subtasks:
   - CORS origins read from `CORS_ORIGINS` env var
   - Security headers applied (helmet or equivalent)
   - Preflight OPTIONS requests return 204

3. **Request logging** *(if stack implies it)*
   Files: `src/middleware/logger.js`
   Subtasks:
   - Every request logs: method, path, status, duration
   - Errors logged with stack trace in development

---

## Phase 6: Testing

### Phase Spec Note content
- Test framework and config (Jest/pytest/go test — version, config file)
- Test DB setup (in-memory vs real connection, teardown)
- Fixture data for each major entity (sample values matching schema constraints)
- Mock patterns for external services (auth, payments, third-party APIs)
- Helper utilities (authenticated request builder, test server setup)

### Tasks

1. **Test setup, config, and fixtures**
   Files: `jest.config.js`, `__tests__/setup.js`, `__tests__/fixtures/`
   Subtasks:
   - `npm test` runs without errors on a clean setup
   - Test DB connects and tears down cleanly
   - Auth helper returns a valid test token

2. **Schema/model unit tests**
   Files: `__tests__/models/`
   Subtasks:
   - Required field missing → validation error
   - All field types enforced
   - Optional fields accept null/undefined

3. **Auth integration tests**
   Files: `__tests__/api/auth.test.js`
   Subtasks:
   - Register returns 201 with token
   - Login with bad password returns 401
   - Protected route with valid token returns 200
   - Protected route with no token returns 401

4. **API integration tests** *(one task per tag group)*
   Files: `__tests__/api/[tag].test.js`
   Subtasks:
   - Happy path for each endpoint returns correct status + shape
   - Missing required field returns 400
   - Auth-protected endpoints return 401 with no token
   - 404 for unknown resource ID

---

## Phase 7: Documentation & Deployment

### Phase Spec Note content
- All env vars (final complete list from Project Context note)
- Deployment platform (infer from stack: Railway, Fly.io, Render, Vercel, Docker)
- Build command, start command, health check path
- Any platform-specific config format (Dockerfile, Procfile, vercel.json)
- README sections: project name, description, setup steps, env vars table, API overview, example requests

### Tasks

1. **README and .env.example**
   Files: `README.md`, `.env.example`
   Subtasks:
   - README includes: project name, description, setup steps, env vars table, API overview
   - `.env.example` contains every variable from the Project Context note
   - No real secrets in `.env.example`
   - No references to `[original_name]` in any documentation file

2. **Deployment config**
   Files: `Dockerfile` (multi-stage), `docker-compose.yml` — or platform equivalent
   Subtasks:
   - `docker build` completes without error
   - `docker-compose up` starts the app
   - `GET /health` returns 200 in the containerized environment
   - All env vars passed through to the container
